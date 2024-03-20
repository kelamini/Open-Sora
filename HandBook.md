# 记录

- 共包括三个阶段，分别是：

    1. 大规模图像预训练；
    2. 大规模视频预训练；
    3. 高质量视频数据微调。

每个阶段都会基于前一个阶段的权重继续训练。相比于从零开始单阶段训练，多阶段训练通过逐步扩展数据，更高效地达成高质量视频生成的目标。


训练流程示意图：

![训练流程](assets\images\handbook\tain_round.png)



在训练阶段首先采用预训练好的 Variational Autoencoder (VAE) 的编码器将视频数据进行压缩，然后在压缩之后的潜在空间中与文本嵌入 (text embedding) 一起训练 STDiT 扩散模型。在推理阶段，从 VAE 的潜在空间中随机采样出一个高斯噪声，与提示词嵌入 (prompt embedding) 一起输入到 STDiT 中，得到去噪之后的特征，最后输入到 VAE 的解码器，解码得到视频。


## 数据准备过程

1. 针对视频，以一个视频为例，随机抽帧 [random.randint(0, max(0, total_frames - self.size - 1)), max(0, total_frames - self.size - 1)]，输出形状：[C, T, H, W]
2. 针对图像，转化为视频，image.unsqueeze(0).repeat(self.num_frames, 1, 1, 1)


## 视频编码 VAE

采用 HuggingFace 中的预训练 VAE 模型：stabilityai/sd-vae-ft-ema，[B, C, T, H, W] -> [B, C, T, H/P, W/P]


## 文本编码 T5

采用 HuggingFace 中的预训练 T5 模型：DeepFloyd/t5-v1_1-xxl

## STDiT

保持其他默认参数不变的情况下，输入视频 [B, C, T, H, W]， 输出 B 不变，C 为原来的两倍， T H W 保持不变， 
