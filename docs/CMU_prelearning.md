学习路径：项目线上课程结合gpt、b站大学以及其他开源资源自学，如有侵权请联系删除🙏

![image-20240719194037589](./markdown-img/CMU_prelearning.assets/image-20240719194037589.png)

## Lec0

### 基本技术

- python
- pytorch
- GitHub
- LLM (ChatGPT等)

可以使用GPT自学

### 简要介绍

计算机视觉应用？ - 自动驾驶、核磁共振（疾病检测）、修图软件（更好地符合人类对美学的理解）、scan一个物品变成3D模型

基本任务：

1. **图像分类（Image Classification）**：
   图像分类的任务是将一张图像分配到一个或多个类别标签中。例如，系统可能需要判断一张图片是显示猫、狗还是汽车。这是计算机视觉中最基础也是最广泛应用的一种形式。<br>
   把每个像素分类 - 识别物品![image-20240719195800971](./markdown-img/CMU_prelearning.assets/image-20240719195800971.png)
2. **目标检测（Object Detection）**：
   目标检测不仅涉及识别图像中的物体类别，还需要定位这些物体的具体位置。这通常通过在图像中为每个检测到的对象绘制边界框（bounding boxes）来实现。
3. **图像分割（Image Segmentation）**：
   图像分割任务旨在将图像中的每个像素分类到特定的类别。这包括语义分割，将图像中所有像素分类到多个类别，以及实例分割，不仅区分类别，同时区分同一类别中的不同个体。

### 在线资源

1. **CS231n** & **EECS 498.008**
2. inverse problem of computer vision - computer graphics **CMU 15-462**
3. 数学基础![image-20240719204611568](./markdown-img/CMU_prelearning.assets/image-20240719204611568.png) 

## Lec1

### 1 - 数学基础

#### Algebraic structures

定义一个algebraic structure需要定义一个集合的元素和这个集合上面的运算，以及运算和元素的性质（结合律、自反律、交换律等）

闭包（封闭性） - 对集合内两个元素做运算，得到的元素依然属于这个集合

一种algebraic structure是向量空间/线性空间

![image-20240719205745642](./markdown-img/CMU_prelearning.assets/image-20240719205745642.png) 

应用：

IMU（Inertial Measurement Unit，惯性测量单元）是一种测量和报告物体的特定物理量的设备，主要用于测量角速度和线性加速度。IMU通常包含三个正交的加速度计和三个正交的陀螺仪，用来感知在三维空间中的加速度和旋转。

主要组件：

- 加速度计（Accelerometers）：用于测量物体在各个方向上的加速度，可用于感知重力导向和运动加速度。
- 陀螺仪（Gyroscopes）：用于测量物体围绕其三个轴的旋转速度。

#### Group - 群

![image-20240719211132759](./markdown-img/CMU_prelearning.assets/image-20240719211132759.png) 

### 2 - 优化问题 - 寻找最小值

损失函数衡量的是模型预测错误的程度，优化的目标就是最小化这个损失函数。在机器学习中，模型训练的过程实质上是通过优化算法（如梯度下降）最小化损失函数，从而找到最佳模型参数的过程。

有些问题我们需要使用循环、迭代的方式找到近似的最优解

感觉很像数值分析学的东西，我们需要评价特定方法（在不同参数下）的误差、cost（迭代次数）

例如梯度下降法![image-20240719212542272](./markdown-img/CMU_prelearning.assets/image-20240719212542272.png)，我们需要采取合适的步长（learning rate）

![image-20240719212656295](./markdown-img/CMU_prelearning.assets/image-20240719212656295.png)不同learning rate的效果 

基于二阶导的梯度下降法计算量过大，通常并不会表现出很好的性能

![image-20240719213546527](./markdown-img/CMU_prelearning.assets/image-20240719213546527.png) 

可微性是指函数在定义域内各点处均具有导数，这是执行梯度下降等基于梯度的优化算法的先决条件。梯度下降算法通过迭代过程寻找损失函数的最小值，逐步调整模型参数以最小化误差。

### 3 - Rigid Body Transformations

刚体变换主要由以下两个组成部分：

1. **旋转（Rotation）**：
   表示物体绕一个固定点（通常是中心点）的旋转。在三维空间中，旋转可以通过旋转矩阵或四元数来表达。旋转矩阵是一个3x3的正交矩阵且行列式为1，而四元数是提供了一种更为紧凑和鲁棒的方式来表示旋转。

