# Go Assembly 补充

by jfxue


## 1 虚拟寄存器 SP

各种材料都提及手写汇编时虚拟寄存器 SP，但却鲜有通过实例来印证该虚拟寄存器用法的，下面用简单的例子来
验证虚拟寄存器 SP 的用法：

### 代码

src/test-virtual-sp/test.go

```go
package main

import (
	"asmpkg"
	"fmt"
)

func main() {
	fmt.Println(asmpkg.Foo(100, 33))
}
```

用汇编语言实现的 package - "asmpkg"

汇编源文件：src/asmpkg/asm_amd64.s

```asm
// func Bar(arg1, arg2 int) int
TEXT ·Bar(SB), NOSPLIT, $0-24
	MOVQ arg1+0(FP), AX
	MOVQ arg2+8(FP), BX
	SUBQ BX, AX
	MOVQ AX, ret+16(FP)
	RET

// func Foo(arg1, arg2 int) int
TEXT ·Foo(SB), NOSPLIT, $40-24
	MOVQ arg1+0(FP), AX
	MOVQ arg2+8(FP), BX
	MOVQ AX, var1-8(SP)
	MOVQ BX, var2-16(SP)
	MOVQ var1-8(SP), AX
	MOVQ var2-16(SP), BX
	MOVQ AX, 0(SP)
	MOVQ BX, 8(SP)
	MOVQ $0, AX
	MOVQ AX, 16(SP)
	CALL ·Bar(SB)
	MOVQ 16(SP), AX
	MOVQ AX, ret+16(FP)
	RET
```

用于声明的 stub 文件：src/asmpkg/stubs.go

```go
package asmpkg

func Bar(arg1, arg2 int) int

func Foo(arg1, arg2 int) int
```

### 分析

首先 build 并运行程序，从输出结果来看是符合预期的
```bash
~/excise/src/test-virtual-sp$ go build test.go 
~/excise/src/test-virtual-sp$ ./test 
67
```

然后我们先来看最终生成的代码（asmpkg.Bar），asmpkg.Foo 非常简单，不详细说明。

使用 gdb 加载调试可执行文件 test，然后像下面这样执行命令(详细参考)

```gdb
(gdb) b main.main
Breakpoint 1 at 0x487d50: file /home/jfxue/excise/src/test-virtual-sp/test.go, line 8.
(gdb) b asmpkg.Foo
Breakpoint 2 at 0x457420: file /home/jfxue/excise/src/asmpkg/asm_amd64.s, line 94.
(gdb) r
Starting program: /home/jfxue/excise/src/test-virtual-sp/test
[New LWP 5433]
[New LWP 5434]
[New LWP 5435]

Thread 1 "test" hit Breakpoint 1, main.main () at /home/jfxue/excise/src/test-virtual-sp/test.go:8
(gdb) disable 1
(gdb) c
Continuing.

Thread 1 "test" hit Breakpoint 2, asmpkg.Foo () at /home/jfxue/excise/src/asmpkg/asm_amd64.s:94
(gdb) 
```

现在调试器停在第二个断点处，即函数 asmpkg.Foo 的第一行指令，这样就得到函数 asmpkg.Foo 的反汇编代码：

```asm
B+> 0x457440 <asmpkg.Foo>       sub    $0x30,%rsp
    0x457444 <asmpkg.Foo+4>     mov    %rbp,0x28(%rsp)
    0x457449 <asmpkg.Foo+9>     lea    0x28(%rsp),%rbp
    0x45744e <asmpkg.Foo+14>    mov    0x38(%rsp),%rax
    0x457453 <asmpkg.Foo+19>    mov    0x40(%rsp),%rbx
    0x457458 <asmpkg.Foo+24>    mov    %rax,0x20(%rsp)
    0x45745d <asmpkg.Foo+29>    mov    %rbx,0x18(%rsp)
    0x457462 <asmpkg.Foo+34>    mov    0x20(%rsp),%rax
    0x457467 <asmpkg.Foo+39>    mov    0x18(%rsp),%rbx
    0x45746c <asmpkg.Foo+44>    mov    %rax,(%rsp)
    0x457470 <asmpkg.Foo+48>    mov    %rbx,0x8(%rsp)
    0x457475 <asmpkg.Foo+53>    xor    %eax,%eax
    0x457477 <asmpkg.Foo+55>    mov    %rax,0x10(%rsp)
    0x45747c <asmpkg.Foo+60>    callq  0x457400 <asmpkg.Bar>
    0x457481 <asmpkg.Foo+65>    mov    0x10(%rsp),%rax
    0x457486 <asmpkg.Foo+70>    mov    %rax,0x48(%rsp)
    0x45748b <asmpkg.Foo+75>    mov    0x28(%rsp),%rbp
    0x457490 <asmpkg.Foo+80>    add    $0x30,%rsp
    0x457494 <asmpkg.Foo+84>    retq
```

