---
title: js中关于数组的算法问题
date: 2019-03-17 21:00:00
author: Echo
tags:
  - Function
  - JavaScript
  - Algorithm
---

在日常开发过程中，关于数组的算法问题是我们最常遇到的，最近解决了几个我觉得比较有代表性的数组算法问题，在这里分享一下解决思路

<!-- more -->

## 问题一：两个数组的交集

给定两个数组，编写一个函数来计算它们的交集。

- 示例 1:
  输入: nums1 = [1,2,2,1], nums2 = [2,2]
  输出: [2,2]

- 示例 2:
  输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
  输出: [4,9]

说明：

- 输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
- 我们可以不考虑输出结果的顺序。

解题思路

由于数组内容的不定性，所以不能使用普通操作整数的方法来处理这个方法，由此可以使用 Map 结构处理
步骤如下

1. 判断两个数组的长度，用大的去循环小的
2. 定义一个 Map，用于将大数组转换为 Map，Map 的 key 为数组的值，value 为该值在数组中出现的次数
3. 循环大数组，判断元素在 Map 中是否已存在，如果已存在，value+1, 如果不存在则设 value 为 1
4. 循环小数组，如果小数组的值在 Map 中存在并且 value 不为 0，则将值 push 进一个新数组，同时将 Map 中的这个值的 value-1
5. 得到的小数组就是数组的交集。

代码实现如下：

```js
const intersect = (nums1, nums2) => {
  let nums3 = [];
  if (nums1.length > nums2.length) [nums1, nums2] = [nums2, nums1]; // 交换数组
  let map2 = new Map();
  for (let i = 0; i < nums2.length; i++) {
    if (map2.has(nums2[i])) {
      map2.set(nums2[i], map2.get(nums2[i]) + 1);
    } else {
      map2.set(nums2[i], 1);
    }
  }

  for (let j = 0; j < nums1.length; j++) {
    if (map2.has(nums1[j]) && map2.get(nums1[j]) > 0) {
      nums3.push(nums1[j]);
      map2.set(nums1[j], map2.get(nums1[j]) - 1);
    }
  }
  return nums3;
};
```

## 问题二：两数之和

给定一个整数数组 nums 和一个目标值 target，在该数组中找出和为目标值的那两个整数，并返回他们的数组下标。

假设每种输入只会对应一个答案，但是不能重复利用这个数组中同样的元素。

举个 🌰：
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以应该返回 [0, 1]

第一种暴力破解方法代码实现如下：

```js
const twoSum = (nums, target) => {
  let index = [];
  for (let i = nums.length - 1; i > 0; i--) {
    for (let j = i - 1; j >= 0; j--) {
      if (nums[j] + nums[i] == target) {
        index.push(j);
        index.push(i);
        return index;
      }
    }
  }
};
```

就是很简单的两重循环数组，但是这种方法的复杂度是 n 的平方，解答过程也不够优雅，所以想了半天可以用空间换时间的思路来解决。
解法二：

1. 定义一个新的数组或对象
2. 同样的循环原数组，在循环的内部将 目标值 - 当前循环的值 = 新数组在当前下标下的值
3. 在循环中加入前置条件：如果当前循环的值 等于 新数组下标为当前循环的值并且值不为空或 undefined 时，就可以得出结果

```js
const twoSum = (nums, target) => {
  let result = [];
  for (let i = 0; i < nums.length; i++) {
    if (typeof result[nums[i]] !== 'undefined') {
      return [result[nums[i]], i];
    }
    let data = target - nums[i];
    result[data] = i;
  }
};
```

解法三：
使用 map 结构，将 key 设置为差值（target - 当前值），value 为对应的当前值的下标
在循环数组的时候，先判断如果在 map 当中存在 key 为当前的值的 value，说明这个 key 就是我们当前循环的值的于目标值的差值，则可以直接 return
若不存在，则在 map 中 set（差值，当前值下标）

这种解法其实思路是第二种是一样的，只是使用的数据结构不同

```js
const twoSum = (nums, target) => {
  let map = new Map();
  for (let i = 0; i < nums.length; i++) {
    let x = target - nums[i];
    if (map.get(x) == undefined) {
      map.set(nums[i], i);
    } else {
      return [map.get(x), i];
    }
  }
};
```

