# 9 月第二周

## 深入理解计算机系统

### 数字存储

#### 整数存储

整型一般分为有符号整型和无符号整型，一般分为 `char、short、int、long、long long`，下面就拿 int 来举例

1. 有符号 int

在 MAC 10.14.6 中来说，int 为 4 字节，32 位。则第 32 位为符号位，第 31 位~第 1 位表示数的值，所以 int 的取值范围为 [-2^31, 2^31-1]

2. 无符号 int

无符号整数和有符号整数差别不太大，只是第 32 位不表示符号，也是表示数的值，则无符号 int 的取值范围 [0, 2^32-1]

#### 整型计算

1. 有符号整数计算

   对于负数来说，存储时并不是按照`原码`来存储的，而是使用`补码`。

   补码：原码取反后加 1。对于负数，基本步骤是，先取负数的绝对值，转换为二进制，取反，取反后将数加 1

   计算时，按照位对齐后，相加减乘除即可

2. 无符号整数计算

   计算时，按照位对齐后，相加减乘除即可

3. **溢出**
   溢出：由于计算结果长度超过数据类型的长度限制

   对于有符号整数溢出：由于有符号整数第 32 位为符号位，如果在计算时，产生进位后致使符号位改变，会致使数符号和值同时改变

   对于无符号整数：溢出时，截断高位超出，保留剩余的数，会致使相加后反而值会减少

4. **乘除优化**
   由于乘除速度较慢，对于常数乘除话，会使用位运算进行优化

   举个栗子：

```
x * 3 => x * 2 + x * 1; // x 加上 x 向右移动一位
```

#### 浮点型存储

[javascript 浮点数](../../note/frontend/javascript/javascript浮点数.md)

单精度和双精度差不多，只是单精度为 32 位，双精度为 64 位

## 算法

### 任务调度器，每次取出最大数

**数组实现**

整体思想是：

1. 数组乱序，末尾插入数字，取出时寻找最大的数
2. 数组有序，插入数字必须保证顺序，取出时取第一位

**链表实现**
整体思想是：

1. 链表乱序，末尾插入数字，取出时寻找最大的数
2. 链表有序，插入数字必须保证顺序，取出时取第一位

**堆实现**

堆是一种完全二叉树存储结构，既然是完全二叉树，则可以使用数组来存储，并保证根节点为最大数

整体思想是：

1. 以数组保存堆，第 0 位不能存储节点，设为哨兵
2. 插入数字时，从末尾插入，并保证每个根节点都大于子节点
3. 取数字时，取根节点，并将末尾节点放置根节点，再保证每个根节点都大于子节点

```
class TaskQueue {
 private:
  int* store_arr;
  int last;
  int len;
  bool recursive_heap(int index); // 递归查看父节点是否大于子节点

 public:
  TaskQueue(int len);
  ~TaskQueue();
  int pop();
  bool insert(int);
  bool isHeap();
};

TaskQueue::TaskQueue(int _len) {
  store_arr = new int[_len];
  store_arr[0] = INT_MAX;
  len = _len;
  last = 1;
}

TaskQueue::~TaskQueue() { delete[] store_arr; }

bool TaskQueue::insert(int node) {
  if (last >= len - 1) {
    return false;
  }

  store_arr[last++] = node;
  int j = last - 1;
  while (store_arr[j / 2] < store_arr[j]) {
    int s = store_arr[j];
    store_arr[j] = store_arr[j / 2];
    store_arr[j / 2] = s;
    j /= 2;
  }
  return true;
}

int TaskQueue::pop() {
  int target = store_arr[1];
  store_arr[1] = store_arr[last];
  last--;
  int j = 1;
  while (j <= last / 2) {
    int left = j * 2;
    int right = left + 1;
    int change_index = store_arr[left] > store_arr[right] ? left : right;
    if (store_arr[j] >= store_arr[change_index]) {
      break;
    } else {
      int s = store_arr[j];
      store_arr[j] = store_arr[change_index];
      store_arr[change_index] = s;
      j = change_index;
    }
  }
  return target;
}

bool TaskQueue::isHeap() { return recursive_heap(1); }

bool TaskQueue::recursive_heap(int index) {
  if (index > (last - 1) / 2) return true;

  int left = index * 2;
  int right = left + 1;
  int right_value = right > last ? INT_MIN : store_arr[right];// 防止超出last限制
  if (store_arr[index] < store_arr[left] || store_arr[index] < right_value) {
    return false;
  }
  return recursive_heap(left) && recursive_heap(right);
}
```

