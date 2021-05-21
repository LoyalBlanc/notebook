# Kanazawa, 2018, HMR

***End-to-end Recovery of Human Shape and Pose***

URL: [https://openaccess.thecvf.com/content_cvpr_2018/papers/Kanazawa_End-to-End_Recovery_of_CVPR_2018_paper.pdf](https://openaccess.thecvf.com/content_cvpr_2018/papers/Kanazawa_End-to-End_Recovery_of_CVPR_2018_paper.pdf)

## Basic

*SMPL: a skinned multi-person linear model*

*Expressive Body Capture: 3D Hands, Face, and Body from a Single Image*

<img src=".\img\SMPL_01.jpg" alt="img" style="zoom: 67%;" />

1. 输入$\theta,\beta,\phi$ ，输出是N个蒙皮点$Mesh$和K个关节点$J(\beta)$

   $Mesh(\theta,\beta,\phi)=W(T_p(\theta,\beta,\phi),J(\beta),\theta,\omega)$

   其中 $\theta\in R^{3(K+1)}$ 表示每个节点相对父节点的旋转关系(yaw,roll,pitch)，$\beta$ 表示体型，$\phi$ 表示面部表情

   $T_p(\theta,\beta,\phi)=\overline T+B_S+B_E+B_P,J(\beta)=\mathbf {J(\overline T+B_S)}$

   平均模型$\overline T$，修正体型Shape，表情Emotion，动作Pose，$J(\beta)$表示从模型的表面顶点中获取每个关键点的坐标

<img src=".\img\SMPL_02.jpg" alt="img" style="zoom: 67%;" />

2. 图像+openpose关键点 $\rightarrow$ Resnet18输出性别概率 $\rightarrow$ 选择预设模型(男/女/中性)

3. 假设已知相机焦距 $\rightarrow$ 估计相机的平移和身体的方位 $\rightarrow$ 逐级优化动作参数$\theta$和体型参数$\beta$ 

4. 使用VAE对动作进行检测 输入是每个关节点的旋转矩阵R

   $L_{KL}=KL(q(Z|R)||N(0,1))\ Z\in R^{32}$ 是VAE的隐空间 $\rightarrow$ 迫使编码器将动作正态分布

   $L_{rec}=||R-\hat R||^2_2\ \hat R$ 是解码器输出 $\rightarrow$ 迫使编码器输出包含完整动作信息

   $ L_{rota}=||\hat R\hat {R'}||^2_2+|det(\hat R)-1|$  $\rightarrow$ 保证解码器结果是旋转矩阵

## Model
<img src=".\img\HMR_01.jpg" alt="image-20210518110558930" style="zoom: 67%;" />

### Encoder & 3D Regression

<img src=".\img\HMR_02.jpg" alt="image-20210518110558930" style="zoom: 67%;" />

1. 单帧图像输入$RGB\ 224\times224 $ $\rightarrow$ ResNet提特征 $\phi\in R^{2048}$ 
1. 初始为默认参数 ```neutral_smpl_mean_param.h5``` 
2. MLP结构: 图像特征 + SPML参数 $\rightarrow$ 参数$\Delta$,  2048+85$\rightarrow$1024(ReLU+dropout)$\rightarrow$1024(ReLU+dropout)$\rightarrow$85
4. Loss

   - 2D Loss: $L_{2D}=||v_i(x_i-\hat{x_i})||_1$ 利用SMPL模型得到的$J(\beta)$ 和2D真值计算的加权loss(可见性)

   - 3D Loss: $L_{3D}=||(X_i-\hat{X_i})||^2_2+||[\beta,\theta]-[\hat\beta,\hat\theta]||^2_2$ 

### Discriminator
1. 对学习到了人体结构进行约束 防止过于离谱的姿态 (投影到2D上是相同的结果)
   - Shape: 10$(\theta)\rightarrow$5(ReLU)$\rightarrow$1
   - Pose: $K\times9\rightarrow32\times1\times9(conv1\times1)\rightarrow$ 单个$MLP\ 32\rightarrow32\rightarrow1$整体$MLP\ K\times32\rightarrow1024\rightarrow1$ 
2. Loss
   - Encoder+3D Regression对抗 $minL_{adv}(E)=\sum_i\Bbb E_{p_E}[(D_i(E(Image))-1)^2]$ 
   - Discriminator目标 $ minL_{D_i}=\Bbb E_{p_E}[D_i(E(I))^2]+\Bbb E_{p_{data}}[(D_i(\Theta)-1)^2]$ 

## Extension
**Learning to Reconstruct 3D Human Pose and Shape via Model-fitting in the Loop**

<img src=".\img\SPIN_01.jpg" alt="image-20210519091134806" style="zoom: 80%;" />

1. 和HMR一样预测SMPL模型的参数 $\beta, \theta$ ，和SMPLify一样迭代式地将3D结果往2D标注上拟合

2. 采用退火方案来处理局部最优解，先提高肢体动作部分再提升双手和脸部

   重投影误差 $E_J=\sum_i\gamma_i\omega_i\rho(\Pi_K(R_\theta(J(\beta))_i)-J_{gt,i})$ $\Pi_K$表示按相机位置投影到2D，$R_\theta(J(\beta))_i$ 表示按动作将关键点旋转

   L2正则项 (手部 $E_{\theta f}$ 脸部 $E_{mh}$ 体型 $E_\beta$ 面部 $E_\phi$) $\rightarrow ||\Theta||^2$ 

   手肘/膝盖先验(只能正向弯曲) $E_\alpha=\sum_{i\in(elbows,knees)}\exp(\theta_i)$ 

   互穿惩罚[?] $E_c$ 基于碰撞的表面模型 属于计算机图形学部分了

3. 整个pipeline只需要2D的真值标注，利用迭代结果作为预测结果的监督计算loss

**VIBE: Video Inference for Human Body Pose and Shape Estimation**

<img src=".\img\VIBE_01.jpg"  style="zoom: 80%;" />

1. Resnet50提取图片的空间特征 $\rightarrow$ GRU处理序列学习时间特征 $\rightarrow$ 回归层得到SMPL参数 $\rightarrow$ 运动鉴别器判断真假

<img src=".\img\VIBE_02.jpg" style="zoom: 67%;" />

2. 连续帧的feature按时序送GRU计算SMPL，鉴别器将GRU的输出通过一个MLP计算各帧权重后再判断真假