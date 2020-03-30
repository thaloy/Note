# MergeSort(归并排序)

> 归并排序（英语：Merge sort，或mergesort），是创建在归并操作上的一种有效的排序算法，效率为{\displaystyle O(n\log n)}{\displaystyle O(n\log n)}（大O符号）。1945年由约翰·冯·诺伊曼首次提出。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行 ---------wikipedia 
> 分治: 将难以直接解决的大规模问题,分解成一些小规模的相同问题,分而治之.	
> 
> 小规模问题如果被成功解决,那么它们间一定具有某种规则存在,找到这种规则是解决/优化问题的必然途径.

归并排序是分治思想在排序算法上的一种体现

### Legand
![merge-legand](./imgs/merge.legand.gif)

### Resolve Procedure

**step1:**
假设K为数组的分解粒度

	Array = K * SubArray 

**step2:**
将子数组依次分解直至不可再分.即:数组长度为1.

**step3:**
逆分解路径合并,并且每一次的合并伴随着排序操作.
每一次合并产生的子数组都是有序子数组(一般是自小到大).

**step4:**
step3得到的最终值就是需要的结果.

### Why?
已知冒泡排序的时间复杂度是O(n^2).
由指数函数曲线可知,数据规模n越大,排序算法的运行时间t越大,其增长率也越大.

为了减少这种恐怖的增长率,能想到的方法就是减小数据的规模.

通过分解成K组数据, 将O(n ^ 2)变成K * O((n / K)^2) + O(merge(n / K))
我们还可以对数据规模n更进一步分解,知道其不可分解.

所以选用适合的K成为了问题的关键.

### K = ?
已知社区内的归并排序实现大部分是基于K=2的.为什么呢？

我们假设数据规模为N, 每一次的分组为K
已知数据规模分解的过程是一个完全'K'叉树
现假设其是一个**满** **完全** 'K'叉树
设这颗数的高度为h,则有
$$
	K ^ h = N	
$$
$$
	h = \log_KN
$$

合并数组的过程就是在子数组中依次寻找最小值的过程.
因为在合并前,待合并的子数组就已经是有序的,所以只要比较子数组的首项元素的大小就可以完成这个过程.
设树的每层的比较次数为L,在不考虑某个子数组为空的情况下有
$$
	L = (k - 1) * (N - 1)
$$
K - 1 是因为如果有M个数字,找到其min/max值需比较的次数为M - 1
N - 1 是因为树的每层的数据的和为N,每次我们都要找最小值,所以会有N - 1次
所以有👆L的比较次数

⚠️  
需要注意的是上面是粗略的计算过程,因为在实际过程中一定会出现某个数组先为空的情况。所以K - 1 是个约值.
同时N-1的值也是约值. 真值为N - K ^ hth 其中hth为第几层.

所以总的时间复杂度为
$$
	T(N) = T(N, K) = L * h = (K - 1) * (N - 1) * \log_KN
$$
有
$$
	T(N) = 	(K - 1) * (N - 1) * lgN / lgK
$$
因为这里对K进行分析,所以忽略数据规模常数N
$$
	T(N) = T(K) = K - 1 / lgK
$$
对其求导得到有
$$
	(K * lgK - K + 1) / (K * lgK) ^ 2
$$
// TODO 这里图片
在[2, +∞)内恒大于0

所以总体上**K = 2**时, **时间复杂度**最小

tips:当K=N时,根据排序的策略其为选择排序.
其时间复杂度为
$$
	T(N) = \log_2N * N = NlogN
$$

#### The way of implement 
归并排序的实现主要有2种
	- **一般归并排序**
		空间复杂度O(N)
	- **原地归并排序**
		空间复杂度O(1)

其区别是,原地归并排序不需要借助额外的辅助空间,空间复杂度为O(1).

#### Part Code
由👆提到的内容,归并要求我们自上而下下分解子数组,然后在逆序分解过程进行排序合并操作
所以实现归并要依赖栈结构

👇的实现都依赖与函数调用栈 ---> 递归

- **一般归并**
```JavaScript
// 伪代码
function merge(leftArray, rightArray) {
	const sortArray = [];
	// 循环比较找到最小值	
	sortArray.push(min);

	return sortArray;
}

function mergeSort(array) {
	let middleIndex; // 这里使用中间索引将数组切分成2个数组
	let leftArray;
	let rightArray;

	if () return; // 递归出口 数组不可再分

	// 继续分解数组
	leftArray = mergeSort(leftArray);
	rightArray = mergeSort(rightArray);

	// 合并
	return merge(leftArray, rightArray);

	// 函数调用栈为
	[merge(lA, rA), merge(lAl, lAr), merge(rAl, rAr)];
}
```
并且因为merge的参数都是子merge的返回值,即: 
```JavaScript
	lA = merge(lAl, lAr);
	rA = merge(rAl, rAr);
```
每个merge函数会生成一个新的内存空间sortArray有
```JavaScript
sortArray.length = leftArray.length + rightArray.length;
```
伴随着每次merge函数完成参数leftArray和rightArray都会被JS的垃圾回收机制回收掉.
所以在函数执行的过程中恒有长度为N的数组内存空间作为辅助
即
$$
	S(N) = N
$$

- **原地归并**
原地归并不需要辅助空间, 这意味着在进行**merge**操作如何进行合并是至关重要的

```JavaScript
// 这里需要多传startIndex, middleIndex, endIndex通过他们来划分子数组
function mergeSort(array, startIndex, endIndex) {
	... 同上		

	mergeSort(array, startIndex, middleIndex);
	mergeSort(array, middleIndex + 1, endIndex);

	merge(array, startIndex, middleIndex, endIndex);
}

```

关于**merge**方法的实现
**将一般归并merge方法的数组push转换成数组移位的方式**
**step1**: 在leftArray中找到第一个大于rightArray[0]的值记为a, 其索引为il
**step2**: 在rightArray中将所有小于a的值依次插入到leftArray[il]后面 **这是个数组移位的过程**
**step3**: 反复step1~step2直至越界

具体如下
```
	i = startIndex
	m = middleIndex
	i         m  j         
	[1][1][3][5][2][5][7][10]
```
[1,1,3,5,2,5,7,10]这个数组被i,m,j分成了2个子数组
leftArray = array[i ,  m]
rightArray = array(m , j]
并且他们都是有顺序的
若想要对leftArray和rightArray原定合并
**step1**
在leftArray中找到第一个大于rightArray[0]的值
```
				i   m  j         
	[1][1][3][5][2][5][7][10]
```
这样array[0 , i) = [1, 1]就是一个已经排好顺序的子数组
**step2**
在rightArray中找到第一个大于array[i]的值
```
				i   m     j
	[1][1][3][5][2][5][7][10]
```
**step3**
这时我们将[3, 5, 2]左移2位或右移1后有
arr[0 , i + j - m - 1)也是一个已经排好顺序的子数组

```
				   i   m  j
	[1][1][2][3][5][5][7][10]
```
**step4**
重复1 ～ 3的过程,实现2个有序数组的原地合并.

#### Code
[归并排序实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Merge/merge.js)
[原地归并实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Merge/mergeInPlace.js)

#### 数组移位
// TODO 超链接

#### 参考
[C语言原地归并实现](https://blog.csdn.net/xiaolewennofollow/article/details/50896881)