下面逐行解释每条指令：

```asm
   sub    $0x30,%rsp            ; (1) 在堆栈上为当前函数保留 48 字节空间
   mov    %rbp,0x28(%rsp)       ; (2) 把 asmpkg.Foo(下简称 Foo) 调用者的 rbp 保留到栈底(48 字节空间的最后 8 字节)
   lea    0x28(%rsp),%rbp       ; (3) 让 rbp 指向栈底元素（就是上面一行保存调用者 rbp 值的堆栈位置），
                                ;     这是传统建立 stack frame 的方法
   mov    0x38(%rsp),%rax       ; (4) 取函数 Foo 的第一个参数 arg1 的值到寄存器 rax
   mov    0x40(%rsp),%rbx       ; (5) 取函数 Foo 的第二个参数 arg2 的值到寄存器 rbx
   mov    %rax,0x20(%rsp)       ; (6) 把 rax 寄存器中的值(即 arg1 的值) 写入局部变量 var1(0x20(rsp)) 中
   mov    %rbx,0x18(%rsp)       ; (7) 把 rbx 寄存器中的值(即 arg2 的值) 写入局部变量 var2(0x18(rsp)) 中
   mov    0x20(%rsp),%rax       ; (9) 从局部变量 var1 中读出 int 值到寄存器 rax 中
   mov    0x18(%rsp),%rbx       ; (8) 从局部变量 var2 中读出 int 值到寄存器 rbx 中
   mov    %rax,(%rsp)           ; (10) 把寄存器 rax 的值（即 var1 的值）保存到当前栈顶 (0(rsp))，用作调用参数
   mov    %rbx,0x8(%rsp)        ; (11) 把寄存器 rbx 的值（即 var2 的值）保存到当前栈顶下一个 (8(rsp))，用作调用参数
   xor    %eax,%eax             ; (12) 初始化 16(rsp) 的值为 0，此空间用于保存调用 asmpkg.Bar 的返回值
   mov    %rax,0x10(%rsp)       ; (13) 实际上在 Bar 函数内初始化该值也可以
   callq  0x457400 <asmpkg.Bar> ; (14) 调用 asmpkg.Bar
   mov    0x10(%rsp),%rax       ; (15) 把 asmpkg.Bar 的返回值（0x10(rsp)）取到寄存器 rax 中
   mov    %rax,0x48(%rsp)       ; (16) 把 rax 的值写入 asmpkg.Foo 返回值的堆栈空间中
   mov    0x28(%rsp),%rbp       ; (17) 从堆栈（0x28(rsp)）中取出在第(2)行保存的 Foo 调用者的 rbp 值
   add    $0x30,%rsp            ; (18) 恢复堆栈指针到刚进入 Foo 函数时的位置，准备返回了
   retq                         ; (19) 返回调用者
```

上述代码已经充分印证了 go 文档中有关虚拟寄存器 FP, SP, 以及真实 SP 寄存器的用法说明是没有问题的，当
然前提是在手写汇编代码中。

Go 工具链的 asm 汇编器生成的汇编文件中虚拟寄存器的使用文档描述不符，它根本不使用虚拟 SP 寄存器，所有
使用 SP 的皆为真实 SP 寄存器，并且 FP 寄存器也与文档描述不符，后面会有详细说明。

堆栈使用的图示（按照传统的画法，上面为栈顶-内存低地址，下面为栈底-内存高地址）：

