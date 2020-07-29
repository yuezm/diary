# July-1

## LeetCode

### 数组中第 k 个最大元素

1. 思路 1：API 排序获取
2. 思路 2：快排，快排的原理是，找到第 i 个位置，第[0,i-1] 比 第个小，第[i+1,maxLength-1] 比第 i 个大，如果此时 i 的位置，正好是 k 的位置，则 nums[i]为目标数
3. 思路 3：基于堆排序

```typescript
function findKthLargest(nums: number[], k: number): number {
  const indexTarget = nums.length - k;
  let res: number | undefined;

  fastSort(0, nums.length - 1);

  return res === undefined ? nums[nums.length - k] : res;

  function fastSort(i: number, j: number): void {
    if (i >= j) {
      return;
    }

    let target = nums[i];

    let leftI = i + 1,
      leftIndex = i,
      rightIndex = j;

    while (leftI <= rightIndex) {
      const s = nums[leftI];
      if (nums[leftI] <= target) {
        nums[leftI] = nums[leftIndex];
        nums[leftIndex] = s;

        leftIndex++;
        leftI++;
      } else {
        nums[leftI] = nums[rightIndex];
        nums[rightIndex] = s;

        rightIndex--;
      }
    }

    if (leftIndex === indexTarget) {
      res = target;
      return;
    }

    indexTarget < leftIndex
      ? fastSort(i, leftIndex - 1)
      : fastSort(leftIndex + 1, j);
  }
}
```

[数组中第 k 个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

### 剑指 Offer 09. 用两个栈实现队列

思路：一个栈存储存储出栈的数据，记为栈 1；另一个栈存储入栈的数据，记为栈 2。当栈 1 为空是，从栈 2 移动数据

```typescript
class Stack {
  private store: number[];
  constructor() {
    this.store = [];
  }

  isEmpty(): boolean {
    return this.store.length < 1;
  }

  push(v: number): void {
    this.store.push(v);
  }

  pop(): number | undefined {
    return this.store.pop();
  }
}

class CQueue {
  private stackPush: Stack;
  private stackPop: Stack;

  constructor() {
    this.stackPush = new Stack();
    this.stackPop = new Stack();
  }

  appendTail(value: number): void {
    this.stackPush.push(value);
  }

  deleteHead(): number {
    if (this.stackPop.isEmpty() && this.stackPush.isEmpty()) return -1;

    if (this.stackPop.isEmpty()) {
      while (!this.stackPush.isEmpty()) {
        this.stackPop.push(this.stackPush.pop()!);
      }
    }

    return this.stackPop.pop()!;
  }
}
```

[剑指 Offer 09. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

### 有序矩阵中第 K 小的元素

1. 思路 1：排序求解
2. 思路 2：排序优化，由于只需要直到第 k 小，且每个小数组都是有序的，可以使用归并排序+二分
3. 思路 3：二分查找，太难想了

```text
1. 确定矩阵最小值为 matrix[0][0]，记为 left
2. 确定矩阵最大值为 matrix[n-1][n-1]，记为 right

采用二分 mid = left + right >> 1

在矩阵寻找 小于等于 mid 的个数，即为 m
    如果 m < k，向右寻找
    如果 m >= k，向左寻找，请注意这里的二分法和普通而发有区别，如果直接取Mid，无法保证mid为矩阵的值
```

```typescript
function kthSmallest(matrix: number[][], k: number): number {
  return half(matrix[0][0], matrix[matrix.length - 1][matrix.length - 1]);

  function half(left: number, right: number): number {
    if (left >= right) return left;

    const mid = (left + right) >> 1;
    const m = calculateLte(mid);

    return m >= k ? half(left, mid) : half(mid + 1, right);
  }

  function calculateLte(num: number) {
    let sum = 0;

    let x = 0;
    let y = matrix.length - 1;

    while (x < matrix.length && y >= 0) {
      while (matrix[x][y] > num && y >= -1) {
        y--;
      }

      x++;
      sum += y + 1;
    }
    return sum;
  }
}
```

[有序矩阵中第 K 小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/)
