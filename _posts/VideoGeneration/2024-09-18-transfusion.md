---
title: Transfusion
author: Chen Feng
date: 2024-09-18
category: ["Video Generation"]
layout: post
mermaid: true
---

# Transfusion: Predict the Next Token and Diffuse Images with One Multi-Modal Model(2024,8)

[Paper](https://www.arxiv.org/abs/2408.11039)
TODO: 目前没有开源代码,实时关注一下 official code,Meta 的工作基本开源的.本文给出了一种**新的 T2I 的方法**.
[lucidrains 的代码](https://github.com/lucidrains/transfusion-pytorch)

![transfusion_2024-09-04_](https://img-blog.csdnimg.cn/img_convert/06202828102e960b817749002511f009.png)

本质是将 LLM 的 transformer 和图像中的 diffusion 结合了起来,使用同一个 transformer 来同时处理文本和图像信息.之前的 DiT 架构都是使用一个预训练的 TextEncoder 来提取文本信息,,并通过 Concat、AdaLN、
CrossAttention、MMDit 等方式将文本信息融入模型,而本文的方式直接同时训练文本和图像信息,并且是使用同一个模型来进行处理.

![transfusion_2024-09-04_](https://img-blog.csdnimg.cn/img_convert/98d230d4e3d67b104ebc0e24724b263d.png)

如上图,图像经过一个 VAE 来得到 tokens,并插入到文本 token 中,文本也会在经过一个 tokenizer 之后通过一个轻量级的模块进行处理,然后再通过一个 transformer 来处理文本和图像的信息.

![transfusion_2024-09-04_](https://img-blog.csdnimg.cn/img_convert/f3c8f445c0d5796466a27f147b67671e.png)

文本的 attention 方式和图像不一致,文本因为要采用 causal 的方式,而图像则需要采用 bidirectional 的方式.

Loss : $\mathcal{L}_\text{Transfusion}=\mathcal{L}_\text{LM}+\lambda\cdot\mathcal{L}_\text{DDPM}$

对于文本采用了 LLM modeing,对于图像采用了 DDPM 的 loss,训练细节和效果可以参看论文.

论文通过提出 Transfusion 方法来解决多模态模型的训练问题。Transfusion 的核心思想是训练一个单一的模型来同时处理离散和连续的数据模态，具体解决方案包括以下几个关键步骤：

- 数据表示：将文本数据表示为离散的 token 序列，将图像数据通过变分自编码器（VAE）编码为连续的潜在空间补丁序列。在混合模态的示例中，使用特殊的开始图像（BOI）和结束图像（EOI）标记来分隔文本和图像序列。

- 模型架构：使用一个单一的 Transformer 模型来处理所有序列，无论其模态如何。对于文本，使用嵌入层将 token 转换为向量；对于图像，尝试了两种将局部窗口的补丁向量压缩成单个 Transformer 向量的方法：简单的线性层和 U-Net 的上下块。

- 注意力机制：结合了因果注意力（用于文本 token）和双向注意力（用于图像补丁）。这允许图像内的每个补丁能够相互注意，同时只能注意序列中先前出现的文本或图像补丁。

- 训练目标：对文本 token 应用语言建模目标（LLM），对图像补丁应用扩散目标（LDDPM）。通过简单地将两种模态上计算的损失相加，并引入一个平衡系数 λ，来训练模型。

- 推理算法：根据训练目标，解码算法在两种模式（LM 和扩散）之间切换。在 LM 模式下，从预测分布中逐个采样 token。当采样到 BOI 标记时，切换到扩散模式，按照扩散模型的标准程序解码图像。

- 实验验证：通过一系列受控实验，论文展示了 Transfusion 在不同模型大小和数据量下的性能，以及与 Chameleon 方法的比较。实验结果表明，Transfusion 在每种模态组合中的扩展性都优于 Chameleon 方法。

- 架构改进：论文还探讨了 Transfusion 模型的不同变体，包括使用不同大小的图像补丁、不同的编码/解码架构（线性层与 U-Net 块），以及限制图像噪声的程度，以提高特定任务的性能。

- 图像编辑能力：论文进一步展示了 Transfusion 模型在图像编辑任务上的潜力，通过在少量图像编辑数据上微调预训练模型，使其能够根据指令执行图像编辑。
