# 提出解决思路

训练解码器，高效编码模型
# 文章贡献

# 模型

## 1. 将原始数据转换为prompt
将场景中所有行人的数据用prompt进行描述，再进行问询

## 2. 语言模型适应
重新训练了一个新的tokenizer（如何与大模型适配？）

提出5个辅助任务：重点建议、移动方向预测、相似模式搜索、组成员预测、碰撞可能检测

多模态轨迹预测：使用beam search

## 3. 使用大语言模型预测
### 3.1 zero-shot 预测（LMTraj-ZERO）
1. 输入prompts指导模型
2. 输出转换回数值坐标

### 3.2 有监督预测（LMTraj-SUP）
1. 选用T5模型全量微调


# 实验验证

### 1. tokenizer效果验证
覆盖率和编码效果

### 2. zero-shot效果分析
![[Pasted image 20240604195726.png](../img/Pasted%20image%2020240604195726.png)

### 3. 有监督效果分析

### 4. 消融实验——数字tokenizer编码器的有效性
分别使用原始的和新训练的tokenizer在数据集上进行对比实验
![[Pasted image 20240604200128.png](../img/Pasted%20image%2020240604200128.png)

### 5. 计算消耗
运算时GPU占用、耗时

# 总结与讨论
