$$
S_i = \frac{e^{x_i}}{\sum_{j=1}^{K}e^{x_j}}
$$
## Safe-Softmax
一般的浮点数是有限的，当x>89的时候，$e^{x}$就会变得很大直到inf，发生数据上溢的问题。
为了保证数值的稳定性，通常会减去最大值，即
$$
S_i = \frac{e^{x_i-max(\{x_j\}_K)}}{\sum_{j=1}^{K}e^{x_j-max(\{x_j\}_K)}}
$$
优点：
+ 避免数据上溢
+ 加快运算速度

## 分层softmax（**hierarchical softmax**，实际不是softmax！！）

当词表很大时，使用softmax去计算所有的概率再找到最大值是很耗时的操作。
word2vec使用了哈夫曼树（**Huffman Tree**）来做分层softmax。
哈夫曼树为带权重最短二叉树：
![[Pasted image 20240822004617.png](../../img/Pasted%20image%2020240822004617.png)
其中b为优化过的哈夫曼树。通过树状搜索来查找词

## 差分softmax（**differentiated softmax**）
