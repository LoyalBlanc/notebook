# Basic Operations

## 反池化(Unpooling)
<img src="./img/gaozhong_forward_01.png"  style="zoom:66%"  align="center"/>
- 通常指代Max pooling的逆过程，通过记录最大池化时最大值点的坐标将卷积等操作后的结果填回对应的位置，其余补零。

## 上采样(Upsampling)
<img src="./img/gaozhong_forward_02.png"  style="zoom:66%"  align="center"/>

1. 最近邻插值，计算开销小，锯齿状明显；

<img src="./img/gaozhong_forward_03.jpg"  style="zoom:66%"  align="center"/>

2.	双线性插值，图像连续，计算量大、图像轮廓可能模糊。

## 转置卷积(Transposed convolution)
<img src="./img/gaozhong_forward_05.png"  style="zoom:66%"  align="center"/>
$$
o = size\ of\ output,\ i = size\ of\ input,\ p = padding,\ s = stride \\
o = s(i-1)-2p+k+(o+2p-k)\%s
$$
## 深度可分离卷积(Separable convolution)
<img src="./img/gaozhong_forward_06.jpg"  style="zoom:66%"  align="center"/>

- Separable = Depthwise + Pointwise