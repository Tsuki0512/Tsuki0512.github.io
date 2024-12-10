# Attention is all you need - paper笔记

## 0 参考链接

[论文链接](https://arxiv.org/pdf/1706.03762)

[参考解读](https://blog.csdn.net/weixin_43334693/article/details/130208816)

## 1 整体架构及拆解 - transformer是什么？

![image-20240814080755005](./markdown-img/transformers.assets/image-20240814080755005.png)

上图就是transformer的整体架构，左边的部分是encoder，右边的部分是decoder。

那么抛开技术细节，最直接的问题其实是什么encoder到底在给什么东西编码，编成什么码；decoder在解什么码，解成什么码。整个transformer将什么转换成了什么。

### 1.1 encoder-decoder

encoder部分概括来说就是将输入序列压缩（编码或特征提取）成指定长度的向量的过程，将现实问题转化为了数学问题。

![image-20240816030258985](./markdown-img/transformers.assets/image-20240816030258985.png)

而decoder则可以理解成encoder的逆运算，将输入的向量转换为目标类型输出（取决于任务是什么，如果是翻译就是输出译文）的过程，将数学问题转化为了现实问题。

![image-20240816034610533](./markdown-img/transformers.assets/image-20240816034610533.png)

而encoder-decoder就可以理解为一个通用的框架，将一个现实问题输入通过这样一个框架输出所指定的另一个现实问题，例如翻译就是读取一段中文文本输入后输出一段英文文本，而看图说话就是读取一张图片输出一串单词。这就是transformer做的事情。

### 1.2 encoder

根据Figure1的encoder，我们可以将其分为两部分：

1. 自注意力机制 + 残差链接 + AddNorm
2. FeedForward + 残差链接 + AddNorm

