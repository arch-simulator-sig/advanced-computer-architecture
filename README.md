# Advanced computer architecture


## 高级体系结构研讨会

| 日期 |                             主题                             |                    视频                     |  成员  |
| :--: | :----------------------------------------------------------: | :-----------------------------------------: | :----: |
| 9.22 |    [超标量处理器系列1 cache](./超标量处理器设计/cache.md)    | https://www.bilibili.com/video/BV1z94y1p7kc | 段震伟 |
| 9.22 | [tilelink入门](https://sagca6ucd2p.feishu.cn/docx/TbABd17ZYoryH8xpWNHcyL73noe) | https://www.bilibili.com/video/BV11N411J7Ty | 丁庆辰 |
| 9.22 |         [cva6乱序执行原理](cva6/cva6乱序执行原理.md)         | https://www.bilibili.com/video/BV1SK4y1F76t | 李子龙 |
| 10.8 |    [cva6架构剖析](./cva6/cva6.md)                | https://www.bilibili.com/video/BV1vG411m7Ft   |唐德宇 |
| 10.8 |    [asim cache解读](./asim/asim.md)              | https://www.bilibili.com/video/BV1xG411m75R  | 朱子谦 |
| 10.8 |    [简单流水线冒险的处理](https://sxl2g9eu0e.feishu.cn/docx/Cy70dffCHonymfxw906cxTNsnnp)              |         | 王京 |
| 10.8 | [香山南湖架构前端解读](./xiangshan/frontend.pdf) |https://www.bilibili.com/video/BV1PN411b7od | 蒋晓天 |
| 10.20 | [微处理器性能分析与优化 上](loongson/微处理器性能分析与优化.md) |https://www.bilibili.com/video/BV1RN411x7MF | 段震伟 |
| 10.20 | [RocketChip DCache分析](rocketchip/Rocket-DCache.pdf) |https://www.bilibili.com/video/BV1HH4y197jt | 丁庆辰 |
| 1.29 | [乱序发射相关基础](./超标量处理器设计/issue_basis.pdf) |https://www.bilibili.com/video/BV1m2421w7tm | 刘汉章 |
| WIP | [超标量处理器设计剩余内容](https://www.zhihu.com/column/c_1772393272914403328) | | 段震伟 |



## 一生一芯高阶体系结构培训大纲

注：时长为通过ysyx B线之后所需时间，每周约40-50h+

### Lab0 [2 month]

- [ ] 微架构 : RV64GC (IMACFA) + MSU + AXI4(burst) + TLB + Cache(un blocked) + BPU (Tournament)
- [ ] 性能 : Coremark 跑分优化 （hint: 硬件计数器）性能要求：coremark IPC 0.6+ , Freq 100M+
- [ ] 外设 : CLINT + PLIC + UART
- [ ] 对齐 : Function model + Perf model
- [ ] 软件测试 : riscv-tests + cpu-tests + coremark + dhrystone + microbench + RT-thread + nommu-Linux + Linux
- [ ] 测试流程 : verilator + vcs + dc + FPGA

注：可以调用rocket-chip api减少工作量，[参考框架](https://github.com/arch-simulator-sig/chisel-env)


### Lab1 [1 month]
软件基础强化
1. [quardStar tutorial](https://github.com/arch-simulator-sig/quard-star-tutorial-2021) [2 week]
1. 运行xv6-riscv
1. 移植和运行Linux

### Lab2
顺序多发 + 性能分析 + 模拟器(not gem5) + 分析后端 + Fpga

参考架构 ridecore，有中文文档

### Lab3
Lab2 + 多核

### Lab4 
Lab3 (Fork Nanhu) , 可联系 dzwduan@163.com 报名，名额有限


### Lab5
Lab4 + PPA (低功耗RTL Fork E203, 模拟器 Cacti/sparta)

### 参考内容

Lab0

1. [yatcpu doc](https://yatcpu.sysu.tech/) and [Lab Axi+CSR+Pipeline+OS](https://github.com/hrpccs/2022-fall-yatcpu-repo)

1. [gatemate-riscv related about bpu and soc](https://github.com/fm4dd/gatemate-riscv)

1. [Nutshell rv64imac + boot Linux](https://github.com/OSCPU/NutShell)

1. [Zhoushan 2-way ooo superscalar](https://github.com/OSCPU-Zhoushan/Zhoushan)

1. [cva6 Labs](https://github.com/sifferman/labs-with-cva6)

1. cpu设计实战 + [openla500](https://gitee.com/loongson-edu/nscscc-openla500)

    

    

### 乱序相关参考

1. [18-740](https://course.ece.cmu.edu/~ece740/f10/doku.php?id=lectures)
1. [南京大学乱序讲义](https://cs.nju.edu.cn/swang/CA_16S/index.htm)
1. [brief into ooo](https://jia.je/tags/#brief-into-ooo)
1. [nop-processor](https://github.com/NOP-Processor/NOP-Core)
1. 现代处理器设计-超标量处理器基础  + [rsd-core](https://github.com/rsd-devel/rsd) + [ridecore](https://github.com/dzwduan/ridecore)
1. 超标量处理设计 + [zhengliu](https://gitee.com/liangliang678/ZhengLiu) + [la32r-pipeline](https://github.com/MaZirui2001/LA32R-pipeline-scala) + [Bergamot](https://github.com/LoveLonelyTime/Bergamot) + [soomRV](https://github.com/mathis-s/SoomRV)
1. [NaxRiscv](https://spinalhdl.github.io/NaxRiscv-Rtd/main/NaxRiscv/introduction/index.html)
1. [boom](https://github.com/riscv-boom/riscv-boom)
1. [xiangshan](https://github.com/OpenXiangShan/XiangShan) + 香山源代码剖析
1. [vRoom](https://github.com/MoonbaseOtago/vroom)
1. [openc910](https://github.com/T-head-Semi/openc910)
1. [高性能cpu架构1](https://www.bilibili.com/video/BV1y3GMeDEnJ) + [高性能cpu架构2](https://www.bilibili.com/video/BV1we48ejEfd) + [架构图](https://1drv.ms/f/s!AiW5eJ6PHsfwlusK5NwzjGa59ZI4nw?e=Yt0H6a)
1. 基于RISC-V指令集的超标量处理器设计与实现

