《UNITER: UNiversal Image-TExt Representation Learning》

## 特点
+ 一种通用的图-文表达，可以适应多种不同的下游任务如
	+ 基于图片的遮掩语言建模
	+ 基于文本的遮掩区域建模
	+ 图-文匹配
	+ 词-区域对齐
+ 相比前人工作，MM（Mask Modeling）是基于完整序列，而不是同时在两种模态上
+ 提出了用WRA（词-区域对齐）任务搭配优化迁移来专门促使图文的细粒度对齐

## 方法
![[Pasted image 20240414162510.png](attach/Pasted%20image%2020240414162510.png)
#### 1. 结构
##### Image Embedder
使用了Faster R-CNN来处理图片，然后和位置坐标一起通过FC层，来映射到同一嵌入空间。最后相加再通过LN层

##### Text Embedder
跟随BERT并且tokenize input into WordPieces
随后加入词编码和位置编码

##### 多任务学习
MLM和MRM都来自BERT，尝试遮挡一部分并恢复出来。
与前人工作不同的是，每次只遮挡一个模态的数据，而不是随机遮挡所有数据，这可以避免潜在的误对齐（如遮掩的词预测遮掩的图）

本文还尝试了在整张图和句子的**实例级别的对齐**。

每个batch都随机选择一个任务进行，并且只有该任务的约束

#### 2. 设置的任务
Masked Language Modeling(MLM)
图像区域$v=\{v_1, \cdots, v_K\}$，输入词区域$w=\{w_1, \cdots, w_T\}$, 掩膜索引$m\in\mathbb{N}^M$，$\mathbb{N}$是自然数，M是遮掩的tokens数，m是遮掩下标集合。15%的词进行替换，这其中10%替换成随即词，10%不变，80%为\[mask\]下标。目标基于图片区域v以及周围词$w_{\\m}$来预测，损失函数：
$$
\mathcal{L}_{MLM}(\theta) = -\mathbb{E}_{w,v}\sim D \log P_\theta (w_m | w_{\m}, v)
$$

Image-Text Matching(ITM)
额外的token \[cls\]用于指示融合的表达，该任务输出是二维标签0,1，表示采样的图文对是否匹配。cls token输出回通过fc和sigmoid来预测0~1之间的分布，定义为$s_\theta(w,v)$，交叉熵损失表达如下：
$$
\mathcal{L}_{ITM}(\theta) = -\mathbb{E}_{(w,v)\sim D}[y\log s_\theta(w,v)+(1-y)\log(1-s_\theta(w, v))]
$$

Word-Region Alignment(WRA)


Mastked Region Modeling(MRM)
基本和MLM任务类似，但由于图片是高维度和连续的，不能通过类别似然来监督。于是提出3种变体适用相同的约束损失：
$$
\mathcal{L}_{ITM}(\theta) = -\mathbb{E}_{(w,v)\sim D}f_\theta(v_m|v_{\m},w)
$$
1) Masked Region Feature Regression：学习将Transformer输出的遮掩区域$v_m^{(i)}$回归到视觉特征。具体来说，将Transformer输出经过fc再和输入进行比较。
2) Masked Region Classification：预测被遮掩区域的语义类别。将Transformer输出通过fc和softmax转换到正则化后的分布。由于没有真实标签，所以使用Faster-RCNN输出的类别作为label来进行预测。
3) Masked Region Classification with KL-Divergence：MRC使用的是硬标签，这个可能存在误导，因此改为采用Faster-RCNN的原始输出作为学习的目标，通过该方式将Faster-RCNN的知识蒸馏到UNITER里面。最后通过最小化KL散度来实现。


#### 3. 预训练数据集
