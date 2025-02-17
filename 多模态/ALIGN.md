《Scaling Up Visual and Vision-Language Representation Learning With Noisy Text Supervision》
### 特点
+ 相比CLIP可以实现更多功能，可以用于图文联合输入任务，图片召回文本或文本召回图片
+ 相比CLIP最根本差异在于，CLIP根据wiki调整了图文对，本文直接使用正常自然分布的图文对
+ 可以进行更加细致的定制化处理，比如去掉什么东西、加上什么东西

### 方法
![[Pasted image 20240414101312.png](../img/Pasted%20image%2020240414101312.png)
#### 1. 大规模噪音图-文数据集
+ 18亿大小
+ 简单的图片过滤：移除色情图片、保留最短边大于200像素
+ 移除同一文本由超过10张图片共享的图文对、稀有描述图片、太短或太长的描述图片

#### 2. 预训练
+ 使用带有global poolin的EfficientNet作为图像编码器
+ 使用BERT作为文字编码器
+ 使用normalized softmax loss来训练
+ 将匹配的图文对作为positive，不匹配的作为negative
+ 最小化两个loss
图-文分类：
$$
\mathcal{L}_{i2t} = -\frac{1}{N}\sum^N_i\log \frac{\exp(x_i^Ty_i/\sigma)}{\sum_{j=1}^N\exp(x_i^Ty_j/\sigma)}
$$
文-图分类：
$$
mathcal{L}_{i2t} = -\frac{1}{N}\sum^N_i\log \frac{\exp(x_i^Ty_i/\sigma)}{\sum_{j=1}^N\exp(x_i^Ty_j/\sigma)}

$$
这里的x和y分别指代图

#### 3. 迁移至图-文匹配与召回
#### 4. 迁移至视觉分类

