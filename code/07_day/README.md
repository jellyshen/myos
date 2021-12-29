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
## 鼠标激活
* 鼠标控制电路包含在键盘控制电路里，键盘控制电路初始化完成，鼠标控制电路的激活也就完成了
* 使用in指令轮询键盘控制电路状态
* 鼠标激活后，即使不动鼠标，也会产生一个中断，这时候读取鼠标数据，会读取掉0xfa
## 从鼠标接受数据
* 鼠标中断处理程序和键盘和相似，除了和pic交互的部分
* bootpack.c先检查键盘缓冲区再检查鼠标缓冲区
## 问题
* 按`Caps lock`，qemu会提示`(cocoa) warning unknown keycode 0xff`，原因？
## 练习
* 整理键盘按键的通码、断码和ascii码表
* 按键时如果有两次中断，同时展示出来
