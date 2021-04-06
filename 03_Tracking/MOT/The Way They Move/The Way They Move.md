# The Way They Move: Tracking Multiple Targets with Similar Appearance

**Paper Reading Note**

URL: [https://openaccess.thecvf.com/content_iccv_2013/papers/Dicle_The_Way_They_2013_ICCV_paper.pdf/](https://openaccess.thecvf.com/content_iccv_2013/papers/Dicle_The_Way_They_2013_ICCV_paper.pdf/)

## TL;DR
提出IHTLS算法(Hankel Total Least Squares)用于快速估计Hankel矩阵的秩，从而加速不同子track-let之间的，相似对象在碰撞、遮挡和相机移动条件下的匹配问题。

### Dynamics-based Multi-tracklet Association

1. $P_{i,j} = \frac{rank(H_{\alpha_i})+rank(H_{\alpha_j})}{min_\beta rank(H_{\alpha_i\beta\alpha_j})}-1$,即对于track-let$\alpha_i,\alpha_j$，首先构造一个用于连接的let$\beta$,并使连接后的Hankel矩阵的秩尽可能低。若$\alpha_i,\alpha_j$同属一个物体的轨迹，那么$\alpha_i,\alpha_j,[\alpha_i,\beta,\alpha_j]$的Hankel矩阵秩应相等，即$P_{i,j} \rightarrow1$,反之则有$P_{i,j} \rightarrow0$;
2. 在序列较长的情况下计算$P_{i,j}$，尤其是其中的$ min_\beta rank(H_{\alpha_i\beta\alpha_j}) $部分，对计算量的要求很大，作者在文中提出新的IHTLS算法来对$H_{\alpha_i\beta\alpha_j}$部分的rank进行估计而非计算;
3. 将检测结果进行简单缝合后（IOU Matching且和第二高重叠结果距离足够大）生成大量小跟踪序列，再将视频按时间分组后匹配以避免超常间隔带来的过大计算量，最后将长度小于5的track-let移除。
