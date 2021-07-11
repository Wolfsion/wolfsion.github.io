---
layout: post
title:  "leetcode刷题(C语言)"
date:   2021-06-15
excerpt: "使用C语言的刷题总结"
tags: [算法, leetcode]
comments: true
---

# C语言 Leetcode



## day1

+ 指针读取与修改

```c
int value = 1;
int *p = &test;            //取地址
int ret = *p;              //取地址中的值
```



+ malloc()

```c
void *malloc(size_t size)   //字节为单位
    
str = (int *) malloc(sizeof(int));
```



+ 上下限

```c
INT_MIN     //整数下限
INT_MAX     //整数上限
```



+ 思路
  + 筛选特殊请况（回文数）
    + 单位数字   true
    + 负数   false
    + 10的倍数   false
  + 基本请况为主程序





## day2

+ c语言字符串长度

```c
#include <string.h>

char str[] = "Test";
int len = strlen(str);
```



+ c语言switch语句

```c
switch(c) {
        case 'I': ret = 1;break;
        case 'V': ret = 5;break;
        case 'X': ret = 10;break;
        case 'L': ret = 50;break;
        case 'C': ret = 100;break;
        case 'D': ret = 500;break;
        default: ret = 1000;break;
    }
```



+ 思路
  + 双指针——快慢指针去重复元素



## day3

+ c幂运算：double pow(double x, double y)

```c
//pow()用来计算以x为底的y次方值，然后将结果返回。设返回值为 ret，则 ret = x^y
#include <math.h>

int x = 2, y = 2;
int ret = pow(x,y);
```



+ c哈希值

```c
//long存储哈希值，即使哈希空间为INT_MAX
int curt = 0;
int range = INT_MAX;
long hays = 0;
for (int i = 0; i < len2; i++) {
    curt = needle[i] - 'a';
    need = (need*26 + curt) % range;
    ...
}

//防止溢出，并将负哈希偏移到正值（类似补码的思维）
hays = (hays*26 - c0*al % range + range) % range;
```



+ 二分搜索

```c
while (left < right) {                    //搜索区间为左开右闭
    mid = left + (right - left)/2;        //防止溢出更新mid
    if (nums[mid] >= target) right = mid; //==时也不断收缩上界，寻找下界
    else left = mid+1;                    //下界向上扩充         
}
//left即为下界边界
```



+ 思路
  + O(1)空间复杂度去重复元素，快慢指针
  + 排序数组的查找或搜索边界使用**二分搜索**
    + 关键
      + 区间开闭问题
      + 左右边界更新问题
    + 本质：确保搜索充分，当区间为[left, right)，区间更新后拆分为[left, mid)和[mid+1, right)







## day4

+ c数组传参与返回

```c
//除了char*，其余传入时均需给出数组大小(digitSize)，返回时大小也均需指定(returnSize)
int* plusOne(int* digits, int digitsSize, int* returnSize) {
    int *ret = (int *)malloc(sizeof(int)*(digitsSize));
    ...
    *returnSize = digitSize;
    return ret;    
}
   
//调用者
caller {
    int *get = plus(...);
    ...
    free(get);
    get = NULL;
}
```



+ 最后一个单词**问题转化**

```c
//s为单词，cnt为最终结果
int cnt = 0;
for (int i = len-1; i > -1; i--) {
    if (s[i] != ' ') cnt++;
    if (s[i] == ' ' && cnt != 0) break;              //正常计数后才会进入的分支
}
```



+ 思路
  + 贪心算法，记录当前最大的变量和**记录历史最大的变量**







## day5

+ c反转字符串

```c
//反转字符串
void reserve(char* s) {
    int len = strlen(s);
    for (int i = 0; i < len / 2; i++) {
        char t = s[i];
        s[i] = s[len - i - 1], s[len - i - 1] = t;
    }
}
```



+ ==c二进制模拟加法==

```c
//二进制加法1
char* addBinary(char* a, char* b) {
    reserve(a);
    reserve(b);

    int len_a = strlen(a), len_b = strlen(b);
    int n = fmax(len_a, len_b), carry = 0, len = 0;
    char* ans = (char*)malloc(sizeof(char) * (n + 2));
    
    for (int i = 0; i < n; ++i) {
        carry += i < len_a ? (a[i] == '1') : 0;    //true转换为整型时为1
        carry += i < len_b ? (b[i] == '1') : 0;
        ans[len++] = carry % 2 + '0';              //模2本位结果
        carry /= 2;                                //模2进位结果
    }

    if (carry) {
        ans[len++] = '1';                         //最高位进位
    }
    ans[len] = '\0';
    reserve(ans);

    return ans;
}

//二进制加法2
char * addBinary(char * a, char * b){
    int lena = strlen(a)，lenb = strlen(b);
    int len = fmax(lena, lenb), carry = 0;
    int i = lena - 1;                      //对a的索引
    int j = lenb - 1;                      //对b的索引
    char *res = (char*)malloc(sizeof(char) * (len + 2));
    
    int k = len + 1;                       //结果的索引
    res[k--] = '\0';                        //末尾赋结束符

    //索引合法，且还存在“加法数”carry
    while (i >= 0 || j >= 0 || carry > 0) {
        carry += (i >= 0) ? a[i--] - '0' : 0;
        carry += (j >= 0) ? b[j--] - '0' : 0;
        res[k--] = carry % 2 + '0';
        carry /= 2;
    }
    
    //当最高位进位时k=-1，反之为0，当未进位时，指针向后偏移一个单位
    return res + k + 1; 
}   
```



