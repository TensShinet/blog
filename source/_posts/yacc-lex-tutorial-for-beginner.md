---
title: '[Tutorial] Lex&Yacc 简明教程'
date: 2022-08-27 11:47:48
tags: [yacc, lex, tutorial]
---

**TL; DR**



最近学习 TinySQL，TinySQL 的 Project2 与和 Yacc 和 Lex 相关，所以总结了一下相关的知识。这篇博客记录了如何单独使用 Lex 以及如何使用 Lex 和 Yacc 构造一个简单的计算器。



<!--more-->



## Lex



Lex 是一个「词法分析器」。很多教程都没有教如何单独使用 Lex 做词法分析。但是，这对理解 Yacc 有着至关重要的作用。这部分的内容来自：https://www.youtube.com/watch?v=54bo1qaHAfk ，大家有兴趣可以看看视频。



词法分析器：就是把文本转换成一个可以识别每一个 Token 的 Token 数组。



举个具体的例子，假设有这样的文本：



```yaml
db_type: mysql
db_name: testdata
db_table_prefix: test_
db_port: 1099
```



通过「词法分析」可以将上述文本装换成下面的 Token 数组：



```C
[TYPE, COLON, SPACE, IDENTIFIER, EOL, NAME, COLON, SPACE, IDENTIFIER, EOL, TABLE_PREFIX, COLON, SPACE, IDENTIFIER, EOL, PORT COLON, SPACE, INTEGER]
```



这样看可能比较抽象，我具体抽出一行来看看效果：



```yaml
db_type:  mysql

```



被转换成了：



```c
[TYPE, COLON, SPACE, IDENTIFIER, EOL]
```



TYPE 表示 `db_type` 被识别成了 TYPE 类型的 Token。同理，COLON 表示 `:` 被识别成了 COLON 类型的 Token；SPACE 表示  被识别成了 SPACE 类型的 Token；`mysql` 被识别成 IDENTIFIER 类型的 Token。`\n` 被识别成 EOL 类型的 Token。



每一个转换后的 Token 类型都是事先定义好的。接下来展示如何使用 Lex 完成这一过程，所有代码在这个仓库中：https://github.com/jengelsma/lex-tutorial 。



### 定义每一个 Token 的类型

```c
// myscanner.h
#define TYPE 1
#define NAME 2
#define TABLE_PREFIX 3
#define PORT 4
#define COLON 5
#define IDENTIFIER 6
#define INTEGER 7
```



### 编写 lex 文件



```c
// myscanner.l
%{
#include "myscanner.h"
%}
%option nounput yylineno

%%
:					return COLON;
"db_type"			return TYPE;
"db_name"			return NAME;
"db_table_prefix"	return TABLE_PREFIX;
"db_port"			return PORT;

[a-zA-Z][_a-zA-Z0-9]*	return IDENTIFIER;
[1-9][0-9]*				return INTEGER;
[ \t\n]					;
.					printf("unexpected character\n");

%%

int yywrap(void)
{
	return 1;
}

```



现在来解释 myscanner.l 这个文件在做什么。Lex 文件通常由下面的结构构成：



```c
DECLARATIONS

%%

RULES

%%

AUXILIARY FUNCTIONS
```





`DECLARATIONS`：一些声明，比如用 %{}% 包裹的头文件的定义，里面包含了定义好的 Token 的类型。%option nounput yylineno 表明要使用的变量，这个变量的使用会在后面展示。



`RULES`：定义一些正则表达式，后面跟着的就是用 C 编写的如果 Lex 成功识别了这个正则表达式需要做些什么。比如当识别文本中 `db_type` 这个文本，就会 Return TYPE（TYPE 在 myscanner.h 定义好的类型）。这个「Return Type」的具体什么意思到后面会展示。



