+ 应用场景：对比学习
+ 目标：匹配时loss值低，不匹配loss值高

### 背景
softmax：
![[Pasted image 20240507203403.png](attach/Pasted%20image%2020240507203403.png)
组合交叉熵损失表示类别匹配：
![[Pasted image 20240507203459.png](attach/Pasted%20image%2020240507203459.png)
存在问题：
+ 如ImageNet数据集，每个图片一类，softmax运算量太大
### NCE
NCE（noise contrastive estimation）讲类别转换为匹配类和噪声类，非匹配为负样本，对其采样计算loss
+ 应用场景：NLP类模型
![[Pasted image 20240507203929.png](attach/Pasted%20image%2020240507203929.png)
存在问题：
+ 很多负样本直接差别较大，不利于模型学习
### Info NCE
![[Pasted image 20240507204504.png](attach/Pasted%20image%2020240507204504.png)
思想：将k个采样的负样本看作k个类别

### 温度系数作用
+ 增大：经过指数运算都变得很小，logits分布更加平滑；样本区分不大
+ 减小：指数运算后都变小，分布更加集中；重点关注困难样本，模型难以收敛或者泛化差（负样本可能和正样本差距小，强行分开困难）
总结：控制对负样本区分度
