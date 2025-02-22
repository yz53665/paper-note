
### 1. 全排序
使用快排，然后去前K个
时间复杂度：O(nlog(n))

### 2. 局部排序
直接用冒泡排序，对k个最大的进行冒泡
时间复杂度：O(nk)

### 3. 堆
1. 如果前k个也不需要排序，也就是不排序TopK
2. 用前K个元素构建小堆（子节点大于父节点），用于存储当前最大的k个元素。
3. 从k+1个元素开始扫描，如果大于堆顶，则替换掉堆顶，并调整堆。这样可以保证对内k个元素总是最大的k个元素
时间复杂度：O(nlog(k))

### 4. 随机选择
快速排序第一步是将所有大于key的放到右边，小于的放到左边。那么只需要找到第k个大的数字，就相当于找到了前k大的数
问题转换为找第k大的数字。

可以先划分一次，获取key的下标，再用key和k作对比
伪代码：

```
int RS(arr, low, high, k){

if(low== high) return arr[low];

i= partition(arr, low, high);

temp= i-low; //数组前半部分元素个数

if(temp>=k)

return RS(arr, low, i-1, k); //求前半部分第k大

else

return RS(arr, i+1, high, k-i); //求后半部分第k-i大

}
```
时间复杂度：O(n)