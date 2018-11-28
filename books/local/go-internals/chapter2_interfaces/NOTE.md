
```go
package benchmark

import (
	"testing"
)

type Addifier interface {
	Add(a, b int) int
}

type Adder struct {
	id int
}

//go:noinline
func (adder Adder) Add(a, b int) int {
	return a + b
}

func BenchmarkDirect(b *testing.B) {
	adder := Adder{id: 6754}
	for i := 0; i < b.N; i++ {
		adder.Add(10, 32)
	}
}

func BenchmarkInterface(b *testing.B) {
	adder := Adder{id: 6754}
	for i := 0; i < b.N; i++ {
		Addifier(adder).Add(10, 32)
	}
}

func BenchmarkInterface2(b *testing.B) {
	adder := Adder{id: 6754}
	for i := 0; i < b.N; i++ {
		Addifier(&adder).Add(10, 32)
	}
}
```

```
$ go test -bench=. -benchmem ./escape_test.go 
goos: linux
goarch: amd64
BenchmarkDirect-2       	500000000	         3.45 ns/op	       0 B/op	       0 allocs/op
BenchmarkInterface-2    	50000000	        34.2 ns/op	       8 B/op	       1 allocs/op
BenchmarkInterface2-2   	200000000	         7.27 ns/op	       0 B/op	       0 allocs/op
PASS
ok  	command-line-arguments	6.016s
```

来看这 3 个函数的汇编代码：

**BenchmarkDirect**

```asm
"".BenchmarkDirect STEXT size=111 args=0x8 locals=0x30
	0x0000 00000 (escape_test.go:20)	TEXT	"".BenchmarkDirect(SB), $48-8
	0x0000 00000 (escape_test.go:20)	MOVQ	(TLS), CX
	0x0009 00009 (escape_test.go:20)	CMPQ	SP, 16(CX)
	0x000d 00013 (escape_test.go:20)	JLS	104
	0x000f 00015 (escape_test.go:20)	SUBQ	$48, SP
	0x0013 00019 (escape_test.go:20)	MOVQ	BP, 40(SP)
	0x0018 00024 (escape_test.go:20)	LEAQ	40(SP), BP
	0x001d 00029 (escape_test.go:20)	FUNCDATA	$0, gclocals·a36216b97439c93dafebe03e7f0808b5(SB)
	0x001d 00029 (escape_test.go:20)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x001d 00029 (escape_test.go:20)	MOVL	$0, AX
	0x001f 00031 (escape_test.go:22)	JMP	77
	0x0021 00033 (escape_test.go:22)	MOVQ	AX, "".i+32(SP)
	0x0026 00038 (escape_test.go:23)	MOVQ	$6754, (SP) ; ①  
	0x002e 00046 (escape_test.go:23)	MOVQ	$10, 8(SP)  ; ②  
	0x0037 00055 (escape_test.go:23)	MOVQ	$32, 16(SP) ; ③  
	0x0040 00064 (escape_test.go:23)	PCDATA	$0, $0
	0x0040 00064 (escape_test.go:23)	CALL	"".Adder.Add(SB) ; ④  
	0x0045 00069 (escape_test.go:23)	MOVQ	"".i+32(SP), AX
	0x004a 00074 (escape_test.go:22)	INCQ	AX
	0x004d 00077 (escape_test.go:22)	MOVQ	"".b+56(SP), CX
	0x0052 00082 (escape_test.go:22)	MOVQ	240(CX), DX
	0x0059 00089 (escape_test.go:22)	CMPQ	AX, DX
	0x005c 00092 (escape_test.go:22)	JLT	33
	0x005e 00094 (escape_test.go:25)	MOVQ	40(SP), BP
	0x0063 00099 (escape_test.go:25)	ADDQ	$48, SP
	0x0067 00103 (escape_test.go:25)	RET
	0x0068 00104 (escape_test.go:25)	NOP
	0x0068 00104 (escape_test.go:20)	PCDATA	$0, $-1
	0x0068 00104 (escape_test.go:20)	CALL	runtime.morestack_noctxt(SB)
	0x006d 00109 (escape_test.go:20)	JMP	0
```

直接调用是很简单的情况，①  把 receiver 作为第一个参数放在栈顶，②  和 ③  依次把参
数放入堆栈，④  进行函数调用。

**BenchmarkInterface**

