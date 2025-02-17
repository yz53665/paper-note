图神经网络表达式
$$
f(H^i,A) = \sigma(AH^iW^i)
$$
其中特征矩阵$H^i\in \mathbb{R}^{N\times F_i}$,邻接矩阵$A \in \mathbb{R}^{N\times N}$
一般的有向图不会考虑自身特征，所以需要保证图具有自环，一般会给邻接矩阵加上单位阵：
$$
\hat{A} = A + I
$$
另外为了避免梯度爆炸的问题，需要对所有特征进行归一化，具体就是让邻接矩阵乘以度矩阵的逆矩阵。度矩阵为对角阵，对角线上的元素依次为各个顶点的度。
$$
\tilde{A}^i = \hat{D}^{-1}\hat{A}H^iW
$$
```
# 生成度矩阵
D = np.array(np.sum(A, axis=0))[0]
D = np.matrix(np.diag(D))
```
