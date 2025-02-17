# RLHF

步骤1：收集示范数据（根据问题写示范答案），训练监督规则
+ 从prompt集合中采样prompt
+ 标注人员选择期望的输出行为
+ 微调GPT-3.5

步骤2：收集对比数据并且训练奖励模型
+ 采样一个prompt和几个模型输出
+ 标注人员对输出排序
+ 用排序数据训练奖励模型
![[Pasted image 20240906205250.png](image/Pasted%20image%2020240906205250.png)
步骤3：使用PPO强化学习算法
+ 从数据集中采样新的prompt
+ 从监督规则初始化PPO模型
+ 规则生成输出
+ 奖励模型针对输出计算奖励值
+ 奖励值使用PPO更新规则
![[Pasted image 20240906205439.png](image/Pasted%20image%2020240906205439.png)
名词解释：
+ policy是GPT输入文本后输出结果的过程
+ Action Space是词表在输出位置的排列
+ observation space是可能的输入token序列
+ Reward Function是基于上面两步得到的奖励模型

# PPO（ Proximal Policy Optimization）算法
核心：根据采取行动和收到的奖励，动态实时调整策略
[狗都能看懂的Proximal Policy Optimization(PPO)PPO算法详解_ppo算法csdn-CSDN博客](https://blog.csdn.net/weixin_42392454/article/details/140641453)
![[Pasted image 20240906211445.png](image/Pasted%20image%2020240906211445.png)
理解：分布调整的奖励函数+KL散度限制（不让模型偏离太远，获得近似的输出）

## 在NLP中实现
实际的奖励函数包含未来收益+当下实时收益
[图解大模型RLHF系列之：人人都能看懂的PPO原理与源码解读 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/677607581)
![[Pasted image 20240906212013.png](image/Pasted%20image%2020240906212013.png)
**在RLHF-PPO阶段，一共有四个主要模型**，分别是：  

- **Actor Model：演员模型**，这就是我们想要训练的目标语言模型
- **Critic Model：评论家模型**，它的作用是预估总收益 Vt，可以由Reward Model初始化得来，最后加了一层MLP将输出映射为一个value值 ![[Pasted image 20240906212552.png](image/Pasted%20image%2020240906212552.png)
- **Reward Model：奖励模型**，它的作用是计算即时收益 Rt，需要冻结
- **Reference Model：参考模型**，它的作用是在RLHF阶段给语言模型增加一些“约束”，防止语言模型训歪（朝不受控制的方向更新，效果可能越来越差，一般直接使用SFT模型初始化，需要冻结）

为什么Reward Model需要冻结：
+ 经过输入、收益对训练，本身输出的就是当前时刻的收益

为什么不直接使用Vt
+ Vt全靠预测
+ Rt更加接近实际的收益

### 损失函数计算

**目标模型的损失**
![[Pasted image 20240906214240.png](image/Pasted%20image%2020240906214240.png)
Adv_t是指当前优势（实际收益-预测收益，更好了要加大奖励力度）：
![[Pasted image 20240906214302.png](image/Pasted%20image%2020240906214302.png)
改进后引入未来项（计算时从后往前算，因为最后一个token的未来项为0，直接取R_T-V_T）：
![[Pasted image 20240906214346.png](image/Pasted%20image%2020240906214346.png)
R_t只考虑最后一个时刻的结果，其它时刻token只关注和ref模型的差异是否过大
![[Pasted image 20240906215414.png](image/Pasted%20image%2020240906215414.png)
由于计算经验消耗太大，所以每次大模型生成response，都会反过来更新多次大模型，每次更新时都会尽可能让模型的输出接近第一次产生response的结果，避免经验重复计算：
![[Pasted image 20240906215909.png](image/Pasted%20image%2020240906215909.png)
另外为了防止超出范围，会进行裁剪，限制范围两个P相除结果差异的范围，超出范围就不再更新

# DPO(Direct Preference Optimization)

无需强化学习的对齐方法
正例概率计算：

![[Pasted image 20240906235648.png](image/Pasted%20image%2020240906235648.png)

DPO本质是使得正例的期望最大化：
![[Pasted image 20240906235809.png](image/Pasted%20image%2020240906235809.png)

![[Pasted image 20240906235924.png](image/Pasted%20image%2020240906235924.png)
即将学习得到的奖励替换为正例的概率
