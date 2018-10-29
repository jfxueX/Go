

asm 实现的 package 命名为 asmpkg，放在 src/asmpkg 目录下。
创建汇编文件 src/asmpkg/asm_amd64.s，内容如下：

```asm
// func sum(n int) (result int)
TEXT ·Sum(SB), $16-16
	MOVQ n+0(FP), AX      // n
	MOVQ result+8(FP), BX // result

	CMPQ AX, $0        // test n - 0
	JG   L_STEP_TO_END // if > 0: goto L_STEP_TO_END
	JMP  L_END         // goto L_STEP_TO_END

L_STEP_TO_END:
	SUBQ $1, AX    // AX -= 1
	MOVQ AX, 0(SP) // arg: n-1
	CALL ·Sum(SB)  // call sum(n-1)
	MOVQ 8(SP), BX // BX = sum(n-1)

	MOVQ n+0(FP), AX      // AX = n
	ADDQ AX, BX           // BX += AX
	MOVQ BX, result+8(FP) // return BX
	RET

L_END:
	MOVQ $0, result+8(FP) // return 0
	RET

// func Syscall_Write(fd int, msg string) int
TEXT ·Syscall_Write(SB), 4, $0-32
	MOVQ $1, AX             // syscall number
	MOVQ fd+0(FP), DI       // arg 1
	MOVQ msg_data+8(FP), SI // arg 2
	MOVQ msg_len+16(FP), DX // arg 3
	SYSCALL
	MOVQ AX, ret+24(FP)
	RET
```

注意，立即数必须加 $，如 Syscall_Write 第一条指令不能写成 `MOVQ 1, AX`。


创建 stub 文件：src/asmpkg/stubs.go，内容如下：

```go
package asmpkg

func Sum(int) int

func Syscall_Write(fd int, msg string) int
```

### 测试程序 1：计算累加和

src/sum/sum.go

```go
package main

import (
	"asmpkg"
	"fmt"
)

func main() {
	fmt.Println(asmpkg.Sum(10))
}
```

### 测试程序 2：通过系统调用打印字符串

src/syscall/syscall.go

```go
package main

import (
	"asmpkg"
)

func main() {
	asmpkg.Syscall_Write(1, "Hello world!\n")
}
```

### 使用 gdb 调试：

1.  build program

    ```bash
    cd src/syscall
    go build syscall.go
    ```

