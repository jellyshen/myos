## 分隔源文件
* 对复杂度的管理
将bootpack.c中的一部分逻辑抽离成graphic.c和dsctbl.c
## 整理Makefile
* 普通规则 -> 一般规则
```
bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

%.gas : %.c Makefile
	$(CC1) -o $*.gas $*.c
```
## 整理头文件
* 复用公共部分 bootpack.h
* 头文件：仅有#define和函数声明等组成的文件
## GDT设置后续
### GDTR
GDTR 特殊寄存器，48位，使用LGDT [addr]命令设置，低16位为GDT的字节数-1，例如只有三个段，那么其值等于`3*8-1=23`，高32位为GDT的起始地址。
### 段的组成
三部分：上限（20位）、起始地址（32位）和管理属性（12位）。使用结构体表示段的8字节组成
```c
struct SEGMENT_DESCRIPTOR {
  short limit_low, base_low;
  char base_mid, access_right;
  char limit_high, base_high;
};
```
## 初始化PIC
* CPU只能处理一个中断
* PIC（programmable interrupt controller）：将8个中断信号集合成一个的装置
* PIC内部有许多寄存器，IMR（interrupt mask register）和ICW（initial control word）
* 设置顺序：IMR、ICW1、ICW2、ICW3、ICW4、IMR，其中IMR、ICW2、ICW3、ICW4端口号相同
* CPU受理中断，PIC会发送两个字的数据，被CPU按照指令（int 中断号码）解释并执行
* 0x00-0x1f中断号被BIOS占用
## 中断处理程序制作
### IRQ（Interrupt Request）
键盘是IRQ（中断信号，interrupt request）1，鼠标是IRQ12
### 描述符（Descriptor）
* 描述符的种类有多个，分别有数据段，代码段，系统段。系统段又分为多种，有调用门，中断门，陷阱门，任务门
* 门描述符描述的是一段可执行的代码、一个程序或者一个任务
* 调用门只存在于GDT和LDT中，中断门和陷阱门只存在于IDT中
### 特权级（Privilege Level）
CPL、RPL和DPL
* CPL（Curren Privilege Level）：当前特权级，存在于CS寄存器的后两位
* RPL（Request Privilege Level）：请求特权级，存在于段寄存器（CS/DS/SS等）的后两位
* DPL（Descriptor Privilege Level）：描述符特权级，存在于描述符的第45、46位  

特权校验
* CPU对内存段访问的特权级保护是在段选择子加载（mov 段寄存器、jmp、call等指令使用）的时进行的。当有新的段选择子准备加载到段寄存器时，CPU会根据段寄存器的类型进行相应的校验
* 应用程序只能按照受控的方式来访问操作系统的代码和数据
* 普通跳转和jmp门跳转不能使特权级发生跃迁，即不会引起CPL的变化，堆栈不会切换
* call指令通过调用门跳转到非一致代码段时，CPL会改变（ring3->ring0），是目前唯一的特权级提高方式
* 低特权级的应用程序不能随意的访问高特权级的操作系统内存；没有提供对应的门描述符或者一致性代码段，应用程序无法调用高特权级的程序  

一致代码段和非一致代码段
* 一致代码段就是系统用来共享、提供给低特权级的程序使用调用的代码。非一致代码段是为了避免被低特权级程序访问而被系统保护起来的代码。用户程序只能访问到内核的某些共享的段，不能访问内核不共享的段
### 中断
* CPU处理中断的过程中会屏蔽中断，不接受新的中断直到此次中断处理结束，陷阱的发生并不屏蔽中断，可以接受新的中断
* 中断是异常控制流的一种，中断、陷阱和故障

中断过程
* 从中断信息中取得中断类型码 n
* pushf
* TF = 0, IF=0
* push cs
* push ip
* 获取n对应的中断描述符中的段选择子和偏移，跳转
* push cs

中断门描述符
```c
struct GATE_DESCRIPTOR {
  short offset_low, selector;
  char dw_count, access_right;
  short offset_high;
};
```
access_right
```
- --  - - ---
P DPL S D TYPE
```
* P，0表示该描述子无效，1有效
* S，0表示系统描述子，1段描述子
* D，0表示16位，1表示32位
* TYPE，110中断门，100调用门，111陷阱门

pushad
* 将8个寄存器（EAX、ECX、EDX、EBX、ESP、EBP、ESI、EDI）压入栈中

iretd
* pop ip
* pos cs
* popf

## bootpack.c
```c
void HariMain(void)
{
    char s[40], mcursor[256];
    int mx, my;
}

```
```asm
PUSH EDI
PUSH ESI
PUSH EBX
LEA EBX,DWORD [-316+EBP]
SUB ESP,304
```
使用栈分配空间
## 源文件分割编译后内存布局
|文件/函数/数据|软盘存储位置|运行时位置|字节数(十六进制/十进制)|文件作用说明|
|--|--|--|--|--|
|bootpack.c|`4330 ~ 4b7d`|`280000 ~ 28084d`|84e 2126||
|naskfunc.nas|`4435 ~ 44e8`|`280105 ~ 2801b8`|b4 180||
|graphic.c|`44e9 ~ 48e7`|`2801b9 ~ 2801b8`|3ff 1023||
|destbl.c|`48e8 ~ 4a3b`|`2805b8 ~ 28070b`|154 340||
|int.c|`4a3c ~ 4b74`|`28070c ~ 280844`|139 313||
|sprintf|`4b7e ~ 505a`|`28084e ~ 280d2a`|4dd 1245|标准库函数|
|(%d, %d)|`505c ~ 5064`|`280d2c ~ 280d34`<br/>`310000 ~ 310008`|9|坐标模版打印数据|
|hankaku.txt|`506c ~ 606b`|`280d3c ~ 281d3b`<br/>`310010 ~ 31100f`|1000 4096|字体数据|
|table_rgb|`606c ~ 609b`|`281d3c ~ 281d6b`<br/>`311010 ~ 31103f`|30 48|调色板|
|cursor|`609c ~ 619b`|`281d6c ~ 281e6b`<br/>`311040 ~ 31113f`|100 256|鼠标数据|
|INT 21 (IRQ-1) : PS/2 keyboard|`619c ~ 61ba`|`281e6c ~ 281e8a`<br/>`311140 ~ 31115e`|1f 31|键盘打印数据|
|INT 2C (IRQ-12) : PS/2 mouse|`61bb ~ 61d7`|`281e8b ~ 281ea7`<br/>`31115f ~ 31117b`|1d 29|鼠标打印数据|
|0123456789ABCDEF0123456789abcde|`61d8 ~ 61f7`|`281ea8 ~ 281ec7`<br/>`31117c ~ 31119b`|20 32|？|
