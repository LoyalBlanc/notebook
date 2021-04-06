# Structure Preserving Object Tracking

**Paper Reading Note**

URL: [https://ieeexplore.ieee.org/document/6619084](https://ieeexplore.ieee.org/document/6619084)

## TL;DR

跟踪器同时计算特征相似度得分和空间关系变化得分。

## Structure-Preserving Object Tracker

1. 原文直接使用的当时的HOG特征检测+SVM分类器进行跟踪；
2. 得分定义 $ s (C;I,w,e)= \sum w_i\phi(I;B_i) - \sum\lambda_{i,j}|| (x_i-x_j)-e_{i,j}||^2 $ ：其中前半部分是HOG特征分类器的得分，即当前坐标下的anchor是否属于待跟踪物体，后半部分是结构关系的得分，即当前状态下的位置关系和上一帧位置关系e的差值。w和e是可学习的参数， $\lambda$ 则为超参数。
3. loss定义 $ loss = max_\hat C(s(\hat C)-s(C)+\sum(1-\frac{B_i\cap\hat B_i}{B_i\cup\hat B_i})) $ ,后面那部分的求和就是IOU的变种，有重叠区域则小反之则大。s(C)是上一帧一个固定的值，因此max这个loss的作用就是找到当前帧最优的bbox组合使得(1)尽量满足HOG分类器且保持结构不变(2)尽量和上一帧的结果不重合，即找和上一帧不重合但最相似的bbox。
4. 梯度更新 $ \theta\leftarrow\theta-\frac{loss}{||\nabla loss||^2+1/2K}\nabla loss $ ,把3中找到的bbox(困难样本)作为loss进行梯度下降，抑制那些得分高但是结构和上一帧不同的bbox序列。
5. 推理阶段直接在当前帧下寻找得分最高的bbox即可。

## Thoughts
1. SPOT保留结构的主要策略在把结构变化编进score，然后抑制得分高的负样本，这部分可以刷在loss里。现在transtrack直接把上一帧的tgt拿来问这一帧的memory，其中是有大量的无用tgt，也即上一帧检测结果中不要的部分。如果只保留有用的tgt会导致学崩的话，那可以考虑针对有用tgt进行结构保留的Loss；
2. 最关键的，必须验证tgt和输出bbox之间是否可以建立一一对应的关系，目前来看原始的训练策略会使得bbox顺序发生变化(IOU Matching条件，存疑)，或许可以考虑移除IOU Matching而是一一对应训练，再加个shuffle，或许能迫使模型学成一一对应，这个时候是否还应该加query embedding待讨论。