2.  debug with gdbtui
  
    ```bash
    gdbtui syscall
    ```

    ```gdb
    Reading symbols from syscall...done.
    (gdb) layout asm
    (gdb) layout regs
    (gdb) b main.main
    Breakpoint 1 at 0x450a70: file /home/jfxue/excise/src/syscall/syscall.go, line 7.
    (gdb) r
    Starting program: /home/jfxue/excise/src/syscall/syscall
    [New LWP 30243]
    [New LWP 30244]
    [New LWP 30245]
    
    Thread 1 "syscall" hit Breakpoint 1, main.main () at /home/jfxue/excise/src/syscall/syscall.go:7
    (gdb) 
    ```

    两个 layout 命令是为了使 gdb 窗口的 assembler 和 registers 窗口激活。然后设置断点 main.main，第一个 main 是包名，第
    二个是函数名。然后启动程序，可以看到 asm 窗口中当前指令在 main 的第一条语句处。

    asm 窗口部分内容：

    ```asm
    B+> 0x450a70 <main.main>    mov    %fs:0xfffffffffffffff8,%rcx
        0x450a79 <main.main+9>  cmp    0x10(%rcx),%rsp
        0x450a7d <main.main+13> jbe    0x450ab9 <main.main+73>
        0x450a7f <main.main+15> sub    $0x28,%rsp
        0x450a83 <main.main+19> mov    %rbp,0x20(%rsp)
        0x450a88 <main.main+24> lea    0x20(%rsp),%rbp
        0x450a8d <main.main+29> movq   $0x1,(%rsp)
        0x450a95 <main.main+37> lea    0x1e14a(%rip),%rax        # 0x46ebe6
        0x450a9c <main.main+44> mov    %rax,0x8(%rsp)
        0x450aa1 <main.main+49> movq   $0xd,0x10(%rsp)
        0x450aaa <main.main+58> callq  0x450a50 <asmpkg.Syscall_Write>
        0x450aaf <main.main+63> mov    0x20(%rsp),%rbp
        0x450ab4 <main.main+68> add    $0x28,%rsp
        0x450ab8 <main.main+72> retq
        0x450ab9 <main.main+73> callq  0x448830 <runtime.morestack_noctxt>
        0x450abe <main.main+78> jmp    0x450a70 <main.main>
        0x450ac0 <main.init>    mov    %fs:0xfffffffffffffff8,%rcx
        0x450ac9 <main.init+9>  cmp    0x10(%rcx),%rsp
        0x450acd <main.init+13> jbe    0x450b08 <main.init+72>
    ```

    其中第一行显示的 `B` 表示断点被命中至少一次，`+` 表示断点是 enabled 的。`>` 指向为当前指令位置。接着使用 disable 禁
    用该断点，否则这个断点会多次命中（go 进行堆栈空间检查，见 0x450abe 处的代码，会 jump 到第一条指令。）我们直接在调用
    `asmpkg.Syscall_Write` 的指令上下断点：

    ```gdb
   	(gdb) disable 1
	(gdb) b *0x450aaa
	Breakpoint 2 at 0x450aaa: file /home/jfxue/excise/src/syscall/syscall.go, line 8.
	(gdb) 
    ```

    现在 asm 窗口的内容为：

    ```asm
    B-> 0x450a70 <main.main>    mov    %fs:0xfffffffffffffff8,%rcx
        0x450a79 <main.main+9>  cmp    0x10(%rcx),%rsp
        0x450a7d <main.main+13> jbe    0x450ab9 <main.main+73>
        0x450a7f <main.main+15> sub    $0x28,%rsp
        0x450a83 <main.main+19> mov    %rbp,0x20(%rsp)
        0x450a88 <main.main+24> lea    0x20(%rsp),%rbp
        0x450a8d <main.main+29> movq   $0x1,(%rsp)
        0x450a95 <main.main+37> lea    0x1e14a(%rip),%rax        # 0x46ebe6
        0x450a9c <main.main+44> mov    %rax,0x8(%rsp)
        0x450aa1 <main.main+49> movq   $0xd,0x10(%rsp)
    b+  0x450aaa <main.main+58> callq  0x450a50 <asmpkg.Syscall_Write>
        0x450aaf <main.main+63> mov    0x20(%rsp),%rbp
        0x450ab4 <main.main+68> add    $0x28,%rsp
        0x450ab8 <main.main+72> retq
        0x450ab9 <main.main+73> callq  0x448830 <runtime.morestack_noctxt>
        0x450abe <main.main+78> jmp    0x450a70 <main.main>
        0x450ac0 <main.init>    mov    %fs:0xfffffffffffffff8,%rcx
        0x450ac9 <main.init+9>  cmp    0x10(%rcx),%rsp
        0x450acd <main.init+13> jbe    0x450b08 <main.init+72>
    ```
    `B-` 表示 0x450a70 处的断点被命中过，当前为 disabled 状态，而新加的断点还没有被命中。

    在 gdb 命令窗口中输入 c 运行到断点处，查看堆栈中当前的参数：

    ```gdb
	(gdb) x/gx $rsp
	0xc42003df50:   0x0000000000000001
	(gdb) x/gx $rsp+8
	0xc42003df58:   0x000000000046ebe6
	(gdb) x/gx $rsp+16
	0xc42003df60:   0x000000000000000d
	(gdb) x/s 0x000000000046ebe6
	0x46ebe6:       "Hello world!\nSIGKILL: killSIGQUIT: .....
	(gdb)
    ```
    正如预期，第一个参数是 fd(stdout) 的值 1，第二个参数为 string 的 data，第三个参数为 string 的长
    度。还可以从内存中看到字符串的内容为 'Hello world!\n'，而且可以看到不同于 C/C++ 的字符串要以 '\0' 结
    尾。

    然后在 gdb 命令行键入 si 指令 step into `asmpkg.Syscall_Write` 函数：

    ```asm
      > 0x450a50 <asmpkg.Syscall_Write>         mov    $0x1,%rax
        0x450a57 <asmpkg.Syscall_Write+7>       mov    0x8(%rsp),%rdi
        0x450a5c <asmpkg.Syscall_Write+12>      mov    0x10(%rsp),%rsi
        0x450a61 <asmpkg.Syscall_Write+17>      mov    0x18(%rsp),%rdx
        0x450a66 <asmpkg.Syscall_Write+22>      syscall
        0x450a68 <asmpkg.Syscall_Write+24>      mov    %rax,0x20(%rsp)
        0x450a6d <asmpkg.Syscall_Write+29>      retq
        0x450a6e                                int3
        0x450a6f                                int3
    B-  0x450a70 <main.main>                    mov    %fs:0xfffffffffffffff8,%rcx
    ```
    可以看到 `asmpkg.Syscall_Write` 的代码与我们写的汇编伪代码基本完全一致，这是因为这个叶子函数非常简
    单，我们指定了 stack frame 大小为 0，也不需要栈分裂，所以没有生成多余的代码。
 