+ c取绝对值

```c
int a = abs(-1);
```



+ c对指针的运算

```c
int nums[5] = {1,2,3,4,5};
int *p = nums;
p = p + 1;         //p指向2
```



+ 思路

  + 求平方根：

    + 牛顿迭代法，两要素：递推关系与初始条件
    + 二分法，区间及更新分支(=,<,>)

  + 动态规划：

    + 两要素：

      + 递推关系：f(x) = f(x-1) + f(x-2)
      + 边界条件：f(1) = 1 和 f(0) = 1

    + 编写技巧：

      + 滚动数组
      + 边界改写为 prev = 0, curt = 1

      



## day6

+ c语言结构体及指针判空

```c
 struct ListNode {
 	int val;
 	struct ListNode *next;
 };

struct ListNode obj1;
...
struct ListNode* obj2;
...
    
int val1 = obj1.val;

//指针型通过->访问成员变量，空类型为NULL
if (obj2 != NULL) {
    int val2 = obj2->val;
}

```

+ c语言qsort()函数

```c

```



+ 思路
  + 双数组合并：从尾合并，剩余补足



## day7

+ 思路
  + 对称二叉树
    + 递归的函数参数为左右子树两个节点
    + 向深层递归时，交叉传入子树的左右子树
  + 平衡二叉树——先序遍历
    + 平衡二叉树的中序遍历是升序序列
    + 由升序序列还原二叉树时使用先序遍历
    + 每次的根为中间索引节点
  + 平衡二叉树判断
    + 先实现计算树高度
    + 递归判断每一个子树均需满足条件
  + 最浅叶子节点
    + 深度优先搜索
    + 递归搜索
    + ==广度优先搜索==



## day8

+ c语言二维数组

```c
//产生二维数组并返回
//结果二维数组和表示二维数组每行大小的数组需由malloc得到
int** generate(int numRows, int* returnSize, int** returnColumnSizes){
    //传入int指针需给出返回的行数，指明int的值
    *returnSize = numRows;                             
    
    //传入int*指针，需给出返回的每行size，指明int*中的各值
    *returnColumnSizes = (int*)malloc(sizeof(int)*numRows); 
    
    int** ret = (int**)malloc(sizeof(int*)*numRows);        //结果二维数组
    
    for (int i = 0; i<numRows; i++) {
        ret[i] = (int*)malloc(sizeof(int)*(i+1));    //单行数组需再次调研malloc指明大小
        (*returnColumnSizes)[i] = i+1;
        ...
    }
    return ret;
}
```



+ c语言批量赋值

```c
//调用memset()
/*
	void *memset(void *str, int c, size_t n)
	
	str -- 指向要填充的内存块。
	c -- 要被设置的值。该值以 int 形式传递，但是函数在填充内存块时是使用该值的无符号字符形式。
	n -- 要被设置为该值的字符数。
*/

memset(row, 0, sizeof(int) * size);   //sizeof(int)表明元素大小，当指针为int*，单体即为int
```



+ 思路
  + 杨辉三角
    + 递推：ret\[i]\[j] = ret\[i-1]\[j-1] + ret\[i-1]\[j]
    + 初始赋值：ret[0] = ret[i] = 1
  + 单行杨辉三角
    + 尾递推：ret[j] += ret[j-1] (1 <= j <= i)
  + 贪心动态规划——股票
    + 状态转移条件：当前股价a和持有股价b的比较
    + 抽象状态转移时的操作
      + 入股(a<b)：入股时对当前股票取反
      + 出股(a>b)：出股时计算差值即为收益
    + 定义保存结果的最大变量
      + 每轮转移均需判断是否改变该值
    + 局限：
      + 持有全局信息
      + 持股时，可零成本换到股价更低的股



## day9

+ c语言异或：^
+ c语言哈希表，使用uthash.h

+ 思路
  + 快慢指针 测链表中的环
    + O(1)的空间复杂度+O(n)的时间复杂度