```
                       |               |
                       |               |
                       |               |
                       |               |            从这里往上的堆栈空间都不是 Bar 的，Bar 没有使用堆栈空间，不需要建立 stack frame
FP(visual FP) --->     ----------------- <------------- 这是进入 Bar 函数后真实 SP 所指位置
                        caller ret addr  <--------- 调用返回地址，即调用 asmpkg.Bar 后面一行代码的地址
                       ----------------- <--------- SP(real SP)  这是真实 SP 所指位置   <--------- 这也是进入 Bar 后 FP 所指位置
                       |     arg1      |            此处存储调用 asmpkg.Bar 的第一个参数 arg1，一般用真实 SP 访问它更方便 arg1+0(SP)
                       -----------------
                       |     arg2      |            此处存储调用 asmpkg.Bar 的第二个参数 arg2，使用真实 SP 访问 arg2+8(SP)
                       -----------------
                       |  Bar 的返回值 |            该空间用于存储调用 asmpkg.Bar 的返回值
                       -----------------
                       |   local var2  |            这是局部变量 var2 使用的空间，使用伪 SP 访问：var2-16(SP)
                       -----------------
                       |   local var1  |            这是局部变量 var1 使用的空间，使用伪 SP 访问：var1-8(SP)
                       ----------------- <--------- SP(visual SP，真实寄存器 BP 也指向这里)
                       |   caller BP   |            这是汇编器生成的附加代码，用于建立 stack frame（与传统方法相同）
FP(visual FP) --->     +---------------+
                        caller ret addr  <--------- 调用返回地址，即调用 asmpkg.Foo 后面一行代码的地址
                       ----------------- <--------- FP(visual FP)
                       |     arg1      |            此处存储调用 asmpkg.Foo 的第一个参数 arg1，使用 arg1+0(FP) 访问
                       -----------------
                       |     arg2      |            此处存储调用 asmpkg.Foo 的第二个参数 arg2，使用 arg2+8(FP) 访问
                       -----------------
                       |  Foo 的返回值 |            此处存储调用 asmpkg.Foo 的返回值，使用 ret+16(FP) 访问
                       -----------------
```

有了上面的堆栈图就更清晰了，需要说明的几点是：

1. 对于使用了堆栈空间的函数，汇编器会自动建立 stack frame，本例是通过代码第 (2), (3), (17) 行来实现的。

   由于汇编器处理了 stack frame，所以手写汇编时就不需要考虑它了，但要记住不能使用真实 SP 来访问调用
   者传递的参数，返回值及局部变量，而是使用 FP 及 虚拟 SP。

   由于建立 stack frame 需要使用一个额外的堆栈单位，所以汇编器会自动调整你写的汇编代码指定的 stack 
   frame 的大小，自动增加 8字节（64位系统），同时这也导致使用真实和虚拟 SP 访问局部变量、返回值代码的偏
   移量做相应调整。对比手写代码和最终生成的代码可以确认这一点。

2. 图中左边标出的 FP 为使用 go 工具链汇编器生成的汇编代码中 FP 所指位置，后面会有进一步说明。

3. 局部变量在堆栈上的布局与传统 C/C++ 是不同的，C/C++ 按照内存地址从低到高的顺序来依次安排局部变量，
   而 Go Assembly 则恰恰相反。

4. Go Assembly 堆栈布局与 C/C++ 还有另一个不同，即调用参数使用栈的方式不同。传统 C/C++ 在使用堆栈传
   递参数（比如 cdecl 调用）时，临时通过调整 SP 为参数分配堆栈空间，当函数返回时根据调用约定的不同由 
   callee 或 caller 来恢复 SP。而 Go Assembly 是在函数入口处就在堆栈中分配好局部变量和函数调用需要的所
   有堆栈空间，当调用其它函数时就使用这个堆栈空间的栈顶部分，而局部变量使用栈底部分，两端向中间增长。

5. 在调用函数时，要在堆栈中压入返回地址，在 Go Assembly 中这个返回地址占用的堆栈空间并不在当前函数的
   栈中，它是在当前函数的栈顶再分配一个堆栈单元，也不能算作 callee 的堆栈空间，这个是特殊的。