### MISC

如何让 go 生成汇编输出？

一般使用类似于下面的命令即可：

```bash
go tool compile -S syscall.go
```

但是有时会报告找不到 package，比如在本例使用了汇编语言实现的 package：

```bash
~/excise/src/syscall$ go tool compile -S syscall.go 
syscall.go:4:2: can't find import: "asmpkg"
```

一个懒惰的解决办法是让 build 输出命令，然后手动改一下让其输出汇编代码:

```bash
~/excise/src/syscall$ go build -x syscall.go 
WORK=/tmp/go-build284840720
mkdir -p $WORK/asmpkg/_obj/
mkdir -p $WORK/
cd /home/jfxue/excise/src/asmpkg
/usr/local/go/pkg/tool/linux_amd64/compile -o $WORK/asmpkg.a -trimpath $WORK -goversion go1.9.2
-p asmpkg -buildid 3b037b19154d4865c223d0607481bb58f14df08e -D _/home/jfxue/excise/src/asmpkg -I $WORK 
-pack -asmhdr $WORK/asmpkg/_obj/go_asm.h ./stubs.go

/usr/local/go/pkg/tool/linux_amd64/asm -trimpath $WORK -I $WORK/asmpkg/_obj/ -I /usr/local/go/pkg/
include -D GOOS_linux -D GOARCH_amd64 -o $WORK/asmpkg/_obj/asm_amd64.o ./asm_amd64.s

pack r $WORK/asmpkg.a $WORK/asmpkg/_obj/asm_amd64.o # internal

mkdir -p $WORK/command-line-arguments/_obj/
mkdir -p $WORK/command-line-arguments/_obj/exe/
cd /home/jfxue/excise/src/syscall

/usr/local/go/pkg/tool/linux_amd64/compile -o $WORK/command-line-arguments.a -trimpath $WORK
-goversion go1.9.2 -p main -complete -buildid 4439993cf4f82038735fc5e30a3fdae52c35dcd0 
-D _/home/jfxue/excise/src/syscall -I $WORK -I /home/jfxue/excise/pkg/linux_amd64 -pack ./syscall.go

cd .
/usr/local/go/pkg/tool/linux_amd64/link -o $WORK/command-line-arguments/_obj/exe/a.out -L $WORK
-L /home/jfxue/excise/pkg/linux_amd64 -extld=gcc -buildmode=exe
-buildid=4439993cf4f82038735fc5e30a3fdae52c35dcd0 $WORK/command-line-arguments.a

mv $WORK/command-line-arguments/_obj/exe/a.out syscall
```

可以看到 build 过程是先编译 stubs.go，再使用汇编器编译 asm_amd64.s，然后使用 pack 将 stubs 与
asm 的 obj 打包，然后编译 syscall.go，最后使用链接器把这些 obj 链接成可执行文件。

把上述 build 命令复制到一个 bash 文件中，比如 build.sh，并在编译相应的文件中加入命令行选项 -S 以输出
汇编文件。如：

```bash
...
/usr/local/go/pkg/tool/linux_amd64/compile -S -o $WORK/command-line-arguments.a -trimpath $WORK
-goversion go1.9.2 -p main -complete -buildid 4439993cf4f82038735fc5e30a3fdae52c35dcd0 
-D _/home/jfxue/excise/src/syscall -I $WORK -I /home/jfxue/excise/pkg/linux_amd64 -pack ./syscall.go
...
```

然后执行 bash 脚本重新 build：

```bash
bash build.sh | tee -a syscall.s
```

这样既可得到 syscall.go 通过汇编器编译生成的汇编代码。
