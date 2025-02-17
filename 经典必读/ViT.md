## 方法
![[Pasted image 20240423100930.png](./attach/Pasted%20image%2020240423100930.png)
将原始的图片$\rm{x}\in \mathbb{R}^{H \times W \times C}$ 转换为展平的2D切片$\rm{x}_p \in \mathbb{R}^{N \times (P^2\cdot C)}$,这里的(P,P)是切片的大小，$N = HW / P^2$，然后使用线性映射将其转换到D维度，再送入Transformer中，将该过程称为切片编码。
另外模仿BERT的做法，在第一位额外加了一个\[class]标签用于输出图片表达的结果，会被直接送入分类头。

由于没有观测到2D位置编码的显著优点，文章直接使用可学习的1D位置编码，并且在每个模块前都使用了Layernorm，整体流程如下：
$$
\rm{z}_0 = [x_{class}; x_p^1E; x_p^2E;\cdots; x_p^NE]+E_{pos},\ \ E\in \mathbb{R}^{(P^2\cdot C)\times D},\ E_{pos} \in \mathbb{R}^{(N+1) \times D}
$$
$$
z'_l = MSA(LN(z_{l-1}))+z_{l-1}, \ \ l= 1,\dots L
$$
$$
z'_l = MLP(LN(z_{l-1}))+z_{l-1}, \ \ l= 1,\dots L
$$
$$
\rm{y} = LN(z_L^0)
$$
#### 归纳偏差
在ViT中，只有MLP层是本地化和迁移不变的，而attn层是全局性的。另外缺乏2维的互相之间的关系

## 微调和更高的分辨率
在微调时将分类头替换为下游任务的类别数。
在更高分辨率下微调效果更好。当送入图片时，我们会保持patch大小不变，生成更长的序列。但预先训练的位置编码不一定有用，所以我们在之前编码基础上使用2D插值。

## 具体实现
16x16的patch大小