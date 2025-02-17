
2001 - Neural language models（神经语言模型）  
2008 - Multi-task learning（多任务学习）  
2013 - Word embeddings（词嵌入）  
2013 - Neural networks for NLP（NLP神经网络）  
2014 - Sequence-to-sequence models  
2015 - Attention（注意力机制）  
2015 - Memory-based networks（基于记忆的网络）  
2018 - Pretrained language models（预训练语言模型）


语言模型分类：统计语言模型、神经语言模型

## 1. 统计语言模型 N-gram
#### 基本概念

长度为m的句子包含词w_1 ~ w_m，则句子的概率为历史出现过词的条件概率连乘：
![[Pasted image 20240911225537.png](image/Pasted%20image%2020240911225537.png)
统计语言模型目的是使得条件概率最大化。
考虑近因效应，一般只考虑当前词前5个词作为条件。所以经典n-gram表达：
![[Pasted image 20240911225714.png](image/Pasted%20image%2020240911225714.png)
#### 特点

局限性：
+ 参数空间爆炸式增长，无法处理更长的context（N>3）
+ 并没有考虑词与词之间的内在联系
	+ The cat is running in the bedroom
	+ The dog is walking in the bedroom
	+ 无法从cat和dog的相似性推测出The cat is walking in the bedroom的概率


## 2. 神经语言模型NNLM
来自《A Neural Probabilistic Language Mode》
![[Pasted image 20240911230406.png](image/Pasted%20image%2020240911230406.png)
词表大小为N，维度m，n表示上下文包含的词数，不超过5

one-hot编码词表存在问题：
+ 数据稀疏，词表很大时，计算量太大
+ 词之间的关系无法表达
转而使用word embedding：
+ 随机初始化得到
+ 通过反向传播学习
+ 是NNLM学习的副产物

NNLM存在问题：
+ 速度慢，词表大，训练耗时久
## Word2Vec
word embedding思想：通过模型学习词语到词性的映射关系，映射关系可以隐含的表示为某个特种空间的嵌入

word2vec是2013年提出的一种高效训练词向量的模型。核心思想：上下文相似的词，词向量也应该相似。
例子：香蕉和梨经常出现在同一上下文中，所以香蕉和梨的词向量比较相似

有两个模型：
+ CBOW（Continuous Bag of word）上下文预测中间某个词
+ SkipGram，当前词预测上下文

整体结构，以句子I drink coffee everyday为例，用 I drink everyday预测coffee：
1. 输入分别和网络降维部分权重相乘，再对所有词取平均，得到隐藏编码
2. 隐藏编码再乘升维权重得到输出得分
3. 输出得分softmax，得到词表每个词概率
![[Pasted image 20240911232443.png](image/Pasted%20image%2020240911232443.png)

#### Hierarchical SoftMax 降低softmax求解复杂度
复杂度O(N) -> O(logN)
思想：常见词放底层，不常见放高层。
构造方式：统计语料词频，构建Huffman树

![[Pasted image 20240911233715.png](image/Pasted%20image%2020240911233715.png)

图中每个节点都有一个可学习向量$\theta_i^w$
每个节点决定往左还是右由logistic回归方法分类：
$$
p_i = \frac{1}{1+e^{-x^T\theta_i^w}}
$$
最终概率为所有路过节点概率连乘。