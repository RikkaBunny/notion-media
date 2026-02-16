# Houdini VEX(十九)格式化打印

一、格式化打印
The format string is a simpler version of the C printf format string. When a % symbol is found in the string, an argument will be printed out in a format specified by the characters following the % symbol. The conversion of the argument is specified by a single letter: g, f, s, d, x, c, p.
格式化字符串是一个简单的版本的C printf格式字符串。当字符串中有一个%符号时，一个参数将以字符的形式在%符号后面以指定的格式输出。参数的转换由一个字母指定：g，f，s，d，x，c，p。
You can prefix the format option with an optional prefix characters to control the formatting of the output. The general form of a prefix is [flags][width][.precision][format], where Flags can be:
你可以使用可选前缀字符对格式选项进行前缀，以控制输出格式。前缀的一般形式是[宽度] [精度]格式]，可用标志有：
[Flags]
-: The result will be left justified in the field
-： 结果会被左对齐。
+: A numeric value will be prefixed with either + for positive values. A non- standard behavior of this flag is that string arguments will be quoted when the + flag is set.
+：如果是数值并且为正，将前缀+。如果是字符串，会加上双引号。
0: For numeric values, leading zeros are used to pad the field.
0：对于数值，最前面的零用于填充字段。
Width
宽度
The width can be specified by one or more decimal digits. Alternately, if an asterisk () is given, the width will be taken from the next value in the printf argument list.
宽度 即打印出多少个字符  不够的话用空格或0补(flag为0则补0 否则补空格)
宽度可以用一个或多个十进制数字来指定。另外，如果一个星号（）是给定的，宽度将从在printf函数的参数列表中的下一个值。

Precision
精度
The precision can be specified by one or more decimal digits. Alternately, if an asterisk () is given, the width will be taken from the next value in the printf argument list.
精度可以用一个或多个十进制数字来指定。另外，如果一个星号（）是给定的，宽度将从在printf函数的参数列表中的下一个值。

The different format characters supported are
所支持的不同格式字符如下

%g, %p, %c
Print an integer float, vector, vector4, matrix3, matrix or string in "general" form.
用一般形式打印一个整数，浮点，矢量，Vector4，matrix3，矩阵或字符串

%f, %e, %E
Print a float, vector, vector4, matrix3 or matrix in floating point form.
用浮点形式打印一个浮点数，矢量，Vector4，matrix3或矩阵。

%s
Print a string.
打印字符串。

%d, %i
Print an integer variable in decimal.
用十进制打印整型变量。

%x, %X
Print an integer variable in hexidecimal. The value will be prefixed with "0x" (i.e. 0×42).
用十六进制打印一个整数变量。该值将以“0x”作前缀。

%o
Print an integer variable in octal.
用八进制打印整型变量。

%%
Print a percent sign (%).
打印百分号（%）。

- 示例：
string var_string = "abcdef";
float var_float = 1.23456789;
int var_int = 256;

printf("string: %+1s, float: %10.3f, integer: %-6d \n", var_string, var_float, var_int);
// string = abcdef
// %[+-][length]s 

// %10s -> ____abcdef
// %-10s -> abcdef____
// %+10s -> __"abcdef"
// %+-10s -> "abcdef"__

 
// float = 1.23456789
// %[+-][0][length][precision]f

// %8.3f -> ___1.235
// %-8.3f -> 1.235___
// %08.3f -> 0001.235
// %+8.3f -> __+1.235 (+ shows sign)


// integer = 256
// %[+-][0][length]d

// %6d -> ___256
// %+6d -> __+256
// %-6d -> 256___
// %06d -> 000256

printf("\n");

// escaping characters in string
// from my testing it requires 4 more backslashes to escape \n
// when using raw strings, they are automatically escaped, but @ symbol still
// needs to be escaped
// following lines will output the same thing
// 4 backslashes are needed probably because hscript is parsing this text field
// and sending to vop's field, see the node bellow
string a = 'abc \\\\\n \\\\\t v\@P, %04.2f';
string b = "abc \\\\\n \\\\\t v\@P, %04.2f";
string c = r"abc \n \t v\@P, %04.2f";
string d = R"(abc \n \t v\@P, %04.2f)";

printf(a + "\n");
printf(b + "\n");
printf(c + "\n");
printf(d + "\n"); 

string multiLine = 
R"(It is possible to easily create multi
line strings with this syntax.
In some cases it might
be useful to do it this way,
rather then using \n
However as you have noticed it has weird
4 characters offset starting on the second line,
not sure if it is a bug or feature)";

printf(multiLine + '\n');

// sprintf() 类似printf() 
//而且起到 将输入值转化为字符串，并且返回该字符串的作用
string ss = sprintf('%010d', 20180412);
printf('%s',ss);

