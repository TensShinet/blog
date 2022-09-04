---
title: '[Bison] Bison 中的 prec' 
date: 2022-09-03 13:00:52
tags: [bison]
---



**TL; DR**



本文主要研究在 Bison 中 %prec 的含义。



结论：



```yacc
%left '+' 'a'
%left '*'
%left ptm

...

    
exp : exp '+' exp            {$$ = $1 + $3;}
    | exp 'a' exp            {$$ = $1 - $3;}
    | exp '*' exp            {$$ = $1 * $3;}
    | 'a' exp %prec ptm      {$$ = $2 * $2;}
    | term                   {$$ = $1;}
    ; 
    
    
```



表示 `exp : 'a' exp %prec ptm` 这条语法的优先级和关联性与 ptm 这个符号一致。

<!--more-->



## 基础知识



### Bison 中的 shift/reduce



```yacc
// calc.l
...
[0-9]+                 {yylval.num = atoi(yytext); return number;}
[ \t\n]                ;
[-+*/a;]           	   {return yytext[0];}
...

// calc.y
...
exp : exp '+' exp            {$$ = $1 + $3;}
    | exp '-' exp            {$$ = $1 - $3;}
    | exp '*' exp            {$$ = $1 * $3;}
    | '-' exp                {$$ = -1 * $2;}
    | term                   {$$ = $1;}
    ; 
    
term : number                {$$ = $1;}
     ;
...
```



上面是一个只有 +-* 的语法，下面用一个实际的例子来解释 Bison 中的 shift/redeuce。



假设要规约 `-4+4;`通过语法分析生成下述的 token 数组 `[-, number, +, number]`，规约过程如下：



```txt
. - number + number
```



读入一个下一个符号，shift：



```txt
- . number + number
```



无法 reduce，shift：



```txt
- number . + number
```



number 可以 reduce，下一个 token 是 `+`，所以选择 reduce：



```txt
- term . + number
```



term 可以 reduce，下一个 token 是 +，所以选择 reduce：



```txt
- exp . + number
```



现在遇到一个问题：`-` exp 可以规约，但是 exp 后面也可以匹配 `+`，也就是此时遇到了 shift/reduce 冲突。Bison 选择能 shift 就 shift，除非有其他的优先级的设置。



> This situation, where either a shift or a reduction would be valid, is called a *shift/reduce conflict*. Bison is designed to resolve these conflicts by choosing to shift, unless otherwise directed by operator precedence declarations. To see the reason for this, let’s contrast it with the other alternative.



这段话来自：https://www.gnu.org/software/bison/manual/html_node/Shift_002fReduce.html 



因为此时没有其他的优先级的设置，所以选择 shift：



```
- exp + . number
```



无法 reduce 选择 shift：



```txt
- exp + number .
```



选择 reduce：



```txt
- exp + term .
```





选择 reduce：



```txt
- exp + exp .
```



选择 reduce：



```txt
exp + exp .
```



选择 reduce：



```
exp .
```



### Bison 中的 Precedence  和 Associativity



Bison 文件中：



```
%right '='
%left '+' '-'
%left '*' '/'
```



这个在 Bison 中表示：



1. 优先级：`=` < `+` == `-` < `*` == `/`
2. 关联性：`=` 右关联，`+-*/` 左关联



优先级好理解，1 + 2 * 3 先乘法后加法。

关联性表示，当遇到 1 + 1 + 2 的时候，左关联表示：

```
(1 + 1) + 2
```



右关联性表示：



```
1 + (1 + 2)
```



赋值语句是右关联的，因为：



```
a = b = c
```



是先给 b 赋值再给 a 赋值的。



### %prec 在 Bison 中的作用



```
%left '+' 'a'
%left '*'
%left ptm

...

    
exp : exp '+' exp            {$$ = $1 + $3;}
    | exp 'a' exp            {$$ = $1 - $3;}
    | exp '*' exp            {$$ = $1 * $3;}
    | 'a' exp %prec ptm      {$$ = $2 * $2;}
    | term                   {$$ = $1;}
    ; 
    
    
```



