#第2章 信息的表示和处理
##1. C语言中移位运算  
左移仅仅在低位补0，右移分为算术位移和逻辑位移两种。算术右移时会在高位补1，如[01100100]算术右移4位为[*1111*0100]；逻辑右移则补0，如[01100100]逻辑右移4位为[*0000*0100]。  
但在Java中，用>>表示算术右移，用>>>表示逻辑右移。  
**注意**：C语言中，对于一个w位的数,如果移动k位(k>=w),在很多机器上则只考虑位移量的低log<sub>2</sub>w位，即k mod w位。但在Java中，已经明确要求按照取模的方式进行移位。
##2. C语言的强制类型转换  
* C语言的强制类型转换并不会改变底层的位模式。例如，考虑如下代码： 

```c
short int v = -12345;
unsigned short uv = (unsigned short) v;
printf("v = %d, uv = %u\n", v, uv);
```
在一台采用补码的机器上会输出：
```c
v = -12345, uv = 53191
```
这是因为，类型为short的有符号数-12345在机器中用二进制补码表示为[1100 1111 1100 0111]<sub>2</sub>，
在将其转换为无符号数uv时，直接把最高位的符号位当成了数值位来计算，即
2<sup>15</sup>+2<sup>14</sup>+2<sup11</sup>+2<sup>10</sup>+2<sup>9</sup>+
2<sup>8</sup>+2<sup>7</sup>+2<sup>6</sup>+2<sup>2</sup>+2<sup>1</sup>+2<sup>0</sup>=53191。
* 在将范围小的数转换成范围大的数时，先进行位的扩展，再进行转换。如：

```c
short x = -12345;
unsigned y = sx;
printf("y = %u:\t", y);
```

结果输出：

```c
uy = 4294954951:
```
其中，4294954951用16进制表示为0XFFFFCFC7，即先进行位扩展，在高位补1后再进行转换。也即``(unsigned)x``等价于
``(unsigned)(int)x``。  
##3. 关于有符号和无符号的建议  
实际编程中建议用有符号数表示，即使这个数不可能为负数。如以下代码：  

```c
float sum_elements(float a[], unsigned length) {
    int i;
    float result = 0;
    for (i = 0; i <= length-1; i++)
        result += a[i];
    return result;
}
```  
注意到这里的`length`是`unsigned`类型变量，如果实际使用时该参数为负数，则会把参数进行强制转换成
一个很大的无符号数（正数），导致数组越界问题。  
##4. 整数加法
* 无符号数运算（只有加法）  
假设无符号数为w位。两个无符号数x和y相加，如果其和超过2<sup>w-1</sup>，则即使w位全都为1也
无法表示这个数，此时就会溢出。由于x和y都是w位的，因此其结果最多为w+1位（假设w=4，则1111+1111=11110）。
如果溢出最高位会被截断，因此最终的结果为x+y-2<sup>w</sup>。
* 补码加法  
补码运算的巧妙之处在于，利用补码可以将减法转换成加一个负数。一个w位的负数，其补码的含义为：最高位
表示-2<sup>w-1</sup>，其余各位分别表示2<sup>i</sup> (0<=i<=w-1),各位表示的数之和就是这个负数的值。  
因此，补码运算可以分为三类：  
**非负数和非负数相加**  
非负数的补码的最高位为0，因此如果非负数之和超过了2<sup>w</sup>，最高位则为1，从补码的含义来看，
他们的和为负数，称为负溢出。  
**负数和负数相加**  
负数补码最高位为1，两个负数相加，不考虑进位，则最高位为0。因此，最高位的其余位之和必须大于
等于2<sup>w-1</sup>,才能保证最高位为1，否则最高位不是1，发生正溢出。即最终结果必须小于-2x2<sup>w-1</sup>+2<sup>w-1</sup>=
-2<sup>w-1</sup>才不会溢出。  
**非负数和负数相加**  
这种情况不会产生溢出，非负数的范围是[0, 2<sup>w-1</sup>-1]，
负数的范围是[-2<sup>w</sup>, 0)，二者之和的范围是[-2<sup>w</sup>, 2<sup>w-1</sup>-1)，均可用w位补码来表示。
因此不会发生溢出。



