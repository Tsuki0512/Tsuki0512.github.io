# 进程管理

## 基本概念

- 进程是资源分配和管理的独立单位

## 进程的形式

- 内核态虚拟内存 - 元数据（进程控制块PCB）
  - <img src="./markdown-img/quiz2准备_调度与同步.assets/image-20241127232049530.png" alt="image-20241127232049530" style="zoom:50%;" /> 
- 用户态虚拟内存 - 实际需要的数据资源
  - 静态部分
    - text section - 代码
    - data section - 进程全局变量、静态变量
  - 动态更新部分
    - heap - 被动态分配的内存
    - stack - 暂时性数据，函数传参、返回值等