首先为什么要引入一个新的符号 `a` 呢？原因在于 `-` 一元运算符的优先级虽然比 `*` 高，但是先计算 `*` 还是先计算 `-` 不影响最后的答案。所以本文构造一个新的运算符 `a`。它在做二元运算的时候和减法一致。一元运算的时候将后者的值平发。并且做二元运算的时候优先级比 `*` 低。



结论就是 `%prec ptm` 把 `exp: 'a' exp ` 的优先级提升到和 ptm 这个符号一致。下面通过实际的例子解释为什么要这样做。



## 实际操作



本文构造的运算符 `a` 在做一元运算符的时候优先级要比 `*` 高。比如当遇到下面的输入的时候：



```txt
a4*4;
```



答案应该为 `(a4)*4 = 64` 而不应该是`a(4*4)=256`。但是 `a` 已经定义过优先级了：



```yacc
%left '+' 'a'
%left '*'
%left ptm
```



比 `*` 的优先级要低。为了实现这一目的可以使用 %prec 这个符号。所有代码如下：



```bison
// calc.y
%{
void yyerror (char *s);
int yylex();
#include <stdio.h>     /* C declarations used in actions */
#include <stdlib.h>
#include <ctype.h>
%}

%union {int num; char id;}         /* Yacc definitions */
%start line
%token <num> number
%type <num> line exp term

%left 'a'
%left '*'
%left ptm

// %4 '+'


%%

/* descriptions of expected inputs     corresponding actions (in C) */

line : exp ';'              {printf("Printing %d\n", $1);}
     | line exp ';'         {printf("Printing %d\n", $2);}
     ;
    

exp : exp '+' exp            {$$ = $1 + $3;}
    | exp 'a' exp            {$$ = $1 - $3;}
    | exp '*' exp            {$$ = $1 * $3;}
    | 'a' exp                {$$ = $2 * $2;}
    | term                   {$$ = $1;}
    ; 
    
term : number                {$$ = $1;}
     ;

%%                     /* C code */
int main (void) {
  /* init symbol table */
  return yyparse ( );
}

void yyerror (char *s) {
  fprintf (stderr, "%s\n", s);} 


```



```flex
// calc.l
%{
#include "calc.tab.h"
void yyerror (char *s);
int yylex();
%}
%%
[0-9]+                 {yylval.num = atoi(yytext); return number;}
[ \t\n]                ;
[-+*/a;]               {return yytext[0];}
.                      {ECHO; yyerror ("unexpected character");}

%%
int yywrap (void) {return 1;}

```



```makefile
calc: lex.yy.c y.tab.c
	gcc -g lex.yy.c calc.tab.c -o calc

lex.yy.c: y.tab.c calc.l
	flex calc.l

y.tab.c: calc.y
	bison -d calc.y

clean: 
	rm -rf lex.yy.c calc.tab.c calc.tab.h calc calc.dSYM
```



当没有 `%prec` 的时候遇到 `a4*4;`：



```sh
$ make calc && ./calc           
bison -d calc.y
calc.y: conflicts: 6 shift/reduce
flex calc.l
gcc -g lex.yy.c calc.tab.c -o calc
a4*4;
Printing 256
```



答案为 256，加上 %prec 以后：



```
// calc.y 部分代码
...
exp : exp '+' exp            {$$ = $1 + $3;}
    | exp 'a' exp            {$$ = $1 - $3;}
    | exp '*' exp            {$$ = $1 * $3;}
    | 'a' exp %prec ptm      {$$ = $2 * $2;}
...
```



```sh
$ make calc && ./calc
bison -d calc.y
calc.y: conflicts: 6 shift/reduce
flex calc.l
gcc -g lex.yy.c calc.tab.c -o calc
a4*4;
Printing 64
```



原因在于提升了 `a exp` 的优先级。当遇到 `a exp . *` 的时候选择 reduce 而不是 shift。



## 参考



+ https://www.gnu.org/software/bison/manual/html_node/Shift_002fReduce.html
+ https://www.gnu.org/software/bison/manual/html_node/Contextual-Precedence.html
+ https://www.ibm.com/docs/en/zos/2.3.0?topic=section-precedence-in-grammar-rules
+ https://www.ibm.com/docs/en/zos/2.4.0?topic=section-precedence-associativity
