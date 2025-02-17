
| 标题                                                                                      | 说明                                 | 简称    | 链接           |
| ----------------------------------------------------------------------------------------- | ------------------------------------ | --- | -------------- |
| MANTRA: Memory Augmented Networks for Multiple Trajectory Prediction                      | 首次将记忆网络应用于轨迹预测         | MANTRA    | [[../阅读记录/2023-10-07.md]] |
| Remember Intentions: Retrospective-Memory-based Trajectory Prediction                     | 简化记忆网络，使用终点来生成未来轨迹 |  MenoNet   | [[../阅读记录/2023-09-15.md]] |
| Online Adaptive Temporal Memory with Certainty Estimation for Human Trajectory Prediction | 实现在线优化轨迹                     | OATMem    | [[../阅读记录/2023-08-30.md]] |

## 记忆库控制方式：
### MANTR
$$
\mathcal{L}_c = e \cdot(1-P(\omega)) + (1-e) \cdot P(\omega)
$$
当误差e小的时候，说明当前记忆充分，不需要这个样本，最小化概率；当e大的时候，说明模型缺乏当前样本的记忆，最大化概率。

#### 问题与讨论
亮点：
+ 根据整体预测误差来决定记忆的控制

问题：
+ 记忆的接收不应该根据对当前观测事件的判断正确与否，这会导致过拟合和对未见样本预测的潜在失误。而应该根据记忆的差异，不断接收未见的新知识
+ 受样本顺序影响

### MemoNet
采用一种基于起止点的循环匹配方式
![[Pasted image 20231008095946.png](../img/Pasted%20image%2020231008095946.png)
算法流程中提到的公式1：
$$
\parallel x_i^{-T_p+1} -x_j^{-T_p+1} \parallel_2 \le \theta_{past}, \ \parallel y_i^{-T_p+1} -y_j^{-T_p+1} \parallel_2 \le \theta_{past}
$$
和MemoNet比优点：
+ 不受顺序影响
+ 无需训练，更加高效
#### 问题与讨论
亮点：
+ 见前文

问题：
+ 基于轨迹起止点的判断无法充分反映运动的多样性
+ 记忆库的鲁棒性和整体性能存疑

### OATMem

更新采用FIFO，总是保留最新的，可能更适合小样本测试？
