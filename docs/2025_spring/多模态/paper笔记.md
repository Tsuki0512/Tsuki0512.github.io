## 0-1 什么是具备多模态理解能力的MLLM

### 0-1-1 Visual Instruction Turing

- [论文链接](https://arxiv.org/abs/2304.08485)

- 参考链接：[1](https://blog.csdn.net/qq_58400270/article/details/135073408) [2](https://zhuanlan.zhihu.com/p/647782091)

#### 1 前置知识

![image-20250303105247532](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303105247532.png)

指令微调和提示微调的目的都是去挖掘语言模型本身具备的知识。不同的是，Prompt 是激发语言模型的**补全能力**，例如根据上半句生成下半句，或是完形填空等。Instruct 是激发语言模型的**理解能力**，它通过给出更明显的指令，让模型去做出正确的行动。**指令微调的优点是它经过多任务的微调后，也能够在其他任务上做zero-shot，而提示微调都是针对一个任务的。泛化能力不如指示学习**。

#### 2 具体实现

##### 2.1 视觉指令数据集

用于指令微调的指示学习阶段。

![image-20250303111943784](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303111943784.png)

我们的输入并不是图片，而是描述图片的文本信息：captions和boxes。

训练过程中，模型通过图片描述和三类指令给出回答。

##### 2.2 LLaVa架构

![image-20250303111756590](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303111756590.png)

用一个简单的例子，比如你给系统输入一张狗的图片，并问它：“这是什么动物？”
LLaVA 处理这个问题的流程如下：

1. **图片输入 (`X_v`)**
      - 你上传一张狗的照片 (`X_v`)。
      - 视觉编码器（ViT-L/14）处理这张图片，提取重要特征，得到 `Z_v`（相当于“图片的数值版”）。
2. **特征转换 (`Z_v` → `H_v`)**
      - 由于 LLM 只能理解文本，所以需要把 `Z_v` 变成类似单词的格式。
      - 通过**投影矩阵 (`W`)**，把 `Z_v` 转换成**视觉标记 (`H_v`)**，让 LLM 能够理解。
3. **语言模型处理 (`H_v` + `H_q`)**
      - 你的问题 `"这是什么动物？"` (`X_q`) 也被语言模型处理。
      - 语言模型结合问题 (`H_q`) 和视觉信息 (`H_v`)，然后生成回答 (`X_a`)。
4. **输出 (`X_a`)**
      - 模型输出 `"这是一只狗"`，完成任务。

##### 2.3 训练过程

![image-20250303135208878](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303135208878.png)

整体训练的输入输出模式为：

![image-20250303135334852](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303135334852.png)

- 绿色的部分用于计算loss。*q: 为什么所有<STOP>token都要被计算？*

- 指令微调过程的输出计算采用自回归函数，也就是给定图像特征和指令后输出为a的条件概率为所有token在图像特征和之前所有token的条件下输出该token的条件概率相乘；对于长度为$L$的对话数据序列，计算可能的回答概率如下：
   ![image-20250303135721366](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303135721366.png)

- $X_{\text{system-message}}$为系统要执行的任务：

  ```c++
  A chat between a curious human and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the human’s questions. and <STOP> = ###
  ```

###### 2.3.1 第一阶段：特征对齐预训练

在2.2的架构中，我们看到语言模型会结合`H_v`和`H_q`给出回答，所以我们在这一步需要做的就是对齐两者的特征。

利用 [图像+人工标记的caption] 数据集，冻结视觉编码器和LLM的参数，仅训练线性层$\text{Projection}W$的参数，在构建输入$X_{instruction}$时，对于每一张图像$X_v$，随机抽取一个问题$X_q$ ，这个问题其实就是让模型（可以理解为助手）简单描述图像的语言指令。真实的预测答案$X_a$就是原来图像所配的文字描述（也就是原始的图像说明文字）。

而我们训练的过程就是调整线性层参数使得$X_a$的$p$最大化。

###### 2.3.2 第二阶段：端到端的微调

始终保持视觉编码器的权重冻结（即不更新视觉编码器的权重），继续更新投影层的预训练权重以及 LLaVA 中的大语言模型（LLM）的权重。

在这里我们考虑两个特定的用例场景：

1. chatbot: 在聊天机器人输出的三种类型回复中，对话类型是多轮的，而另外两种回复类型是单轮的。在训练过程中，对这三种回复类型进行均匀采样。
2. Science QA: 将数据组织成单轮对话形式，模型需要完成两项任务，一是以自然语言的形式提供推理过程，二是在多个选项中选择答案，训练中我们把问题和上下文作为$X_{instruction}$，把推理过程和答案作为$X_a$。

### 0-1-2 MiniGPT-4: Enhancing Vision-Language Understanding with Advanced Large Language Models

- [论文链接](https://arxiv.org/abs/2304.10592)

- 参考链接：[1](https://jackcrum.blog.csdn.net/article/details/131258219?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-131258219-blog-130386728.235%5Ev43%5Epc_blog_bottom_relevance_base8&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-131258219-blog-130386728.235%5Ev43%5Epc_blog_bottom_relevance_base8&utm_relevant_index=5) [2](https://blog.csdn.net/m0_52911108/article/details/142914183)

#### 1 模型结构

MiniGPT-4处理视觉和语言任务的流程：

![image-20250303152941564](./paper%E7%AC%94%E8%AE%B0.assets/image-20250303152941564.png)

- 视觉部分
     1. **输入图像**：右下角的火烈鸟logo是输入的视觉信息。
     2. **视觉编码器**：
        - ViT（Vision Transformer）是一种用于计算机视觉任务的 Transformer 架构，它会对输入的火烈鸟 logo 图像进行特征提取，将图像信息转化为特征向量。
        - Q - Former 则可能是用于处理这些特征向量，以便后续与语言模型进行交互。
- **线性层（投影层）**</br>
   中间的橙色方框 “Linear Layer” 是一个线性层，也可理解为投影层。它的作用是将视觉编码器提取的特征向量投影到与语言模型相匹配的特征空间中，使得视觉特征能够被语言模型理解和处理，实现视觉信息与语言信息在特征层面的**融合**。</br>
   *q: LLaVA是同时训练调整视觉特征和指令特征在线性层的投影参数，mini-GPT4是指训练调整视觉特征在线性层的投影参数使其和输入的语言问题对齐？*
- 语言模型部分
     1. **Vicuna**：是一个经过训练的大语言模型，在 MiniGPT - 4 中它被冻结，参数在训练过程中不更新。
     2. **输入与输出**：图中左侧 “Human:” 代表人类输入的问题，例如 “ What do you think of this logo design? ”。经过线性层处理后的视觉特征与语言问题一起输入到 Vicuna 模型中，然后 Vicuna 模型生成回答（图中右侧 “Assistant:” ），图中给出的回答示例是一段对火烈鸟 logo 设计的评价，描述了 logo 设计简约、易识别，使用火烈鸟作为标志增添了趣味性等特点。

mini-GPT4训练的是线性层的参数，使得视觉特征和语言信息在特征层面融合。

基于该模型的主要发现：

- 该模型具有高级视觉语言能力
- 仅训练线性层即可对齐视觉编码器和LLM的特征，且训练开销不大 *q: 这里的特征对齐具体是什么含义*
- 采用image caption对模型训练效果并不好，提供小而细节的图像描述可以更好地解决这个问题

#### 2 训练过程

##### 2.1 第一阶段预训练

通过大量对齐的图片-文本对去训练模型整体的视觉-语言能力，将线性层输出作为LLM的软提示输入，目的是让LLM输出正确的回答。这个过程只训练线性层参数。

训练过程中的问题：会生成不连贯的语言输出，例如单词或句子重复、句子碎片化，或者内容不相关。为了解决这个问题，他们为视觉语言领域策划了一个高质量的对齐数据集。

数据集制作过程：

1. **初始对齐的图像-文本生成**
   使用从第一阶段预训练得到的模型来生成输入图像的全面描述，LLM输入为：
   ![image-20250304162914505](./paper%E7%AC%94%E8%AE%B0.assets/image-20250304162914505.png)
   `<ImageFeature>`是线性层的输出（视觉特征）。
   之后检查生成的句子是否超过 80 个词元，如果没有超过，添加一个额外的提示，`###Human: Continue ###Assistant:`，促使模型继续生成内容。通过将两个步骤的输出连接起来创建出更全面的图像描述。
2. **数据后处理**
   上述自动生成的图像描述包含有噪声或不连贯的内容，例如单词或句子的重复、句子支离破碎，或者出现不相关的信息。为了解决这些问题，我们使用 ChatGPT，通过以下提示来修正这些描述：
   ![image-20250304163341261](./paper%E7%AC%94%E8%AE%B0.assets/image-20250304163341261.png)
3. 人工验证每一条图像描述的正确性，以确保其高质量：
      - 找出了一些频繁出现的错误，编写硬编码规则来自动过滤掉这些错误；
      - 人工优化生成的图像说明文字，删除那些 ChatGPT 未能检测到的冗余单词或句子。

我们将处理完毕的数据集用于第二阶段的微调训练。

##### 2.2 第二阶段微调

通过第一阶段制作的数据集，从指令集中随机抽取指令作为`<Instruction>`，在LLM中输入：

![image-20250304163648618](./paper%E7%AC%94%E8%AE%B0.assets/image-20250304163648618.png)

进行指令微调训练，对于这个特定的文本 - 图像提示，我们并不计算回归损失。这使得模型能够生成更自然可靠的语言输出，且该微调过程效率较高。

*q: 为什么我们在这个过程中不计算回归损失，那我如何达到这个阶段的训练目的*

*q: 第一阶段得到的是文本-图像数据集，但是模型输入不是图像+语言吗，这个文本在这里起到什么作用*

*a: 作为训练的输入输出对的输出内容，因为指令集里面的指令都是换着不同方式表达同一个意思（描述图片details）*

## 0-2 常见的图像生成范式

### 0-2-1 High-Resolution Image Synthesis with Latent Diffusion Models

---

#### 前置：Diffusion Model

学习链接：[1](https://www.bilibili.com/video/BV1xih7ecEMb/?buvid=Y24FF22823969EE34A5598A5D4F4695016E2&from_spmid=search.search-result.0.0&is_story_h5=false&mid=dhneVveoAEYYORADGVP2pA%3D%3D&plat_id=116&share_from=ugc&share_medium=iphone&share_plat=ios&share_session_id=EF9298D6-031B-4816-AF2D-B42CAEAF07E7&share_source=WEIXIN&share_tag=s_i&spmid=united.player-video-detail.0.0&timestamp=1741161005&unique_k=LyXYQ8H&up_id=527467660&vd_source=fef878b8598786a5cbf02ab622b4d684) [2](https://huggingface.co/learn/diffusion-course/unit1/1)

**1. 什么是DM？**

给定前向过程，每一次往前一张图片加噪点：

![image-20250305165621261](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305165621261.png)

加噪的具体过程是每次加一个0-1高斯分布的噪点：

![image-20250305170026101](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305170026101.png)



而我们可以考虑其反向过程：

![image-20250305165739898](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305165739898.png)

就是逐步去噪生成目标图片的过程，我们训练出一个可以预测噪声的CNN，把原始的照片减去预测出来的噪声得到$x_{t-1}$：

![image-20250305170309616](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305170309616.png)

每一步使用的模型是同一个模型，但是我们会输入$t$作为当前步骤/时刻的信息。

**2. 具体过程**

*一堆数学推导，两年前学的概率论忘完了，先放着吧*

前向加噪，通过公式推导我们可以得出这个过程中每一步都是符合高斯分布的：

![image-20250305172334527](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305172334527.png)

我们也可以通过公式推导把逐步加噪变成单步加噪：

![image-20250305172655087](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305172655087.png)

但是反向过程是不可能的，我们需要神经网络拟合，通过一个高斯分布去拟合另一个高斯分布：

![image-20250305210215469](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305210215469.png)

拟合的方式是最大似然估计，这里的数学推导真的是天书，没学会，不记了（和整个模型的使用关系不大）

后面也是天书。

**3. 训练过程**

加噪+预测噪声，采用$L2$评估损失

![image-20250305211616989](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305211616989.png)

**4. 采样过程**

逐步去噪。

![image-20250305211731727](./paper%E7%AC%94%E8%AE%B0.assets/image-20250305211731727.png)

最后一步和前面的区别是不会加扰动项了。

---

- [论文链接](https://arxiv.org/abs/2112.10752)
- 参考链接：[1](https://blog.csdn.net/weixin_57974242/article/details/134180461)



### 0-2-2 Taming Transformers for High-Resolution Image Synthesis

- [论文链接](https://arxiv.org/abs/2012.09841)
- 参考链接：[1](https://blog.csdn.net/weixin_43357695/article/details/135995463)

//todo
