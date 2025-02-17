# 1998 LeNet
结构
2X（卷积+降采样）+2（全连接）+1（高斯连接）
![[Pasted image 20240827215445.png](image/Pasted%20image%2020240827215445.png)
# AlexNet
结构：
5个卷积（+池化）+3个全连接
![[Pasted image 20240827215704.png](image/Pasted%20image%2020240827215704.png)
创新：
+ 引入ReLU、局部归一化、Overlap
+ 为了减轻过拟合：1. 平移反转；2. 改变RGB通道强度；3. Dropout；4. LR减小
# GooogleNet（Inception）

### v1

核心思想：将全连接替换为稀疏连接，为了防止稀疏计算的低效，将稀疏矩阵聚类为密集子矩阵
![[Pasted image 20240827220243.png](image/Pasted%20image%2020240827220243.png)
优点：
1. 多尺度特征提取，特征更加丰富
2. 稀疏矩阵分解为密集矩阵，加快运算
1x1卷积核作用：
1. 在相同大小上叠加更多卷积核，丰富模型学习到的特征
2. 对模型降维，减少运算量

### v2
加入BN层，并使用2个3X3卷积核替换1个5x5卷积核
优点：
1. 减小内部协方差偏移，将输出规范化到标准正态分布，增加模型鲁棒性
2. 3x3卷积核增加网络深度，但也增加了运算量和权重参数量

### v3
1. 提出卷积分解
2. 增加网络深度

### v4
1. 利用残差连接改进V3结构

# 2015 VGG
在图像识别上略差于Inception，但在很多诸如图像转换问题上的表现很好。
和AlexNet很相似，不同的是kernel大小和stride

# ResNet
前人问题：
+ 网络深度加深，精度下降但并非由于过拟合造成的（过拟合在训练集上精度应该很高）
两种残差结构：
![[Pasted image 20240827221839.png](image/Pasted%20image%2020240827221839.png)

# MobileNet

##### 深度可分离卷积

普通卷积：
![[Pasted image 20240416100923.png](attach/Pasted%20image%2020240416100923.png)

深度卷积：
![[Pasted image 20240416100958.png](attach/Pasted%20image%2020240416100958.png)
逐点卷积：
![[Pasted image 20240416101009.png](attach/Pasted%20image%2020240416101009.png)
效果对比：
![[Pasted image 20240416101056.png](attach/Pasted%20image%2020240416101056.png)

普通卷积运算量：
![[Pasted image 20240416101151.png](attach/Pasted%20image%2020240416101151.png)

深度可分离卷积运算量
![[Pasted image 20240416101214.png](attach/Pasted%20image%2020240416101214.png)

相当于把融合过程的