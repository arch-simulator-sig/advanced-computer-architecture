# cva6流水线（主线）

[toc]

> 参考：
>
> https://docs.openhwgroup.org/projects/cva6-user-manual/03_cva6_design/index.html
>
> https://zhuanlan.zhihu.com/p/496078836

## Intro

cva6是一颗具备6级流水、顺序发射、乱序执行、顺序提交的RISC-V CPU。利用scoreboard技术，避免WAW、WAR、RAW数据依赖性，通过动态调度流水线的方式，实现乱序执行，提高流水线效率。

流水线各级之间大致通过valid/ready，以级间握手的方式，加上全局的控制器（实现非相邻流水段之间的信号传递）实现流水。

## Frontend（PC-Gen+IF）

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230927173918768.png" alt="image-20230927173918768" style="zoom:50%;" />

前端被I$划分为PC-Gen和IF两个阶段

### PC-Gen

**生成PC阶段**阶段用于生成`fetch_addr`，即I$的取下一条指令的地址。`fetch_addr`来源有如下几个：

1. `predict_addr`：分支预测结果地址
2. `fetch_addr+4`：顺序执行的下一条指令地址
3. `replay_addr`：当指令队列满了，新的指令无法入队，需要重新取这条指令
4. `resolved_addr`：当分支预测失败，需要从正确的分支目标地址开始取指
5. `epc`：环境调用的返回地址
6. `tv_base`：中断/异常的入口地址
7. `pc_commit`：某些CSR/AMO指令会在commit阶段才知道要冲刷流水线，并通知前端从该指令的下一条开始取指
8. `debug_pc`：debug地址，优先级最高

### IF

**取指阶段**用PC-Gen生成的`fetch_addr`发送给I$取指，并对指令做一些处理，比如指令对齐、分支预测工作。并设置一个指令队列FIFO用于解耦前段和后端逻辑，接收来自前端IF阶段处理后的指令，并发送给后端ID阶段。取指具体步骤如下：

1. 等待$I获取到指令。
2. 将指令输入到指令对齐模块，并且将指令对应的虚拟地址输入分支预测模块，分支预测包含了BTH、BTB和RAS模块，分支预测模块将给出分支预测结果。
3. 将对齐后的指令做简单的跳转相关解析，通过解析结果更新RAS、通过上一步的分支预测结果确定控制流类型（jump、branch...）、分支目标地址。同时将对齐后的指令输入指令队列。
4. 指令队列根据fifo的状态、flush等，决定IF是否需要replay（重新取指）、输出最终取指结果到后端。

## ID

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230927141752757.png" alt="image-20230927141752757" style="zoom:50%;" />

**译码阶段**负责将取指阶段得到的指令经过decoder进行译码，输出`scoreboard entry`，即一条指令对应一条计分板条目，后续的阶段都会根据计分板运行。

`scoreboard entry`包含的内容主要如下：

- 指令地址
- entry在scoreboard中的索引，在写回的时候用于找回entry
- 指令使用到的功能单元、做什么运算
- 源寄存器的目的寄存器的地址
- 写回结果寄存器
- 操作数b的是否使用imm
- 操作数a是否使用zimm或pc
- 异常
- 分支预测
- 是否压缩指令

| pc       | trans_id                  | fu/op                            | rs1/rs2/rd               | result/valid                                                 | use_imm            | use_zimm/use_pc         | ex   | bp           | is_compressed |
| -------- | ------------------------- | -------------------------------- | ------------------------ | ------------------------------------------------------------ | ------------------ | ----------------------- | ---- | ------------ | ------------- |
| 指令地址 | entry在scoreboard中的索引 | 指令使用的功能单元以及相应的运算 | 指令的源和目的寄存器地址 | 算是一个保存misc data的寄存器：存放指令需要用到的立即数、地址等 | 操作数b是否使用imm | 操作数a是否使用zimm或pc | 异常 | 分支预测结果 | 是否压缩指令  |



## ISSUE

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230927174226941.png" alt="image-20230927174226941" style="zoom:50%;" />

**发射阶段**负责将解码后的指令，并且准备好操作数，发送给相应的FU。发射之后就是执行了，那么就得保证发射出去的数据都是正确的，得解决数据冒险。`scoreboard`提供了整个后端数据流的上帝视角，基于scoreboard可以解决数据冒险的发生的阻塞，在cva6中scoreboard以fifo的方式组织，发射一条指令则入队，提交一条指令则出队。