2. **平移（Translation）**：
   描述物体沿直线从一个位置移动到另一个位置的过程。在数学表达上，平移可以通过向量来描述，这个向量指明了移动的方向和距离。

   使用矩阵实现无旋转的平移![image-20240719220938074](./markdown-img/CMU_prelearning.assets/image-20240719220938074.png)

比较难的就是旋转操作：

在二维平面上我们使用乘一个旋转矩阵的方式![image-20240719215148303](./markdown-img/CMU_prelearning.assets/image-20240719215148303.png)

三维平面我们根据不同的旋转轴有三个不同的旋转矩阵![image-20240719215330182](./markdown-img/CMU_prelearning.assets/image-20240719215330182.png)

而我们结合这三个方向的旋转（我的理解是把一个旋转正交分解）可以得到一个一般的旋转矩阵：

![image-20240719215415605](./markdown-img/CMU_prelearning.assets/image-20240719215415605.png) 

![image-20240719215550660](./markdown-img/CMU_prelearning.assets/image-20240719215550660.png) 

这个方法的局限性也就在于我们不同的旋转顺序会导致不一样的结果，单方向的旋转会影响其他方向

还会有“万向锁”问题。当俯仰角接近±90度时，偏航角和滚转角会合并在一起，造成自由度的丧失，这可以通过四元数或其他旋转表示方法来避免。

因此我们一般也不用欧拉角实现旋转相关代码。

所以我们考虑别的表达方式：![image-20240719220129322](./markdown-img/CMU_prelearning.assets/image-20240719220129322.png)

还有一种几乎相同的表达方式：![image-20240719220150591](./markdown-img/CMU_prelearning.assets/image-20240719220150591.png)

这就不会有万向锁的问题

如何通过这个表示方法计算？![image-20240719220322452](./markdown-img/CMU_prelearning.assets/image-20240719220322452.png)

$\theta$是旋转角，$K$是旋转轴

另外一种方式 - Quaternion ![image-20240719220621930](./markdown-img/CMU_prelearning.assets/image-20240719220621930.png)

为什么使用矩阵？现在的GPU基本被设计成适用于解决矩阵问题模式

二维平面结合旋转和平移![image-20240719221146703](./markdown-img/CMU_prelearning.assets/image-20240719221146703.png)

三维情况下我们拓展变换矩阵维度即可

可以来描述all kinds of motions

### 4 - Camera Model

what is a camera? - a projection from 3D to 2D

基本成像模型：Pinhole Camera

![image-20240719221953864](./markdown-img/CMU_prelearning.assets/image-20240719221953864.png) 

那么怎么找相机的pinhole就成为了一个问题

本质还是一个三维到二维的映射问题![image-20240719222055709](./markdown-img/CMU_prelearning.assets/image-20240719222055709.png)

使用相似三角形可以推![image-20240719222210078](./markdown-img/CMU_prelearning.assets/image-20240719222210078.png)

对于这个变换我们也可以写成**矩阵形式**：

![image-20240721195251639](./markdown-img/CMU_prelearning.assets/image-20240721195251639.png) 

*问题：不可逆，没有办法从2D照片推出具体距离（一般使用雷达解决）*

对于成像是倒着的问题，我们只需要把$x'$轴和$y'$轴翻转即可

光圈大小的影响？

大光圈 - 更模糊![image-20240719222543059](./markdown-img/CMU_prelearning.assets/image-20240719222543059.png)

但是光圈太小，进入的光线会太少；所以我们需要lens（凸透镜）汇聚光线，但是也就带来了变形问题：

![image-20240719222655973](./markdown-img/CMU_prelearning.assets/image-20240719222655973.png) ![image-20240719222722707](./markdown-img/CMU_prelearning.assets/image-20240719222722707.png)

解决方法：多个镜头（？，camera matrix好像也可以解决（这块我没有特别理解

3D to 2D 映射的特点：![image-20240721195457988](./markdown-img/CMU_prelearning.assets/image-20240721195457988.png)

## Lec2: Deep Learning Basics

