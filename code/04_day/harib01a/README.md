## 用C语言实现内存写入
* 目标：画图形（数据写入VRAM 320*200 a0000 ~ affff 64kb）
* 实现：
	1. 使用c语言，c语言没有直接写入内存地址的语句 
	2. 汇编实现，c调用汇编
  3. mov指令，哪些寄存器可以使用（EAX、ECX、EDX），如何传参（ESP + 4n） 
    ```
    _write_mem8:	; void write_mem8(int addr, int data);
		MOV		ECX,[ESP+4]		;
		MOV		AL,[ESP+8]		;
		MOV		[ECX],AL
		RET
    ```
	4. 重复执行write_mem8（for循环 c语言实现/自己汇编实现/和编译器对比）
