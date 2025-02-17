## 方法
##### 全量微调
一个自回归语言模型$P_\Phi(y|x)$，参数是$\Phi$，数据集为语义目标对$\mathcal{Z}={(x_i,y_i)}_{i=1,\cdots,N}$，在初始化权重$\Phi_0$时，需要最大化如下条件语言模型来达到$\Phi_0+\Delta\Phi$：
$$
\arg \max_{\Phi}\sum_{(x,y)\in\mathcal{Z}}\sum^{|y|}_{t=1} \log(P_{\Phi}(y_t|x,y\lt t))
$$
##### 本文方法
通过优化一个满足$|\Theta|\lt\lt |\Phi_0|$的更小的参数子集$\Theta$来取代全量微调：
$$
\arg \max_{\Theta}\sum_{(x,y)\in\mathcal{Z}}\sum^{|y|}_{t=1} \log(P_{\Phi_0+\Delta\Phi(\Theta)}(y_t|x,y\lt t))
$$
##### 具体实现
 
 ![[Pasted image 20240421102209.png](attach/Pasted%20image%2020240421102209.png)
 对于一个预训练权重矩阵$W_o\in\mathbb{R}^{d\times k}$，将其更新替换为低秩分解：
$$
 W_0+\Delta W = W_0 + BA
$$
这里的$B\in \mathbb{R}^{d\times r},\ A\in\mathbb{R}^{r\times k},\ r\ll \min{(d,k)}$。前向传播时输出h和输入x的关系如下：
$$
h = W_0x + \Delta Wx = W_0x + BAx
$$
训练时W_0被冻结停止更新，只更新包含可训练参数的A和B。本文对A使用高斯分布随机初始化，B为全0，BA则为全0，然后使用$\frac{\alpha}{r}$来缩放BA相乘的结果，防止梯度爆炸，这里的$\alpha$用于微调比例，是一个常数，文章直接设置为尝试的第一个r。

##### 方法的泛化性
相比之前的MLP或基于前缀的方法，LoRA通过增大r可以不断逼近原始的全量微调效果。

##### 无需额外的推理
在实际部署推理时，可以直接预先计算$W=W_0+BA$，达到和全量微调一样的实际使用效果。


## 实验及分析

#### 对Transformer的影响
在Transformer中共有4个权重矩阵$W_q,W_k,W_v,W_o$，还有两个在MLP中。影响如下：
![[Pasted image 20240421172259.png](attach/Pasted%20image%2020240421172259.png)

#### 优点与不足
优点：
+ 降低训练时的存储占用
+ 极大降低checkpoints的大小
+ 通过切换LoRA权重，可以快速在不同任务中进行切换
+ 加速训练过程（在GPT-3 175B上和全量微调相比只替换QV可以提升25%的速度）

不足：
+ 如果将A和B整合进weight中，将难以实现在一次前向传播过程中同时推理包含不同任务的同一个batch数据


### 思考
B矩阵全零初始化，A矩阵正态分布初始化的原因？
+ 全零初始化可以使得模型一开始训练时LoRA分支输出为0，让模型的结果等于预训练模型
+ 如果全部都是随机初始化或正态分布初始化，会导致一开始就偏离预训练模型
+ 如果全0初始化，会出现梯度消失问题