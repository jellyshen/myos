## 指针的应用（1）
### 绘制条纹代码变形
```c
int i;
char *p;
p = (char *) 0xa0000;

for (i = 0; i <= 0xffff; i++) {
  *(p + i) = i & 0x0f;
}
```
对应汇编
```asm
    XOR EDX,EDX
L6:
    MOV AL,DL
    AND EAX,15
    MOV BYTE [655360+EDX],AL
    INC EDX
    CMP EDX,65535
    JLE L6
```
## 指针的应用（2）
### 绘制条纹代码变形
```c
int i;
char *p;
p = (char *) 0xa0000;

for (i = 0; i <= 0xffff; i++) {
  p[i] = i & 0x0f;
}
```
对应汇编与上面汇编相同，由此可得`p[i] = *(p + i)`, 另外`p[i] = i[p]`
