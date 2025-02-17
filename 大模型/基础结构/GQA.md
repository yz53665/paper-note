GQA(Group Query Attention)是对MHA（多头注意力）和MQA（多query注意力）的平衡
![[Pasted image 20240821003047.png](../../img/Pasted%20image%2020240821003047.png)
实现了查询头的分组。GQA将查询头分成了G个组，每个组共享一个公共的键（K）和值（V）投影。
## 优点
可以减少存储每个头的键和值的开销(为KV cache考虑)