## 获取按键编码
* 使用out指令PIC中断信号受理完成 `io_out8(PIC0_OCW2, 0x60+IRQ号码)`
* 使用in指令获取外设数据 `io_in8(0x0060)`
* 按下一个键产生的码值叫`通码`，松开一个键产出的叫`断码`，`断码=通码+0x80`
## 加快中断处理
```c
struct KEYBUF {
    unsigned char data, flag;
}
```
* 中断程序只获取数据
* `SII`之后跟`HLT`时，CPU不受理这两条指令之间的中断，同样的还有`mov ss,reg` `mov sp, reg`
* 按右Crtl，产生两个码值`E0 1D`，松开产生两个`E0 9D`
## 制作FIFO缓冲区
```c
struct KEYBUF {
    unsigned char data[32];
    int next;
}
```
* 固定从缓冲区头部消费，需要移位操作
## 改善FIFO缓冲区
```c
struct KEYBUF {
    unsigned char data[32];
    int next_r, next_w, len;
}
```
* ringbuff，环形数组，不需要移位，最多容纳32个数据，再多会丢弃
## 整理FIFO缓冲区
```c
struct FIFO8 {
    unsigned char *buf;
    int p, q, size, free, flags;
}
```
* 实现init, put, get, status方法