## leetcode

### 回文数字

[回文数](https://leetcode-cn.com/problems/palindrome-number/)

思路：

存在变量： remain_num：保存余数组成的数；n：为输入数字

1. 将 n 余除以 10，得到余数
2. 将 remain_num \*10 + 余数（相当于将末尾数字和头部数字调换）
3. n = n / 10;
4. 如果 remain_num >= n，则数转换已经过半或正好一半，此时，如果 n = remain_num 相等，或者 remain_num \*去掉末尾一位后\_ 与 n 相等，则认为是回文数，否则不是回文数
5. 否则继续第一步骤

```
bool isPalindrome(int n) {
  // n 如果为负数，则完全不可能是回文数
  // 如果n末尾为0，则除了0也不可能是回文数

  if (n < 0 || (n != 0 && n % 10 == 0)) return false;

  int remain_num = 0;
  while (remain_num < n) {
    remain_num = remain_num % 10 + n % 10;
    n /= 10;
  }
  return remain_num == n || remain_num / 10 == n;
}
```

### 正则匹配算法

**状态机**算法，该算法先不写，后面专门写一篇文章来介绍

### 除自身以外数组的乘积

[除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/)

思路：

1. 从左向右循环数组，计算出第 i 左边的乘积
2. 从右向左循环，计算出第 i 右边的乘积
3. 将两个乘积相乘

优化：

1. 循环一次，找一个变量保存到第 i 位左边的乘积 left_plus，找一个变量保存第(length-1-i)位右边的乘积 right_plus
2. 第 i 位暂时保存自己左边的乘积，第(length-1-i)位暂时保存自己右边乘积
3. 当 i 循环超过一半是，左右开始交汇，则 第 i 位为`left_plus * 自身`；第(length-1-i)为将 `right_plus * 自身`

```
vector<int> productExceptSelf(vector<int>& nums) {
  int size = nums.size();
  vector<int> left_nums(size, 1);
  vector<int> right_nums(size, 1);

  for (int i = 1; i < size; i++) {
    left_nums[i] = left_nums[i - 1] * nums[i - 1];
  }

  for (int j = size - 2; j >= 0; j--) {
    right_nums[j] = right_nums[j + 1] * nums[j + 1];
  }

  for (int i = 0; i < size; i++) {
    nums[i] = left_nums[i] * right_nums[i];
  }
  return nums;
}
```

```
// --------- 优化 ----------
vector<int> productExceptSelf(vector<int>& nums) {
  int size = nums.size();
  int last = size - 1;
  vector<int> store_nums(size, 1);

  int left_plus = 1;
  int right_plus = 1;

  for (int i = 0; i < size; i++) {
    store_nums[i] = left_plus * store_nums[i];
    store_nums[last - i] = right_plus * store_nums[last - i];
    left_plus *= nums[i];
    right_plus *= nums[last - i];
  }
  return store_nums;
}
```

### 任务调度器

[任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

思路 1：

冷却长度：n；执行总步骤 m

1. 将每个任务计数，按照任务数从大到小排序
2. 每一轮执行任务，选择 n 个前列的任务，即将前 n 个任务数减 1，执行步骤 m += n
3. 当任务数量 < n，且不为同一种任务时，则结束任务执行 m += 任务数量

tips：自己想的方法，执行时间较长

思路 2：

1. 找出**最多的任务数**的任务，任务 X
2. 找出**同等任务数**的任务,任务 T[]
3. 想象将任务数最多的任务排开，每个任务中间空出 n 个，其中插入其他任意任务，（如果其他任务没了，相同任务也需要间隔 n 才能执行，所以中间仍然是 n）
4. 当执行到最后一步时，会剩下 T.length + 1 个任务
5. 执行完毕

tips：大佬的方法

```
int leastInterval(vector<char>& tasks, int n) {
      if (n == 0 || tasks.size() <= 1) return tasks.size();

      int task_counts[26] = {0};

      for (int i = 0; i < tasks.size(); i++) {
        task_counts[tasks[i] - 65] += 1;
      }

      sort(task_counts, task_counts + 25, greater<int>());

      int max_count = task_counts[0];
      int task_sum = 0;

      for (int i = 0; i < 26; i++) {
        if (task_counts[i] < max_count) break;
        task_sum++;
      }

      return max(int(tasks.size()), (max_count - 1) * (n + 1) + task_sum);
    }
```
