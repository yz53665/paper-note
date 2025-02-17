![[Pasted image 20240812131838.png](../img/Pasted%20image%2020240812131838.png)

### Llama发展

#### Llama1
相比GPT，在更长的，超过1T token的语料上进行预训练

#### Llama2
训练语料扩充到2T token，引入了分组查询注意力机制（GQA）
进一步通过有监督微调（SFT），基于人类反馈强化学习（RLHF）对模型进行迭代优化，得到Llama2-chat

#### Llama3
支持8k长文本，扩充了词表，超过15T token的训练语料

### Llama模型架构
和GPT主要差异：
+ 为了增强训练稳定性，使用了前置RMSNorm用于层归一化
+ 为了提高模型性能，采用SwiGLU作为激活函数
+ 为了更好建模长序列，使用RoPE作为位置编码
+ 为了平衡效率和性能，采用分组查询注意力（GQA）

