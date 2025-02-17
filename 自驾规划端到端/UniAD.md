## Abstract
+ 传统模块化模型可能受累积误差或任务协调不足影响
+ 理想框架应该追求最终的规划。本文设计架构，通过统一的查询接口通信，促进互相规划

## Intro
+ 受芯片带宽限制，自驾普遍设计独立模块，可能导致
	+ 模型间信息损失
	+ 误差累积
	+ 特征不对齐
+ 多任务
	+ 可能导致负迁移问题，即盲目跨任务知识共享和迁移迫使不相关任务产生不当知识迁移，导致过拟合
+ 直接预测规划轨迹
	+ 没有安全保证
	+ 缺乏可解释性

## Methodology
![[Pasted image 20240404191222.png](attach/Pasted%20image%2020240404191222.png)
### 1. 感知：追踪和建图
### 2. 预测
#### MotionFormer
从TrackFormer中接收的动态个体特征QA和MapFormer的静态信息MapFormer，基于场景中心方法预测所有个体轨迹。
包含N层，每一层捕获3类交互：个体-个体、个体-图、个体-目标
其它个体Q_A，地图元素Q_M
$$
Q_{a/m} = MHCA(MHSA(Q), Q_A/Q_M)
$$
MHCA多头交叉注意力，MHSA多头自注意力
使用DeformAttn表达个体和目标之间的attention：
$$
Q_g = DeformAttn(Q,\hat{x}_T^{l-1}, B)
$$
需要注意的是该部分有多层，这里的$\hat{x}_T^{l-1}$是上一层预测轨迹的终点，这里使得模型可以关注终点附近的环境信息$B$。

#### MotionQueries
每一层的输入Q包含两个部分，上一层的解码后的输出$Q_{ctx}$，位置特征$Q_{pos}$:
$$
Q_{pos} = MLP(PE(I^s)) + MLP(PE(I^\alpha)) + MLP(PE(\hat{x}_0)) + MLP(PE(\hat{x}_T^{l-1}))
$$
这里的$x_0$表示当前位置，$I^s$是场景anchor，是对先前运动的全局统计信息；$I^\alpha$是个体anchor，是个体的意图信息，所有anchor均由K-means聚类得到。

#### 非线性优化
直接使用上游探测到的不真实轨迹对下一步预测回归可能带来额外的误差和不符合物理学的预测，因此本文使用非线性平滑方法来调整需要预测的真实轨迹：
$$
\tilde{x}^* = \arg\min_{\rm{x}} c(\rm{x}, \tilde{x})
$$
$$
c(\rm{x}, \tilde{x}) = \lambda_{xy}\Vert x, \tilde{x}\Vert_2 + \lambda_{goal}\Vert x_T, \tilde{x}_T \Vert_2 + \sum_{\phi\in\Phi}\phi(x)
$$
$\tilde{x}^*$是平滑轨迹，$\tilde{x}$是真实轨迹。前两项是对平滑轨迹的整体和终点误差进行限制，最后一项是物理学限制，包括加加速度、曲率、曲率率、加速度和横向加速度。该优化只在训练过程中进行，不会对推理过程产生影响。

### 3. 占用网格预测（Occupancy Prediction）
OccFormer输入长度T_o的序列块，每块由个体特征G^t和来自上一层的状态特征$F^{t-1}$组成，并生成下一帧t的状态特征。其中$G^t$包含两个部分，在来自MotionFormer层特征的多模态维度最大池化后的特征$Q_x\in\mathbb{R}^{N_a\times D}$，这里的D是特征维度。
然后和当前位置编码$P_A$，来自上游的$Q_A$通过MLP融合
$$
G^t = MLP_t([Q_A, P_A, Q_x]),\ t=1, \cdots T_o
$$
另外将BEV场景特征B进行下采样为原来的1/4作为$F^0$。为了节约训练内存占用，所有块都采用了降采样-上采样的方法。中间插入了attention机制来建模像素级别个体间交互，记为$F_{ds}^t$
#### 像素级别个体（Pixel-agent）交互
使用$F_{ds}^t$作为Q，个体级别特征作为K和V来更新特征。$F_{ds}^t$首先通过子注意力机制来建模不同网格间的关系，然后使用交叉注意力机制建模个体特征和网格特征之间的关系。
另外为了对齐像素和个体之间的关系，限制每个像素只能查看在时刻t占用它的个体信息：
$$
D^t_{ds} = MHCA(MHSA(F_{ds}^t), G^t,attn_mask=O^t_m)
$$
$$
F^t = D^t_{ds} + F^{t-1}
$$
#### 实例级别（Instance-level）网格占用
表达每个个体的网格占用。$F^t$被卷积上采样为$F_{dec}^t$，掩膜特征$U^t=MLP(G^t)$，最后实例级别的网格占用表达如下：
$$
\hat{O}_A^t = U^t\cdot F^t_{dec}
$$
### 4. 规划
将原始的指引信号转换为可学习的表达——指令表达。来自MotionFormer的ego表达已经包含多模态意图，所以可以直接搭配指令表达进行查询。文章还加入了BEV场景信息来感知环境信息，之后再解码得到路径点$\hat{\tau}$。
为了进一步防止规划轨迹发生碰撞，文章在推理时基于牛顿方法优化原始轨迹:
$$
\tau^* = \arg\min_{\tau}f(\tau, \hat{\tau}, \hat{O})
$$
$\hat{\tau}$是原始规划预测，$\tau^*$是优化规划，是从Multiple-shooting轨迹$\tau$中根据最小化费用函数$f()$所得。$\hat{O}$是标准二值化占用网格图来自实例级别的占用网格预测。费用函数计算：
$$
f(\tau, \hat{tau}, \hat{O}) = \lambda_{coord}\Vert\tau, \hat{\tau}\Vert_2 + \lambda_{obs}\sum_t\mathcal{D}(\tau_t,\hat{O}^t)
$$
$$
\mathcal{D}(\tau_t,\hat{O}^t) = \sum_{(x,y)\in S}\frac{1}{\sigma\sqrt{2\pi}} \exp(-\frac{\Vert\tau_t-(x,y)\Vert_2^2}{2\sigma^2})
$$
t是未来的时间长度，第一项l2损失是使得优化轨迹尽可能靠近模型预测轨迹，后一项是防止和占用网格碰撞的惩罚项，这里的(x,y)只考虑相近的位置。
### 5. 训练
训练分为两个阶段，第一阶段先训练6个epoch的感知模块，然后完整训练整个网络20个epoch

**shared matching?**
不断使用引入的真实坐标进行匹配？