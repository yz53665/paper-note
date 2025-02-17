支持向量机，间隔最大线性分类器，通过选取核，可以实现非线性分类器

##### 基本思想
求解能正确划分数据集并使得几何间隔最大的分离超平面。
![[Pasted image 20240827222559.png](image/Pasted%20image%2020240827222559.png)
几何间隔计算公式：
![\gamma _i=y_i\left( \frac{\boldsymbol{w}}{\lVert \boldsymbol{w} \rVert}\cdot \boldsymbol{x}_{\boldsymbol{i}}+\frac{b}{\lVert \boldsymbol{w} \rVert} \right)](https://www.zhihu.com/equation?tex=+%5Cgamma+_i%3Dy_i%5Cleft%28+%5Cfrac%7B%5Cboldsymbol%7Bw%7D%7D%7B%5ClVert+%5Cboldsymbol%7Bw%7D+%5CrVert%7D%5Ccdot+%5Cboldsymbol%7Bx%7D_%7B%5Cboldsymbol%7Bi%7D%7D%2B%5Cfrac%7Bb%7D%7B%5ClVert+%5Cboldsymbol%7Bw%7D+%5CrVert%7D+%5Cright%29+&consumer=ZHI_MENG)
这里为点到直线距离计算公式，带正负号，所以需要额外乘上标签y
![\gamma =\underset{i=1,2...,N}{\min}\gamma _i](https://www.zhihu.com/equation?tex=+%5Cgamma+%3D%5Cunderset%7Bi%3D1%2C2...%2CN%7D%7B%5Cmin%7D%5Cgamma+_i+&consumer=ZHI_MENG)

则约束方程为：
![\underset{\boldsymbol{w,}b}{\max}\ \gamma](https://www.zhihu.com/equation?tex=+%5Cunderset%7B%5Cboldsymbol%7Bw%2C%7Db%7D%7B%5Cmax%7D%5C+%5Cgamma+&consumer=ZHI_MENG)

![s.t.\ \ \ y_i\left( \frac{\boldsymbol{w}}{\lVert \boldsymbol{w} \rVert}\cdot \boldsymbol{x}_{\boldsymbol{i}}+\frac{b}{\lVert \boldsymbol{w} \rVert} \right) \ge \gamma \ ,i=1,2,...,N](https://www.zhihu.com/equation?tex=+s.t.%5C+%5C+%5C+y_i%5Cleft%28+%5Cfrac%7B%5Cboldsymbol%7Bw%7D%7D%7B%5ClVert+%5Cboldsymbol%7Bw%7D+%5CrVert%7D%5Ccdot+%5Cboldsymbol%7Bx%7D_%7B%5Cboldsymbol%7Bi%7D%7D%2B%5Cfrac%7Bb%7D%7B%5ClVert+%5Cboldsymbol%7Bw%7D+%5CrVert%7D+%5Cright%29+%5Cge+%5Cgamma+%5C+%2Ci%3D1%2C2%2C...%2CN+&consumer=ZHI_MENG)

该方程可以使用拉格朗日乘子法得到其对偶问题

为什么要转换为对偶问题？
原有拉格朗日方程需要先求解最外层的min条件下的w和b，再求解内层的alpha，求解过程困难
##### 核函数
转换为它的一个对偶问题后，替换里面的一个内积项