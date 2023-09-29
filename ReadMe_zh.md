# 混乱的 C 代码
[Click here for English version](./ReadMe.md)
## 介绍
作者在了解到[国际混乱 C 代码大赛](https://www.ioccc.org/)后也有了类似的想法，因而创建了该仓库。

注意，我们的目标在于给出一些 C 语言中**微妙**的地方和一些**生僻的细节**，并尽可能保证不滥用某种“特性”。所以诸如如下做法将不考虑在列：
``` c
#define _ int
#define __ main
#define ___ (
#define ____ )
#define _____ {
#define ______ }
#define _______ printf
#define ________ "hello world"
#define _________ ;
#define __________ <stdio.h>
#include __________
_ __ ___ ____ _____ _______ ___ ________  ____ _________ ______
```

## 编译
因为该代码依赖一些 implementation-defined behaviour，所以我们指定编译器为 gcc。
``` bash
./build.sh
```
## 运行
``` bash
./occ
```
## 代码解释
原代码（代码1）：
``` C
%:define O0oO <std
#define XxxX(foo, bar) foo %:%: t ??=??= bar
??=define Il1L io.h>
%:include O0oO Il1L
??=define WHAT_THE_FUCK??/
	XxxX(pu, s)
int main(argc,argv) int argc; char* argv??(:>;??<WHAT_THE_FUCK(&3
<:"\U00000024\x10\u0040\x61\x72\x65\x20\x79??/
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"??));return 0;%>
```
### 三字符序列 trigraph sequences
三字符序列在C89（ANSI C）标准中被引入，现在已基本弃用，但如 gcc 之类的编译器还是可以通过编译选项开启该功能。gcc 对应的编译选项为 `-trigraphs`。

三字符序列和转义字符类似，其使用三个字符表示一个“转义”后的字符，如下表
|trigraph|escaped character|
|---|-|
|??=|#|
|??(|[|
|??/|\|
|??)|]|
|??'|^|
|??<|{|
|??!|||
|??>|}|
|??-|~|

代码 1 经过 trigraph 转换后的等价形式代码 2 为：
``` C
%:define O0oO <std
#define XxxX(foo, bar) foo %:%: t ## bar
#define Il1L io.h>
%:include O0oO Il1L
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(argc,argv) int argc; char* argv[:>;{WHAT_THE_FUCK(&3
<:"\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;%>
```
### C 语言中的 Punctuators
直至最新的 C 标准（C23）中，有六个罕见的 punctuators 始终被保留，分别是 `<: :> <% %> %: %:%:`，分别等价于 `[ ] { } # ##`。

代码 2 经过 punctuators 转换后的等价形式代码 3 为：
``` C
#define O0oO <std
#define XxxX(foo, bar) foo ## t ## bar
#define Il1L io.h>
#include O0oO Il1L
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(argc,argv) int argc; char* argv[];{WHAT_THE_FUCK(&3
["\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;}
```

### C 语言中的 include
include directive 可以是如下三种形式
``` C
# include < h-char-sequence > new-line
# include " q-char-sequence " new-line
# include pp-tokens new-line
```
第三种形式下 `pp-tokens` 里所有的宏都会被展开，展开后的结果应该符合前两种情况。且第三种情况下如何将 `pp-tokens` 展开后的结果如何拼接成一个头文件名是 implementation-define behaviour，在 gcc 中两个相邻的宏展开后的结果会被紧凑地拼接在一起，而 msvc 则会保留空格，如
``` C
#define A <a
#define B .h>
#include A B
// gcc: equals to #include <a.h>
// msvc: equals to #include <a B
```
``` C
#define A <a
#define B .h>
#define C A B
#include C
// gcc: equals to #include <a.h>
// msvc: equals to #include <a .h>
```
代码 3 经过 include 转换后的等价形式代码 4 为：
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(argc,argv) int argc; char* argv[];{WHAT_THE_FUCK(&3
["\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;}
```

### K&R 风格的函数声明
早期的 C 语言允许 K&R 风格的函数声明，如：
``` C
int f(int x, double y);
// K&R style
int f(x,y) int x; int y;
```
但在 C89 标准中不在推荐，在 C23 标准中彻底移除。

代码 4 经过 K&R 风格转换后的等价形式代码 5 为：
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(int argc，char* argv[]){WHAT_THE_FUCK(&3
["\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;}
```

### C 语言中的 Universal Character Name 与 hexadecimal character
Universal Character Name 是 C 语言中用于表达 Unicode 的方式，其形式为 `\uxxxx` 或 `\Uxxxxxxxx`，其中 `x` 表示一个十六进制数字。其取值可以是所有 Unicode 码点，除去可用 ASCII 表示的字符，再加上 `$` 和 `@`。

Hexadecimal Character 很简单但并不是很常见，就是用十六进制数表示字符。如字符`A`的十六进制 ASCII 值为 41，那么可以用 Hexadecimal Character 表示为 `\x41`。

代码 5 经过 Universal Character Name 和 Hexadecimal Character 转换后的等价形式代码 6 为：
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(int argc，char* argv[]){WHAT_THE_FUCK(&3
["$\x10@are y\
ou ok???"]);return 0;}
```
### 代码风格转换
让我们把代码 6 转换为正常的代码风格
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK XxxX(pu, s)
int main(int argc，char* argv[])
{
    WHAT_THE_FUCK(&3["$\x10@are you ok???"]);
    return 0;
}
```
把宏 `WHAT_THE_FUCK` 和 `XxxX` 展开得到代码 7：
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts(&3["$\x10@are you ok???"]);
    return 0;
}
```
### C 语言中的 `[]` 操作符
`a[b]` 完全等价于 `*((a)+(b))`，而后者对于 `a` 和 `b` 的先后顺序是没要求的，所以 `[]` 操作符也是没要求的。

通过交换`[]`操作数的顺序我们得到代码 8：
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts(&"$\x10@are you ok???"[3]);
    return 0;
}
```
### C 语言中的 string literal 是左值
string literal 是左值，其对象类型为 `char[len of string literal]`。所以 `"hello"[2]` 等价于 `char s[] = {'h', 'e', 'l', 'l', 'o', '\0'}; s[2];`
### 最终形式
我们得到最终的等价代码，代码 9：
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts("are you ok???");
    return 0;
}
```
现在是不是看懂了 `:)`