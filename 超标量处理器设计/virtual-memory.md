# 虚拟存储器

## 概述

==为什么需要虚拟地址？==
应用太大，物理内存无法一次性全部装入，因此需要换入换出，使得产生更大空间的假象，该空间即为虚拟地址空间，大小和处理器位数相关。

<img src="./../image/virtual-memory/image-20230919193726337.png" alt="image-20230919193726337" style="zoom: 67%;" />

特征：
1. 应用会产生独占处理器空间的假象

2.   安全性

     

## 地址转换

基于分页
名词解释 ：VPN(virtual page number)    FPN (physical frame number)

<img src="./../image/virtual-memory/image-20230921162448216.png" alt="image-20230921162448216" style="zoom:67%;" />

### 单级页表

<img src="./../image/virtual-memory/image-20230921165350566.png" alt="image-20230921165350566" style="zoom:67%;" />

==为什么不和cache一样，在页表加入tag?==  

多进程时的地址转换

<img src="./../image/virtual-memory/image-20230921185724922.png" alt="image-20230921185724922" style="zoom: 50%;" />

访问两次物理内存得到数据

<img src="./../image/virtual-memory/image-20230921185910959.png" alt="image-20230921185910959" style="zoom:67%;" />

### 多级页表

==为什么需要多级页表== 

单级页表占据连续物理空间，但是多级页表中相邻子页表不连续存放在物理内存中，提高了利用率

多级页表流程

<img src="./../image/virtual-memory/image-20230921191915544.png" alt="image-20230921191915544" style="zoom:50%;" />

页表具有易拓展性，处理器位数增加，通过拓展页表级数完成

<img src="./../image/virtual-memory/image-20230921201335805.png" alt="image-20230921201335805" style="zoom: 50%;" />

但是页表级数增多，访问物理内存次数也增加，下图两级页表，访问3次物理内存

<img src="./../image/virtual-memory/image-20230921201917891.png" alt="image-20230921201917891" style="zoom:50%;" />

### page falut

访问页表时，发现valid为0，即没有存放映射关系，需要从下一级存储器取出对应页放到物理内存中。

==page fault 基于硬件还是软件？== 1. 访问硬盘时间太久  2. 硬件的页替换算法不够灵活

==基于软件，硬件需要提供哪些支持？==  主要是替换算法

<img src="./../image/virtual-memory/image-20230921204712849.png" alt="image-20230921204712849" style="zoom:50%;" />

不发生page fault的访存流程

<img src="./../image/virtual-memory/image-20230921204757337.png" alt="image-20230921204757337" style="zoom: 67%;" />

发生page fault的访存流程 （考虑dirty)

<img src="./../image/virtual-memory/image-20230921204819609.png" alt="image-20230921204819609" style="zoom:67%;" />



## 程序保护