```
Lecture Topic 1: From Neural Nets to CNN
Lecture Topic 2: Legendary Alexnet and ResNet
Lecture Topic 3: More Architectures: Unet, YOLO and more
```

首先，我们要理解一个神经单元的定义：

![image-20240721201304960](./markdown-img/CMU_prelearning.assets/image-20240721201304960.png) 

最后一步是因为二维空间的分类可能是非线性的![image-20240721201518399](./markdown-img/CMU_prelearning.assets/image-20240721201518399.png)

现在我们需要把这些东西写成矩阵形式而不是求和 - 更快的计算速度

这个模型的参数由上面提到的梯度下降法训练确定

### 1 - Image formation model

一张好的图片要让the color blind也可以轻松辨别，也就是每个颜色维度都需要有辨识度

![image-20240721203108025](./markdown-img/CMU_prelearning.assets/image-20240721203108025.png) 

#### 拜耳滤镜阵列（Bayer mosaic）

拜耳滤镜是一种广泛用于数码摄像机和其他成像设备中的彩色滤镜阵列。这种排列模式包含红色、绿色和蓝色滤镜，分布在图像传感器的像素上。该图解显示了每四个像素中，通常有两个绿色像素，一个红色和一个蓝色像素。这种设计模仿了人类眼睛中视网膜的锥细胞，人眼对绿色的灵敏度比红色或蓝色更高，因此增加绿色像素有助于提高图像的亮度感知和细节分辨率。

#### 2 - 卷积层

去噪 - 平均/中位去噪法（左右两边的点一起）

滤波（内积） - 见dip笔记

举个例子：![image-20240721205633688](./markdown-img/CMU_prelearning.assets/image-20240721205633688.png)

![image-20240721210021408](./markdown-img/CMU_prelearning.assets/image-20240721210021408.png)

效果：

- Horizonal Sober filter - 第二张
- Vertical Sobel filter - 第三张

![image-20240721210845389](./markdown-img/CMU_prelearning.assets/image-20240721210845389.png)

在pytorch内有现成的可以调用的函数

加速：running your image over your filter (convolution的交换律成立)

使用不同的kernel，我们可以得到不同层级的特征图，包含不同级别的特征

![image-20240721211846856](./markdown-img/CMU_prelearning.assets/image-20240721211846856.png) 

#### 3 - 卷积神经网络
##### 3.1 梯度消失的解决
避免gradient vanishing: ![image-20240721221842997](./markdown-img/CMU_prelearning.assets/image-20240721221842997.png)

（在underflow的时候误以为取到了opt）

##### 3.2 ResNet
![image-20240721222012078](./markdown-img/CMU_prelearning.assets/image-20240721222012078.png) 

通过这个方式解决了gradient vanishing，也就增加了支持的学习layer的个数

在传统的神经网络中，随着网络深度的增加，信息在通过网络的每一层时都会逐渐丧失，导致梯度消失，使得深层网络难以训练。ResNet 通过引入“残差学习”的概念来解决这个问题。如果我们将网络层设计为学习输入和输出的残差（即差异）而不是直接学习输出，网络可以更容易地优化，因为在理想情况下残差应是零，网络可以更容易地推动其输出接近输入。

#### 残差块（Residual Blocks）

ResNet 的基础构件是残差块，每个残差块包括两条路径：

1. **主路径**：主路径包括权重层（通常是卷积层），包括激活函数和批归一化等。
2. **快捷连接（Shortcut Connection）**：快捷连接允许输入直接“跳过”一些层，通过添加输入和主路径的输出来形成块的最终输出。

这种结构允许梯度直接通过快捷连接反向传播，避免了在深层网络中常见的梯度消失问题。

## Lec3

### Deep Learning Foundations

1. LeNet<br>
早期卷积神经网络，应用：数字识别<br>
主要包括：输入层、几个卷积层（使用卷积核提取图像特征）、池化层（下采样层，减少参数数量和计算复杂度），全连接层和输出层

2. AlexNet
使用ReLU作为激活函数，有效解决梯度消失问题，加速网络训练过程

3. VGG

### 3D Vision: Triangulation and Bundle Adjustment

### Know About Your Sensors

### SfM and SLAM

## Lec4

### Transformers: Backbone of Modern AI

### Generative Models: from VAE to Diffusion

### Physics Based Machine Vision

<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]
  }
};
</script>
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js">
</script>