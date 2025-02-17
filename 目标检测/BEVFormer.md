## 背景

#### 为何用BEV
+ 不同视角在BEV下统一进行表征便于支撑后续任务（融合时序信息及其它模态特征
+ BEV视角下没有图像的尺度和遮挡问题

#### 为什么要用时空融合
+ 时序信息很重要，但**当前基于视觉的方法没有用这一信息**
+ 是**视觉信息的补充**，帮助检测遮挡物体
+ 帮助判断物体运动状态，先前方法在缺少时序信息下无法有效判断物体运动速度

## 特点
+ transformer实现
+ 时序空间attention
![[Pasted image 20240418202448.png](attach/Pasted%20image%2020240418202448.png)

## 方法
![[Pasted image 20240418203107.png](attach/Pasted%20image%2020240418203107.png)
#### 1. 定义BEV Queries
定义一组``H*W*C``的可学习参数作为queries。nuScenes数据集上，queries空间分辨率200x200，对应自车周围100mx100m的范围。每个位于(x,y)位置的query都仅负责表征其对应小范围的区域。
这些queries通过对spatial space和tempoal space轮番查询从而能够将时空信息聚合在query特征中。能够支持包括3D目标检测和地图语义分割在内多种自驾感知任务

#### 2. Spatial Cross-Attention
通过空间交叉注意力机制，使BEV queries从多相机特征中通过注意力机制提取所需特征。由于维度较高，直接attention计算代价大，于是使用一种deformable attention的稀疏注意力机制使每个BEV query和部分图像区域进行交互。
对于每个(x,y)坐标，可以计算现实坐标(x', y')，然后进行lift操作，在z轴上获取多个3D points，并将这些点映射到2D平面，由于参数限制，每个BEV query一般只在1-2个图像上有有效的投影点，我们记录这些有落点的视图为$v_{hit}$。然后可以通过相机内外参数获取3Dpoints在图像平面上的投影点。以这些投影点为参考点，在周围进行特征采样，然后BEV query使用加权的采样特征进行更新。公式表达如下：
$$
SCA(Q_p,F_t) = \frac{1}{|V_{hit}|} \sum_{i\in v_{hit}}^{} \sum_{j=1}^{N_{ref}} DeformAttn(Q_p, \mathcal{P}(p, i, j), F^i_j)
$$
这里的$F^i_j$是第i个相机的特征，对于做个位置的BEV query $Q_p$，我们使用映射函数$\mathcal{P}(p, i, j)$来获得第j个参考点在第i个相机的图片坐标。

**如何获得映射坐标**
首先计算$Q_p$的真实世界坐标(x', y')，在3D空间中，在该位置的物体会有一个额外的z轴坐标，所以我们预先定义一系列的高度anchor$\{z'_j\}_{j=1}^{N_{ref}}$，最后通过相机的映射矩阵将这些做坐标映射到不同图片中：
$$
\mathcal{P}(p, i, j) = (x_{ij}, y_{ij})
$$
$$
where\ z_{ij}\cdot [x_{ij}\ y_{ij}\ 1]^T = T_i\cdot [x'\ y'\ z'\ 1]^T
$$
这里的$T_i\in \mathbb{R}^{3\times 4}$是已知的相机映射矩阵。
#### 3. Temporal Self-Attention
从经典RNN网络得到启发，将BEV视为能够传递寻猎信息的memory。每一个时刻生成的BEV特征都从上一时刻的BEV特征获取了所需的时序信息。
具体来说：
+ 给定上一时刻的BEV特征，根据ego motion将上一时刻的BEV特征和当前时刻对齐，确保统一位置的特征均对应于现实中的同一位置。对于一些可能云端距离比较大的情况，使用selfattn来对齐，$B'_{t-1}$是上一时刻手动对齐后的BEV特征：
$$
TSA(Q_p,\{Q, B'_{t-1}\}) = \sum_{V\in \{Q, B'_{t-1}\}} DeformAttn(Q_p, p, V)
$$
+ 对于当前时刻位于（x,y）处的BEV query，表征物体可能静态或动态，但可以知道上一时刻会出现在(x,y)一定范围内，于是再次利用deformable attention以(x,y)为参考点进行特征采样
+ 没有遗忘门，而是通过attention weights来平衡历史时序特征和当前特征融合过程。

**上述两个过程会重复多次，确保时空特征相互促进**

#### 4. 使用BEV特征支撑多种感知任务