看一下汇编器编译输出的汇编代码（生成方法参考[1](../simple-examples-using-go-assembly/README.md#MISC)）：

```asm
"".Bar STEXT nosplit size=19 args=0x18 locals=0x0
    0x0000 00000 (./asm_amd64.s:85) TEXT    "".Bar(SB), NOSPLIT, $0-24
    0x0000 00000 (./asm_amd64.s:85) FUNCDATA    $0, "".Bar.args_stackmap(SB)
    0x0000 00000 (./asm_amd64.s:86) MOVQ    arg1+8(FP), AX
    0x0005 00005 (./asm_amd64.s:87) MOVQ    arg2+16(FP), BX
    0x000a 00010 (./asm_amd64.s:88) SUBQ    BX, AX
    0x000d 00013 (./asm_amd64.s:89) MOVQ    AX, ret+24(FP)
    0x0012 00018 (./asm_amd64.s:90) RET
    0x0000 48 8b 44 24 08 48 8b 5c 24 10 48 29 d8 48 89 44  H.D$.H.\$.H).H.D
    0x0010 24 18 c3                                         $..
"".Foo STEXT nosplit size=85 args=0x18 locals=0x30
	0x0000 00000 (./asm_amd64.s:106)	TEXT	"".Foo(SB), NOSPLIT, $48-24
	0x0000 00000 (./asm_amd64.s:106)	SUBQ	$48, SP
	0x0004 00004 (./asm_amd64.s:106)	MOVQ	BP, 40(SP)
	0x0009 00009 (./asm_amd64.s:106)	LEAQ	40(SP), BP
	0x000e 00014 (./asm_amd64.s:106)	FUNCDATA	$0, "".Foo.args_stackmap(SB)
	0x000e 00014 (./asm_amd64.s:107)	MOVQ	arg1+56(FP), AX
	0x0013 00019 (./asm_amd64.s:108)	MOVQ	arg2+64(FP), BX
	0x0018 00024 (./asm_amd64.s:109)	MOVQ	AX, var1+32(SP)
	0x001d 00029 (./asm_amd64.s:110)	MOVQ	BX, var2+24(SP)
	0x0022 00034 (./asm_amd64.s:111)	MOVQ	var1+32(SP), AX
	0x0027 00039 (./asm_amd64.s:112)	MOVQ	var2+24(SP), BX
	0x002c 00044 (./asm_amd64.s:113)	MOVQ	AX, (SP)
	0x0030 00048 (./asm_amd64.s:114)	MOVQ	BX, 8(SP)
	0x0035 00053 (./asm_amd64.s:115)	MOVQ	$0, AX
	0x0037 00055 (./asm_amd64.s:116)	MOVQ	AX, 16(SP)
	0x003c 00060 (./asm_amd64.s:117)	CALL	"".Bar(SB)
	0x0041 00065 (./asm_amd64.s:118)	MOVQ	16(SP), AX
	0x0046 00070 (./asm_amd64.s:119)	MOVQ	AX, ret+72(FP)
	0x004b 00075 (./asm_amd64.s:120)	MOVQ	40(SP), BP
	0x0050 00080 (./asm_amd64.s:120)	ADDQ	$48, SP
	0x0054 00084 (./asm_amd64.s:120)	RET
	0x0000 48 83 ec 30 48 89 6c 24 28 48 8d 6c 24 28 48 8b  H..0H.l$(H.l$(H.
	0x0010 44 24 38 48 8b 5c 24 40 48 89 44 24 20 48 89 5c  D$8H.\$@H.D$ H.\
	0x0020 24 18 48 8b 44 24 20 48 8b 5c 24 18 48 89 04 24  $.H.D$ H.\$.H..$
	0x0030 48 89 5c 24 08 31 c0 48 89 44 24 10 e8 00 00 00  H.\$.1.H.D$.....
	0x0040 00 48 8b 44 24 10 48 89 44 24 48 48 8b 6c 24 28  .H.D$.H.D$HH.l$(
	0x0050 48 83 c4 30 c3                                   H..0.
	rel 61+4 t=8 "".Bar+0
```

与原始手写代码相比，汇编器生成的代码有几点明显不同：

1. 正如前面提到的，因为生成 stack frame 需要一个额外的堆栈单元，所以可以看到 TEXT 命令的 stack 空间
   大小由手写的 $40 被增加到 $48，同时生成了建立 stack frame 的代码，即偏移 0x0004, 0x0009, 0x004b 处的
   代码。

2. Go 工具链汇编器生成的汇编码代中从不使用 virtual SP，所有使用 SP 的均为真实 SP，这与手写的是不同的。
   对比代码可以看出原始汇编码中使用 virtual SP 的均被转换为使用真实 SP 的代码，这也导致相应代码中指令的
   偏移量发生改变。

3. Go 工具链汇编器生成的汇编代码中 FP 也与文档中描述的完全不同，这里的 FP 所指的位置，总是当前函数的
   栈顶，即函数入口处的真实 SP 的值。

另外，如果当前函数不需要自己的堆栈空间，即没有局部变量，也不需要通过堆栈传递参数或获取返回值（如不调用
其它函数的叶子函数，或者调用其它函数既无参数也无返回值），那么汇编器不会为该函数建立 stack frame。

这样的函数被称为叶子函数（ **Leaf Function** ），关于叶子函数的解释：

> Leaf function，A function that does not require a stack frame. A leaf function does not require a  
  function table entry. It cannot call any functions, allocate space, or save any nonvolatile  
  registers. It can leave the stack unaligned while it executes.
