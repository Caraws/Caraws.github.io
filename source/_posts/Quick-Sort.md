---
title: Quick Sort
date: 2017-10-17 21:42:33
tags:
- JavaScript
categories:
- 笔记📒
---

快速排序的实现, 参考资料[阮一峰](http://www.ruanyifeng.com/blog/2011/04/quicksort_in_javascript.html)

> ## 思路
快排法的思想， 分为以下三步
1.  在数据集中， 选择任意一个数据作为参照物（一般取中间位置的元素）。
2. 所有小于参照物的元素都放在左边，大于参照物的元素都放在右边。
3. 对左右两边的子集递归， 至到排序完成。

## 例子
- 非原地快速排序
```javascript
var array = [11, 65, 23, 108, 99, 11, 55, 11, 33, 100, 108, 100];

console.time('快排');
const quickSort = array => {
    // 数组只剩一个元素时停止
    if (array.length <= 1) return array;
    // 取参照物
    let pivotIndex = Math.floor(array.length / 2),
        // 这里用splice删除参照物避免重复循环
        pivot = array.splice(pivotIndex, 1),
        leftArr = [],
        rightArr = [];
    // 分区
    for (let i = 0; i < array.length; i++) {
        if (array[i] < pivot) {
            leftArr.push(array[i]);
        } else {
            rightArr.push(array[i]);
        }
    }
    return quickSort(leftArr).concat(pivot, quickSort(rightArr));
}
console.log(quickSort(array)); // [ 11, 11, 11, 23, 33, 55, 65, 99, 100, 100, 108, 108 ]

console.timeEnd('快排'); // 4ms 左右
```
- 原地快速排序
```javascript
console.time('快排2');
// 互换
const swap = (items, firstIndex, secondIndex) => {
    let temp = items[firstIndex];
    items[firstIndex] = items[secondIndex];
    items[secondIndex] = temp;
}
// 分区
const partition = (items, left, right) => {
    let pivot = items[Math.floor((right + left) / 2)],
        i = left,
        j = right;

    while (i <= j) {
        while (items[i] < pivot) {
            i++;
        }
        while (items[j] > pivot) {
            j--;
        }
        if (i <= j) {   
            swap(items, i, j);
            i++;
            j--;
        }
    }
    return i;
}
// 排序
const quickSortTwo = (items, left, right) => {
    let index;
    if (items.length > 1) {
        left = typeof left != "number" ? 0 : left;
        right = typeof right != "number" ? items.length - 1 : right;

        index = partition(items, left, right);

        if (left < index - 1) {
            quickSortTwo(items, left, index - 1);
        }

        if (index < right) {
            quickSortTwo(items, index, right);
        }
    }
    return items;
}

console.log(quickSortTwo(array));
console.timeEnd('快排2');
```

- 原生sort
```javascript
console.time('原生');
array.sort((a, b) => {
    if (a > b) return 1
    else if (a < b) return -1
    else return 0
});
console.log(array);
// 5ms 左右
console.timeEnd('原生')
```
三种方法， 当数组达到1W时差距就比较大了，原生需要17ms左右，原地快排需要13ms左右，而非原地快排则需要88ms左右。

Created on 2017-9-7 by Cara