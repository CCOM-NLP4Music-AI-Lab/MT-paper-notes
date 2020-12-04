
# Music SketchNet

[paper link](https://arxiv.org/pdf/2008.01291v1.pdf)

[code](https://github.com/RetroCirce/Music-SketchNet)

Tags: Controllable generation, symbolic, pitch and rhythm, BERT-inspired, monophonic music impainting

## Background

ISMIR2020，一作是UCSD的Ke Chen.

## Problem Define

给定前后的**旋律**片段，生成中间缺失的小节，并且可以由用户指定音高轮廓和节奏型

![image-20201204123400429](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/sketchnet-probdef.png)

## 

## Related Work

- [**Learning Interpretable Representation for Controllable Polyphonic Music Generation**](https://arxiv.org/pdf/2008.07122.pdf)
    - ISMIR 2020，VAE解耦chord 和texture, 出自music X lab
- Measure-VAE: 
    - ISMIR 2019
- [**LEARNING TO TRAVERSE LATENT SPACES FOR MUSICAL SCORE INPAINTING**](https://arxiv.org/pdf/1907.01164)
- EC2-VAE:
    - [**DEEP MUSIC ANALOGY VIA LATENT REPRESENTATION DISENTANGLEMENT**](https://arxiv.org/pdf/1906.03626.pdf)
    - ISMIR2019, 出自music X lab
    - [一个笔记](https://ldzhangyx.github.io/2019/07/29/ec2vae/)

## Dataset

Irish and Scottish monophonic music数据集：4/4拍，16000训练， 2000测试

## Methods
![image-20201204131948525](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/sketchnet-model.png)

![image-20201204215114461](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/sketchnet-formular.png)

- ### SketchVAE

    - SketchNet模型结构中SketchVAE的encoder和decoder部分并不是连在一起的，而是中间插入了Inpainter和Connector
    - Encoder
        - 把旋律抽出音高轮廓序列和rhythm 序列，音高做embedding后，丢进两层双向GRU，得到隐变量![formula](https://render.githubusercontent.com/render/math?math=Z_%7Bpitch%7D%2C%20Z_%7Brhythm%7D)， 其中![formula](https://render.githubusercontent.com/render/math?math=Z%5Ep%2CZ%5Ef)丢给inpainter，![formula](https://render.githubusercontent.com/render/math?math=Z%5Em)丢给connector用于随机unmask
    - Hierarchical Decoder
        - upper beat GRU
        - lower tick GRU
        - 类似于MeasureVAE的decoder架构, （具体的结构原理我还没来及看measureVAE的paper

- ### SketchInpainter

    组合了一堆GRU , 用来根据![formula](https://render.githubusercontent.com/render/math?math=Z%5Ep%2CZ%5Ef)预测![formula](https://render.githubusercontent.com/render/math?math=Z%5Em)

    ![image-20201204215407658](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/sketchnet-inpainter.png)

- ### Sketch Connector

    - 用来将inpainter预测的missing part的隐变量与用户输入结合，从而达到conditioned生成的目的
    - 这里训练和预测采用不同的策略：
        - 训练时采用类似于BERT的随机unmask方法，输入ground truth经过encode得到的隐变量 （下图所展示的过程）
        - 预测时输入User Input经过encode得到的隐变量
    - 结构上也是采取类似于transformer的self-attention+positional encoding策略

    ![image-20201204224608071](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/sketchnet-connector.png)



图片插入可以引用github超链接（放到本repo的resources目录里）（图片名字不要带空格）

![image-20201028105709184](https://github.com/CCOM-AI-Music-Lab/MT-paper-notes/blob/main/resources/music_transformer_relative_local_attention.png)



## Results

- 针对sketchVAE的重建能力
    - baseline: measureVAE, EC2-VAE
    - 简单地比较了一下acc
- 针对sketchNet的生成能力
    - baseline: music inpaintNet
    - objective
        - loss
        - pitch acc
        - rhythm acc
        - 几个数据集均是SOTA
    - subjective
        - 从complexity structure musicality上都比inpaintNet强，但不如ground truth

## Discussion

感觉和弦也可以作为某个小节的隐变量编码进去（EC2-VAE里似乎做了和弦的disentanglement

怎么做到的mask 某个小节？（不同的小节的256维隐变量如何concat起来的

**为什么要先训sketchVAE, 再训sketchInpainter, 最后训sketchConnector? 不能直接端到端训练嘛？如果sketchVAE单独训练的话不就和measureVAE差不多了？**