`UXILIARY FUNCTIONS`：一些辅助函数。就是一些 lex 在做词法分析的时候可能需要使用的函数。lex 在扫描完一个 file 以后不知道是否要继续扫描就会调用 yywrap 这个函数，如果返回 0 就继续扫描下一个。返回非 0 就停止扫描。关于 yywrap 可以看 https://silcnitc.github.io/lex.html ，里面有介绍，返回 0 的时候如何告诉 lex 下一个扫描哪个 file。



**在 RULES 还要注意一点的是优先级的问题，写在前面的 Rule 优先级越高。不然 IDENTIFIER 的正则表达式就能匹配 TYPE 了，发生冲突了。**



### 使用 lex



使用下述命令把 myscanner.l 变成一个 c 文件：



```shell
$ lex myscanner.l
```



可以看到多了一个 `lex.yy.c` 的文件，这个文件就是 lex 这个工具生成的用于词法分析的 C 代码。



### 使用 lex.yy.c



```c
// myscanner.c
#include <stdio.h>
#include "myscanner.h"

extern int yylex();
extern int yylineno;
extern char* yytext;

char *names[] = {NULL, "db_type", "db_name", "db_table_prefix", "db_port"};

int main(void) 
{

	int ntoken, vtoken;

	ntoken = yylex();
	while(ntoken) {
		printf("%d\n", ntoken);
		if(yylex() != COLON) {
			printf("Syntax error in line %d, Expected a ':' but found %s\n", yylineno, yytext);
			return 1;
		}
		vtoken = yylex();
		switch (ntoken) {
		case TYPE:
		case NAME:
		case TABLE_PREFIX:
			if(vtoken != IDENTIFIER) {
				printf("Syntax error in line %d, Expected an identifier but found %s\n", yylineno, yytext);
				return 1;
			}
			printf("%s is set to %s\n", names[ntoken], yytext);
			break;
		case PORT:
			if(vtoken != INTEGER) {
				printf("Syntax error in line %d, Expected an integer but found %s\n", yylineno, yytext);
				return 1;
			}
			printf("%s is set to %s\n", names[ntoken], yytext);
			break;
		default:
			printf("Syntax error in line %d\n",yylineno);
		}
		ntoken = yylex();
	}
	return 0;
}

```





理解这段代码首先需要理解三个 extern 的东西。这三个东西都是在 lex.yy.c 中定义的函数或者变量。



```c
extern int yylex();
extern int yylineno;
extern char* yytext;
```



+ yylex



这个函数每次调用都会返回一个 Token Type，也就是在 myscanner.l 中写的 Return XXX。当然它的返回不是一成不变的，是按照识别的顺序一个 Token Type 一个 Token Type 返回的。举个具体例子。假设词法分析最终的结果是：



```c
[TYPE, COLON, SPACE, IDENTIFIER, EOL, NAME, COLON, SPACE, IDENTIFIER, EOL, TABLE_PREFIX, COLON, SPACE, IDENTIFIER, EOL, PORT COLON, SPACE, INTEGER]
```



那么第一次调用 yylex，返回的就是 TYPE。

第二次调用 yylex，返回的就是 COLON。

...

以此类推。



+ yylineno



yylex() 只是返回一个 Token Type。但是这个 Token Type 在哪一行就是通过这个 yylineno 全局变量存储的。



+ yytext



yylex() 只是返回一个 Token Type。但是这个 Token Type 的字面量是什么是通过 yytext 这个全局变量存储的。





如果实在不理解，可以具体在程序中调试调试。



### 最后结果





```shell
$ gcc lex.yy.c myscanner.c -o myscanner
$ ./myscanner < config.in 
1
db_type is set to mysql
2
db_name is set to testdata
3
db_table_prefix is set to test_
4
Syntax error in line 4, Expected an integer but found a1099
```



能够使用 Lex 以后，本文继续介绍如何使用 Yacc。





## Yacc



Yacc 是个「语法分析器」，通过 lex 得到每一个 Token，去找到匹配的语法。这一部分的内容也来自：https://www.youtube.com/watch?v=__-wUHG2rfM ，所有的代码来自：https://github.com/jengelsma/yacc-tutorial 。



