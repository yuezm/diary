# July-1

## LeetCode

### 数组中第 k 个最大元素

思路 1：API 排序获取
思路 2：快排，快排的原理是，找到第 i 个位置，第[0,i-1] 比 第个小，第[i+1,maxLength-1] 比第 i 个大，如果此时 i 的位置，正好是 k 的位置，则 nums[i]为目标数
思路 3：基于堆排序

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

### 剑指 Offer 09. 用两个栈实现队列

思路：一个栈存储存储出栈的数据，记为栈1；另一个栈存储入栈的数据，记为栈2。当栈1为空是，从栈2移动数据

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

### 有序矩阵中第 K 小的元素