```asm
"".BenchmarkInterface STEXT size=160 args=0x8 locals=0x38
	0x0000 00000 (escape_test.go:27)	TEXT	"".BenchmarkInterface(SB), $56-8
	0x0000 00000 (escape_test.go:27)	MOVQ	(TLS), CX
	0x0009 00009 (escape_test.go:27)	CMPQ	SP, 16(CX)
	0x000d 00013 (escape_test.go:27)	JLS	150
	0x0013 00019 (escape_test.go:27)	SUBQ	$56, SP
	0x0017 00023 (escape_test.go:27)	MOVQ	BP, 48(SP)
	0x001c 00028 (escape_test.go:27)	LEAQ	48(SP), BP
	0x0021 00033 (escape_test.go:27)	FUNCDATA	$0, gclocals·a36216b97439c93dafebe03e7f0808b5(SB)
	0x0021 00033 (escape_test.go:27)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
	0x0021 00033 (escape_test.go:27)	MOVL	$0, AX
	0x0023 00035 (escape_test.go:29)	JMP	123
	0x0025 00037 (escape_test.go:29)	MOVQ	AX, "".i+32(SP)
	0x002a 00042 (escape_test.go:30)	MOVQ	$6754, ""..autotmp_3+40(SP); ①  
	0x0033 00051 (escape_test.go:30)	LEAQ	go.itab."".Adder,"".Addifier(SB), AX; ②  
	0x003a 00058 (escape_test.go:30)	MOVQ	AX, (SP); ③  
	0x003e 00062 (escape_test.go:30)	LEAQ	""..autotmp_3+40(SP), CX; ④  
	0x0043 00067 (escape_test.go:30)	MOVQ	CX, 8(SP); ⑤  
	0x0048 00072 (escape_test.go:30)	PCDATA	$0, $0
	0x0048 00072 (escape_test.go:30)	CALL	runtime.convT2I64(SB); ⑤  
	0x004d 00077 (escape_test.go:30)	MOVQ	24(SP), AX; ⑥  
	0x0052 00082 (escape_test.go:30)	MOVQ	16(SP), CX; ⑦  
	0x0057 00087 (escape_test.go:30)	MOVQ	32(CX), CX; ⑧  
	0x005b 00091 (escape_test.go:30)	MOVQ	$10, 8(SP); ⑨  
	0x0064 00100 (escape_test.go:30)	MOVQ	$32, 16(SP); ⑩  
	0x006d 00109 (escape_test.go:30)	MOVQ	AX, (SP); ⑪ 
	0x0071 00113 (escape_test.go:30)	PCDATA	$0, $0
	0x0071 00113 (escape_test.go:30)	CALL	CX; ⑫  
	0x0073 00115 (escape_test.go:30)	MOVQ	"".i+32(SP), AX
	0x0078 00120 (escape_test.go:29)	INCQ	AX
	0x007b 00123 (escape_test.go:29)	MOVQ	"".b+64(SP), CX
	0x0080 00128 (escape_test.go:29)	MOVQ	240(CX), DX
	0x0087 00135 (escape_test.go:29)	CMPQ	AX, DX
	0x008a 00138 (escape_test.go:29)	JLT	37
	0x008c 00140 (escape_test.go:32)	MOVQ	48(SP), BP
	0x0091 00145 (escape_test.go:32)	ADDQ	$56, SP
	0x0095 00149 (escape_test.go:32)	RET
	0x0096 00150 (escape_test.go:32)	NOP
	0x0096 00150 (escape_test.go:27)	PCDATA	$0, $-1
	0x0096 00150 (escape_test.go:27)	CALL	runtime.morestack_noctxt(SB)
	0x009b 00155 (escape_test.go:27)	JMP	0
```
	
①  创建临时变量 Adder{id: 6754}
②  ~ ⑤   调用 convT2I64 把 Adder{id: 6754} 转换为 Addifier 接口。这里会有一次
内存分配（在循环体内）。
⑥  取出返回值 iface.data 到 AX
⑦  取出返回值 iface.tab 到 CX
⑧  从返回的 itab 中取出接口方法 `Add(a, b int) int` 的函数地址到 CX
⑨ ,⑩  参数 10, 32 入栈
⑪ receiver(即 &Adder{id: 6754}，临时变量的地址) 入栈
⑫ 间接调用接口方法 `Add(a, b int) int`

**BenchmarkInterface2**