代码仓库中定义的 yacc 文件：



```c
%{
void yyerror (char *s);
int yylex();
#include <stdio.h>     /* C declarations used in actions */
#include <stdlib.h>
#include <ctype.h>
int symbols[52];
int symbolVal(char symbol);
void updateSymbolVal(char symbol, int val);
%}

%union {int num; char id;}         /* Yacc definitions */
%start line
%token print
%token exit_command
%token <num> number
%token <id> identifier
%type <num> line exp term 
%type <id> assignment

%%

/* descriptions of expected inputs     corresponding actions (in C) */

line    : assignment ';'		{;}
		| exit_command ';'		{exit(EXIT_SUCCESS);}
		| print exp ';'			{printf("Printing %d\n", $2);}
		| line assignment ';'	{;}
		| line print exp ';'	{printf("Printing %d\n", $3);}
		| line exit_command ';'	{exit(EXIT_SUCCESS);}
        ;

assignment : identifier '=' exp  { updateSymbolVal($1,$3); }
			;
exp    	: term                  {$$ = $1;}
       	| exp '+' term          {$$ = $1 + $3;}
       	| exp '-' term          {$$ = $1 - $3;}
       	;
term   	: number                {$$ = $1;}
		| identifier			{$$ = symbolVal($1);} 
        ;

%%                     /* C code */

int computeSymbolIndex(char token)
{
	int idx = -1;
	if(islower(token)) {
		idx = token - 'a' + 26;
	} else if(isupper(token)) {
		idx = token - 'A';
	}
	return idx;
} 

/* returns the value of a given symbol */
int symbolVal(char symbol)
{
	int bucket = computeSymbolIndex(symbol);
	return symbols[bucket];
}

/* updates the value of a given symbol */
void updateSymbolVal(char symbol, int val)
{
	int bucket = computeSymbolIndex(symbol);
	symbols[bucket] = val;
}

int main (void) {
	/* init symbol table */
	int i;
	for(i=0; i<52; i++) {
		symbols[i] = 0;
	}

	return yyparse ( );
}

void yyerror (char *s) {fprintf (stderr, "%s\n", s);} 


```



定义的 lex 文件如下：



```c
%{
#include "y.tab.h"
void yyerror (char *s);
int yylex();
%}
%%
"print"				   {return print;}
"exit"				   {return exit_command;}
[a-zA-Z]			   {yylval.id = yytext[0]; return identifier;}
  [0-9]+                 {yylval.num = atoi(yytext); return number;}
[ \t\n]                ;
[-+=;]           	   {return yytext[0];}
.                      {ECHO; yyerror ("unexpected character");}

%%
int yywrap (void) {return 1;}
```





这个规则主要的功能如下：



```shell
➜  yacc-tutorial git:(master) ✗ ./calc
print 10+10;
Printing 20
a=10;
b=10;
print a+b;
Printing 20

```





本章主要说明 clac.y clac.l 这两个文件的每一行代码的含义。



### Yacc 文件的基本结构



```c
/* definitions */
 ....

%% 
/* rules */ 
....
%% 

/* auxiliary routines */
.... 
```





基本与 Lex 一致。



### Definitions



```c
%{
void yyerror (char *s);
int yylex();
#include <stdio.h>     /* C declarations used in actions */
#include <stdlib.h>
#include <ctype.h>
int symbols[52];
int symbolVal(char symbol);
void updateSymbolVal(char symbol, int val);
%}
```



这一部分写 yacc 生成文件的头部。symbols 存储 52 个变量（[a, z] + [A, Z]）的值，只能是 int。symbolVal 查询变量的值。updateSymbolVal 更新变量的值。



```c
%union {int num; char id;}         /* Yacc definitions */
%start line
%token print
%token exit_command
%token <num> number
%token <id> identifier
%type <num> line exp term 
%type <id> assignment
```



要看懂这一部分在干什么，需要理解：**一个 Token 除了它的字面量以外还有它的语义值**。



