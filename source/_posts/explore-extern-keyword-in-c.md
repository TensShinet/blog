---
title: '[C] 探究 C 语言中的 extern 关键字'
date: 2022-08-27 16:40:46
tags: [C]
---



**TL; DR**



查资料得知 extern 关键字是变量在其他文件的声明。突然感觉很奇怪，那如果文件 A，在头文件中定义了某一个变量，文件 B 不使用 extern 不也是可以使用的吗？为什么还要用 extern 这个关键字呢？



结论：



1. 使用 extern 可以使用在其他文件定义（.c）的变量，即使这个文件没有在头文件中定义。
2. 假设只有两个文件 a.c b.c，在 b.c 中 include a.h，a 文件和 b 文件会共用 include 中的变量。
3. 如果要使用其他文件定义的变量，尽量使用 extern，这样也很明了。



<!--more-->



## extern 的用法



在 ext 目录下：

```c
➜  ext cat a.c 
int foo_a = 10;                                                                                 
➜  ext cat b.c 
#include<stdio.h>

extern int foo_a;

int main() {
    printf("%d ", foo_a);
    return 0;
}                                                                                               
➜  ext gcc a.c b.c -o p
➜  ext ./p
10  
```



没啥问题，b.c 中 extern 声明了一个在其他文件定义的变量。然后打印出来 10。



## 不使用 extern 但是在头文件中定义



```c
➜  ext cat a.h
int foo_a;
int get_foo_a();
➜  ext cat a.c
int foo_a = 10;

int get_foo_a() {
    return foo_a;
}                                                                                               
➜  ext cat b.c
#include<stdio.h>
#include "a.h"

int main() {
    printf("%d\n", foo_a);
    foo_a = 100;
    printf("%d\n", get_foo_a());
    return 0;
}                                                                                              
➜  ext gcc a.c b.c -o p
➜  ext ./p
10
100
```



虽然没有使用 extern，但是 a.c 和 b.c 使用的 foo_a 地址都是一样的。这说明使用 include 的也能使用其他位置定义的变量。



## 不使用 extern 在头文件中也没有定义



```c
➜  ext cat a.c 
int foo_a = 10;
➜  ext cat b.c
#include<stdio.h>

int main() {
    printf("%d\n", foo_a);
    return 0;
}                                                                                             
➜  ext gcc a.c b.c -o p
b.c:4:20: error: use of undeclared identifier 'foo_a'
    printf("%d\n", foo_a);
                   ^
1 error generated.
```



这样编译不过。



## 参考



+ https://stackoverflow.com/questions/1433204/how-do-i-use-extern-to-share-variables-between-source-files



