# A Manual for the Plan 9 assembler

Rob Pike

rob@plan9.bell-labs.com

## Machines

There is an assembler for each of the <code>MIPS</code>, <code>SPARC</code>, <code>Intel 386</code>, <code>AMD64</code>, <code>Power PC</code>, and <code>ARM</code>. The <code>68020</code> 
assembler, <code>2a</code>, (no longer distributed) is the oldest and in many ways the prototype. The assemblers 
are really just variations of a single program: they share many properties such as left-to-right 
assignment order for instruction operands and the synthesis of macro instructions such as <code>MOVE</code> to 
hide the peculiarities of the load and store structure of the machines. To keep things concrete, the 
first part of this manual is specifically about the <code>68020</code>. At the end is a description of the 
differences among the other assemblers.

The document, [How to Use the Plan 9 C Compiler](http://doc.cat-v.org/plan_9/4th_edition/papers/comp), by Rob Pike, is a prerequisite for this manual.

## Registers

All pre-defined symbols in the assembler are upper-case. Data registers are <code>R0</code> through <code>R7</code>; address 
registers are <code>A0</code> through <code>A7</code>; floating-point registers are <code>F0</code> through <code>F7</code>.

A pointer in <code>A6</code> is used by the C compiler to point to data, enabling short addresses to be used more 
often. The value of <code>A6</code> is constant and must be set during C program initialization to the address of 
the externally-defined symbol a6base.

The following hardware registers are defined in the assembler; their meaning should be obvious given 
a <code>68020</code> manual: <code>CAAR</code>, <code>CACR</code>, <code>CCR</code>, <code>DFC</code>, <code>ISP</code>, <code>MSP</code>, <code>SFC</code>, <code>SR</code>, <code>USP</code>, and <code>VBR</code>.

The assembler also defines several pseudo-registers that manipulate the stack: <code>FP</code>, <code>SP</code>, and <code>TOS</code>. <code>FP</code> 
is the frame pointer, so 0(<code>FP</code>) is the first argument, 4(<code>FP</code>) is the second, and so on. <code>SP</code> is the 
local stack pointer, where automatic variables are held (<code>SP</code> is a pseudo-register only on the <code>68020</code>); 
0(<code>SP</code>) is the first automatic, and so on as with <code>FP</code>. Finally, <code>TOS</code> is the top-of-stack register, used 
for pushing parameters to procedures, saving temporary values, and so on.

The assembler and loader track these pseudo-registers so the above statements are true regardless of 
what has been pushed on the hardware stack, pointed to by <code>A7</code>. The name <code>A7</code> refers to the hardware 
stack pointer, but beware of mixed use of <code>A7</code> and the above stack-related pseudo-registers, which 
will cause trouble. Note, too, that the <code>PEA</code> instruction is observed by the loader to alter <code>SP</code> and 
thus will insert a corresponding pop before all returns. The assembler accepts a label-like name to 
be attached to <code>FP</code> and <code>SP</code> uses, such as <code>p+0(FP)</code>, to help document that <code>p</code> is the first argument to a 
routine. The name goes in the symbol table but has no significance to the result of the program.

## Referring to data

All external references must be made relative to some pseudo-register, either <code>PC</code> (the virtual 
program counter) or <code>SB</code> (the ‘‘static base’’ register). <code>PC</code> counts instructions, not bytes of data. 
For example, to branch to the second following instruction, that is, to skip one instruction, one 
may write

```asm
    BRA 2(PC)
```

Labels are also allowed, as in

```asm
    BRA return

    NOP

return:

    RTS
```

When using labels, there is no (<code>PC</code>) annotation.

The pseudo-register <code>SB</code> refers to the beginning of the address space of the program. Thus, references 
to global data and procedures are written as offsets to <code>SB</code>, as in

```asm
    MOVL    $array(SB), TOS
```

to push the address of a global array on the stack, or

```asm
    MOVL    array+4(SB), TOS
```

to push the second (4-byte) element of the array. Note the use of an offset; the complete list of 
addressing modes is given below. Similarly, subroutine calls must use <code>SB</code>:

```asm
    BSR exit(SB)
```

<b>File-static variables have syntax</b>

```asm
    local&lt;&gt;+4(SB)
```

<i>The <code>&lt;&gt;</code> will be filled in at load time by a unique integer.</i>

When a program starts, it must execute

```asm
    MOVL    $a6base(SB), A6
```

before accessing any global data. (On machines such as the <code>MIPS</code> and <code>SPARC</code> that cannot load a 
register in a single instruction, constants are loaded through the static base register(<code>SB</code>). The 
loader recognizes code that initializes the static base register and treats it specially. You must 
be careful, however, not to load large constants on such machines when the static base register is 
not set up, such as early in interrupt routines.)

## Expressions

Expressions are mostly what one might expect. Where an offset or a constant is expected, a primary 
expression with unary operators is allowed. <i>A general C constant expression is allowed in 
parentheses.</i>

Source files are preprocessed exactly as in the C compiler, so <code>#define</code> and <code>#include</code> work.

## Addressing modes

The simple addressing modes are shared by all the assemblers. Here, for completeness, follows a 
table of all the <code>68020</code> addressing modes, since that machine has the richest set. In the table, <code>o</code> is 
an offset, which if zero may be elided, and <code>d</code> is a displacement, which is a constant between <code>-128</code> 
and <code>127</code> inclusive. Many of the modes listed have the same name; scrutiny of the format will show 
what default is being applied. For instance, indexed mode with no address register supplied operates 
as though a zero-valued register were used. For "offset" read "displacement." For "<code>.s</code>" read one 
of <code>.L</code>, or <code>.W</code> followed by <code>*1</code>, <code>*2</code>, <code>*4</code>, or <code>*8</code> to indicate the size and scaling of the data.

![](https://9p.io/sys/doc/asm0.png)

## Laying down data

Placing data in the instruction stream, say for interrupt vectors, is easy: the pseudo-instructions 
<code>LONG</code> and <code>WORD</code> (but not <code>BYTE</code>) lay down the value of their single argument, of the appropriate size, 
as if it were an instruction:

```asm
    LONG    $12345
```

places the long <code>12345</code> (base 10) in the instruction stream. (On most machines, the only such operator 
is <code>WORD</code> and it lays down 32-bit quantities. The <code>386</code> has all three: <code>LONG</code>, <code>WORD</code>, and <code>BYTE</code>. The <code>AMD64</code> 
adds <code>QUAD</code> to that for 64-bit values. The <code>960</code> has only one, <code>LONG</code>.)

Placing information in the data section is more painful. The pseudo-instruction <code>DATA</code> does the work, 
given two arguments: an address at which to place the item, including its size, and the value to 
place there. For example, to define a character array array containing the characters <code>abc</code> and a 
terminating null:

```asm
    DATA    array+0(SB)/1, $’a’

    DATA    array+1(SB)/1, $’b’

    DATA    array+2(SB)/1, $’c’

    GLOBL   array(SB), $4
```

or

```asm
    DATA    array+0(SB)/4, $"abc\z"

    GLOBL   array(SB), $4
```

The <code>/1</code> defines the number of bytes to define, <code>GLOBL</code> makes the symbol global, and the <code>$4</code> says how 
many bytes the symbol occupies. Uninitialized data is zeroed automatically. The character <code>\z</code> is 
equivalent to the C <code>\0</code>. The string in a <code>DATA</code> statement may contain a maximum of eight bytes; build 
larger strings piecewise. Two pseudo-instructions, <code>DYNT</code> and <code>INIT</code>, allow the (obsolete) Alef 
compilers to build dynamic type information during the load phase. The <code>DYNT</code> pseudo-instruction has 
two forms:

```asm
    DYNT    , ALEF_SI_5+0(SB)

    DYNT    ALEF_AS+0(SB), ALEF_SI_5+0(SB)
```

In the first form, <code>DYNT</code> defines the symbol to be a small unique integer constant, chosen by the 
loader, which is some multiple of the word size. In the second form, <code>DYNT</code> defines the second symbol 
in the same way, places the address of the most recently defined text symbol in the array specified 
by the first symbol at the index defined by the value of the second symbol, and then adjusts the 
size of the array accordingly.

The <code>INIT</code> pseudo-instruction takes the same parameters as a <code>DATA</code> statement. Its symbol is used as the 
base of an array and the data item is installed in the array at the offset specified by the most 
recent <code>DYNT</code> pseudo-instruction. The size of the array is adjusted accordingly. The <code>DYNT</code> and <code>INIT</code> 
pseudo-instructions are not implemented on the <code>68020</code>.

## Defining a procedure

Entry points are defined by the pseudo-operation <code>TEXT</code>, which takes as arguments the name of the 
procedure (including the ubiquitous (<code>SB</code>)) and the number of bytes of automatic storage to pre-
allocate on the stack, <i>which will usually be zero when writing assembly language programs.</i> On 
machines with a link register, such as the <code>MIPS</code> and <code>SPARC</code>, the special value -4 instructs the loader 
to generate no <code>PC</code> save and restore instructions, even if the function is not a leaf. Here is a 
complete procedure that returns the sum of its two arguments:

```asm
TEXT    sum(SB), $0

    MOVL    arg1+0(FP), R0

    ADDL    arg2+4(FP), R0

    RTS
```

An optional middle argument to the <code>TEXT</code> pseudo-op is a bit field of options to the loader. Setting 
the 1 bit suspends profiling the function when profiling is enabled for the rest of the program. For 
example,

```asm
TEXT    sum(SB), 1, $0

    MOVL    arg1+0(FP), R0

    ADDL    arg2+4(FP), R0

    RTS
```

will not be profiled; the first version above would be. Subroutines with peculiar state, such as 
system call routines, should not be profiled.

Setting the 2 bit allows multiple definitions of the same <code>TEXT</code> symbol in a program; the loader will 
place only one such function in the image. It was emitted only by the Alef compilers.

Subroutines to be called from C should place their result in <code>R0</code>, even if it is an address. Floating 
point values are returned in <code>F0</code>. Functions that return a structure to a C program receive as their 
first argument the address of the location to store the result; <code>R0</code> is unused in the calling protocol 
for such procedures. A subroutine is responsible for saving its own registers, and therefore is free 
to use any registers without saving them (‘‘caller saves’’). <code>A6</code> and <code>A7</code> are the exceptions as 
described above.

## When in doubt

If you get confused, try using the <code>-S</code> option to <code>2c</code> and compiling a sample program. The standard 
output is valid input to the assembler.

## Instructions

The instruction set of the assembler is not identical to that of the machine. It is chosen to match 
what the compiler generates, augmented slightly by specific needs of the operating system. For 
example, <code>2a</code> does not distinguish between the various forms of <code>MOVE</code> instruction: move quick, move 
address, etc. Instead the context does the job. For example,

```asm
    MOVL    $1, R1

    MOVL    A0, R2

    MOVW    SR, R3
```

generates official <code>MOVEQ</code>, <code>MOVEA</code>, and <code>MOVESR</code> instructions. A number of instructions do not have the 
syntax necessary to specify their entire capabilities. Notable examples are the bitfield 
instructions, the multiply and divide instructions, etc. For a complete set of generated instruction 
names (in <code>2a</code> notation, not Motorola’s) see the file <code>/sys/src/cmd/2c/2.out.h</code>. Despite its name, this 
file contains an enumeration of the instructions that appear in the intermediate files generated by 
the compiler, which correspond exactly to lines of assembly language.

## Laying down instructions

The loader modifies the code produced by the assembler and compiler. It folds branches, copies short 
sequences of code to eliminate branches, and discards unreachable code. The first instruction of 
every function is assumed to be reachable. <i>The pseudo-instruction <code>NOP</code>, which you may see in compiler 
output, means no instruction at all, rather than an instruction that does nothing.</i> The loader 
discards all <code>NOP</code>’s.

To generate a true <code>NOP</code> instruction, or any other instruction not known to the assembler, use a <code>WORD</code> 
pseudo-instruction. Such instructions on <code>RISCs</code> are not scheduled by the loader and must have their 
delay slots filled manually.

## MIPS

The registers are only addressed by number: <code>R0</code> through <code>R31</code>. <code>R29</code> is the stack pointer; <code>R30</code> is used as 
the static base pointer, the analogue of <code>A6</code> on the <code>68020</code>. Its value is the address of the global 
symbol <code>setR30(SB)</code>. The register holding returned values from subroutines is <code>R1</code>. When a function is 
called, space for the first argument is reserved at <code>0(FP)</code> but in C (not Alef) the value is passed in 
<code>R1</code> instead.

The loader uses <code>R28</code> as a temporary. The system uses <code>R26</code> and <code>R27</code> as interrupt-time temporaries. 
Therefore none of these registers should be used in user code.

The control registers are not known to the assembler. Instead they are numbered registers <code>M0</code>, <code>M1</code>, 
etc. Use this trick to access, say, <code>STATUS</code>:

```asm
#define STATUS  12

    MOVW    M(STATUS), R1
```

Floating point registers are called <code>F0</code> through <code>F31</code>. By convention, <code>F24</code> must be initialized to the 
value <code>0.0</code>, <code>F26</code> to <code>0.5</code>, <code>F28</code> to <code>1.0</code>, and <code>F30</code> to <code>2.0</code>; this is done by the operating system.

The instructions and their syntax are different from those of the manufacturer’s manual. There are 
no <code>lui</code> and <code>kin</code>; instead there are <code>MOVW</code> (move word), <code>MOVH</code> (move halfword), and <code>MOVB</code> (move byte) 
pseudo-instructions. If the operand is unsigned, the instructions are <code>MOVHU</code> and <code>MOVBU</code>. The order of 
operands is from left to right in dataflow order, just as on the <code>68020</code> but not as in <code>MIPS</code> 
documentation. This means that the <code>Bcond</code> instructions are reversed with respect to the book; for 
example, a va <code>BGTZ</code> generates a <code>MIPS</code> <code>bltz</code> instruction.

The assembler is for the <code>R2000</code>, <code>R3000</code>, and most of the <code>R4000</code> and <code>R6000</code> architectures. It understands 
the 64-bit instructions <code>MOVV</code>, <code>MOVVL</code>, <code>ADDV</code>, <code>ADDVU</code>, <code>SUBV</code>, <code>SUBVU</code>, <code>MULV</code>, <code>MULVU</code>, <code>DIVV</code>, <code>DIVVU</code>, <code>SLLV</code>, <code>SRLV</code>, 
and <code>SRAV</code>. The assembler does not have any cache, load-linked, or store-conditional instructions.

Some assembler instructions are expanded into multiple instructions by the loader. For example the 
loader may convert the load of a 32 bit constant into an <code>lui</code> followed by an <code>ori</code>.

Assembler instructions should be laid out as if there were no load, branch, or floating point 
compare delay slots; the loader will rearrange—schedule—the instructions to guarantee correctness 
and improve performance. The only exception is that the correct scheduling of instructions that use 
control registers varies from model to model of machine (and is often undocumented) so you should 
schedule such instructions by hand to guarantee correct behavior. The loader generates

```asm
    NOR R0, R0, R0
```

when it needs a true no-op instruction. Use exactly this instruction when scheduling code manually; 
the loader recognizes it and schedules the code before it and after it independently. Also, <code>WORD</code> 
pseudo-ops are scheduled like no-ops.

The <code>NOSCHED</code> pseudo-op disables instruction scheduling (scheduling is enabled by default); <code>SCHED</code> re-
enables it. Branch folding, code copying, and dead code elimination are disabled for instructions 
that are not scheduled.

## SPARC

Once you understand the Plan 9 model for the <code>MIPS</code>, the <code>SPARC</code> is familiar. Registers have numerical 
names only: <code>R0</code> through <code>R31</code>. Forget about register windows: Plan 9 doesn’t use them at all. The 
machine has 32 global registers, period. <code>R1</code> [sic] is the stack pointer. <code>R2</code> is the static base 
register, with value the address of <code>setSB(SB)</code>. <code>R7</code> is the return register and also the register 
holding the first argument to a C (not Alef) function, again with space reserved at <code>0(FP)</code>. <code>R14</code> is 
the loader temporary.

Floating-point registers are exactly as on the <code>MIPS</code>.

The control registers are known by names such as <code>FSR</code>. The instructions to access these registers are 
<code>MOVW</code> instructions, for example

```asm
    MOVW    Y, R8
```

for the <code>SPARC</code> instruction

```asm
    rdy %r8
```

Move instructions are similar to those on the <code>MIPS</code>: pseudo-operations that turn into appropriate 
sequences of <code>sethi</code> instructions, adds, etc. Instructions read from left to right. Because the 
arguments are flipped to <code>SUBCC</code>, the condition codes are not inverted as on the <code>MIPS</code>.

The syntax for the ASI stuff is, for example to move a word from ASI 2:

```asm
    MOVW    (R7, 2), R8
```

The syntax for double indexing is

```asm
    MOVW    (R7+R8), R9
```

The <code>SPARC</code>’s instruction scheduling is similar to the <code>MIPS</code>’s. The official no-op instruction is:

```asm
    ORN R0, R0, R0
```

## i960

Registers are numbered <code>R0</code> through <code>R31</code>. Stack pointer is <code>R29</code>; return register is <code>R4</code>; static base is 
<code>R28</code>; it is initialized to the address of <code>setSB(SB)</code>. <code>R3</code> must be zero; this should be done manually 
early in execution by

```asm
    SUBO    R3, R3
```

<code>R27</code> is the loader temporary.

There is no support for floating point.

The Intel calling convention is not supported and cannot be used; use <code>BAL</code> instead. Instructions are 
mostly as in the book. The major change is that <code>LOAD</code> and <code>STORE</code> are both called <code>MOV</code>. The extension 
character for <code>MOV</code> is as in the manual: <code>O</code> for ordinal, <code>W</code> for signed, etc.

## i386

The assembler assumes 32-bit protected mode. The register names are <code>SP</code>, <code>AX</code>, <code>BX</code>, <code>CX</code>, <code>DX</code>, <code>BP</code>, <code>DI</code>, and 
<code>SI</code>. The stack pointer (not a pseudo-register) is <code>SP</code> and the return register is <code>AX</code>. There is no 
physical frame pointer but, as for the <code>MIPS</code>, <code>FP</code> is a pseudo-register that acts as a frame pointer.

Opcode names are mostly the same as those listed in the Intel manual with an <code>L</code>, <code>W</code>, or <code>B</code> appended to 
identify 32-bit, 16-bit, and 8-bit operations. The exceptions are loads, stores, and conditionals. 
All load and store opcodes to and from general registers, special registers (such as <code>CR0</code>, <code>CR3</code>, <code>GDTR</code>, 
<code>IDTR</code>, <code>SS</code>, <code>CS</code>, <code>DS</code>, <code>ES</code>, <code>FS</code>, and <code>GS</code>) or memory are written as

```asm
    MOVx    src,dst
```

where <code>x</code> is <code>L</code>, <code>W</code>, or <code>B</code>. Thus to get <code>AL</code> use a <code>MOVB</code> instruction. If you need to access <code>AH</code>, you must 
mention it explicitly in a <code>MOVB</code>:

```asm
    MOVB    AH, BX
```

There are many examples of illegal moves, for example,

```asm
    MOVB    BP, DI
```

that the loader actually implements as pseudo-operations.

The names of conditions in all conditional instructions (<code>J</code>, <code>SET</code>) follow the conventions of the 68020 
instead of those of the Intel assembler: <code>JOS</code>, <code>JOC</code>, <code>JCS</code>, <code>JCC</code>, <code>JEQ</code>, <code>JNE</code>, <code>JLS</code>, <code>JHI</code>, <code>JMI</code>, <code>JPL</code>, <code>JPS</code>, <code>JPC</code>, 
<code>JLT</code>, <code>JGE</code>, <code>JLE</code>, and <code>JGT</code> instead of <code>JO</code>, <code>JNO</code>, <code>JB</code>, <code>JNB</code>, <code>JZ</code>, <code>JNZ</code>, <code>JBE</code>, <code>JNBE</code>, <code>JS</code>, <code>JNS</code>, <code>JP</code>, <code>JNP</code>, <code>JL</code>, <code>JNL</code>, 
<code>JLE</code>, and <code>JNLE</code>.

The addressing modes have syntax like <code>AX</code>, <code>(AX)</code>, <code>(AX)(BX*4)</code>, <code>10(AX)</code>, and <code>10(AX)(BX*4)</code>. The offsets 
from <code>AX</code> can be replaced by offsets from <code>FP</code> or <code>SB</code> to access names, for example <code>extern+5(SB)(AX*2)</code>.

Other notes: Non-relative <code>JMP</code> and <code>CALL</code> have a <code>*</code> added to the syntax. Only <code>LOOP</code>, <code>LOOPEQ</code>, and <code>LOOPNE</code> 
are legal loop instructions. Only <code>REP</code> and <code>REPN</code> are recognized repeaters. These are not prefixes, but 
rather stand-alone opcodes that precede the strings, for example

```asm
    CLD; REP; MOVSL
```

Segment override prefixes in <code>MOD/RM</code> fields are not supported.

## AMD64

The assembler assumes 64-bit mode unless a <code>MODE</code> pseudo-operation is given:

```asm
    MODE $32
```

to change to 32-bit mode. The effect is mainly to diagnose instructions that are illegal in the 
given mode, but the loader will also assume 32-bit operands and addresses, and 32-bit <code>PC</code> values for 
call and return. The assembler’s conventions are similar to those for the <code>386</code>, above. The 
architecture provides extra fixed-point registers <code>R8</code> to <code>R15</code>. All registers are 64 bit, but 
instructions access low-order <code>8</code>, <code>16</code> and <code>32</code> bits as described in the processor handbook. For example, 
<code>MOVL</code> to <code>AX</code> puts a value in the low-order 32 bits and clears the top 32 bits to zero. Literal 
operands are limited to signed 32 bit values, which are sign-extended to 64 bits in 64 bit 
operations; the exception is <code>MOVQ</code>, which allows 64-bit literals. The external registers in Plan 9’s 
C are allocated from <code>R15</code> down.

There are many new instructions, including the <code>MMX</code> and <code>XMM</code> media instructions, and conditional move 
instructions. <code>MMX</code> registers are <code>M0</code> to <code>M7</code>, and <code>XMM</code> registers are <code>X0</code> to <code>X15</code>. As with the 386 
instruction names, all new 64-bit integer instructions, and the <code>MMX</code> and <code>XMM</code> instructions uniformly 
use <code>L</code> for ‘long word’ (32 bits) and <code>Q</code> for ‘quad word’ (64 bits). Some instructions use <code>O</code> (‘octword’) 
for 128-bit values, where the processor handbook variously uses <code>O</code> or <code>DQ</code>. The assembler also 
consistently uses <code>PL</code> for ‘packed long’ in <code>XMM</code> instructions, instead of <code>Q</code>, <code>DQ</code> or <code>PI</code>. Either <code>MOVL</code> or 
`MOVQ` can be used to move values to and from control registers, even when the registers might be 64 
bits. The assembler often accepts the handbook’s name to ease conversion of existing code (but 
remember that the operand order is uniformly source then destination).

C’s `long long` type is 64 bits, but passed and returned by value, not by reference. More notably, C 
pointer values are 64 bits, and thus `long long` and `unsigned long long` are the only integer types 
wide enough to hold a pointer value. The C compiler and library use the `XMM` floating-point 
instructions, not the old 387 ones, although the latter are implemented by assembler and loader. 
Unlike the 386, the first integer or pointer argument is passed in a register, which is `BP` for an 
integer or pointer (it can be referred to in assembly code by the pseudonym `RARG`). `AX` holds the 
return value from subroutines as before. Floating-point results are returned in `X0`, although 
currently the first floating-point parameter is not passed in a register. All parameters less than 8 
bytes in length have 8 byte slots reserved on the stack to preserve alignment and simplify variable-
length argument list access, including the first parameter when passed in a register, even though 
bytes 4 to 7 are not initialized.

## Power PC

The Power PC follows the Plan 9 model set by the MIPS and SPARC, not the elaborate ABIs. The 32-bit 
instructions of the 60x and 8xx PowerPC architectures are supported; there is no support for the 
older POWER instructions. Registers are `R0` through `R31`. `R0` is initialized to zero; this is done by C 
start up code and assumed by the compiler and loader. `R1` is the stack pointer. `R2` is the static base 
register, with value the address of `setSB(SB)`. `R3` is the return register and also the register 
holding the first argument to a C function, with space reserved at `0(FP)` as on the MIPS. `R31` is the 
loader temporary. The external registers in Plan 9’s C are allocated from `R30` down.

Floating point registers are called `F0` through `F31`. By convention, several registers are initialized 
to specific values; this is done by the operating system. `F27` must be initialized to the value 
`0x4330000080000000` (used by float-to-int conversion), `F28` to the value `0.0`, `F29` to `0.5`, `F30` to `1.0`, 
and `F31` to `2.0`.

As on the MIPS and SPARC, the assembler accepts arbitrary literals as operands to `MOVW`, and also to 
`ADD` and others where ‘immediate’ variants exist, and the loader generates sequences of `addi`, `addis`, 
`oris`, etc. as required. The register indirect addressing modes use the same syntax as the SPARC, 
including double indexing when allowed.

The instruction names are generally derived from the Motorola ones, subject to slight 
transformation: the ‘`.`’ marking the setting of condition codes is replaced by `CC`, and when the 
letter ‘`o`’ represents ‘`OE=1`’ it is replaced by `V`. Thus `add`, `addo`. and `subfzeo`. become `ADD`, `ADDVCC` 
and `SUBFZEVCC`. As well as the three-operand conditional branch instruction `BC`, the assembler 
provides pseudo-instructions for the common cases: `BEQ`, `BNE`, `BGT`, `BGE`, `BLT`, `BLE`, `BVC`, and `BVS`. The 
unconditional branch instruction is `BR`. Indirect branches use `(CTR)` or `(LR)` as target.

Load or store operations are replaced by `MOV` variants in the usual way: `MOVW` (move word), `MOVH` (move 
halfword with sign extension), and `MOVB` (move byte with sign extension, a pseudo-instruction), with 
unsigned variants `MOVHZ` and `MOVBZ`, and byte-reversing `MOVWBR` and `MOVHBR`. ‘Load or store with update’ 
versions are `MOVWU`, `MOVHU`, and `MOVBZU`. Load or store multiple is `MOVMW`. The exceptions are the 
string instructions, which are `LSW` and `STSW`, and the reservation instructions `lwarx` and `stwcx`., 
which are `LWAR` and `STWCCC`, all with operands in the usual data-flow order. Floating-point load or 
store instructions are `FMOVD`, `FMOVDU`, `FMOVS`, and `FMOVSU`. The register to register move instructions 
`fmr` and `fmr.` are written `FMOVD` and `FMOVDCC`.

The assembler knows the commonly used special purpose registers: `CR`, `CTR`, `DEC`, `LR`, `MSR`, and `XER`. The 
rest, which are often architecture-dependent, are referenced as `SPR(n)`. The segment registers of the 
60x series are similarly `SEG(n)`, but n can also be a register name, as in `SEG(R3)`. Moves between 
special purpose registers and general purpose ones, when allowed by the architecture, are written as 
`MOVW`, replacing `mfcr`, `mtcr`, `mfmsr`, `mtmsr`, `mtspr`, `mfspr`, `mftb`, and many others.

The fields of the condition register `CR` are referenced as `CR(0)` through `CR(7)`. They are used by the 
`MOVFL` (move field) pseudo-instruction, which produces `mcrf` or `mtcrf`. For example:

```asm
    MOVFL   CR(3), CR(0)

    MOVFL   R3, CR(1)

    MOVFL   R3, $7, CR
```

They are also accepted in the conditional branch instruction, for example

```asm
    BEQ CR(7), label
```

Fields of the `FPSCR` are accessed using `MOVFL` in a similar way:

```asm
    MOVFL   FPSCR, F0

    MOVFL   F0, FPSCR

    MOVFL   F0, $7, FPSCR

    MOVFL   $0, FPSCR(3)
```

producing `mffs`, `mtfsf` or `mtfsfi`, as appropriate.

## ARM

The assembler provides access to `R0` through `R14` and the `PC`. The stack pointer is `R13`, the link 
register is `R14`, and the static base register is `R12`. `R0` is the return register and also the 
register holding the first argument to a subroutine. The external registers in Plan 9’s C are 
allocated from `R10` down. `R11` is used by the loader as a temporary register. The assembler supports 
the `CPSR` and `SPSR` registers. It also knows about coprocessor registers `C0` through `C15`. Floating 
registers are `F0` through `F7`, `FPSR` and `FPCR`.

As with the other architectures, loads and stores are called `MOV`, e.g. `MOVW` for load word or store 
word, and `MOVM` for load or store multiple, depending on the operands.

Addressing modes are supported by suffixes to the instructions: `.IA` (increment after), `.IB` 
(increment before), `.DA` (decrement after), and `.DB` (decrement before). These can only be used with 
the `MOV` instructions. The move multiple instruction, `MOVM`, defines a range of registers using 
brackets, e.g. `[R0-R12]`. The special `MOVM` addressing mode bits `W`, `U`, and `P` are written in the same 
manner, for example, `MOVM.DB.W`. A `.S` suffix allows a `MOVM` instruction to access user `R13` and `R14` 
when in another processor mode. Shifts and rotates in addressing modes are supported by binary 
operators `<<` (logical left shift), `>>` (logical right shift), `->` (arithmetic right shift), and `@>` 
(rotate right); for example `R7>>R2` or `R2@>2`. The assembler does not support indexing by a shifted 
expression; only names can be doubly indexed.

Any instruction can be followed by a suffix that makes the instruction conditional: `.EQ`, `.NE`, and so 
on, as in the ARM manual, with synonyms `.HS` (for `.CS`) and `.LO` (for `.CC`), for example `ADD.NE`. 
Arithmetic and logical instructions can have a `.S` suffix, as ARM allows, to set condition codes.

The syntax of the `MCR` and `MRC` coprocessor instructions is largely as in the manual, with the usual 
adjustments. The assembler directly supports only the ARM floating-point coprocessor operations used 
by the compiler: `CMP`, `ADD`, `SUB`, `MUL`, and `DIV`, all with `F` or `D` suffix selecting single or double 
precision. Floating-point load or store become `MOVF` and `MOVD`. Conversion instructions are also 
specified by moves: `MOVWD`, `MOVWF`, `MOVDW`, `MOVWD`, `MOVFD`, and `MOVDF`.
