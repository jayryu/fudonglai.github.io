---
title: 现代C语言--I
date: 2018-11-19 01:09:53
tags:
categories:
	- C Programming Language
---





C语言简单而隐晦的角落。

<!--more-->

## scanf

scanf 遇到的字符不是当前内容时， 会把它放回原处，在扫描下一个输入项或下一次调用 scanf 时再试图读入。

```c
int i, j;
float x, y;
scanf("%d%d%f%f", &i, &j, &x, &y);
```

Suppose that the user enters three lines of input:

> 1-20.3-4.0e3

* 第一个转换符是 %d，第一个非空字符是 1，可以出现在整数中，下一个字符是 ”-“，scanf 认为它不应该出现在整数内部，所以把 “-” 放回原处，把 1 读入 i 。
* 第二个读入 -20 之后发现小数点，scanf 认为它不该出现在整数中，所以把 -20 读入 j 。
* 第三个读入 .3，发现负号不该出现在浮点数中间，所以把 0.3 读入 x 。
* 最后把 -4*10^3 读入 y，并把最后的换行符放回原处。

在读取格式串时遇到任何空白字符，scanf 会跳过空白符直到遇到非空白符尝试读取。

如 `"%d/%d"` 格式串，输入`	5/	96`，能够成功读取两个整数。

如输入`	5  /	96`，则在读取 5 后无法匹配 `/`而把之后的所有留给下次 scanf 调用。



## += 的副作用（side effect）

如果 v 有副作用，那么 v += e 不等价于 v = v + e 。

因为 v += e 只会导致一次 v 的求值，而 v = v + e 有两次。

如 `a[i++] += 1` 只会导致 i 自增 1；

而 `a[i++] = a[i++] + 1` 导致 i 自增 2 。



## getchar

getchar(), putchar() faster than scanf and printf.

**idiom**:

```c
while (getchar() != '\n') /*skips rest of line*/
```

```c
while ((ch = getchar()) == ' ') /*skips blanks*/
```

如果混用 scanf 和 getchar，注意 scanf 读取剩下的字符（包括换行符）会被 getchar 读取。



## 使用 %lf 读取 double 型的值，用 %f 进行显示

因为 printf 和 scanf 都没有限制参数个数，使用了可变长度参数列表。

当调用可变长度参数列表函数时编译器安排 float 自动转化成 double 型，所以 printf 无法区分 float 和 double，所以可以用 %f 表示 float 和 double。

但是 scanf 用指针指向变量，必须知道地址上存储的类型，否则如果 float 和 double 位模式不同，scanf 会存储错误的字节数量。



## typedef 优于宏定义

```c
#define PTR_TO_INT int *
PTR_TO_INT p, q;
// int *p, q;
// typedef will be better
```



## 远离 1

下标从 0 开始，注意数组的下标越界陷阱。

```c
int a[10], i;
for (i = 1; i <= 10; i++){
    a[i] = 0;
}
```

这里可能会导致无限循环。因为 `a[10]`已经越界，如果变量 `i`如果储存在这块空间，则错误地把`i`赋值为 0，无限循环。



## 数组初始化

如果给定长度和初始化列表，列表长度为0 或超过长度都是非法的，如果小于长度，剩余位置自动初始化为 0 。

```c
int a[10] = {1,2}
/*initial value: 1,2,0,0,0,0,0,0,0,0*/
```

**多维数组初始化：**

* 如果一行给出的列表不能填满该行，该行剩余填满 0 。
* 如果给出的列表行数不足数组行数，剩余行数全部填满 0 。
* 如果去掉大括号也可以，一旦填满一行，编译器自动去填下一行。（危险）

```c
int m[4][4] = {{1,2},
               {1,2,3,4}};
/* 1,2,0,0,
   1,2,3,4,
   0,0,0,0,
   0,0,0,0 */
```

所以要全部初始化为 0 应该是：

```c
int m[4][4] = {{0}};
```



## 数组和指针区别

1. 数组名不能被赋予新值。

   ```c
   int a[3] = {1,2,0};
   while (*a != 0)
       a++;		/*WRONG*/
   ```

2. `sizeof()`对数组名和指针效果不同。

   ```c
   int a[3] = {1,2,0};
   int len_of_a = sizeof(a) / sizeof(int); /* len_of_a == 3 */
   
   int *p = a;
   int len_of_p = sizeof(p); /* len_of_p == sizeof(int*) */
   ```

   sizeof 对普通指针计算一个指针在内存中的字节长度，这取决于机器。

   sizeof 对数组名计算该数组在内存中的总字节长度，所以可以那样求得数组元素个数。



## 函数形式参数中 int *a 还是 int a[ ] ?

考虑到函数参数传入的是拷贝，`int *a` 和 `int a[]` 完全相同，即便后者形式好像是数组，实际上也是普通指针，没有任何数组名的特殊技能。

```c
int fun(int a[]){
    int len = sizeof(a) / sizeof(a[0]);  /*WRONG*/
}
```



## 数组型参数

传入一维数组时，不需要写长度，写了也没用。

传入多为数组时，只有第一维长度是不必要的。因为要让编译器知道一个元素长度到底是多少。

```c
int fun1(int a[10]){  //OK
    /* equal to fun(int a[]) and fun(int* a) */
}

int fun2(int a[][10]){ //correct
    /* because complier konws there are sizeof(int) * 10 
    bits between a[0] and a[1] */
    
    /* fun2(int a[][]) is WRONG */
}
```



## i [a] ???

对编译器而言：`i[a] == *(i + a) == a[i]`



## 字符串字面量

最后一个是空字符`/0`，其 ASCII 码为 0 ，而数字 `0` 的 ASCII 码为 48 。



## 字符数组和数组指针

```c
char date_arr[] = "Nov 20";
char* date_ptr = "Nov 20";
```

* `date_ptr`是指针变量，可以再指向其他值，`date_arr`不能。
* `date_ptr`指向的字符串是字面值，不可修改，`date_arr`可以随意修改。

初始化字符数组时类似普通数组，若填不满空位添 0 ，超过限制就非法，但是不会检查最后是否有停止标识符`/0`。

考虑 `printf`和`scanf`第一个参数都是接受字符串常量，即该串的首地址。



## 读写字符串

### printf && puts

printf 逐个读取字符，直到读到空字符，如果空字符丢失，继续读，知道在内存找到一个空字符为止。

puts 接受一个参数没有格式串，写完字符串后自动添加换行符。

```c
char str[] = "abcdef";
printf("%.3s", str); // abc
puts(str); // abcdef
```

### scanf && gets

```c
#define LEN 11
char str[LEN];
scanf("%s", str);
/* input: "   hello world"*/
/*
   str: 'h','e','l','l','o','\0', ...
   scanf never get string with space, skip blank and end at blank,
   and add '\0' automatically.
*/
gets(str); //all will be recieved, but dangerous
```

