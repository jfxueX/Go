1. 使用 gdb tui 调试
   可以同时显示 3 个窗口(除必须显示的命令窗口外，还可以显示源码、汇编、寄存器窗口的任意一个或两个)。
   在 ~/.gdbinit 文件中做如下配置以初始化 TUI：

   ```
   tui enable
   layout asm
   layout regs
   set tui border-kind space
   set tui border-mode bold
   tabset 4
   winheight regs -11
   winheight asm +8
   winheight cmd +3
   focus cmd
   refresh
   ```

   配置好之后直接使用 gdb exename 就会自动打开 TUI，并初始化好。

2. 不要把断点设在函数的第一条指令上，因为头部代码检查堆栈可能会多次中断在第一条指令上。

   ```
   B+> 0x48a5d0 <main.main>        mov    %fs:0xfffffffffffffff8,%rcx                          
       0x48a5d9 <main.main+9>      cmp    0x10(%rcx),%rsp
       0x48a5dd <main.main+13>     jbe    0x48a679 <main.main+169>
       0x48a5e3 <main.main+19>     sub    $0x38,%rsp
       0x48a5e7 <main.main+23>     mov    %rbp,0x30(%rsp)
       0x48a5ec <main.main+28>     lea    0x30(%rsp),%rbp
       0x48a5f1 <main.main+33>     lea    0x20588(%rip),%rax        # 0x4aab80
       0x48a5f8 <main.main+40>     mov    %rax,(%rsp)
       0x48a5fc <main.main+44>     callq  0x40f030 <runtime.newobject>
       0x48a601 <main.main+49>     mov    0x8(%rsp),%rax
       0x48a606 <main.main+54>     mov    %rax,0x28(%rsp)
       0x48a60b <main.main+59>     xor    %ecx,%ecx
       0x48a60d <main.main+61>     jmp    0x48a660 <main.main+144>                                                                        
       0x48a60f <main.main+63>     mov    %rcx,0x20(%rsp)
       0x48a614 <main.main+68>     mov    %rax,(%rsp)
       0x48a618 <main.main+72>     movq   $0x1,0x8(%rsp)
       0x48a621 <main.main+81>     callq  0x461130 <sync.(*WaitGroup).Add>
       0x48a626 <main.main+86>     mov    0x28(%rsp),%rax
       0x48a62b <main.main+91>     mov    %rax,0x10(%rsp)
       0x48a630 <main.main+96>     mov    0x20(%rsp),%rcx
       0x48a635 <main.main+101>    mov    %rcx,0x18(%rsp)
       0x48a63a <main.main+106>    movl   $0x10,(%rsp)
       0x48a641 <main.main+113>    lea    0x36e78(%rip),%rdx        # 0x4c14c0
       0x48a648 <main.main+120>    mov    %rdx,0x8(%rsp)            
       0x48a64d <main.main+125>    callq  0x430910 <runtime.newproc>
       0x48a652 <main.main+130>    mov    0x20(%rsp),%rax                                                                                                                                                                                                                       
       0x48a657 <main.main+135>    lea    0x1(%rax),%rcx                                                                                                                                                                                                                        
       0x48a65b <main.main+139>    mov    0x28(%rsp),%rax                                                                                                                                                                                                                       
       0x48a660 <main.main+144>    cmp    $0x5,%rcx                                                                                                                                                                                                                             
       0x48a664 <main.main+148>    jl     0x48a60f <main.main+63>                                                                                                                                                                                                               
       0x48a666 <main.main+150>    mov    %rax,(%rsp)                                                                                                                                                                                                                           
       0x48a66a <main.main+154>    callq  0x4612c0 <sync.(*WaitGroup).Wait>                                                                                                                                                                                                     
       0x48a66f <main.main+159>    mov    0x30(%rsp),%rbp                                                                                                                                                                                                                       
       0x48a674 <main.main+164>    add    $0x38,%rsp                                                                                                                                                                                                                            
       0x48a678 <main.main+168>    retq                                                                                                                                                                                                                                         
       0x48a679 <main.main+169>    callq  0x44f2e0 <runtime.morestack_noctxt>                                                                                                                                                                                                   
       0x48a67e <main.main+174>    jmpq   0x48a5d0 <main.main>                                                                                                                                                                                                                  
       0x48a683                    int3                                                                                                                                                                                                                                         
   ```
   如果断点设在 0x48a5d0 上，那么代码就会在 0x48a5d0 -> 0x48a5d9 -> 0x48a5dd -> 0x48a679 -> 0x48a67e 之间循环多次。所以
   可以一般做法为先把断点设在函数第一条指令上（因为直接设置函数断点就是这样），然后立即禁用该断点，并在跳过堆栈检查的代
   码上设临时断点，本例为 0x48a5e3，然后 continue 这样就可以跳过堆栈检查。具体做法：

   <pre>
   Focus set to cmd window.
   Reading symbols from test_gls...done.
   Loading Go Runtime support.
   (gdb) <b>b main.main</b>
   Breakpoint 1 at 0x48a5d0: file /home/jfxue/excise/src/test_gls/test_gls.go, line 9.
   (gdb) <b>r</b>
   Starting program: /home/jfxue/excise/src/test_gls/test_gls
   [New LWP 9962]
   [New LWP 9963]
   [New LWP 9964]
   
   Thread 1 "test_gls" hit Breakpoint 1, main.main () at /home/jfxue/excise/src/test_gls/test_gls.go:9
   (gdb) <b>disable 1</b>
   (gdb) <b>tb *0x48a5e3</b>
   Temporary breakpoint 2 at 0x48a5e3: file /home/jfxue/excise/src/test_gls/test_gls.go, line 9.
   (gdb) <b>c</b>
   Continuing.
   
   Thread 1 "test_gls" hit Temporary breakpoint 2, 0x000000000048a5e3 in main.main () at /home/jfxue/excise/src/test_gls/test_gls.go:9
   (gdb) <b>p/x $pc</b>
   $1 = 0x48a5e3
   (gdb) 
   </pre>

3. gdb 支持 go 变量，如
   <pre>
   (gdb) <b>p m</b>
   $1 = map[interface {}]interface {}Python Exception <class 'gdb.error'> Attempt to take contents of a non-pointer value.:
   (gdb) <b>p *m</b>
   $3 = {
     count = 842350616512,
     flags = 133 '\205',
     B = 168 '\250',
     noverflow = 72,
     hash0 = 0,
     buckets = 0x49b080,
     oldbuckets = 0x4cbf10 <main.statictmp_1>,
     nevacuate = 4827712,
     extra = 0xc420068030
   }
   (gdb) <b>ptype(m)</b>
   type = struct hash<interface {},interface {}> {
       int count;
       uint8 flags;
       uint8 B;
       uint16 noverflow;
       uint32 hash0;
       struct bucket<interface {},interface {}> *buckets;
       struct bucket<interface {},interface {}> *oldbuckets;
       uintptr nevacuate;
       struct runtime.mapextra *extra;
   } *
   </pre>

   也可以调用 go 的方法，如：
   <pre>
   (gdb) <b>p unsafe.Sizeof(m)</b>
   $4 = 8
   </pre>
   

