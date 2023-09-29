# Obfuscated C Code
[Click here for Chinese version](./ReadMe_zh.md)
## Introduction
The author also had a similar idea after learning about the [The International Obfuscated C Code Contest](https://www.ioccc.org/). And that's why this repository is created.

Note that our aim is to show some of the **subtleties** and **infrequent details** of C language. It should be greatly guaranteed that certain "feature" shall not be abused. So those codes that resemble the following is out of our consideration.
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

## Compile
We specify the compiler as gcc because there's some implementation-defined behaviour in the source code.
``` bash
./build.sh
```
## Run
``` bash
./occ
```
## Interpretation of code
Origin source code(code 1):
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
### trigraph sequences
The trigraph sequences is introduced at C89(ANSI C)standard, but deprecated nowadays. However, some compilers, such as gcc, can enable trigraph sequences by certain compile option. The corresponding option is `-trigraphs` in gcc.

The trigraph sequences is like the escape character, which use three characters to express a "escaped" character, as described at the following table:
|trigraph|escaped character|
|---|-|
|??=|#|
|??(|[|
|??/|\|
|??)|]|
|??'|^|
|??<|{|
|??!|\||
|??>|}|
|??-|~|

The code 2 which is the equivalent form of code 1 after trigraph conversion is:
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
### Punctuators of C language
There are six infrequent punctuators still exist util the newest C standard(C23), which are `<: :> <% %> %: %:%:`. And those punctuators are equivalent to `[ ] { } # ##`,respectively.

The code 3 which is the equivalent form of code 2 after infrequent punctuators conversion is:
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

### 'include' of C language
include directive can be the three following forms:
``` C
# include < h-char-sequence > new-line
# include " q-char-sequence " new-line
# include pp-tokens new-line
```
In the form of the third, all macros in `pp-tokens` will be expanded, which result should match one of the two previous forms.And the method by which a sequence of preprocessing tokens between a < and a > preprocessing token pair or a pair of " characters is combined into a single header name preprocessing token is implementation-defined.

In gcc, the result of the expanding of two adjacent macro will be concatenate tightly(with no whitespace insterted), while in msvc the whitespace will be inserted. For example:
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
The code 4 which is the equivalent form of code 3 after the macro expanding is:
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(argc,argv) int argc; char* argv[];{WHAT_THE_FUCK(&3
["\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;}
```

### K&R style function declaration
Early C language allow K&R style function declaration, such as:
``` C
int f(int x, double y);
// K&R style
int f(x,y) int x; int y;
```
However, it's not recommended after the C89 standard and removed at C23 standard.

The code 5 which is the equivalent form of code 4 after K&R style conversion is:
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(int argc，char* argv[]){WHAT_THE_FUCK(&3
["\U00000024\x10\u0040\x61\x72\x65\x20\x79\
\x6f\x75\x20\x6f\x6b\x3f\x3f\x3f"]);return 0;}
```

### Universal Character Name and Hexadecimal Character of C language
Universal Character Name is the way of representation of Unicode in C language, which form is `\uxxxx` or `\Uxxxxxxxx`, where `x` is a hex-digit. The value of Universal Character Name can be any code point of all possible Unicode except those can be represented by ASCII(except for `$` and `@`).

Hexadecimal Character is very simple but not such usual, which represents character by hex-digit. e.g. the ASCII value of `A` is 41, so it can be represented as `\x41` by Hexadecimal Character.

The code 6 which is the equivalent form of code 5 after Universal Character Name and Hexadecimal Character conversion is:
``` C
#define XxxX(foo, bar) foo ## t ## bar
#include <stdio.h>
#define WHAT_THE_FUCK\
	XxxX(pu, s)
int main(int argc，char* argv[]){WHAT_THE_FUCK(&3
["$\x10@are y\
ou ok???"]);return 0;}
```
### Code style Conversion
Let's convert code 6 to usual code style:
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
The code 7 after expanding of `WHAT_THE_FUCK` and `XxxX` is:
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts(&3["$\x10@are you ok???"]);
    return 0;
}
```
### `[]` operator of C language
`a[b]` is completely equivalent to `*((a)+(b))`. The latter form impose no limitation of the order of `a` and `b`, therefore, so does the former.

The code 8 which is the equivalent form of code 7 after exchange the two operator of `[]` is:
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts(&"$\x10@are you ok???"[3]);
    return 0;
}
```
### String Literal of C language is lvalue
The string literal is lvalue, which object type is `char[len of string literal]`. Thus, `"hello"[2]` is equivalent to `char s[] = {'h', 'e', 'l', 'l', 'o', '\0'}; s[2];`.
### Final form
We finally get the final form, i.e. code 9:
``` C
#include <stdio.h>
int main(int argc，char* argv[])
{
    puts("are you ok???");
    return 0;
}
```
Do you understand now `:)`