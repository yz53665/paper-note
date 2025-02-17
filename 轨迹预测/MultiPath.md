---
Title: "MultiPath: Multiple Probabilistic Anchor Trajectory Hypotheses for Behavior Prediction"
Year: "2019"
Publisher: CoRL; Waymo LLC组
tags: 
Datasets: 
输入: 
输出: 
多模态: 
概率建模: 
基于意图: false
关系提取: 
关系衡量: 
解决问题:
  - 预测的分布模态趋同
预测结构: 
未来环境建模: 
终点聚类: false
---
![[Pasted image 20240315152458.png](../img/Pasted%20image%2020240315152458.png)
### 模型特点
1. 一组简洁，带有权重的离散轨迹，并且可以覆盖可能的空间
2. 对任何可能轨迹的封闭式验证
这可以帮助下游任务进行关键推断，诸如类人行为假设和概率查询，如碰撞概率。

### 前人问题
+ CVAEs和GANs：非参数化的模型使得在更大的模型中重现和分析结果变得困难
+ 期望覆盖广、离散的轨迹的模型常常遭遇模式崩溃（单模态？）的问题
+ 而可追踪的概率推理会随着轨迹数量增长变得十分困难

##### 获得anchor
将轨迹用映射矩阵M转换到平移旋转不变的空间后再进行聚类获得anchor

##### anchor 应用
对于anchor集合$\mathcal{A} = \{a^k\}_{k=1}^K$来说，使用softmax建模不确定性：
$$
\pi(a^k|x) = \frac{\exp{f_k(x)}}{\sum_i\exp{f_i(x)}}
$$
这里的$f(x)$是网络输出。然后获得每一步的高斯分布：
$$
\phi(s_t^k|a^k,x) = \mathcal{N}(s_t^k | a_t^k+\mu_t^k(x), \Sigma_t^k(x))
$$
这里的$\mu\ \Sigma$都是网络训练得到。注意为了训练便捷，这里的每一步分布都和上一步无关。
最终获得整个状态空间上的联合分布：
$$
p(s|x) = \sum_{k=1}^K\pi(a^k|x)\prod_{t=1}^T\phi(s_t|a^k,x)
$$
##### 损失函数
是对标准GMM似然的时间序列扩展，本质是极大似然估计（使得获得的样本为概率最大的情况）：
$$
\mathscr{l}(\theta) = -\sum_{m=1}^M\sum_{k=1}^K\mathbb{1}(k=\hat{k}^m)[\log \pi(a^k|x^m;\theta)+\sum_{t=1}^Tlog\mathcal{N}(s_t^k|a_t^k+\mu_t^k,\Sigma_t^k;x^m;\theta)]
$$
这里的M是指样本个数，需要完成对anchor不确定性分布（第一项）和轨迹不确定性分布（第二项）的参数学习。



