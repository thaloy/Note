# InsertSort (插入排序)

> 插入排序（英语：Insertion Sort）是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通常采用 in-place 排序（即只需用到{\displaystyle O(1)}{\displaystyle O(1)}的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。------ wikipedia

插入排序的过程就是摆放扑克牌的过程

#### Legand
![insert-legand](./imgs/insert.legand.gif)

#### Resolve Procedure

- **step1(i)**
	设j = i;
	比较array[j]和array[j - 1]。若：
	-	**array[j - 1] > array[j]**
		交换索引i和j的元素位置
		j -= 1;
	- **array[j - 1] <= array[j]**
		退出循环**step2**	

- **step2(i)**
	循环**step1(i)**，直到
	- **触发了step1的退出条件**
	- **j越界(j < 1)**
		因为i最小值是1

	每次step2执行完,都能确定i在数组中的位置。即:[0, i]是有序的

- **step3**
	循环**step2(i)**,其中i ∈ [1, array.length - 1];
	i 表示为array[i]需要被插入到有序数组中

	Q: i 为什么从 1 开始？
	A: 这是因为插入排序是交换排序,所以其首位置在后面的比较/交换中就能确定位置.
		 如: [3, 2]
		 只需要确定2的位置，3的位置也就确定了.
				
#### Code
```JavaScript
function insertSort(array) {
	for (let i = 1; i < array.length; i += 1) {
		let j = i;
			
		while(j > 0 && array[j - 1] > array[j]) {
			[array[j], array[j - 1]] = [array[j - 1], array[j]];
			j -= 1;
		}	
	}

	return array;
}
```

#### Git地址
[插入排序实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Insert/index.js)