## 问题三：有效数独矩阵

题目：
判断一个 9x9 的数独是否有效。只需要根据以下规则，验证已经填入的数字是否有效即可。

数字 1-9 在每一行只能出现一次。
数字 1-9 在每一列只能出现一次。
数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Sudoku-by-L2G-20050714.svg/250px-Sudoku-by-L2G-20050714.svg.png" width="300"/>

数独部分空格内已填入了数字，空白格用 '.' 表示。

注意：
一个有效的数独（部分已被填充）不一定是可解的。
只需要根据以上规则，验证已经填入的数字是否有效即可。
给定数独序列只包含数字 1-9 和字符 '.' 。
给定数独永远是 9x9 形式的。

思路：这个题目乍一看挺简单，暴力破解的话依次循环出 3 种条件下是否有重复的就好，所以最简单的第一种解法如下所示

```js
//   检查每一行
for (let arr of board) {
  let rowSet = new Set();
  for (let c of arr) {
    if (c !== '.' && rowSet.has(c)) {
      return false;
    } else {
      rowSet.add(c);
    }
  }
}

//   检查每一列
for (let i = 0; i < 9; i++) {
  let col = [];
  board.map(arr => {
    if (arr[i] !== '.') col.push(arr[i]);
  });
  let set = new Set(col);
  if (set.size !== col.length) return false;
}

//   检查3*3
for (let x = 0; x < 9; x += 3) {
  for (let y = 0; y < 9; y += 3) {
    let box = [];
    for (let a = x; a < 3 + x; a++) {
      for (let b = y; b < 3 + y; b++) {
        if (board[a][b] !== '.') box.push(board[a][b]);
      }
    }
    let set = new Set(box);
    if (set.size !== box.length) return false;
  }
}

return true;
```

以上方法虽然可以没有错误的判断这个数独矩阵是否有效，但是太过于粗暴，所以思考了很久之后我得出了下面一种解法，只需要一个双层循环就能得出答案，非常巧妙

基本思路如下：
在循环这个 9x9 的二维数组时，假设循环的行数组下标为 arr[i][j]，那么在循环行数组的同事，根据行列索引相反的规律，那么列数组可以通过 arr[j][i] 这种规律来判断进行 set 存值

比如说在循环到第一行的数组时，索引值依次为
```js
[0, 0],[0, 1],[0, 2],[0, 3] ... [0, 7],[0, 8]; // 行
```
那么这个时候我们就可以同时得出列数组的索引
```js
[0, 0],[1, 0],[2, 0],[3, 0] ... [7, 0],[8, 0]; // 行
```

同时，3x3 的数组也可以通过找到某种特定的与横向数组对应的规律来组合判断（过程比较复杂，自行推断）
```js
let x = parseInt(i / 3) * 3 + parseInt(j / 3);
let y = 3 * (i % 3) + (j % 3);
```
其中，数组的是否重复可以通过 set 结构来判断：
具体实现如下

```js
for (let i = 0; i < board.length; i++) {
  let set1 = new Set();
  let set2 = new Set();
  let set3 = new Set();
  for (let j = 0; j < board[i].length; j++) {

    //  3*3时要用到的公式
    let x = parseInt(i / 3) * 3 + parseInt(j / 3);
    let y = 3 * (i % 3) + (j % 3);

    // 行重复
    if (board[i][j] !== '.' && set1.has(board[i][j])) {
      return false;
    } else {
      set1.add(board[i][j]);
    }
    // 列重复
    if (board[j][i] !== '.' && set2.has(board[j][i])) {
      return false;
    } else {
      set2.add(board[j][i]);
    }
    // 3*3重复
    if (board[x][y] !== '.' && set3.has(board[x][y])) {
      return false;
    } else {
      set3.add(board[x][y]);
    }

    if (+i === 8 && +j === 8) {
      return true;
    }
  }
}
```

由上可以看出，解法二的算法复杂度相比较于第一种要大大减低，解决方法也是更加优雅～

这几个题目当中的解题思路个人觉得是比较典型的，充分利用了 map / set 等数据结构来解决，希望这些问题可以为之后开发过程中遇到的新问题有一点解决灵感～～