比如，1000 按照 clac.l 定义它是一个 number，它的字面量是 '1000'，是一个 string 可以由全局变量 yytext 获得。同理，它的语义值 1000，需要存储在全局变量 yyval 中。由于它是一个全局变量，每一个 Token 的语义值的类型可能是不一样的。所以需要用 union 这个关键字，指定这个 Token 对应的语义值如何存储，语义值的类型是什么。与 C 的 Union 的关键字含义一致，共用 yyval 的内存。比如，在 clac.l 中当匹配 [0-9]+ 的时候这个 Token 的语义值就被放入了 yyval.num 中。



接着：%start 规定了从哪个语法开始解析，%token 定义了新的 token，%type 定义了非终结符的类型，可以用来做非终结符的类型检查。具体来说：





1. `%token <num> number` 在定义了新的 token number 的同时还定义了 这个 number 的语义值存储在 yyval 的 num 中。

2. `%token <id> identifier` 在定义了新的 token identifier 的同时还定义了 这个 identifier 的语义值存储在 yyval 的 id 中。

3. `%type <num> line exp term` 定义了非终结符的 type 为 num，这个 type 来自 %union，可以用来做非终结符的类型检查。

4. `%type <num> assignment` 定义了非终结符的 type 为 id，这个 type 来自 %union，可以用来做非终结符的类型检查。



### Rules



```c

line    : assignment ';'		{;}
		| exit_command ';'		{exit(EXIT_SUCCESS);}
		| print exp ';'			{printf("Printing %d\n", $2);}
		| line assignment ';'	{;}
		| line print exp ';'	{printf("Printing %d\n", $3);}
		| line exit_command ';'	{exit(EXIT_SUCCESS);}
        ;

assignment : identifier '=' exp  { updateSymbolVal($1,$3); }
			;
exp    	: term                  {$$ = $1;}
       	| exp '+' term          {$$ = $1 + $3;}
       	| exp '-' term          {$$ = $1 - $3;}
       	;
term   	: number                {$$ = $1;}
		| identifier			{$$ = symbolVal($1);} 
        ;
```





这一部分定义了语法。主要有以下需要注意的地方：



1. `{}` 定义了发现了这个语法以后需要做什么。
2. `line assignment` 这样的递归写法可以当发现了一个 `assignment;` 或者`exit_command;` 或者 `print exp;`  以后继续读取 token。
3. `$$ = $1 + $3;` 表示左边的语义值等于右边第一个参数的语义值加上第三个参数的语义值。
4. 一个语法内部的优先级从上到下依次降低。





### Auxiliary routines



```c
int computeSymbolIndex(char token)
{
	int idx = -1;
	if(islower(token)) {
		idx = token - 'a' + 26;
	} else if(isupper(token)) {
		idx = token - 'A';
	}
	return idx;
} 

/* returns the value of a given symbol */
int symbolVal(char symbol)
{
	int bucket = computeSymbolIndex(symbol);
	return symbols[bucket];
}

/* updates the value of a given symbol */
void updateSymbolVal(char symbol, int val)
{
	int bucket = computeSymbolIndex(symbol);
	symbols[bucket] = val;
}

int main (void) {
	/* init symbol table */
	int i;
	for(i=0; i<52; i++) {
		symbols[i] = 0;
	}

	return yyparse ( );
}

void yyerror (char *s) {fprintf (stderr, "%s\n", s);} 
```





这里就定义了一些函数，通过 yyparse() 开始执行。



### 结果



下载仓库，直接使用 `make calc` 就能得到一个解析加减的计算器。你可以看看 Makefile 中是怎么生成 yacc 的 c 文件的。



## 参考



+ https://silcnitc.github.io/lex.html
+ https://www.geeksforgeeks.org/introduction-to-yacc/
+ https://www.youtube.com/watch?v=54bo1qaHAfk
+ https://www.youtube.com/watch?v=__-wUHG2rfM
