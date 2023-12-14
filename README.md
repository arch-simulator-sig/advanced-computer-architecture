# Advanced computer architecture



## Plan

riscv-mini(熟悉chisel) -> [cva6 Labs](https://github.com/sifferman/labs-with-cva6)(A线) -> rocket-core(非阻塞cache) -> boom(都有) -> [vRoom](https://github.com/MoonbaseOtago/vroom)(多核) -> Xiangshan(世界最高峰）




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



## 一生一芯高阶体系结构培训大纲

注：时长为通过ysyx B线之后所需时间，每周约40-50h+

### Lab0 [1.5 month]
微架构：RV64IMAC + AXI4(not Lite) + TLB + Cache(blocked) + BPU(gshare)

1. Learn scala + chisel, 能看懂Nutshell以及OpenXiangShan/Utility全部语法为达标
1. Pipeline + AXI4(not Lite) + TLB + Cache + BPU, difftest接入DRAMSim3
1. Coremark 跑分优化 （hint: 硬件计数器）性能要求：coremark IPC 0.6+
1. 移植RT-thread
1. 要求: 最终实现的处理器不要与任何一个参考核雷同！
### Lab1 [1 month]
软件基础强化
1. [quardStar tutorial](https://quard-star-tutorial.readthedocs.io/) [2 week]
1. 运行xv6-riscv
1. 移植和运行Linux

### Lab2
顺序多发 + 性能分析 + 模拟器(not gem5) + 分析后端 + Fpga

参考架构 ridecore，有中文文档

### Lab3
Lab2 + 多核

### Lab4 
Lab3 + 乱序 (Fork nanhu/kunminghu) 全流程


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
1. [rsd-core](https://github.com/rsd-devel/rsd)
1. [zhengliu](https://gitee.com/liangliang678/ZhengLiu)
1. [la32r-pipeline](https://github.com/MaZirui2001/LA32R-pipeline-scala)
1. [boom](https://github.com/riscv-boom/riscv-boom)
1. [xiangshan](https://github.com/OpenXiangShan/XiangShan)
1. [openc910](https://github.com/T-head-Semi/openc910)

