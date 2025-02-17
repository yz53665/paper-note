BLIP：一种基于预训练的方法，通过联合训练视觉和语言模型提升多模态任务性能
BLIP-2：提出了更加简洁的预训练方法，利用现有单模态视觉和文本预训练模型，减少计算成本，避免灾难性遗忘的问题

## 背景
自从CLIP后，视觉语言预训练模型（Vision-Language Pre-training，VLP）模型开始逐渐涌现。显著提高了各种视觉语言任务的性能。但主要存在如下问题：
+ 模型：大多基于编码器模型（encoder-based model）或编码器-解码器模型，前者无法生成文本，后者无法完成图像文本检索，难以兼顾
+ 数据：CLIP类模型从互联网上收集数据，带有噪音的文本作为监督信号存在问题

## BLIP目的
+ 兼顾图文多模态的理解和生成能力

## BLIP（Bootstrapping Language-Image Pretraining）

+ 引入跨模态编码器和解码器，实现跨模态信息流动
+ 引入CapFilt来去除噪音信号
+ Boostrapping是一种有放回重抽样的方法，每次有放回抽样（一个样本可能被抽到多次），用抽样样本进行训练，重复多次
+ 本文的Boostrap是指增加了一个在线数据打标签和清理的任务，把处理好的数据继续用于迭代模型

#### 模型结构
引入了编码器-解码器的多模态混合结构，可以作为：
+ 图像编码器：基于transformer的ViT架构，使用\[cls\]来表示全局图像特征。相当于CLIP中的Image Encoder
+ 文本编码器：基于BERT架构，将``[cls]``Token用于总结句子，相当于CLIP中的Text Encoder
+ 以图像为基础的文本编码器Image-grounded text encoder：在self-attn层后添加一个交叉attn来引入视觉信息，将``[Encode]``加入文本开头以表示特定任务。该模块用于提取文本特征并将其和图像特征对齐，相当于CLIP中更加精细化的Text-Image对齐
+ 以图像为基础的解码器：将Image-grounded text encoder的self-attn换成**causal self-attn**，将``Decode]``token和``[EOS]``token加入文本开头和结尾来表示开始和结束。用来生成符合图像和文本特征的文本描述，是CLIP中没有的。

> [!NOTE] 以图像为基础的解码器
> Image-grounded text decode生成的文本描述不同于原始的图像-文本对中的文本，原始的文本来自互联网，存在大量噪音；而该模块生成的描述更加精炼。是captioner的雏形，captioner也是基于训练完成的该模块输出进行微调。


