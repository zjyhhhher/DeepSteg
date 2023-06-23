# DeepSteg
Hey! This is a implementation of the paper ：**Hiding Images in Plain Sight: Deep Steganography, by Shumeet Baluja(Google), at NIPS 2017**

你可以使用如下命令加载保存好的模型直接进行测试：
```python
autoencoder_model.load_weights('autoencoder_model.hdf5')
encoder_model.load_weights('encoder_model.hdf5')
reveal_model.load_weights('reveal_model.hdf5')
```

**Abstract:** Steganography is the practice of concealing a secret message within another, ordinary, message. Commonly, steganography is used to unobtrusively hide a small message within the noisy regions of a larger image. In this study, we attempt to place a full size color image within another image of the same size. Deep neural networks are simultaneously trained to create the hiding and revealing processes and are designed to specifically work as a pair. The system is trained on images drawn randomly from the ImageNet database, and works well on natural images from a wide variety of sources. Beyond demonstrating the successful application of deep learning to hiding images, we carefully examine how the result is achieved and explore extensions. Unlike many popular steganographic methods that encode the secret message within the least significant bits of the carrier image, our approach compresses and distributes the secret image's representation across all of the available bits.

## Model
Perp Network主要负责对秘密图像进行预处理，并学习将基于颜色的像素转化为更有用的图像特征，以便对图像进行简洁编码，提高嵌入效率。Hiding Network接收预处理网络的输出和Cover image作为输入，主要负责将秘密图像嵌入到载体图像中实现隐写。Reveal Network是接收方使用的网络，也是解码器。它仅接收生成的载体图像作为输入，作用是从载体图像中移除原始Cover image，以揭示出隐藏的秘密图像。
![image](https://github.com/zjyhhhher/DeepSteg/assets/105298348/e62c9a50-4e82-4aca-9fa3-d108f690e46a)

作者提到对Hiding Network采用了多种网络结构进行尝试，包括不同数量的隐藏层和卷积核尺寸。其中，表现最佳的结构是由5个卷积层组成的，每个卷积层有各50个不同大小的卷积核（滤波器），卷积核的大小分别为3x3，4x4和5x5。所以在后续的代码实现中，我也采用了作者推荐的结构。

## Loss
在训练过程中，作者采用以下损失进行梯度下降优化。直观理解，该loss的目的是减小含秘图像与原始图像、提取后的秘密图像与原始秘密图像差距的范数。

![image](https://github.com/zjyhhhher/DeepSteg/assets/105298348/49688c86-8c86-4920-9c76-e5fc5fae15e6)

网络的后向传播方式如下图所示。首先，loss中的第一项衡量了原始图像与嵌入秘密信息后图像之间的差异。这个误差项的权重不会影响到接收载体图像并提取秘密图像的揭示网络（Reveal Network）的训练。第二个误差项是β||s – s’||，它衡量了秘密图像与提取出的秘密图像之间的差异。这个误差项的权重会影响到所有网络的训练，包括准备网络、隐藏网络和揭示网络。通过调整权衡因子β的值，可以控制这个误差项对各个网络的训练的影响程度。我觉得这样做的目的是让Prep network和Hiding network也能学习到秘密图像提取的信息，保证提取的效果，同时第一项不参加Reveal Network的训练可能是因为害怕将网络方向弄反。
![image](https://github.com/zjyhhhher/DeepSteg/assets/105298348/22fa4c1e-2059-48b7-a3f3-5ac4a6ee4c93)

 


