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
