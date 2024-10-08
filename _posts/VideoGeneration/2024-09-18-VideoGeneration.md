---
title: Stable Video Diffusion Scaling Latent Video Diffusion Models to Large Datasets
author: Chen Feng
date: 2024-09-18
category: [Paper, VideoGeneration]
layout: post
---

# Stable Video Diffusion: Scaling Latent Video Diffusion Models to Large Datasets

主要贡献：设计了一套数据清洗策略来清洗大规模的低质量的数据，用于训练 T2V 的 SOTA 模型，并证明了此模型具有足够强的关于**动作和 3D 的先验知识**可以用于视频相关的下游任务。

> 目前主要的 T2V 的模型都是基于 T2I 模型进行一些额外的修改（添加序列信息，添加 condition 等），因此大部分 T2V 模型的训练会经过以下的流程：image_pretrain ---> video_pretrain ---> video_finetune。模型会现在图片上进行训练，之后是大规模低质量数据，最后实在高质量的数据集上优化。

**Data curation workflow**

cut detection pipeline $\rightarrow$ 3 caption method $\rightarrow$ -> dense optical flow(remove static or text img) $\rightarrow$ alcu-
late aesthetics scores and text-image similarities.

**3 training stage**

![img](https://img2023.cnblogs.com/blog/3183309/202401/3183309-20240129172731315-1474069349.png)

这几个数据集主要揭示了使用经过处理之后的 vedio 数据集进行预训练，即使经过了后续的 finetune，最终的模型性能仍然更加优秀。上述图片是实验结果。

# VideoFactory: Swap Attention in Spatiotemporal Diffusions for Text-to-Video Generation

主要贡献：1. 提出了一个交换空间的交叉注意力方式。2. 收集了 130M 的高质量、无水印的数据集。

**swap Spatiotemporal Cross Attention**

![img](https://img2023.cnblogs.com/blog/3183309/202401/3183309-20240129172747883-2020249494.png)

主要是对于连续的 UNet Blk，分别使用空间特征和时序特征作为 Q 来进行交叉注意力。

**HD-VG-130M**

![img](https://img2023.cnblogs.com/blog/3183309/202401/3183309-20240129172802425-1717788009.png)

没有详细介绍数据集的收集和清洗过程，使用的是 PySceneDetect 作为分析工具。

# LaMD: Latent Motion Diffusion for Video Generation

主要思想：将视频的 motion 提取出来，并对 motionEbeding 做扩散。

![img](https://img2023.cnblogs.com/blog/3183309/202401/3183309-20240130164806147-432598849.png)

Stage1 包含两个 Encoder，2D-CNN Encoder 致于提取 content 特征，3D-CNN Encoder 致于提取 motion 特征，并且在 3D Encoder 使用了 VAE 常用的和正太函数比较 KL 散度，Decoder 会结合 content 特征和 motion 特征生成最终图像，并在最终添加了一个 Discriminator 进行监督，增强图像质量。

Stage2 是对 Stage1 得到的 motion 特征进行扩散（content 特征也会作为 condition），并且使用 CNN 作为模型主要结构，由于 motion 特征维度也很低。

# VideoCrafter2: Overcoming Data Limitations for High-Quality Video Diffusion Models (2024.1)

目前在视频生成方面主要具有两种训练方式，首先模型分为两种模块，Spatial、Temporal Modul。一种训练方式固定 Spatial（是 SD 的预训练权重），一种是在 Video 下 Spatial、Temporal 一起训练。

如果我们想要使用高质量的图片来微调在低质量视频下训练的模型，我们需要调研不同训练方式（上面两种）和不同微调方式（finetune、LORA）对于模型的影响。

本文做了实验解决了上面的问题，并在图像质量、视频动作方面进行评估。

- 只训练 Temporal 得到的模型在之后做 LORA 会严重降低 motion，这是因为 Full Train 会引入一种 Spatial-Temporal Consistency。
- 采用上面的方式，fine-tune 是更好的微调方式，并且只 finetune Spatial 会得到更好的 motion 和 Picture 的效果。

本文还使用了 SDXL、Midjourney 生成的图像作为微调的数据集（具有真实世界不存在的 concept）用于增强模型对于 concept 的理解。

# VideoCrafter1: Open Diffusion Models for High-Quality Video Generation (2023)

似乎是早期的 T2V、I2V 的模型的通用的实现方式

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201153110340-1143984416.png)

- 在 Video 数据上训练一个 VAE $\mathcal{E,D}$,然后在 Video Latent Space 上做扩散。模型结构如上。
- Spatial Transformer、Temporal Transformer 如下：
  $$\begin{aligned}\mathrm{ST=Proj_{in}\circ(Attn_{self}\circ Attn_{cross}\circ MLP)\circ Proj_{out},}\\\mathrm{TT=Proj_{in}\circ(Attn_{temp}\circ Attn_{temp}\circ MLP)\circ Proj_{out}.}\end{aligned}$$

- 对于 ImageEncoder，采用的是 CLIPViT 最后一层的特征，而不是最后模型对其的输出，认为这层特征包含更多细节信息。
- 当具有 Image 作为 prompt 时候，采用了额外结构来对其 Image 和 Text Prompt。如下：
  ![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201154515645-1036063551.png)

$$\mathbf{F}_{out}=\mathrm{Softmax}(\frac{\mathbf{Q}\mathbf{K}_{text}^\top}{\sqrt{d}})\mathbf{V}_{text}+\mathrm{Softmax}(\frac{\mathbf{Q}\mathbf{K}_{img}^\top}{\sqrt{d}})\mathbf{V}_{img}$$

# Latte: Latent Diffusion Transformer for Video Generation(2024.1)

上海 AILab 的论文，似乎结果并不怎么好，对比实验给的大部分都是 GAN 的。

论文对于模型架构、CLIP patch embedding、timestep-clas information injection，temporal positional embedding,learning strategies 等做了大量的实验，选取了最好的一个模型。

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201201000662-1507100383.png)

- 上述关于模型架构，Variant 4 效果最好。

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201201114550-1337072970.png)

- 关于视频编码，a 略微优于 b

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201201807619-1839636358.png)