```asm
"".BenchmarkInterface2 STEXT size=149 args=0x8 locals=0x38
	0x0000 00000 (escape_test.go:34)	TEXT	"".BenchmarkInterface2(SB), $56-8
	0x0000 00000 (escape_test.go:34)	MOVQ	(TLS), CX
	0x0009 00009 (escape_test.go:34)	CMPQ	SP, 16(CX)
	0x000d 00013 (escape_test.go:34)	JLS	139
	0x000f 00015 (escape_test.go:34)	SUBQ	$56, SP
	0x0013 00019 (escape_test.go:34)	MOVQ	BP, 48(SP)
	0x0018 00024 (escape_test.go:34)	LEAQ	48(SP), BP
	0x001d 00029 (escape_test.go:34)	FUNCDATA	$0, gclocals·09b80ec389a9e6ac09cfa1cd3c45263d(SB)
	0x001d 00029 (escape_test.go:34)	FUNCDATA	$1, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
	0x001d 00029 (escape_test.go:34)	LEAQ	type."".Adder(SB), AX; ① 
	0x0024 00036 (escape_test.go:35)	MOVQ	AX, (SP); ② 
	0x0028 00040 (escape_test.go:35)	PCDATA	$0, $0
	0x0028 00040 (escape_test.go:35)	CALL	runtime.newobject(SB); ③ 
	0x002d 00045 (escape_test.go:35)	MOVQ	8(SP), AX
	0x0032 00050 (escape_test.go:35)	MOVQ	AX, "".&adder+40(SP)
	0x0037 00055 (escape_test.go:35)	MOVQ	$6754, (AX); ④  
	0x003e 00062 (escape_test.go:34)	MOVL	$0, CX
	0x0040 00064 (escape_test.go:36)	JMP	112
	0x0042 00066 (escape_test.go:36)	MOVQ	CX, "".i+32(SP)
	0x0047 00071 (escape_test.go:37)	MOVQ	$10, 8(SP); ⑤  
	0x0050 00080 (escape_test.go:37)	MOVQ	$32, 16(SP); ⑥  
	0x0059 00089 (escape_test.go:37)	MOVQ	AX, (SP); ⑦  
	0x005d 00093 (escape_test.go:37)	PCDATA	$0, $1
	0x005d 00093 (escape_test.go:37)	CALL	"".(*Adder).Add(SB); ⑧  
	0x0062 00098 (escape_test.go:37)	MOVQ	"".i+32(SP), AX
	0x0067 00103 (escape_test.go:36)	LEAQ	1(AX), CX
	0x006b 00107 (escape_test.go:36)	MOVQ	"".&adder+40(SP), AX
	0x0070 00112 (escape_test.go:36)	MOVQ	"".b+64(SP), DX
	0x0075 00117 (escape_test.go:36)	MOVQ	240(DX), BX
	0x007c 00124 (escape_test.go:36)	CMPQ	CX, BX
	0x007f 00127 (escape_test.go:36)	JLT	66
	0x0081 00129 (escape_test.go:39)	MOVQ	48(SP), BP
	0x0086 00134 (escape_test.go:39)	ADDQ	$56, SP
	0x008a 00138 (escape_test.go:39)	RET
	0x008b 00139 (escape_test.go:39)	NOP
	0x008b 00139 (escape_test.go:34)	PCDATA	$0, $-1
	0x008b 00139 (escape_test.go:34)	CALL	runtime.morestack_noctxt(SB)
	0x0090 00144 (escape_test.go:34)	JMP	0
```
①  ~ ③  在堆上分配 Adder 对象，通过命令 `go test -gcflags "-m -m" ./escape_test.go` 
可以看到由于 "receiever in indirect call" 导致 adder 逃逸到堆上，但是还好这不在
循环体内。接下来 ④  进行初始化。 
⑤ , ⑥  参数入栈。
⑦  receiver（在堆上分配出来的） 入栈
⑧  这也是一个间接调用，但是不同于 BenchmarkInterface 的通过接口方法表获取方法地址
的调用。当然这种间接调用相对于直接调用开销也不小。

对比 3 个函数，直接调用的开销最小，BenchmarkInterface 的开销相当于直接调用的 10
倍，这主要是因为每次都有一次内存分配。而 BenchmarkInterface2 也有相当于直接调用
2 倍的开销，这主要由于间接调用的额外开销。
