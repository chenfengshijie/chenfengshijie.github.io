---
title: CogVideoX Text-to-Video Diffusion Models with An Expert Transformer
author: Chen Feng
date: 2024-09-18
category: [Paper, VideoGeneration]
layout: post
---

# CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer[2024,8,14]

[Diffusers](https://github.com/huggingface/diffusers/blob/0c1e63bd11a5746db8933111a962854fa9b36582/src/diffusers/pipelines/cogvideo/pipeline_cogvideox.py)
[Github](https://github.com/THUDM/CogVideo)
[CogVideoX-2b](https://huggingface.co/THUDM/CogVideoX-2b) : 只开源了 2B 的模型,5B 模型目前没有开源.

清华智普最新的 T2V 模型,目前测试下来在第二梯队左右,相比于快手的可灵,Luma 等还要差一点(至于现在没啥消息的 Sora 是另一回事了).可以生成 460x720 的 6s 视频,但是 fps 只有 8.

## Method

### DiT Architecture

![CogVideoX_2024-08-15_](https://s2.loli.net/2024/08/15/lC6HVvcRysBEmQM.png)

prompt 经过 T5 的到的 Embedding 和 Video 经过 VAE 得到的 Embedding 在 Sequence 层面 Cat 在一起,由于没有经过 Norm 的操作,这两个 embedding 完全不一致,利用两个完全独立的 AdaLyaer 来映射为不同的 Scale 参数.

Block 的设计参看[代码](https://github.com/huggingface/diffusers/blob/0c1e63bd11a5746db8933111a962854fa9b36582/src/diffusers/models/transformers/cogvideox_transformer_3d.py),是最直接的方法.

### 3D causal VAE

![CogVideoX_2024-08-15_](https://s2.loli.net/2024/08/15/bohPeTjKBylzca3.png)

在训练 VAE 采用了 3D causal Conv 方式(当前帧只和后面帧进行连接,在 h,w 上不做限制,类似于 NLP 中的 causal 方式),能够压缩 8x8x4 倍.

### PE

采用了 3D RoPE 的方式,在(x,y,t)三个维度上都做了 RoPE,并且三个 PE 在 Channel 维度上分别占用$3/8,3/8,2/8$

### other tricks

- 训练 3D VAE 的时候加上了 L2 Loss ,LPIPS perceptual loss, GAN loss(from 3D discriminator).
- 图像和视频混合训练,并且不固定视频和图像出现的位置,论文表示之前图像的视频混合训练都是前固定帧数为图像,后面固定帧数为视频,这样等同于训练了两个独立的模型,对于模型的理解能力不能得到有效提高.

![CogVideoX_2024-08-15_](https://s2.loli.net/2024/08/15/jEetqPRwGVv8Hzb.png)

- t 采样方式,之前关于时间步 t 的采样都是均匀随机从[1,T]中采样,但是这种方式在模型每一步更新中不能保证是均匀采样的,论文认为这种方式会对模型训练具有一定影响.因此论文为每一张卡设定了一个采样区间$[t_i,t_{i+1}]$,这样能够保证每次模型更新都是均匀采样得到的.Loss 更加稳定,结果如下(d).

![CogVideoX_2024-08-15_](https://s2.loli.net/2024/08/15/bCfL5wDF2jkOMeA.png)

- 模型训练方式也是采用在 LQ 视频下预训练,在 HQ 视频下微调的方式,最后在高质量的文本-视频对下进行微调.
