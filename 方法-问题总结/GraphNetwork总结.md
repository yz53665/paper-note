| 标题                                                                                                       | 说明                                                        | 连接           |
| ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | -------------- |
| Social-STGCNN: A Social Spatio-Temporal Graph Convolutional Neural Network for Human Trajectory Prediction | 首次提出在行人轨迹预测领域使用图网络                        | [[../阅读记录/2023-09-16.md]] |
| SGCN:Sparse Graph Convolution Network for Pedestrian Trajectory Prediction                                 | 通过构建稀疏有向图，减少了连接边的数量并赋予方向            | [[../阅读记录/2023-09-18.md]] |
| Disentangled Multi-Relational Graph Convolutional Network for Pedestrian Trajectory Prediction             | 构建解耦多尺度图，减少边的数量并通过DropeEdge提升模型泛化性 | [[../阅读记录/2023-09-19.md]] |
| Learning Pedestrian Group Representations for Multi-modal Trajectory Prediction                            | 提出将组整体看作一个节点                                    | [[../阅读记录/2023-09-20.md]] |
| GroupNet: Multiscale Hypergraph Neural Networks for Trajectory Prediction with Relational Reasoning        | 提出提取多尺度多类型特征                                                            | [[../阅读记录/2023-09-21.md]]               |

``` mermaid
graph TD
	first["Social-STGCNN"] 
	subgraph subone["稀疏有向图(2021)"]
		second[SGCN]
		third[DMRGCNN]
	end
	first --> subone
	subgraph subtwo["增加多尺度图表达(2022)"]
		fourth[GroupNet]
		fifth[GPGraph]
	end
	subone --> subtwo
```

趋势：
图网络本身可以表达丰富的轨迹信息，但当前制约模型精确度的并不是模型社会关系表达方式，而是一些其它的因素。目前单纯依靠图网络改进来提升模型的整体精度已经非常困难了。另外目前榜单得分最高模型的特点也说明还有很多更加轻量级的表达和提取方法。
