## 背景
自回归问题：
$$
p(x_{n+1}|x_1,\cdots, x_n) = softmax(Wh_n^{(m)})
$$
掩码问题：
$$
p(x_{i}|x_1,\cdots, x_{i-1},x_{i+1},\cdots,x_n) = softmax(Wh_n^{(m)})
$$
#### 和之前PeFT（Parameter-Efficient Fine-Tuning）类方法的差异
##### 干预目标
+ PEFT修改权重或引入额外权重矩阵
+ ReFT不修改权重，直接干预前向传播时的隐藏层表示
##### 适应机制
+ PEFT学习权重更新或模型权重矩阵低秩近似值，推理时合并
+ ReFT学习干预，推理时操纵模型表示
##### 动机
+ PEFT减少调优大模型计算成本和内存需求
+ ReFT利用编码知识更有效适应模型

## 方法
#### 交换干预
线性表达假设认为概念被编码于神经网络表达的线性子空间中。

为了探究某个模块的功能，前人工作提出交换干预的方法，即将一个隐藏层输出固定到某处，再引入一个反事实的输入，如果该干扰持续影响模型的输出，并且影响方式符合我们的预期，那么该模块实现了我们预期中的因果功能。
对于一个隐藏层输出b，一个反事实输入s，表达如下：
$$
D||(b,s,R) = b+R^T(Rs-Rb)
$$
这里的$R\in \mathbb{R}^{r\times d}$是一个低秩映射矩阵，r是我们要干涉的子空间维度。

> [!NOTE] 个人理解
> 将两个隐藏层输出转换到另一低秩空间求取差异（一个正常一个反事实），再转换回来加到原始的隐藏层表达上


#### 低秩线性空间ReFT(LoReFT)
受交换干预（interchange intervention）启发，低秩线性空间ReFT表达如下：
$$
\Phi_{LoReFT}(h) = h + R^T(Wh+b-Rh)
$$
差异在于将干扰源换成了可学习的线性映射$Rs = Wh+b$，可以理解为转换到r维子空间来修改特征表达。这里的可学习参数包含$\phi = \{R,W,b\}$，其中$R\in \mathbb{R}^{r\times d}\ W\in \mathbb{R}^{r\times d}\  b \in \mathbb{R}^r$，语言模型LM的参数已被冻结。

###### 生成任务
记引入LoReFT的模型从$p(\cdot)$变为$p_\Phi(\cdot)$，记录输入x的普通LM隐藏表达为$h(x)$，引入干预的为$h_\Phi(x)$。对于生成任务：
$$
\arg\min_{\phi}\{- \sum_{m}^{i=1}\log p_\Phi(y_i|\rm{xy}_{<i}) \}
$$

###### 分类任务
添加一个分类头$H_\theta$
$$
H_\theta(\cdot | h ) = softmax(W_o(tanh(W_dh_1^{(m)}+b_d))+b_o)
$$
最终目标是学习分类头和中间的干预成分：
$$
\arg\min_{\phi,\theta}\{-\log H_\theta(y|h_\Phi(\rm{x}))\}
$$

#### 总结
ReFT可以总结为在前向传播时修改隐藏层输出量，包括如下几个关键量：
+ 带有可学习参数$\phi$的干扰函数$\Phi$
+ 一个输入位置集合$P\subseteq \{1,\cdots, n\}$，表明输入序列的第几项应用干扰函数
+ 一个层数集合$L\subseteq \{1,\cdots, m\}$，指明第几层应用干扰函数

![[Pasted image 20240423095025.png](attach/Pasted%20image%2020240423095025.png)