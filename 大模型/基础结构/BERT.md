### 特点
+ 采用MLM对双向Transformer进行预训练
+ 只需添加一个额外的输出层进行fine-tune，就可以在各种下游任务中取得SOTA表现

### 方法
如何判断时属于哪个句子：
+ 把分割token插入到每个句子后面，以分开不太的句子
+ 可学习的分割embedding来指示属于哪个句子
![[Pasted image 20240415211216.png](../attach/Pasted%20image%2020240415211216.png)

##### 预训练方法

###### 1. 随机遮掩词语预测MLM
###### 背景
Bi-RNN具有双向学习的能力，Transformer也可以构建双向语言模型，最简单就是两个Transformer从不同方向扫描输入序列。但该方式耗时、参数多。

###### 本文方法
通过随机15%token遮掩词语预测，迫使模型像双向模型一样思考
预测mask位置原有的单词，**但这会带来额外的对mask敏感问题**（下游任务中），所以进行如下改进：
+ 80%是mask
+ 10%是随机其它token
+ 10%是原来的token

模型输出是被遮掩的token序列的预测结果，如“我是中国人”，遮掩掉了国，模型就要输出“国”
###### 2. 是否下一句二分类NSP（Next Sentence Prediction）
判断句子B是否是A的下一句。迫使模型学习如何输出更好的句子表示

## 常见问答

为什么采用80%mask+10%其它token+10%原来token？
由于mask在下游微调任务中并不会出现，只使用mask会导致模型只对mask敏感。所以需要部分保持不变，部分随机成其它token