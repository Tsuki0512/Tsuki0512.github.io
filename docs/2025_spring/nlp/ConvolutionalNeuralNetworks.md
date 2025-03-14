# Chap3 Convolutional Neutral Networks

卷积神经网络通常用于处理视觉对象。

## 1 基本原理

每一个神经元其实就是在做特征检测，第一层会做非常简单的对象检测，第二层的输入是第一层的结果，检测简单的形状，而后面的层可能就会检测形状的组合。

![image-20250314134024002](./ConvolutionalNeuralNetworks.assets/image-20250314134024002.png)

### 1.1 图像特征识别的特点

1. 在我们做检测的时候，同一个特征会出现在不同图片的不同区域。每个神经元只能检测指定位置的特征，因此不同位置的特征需要不同的神经元，但是由于检测都是同一个特征，所以神经元都是差不多的：
    ![image-20250314140916477](./ConvolutionalNeuralNetworks.assets/image-20250314140916477.png)
2. 下采样：通过例如移除一张图片所有奇数列和奇数行的方式缩放图片，这个缩小不影响神经元的特征检测。</br>
基于上述特性，完整的CNN如下图所示：
    ![image-20250314140932671](./ConvolutionalNeuralNetworks.assets/image-20250314140932671.png)

### 1.2 卷积

过程就是逐位相乘再相加，所得结果为中间格的值。不同卷积核提取不同特征，比如这个检测到了从左上到右下的斜线：

![image-20250314140824745](./ConvolutionalNeuralNetworks.assets/image-20250314140824745.png)

- 因为我们关注的特征一般是小于这个图像的，所以我们用一个比较小的卷积核去做卷积（特征提取），得到的结果是关于这个特征的热力图，第二层就是在这个热力图的基础上卷积；

- 而由于特征可能出现在不同的位置，所以我们用卷积核遍历整张图片；

- 因为边缘可以补0，所以卷积之后图片大小不变：zero padding:
    ![image-20250314135959494](./ConvolutionalNeuralNetworks.assets/image-20250314135959494.png)
在神经网络中的实现就是在一个神经元内，只有被对应卷积核覆盖的像素点才有值，而不同位置只是置1的输入变量不一样，权重参数是一样的(Shared Weights)：
    ![image-20250314141001586](./ConvolutionalNeuralNetworks.assets/image-20250314141001586.png)

### 1.3 池化

对应特性：下采样不影响特征识别

Max Pooling：对feature map分块，保留块内最大值，检测区域内最强特征。而池化之后的图片缩小，也减少了后续的计算量。

![image-20250314140724388](./ConvolutionalNeuralNetworks.assets/image-20250314140724388.png)

平均池化：检测一个模式/特征有多频繁，例如alpha go。

如何设计卷积核的数量？- 前少后多，因为模式的组合的可能性比简单的特征、形状更多，需要更多的神经元检测器。池化也是减少卷积核数量增加的一种方式。

### 1.4 Flatten

当池化后的特征图足够小以致于无法继续池化的时候，我们将其拉直输入MLP进行分类：

![image-20250314141650068](./ConvolutionalNeuralNetworks.assets/image-20250314141650068.png)

### 1.5 例子：CNN处理围棋问题

为什么可以？- 围棋棋面符合CNN的三个特征

- 特征小于整张图
- 特征可能出现在整张图的不同位置
- *下采样不影响特征识别* - 存疑

分类问题是next move(19*19个位置)

## 2 CNN的历史

人类的一个视觉神经会处理多个视觉感知器的输入，因此除了感知光线之外也能直接感受形状。

人类的视觉神经元以以下方式组织：$\text{simple cells} \to \text{complex cells} \to \text{hyper-complex cells}$

且具有平移不变性。

![image-20250314143344717](./ConvolutionalNeuralNetworks.assets/image-20250314143344717.png)

而之后人们将脑神经科学的结构做了计算机模拟，就有了最早的CNN。

平移不变 $\to$ 权值共享 $\to$ 数据增强（对输入数据做变化，比如翻转旋转输入、将输入trunking，实现旋转缩放不变性）、最大池化 $\to$ LeNet（使用tanh作为激活函数）$\to$ **AlexNet（dropout：每次训练的时候随机丢弃神经元，最后推理阶段激活所有神经元，防止过拟合）**

## 3 CNN的应用与变形

### 3.1 CNN的变形

