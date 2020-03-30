# Bubble

#### 什么是冒泡排序?

> 冒泡排序（英語：Bubble Sort）又稱為泡式排序，是一種簡單的排序算法。它重複地走訪過要排序的數列，一次比較兩個元素，如果他們的順序錯誤就把他們交換過來。走訪數列的工作是重複地進行直到沒有再需要交換，也就是說該數列已經排序完成。這個算法的名字由來是因為越小的元素會經由交換慢慢「浮」到數列的頂端。---- wikipedia

#### Legand
![bubble-legand](./imgs/bubble.legand.gif)

#### Resolve Procedure

> 这里假设由小到大排序

- **step1**
  设 i = 0; j = i + 1; 比较 i 和 j 索引处的大小,若: - array[i] <= array[j]
  无操作 - array[i] > array[j]
  交换 i 和 j 索引处的元素

- **step2**
  i ++ 然后执行 **step1**

- **step3**
  循环**step2**直到 j == array.length - 1, 这样**step3**结束后就能确定一个第 n 大的值
  第一次循环确定了最大的值
  第二次循环确定了第二大的值
  .....

- **step4**
  循环**step3** array.length - 1 次
  结束

综上所述：冒泡排序就是相邻元素比较，交换寻找最大/小值的过程.

```JavaScript
function bubbleSort(array) {
	for (let i = 1; i < array.length; i += 1) { // 循环n - 1次 每次确定一个最大的值Max
		for (let j = 0; j < array.length - 1; j += 1) { // 循环n - 1次，确定最大值Max的过程
			if (array[j] > array[j + 1])
				[array[j], array[j + 1]] = [array[j + 1], array[j]];
		}
	}

	return array;
}
```

#### Optimization

已知冒泡排序随着外圈循环的次数,数组尾部逐渐有序。
这就意味着 👆 的代码在外/内圈循环的终止条件不够精确，浪费了额外的时间在无用的比较上.

- **step1**
  使用 lastIndex 来记录内圈循环最后一次交换的值.
  这表示数组 array 区间[lastIndex + 1, array.length - 1]是有序的.

- **step2**
  更新内/外圈终止条件 - **外圈**
  lastIndex + 1; - **内圈**
  lastIndex;

```JavaScript
function bubbleSort(array) {
  let needSortCount = array.length;

  while (needSortCount > 1) {
		// 这里是为了处理有序数组,退出循环
    let lastIndex = 0;

		for (let j = 0; j < needSortCount - 1; j += 1) {
      if (array[j] > array[j + 1]) {
        [array[j], array[j + 1]] = [array[j + 1], array[j]];
        lastIndex = j;
      }
    }

    needSortCount = lastIndex + 1;
  }

  return array;
}
```

#### 地址
[冒泡排序实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Bubble/v1.js)
[冒泡排序优化后的实现](https://github.com/thaloy/Algorithm/blob/master/Sort/Bubble/v2.js)
