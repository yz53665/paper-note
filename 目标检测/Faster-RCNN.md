完整结构：
![[Pasted image 20240401100833.png](attach/Pasted%20image%2020240401100833.png)
1. conv layers，模型使用基础的conv+relu+pooling层提取image的特征图
2. Region Proposal Networks。通过softmax判断anchors属于positive或者negative，再利用bounding box regression 修正anchors获得精确的proposals
3. Roi Pooling。收集输入的feature maps和proposals，综合信息后提取proposal feature maps，送入后续判断目标类别
4. Classification。利用paoposal feature maps计算proposal的类别，再次bounding box regression获得检测框最终精确位置。
![[Pasted image 20240401211135.png](attach/Pasted%20image%2020240401211135.png)

## 1. Conv layers
以VGG16为backbone的网络为例，Conv layers有13层卷积，13个relu层，4个pooling层，并且：
1. conv：keynel_size=3， pad=1，stride=1，保证输出尺寸不变
2. pooling：kernel_size=2, pad = 0, stride=2，保证输出尺寸为原来一半

## 2. Region Proposal Networks(RPN)
经典检测方法：
+ Opencv adaboost 滑动窗口+图像金字塔
+ R-CNN使用SS（Selective Search）生成检测框
本模型使用RPN生成，极大提升速度：
![[Pasted image 20240401211317.png](attach/Pasted%20image%2020240401211317.png)
分支1用于获得positive和negative的分类
分支2用于计算相较于anchors的bounding box的regression偏移量

最后的Proposal负责综合positive anchors和对应的bounding box regression偏移量获取proposals，同时剔除太小和超出便捷的proposals。

#### 2.1 多通道图像卷积基础
![[Pasted image 20240401212746.png](attach/Pasted%20image%2020240401212746.png)
多通道卷积如图所示，