## 1. 接收启动信息
### 从asmhead.nas中获取vram，xsize和ysize
* c语言代码
```c
char *vram;
int xsize, ysize;
short *binfo_scrnx, *binfo_scrny;
int *binfo_vram;

binfo_scrnx = (short *) 0x0ff4;
binfo_scrny = (short *) 0x0ff6;
binfo_vram = (int *) 0x0ff8;
xsize = *binfo_scrnx;
ysize = *binfo_scrny;
vram = (char *) *binfo_vram;

init_screen(vram, xsize, ysize);
```
* 对应汇编
```asm
MOVSX EAX,WORD [4086]
MOVSX EDX,WORD [4084]
PUSH  EAX
PUSH  EDX
PUSH  DWORD [4088]
CALL  _init_screen
ADD   ESP,12
```
### MOVSX（有符号扩展并传送）
将源操作数内容复制到目的操作数，并把目的操作数符号扩展到16位或32位。这条指令只用于有符号整数，有三种不同的形式：
```
MOVSX reg32, reg/mem8 
MOVSX reg32, reg/mem16 
MOVSX reg16, reg/mem8 
```
例如：`MOVSX EAX,BYTE [4086]`，假设`BYTE [4086]`的值为`1000 0001`，则EAX为`1111 1111 1111 1111 1111 1111 1000 0001`
## 2. 试用结构体
### 简化1小节中写法
* c语言代码
```c
struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram
}

char *vram;
int xsize, ysize;
struct BOOTINFO *binfo;

binfo = (struct BOOTINFO *) 0x0ff0;
xsize = (*binfo).scrnx;
ysize = (*binfo).scrny;
vram = (*binfo).vram;
init_screen(vram, xsize, ysize);
```
* 汇编
与第1小节中相同
## 3. 试用箭头记号
`xsize = (*binfo).scrnx` 等价于 `xsize = binfo->scrnx`
## 4. 显示字符
### 思路 
16位模式可以使用BIOS显示字符，32位模式 `8*16`长方形像素点阵 16字节
### 显示字符A
```c
static char font_A[16] = {0x00, 0x18, 0x18, 0x18, 0x18, 0x24, 0x24, 0x24, 0x24, 0x7e, 0x42, 0x42, 0x42, 0xe7, 0x00, 0x00};

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
	int i; // 循环变量
	char *p, d; // p 像素点阵行起始内存地址 d 像素点阵行数据
	for (i = 0; i < 16; i++) {
		p = vram + (y + i) * xsize + x;
		d = font[i];
		if ((d & 0x80) != 0) { p[0] = c; }
		if ((d & 0x40) != 0) { p[1] = c; }
		if ((d & 0x20) != 0) { p[2] = c; }
		if ((d & 0x10) != 0) { p[3] = c; }
		if ((d & 0x08) != 0) { p[4] = c; }
		if ((d & 0x04) != 0) { p[5] = c; }
		if ((d & 0x02) != 0) { p[6] = c; }
		if ((d & 0x01) != 0) { p[7] = c; }
	}
	return;
}
```
```asm
_putfont8:
000002C2 55                              	PUSH	EBP
000002C3 89 E5                           	MOV	EBP,ESP
000002C5 57                              	PUSH	EDI
000002C6 56                              	PUSH	ESI
000002C7 31 F6                           	XOR	ESI,ESI                     ; int i = 0 使用ESI做循环变量，并初始化为0
000002C9 53                              	PUSH	EBX
000002CA 8B 7D 1C                        	MOV	EDI,DWORD [28+EBP]          ; EDI = *front 使用EDI存储字符数据地址
000002CD 8A 5D 18                        	MOV	BL,BYTE [24+EBP]            ; BL = c 使用BL存储颜色值
000002D0                              L43:                                ; 外层for循环标记
000002D0 8B 45 14                        	MOV	EAX,DWORD [20+EBP]          ; EAX = y 使用EAX存储字符显示位置（左上角纵坐标）
000002D3 8B 55 10                        	MOV	EDX,DWORD [16+EBP]          ; EDX = x 使用EDX存储字符显示位置（左上角横坐标）
000002D6 01 F0                           	ADD	EAX,ESI                     ; 分步计算表达式p = vram + (y + i) * xsize + x，括号内加法 y = y + i
000002D8 0F AF 45 0C                     	IMUL	EAX,DWORD [12+EBP]        ; 分步计算表达式p = vram + (y + i) * xsize + x，乘法 y = y * xsize
000002DC 03 45 08                        	ADD	EAX,DWORD [8+EBP]           ; 分步计算表达式p = vram + (y + i) * xsize + x，加法 y = y + varm
000002DF 8D 0C 02                        	LEA	ECX,DWORD [EDX+EAX*1]       ; 分步计算表达式p = vram + (y + i) * xsize + x，加法 p = y + x，并将结果存储到ECX
000002E2 8A 14 3E                        	MOV	DL,BYTE [ESI+EDI*1]         ; d = front[i] 使用DL存储字体行数据
000002E5 84 D2                           	TEST	DL,DL                     ; DL AND DL 结果的最高位赋值给SF 这里等价于 d & 0x80 SF singed flag 用来指示结果是否时负数 是1否0
000002E7 79 02                           	JNS	L35                         ; if (SF == 0) 
000002E9 88 19                           	MOV	BYTE [ECX],BL               ; p[0] = c
000002EB                              L35:
000002EB 88 D0                           	MOV	AL,DL
000002ED 83 E0 40                        	AND	EAX,64
000002F0 84 C0                           	TEST	AL,AL
000002F2 74 03                           	JE	L36                         ; if (ZF == 0)
000002F4 88 59 01                        	MOV	BYTE [1+ECX],BL             ; p[i] = c
000002F7                              L36:
000002F7 88 D0                           	MOV	AL,DL
000002F9 83 E0 20                        	AND	EAX,32
000002FC 84 C0                           	TEST	AL,AL
000002FE 74 03                           	JE	L37
00000300 88 59 02                        	MOV	BYTE [2+ECX],BL
00000303                              L37:
00000303 88 D0                           	MOV	AL,DL
00000305 83 E0 10                        	AND	EAX,16
00000308 84 C0                           	TEST	AL,AL
0000030A 74 03                           	JE	L38
0000030C 88 59 03                        	MOV	BYTE [3+ECX],BL
0000030F                              L38:
0000030F 88 D0                           	MOV	AL,DL
00000311 83 E0 08                        	AND	EAX,8
00000314 84 C0                           	TEST	AL,AL
00000316 74 03                           	JE	L39
00000318 88 59 04                        	MOV	BYTE [4+ECX],BL
0000031B                              L39:
0000031B 88 D0                           	MOV	AL,DL
0000031D 83 E0 04                        	AND	EAX,4
00000320 84 C0                           	TEST	AL,AL
00000322 74 03                           	JE	L40
00000324 88 59 05                        	MOV	BYTE [5+ECX],BL
00000327                              L40:
00000327 88 D0                           	MOV	AL,DL
00000329 83 E0 02                        	AND	EAX,2
0000032C 84 C0                           	TEST	AL,AL
0000032E 74 03                           	JE	L41
00000330 88 59 06                        	MOV	BYTE [6+ECX],BL
00000333                              L41:
00000333 83 E2 01                        	AND	EDX,1
00000336 84 D2                           	TEST	DL,DL
00000338 74 03                           	JE	L33
0000033A 88 59 07                        	MOV	BYTE [7+ECX],BL
0000033D                              L33:
0000033D 46                              	INC	ESI
0000033E 83 FE 0F                        	CMP	ESI,15
00000341 7E 8D                           	JLE	L43
00000343 5B                              	POP	EBX
00000344 5E                              	POP	ESI
00000345 5F                              	POP	EDI
00000346 5D                              	POP	EBP
00000347 C3                              	RET
```
[LEA指令](https://www.jianshu.com/p/01e8d5ef369f)
### xor eax, eax 和 mov eax, 0 的区别
* 前者2字节，后者5字节
* 前者执行更快
* 程序优化时，应选择占用空间少，执行速度快的指令
### putfont8重构
```c
void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i, j;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        for (j = 0; j < 8; ++j) {
            if ((d & 2 << (7 - j)) != 0) {
                p[j] = c;
            }
        }
    }
    return;
}
```
