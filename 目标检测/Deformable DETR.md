### DETR存在问题
![[Pasted image 20240417231806.png](attach/Pasted%20image%2020240417231806.png)
+ 计算复杂度限制，不能用大分辨率特征图，小目标性能差
+ 注意力权重矩阵往往稀疏，计算全部注意力导致收敛速率慢

## 本文思想
使用参考点附近采样少数点作为注意力的k
![[Pasted image 20240417232113.png](attach/Pasted%20image%2020240417232113.png)
## 方法
1. 使用网格作为reference point
2. 用FC计算偏移量
3. 针对采样点计算注意力