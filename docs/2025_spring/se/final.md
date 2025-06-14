# 软件工程 知识点

## 第一章 软件的本质

- 软件的定义是构成一个配置的一系列条目或对象的集合
    - 指令
    - 数据结构
    - 文档
- 特性：损耗与退化
    - no wear out but deteriorate
    - 失效率 - 浴盆曲线
    - 退化：持续变更引入新错误导致质量下降
    - 无备件
- 应用类型
    - 系统软件
    - 应用软件
    - 工程/科学软件
    - 嵌入式软件
    - 产品线软件
    - web/移动应用
    - 人工智能软件
- 遗漏软件
    - 长生命周期
    - 业务关键性
    - 如果重大变更 —— Reengineering
- 软件神话
    - 错误观念

## 第二章 软件工程

- 层次化技术
    - 质量关注 quality focus
    - 过程 process - 管理和组织开发
    - 方法 methods - 开放和分析软件的技术和步骤
    - 工具 tools - 支持过程和方法的自动化工具
- 过程框架的五个活动
    1. 沟通
    2. 策划
    3. 建模
    4. 构建
    5. 部署
- 七项通用原则
    - 存在皆为价值
    - 保持简单
    - 保持愿景
    - 他人将消费你所生产的
    - 拥抱未来
    - 为复用提前计划
    - 思考

- umbrella activities - 软件工程的支撑活动，项目管理、质量保证、配置管理等

- agile process models - 敏捷过程模型

## 第三章 软件过程结构

- 四种过程流 process flow
    - 线性 linear
    - 迭代 iterative
    - 演化 evolutionary
    - 并行 parallel
- 过程模式 - 在软件工程上下文中一致地描述问题的解决方案
    - 模式名称 pattern name
    - 意图 intent - 模型遇到的环境和问题
    - 类型
        - 阶段模式 stage pattern - 过程框架活动
        - 任务模式 task pattern - 软件工程动作或工作任务
        - 制品模式 phase pattern - 框架活动的顺序
    - 初始背景
    - 问题
    - 方案
    - 结果背景
    - 相关模式
    - 已知应用和示例
- 过程评估 - 软件过程的当前状态
    - 通用标准：ISO 9001:2000 for software & SPICE
    - CMMI
        1. 初始级 initial
        2. 已管理级 managed
        3. 已定义级 defined
        4. 已量化管理级 quantatively managed
        5. 优化级 optimizing

## 第四章 过程模型

*不允许skip*

- 规定性过程模型
    - 瀑布模型 waterfall model - 系统的、顺序的软件开发方法
        - *需求定义明确时的一种合理的方法*
    - 增量过程模型 incremental process model - 每个线性序列产生一个可交付的软件
        - *需要快速得到可运行的核心产品的一种良好方法*
        - RAD模型 - 高速的增量模型
    - 演化过程模型 Evolutionary Process Model - 迭代模型
        - *具有迭代性、能够轻松适应产品需求变化、通常不会产生一次性系统*
        - 原型模型 prototyping model
            - *用于客户需求不明确的情况*
        - 螺旋模型 spiral model
            - *强调风险分析*
        - 并发开发模型 concurrent development model
            - *并发工程的另一种称呼、定义触发工程活动状态转换的事件*
- 专用过程模型 Specialized Process Models
    - 基于构件的开发 (Component-Based Development)
        - *依赖于面向对象技术的支持*
    - 形式化方法模型 (The Formal Methods Model)
        - *通过数学方法，精确定义系统需求*
    - 面向方面软件开发 (Aspect-Oriented Software Development)
- 统一过程 The Unified Process - UP
    1. 初始阶段 Inception
    2. 精化阶段 Elaboration
    3. 构建阶段 Construction
    4. 移交阶段 Transition
    5. 生产阶段 Production

- 个体和团队过程模型 Personal and Team Process Models
    - 个体软件过程 PSP
    - 团队软件过程 TSP
