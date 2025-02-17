RMS（_Root Mean Square Layer Normalization_）均方根层归一化

移除了均值计算，简化了层归一化的计算量。

原始Norm：
![[Pasted image 20240821000313.png](../../img/Pasted%20image%2020240821000313.png)
RMSNorm认为每次都要计算均值方差过于复杂， 于是移除了均值计算和可训练偏移部分：
![[Pasted image 20240821000405.png](../../img/Pasted%20image%2020240821000405.png)
