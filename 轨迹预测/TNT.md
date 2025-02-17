---
Title: "TNT: Target-driveN Trajectory Prediction"
Year: "2020"
Publisher: AAAI
tags:
  - waypoints
  - goal
  - refine
  - GMM
  - GCN
Datasets:
  - ETH-UCY
  - SDD
输入:
  - position
输出:
  - position
  - disaplacement
多模态:
  - distribution
概率建模:
  - GMM
基于意图: true
关系提取:
  - GCN
关系衡量:
  - relative position
  - disaplacement
解决问题:
  - 累积误差
  - 未来关系
预测结构: 
未来环境建模: 用于训练阶段反向训练基于RNN的VAE分布
---

# 轨迹预测前人研究问题
将行人意图作为潜在的隐变量。

# 提出解决思路
行人运动的不确定可以分为
+ 意图不确定性（target or intent uncertainty）
+ 控制不确定性（control uncertainty）

预测分为三个阶段：
+ 目标预测
+ 基于目标的轨迹预测
+ 轨迹评分和选择
# 模型
![[Pasted image 20240308194905.png](../img/Pasted%20image%2020240308194905.png)
### 1. 场景图编码
有HD图用VectorNet，只有俯视图就用卷积网络
### 2. 目标预测
候选点生成：
车辆轨迹：在HD地图车道线中央均匀生成
行人轨迹：在行人周围使用网格均匀采样
首先提供目标的概率分布$p(\tau|x)$，文章将目标建模为离散的、量化的候选位置与连续的偏差：
$$
\tau = \{\tau^n\} = \{(x^n, y^n) + (\Delta x^n+\Delta y^n)\}^N_{n=1}
$$
所以最终的目标概率分布由离散的采样点和偏差所组成：
$$
p(\tau|\rm{x}) = \pi(\tau^n|\rm{x}) \cdot \mathcal{N}(\Delta x^n | \upsilon^n_x(x)) \cdot \mathcal{N}(\Delta y^n | \upsilon^n_y(x))
$$
其中第一项本质是个softmax，求取各个点是离目标最近候选点的概率：
$$
\pi(\tau^n|\rm{x}) = \frac{\exp f(\tau^n,x)}{\sum_{\tau'}\exp f(\tau',x)}
$$
$f,\upsilon$都是2层的MLP，并且都使用候选目标坐标和编码的历史和场景特征作为输入。目的是为了预测目标位置附近的分布和最优可能的偏移。损失函数：
$$
\mathcal{L}_{S1} = \mathcal{L}_{cls}(\pi, u) + \mathcal{L}_{offset}(\upsilon_x, \upsilon_y, \Delta x^u, \Delta x^u)
$$
第一项是交叉熵损失，表示离目标最近的点的概率预测，第二项是Huber损失，表示偏差预测是否准确。u是离目标最近的候选点，$\Delta x^u, \Delta x^u$是最近候选点u离目标的偏移（这里都是真实值）。最终目标是生成$(\pi, \Delta x^u, \Delta x^u)$。

最大优点：不用担心mode averaging
召回率：正确预测为正的占实际为正的比例

### 3. 基于目标的运动估计
$$
p(\rm{s}_F | \tau, \rm{x}) = \prod_{t=1}^Tp(s_t|\tau,\rm{x})
$$
两个猜想：
+ 未来每一步都是独立的，避免序列预测
+ 分布是关于目标的的单峰分布
输出关于每个未来意图的轨迹。
为了保证预测的平滑性，在训练阶段采用了teacher forcing technique（教师强制技术），将真实的坐标喂给模型。该阶段使用的Huber损失
$$
\mathcal{L}_{S2} = \sum_{t=1}^T \mathcal{L}_{reg}(\hat{s}_t, s_t)
$$
### 4. 轨迹打分和选择
最后一个阶段估计完整轨迹的似然性。一条轨迹的目标似然可能很大，但组合在一起形成完整轨迹就不一定了。
文章使用最大熵模型来打分：
$$
\phi(\rm{s_F}|\rm{x}) = \frac{\exp(g(\rm{s_F}, \rm{x}))}{\sum_{m=1}^M\exp(g(s_F^m, \rm{x}))}
$$
g是两层MLP。
损失函数是预测分数和真实分数之间的交叉熵
$$
\mathcal{L}_{S3} = \mathcal{L}_{CE}(\phi(\rm{S}_F|x), \psi(\rm{S}_F))
$$
每条预测轨迹的目标分数是由和真实轨迹之间的距离定义的：
$$
\psi(\rm{s}_F) = \frac{\exp(-D(\rm{s}, \rm{s}_{GT}) / \alpha)}{\sum_{s'}\exp(-D(\rm{s'}, \rm{s}_{GT}) / \alpha)}
$$
这里的D就是距离函数。本文的距离函数定义如下:
$$
D(s^i, s^j) = \max(\parallel s^i_1 - s^j_1\parallel_2^2, \cdots, \parallel s^i_t - s^t_1\parallel_2^2)
$$
##### 5. 训练和推理细节
$$
\mathcal{L} = \lambda_1\mathcal{L}_{S1} + \lambda_2 \mathcal{L}_{S2} + \lambda_3\mathcal{L}_{S3}
$$
组成：
1. 目标预测：采样点概率（CE损失）+偏差预测准确度（Huber损失）
2. 轨迹预测：轨迹偏差（Huber损失）
3. 完整轨迹似然性：轨迹得分（CE损失）

# 实验验证

# 总结与讨论
