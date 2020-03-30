# SelectSort (选择排序)

> 选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。-------wikipedia

选择排序是依次找到最小值位置，然后将其置换到相应位置的过程.

#### Legand
![select-legand](./imgs/select.legand.gif)

#### Resolve Procedure

- **step1(minValue, minValueIndex, i)**
	从i + 1开始循环到数组结束	
	设 j = i + 1;
	若:
	- **array[j] < minValue**
		minValue = array[j]
		minValueIndex = j;
		j += 1;
	- **array[j] >= minValue**
		j += 1;	

- **step2(i)**
	minValueIndex = i;
	minValue = array[i];
	交换由**step1(minValue, minValueIndex, i)**计算出的minValueIndex和i处索引元素的位置

- **step3**
	循环执行**step(i)**, i ∈ [0, array.length -1];
	每次循环都能确定索引i的元素

#### Code

```JavaScript
function selectSort(array) {
	for (let i = 0; i < array.length; i += 1) {
		let minValue = array[i],
			minValueIndex = i;

		for (let j = i + 1; j < array.length; j += 1) {
			if (minValue > array[j]) {
				minValue = array[j];
				minValueIndex = i;
			}
		}

		[array[minValueIndex], array[i]] = [array[i], array[minValueIndex]];
	}

	return array;
}
```

#### Git 地址
[选择排序实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Select/index.js)