- VGGNet - 最后的任务重用图像分类，完成图像分割、跟踪等任务。实验结果表明效果最好的层数为16、19层。
    - 为什么不是越深越好？- 有梯度消失问题
- GoogLeNet - 把网络变宽。同一层内使用不同大小卷积核。
    ![image-20250314144255710](./ConvolutionalNeuralNetworks.assets/image-20250314144255710.png)
    - 为什么之前不用`5*5`而用`3*3`？- 减少计算量，两个`3*3`叠加就是`5*5`
    - 但是之前的架构可能会在不同层次检测同一个特征，造成冗余，所以我们在同一层使用不同大小的卷积核检测同一特征
    - `1*1`卷积 - 做整体缩放，如下图把32张feature map做线性组合将其变成1张，减少计算量；例如transformer也采取了这个思想
        ![image-20250314144653902](./ConvolutionalNeuralNetworks.assets/image-20250314144653902.png)
    - 传统多任务学习：通过同时训练多个相关任务，模型能够利用不同任务之间的相关性。例如，在图像分类和物体检测的任务中，图像的特征可以被同时用于这两个任务，从而提高整体性能。多任务学习可以起到正则化的作用，减少过拟合的风险。当模型在多个任务上进行训练时，它需要学习更通用的特征，从而避免在单一任务上过拟合。
        - 辅助任务学习：在网络中间层加入输入信号（多任务一般是在最后一层），在中间层就开始分类，强化中间层能力。
- ResNet - 解决了梯度消失的问题
    ![image-20250314145436322](./ConvolutionalNeuralNetworks.assets/image-20250314145436322.png)
    本来学的是x怎么变成y，现在学的是x和y的差异，x加什么变成y，增加了一个跳变，解决梯度下降之后大大增加了网络的深度。
- DenseNet - 在连接之间插入dense block，跳变可能不止一层：
    ![image-20250314145859499](./ConvolutionalNeuralNetworks.assets/image-20250314145859499.png)

### 3.2 卷积核的变种

- 1D卷积核：滑动窗口
    ![image-20250314145950430](./ConvolutionalNeuralNetworks.assets/image-20250314145950430.png)
- 2D卷积核（如上述介绍）
    - 2D卷积处理3D输入
        ![image-20250314150022825](./ConvolutionalNeuralNetworks.assets/image-20250314150022825.png)
        利用之前的`1*1`卷积核
- 3D卷积核（例如处理视频，第三个维度是时间）
    ![image-20250314150111919](./ConvolutionalNeuralNetworks.assets/image-20250314150111919.png)
- 图卷积 - 图由节点（vertices）和边（edges）组成，节点可以表示实体，边表示实体之间的关系；图卷积旨在利用这些结构信息来学习节点或图的特征
    ![image-20250314152412280](./ConvolutionalNeuralNetworks.assets/image-20250314152412280.png)
- 球卷积 - 把3D物品投影到球然后做卷积
    ![image-20250314152247573](./ConvolutionalNeuralNetworks.assets/image-20250314152247573.png)
- 转置卷积 - 一般是用来做生成的，本来是从大卷到小，用于生成需要从小卷到大，例如文生图任务，实现方式是小图和大卷积核，常见应用风格迁移
    ![image-20250314150344508](./ConvolutionalNeuralNetworks.assets/image-20250314150344508.png)

### 3.3 Capsule Network

卷积神经网络的问题：

- 池化丢失了**位置信息**，例如旋转人脸导致无法识别，但是交换眼睛和嘴巴的位置却导致依旧识别为人：
    ![image-20250314150630815](./ConvolutionalNeuralNetworks.assets/image-20250314150630815.png)
针对上述问题，有一个尝试是capsule network：
![image-20250314150917527](./ConvolutionalNeuralNetworks.assets/image-20250314150917527.png)

传统的神经元基本处理的是标量，标量没有方向，而capsule做的就是将输入变成向量（数组实现），输出也是数组。而我们在神经元会对其进行仿射变换，加权求和的过程也是根据注意力机制动态变化的：

![image-20250314152221741](./ConvolutionalNeuralNetworks.assets/image-20250314152221741.png)

检测形状和形状的朝向，初始层是三角形的capsule和长方形的capsule，然后对这两个capsule做动态路由，在下一层匹配成帆船/房子的capsule。

没有成功可能是硬件不匹配。