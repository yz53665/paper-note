
SwiGLU是由Swish激活函数和GLU门单元组合而成

## Swish激活函数（pytorch中叫SiLU）
$$
Swish(x) = xSigmoid(\beta x)
$$
图像如下：
![[Pasted image 20240821001235.png](../../img/Pasted%20image%2020240821001235.png)
优点：
+ 相比以往的RELU，Swish可以在0附近提供更加平滑的转换，这可以改善模型表现，缓解神经元死亡问题

### GLU

$$
GLU(x) = sigmoid(Wx+b)\odot (Vx+c)
$$
+ 用于捕获远程依赖关系，避免LSTM和GRU的梯度消失问题

## SwiGLU
$$
SwiGLU(x) = Swish(x)\odot (Vx)
$$
