## 挑战指针
### 目的：只用c语言实现内存写入
### 实现：
1.  mov byte [i] = i & 0x0f
```asm
MOV EDX, i
MOV AL, DL
AND EAX, 0x0f
MOV BYTE [EDX], AL
```
note: EDX本身被赋值，EDX指向的内存地址被赋值

2. 借助一元操作符 * 
```c
*i = i & 0x0f;
```
note: * 对应[]，[]前面的byte没有体现

3. 编译提示：
* invalid type argument of `unary * ： i是普通数值，不可以作为一元操作符*的操作数
* invalid suffix on integer constant ： 待解释
4. 借助指针将i变为特殊数值（内存地址）
```
char *p; // 声明变量p的数值是地址，本身4字节，指向的是字符变量
p = i; // p赋值
```
note: 变量=地址+最大值（大小）+实际值

5. 提示：`assignment makes pointer from integer without a cast`
6. 增加类型转换 p = (char *) i
7. *((char *) i) = i & 0f
8. c设计思想：普通数值和表示内存地址的数值
### 思考
1. 寄存器，地址已知的变量，不用声明直接使用
2. 变量的类型，存储在程序当中，或者说对变量的解释和可允许的操作，是由程序决定的
3. 指针就是一种解释，看待和使用内存数据的方式。比如说在32位系统中，你可以把连续的四个地址存储的值作解释为一个内存地址（指针），然后获取这个地址存储的值，或者给这个地址复制