- 一种新的注入 timestep 信息和类别信息$c(\alpha_c)$的方式。将 timestep 和类别共同编码为$\alpha_c, \beta_c, \gamma_c$,然后$\beta,\gamma$作为 AdaIN 的参数，而$\alpha$作为 Atten 的 Scale 参数。

- 让模型交替生成视频和图片能够极大提升模型最终的 FID 和 FVD。

消融实验：
![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240201202631262-1933367100.png)

# AnimateDiff: Animate Your Personalized Text-to-Image Diffusion Models without Specific Tuning

类似于 LoRA 的可插入式模型，固定 base 模型，并在其中添加 Motion Modeling Module，在这之上训练的 Motion Module 可以去迁移到其他任意类似模型上，作为类似 LoRA 的微调模型，生成视频的作用。

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240202162617937-2093255420.png)

motion Module 就是 Temporal Block，在视频上训练这些模块，固定 base 模型，这要求 base 模型的数据分布不能和视频的数据分布差别太大，否则无法训练良好的 Temporal Block，同时迁移到其他模型也不能和训练的 Video 数据分布差别太大。

![img](https://img2023.cnblogs.com/blog/3183309/202402/3183309-20240202162709112-943428997.png)

# StreamingT2V: Consistent, Dynamic, and ExtendableLong Video Generation from Text

[code](https://github.com/Picsart-AI-Research/StreamingT2V)

![img](https://img2023.cnblogs.com/blog/3183309/202403/3183309-20240328193459513-1646286873.png)

主要流程是

1. 使用一个 T2ShortV 生成一个简短的 video。
2. 使用简短的 video 作为 condition，利用了 Condition Attention Module(CAM),Appearance Preversation Module 来生成长视频。
3. 使用一种 Randomized Blending 生成更长的视频。

![VideoGeneration_2024-07-22_](https://s2.loli.net/2024/07/22/8DO6kBN1lmcpRTh.png)

- 希望 CAM 作为一个短时的模块引入 motion 的变化，使用 APM 来维持 ID 的稳定性。

- Randomized Blending.这个有点难理解，需要跟进代码。

# ShareGPT4Video: Improving Video Understandingand Generation with Better Captions

主要是做数据处理的工作。

- 差异化滑动窗口字幕策略（DiffSW）：采用一种新颖的字幕生成方法，将视频字幕生成任务转化为差异性描述任务。首先为第一帧生成详细字幕，然后使用长度为 2 的滑动窗口对后续帧进行差异性描述。最后使用 GPT-4 或者其他语言模型对生成的 caption 进行总结，输出最终的 video caption。
  ![VideoGeneration_2024-07-22_](https://s2.loli.net/2024/07/22/3Tx1WJq9Zau2FIX.png)
- 构建 ShareGPT4Video 数据集：利用 DiffSW 策略，从多种来源收集的视频中生成了 40K 高质量的视频字幕对，这些字幕涵盖了丰富的世界知识、对象属性、摄像机运动和详细的事件时间描述。
- 开发 ShareCaptioner-Video 模型：基于 ShareGPT4Video 数据集，进一步开发了一个高效的字幕生成模型 ShareCaptioner，能够为任意视频生成高质量的字幕。使用该模型为 480 万美学上有吸引力的视频生成了字幕。该模型能够支持 4 种方式的 caption。
  ![VideoGeneration_2024-07-22_](https://s2.loli.net/2024/07/22/IUjXefPlJmgMS5c.png)

- 最后在这个 480 万的数据下训练了一个基于 LLaVANext-8B 模型的 ShareGPT4Video-8B。

---

最近尝试使用这个模型来对视频进行 recaption，任务描述主要是当前的 caption：1. caption 太短，不详细。2. 并且基本上是人工的 caption 模式，和 Sora 模型、LLM 生成的 caption 模型具有较大差别。

最先尝试 ShareGPT4-Video，他的技术是：从视频中抽样 16 帧，并且组成 4x4 的图片，输入模型生成 caption。

- 生成的 caption 相比于之前的 caption 更长。相比于之前的(1).基于 masp 的 caption。(2).使用视频的简短人工的 caption 和视频抽帧使用 Image Caption 并且使用 LLM 总结得到最终的 caption。的两种方式的 caption 更长，但是并不具备更多的信息，多的部分主要在于描述背景。
- 生成的类似于 masp 的 caption，主要因为 masp 也是基于 GPT4-V 的数据进行训练的，因此也很类似 Sora 的 caption 模式。
- 对于长视频，非常容易生成多段文本。类似于下述这种
  > The video captures a series of intimate moments among a group of young women standing closely together, portraying a strong sense of camaraderie and affection. Initially, the women are seen leaning into each other with their eyes closed, hands placed gently on each other's arms or shoulders, against a plain, light-colored background. This image sets a tone of tranquility and connectedness among the group.

> As the video progresses, the framing and composition subtly shift to include more women in the foreground, while maintaining the serene atmosphere and the physical closeness of the group. One woman, notable for her bright yellow eyeshadow, becomes more prominent in the visual narrative, suggesting a slight change in the camera angle or the group's stance to bring her into clearer focus. Throughout, the women maintain their tranquil expressions and gentle touches, reinforcing the portrayal of a closely knit community.

> Further into the video, the focus broadens slightly to include additional women, enriching the portrayal of the group's dynamics. The central woman with yellow eyeshadow remains a focal point, surrounded by friends who now share the visual space more equally, emphasizing the collective identity of the group. The background remains consistent, providing a neutral canvas that keeps the emphasis firmly on the subjects.

> In the final moments captured, the composition stabilizes with minor adjustments in the positioning of the central woman and her immediate companions, maintaining the established focus and intimacy. The framing continues to highlight the central figures while subtly acknowledging the presence of others in the background, reinforcing the sense of closeness and unity among the group. Throughout, the video maintains a consistent, soft lighting that casts a warm glow over the scenes, accentuating the tranquil and connected
>
> 之后尝试了 ShareCaptioner，效果不太行，他的 fastCaption 也容易出现上述分成多段进行总结的情况。

# Visual Instruction Tuning(Llava)

![VideoGeneration_2024-07-22_](https://s2.loli.net/2024/07/22/Z6onJKFMTsGgIU8.png)

分成两个阶段训练模型，

1. 固定 LLM 和 Vision Encoder，训练一个 Projection W，这里是一个 Linear 层。
2. 固定 Vision Encoder，训练 Projection 和 LLM。
