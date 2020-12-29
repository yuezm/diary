# Sep-1.md

## CSS

### box-shadow

阴影

```css
box-shadow: inset <offset-x> <offset-y> <blur-radius> <spread-radius> <color1> <color2>

inset：内阴影
offset-x：x便宜，可为负
offset-y：y便宜，可为负
blur-radius：模糊度，越大越模糊，最小位0
spread-radius：阴影宽度，越大阴影越宽，负值导致阴影缩小
```

## LeetCode

### 移动0

1. 解法1（不考虑空间）：使用另一个数组存储，如果是0，则跳过，空出的以0填充

2. 解法2（考虑空间）：双指针，对于0不做处理，缺点是在最后对0进行补充

```typescript
function moveZeroes(nums: number[]): void {
  let x = 0, y = 0;

  while (x < nums.length && y < nums.length) {
    console.log(x, y);

    if (nums[ y ] !== 0) {
      nums[ x ] = nums[ y ];
      x++;
    }
    y++;
  }

  while (x < y) {
    nums[ x ] = 0;
    x++;
  }
};
```

1. 解法3（考虑操作次数）：双指针，使用后续的非0数字替换前面的0，看起来像不停的把0往后移动，对0也进行移动，缺点是需要赋值两次

```typescript
function moveZeroes(nums: number[]): void {
  let x = 0, y = 0;

  while (x < nums.length && y < nums.length) {
    if (nums[ y ] !== 0) {
      const s = nums[ x ];
      nums[ x ] = nums[ y ];
      nums[ y ] = s;
      x++;
    }
    y++;
  }
};
```

[移动0](https://leetcode-cn.com/problems/move-zeroes/)

### 有效数独

1. 暴力法：多次循环，根据规则判定是否符合

  - 可选择使用数组，来存储已存在的数字，用来去重，即以 _rows[0][2] 表示0横排，存在数字2_，_cols[1][3] 表示第1竖排存在数字3_
  - 可使用map来存储已存在的数字，用来去重

**最关键的是如何来表示3x3宫格**：无论是横还是竖，(0,1,2)是一组，(3,4,5)是一组，(6,7,8)是一组，根据规律，可以看出可以由 index%3 进行分组，matrix[0][0][2] 表示左上角的3x3存在数字2

```typescript
function isValidSudoku(board: string[][]): boolean {
  const rows: number[][] = new Array(10); // 存储横排
  const cols: number[][] = new Array(10); // 存储竖排

  for (let i = 0; i < 10; i++) {
    rows[ i ] = new Array(10).fill(0);
    cols[ i ] = new Array(10).fill(0);
  }

  const threeMatrix: number[][][] = new Array(3); // 存储3x3
  for (let i = 0; i < 3; i++) {
    threeMatrix[ i ] = new Array(3);
    for (let j = 0; j < 3; j++) {
      threeMatrix[ i ][ j ] = new Array(10).fill(0);
    }
  }

  for (let row = 0; row < 9; row++) {
    for (let col = 0; col < 9; col++) {
      const v = board[ row ][ col ];
      if (v === '.') continue;
      const nv = Number(v);

      const zcCol = Math.floor(col / 3);
      const zcRow = Math.floor(row / 3);

      if (rows[ row ][ nv ] !== 0 || cols[ col ][ nv ] !== 0 || threeMatrix[ zcCol ][ zcRow ][ nv ] !== 0) {
        return false;
      }
      rows[ row ][ nv ] = 1;
      cols[ col ][ nv ] = 1;
      threeMatrix[ zcCol ][ zcRow ][ nv ] = 1;
    }
  }
  return true;
};
```

[有效数独](https://leetcode-cn.com/problems/valid-sudoku/)

### 矩阵置零

1. 使用其余空间，进行存储，不对原始数组进行修改，空间复杂度O(m*n)

```typescript
function setZeroes(matrix: number[][]): void {
  const col = matrix.length;
  const row = matrix[ 0 ].length;

  const s: number[][] = new Array(col);
  for (let i = 0; i < col; i++) {
   s[ i ] = new Array(row).fill(0);
  }
};
```

1. 使用数组，对索引进行存储，标记对应的列(cols)、行(rows)为0，空间复杂度为O(m+n)

```typescript
function setZeroes(matrix: number[][]): void {
  const col = matrix.length;
  const row = matrix[ 0 ].length;


  const cols = new Array(col).fill(0);
  const rows = new Array(row).fill(0);

  for (let i = 0; i < col; i++) {
    for (let j = 0; j < row; j++) {
      if (matrix[ i ][ j ] === 0) {
        cols[ i ] = 1; // 记录哪一排为0
        rows[ j ] = 1; // 记录哪一行为0
      }
    }
  }


  for (let i = 0; i < col; i++) {
    // 如果一排都为0，直接替换
    if (cols[ i ] === 1) {
      matrix[ i ] = new Array(row).fill(0);
      continue;
    }
    // 如果一列为0，则单个替换
    for (let j = 0; j < row; j++) {
      if (rows[ j ] === 1) {
        matrix[ i ][ j ] = 0;
      }
    }
  }
};
```

1. 在原数组进行修改，但存在一个特殊情况，如果一个元素为0，则直接将其竖排和横排置为0，但是如果直接置为0，则在其他循环可能会造成误解（_这个0到底是被别的位重置的还是自己本身就是0_）,所以可以使用**标记**来区分

  - 暴力标记法：当在matrix[i][j]为0时，将横排和竖排同时标记为 Number.MAX_VALUE，但是有个缺点，如果同排、列出现0时，会重复标记，效率较低
  - 优化标记法：使用标记其实是为了避免后续遍历时，搞混这个0是怎么来的，那么把标记往前挪，挪到已经遍历过的地方，则不会出现这个问题，对于竖排，放置于matrix[0][j]，对于横排，放置于matrix[i][0]，**但是这里存在一个特殊值 matrix[0][0]**，此点是交汇点，需要特殊处理，以 matrix[0][0] 表示是否竖排为0，以 isFirstColZero 表示横排是否为0

```typescript
/**
 Do not return anything, modify matrix in-place instead.
 */
function setZeroes(matrix: number[][]): void {
  const col = matrix.length;
  const row = matrix[ 0 ].length;

  let isFirstColZero = false;

  // 单独循环第一行，特殊处理
  for (let j = 0; j < row; j++) {
    if (matrix[ 0 ][ j ] === 0) {
      isFirstColZero = true;
      matrix[ 0 ][ j ] = 0;
    }
  }

  for (let i = 1; i < col; i++) {
    for (let j = 0; j < row; j++) {
      if (matrix[ i ][ j ] === 0) {
        // 对横排以及竖排标记
        matrix[ 0 ][ j ] = 0;
        matrix[ i ][ 0 ] = 0;
      }
    }
  }

  for (let i = 1; i < col; i++) {
    if (matrix[ i ][ 0 ] === 0) {
      matrix[ i ] = new Array(row).fill(0);
      continue;
    }

    for (let j = 0; j < row; j++) {
      if (matrix[ 0 ][ j ] === 0) {
        matrix[ i ][ j ] = 0;
      }
    }
  }

  // 单独处理第一行，切记最后处理第一排，不然会因为第一排置为0，导致全部为0
  isFirstColZero && (matrix[ 0 ] = new Array(row).fill(0));
}
```

[矩阵置零](https://leetcode-cn.com/problems/set-matrix-zeroes/)
