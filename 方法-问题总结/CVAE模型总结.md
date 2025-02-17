
| 标题                                                                                             | 说明                                                                                       | 链接           |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------ | -------------- |
| The Trajectron: Probabilistic Multi-Agent Trajectory Modeling With Dynamic Spatiotemporal Graphs | 轨迹预测领域应用CVAE开山鼻祖                                                               | [[../阅读记录/2023-09-26.md]] |
| It is not the Journey but the Destination: Endpoint Conditioned Trajectory Prediction            | 将CVAE和intention结合                                                                      | [[../阅读记录/2023-09-27.md]] |
| Trajectron++: Dynamically-Feasible Trajectory Forecasting With Heterogeneous Data                | 集大成作，结合了CVAE，简单的基于距离的关系图，语义图                                       | [[../阅读记录/2023-09-28.md]] |
| BiTraP: Bi-Directional Pedestrian Trajectory Prediction With Multi-Modal Goal Estimation         | 提出了当前循环神经网络存在累积误差的问题，尝试引入反向过程；另外提出使用相对位置输出的好处 | [[../阅读记录/2023-09-29.md]] |
| AgentFormer: Agent-Aware Transformers for Socio-Temporal Multi-Agent Forecasting                 | 提出前人将时间关系和空间关系分开处理，会损失部分时空信息，于是提出AgentFormer同时处理两者  | [[../阅读记录/2023-10-03.md]] |
| Stepwise Goal-Driven Networks for Trajectory Prediction                                          |  将CVAE、RNN和intention结合                                                                                          |                |
``` mermaid
graph TD
	first["GAN"] --> second["CVAE（2019-2022）"]
	second --> third["Diffusion（2022-）"]
```
