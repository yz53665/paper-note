## Adapter Tuning
谷歌设计的针对BERT的微调结构：
![[Pasted image 20240820222201.png](../../img/Pasted%20image%2020240820222201.png)
+ 在原有的Transformer结构Encoder的FF层后面加入，包含先降维，再升维的过程。
+ 为了避免最差情况，残差连接可以直接跳过中间的层

## prefix tuning
前人工作：人工设计template，自动化搜索离散的template，但该方法对于template特别敏感，容易造成性能不稳定。
![[Pasted image 20240820225435.png](../../img/Pasted%20image%2020240820225435.png)
特点：
+ 每层transformer前都要加
+ 输入token前构造任务相关的virtual token作为prefix
+ 和前人最大差异在于可学习的prefix是连续自由可调的
+ 对于encoder-decoder结构，需要分别加入prefix，分别引导模型去编码或是解码
+ 只训练prefix相关的参数
+ 为了防止直接训练导致的不稳定，在prefix前面加入了MLP结构
+ 优于adapter和finetune最上面两层，和全参数微调差不多
+ **适用于需要捕捉复杂上下文信息**

和prompt区别？
+ prompt是人为构造的显示表达，prefix是隐式表达

## prompt tuning
prefix tuning的简化版本，差异：
+ 前置于embedding输入，需要参数更少
+ 允许transformer更新中间层表示
+ **特别适用于任务复杂度较低或数据量较少的下游任务**

## P-Tuning
和prefix-tuning类似，认为离散的搜索prompt模板不适合连续表达的神经网络。于是放弃了自然语言表达的模板，改为连续参数的优化问题：
![[Pasted image 20240820230251.png](../../img/Pasted%20image%2020240820230251.png)
+ 左图只能使用离散的结果来更新prompt生成器，右边的模型通过伪标签可以输入连续的内容。
+ 具体做法是可以使用unseened token作为伪prompt用于训练更新

和prefix-tuning的区别：
+ 仅在输入加，而不是每层都加伪标签
+ 不一定是前缀，中间也可以

在训练时也不是直接初始化几个离散的值用于训练，这样容易陷入局部最优，而是用一个LSTM+MLP编码这些virtual token再输入模型

## LoRA
1. LoRA两个矩阵的初始化方式是什么？
    
    1. 第一个正态分布
        
    2. 第二个全零初始化
        
2. 为什么要这样初始化？
    
    1. 全零会导致梯度消失
        
    2. 全部用正态分布，有概率产生过大的梯度导致引入过多噪音，导致难以收敛

## QLoRA
结合模型量化和LoRA微调模型：
+ 4bit NormFloat量化
+ 双量化，对量化后常量再次量化(scale数据)
+ 提出新的数据类型NF4
+ ![[Pasted image 20240820233332.png](../../img/Pasted%20image%2020240820233332.png)
+ 使用分页方法管理控制内存峰值
![[Pasted image 20240820231935.png](../../img/Pasted%20image%2020240820231935.png)

### 1. 什么是量化
+ 将神经网络参数和激活值由float转换为int型，计算过程中再将int型表示用于计算浮点数：
![[Pasted image 20240820232236.png](../../img/Pasted%20image%2020240820232236.png)
+ 可量化对象：参数、激活值、梯度
+ 一般参数分布较稳定，是常用的量化对象（如QLoRA）
+ 激活值常常存在异常值，容易丢失精度，所以需要特殊处理
+ 大模型通常采用16-bit精度(FP16/BF16)计算，所以大模型运算时也通常将int转换为这类精度
### 2. 量化优点
+ 减少内存和显存占用
+ 提高运算速度（不一定，看情况）
	+ 部分适配低精度的硬件有加速，如可以用int8 GEMM kernel 计算
	+ 减少单位数据的bit数，降低IO量
+ 由于显存占用少了可以增大batch size
### 3. 量化方法
+ QAT量化后学习，在线量化
+ PTQ，使用时权重缩放
	+ 训练后动态量化，QLoRA，直接用量化公式转换
	+ 训练后校正量化，GPTQ，要重新微调
+ 