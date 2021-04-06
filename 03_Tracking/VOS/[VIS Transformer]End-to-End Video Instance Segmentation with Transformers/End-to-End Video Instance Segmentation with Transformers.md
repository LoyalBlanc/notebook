# End-to-End Video Instance Segmentation with Transformers

**Paper Reading Note**

URL: [End-to-End Video Instance Segmentation with Transformers](https://arxiv.org/abs/2011.14503)

## TL;DR

把连续的图片传入DETR从而实现分割效果。

## Basic Architecture

1. Backbone  $ResNet+1\times1\ Conv: T\times 3\times H\times W \rightarrow T\times C \times H \times W \rightarrow T\times d \times H \times W$
2. Encoder  $ T\times d \times H \times W \rightarrow d\times (THW) $ , $ PE(pos,i) = \left\{ \begin{matrix}sin(pos\ \omega_k) ,i=2k \\ cos(pos\ \omega_k),i=2k+1  \end{matrix} \right . $
3. Decoder  默认query和instance按顺序一一对应

## Instance Sequence Matching/Segmentation

1. 将数据集中不足N个的真值补零到N 然后用匈牙利算法计算二分图 不同之处在于是seq到seq的匹配 这里作者是如何保证transformer输出顺序的文中似乎没有提及[?] 可能是数据集本身的分布比较稀疏
   - Cross Entropy(class) + IOU&L1(bbox)
2. 单帧的hs和memory作为qk送入MHAttention计算注意力图 这个注意力再和memory一起用于计算seg的mask(纯卷积+最后一层Deformable) 理论上这里得到了每帧一张的seg mask 但似乎并不在loss中体现[?]
3. $ 3D\ Conv: Single\ 1\times c\times H\times W \rightarrow Seq\ 1\times c\times T\times H\times W \rightarrow Seg\ 1\times 1\times T\times H\times W$
   - Dice loss + Sigmoid Focal loss