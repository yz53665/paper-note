1. 训练效率
2. 工程实现优势
3. Encoder双向注意力存在低秩的问题


|       | Encoder | Decoder | 是否共享参数 |
| ----- | ------- | ------- | ------ |
| GPT   | 单向      | 单向      | 是      |
| UniLM | 双向      | 单向      | 是      |
| T5    | 双向      | 单向      | 否      |

## 1. 训练效率

## 2. 工程实现优势

## 3. 低秩问题
+ Attention矩阵组成：低秩分解所得，$n\times d$和$d\times n$，而这里的n远远大于d
+ Decoder-only的矩阵是一个下三角阵，由于softmax存在一定是满秩的
+ 满秩的表达能力理论上更强
+ 侧面印证，去掉可以升秩的softmax后性能下降：https://kexue.fm/archives/8338