需要注意的是，**图中相同颜色的模块都共用了参数**
![[Pasted image 20240417104634.png](attach/Pasted%20image%2020240417104634.png)
可以类比如下：
![[Pasted image 20240417104747.png](attach/Pasted%20image%2020240417104747.png)
通过2个理解任务，1个生成任务的损失函数联合进行预训练：
+ **图像-文本对比损失ITC**（Image-Texture Contrastive）：用于训练两个单模态编码器。最大化正样本相似度，最小化负样本相似度。**映射角度来看将相关图文映射到空间中的相邻点**，更加方便于学习。用了ALBEF中的动量编码器生成伪标签辅助训练。
+ **图像-文本匹配损失ITM**（Image-Texture Matching）：学习**联合表征**实现细粒度对齐。通过二分类任务，预测是正样本还是负样本。用了ALBEF中的hard negative mining更好捕捉负样本信息。

> [!NOTE] ITC和ITM的差异理解
> ITC虽然对齐了图文特征空间，但没有显式区分正负样本，主要是用于评价图文匹配情况，给出任意图文的相似度。比如军舰和海警船，都是船，相似度很高，但不匹配。而ITM则是计算样本损失来区分正负样本，激励正样本的图文使其更加匹配。所以说ITM是更加细粒度的对齐。

+ **语言建模损失LM**（Language-Modeling）：实现生成文本描述的任务。通过优化交叉熵损失函数，训练模型自回归的方式最大化文本概率。还使用了0.1的标签平滑损失。

##### CapFilt模块
BLIP用于训练的模型包含三部分样本来源：
+ COCO数据集上的高质量人标注图像-文本对样本
+ 过滤后的网络图-文对样本
+ 网络图-重新生成的文对样本

于是BLIP提出字幕和过滤机制CapFilt(Captioning and Filtering, CapFilt)处理网上获取图文对（存在信息不准和错误信息），包含如下两个子模块：
+ 字幕器captioner（本质基于Image-grounded text decoder）：一个视觉文本解码器，基于Image-grounded text decoder，用于生成给定图像的字母，使用LM损失函数在COCO数据集上进行微调。给定网络图片$I_w$，Captioner生成字幕$T_w$
+ 过滤器Filter（ 本质基于Image-grounded text encoder）：一个视觉文本编码器，用于去除文本噪声。使用ITC和ITM损失函数在COCO数据集上进行微调。Filter通过比对文本和图像的匹配情况，删除原始Web文本T_w和合成文本T_s中的噪声

![[Pasted image 20240416233737.png](attach/Pasted%20image%2020240416233737.png)
captioner和Filter都是从预训练模型初始化的，并在COCO人工数据集上单独进行微调。

##### BLIP训练流程
+ 使用含有噪音的网络数据训练一遍BLIP
+ 在COCO数据集上进行微调来训练Captioner和Filter
+ 使用Filter删除嘈杂字幕，使用Captioner生成字幕，得到干净数据
+ 使用干净数据训练一遍得到高性能BLIP

### 改进方向
+ 多轮数据集的Boostrapping
+ 为每一张图像生成多个合成字幕，进一步扩大训练语料库
+ 训练多个不同的Captioner和Filter进行集成

## BLIP-2

### 特点
+ 模态对齐（Q-Former）
+ 高效训练（二阶段训练范式）

目的：
+ 减少计算成本
+ 避免灾难性遗忘

方法：
+ 冻结预训练图像模型和语言模型

问题：
+ 视觉特征和文本特征难以对齐

解决方法：
+ 两阶段预训练Q-Former
	+ 表示学习阶段，通过轻量级Quarying Former缩小模态之间的gap，学习视觉语言表征
	+ 生成学习阶段，进行视觉到语言的文本生成学习

### 优点
+ 高效利用frozen预训练的视觉及语言模型
+ 可以根据提示zero-shot图像到文本生成
+ 相比起之前的SOTA，计算更加高效

![[Pasted image 20240417000125.png](attach/Pasted%20image%2020240417000125.png)
### 模型组成
+ 预训练的Image Encoder，文章对比了CLIP训练的ViT-L和EVA-CLIP训练的ViT-g
+ 预训练的LLM，文章对比了基于decoder的LLM和基于encoder-decoder的LLM
+ 可学习的Q-Former，由Image Transformer和Text Transformer组成，共享自注意力层
	+ Image Transformer 与图像编码器交互提取视觉特征。输入是可学习的Query，通过自注意力层互相交互，通过交叉注意力层与冻结的图像特征交互，通过共享自注意力层间接和文本交互
	+ Text Transformer 同时起到文本编码器和解码器作用，自注意力与Image Transormer共享，根据预训练任务，应用不同的自注意力掩码控制Query和文本的交互方式

### Q-Former结构
![[Pasted image 20240417170915.png](attach/Pasted%20image%2020240417170915.png)
有两个共享self-attention层的transformer子模块：
+ 图像Transformer，在Frozen Image Encoder基础上进行特征调整。Query通过selfattn再用crossattn和frozen交互

> [!NOTE] 为什么不直接用Frozen Encoder输入
> 可以用text来类比。text的transformer前面会加一个\[cls\]用于整合信息，最后取输出时只取该部分。而这里的Image Encoder的Query也是类似的处理，相当于定义了固定长度的Query输出用于整合来自可能不同长度的图片输入。

+ 文本Transformer，可以作为文本的编解码器

**初始化QFormer**：使用BERT来初始化，crossattn随机初始化
##### 1. 表示学习阶段

使用类似于BLIP的3个学习任务，共享输入格式及模型参数，每个学习任务通过不同的attention mask策略控制query和text之间的相互影响

###### 1.1 图文对比学习（ITC）
计算iamge-trans输出query表征和text-trans输出中\[cls\]的相似性，选取最大值作为图像文本对相似度。**为了防止信息泄漏，使用了单模态self-attn的mask，query和text互相不可见，防止直接从文本学习**。另外由于image Encoder被frozen，可以提前获取整个batch的负样本，而不用像BLIP中用队列一个个算

###### 1.2 基于图像文本生成（ITG）
根据输入图像训练QFormer训练文本。由于不允许image encoder于text token直接交互，文本生成所需要的图片信息通过query在cross-attention进行提取，再通过self-attention传递至text token。**作者使用多模态因果self-attention控制query-text交互，query无法获取text token，当前text token可以获取所有query及之前的text token，并且将\[CLS\]换成\[DEC\] token作为解码任务标志**。

###### 1.3 图文匹配(ITM)
**需要图文融合，使用bi-direction self-attention mask，所有query与text相互可见**，输出的query-embedding z捕获多模态信息，通过二类线性分类器取logit，其均值为匹配得分。作者使用难负样本挖掘策略创建负样本对。

> [!NOTE] 难负样本挖掘策略
> 当负样本图文对有相同语义但细粒度细节不同，那么就是难样本。通过对比相似度寻找batch内的hard negatives。对于每张图片，作者根据对比相似性分布从相同的batch中抽取一个负样本，与图像更加相似的文本有更高可能被采样。反过来，作者还为每个文本采样一个hard negative样本。


##### 2. 生成学习阶段
![[Pasted image 20240417195309.png](attach/Pasted%20image%2020240417195309.png)

### 模型预训练
+ 使用BLIP生成10个caption（使用CapFilt对网图生成caption）
+ 生成10个caption+原始web caption通过CLIP ViT-L/14模型与对应图像进行相似度排序
+ 选取top2作为该图的caption

