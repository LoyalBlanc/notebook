# Lukezic, 2019, D3S

*A Discriminative Single Shot Segmentation Tracker*

## Forward
<img src="./img/gaozhong_forward_01.png"  style="zoom:40%"  align="center"/>

训练过程：
1. 第一帧送入Backbone提取特征，通过不同分支获得定位(L, GEM)和前景F、背景B、Posterior(GIM);
2. LFP三个特征cat后送入Refinement，和Backbone的特征混合学习第一帧的Mask(Pos/Neg)。

推理过程：
1. 搜索区域送入Backbone提取特征，通过GIM获取前景F、背景B和Posterior；
2. 遍历模板和搜索，取相似度最高的K个像素求当前帧相似度的均值；
3. 同样cat送入Refinement获取当前帧的Mask和跟踪结果。

<img src="./img/gaozhong_forward_02.png"  style="zoom:50%"  align="center"/>

GIM(geometrically invariant model, 几何不变)
1. 共享参数计算Features(3x3卷积无激活函数)并做L2标准化，模板的Mask插值到当前帧的搜索空间大小；
2. 对模板和搜索作`einsum('ijkl,ijmn->iklmn')`，在通道维度相乘计算相似度，并将mn(搜索区域维度)变形为向量；
3. 找到mn向量上的topK特征图(Bkl)，求平均即可得到搜索帧上的前景和背景特征，前景F和背景B在cat后做Softmax，获得当前帧的Posterior。

<img src="./img/gaozhong_forward_03.png"  style="zoom:72%"  align="center"/>

GEM(geometrically constrained Euclidean model, 几何约束)



<img src="./img/gaozhong_forward_04.png"  style="zoom:66%"  align="center"/>

Refinement Module
1. Posterior，前景F和定位L一起cat后送入Refinement，通过Upsample和Backbone特征混合；
2. 最终得到两通道特征图，Softmax后即为当前帧的跟踪结果(Mask)。

## Backward
