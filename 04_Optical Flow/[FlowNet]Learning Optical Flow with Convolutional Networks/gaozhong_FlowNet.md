# Dosovitskiy, 2015, FlowNet

*FlowNet: Learning Optical Flow with Convolutional Networks*

## Forward
1. 模型结构
   <img src="./img/gaozhong_forward_01.png"  style="zoom:75%"  align="center"/>
   - Simple：两张图片Cat在一起进CNN；
   - Corr：各自提取Feature后做Corr操作
     <img src="./img/gaozhong_forward_03.png"  style="zoom:50%"  align="center"/>
     - $ c(x_1,x_2) = \sum_D\sum_i<f_1(x_1+i),f_2(x_2+i)>$，其中$ i\in [-k,k]\times[-k,k]$
     - 对map2上任意一个vec，取map1上范围D(文中D=41，stride=2)，做向量内积得441通道数；
     - 通过redir部分将key image的特征cat到后面。
   - 通过refinement进行上采样恢复feature分辨率
2. Refinement Module
   <img src="./img/gaozhong_forward_02.png"  style="zoom:50%"  align="center"/>
   - 类似decoder结构，通过反卷积逐层叠加(cat)更高分辨率特征；
   - 每层结果均输出一个低分辨率的光流预测，上采样后同样cat到下一层(文中未提及中间监督问题)；
   - 作者对比了更多层的refine结果，并没有较大提升，考虑到计算量的问题最后采用了线性上采样。

## Backward

1. Flying Chairs
   <img src="./img/gaozhong_backward_01.png"  style="zoom:75%"  align="center"/>
   - 考虑到没有足够的含光流真值的数据集可供训练，作者模拟了一个数据集。