`scoreboard`还对外暴露一个名为`rd_clobber`的信号，其内容是所有逻辑寄存器被fu占用的情况（由此可知`rd_clobber`的命名含义是目的寄存器的破坏者是谁），比如有一条指令需要使用alu，并将运算结果放到x3寄存器中，那么`rd_clobber`的内容如下：（x6~x31省略）

| x0   | x1   | x2   | x3   | x4   | x5   |
| ---- | ---- | ---- | ---- | ---- | ---- |
| none | none | none | alu  | none | none |

利用`rd_clobber`可以很方便地检测WAW并只允许一条指令写入特定的目的寄存器，简化了forawrding的设计。

另外发射阶段还有模块`issue_read_operands`负责读写寄存器堆、决定运算操作数以及最终的发射（与EX进行握手等）。

发射一条指令的步骤如下：

1. scoreboard检测从ID传来解码后的指令有效、所需的fu空闲、没有其他指令要写入同一个寄存器三个条件同时成立（还有一些短路情况，就不在此列举了），将指令入队scoreboard
2. 更新rd_clobber
3. 选择读寄存器、scoreboard旁路、立即数等其中之一作为FU的操作数
4. 与FU握手，指设置valid等，开始执行（always_ff）



## EX

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230927174301725.png" alt="image-20230927174301725" style="zoom:50%;" />

**执行阶段**负责执行具体的操作，包含所有的FU（ALU、分支单元、加载存储单元(LSU)、CSR缓冲器和乘除单元），每个FU都通过ready/valid与发射模块独立进行握手，并独立执行没有数据依赖。由于每个FU的执行周期可能不同，因此将结果乱序写回scoreboard，但最终是顺序提交并将结果数据写回regfile/cache。

## COMMIT

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230927174353248.png" alt="image-20230927174353248" style="zoom:50%;" />

**提交阶段**负责更新体系结构的状态，包括写回寄存器堆、写入内存等行为，这一步称为指令的提交，指令一旦提交，便不可逆转，提交的同时将scoreboard队列出队，即commit指针+1。另外，为了中断精确发生，前面阶段所发生的异常都推迟到提交阶段处理。

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/202309221422812.png" alt="image.png" style="zoom:50%;" />



## Controller（全局控制器）

<img src="https://blog-1252412046.cos.ap-shanghai.myqcloud.com/image-20230928181459368.png" alt="image-20230928181459368" style="zoom:50%;" />

**控制器**负责控制流水线的冲刷、暂停等。比如收到提交`fence.i`指令信号的时候：

``` systemverilog
        // ---------------------------------
        // FENCE.I
        // ---------------------------------
        if (fence_i_i) begin
            set_pc_commit_o        = 1'b1;
            flush_if_o             = 1'b1;
            flush_unissued_instr_o = 1'b1;
            flush_id_o             = 1'b1;
            flush_ex_o             = 1'b1;
            flush_icache_o         = 1'b1;
// this is not needed in the case since we
// have a write-through cache in this case
            if (DCACHE_TYPE == int'(cva6_config_pkg::WB)) begin
              flush_dcache           = 1'b1;
              fence_active_d         = 1'b1;
            end
        end
```

控制器会通知冲刷之前所有的流水段，冲刷icache，并用`set_pc_commit_o`信号通知pc-gen模块从被提交指令所在地址的之后的一条指令开始取指。

## 运行cva6前遇到的一些坑

- 装rv工具链的时候，可能会因为网络原因导致submodule下载很慢，而submodule下载的时候默认是没有进度条提示的，最好手动修改一下Makefile中的git submodule update，添加一个--progress参数，这样就能看到速度和进度条了，当你发现速度不对劲的时候，停下来并挂个梯子。下载+编译整个过程可能需要花费数小时
- 构建cva6的verilator模型时，经过多次更换verilator版本后，v4.110最合适，另外bison版本可能也要降到3.5.1
- make verilate的时候，需要加上TRACE_FAST=1才能生成波形（Makefile中会给verilator加上--trace选项）。如果不是用它提供脚本来安装verilator，需要手动指定VERILATOR_INSTALL_DIR，VL_INC_DIR最好也手动指定一下，比如我的VL_INC_DIR := /opt/verilator/include
