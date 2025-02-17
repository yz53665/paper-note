- 它能为每个时间步输出一个独一无二的编码；
- 不同长度的句子之间，任何两个时间步之间的距离应该保持一致；
- 模型应该能毫不费力地泛化到更长的句子。它的值应该是有界的；
- 可被one-hot编码；
- 它必须是确定性的；
- 对于序列中的任何偏移量，都可以通过一个线性变换T进行转换，且T只与当前向量中的位置m、特征维度以及相对偏移量k有关，而与该词向量在句子中的绝对位置无关
### Attention 结构
$$
Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
d_k为K的向量维度（单词个数）。QK相乘后符合均值为0，方差为$d_k$的正态分布，所以需要除以$\s{d_k}$
PS: **先mask再计算softmax**

![[Pasted image 20231123113354.png](../attach/Pasted%20image%2020231123113354.png)
### 多头注意力机制
![[Pasted image 20240311190332.png](../attach/Pasted%20image%2020240311190332.png)
1. 扩展了模型关注不同位置的能力，
2. 给与注意力层更多的表达子空间。
用一个线性层处理后，reshape成多个头，再分别求attention，然后合并，最后再过一个FC层。需要注意的是：先过线性层再切分，而不是先切分再过8个线性层，8个更小的线性层相比一个合并的线性层少了互相之间的信息交互。
### Transformer 结构
![[Pasted image 20231123113318.png](../attach/Pasted%20image%2020231123113318.png)

1.  cross-attention中, Encoder的输出作为KV, Decoder的输入作为Q
2. encoder只使用padding mask, decoder需要使用padding mask和sequence mask
##### 推理过程
翻译“我爱你”到“I love you”为例：
+ encoder输入完整的中文embedding，最终得到kv
+ decoder先输入起始符，然后输出I
+ 将I输入decoder，最后得到love
+ 重复上述过程，得到输出：I love you (结束符)

该过程是串行的，可以实现不同长度句子的输出。
##### 训练过程
为了实现并行训练串行过程，输入完整序列，使用上三角矩阵，在softmax前一步mask掉attention

##### 对Transformer结构的理解
+ encoder是并行处理序列，BERT认为GPT是单向模型，没有充分发挥Transformer的特征提取能力。所以BERT使用了encoder部分，完全依赖位置编码刻画序列数据中的时空关联信息（BERT）
+ decoder是串行结构，GPT认为对于时间循环更加有利于刻画类似文本这样的序列数据（GPT）
当前的LLM模型大多数是decoder-only结构，图像编码器大多数是encoder-only结构（来源于BERT）
## 位置编码
长度为n的序列，t表示词在序列中的位置，d是维度向量，第i个特征的位置编码
$$
f(t)^{(i)} = 
\begin{cases}
\sin(\omega_k\cdot t), & if\ i=2k\\
\cos(\omega_k\cdot t), & if\ i=2k+1\\
\end{cases}
$$
$$
\omega_k = \frac{1}{10000^{2k/d}}
$$
特点：
+ 每个时间步输出独一无二
+ 不同长度的句子任何两个时间步之间距离一致
+ 值有界，可以泛化到更长的句子
+ 是确定的
Transformer位置编码为何采用三角函数：
+ 周期性、连续性、值有界
+ 另外任意两个元素之间的关系可以通过一个变换矩阵来表达，且该变换矩阵只与两个元素的相对位置，元素在特征维度的位置相关，与总长度


#### 计算量和显存占用量
![[attach/9891c65c58e35288939633892d8171b.jpg]]

#### 优点
+ 可以并行计算
+ 相比CNN，计算两个位置之间的关联所需要的操作次数不会随着距离增长而增加
+ attention机制可以产生更加具备解释性的模型，可以从模型中检查attention分布，各个head可以学会不同的任务

#### 缺点
+ 局部信息的获取不如RNN和CNN，总是更加倾向于提取长距离信息，层数越多越明显
+ 位置信息不具备词向量的可线性变换，在self-attention时可能会有信息损失
	+ ![[Pasted image 20240413165806.png](attach/Pasted%20image%2020240413165806.png)
	+ 下面两条线是在位置向量相乘后中间加入线性变换（也就是attention部分），会出现信息丢失。这里的线性变换矩阵W是随机的
+ Transformer模型由残差和层归一化组合而成，层归一化位于残差之间（残差->归一化->fc->残差），这会导致连乘，
+ attention机制使用softmax，除了会导致运算量增加，也会导致对异常值敏感，抑制较小数值的表达