# C语言数据结构

+ c语言栈

```c
int stack[10];                //stack为存储元素类型的指针
int top = 0;                  //栈维护“指针”

//测试实例
int item = 1;
//入栈
stack[top++] = item;

//出栈
item = stack[--top];

//计算size
int size = top;
```

+ c语言队列

```c
int queue[10];                //queue为存储元素类型的指针
int front = 0;                //队列维护“指针”
int rear = 0;                 //队列维护“指针”

//测试实例
int item = 1;

//入队列
queue[rear++] = item;

//出队列
item = queue[front++];

//计算size
int size = rear - front;
    
//创建example,存储树的节点的队列，树的节点为struct TreeNode*
struct TreeNode *queueList[MAX_SIZE];  
```



+ 链表递归遍历

```c

```



+ 链表的迭代遍历

```c

```



+ 树的递归遍历

  + 深度优先

  ```c
  
  ```

  

  + 广度优先

  ```c
  
  ```

  

  + 前中后序

  ```c
  //中序 左根右
  	//传入的参数为指针，对递归全局可见，包括数组和数组索引
  void helper(struct TreeNode* root, int* ret, int* retSize) {
      if (root == NULL) return;
      helper(root->left, ret, retSize);          //中序代码框架start
      ret[(*retSize)++] = root->val;      //(*retSize)使用指针访问值
      helper(root->right, ret, retSize);         //中序代码框架end
  }
  
  
  int* inorderTraversal(struct TreeNode* root, int* returnSize){
      *returnSize = 0;
      int* ret = (int*)malloc(sizeof(int)*101);    //数字101需要根据题干意思，保证不越界
      if (!root)	return ret;
      helper(root, ret, returnSize);
      return ret;
  }
  
  //前序 根左右
  void helper(struct TreeNode* root, int* returnSize, int* ret) {
      if (root == NULL) return;
      ret[(*returnSize)++] = root->val;
      helper(root->left, returnSize, ret);
      helper(root->right, returnSize, ret);
  }
  
  //后序 左右根
  void helper(struct TreeNode* root, int* returnSize, int* ret) {
      if (root == NULL) return;
      helper(root->left, returnSize, ret);
      helper(root->right, returnSize, ret);
      ret[(*returnSize)++] = root->val;
  }
  ```

  

+ 树的迭代遍历

  + 深度优先

  ```c
  
  ```

  

  + 广度优先

  ```c
  
  ```

  

  + 前中后序

  ```c
  //中序 左根右
  int* inorderTraversal(struct TreeNode* root, int* returnSize){
      *returnSize = 0;
      int* ret = (int*)malloc(sizeof(int)*101);
      if (!root) return ret;
      int top = 0;                   //top <= 0 时栈为空
      struct TreeNode** stack = (struct TreeNode**)malloc(sizeof(struct TreeNode*)*101);
      while (root != NULL || top > 0) {
          while (root != NULL) {            //一直入栈左子树直到为空;也可能是上次循环中的root->right为空
              stack[top++] = root;
              root = root->left;
          }
          root = stack[--top];                      //出栈保存值
          ret[(*returnSize)++] = root->val;
          root = root->right;                       //遍历右子树
      }
      return ret;
  }
  
  //前序 根左右
  int* preorderTraversal(struct TreeNode* root, int* returnSize){
      int *ret = (int*)malloc(sizeof(int)*101);
      *returnSize = 0;
      if (root == NULL) return ret;
  
      int top = 0;
      struct TreeNode* stack[100];
      struct TreeNode* curt = root;
  
      while (top != 0 || curt != NULL) {
          while (curt != NULL) {
              ret[(*returnSize)++] = curt->val;
              stack[top++] = curt;
              curt = curt->left;
          }
          curt = stack[--top]->right;
      }
      return ret;
  }
  
  //后序 左右根
  int *postorderTraversal(struct TreeNode *root, int *returnSize) {
      int *res = malloc(sizeof(int) * 101);
      *returnSize = 0;
      if (root == NULL) {
          return res;
      }
      struct TreeNode *stk[101];
      int top = 0;
      struct TreeNode *prev = NULL;                           //prev用来保存上次的右子树节点
      while (root != NULL || top > 0) {
          while (root != NULL) {
              stk[top++] = root;
              root = root->left;
          }
          root = stk[--top];                                  //出栈保存根节点
          if (root->right == NULL || root->right == prev) {   //右子树为空或已保存过，直接保存根节点
              res[(*returnSize)++] = root->val;
              prev = root;
              root = NULL;
          } else {                                            //进入右子树
              stk[top++] = root;
              root = root->right;
          }
      }
      return res;
  }
  ```
  
  
