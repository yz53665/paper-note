
# DALL-E
![[Pasted image 20240417222943.png](attach/Pasted%20image%2020240417222943.png)

#### 核心思想
+ 把文本token和图像token当成一个数据序列，通过Transformer进行自回归
+ 避免图像分辨率太大，引入dVAE降低图片分辨率

### 整体流程
+ 训练一个dVAE把每张256x256的RGB图片压缩成32x32的图片token，最后得到一个32x32x8192维度大小的logits
+ 用BPE Encoder 对文本进行编码，得到最多256个文本token
+ 然后与1024图像token拼接，得到长度1280的数据
+ 送入Transformer中进行自回归训练
+ **推理阶段**：给定图文，用transformer融合后的token，用dVAE的decoder生成图片。最后用训练好的CLIP得到不同采样图片的分数排序

### dVAE模块
VAE结构：
![[Pasted image 20240417223625.png](attach/Pasted%20image%2020240417223625.png)
VQVAE结构：
![[Pasted image 20240417223651.png](attach/Pasted%20image%2020240417223651.png)

dVAE和VQVAE主要差别：
+ 类似VQVAE，dVAE将图像映射到8192的词表中，由于离散不可导，使用softmax来近似max求导
+ 重建图像时，真实的像素值