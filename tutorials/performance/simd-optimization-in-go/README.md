[Assembly SIMD Optimization in Go][1]
=====================================

February 3rd, 2016

original link: <https://goroutines.com/asm>

[1]: https://goroutines.com/asm

There's something beautiful about programming in assembly.

With the complexity of modern computers, it's easy to forget the relatively 
simple interface they provide to the programmer at the lowest level. Machine 
code is an almost impervious abstraction over the unavoidably physical 
transistors and capacitors that make up a computer.  The abstractions are still 
human-made, but they are well-tested and reliable. The computers we program are 
machines underneath, and programming in assembly lets you see just how machiney 
they really are.

Assembly programming can be very slow and crash-filled compared to
programming in a language like Go, but very occasionally it's a good
idea or at least a lot of fun. The amazing assembly programming game
[TIS-100](http://www.zachtronics.com/tis-100/), with its [14 page
instruction
manual](http://www.vidarholen.net/contents/junk/files/TIS-100%20Reference%20Manual.pdf),
captures the fun of programming assembly without the mess of doing it on
a real computer with a [3,883 page reference
manual](http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-manual-325462.pdf)
and an arsenal of build tools.

Why waste your time programming in assembly when there are perfectly
good languages to program in? Your father built that language himself
and now it's not good enough for you? He spent years unrolling those
loops by hand before he got put out of a job by -funroll-loops and all
those highfalutin optimizing compilers. Don't make the same mistake he
did.

Even with today's fancy compilers, there are still a few cases where you
might want to write assembly. A few places that stand out are
[cryptography](https://golang.org/src/crypto/aes/asm_amd64.s),
[performance
optimization](https://go-review.googlesource.com/#/c/8968/), or
[accessing things not normally available from the
language](https://golang.org/src/syscall/asm_linux_amd64.s). The most
fun being, of course, performance optimization.

When the performance of some piece of your code actually does matter to
a user, and you've already tried all the easier ways of making it
faster, assembly could be a good place to go next. Although the compiler
might be better at optimizing assembly than you, you know more about
your particular case than the compiler can safely assume.

Writing Assembly in Go

The best place to start here is to write the simplest function possible.
So here's one that adds two int64s together, called
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">add</span>.

</div>

<div>

[download](zip/asm-add-go.zip)

</div>

<div>

``` prettyprint
package main

import "fmt"

func add(x, y int64) int64 {
    return x + y
}

func main() {
    fmt.Println(add(2, 3))
}
```

</div>

<div style="background-color:#DBDBDB;border-radius:0 0 6px 6px;margin:6px 0;padding:10px;position:relative;top:-10px;">

<div style="font-family:Monaco, monospace;font-size:16px;">

go get goroutines.com/asm-add-go

</div>

</div>

<div style="margin: 15px 0; line-height: 1.5;">

To implement this in assembly, you create a separate file called
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">add\_amd64.s</span>
where you put the assembly implementation. I'm using a Macbook for these
examples, so the assembly will be for the AMD64 architecture.

</div>

<div>

[download](zip/asm-add.zip)

</div>

<div>

``` prettyprint
// add.go
package main

import "fmt"

func add(x, y int64) int64

func main() {
    fmt.Println(add(2, 3))
}

// add_amd64.s
TEXT ·add(SB),NOSPLIT,$0
    MOVQ x+0(FP), BX
    MOVQ y+8(FP), BP
    ADDQ BP, BX
    MOVQ BX, ret+16(FP)
    RET
```

</div>

<div style="background-color:#DBDBDB;border-radius:0 0 6px 6px;margin:6px 0;padding:10px;position:relative;top:-10px;">

<div style="font-family:Monaco, monospace;font-size:16px;">

go get goroutines.com/asm-add

</div>

</div>

<div style="margin: 15px 0; line-height: 1.5;">

To run this example, use
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">go
get</span>:

</div>

<div>

``` prettyprint
go get goroutines.com/asm-add
asm-add
```

</div>

<div style="margin: 15px 0; line-height: 1.5;">

The assembly syntax is...obscure at best. There is the [official Go
guide](https://golang.org/doc/asm) and the what appears to be a somewhat
[ancient manual for the Plan 9
assembler](http://doc.cat-v.org/plan_9/4th_edition/papers/asm) that give
some hints as to how the Go assembly language works. The best references
are existing Go assembly code and the compiled assembly versions of Go
functions which you can get with:

</div>

<div>

``` prettyprint
go tool compile -S <go file>
```

</div>

<div style="margin: 15px 0; line-height: 1.5;">

The most important things to know are the function declaration and the
stack layout.

</div>

<div style="margin: 15px 0; line-height: 1.5;">

The magical incantation to start the function is
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">TEXT
·add(SB),NOSPLIT,$0</span>. There is a unicode dot character
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">·</span>
separating the package name from the function name. In this case the
package name is
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">main</span>
so the package is left off of the identifier and the name of the
function is
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">add</span>.
The directive
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">NOSPLIT</span>
means that you don't need to write the size of the arguments as the next
parameter. The
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">$0</span>
constant at the end is where you would need to put the size of the
arguments, but since we have
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">NOSPLIT</span>
we can just leave it as
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">$0</span>.
Why is the
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">(SB)</span>
there after the function name? That's not really clear, but it doesn't
work without it.

</div>

<div style="margin: 15px 0; line-height: 1.5;">

The stack layout puts every argument to the function in order on the
stack starting at the address
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">0(FP)</span>
which means a zero-byte offset from the
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">FP</span>
pointer, continuing on through all the arguments until there is a space
for the return value. For
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">func
add(x, y int64) int64</span>, it looks like
this:

</div>

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA4QAAAFwCAMAAADXDEUvAAAABGdBTUEAALGPC/xhBQAAAAFzUkdCAK7OHOkAAAMAUExURUxpcQABAjZcqjVdqs7/y8//y//U1M3/ys//y//T087/yM//ywAAAAQJFwAAAAAAAAAAAAAAAAAAAAAAAP/T0//U1AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADVcqjVbqjZdqgAAAAMGCwAAAAAAAAAAAAAAAAAAADZdqQAAAAAAADVdqTZcqjZbqDVcqjVdqjVcqDZdqjRcqOrjyTZdqjZcqjVcqs//zDZdqv/S0s3/yf/U1P/V1dD/zAAAADdeq095q9J/TMbSevKvf9D/xMf/zPLV1QAAJQA2fksAAAAAP//NrP/VzFwAAND8vUGRpABBeqfyzP/UxCUAAND2pJ1EAQAAQu7V1YJOAL/5zLPvyUEAAPrV1ZGrx/vS0p6laML/zAAANWWglykqJc7L1QBOfp6FhgAAYKy9f8XG0wAATVVmUaLxzC4AAM7/y1OJienP1HCJbol0dNzI0lAAAKu60erExc/rm5DiydD4qv/QtDUAAABCgGJ5YNzS1UWap9Oehvm6l/3DoodgALeVk8XNdeK8u3yFns17P4Gff0B4qoDVxAAvaExdRGUAALtpLZd/Lq35zAApd2RXWd6QXcKKbWV0j83ikABah3uavfzNxzOPo09/rndjZS5lm2a4tTZDNND+tcixvKeVQmt+ZntPAHfLvrf+zJx8eAA/cl8sAIdobKpWEXu2rSxdeF5PPY+xjsjfmZAyAFqqrng/AHaWfCg6NwBpkvTBsTuEkV6ErJuwy1lyfHAtABopAMraiLXTnYtoJeqhcoxXPrCkUJeZt6uVmk1tX55tT6+JfLaxXEUtAISwmXQAAEs1Jfi0ioNsZZvPswAmb+e0pr7BbEZsl96ql6Tevs7wtjowNrrmtpaXqyV5k+Cbc8GTgU1miwBJZH+JWVKXkUpLVMGkpLCvxaepvStCVXJKMIWYcKa1gq6crWFwXJrCoXJfQW6NtY+lggA7SgAmJlxIAL19V7vJhHBmJQAqSpWLnTkASnZwP6u9iXqCcNvAfL//xZ6xwhoG3VwAAAA8dFJOUwABu4XPZ6xXwFIq1d8GIGp179Ab1SoVMMGARLOZDaNk3BEKPIxgJk2eVqZfuDhvyEbnKhH4jZCmn0RIj2fnip0AACAASURBVHja7J1fqBzVHcd37EtpZ+7s7MDOzs5ukv3f3dokldb6krkQroTk4bZiHkzyEDVISpVbAgUNIj6EPgRrSfqSgg8mVCMt2BLaRAStCoJQRQgUlDypL8VgBCltHwqlc86ZP+fMnPmzudk/434/Prgzuzvn7Nzz2d85v3POplIBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgKXiu7t3774Ht2Gl2eu1gb24DYtj1/r6+k7chpXmO14b2IHbAAnB4rgPEkJCgEgICXEb7iA9e5z5vOL9BwnBEkmoNFuO09LN3BfqruuaJbihfa+eRsbzpvd8CxKC5ZGw6gZYBSRcW/77qdAP082WUIWEYFkk1Aw3onnHJTQtazj3zwQJQakkZA62HGZh+w5LqOR0DGfXHVUrkBCUQ8IBUY9kMZSWmztSuj0Ja/P/VJY9qkBCUBIJSSD0Gyy1cAYSqkt3wyEhWCIJ+e4izWcokBASQsK5R8IaJISEkHCxkTAQr52bRIkkVLptbbsSKu12J3kRrdNpt7XZzqV34rWSVwUSQsK5QCTss4ckp+gUktCkuVSDzT0otm23hBbcbtl2NU/CkUrTsaowgaHowYSJ2sqa2bD6th0U1Wezm1rf7iu0aEq/J9N+5HjXNxquWCtpVSAhJJwXNml+OnnUJI86BSRUJsG0oh0G0z7/sn50Ii07akVzk1xCdsxNWWYGZTWI3+Pw+ibNKq2Fb6/Kqx+iZlcFEkLCuaG5vkPDAq2QLltTuJZMWzqd5dBiXdyu17s1DBZiDJ9oyZtgWyM42+TPZs5skEU+dfKgEZY9ol5lSKgYrkxCeVUgISScIz3WJHW3wIweedFQZb3FaHI/HgqbrDWvuQmCpA57qm92WRQa898HjtVu13u2mhkJSaWHQdFstV2VBmalQVBlEjIHDXusTzgJ5VWBhJBwrujJHlreSw0vCilq6J4uhkKDGZIhIT1gYbERTU6awvJVrZ5REdO3rO7VxXUn5FSNM6gukbDP1YDLjsqrAgkh4XwZFe6L+RI6Ua+TjsyUcHwYuuT9v1sdDAak32gMfKpt7jJtru86DAdnBZe4kXdN/J6wwcwxuNkTMylhnX7LKJWYhClVgYQrx/e9P8Dd35g5e1OKbwXxTSsmYSDrIJzeqPKhkHT2BuFbZJ1cYfRp+z75GZKCcxO+RTVW8zozSMuQsMZfPJIwpSoxds7+j7PHawN3QYXF8YP1ufBDeVDh8hXDQhIGR90wnapxmRBN6JvKpihMN0iriEKsFdpOFTml0KsPWNF1XuCkhG3h0mGZaVWJsWc+f6BdUGFFJaQO1kaFNhTqQptVokFWP4ozYyGcyCRkQ8gAK4yVftrVGBWYNq9S04m2XVaCxZeTlHAgjPdC29KqAgkh4RwldPwOZjvs2OVI2K9IJNTC2caKeBWZhK1kxoY9MQwOHSuvVzqiRVeJ7yqdENH5miUlbAhnQglTqwIJV1HCXXfNnJ1pSRk19KjA2lFTlHDIiaX4IqmVTAlryZbvl9pVwzN2toYmnU5QyTZknW5GdviBaFJCsbMdSpheFYF7Z//HIXmBPVBhJbOjXLvTXCHJmSZhtIBb4/avd4NQSCJr73YlrFR6RrEBKim7qtDMZoeGcpWvWUJCxXXjz08jIbKjkHB2NPmBYNvNmyjTE005bLMTdqDF2nGahNXUMjp2oMMo59ujMWTVpeM64fcs8iTs8RJWl6INQMKVlVAXpubGeZFAlFDnBSPxyOsQ6rFgmpaYyVwf1/MDlJadHlVtdh2SFRoLNc+TUOUTMy1ICBYpoSO01q6b85OGgoRKlIwJMzxK/BeWZAu4e/krU+pG3s9O+fGy5w9DDcH1HAnH4UC4t8hFMpAQEgbmxNKdWlEJdVG4unQRtCJp5ZorjhulGDkdRSqSQctXqIP8NHsyMWNwH9R0xWxUDxKCBXdHXTFEFe2O9uK7LhqyhIoi+wk3R15OOx7psiQ0ue1OrGQ9S0I1KrLOL5R1FpiLgYSQkEBnKCZ8hDLynDU5fYXma0on2lQuNmpDLgOkhu+tN1Tma0OcX8wMUeEWrOBT8PKbopMVf2Fdw//yMCIJ5VWBhJBwfrBFKg3aqbTy18wQ85xevdtuGpIXs3TKQBJrHS/IKXU7NJwZ3PTavtIZBL9yQ3M4ukm6wwqrSye35hYnZEeU0Ok1m83RmuCsajVpSJxE6SJpVSAhJJwj/j7aRqtluPk7CnVxSi22+W4oS2kGW4AN4ep+19UwuK5huFk4ONvPrEuNK0yNRWXTFfb+c4kcVm/+N2ZkVYGEkHCeVMXt5soUEiaCpipL+Hekv1jhCFeyBQldfsNUKvw/+lKN2WNK9tdPuO8OYaW2pCqQEBLOFdNIDW3yIaSvq2Qq3ZAuPtXC5t/ieozDcIWaOvbjmdLjfKjl7egYcIrVY85GEvbEN5DXddkXQy2rKpAQEs5ZQ7tmqIbqNItlCbVupy79fcDUTQjK2rg3bsb17PZ0fdA0xUK7w+Zgrapbs5BBaVpN+YWlVYGEkLB8GAvcmF56ICEkvAPoS/lj25AQrI6EVv52RAAJIeHsUAb5kwoAEkLC2TB2nVYt9zezASSEhDMdCy50P+zXgvsgISS8farp//gDQCQsC/fu2LFjb2lrP2w5TmPSRBjcFju9NrATtwEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADw9eVb3wQrz4/gwWIl3AdWnm/DA0gIICEkBJAQQEIACSEhgIQAEgJICAkBJASQEEBCSAggIYCEABJCQgAJASQEkBASAkgIICGAhJAQQEIACQEkhIQAEgJICCAhJASQEEBCAAkhIYCEABICSAgJy8vGzVMfHIFukHBRrFUn9sS2lFWW8Jzrur96Zfsuv/35M49/8czVDz66HxIuM4qhChgWPe2EJxr9nja/6vTcgOrqSnj6oPf5Dzy1zatcOv5geDN/ErPwyQ8vXvzjEUi4LBK6MVjjrwnn1PqcatPnC11dCX8xhYRbb9+6dX0ref7CQe5ePiZKuPkX79yhJyBhmSR03eFcKqPTsoyJSv/fWOnu6O8fLvbaw0+77m+Sr32LhcFr7z3/PomE4gveJU8degASLpOEhl4NsK1QwoZ35Pvgut051MUMfe8Y5NF4ZRMz/y2emDl8MBHoSH/zGLmBrx8ljz+++U+xO3rBhYTLJ2Gyt1mLzg7pn6w2h7oYke0sQGuYoigkYXzIt2/jIXL7XgyPhecvvQkJl1DCNamE4VkaDdtFLra9vCgpZuQfDMjBABLenoS/JHfvzynv+J0LCcsnYVsmhIzv7dm7jaq0SDHCUNWAhLcl4cYZkoxJGVS+5ULCEkpIj6pFJFxf34aGvOoOayhtSBhj88aNG/fnSfgkyco8lzJz8Wvvuc/egYRlk7BVXMIpNFQUJdEb7bDHlv9tPS6LhFtXXjp16qVXw+N/eYdvnJ/+OuQyFO5a7DQ9ceVLclceefThmISxoPduxmQ/XQjwv4cgYdkknEwjYVENlVjBVa43yjqjiXTQEkdC1skLos8FEop+O9U6lc3jJ8/7gznKC/Ex3gv7Ns8FT56M/D58LJEdpb3RH8uLoQU8t/FyKOHGOyyHCgmXXEJ1OgmLaRiXkEwSTqI5+1Y/OWG/zN3Rv5Mb+bMHoi7fyVemUvBBOnseSShItN878ezmy9G87Wtkev7CsRMnrtH5wGsnGAfot8ClP6TO9dN1AM/S7GkgoXfVnx+FhKVIzFjTSFhEw7iEDe9Yp4/qpDxl5JYpErJ+HsuGbJ2bctnZJl1hRiQ8/eXzHh/KJPyMFvDV1U+phU/t440NuRwER6LYxzcfv3jx4hu3oji55a9KjUlYKg1XVMIJlWI6CfM1jEsYZX8MOlVhJktdagnpWjDqzt+yJgjSFPTC6Hm+b5uQkI4GLwcHL2ZIuJ/G4Y3jwclHXuXDNfl24CU8w15UGg1XYNmanpSQhCTXrkwrYZ6GqRLqrB9aNgn9VSqX2YDwta0pFTzw6BHBOZmEnxwNJ/rIkPHK1bNnz37qvf0/5AHh6kfBqx/bPMP9VV/fCpM4XmdUkHDfxk/fL5WGKyBhNSHh2C0aCOMSZmsok3AYdH7bfoa0TBKyyHTo+ptTDAhlCqZKGCQ8hadPJ5etkec/+Td5xz+ufv4nN4zLNOjRNakbQna0XBqugIR2ZISlaW3L39cQLmy75+4M9qyvF9dQJuEalwVaK52EbGG0W3xAKFcwTcJw28N+fl+EZJ7Qe/6vtH97nc6WPE16pE/43WRWs/+zd/exVV91HMd7o2bR3NInhdKWP9ryEDCbcfvDaOzPhFys1FEGW7RQFShZQKjXscwUrKQq8gfBIpt/sGQhLfIwZ0IH8mBIGJSE2IQREpRIJfwxGImDwILBoZuJ8Z7zezq/p9vfbXt77+l9f/4Y9+HX3u729+r5nXO+59yUb4oitXGpNgynK8La5jorC2a4IkIXUTz9jVzzpSejX7nGhzBzMVxnz1TU6IfQ7l8pVZvjIBiFcEC991x2hLLhtEDt6zUvQnt6na6qROhZyqQPw9Iq4HZSrSyhyBnhl5/ylWGXz5RpFC3dgkbrjtMnLHcGijREaM4AOF2wcRKMQOhOO8ZE6LTGB+VVaLso6h4yj0x3qONAejEsrdFRS2CFWjuWaPpilnxqbIJWObg/CRthvTNbOEuveUKlWxhVMRZCcF1X2JKlUIR9LXERyvEXZWhINIFt3ZLmpkPbZUSjOJL59/5RD0Md+oalhVA0U03+AZlELgMzIQSD64VthGJQtHa+2/zN1hBh6oK/oCXiuA7D11qNjXB5fISdzmyFM3bTtmJx2PvuLcwxqwyMYCsJwkIhrMn5mz05NkFr1WAIwkr7drPbYlZrhvCSXdAyVsVa9mu/CSKUe9QozbGYwYyDsH1Nb+QVMggLXbY2HoQRBDN9wqZyGVEX02zeLG9SirarlR9snl4Izzvn9thT9dkYThChOT6029MytnWff3+zk35xQL+4cUIzgiCMjzCSYPQURcK7kcYsZVGFJgi3LRNjksO98bqFWRjmitC/vcUl708gqge8W9akAgMzuhAEYVyEMQgGEVprCOcppaRGmU4IZQPUutVTyj0+hrki9JcGyPWEfZ7pS69TuYqiO95QLQh1RBiLYAjCWepGh41he48WN0KnZFRWWsdbyBTOMCeEncHxHTnu4yCTXcSBwAHuZL1OBEEYB2FMgiEIy9TNnZJhW7wVNcLzzrzAAVnK3Rez/bQZ3l4dMSORHaGsG7fFj1o7kMrG+PW17oitr61UEepFEIRjI4xNMAzhLGdLi/La0E24ixmh7BBa57qsTVHmCOIxdJsmoWzomMjVMRGa+zYNXU+3tA/fsncgNZceynrvnschJTzqKoqTWhEE4Vh5+qny3F7Z9xJmoWptZX3oNk9FjdAckdyqztnH3+LaZOgev9g/g5ANoVWl03pO3Wrb/DNgjNw367dvpqMRdmhFEIST/sr+l6j1zR3qg9C3hlA2T3E30bYYeltCD8KdWRDam23LOHtxe2YFzwT2JT2sItSJ4PT8VKZkaO1o7dQgbPQ/WOGcOA0htTnFi1AOftx0z3VzNd9ALk3pxo9DWsIB15m3bE0d7ez5xD787HXnweOPgmt6Pe22NX6b+qBLr0+G4aPR8t8yNy9KJpP1C0Ova9nyMCKjww8vPzzm+wS04bv9g4OnR45Os086BGFhA0ICQhASEIKQgJCAkIAQhASEBIQEhCAkICQgJCAEIQEhASEBIQgJCAkICQhBSEBIQEhACEICQgJCAkIQEhASEBIQgpCAkICQgBCEBIQEhASEICQgJCAkIAQhASEBIQEhCAkIySTmmc+Qks8TOCCEEEIIIaRk+4RPkJLP13DA6ChhdBSEBIQEhASEICQgJCAkIAQhASEBIQEhCAkICQgJCEFIQEhASEAIQgJCAkICQhASEBIQEhCCkICQgJCAEIQEhASEBIQgJCAkICQgBCEBIQEhASEI9Uzq3pbLq+AGwkIlUddQX99QUVPKCPcahvHrNyduefju9s33t5+6fH0JCEn8VBp2ZpUswm1LM//7rVsn+F0OrHneeS+f8ynsuTI4OLQKhCQk5VWGm7qSRfhKDgjTww8eXE0HH9+3VHkrX/QibD+ZeaytG4QkJKbBhnrz1JlTypejb62Nd+wLvYbxWvDY82YzeHqk/13REnoPuCiealsBQhLMPHFyLBQdwwaJsWQHZv4bf2DmhaWBhk5cb3aKN/DManF79N4j7+XoPgOEJFtDuMC8KRUyRREPob/L15LqEG/fgHPf8/yBd0BIIgdGM6dGlXLbSIBwfAh3infvRMRXHDRASLK1hNUgnDjC1A4xGBPRqTxvgJBkbQlteHOUVhGEysDmjRs3loyFsEeMyuyOmLnYk3nu9gUQEmku4W/pBMK55s25mZv1+iDcdmvLlkOes/r4oS1bunKeJBdfJXPoqO9h+cDxD8VbtK5rrQ+hr9G7mGWyXxYC/K8DhMRq92Z4H5otTrEKcatO3GrUB6GgYNz0n+w3c2ni1mx41erMybzh7+O90dK+135SHGq/cmdgdFRejS4Pfxn5ArtThx2EqQvmGCoIQWjO1RtWWzg/dIaimC9HReOzrturMoeil/Y1z8vZcxehB9HizAN97Yfd2fc/ien5fZ3r15+W84Gn15tpldegB/4Q+dqyDqBPjp7aCDPf9cerQQhCO83yDEtWiP9Wa1W2JkvNzrj3L2Xuvr4kB4JmCcu2D/szuRKG8LZsB/956pZ8j7a2qGKdHLEbR0Fs9N7mwcHBsw/cHyNtVaX6EGrFEIR5RlhW4ZxOyTKtEMqBf7cpbH/HFhGXoGFselUdwAwglL3BI/adgSwIxQEb3kytsR9c5/Qvr1n1cCrCHeZB2jAEYb4Rli2wTpzaMs0QiubH7ciZDnIh2Nq1ymMuDOF7qx3v4pWOn9q1a9etzJf/W9wQOXXdPvrF9h2KzTNpt+fa1+JB2JLa+K5WDEGYd4QN1mlTVa4ZQknDLopOX7DO9vEQjERoD3h6nt4WLFsTz7/3WHzFP07dfdtwJu5loydrUlOe0VG9GIIwzwgTyiqK+Zoh7Ol1bYjbMVYphBOMQuh8w8XquoiQecLM8+fk9e1Vce+jXvtC+ZKzOCPlm6JIbVyqDUMQTjLCmmDJjGFUL4haUFjck/V7XShirPRmepwEoxAOqPeey45QNpwWqH29ZrMs/0qYtWwSoeePhD4MQTgJKZ8p0zhDVGs3WnfMp+qt3uAc83yYqRdCWafS51Sl7B4vwQiE7lhrTIRb1Qvlt9a2i6LuIfPIdIc6DqQXQxBOQpJGSBLOoEzSnTDUrXZUNIWbVljDltnXA9oE13WFLVkKRdjXEhehHH8xpxKVi2NJc9Oh7TKiURzJ/Hv/qIehDn1DEE5CqiMRKvBMhbP1QrjPagrlAMhAtiPNpUZqazU2wuXxEXYangkSMXbTtmJx2PvuLcwxm3Aj2EqCcJqlKgphndoRNK9I9UIox0Qz56+4Lh3jNM5+7TdBhLJwQLkcFrtZxEHYvqY38goZhNOrT9hULjMz8+tuNm+WN9kT9e7CiYUh16PFvopCrhLqc+bxxs1wggjN+ffdnpaxrfv8+5ud9IsD+sWNE5oRBGF+pyjEsEylc6/JCI6eFjtCefZvuLEnXtloNMNcEfq3t7jkRSguk71d1FRgYEYXgiDMP8K5nqeNcr0QOgtmY5aNRjHMFaG/Nscdp3UmTLxO5SqK7nhDtSAsJYQVnl5gs4aXo04hZtyy0QiGOSHsDI7vyHEfB5nsIg4EDnAn63UiCML8IpQzFIvsnqMRsrS++FfWW6MfG+Jvn+0wvL06YkYiO0K5h6jd8I5aO5Bek62xeQWauhD8gVSEehEEYX4RJszKbTlIMyu0Zqb4EVpr/vpyaj4thm7TJJQNHRO5OiZCc9+moevplvbhW/YOpOaPIeu9ex4bwQkTdRXFSa0IgjC/CM05igzDhgZzGqO6TDuE5vKiXDe3Nhm6X7XYP4OQDaFcp5tRdE7darvHHGYZuW/Wb/tL6FSEHVoRBGGeESofRCFrZxIaIpRXhzdz/jLB0NsSehDuzILQ3mxbxtmL2zMreCawL+lhFaFOBEE46Qj9u8jUKDP5C4NfogFCubPu7vGM6Wz8OKQlHHCdecvW1NHOnk/sw89edx48/ii4ptczgLTJQvhBl16fDAPCvKdmdnVVsipZX5co0xLhNSNsV/p8Z3T44eWHx3yfgDZ8t39w8PTI0Wn2SYcgLGw+p0dDeKSFgBCEhcrB4EcgERCCcIovRsfVIyQgBOEEk07LaW/Du+8hASEIpywXjZHtb3t2YyIgBOGU9wXNgrUVMAEhCAuJ8DaDMiAEYWHy0fbNm/tHLq/GCAhBSEBIQEhACEICQgJCAkIQEhASEBIQgpCAkICQgBCEBIQEhASEICQgJCAkIAQhASEBIQEhCAkICQgJCEFIQEhASECoM8JPk5LPV3BACCGEEEJIqeaZz5KSz1dxUNB84euk5PN5HICQgBCEBIQEhASEICQgJCAkIAQhASEBIQEhCAkICQgJCEFIQEhASEAIQgJCAkICQhASEBIQEhCCkICQgJCAEIQEhASEBIQgJCAkICQgBKHu+YGTqX7lZ3/+8n9eAmEpZ0ZFQ/2ihsrmxtJG+O1XDCutP5nil/5N5kX/MmGFz/7t9/t/u3//y3/+3fdAWKRJ1CaTVfMCD1cYTpKljXCZ80Z8vwCvPFH5K//1I+fn/9Y3vc/98srfrwy9BMKCZ6b89VT6Hp1jKKnON8K/3rkzuX+lJ5VCZ6EQ/nRPLgjD38RfLFN+kT/zIlz5x8xj3/khCAsdq8HzIWy0msBFtVVT0BKK0/y17xYrwpW/Ernz//bOLzSqY4/jyUufsjEmoNbUh5i2QV/72OWwZSEpdH0JRAuuD4u6lw0EJDE+BBETJIqU24dEMSGGi0+JBgOl2GCs0IrBcGktDQpB1IKtLQiVWwq3fbtnfnP+zJyZOTmz/5L2fr8v7j9n5zc7n/Ob+f1+c3Kv/hDScnT+UCWDOMrd4K2JxX8zTyi//wF76+AxQLi5ene7o4MwRa+9naLHna01h/CAcpHeevpgEyC0CcxoB3GM9rPPh9jj6bOfy8vRUQcQbr7amoKFigxhK3tpZ7hrrAOEHwLC6g9iro/9kFNhnFdZjALCzVYL/Qpd26IQNrOX99UvRQEIazSIWfZDvjb8hwsOINwi+8HWPQ0KhLt0wRhA+JeDkBzhacOe8qEDCLeGJ9zV5rHYEt0RtlUFwulT4+On5JDd9Lj7UjUg1LTNmz+V3woQauysphQ7NYM4xqIyy4bI6zfue5cXAGHdM4KN8u6uIxU4RBHCLvf59koqZsZuDH533/338T1+sX0ezJbeZ9fplU9eKvPnkLYJL4RwY/DGT3Ftj3qvrtGrhfNxEUX3s4PnuyNfFr6ytPrLgNOz/uLqoNgDDYTGLhrttBDrJEnqQ5ydmkGcjMn1Ux3AoT5AWG8G3XFvNqxKWyKr0ZZKIGS7jYvpkY+DHLE/N56GKbdH4vw5qQT2eBOaZ/q2s/Rq7wP/1X+cMffua/aB4eiEnPUcxAMxQXoxDkJjF412JkqIXKK+Z7V9iLNTHURajWb0X0PtD+cECHMLPIgKCLcEhCxtsacSCNl8nREKTfxV0qQ4wY/T7Bw9efToLcplrR/l6ln2m8hIDWbi2mZzaqa3L2z9C/OStPebkDnu0lxkenjK+uFhsYvSDFYgNHZRZ2diBAd49jyr7YPBTsMgjnxpTPVTGcBMWoLQfXx5CBBuDQjZxxx33dq4u72pqfWtneVBeOdz1syL4oOBAEI+se5cGz9LDN1OS5NNLkuJhVBtmzVzmzk05z+D3E3E1JlMRupEWJNX8j6Orn4eLM79Uj6EOjsTI+h1beTei8XFxW91EKp2GgaR+UaG2PSzX9fX/3X1VXdkMXpCgdAKQ0BYQwjb2I/R0LAv+EF3lwMh07w7nUaCeAFdfQu0b6JreWTZZQGh2rbfTOGl/2QqPo4ooEH9Wg4zZ/MlDWMWEGrtTIyg4/QfkyKYCoSqnYZBzNKX5y75rxWeiCty5iNVCC0wBIQ1hrCVNoa+3iwPwkfdUtDuglAISRywiTI2VywW59zZ99+ip7lSEggjbXvT8JMhzW5Nv2cN0ZgMdpa0jJztTlcGodbOxAj2nD8hM6eDMGKnYRDZh0+La1fnUV6+CokQpnNnr1thCAhrDWE7FW5ve6eV5/PLgfBKtxQ5H5G2YhQL8fdtzKOdzqcTrfW0bXuT8/szBgeiK6icEgtHpoK5ebw7XRmEMXZaImiEUGunOojswz98Retrb3nNE/e5Bf+yk5Ojo3YYAsLaQsgrStuC6hknZQ+h52qCuuKstFPLikFTTYorHsJo27zxYJ+XdeJrUcNZGCzafEcYBjLKhTDGTksETRDq7VQH0X37Lq1vaWn8mO13CyXJzlw0RZE7eyAxhoCwXAjbEu4JXRC9nOJe9mSbPYTePOw9t3LuVd57sVA65elp/PzZAMJo25FtYNbZIPlPH18Ot0K3AzSv5CuFMMZOSwRNEOrt1EIorF3To4cpIsqjT68D4yNHmZJjCAhtlNpB6mBOravDe7IhhHvEt5usIZzVZYdlBdsyWwhntVQFC8mNIaQlKM9ijAauhQIqR9KVQhhjpyWCBggNdhohHBa3wh8eok3iTe9a0yfFgewwBIQ2anI0aoxPUTj75Sq2Rus8YVpX0C8puAbbQjijna0z6cQQ0oqMlmZCop55iMJwpRDG2WlG0FDjo4XQYKc6iDybGmZMmYEHS4Rm/3dXScwpTrAH92UMk+wNAaGNWsuBsFnJG9pBmKkhhJkNZuvGEJLXy3i5NM9XZOVqPNo4yAAABLxJREFU5npAGHy4ZF42ZxLaqYHwpHw3APaJg8eyutkQDSZTdMnRuElAWKa2lwNhm+xJqwNhRrhzmXDzsvpDSEuz/jO8ueN5HYQLFUBosFNDYfzSr0IIDzhSATdbhCeCsPfSYfMSGRCWtyfcmyKxu8l08oepvTFla5Se2FFlT0jTfjbxKRypCbouVxdCik5M8ZaPpDUQkhtJDmHYxRg7rTGsDEKKNAmlQ1RBU3q4OBFokSqP2IPXtggCwpqmKBp2y55wb1mBmYxuG9Z/xgjh6RgIP3WqDqF3iuBr4ZxBVlw58gmcHMKwizF2WmNoC2FkECdlT8jSo3K6JKcGZpIiCAhrC2FKTkrsU4/4lgOh53tMEEZCiBeECMRTpwYQ0pnWl33CyXPyfVNSDXZmo1MUmi7G2GmNoS2EkUGk84S3JSbVcxbSjjU2VgsI6wghj+Q0iqvRzsohJN8iXXbz0tJvWDlv5FWauA9/qz6E3h1WhC5RLzxP8TTBKQpDF2PstMbQCkJ1EGl/KoW/IleHSLJ+2gJBQFhdCN+JZuN3sl9il/dkP6/nrhhCfnOv+eBavfZxMH8ICD/9NUI3zwzqTnK/e3UfVYaQHysUQxI8VMkcR+/v3nszEaOWlTScpotmO60xzEbzMTF2agaR2+idpaKLQ8RVShDaIQgIqwThjq7drjrZyLd3soddzYIrbGW1aql2+cZrFUDI7yxUeOL+xNNrq9fFu59QGcxNd+JMrz3ghWh0XS88GacT6oVvawChF4XvKUWwfP7jKvvug6sDYv2MF0elW5C+yoeuR9dFs52JMBRcEzPkJvvKlZ82tlMdRH6Kw5lnDS59pTlbIp2i+MwKQUBYJQhblEj1m8Ku0HH27+dnKd5qqAqEfkH/b3cjB+7TH/2TT4C74W2ixcKTI5M1gNAvBc+rK1SKZ4xS5EY5GuKEt9g1ddFsZyIMhU2acr4+zk7NIPoHJCeu8vrtaCm5cpQpOYKAsEoQblMg9LeGzU7cSaYkEOqOEvUuSF/2KC+fa5DmuDel2MQ4Es0TXqwKhHyCiivMj750wvN6zNEVSjoIfahMXYyxMwmGEU+YGELNIEaOGiodiUBogyAgLB/CjngIgzNLHWGtm+ZofQII9fHBx9f9Vn/+QwpBjN3z37hzzZvi/p2chqRaNX3bDwfEMGDWSXRH70+VGwL6N2+5fMLzi8uReL/8Fx0MXYyzMwGGf+o84VQSO9VBdMkMLyz3tRU7/QGE5+3+NAwgrL12tOxqampq79S9V8F9R5fOzc0VV66pv/f049WV1ZVrwpxaelYsvhpK11VL54rFH5NPRnMXjXbWVOogptNrc4vr67cmnlT7j30Aws0V/lIvBAgBIQQIASEECCFACAFCQAgBQggQQoAQEEKAEAKEECAEhBAghAAhBAgBIQQIIUAIAUJACAFCCBBCgBAQQoAQAoQQIASEECCEACEECAEhBAghQAgBQkAIAUIIEEKAEBBCgBAChBAgBIQQIIQAIQQI/1567w3o/17vg4PNVCOGAIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCIIgCNpy+h/bS3kA1ZxOpgAAAABJRU5ErkJggg==)

<div style="margin: 15px 0; line-height: 1.5;">

Here's the assembly again for reference:

</div>

<div>

``` prettyprint
TEXT ·add(SB),NOSPLIT,$0
    MOVQ x+0(FP), BX
    MOVQ y+8(FP), BP
    ADDQ BP, BX
    MOVQ BX, ret+16(FP)
    RET
```

</div>

<div style="margin: 15px 0; line-height: 1.5;">

The assembly version of the
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">add</span>
function is loading the variable
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">x</span>
at memory address
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">+0(FP)</span>
into the register
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BX</span>.
It then loads
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">y</span>
at memory address
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">+8(FP)</span>
into the register
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BP</span>,
adds
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BP</span>
to
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BX</span>
storing the result in
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BX</span>,
and finally copies
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">BX</span>
to the memory address
<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">+16(FP)</span>
and returns from the function. The calling function, which put all the
arguments onto the stack, will read the return value from where we left
it.

</div>

<div style="color:#888888;font-size:24px;margin-top:40px;">

Optimizing a Function with Assembly

</div>

<div style="margin: 15px 0; line-height: 1.5;">

We don't really need to write assembly to add two numbers together, so
what could we actually use it for?

</div>

<div style="margin: 15px 0; line-height: 1.5;">

Let's say you have a bunch of vectors, and you want to multiply them by
a transformation matrix. Maybe the vectors are points and [you want to
translate them through 3D
space](http://blog.wolfire.com/2010/07/Linear-algebra-for-game-developers-part-3).
We'll use 4D vectors with a 4x4 transformation matrix with the vectors
in a densely packed array. This is better than say, an array of objects
with a vector property on each, because doing a linear scan through
densely packed memory is more cache friendly.

</div>

<div>

[download](zip/asm-process-vectors.zip)

</div>

<div>

``` prettyprint
type V4 [4]float32
type M4 [16]float32

func M4MultiplyV4(m M4, v V4) V4 {
    return V4{
        v[0]*m[0] + v[1]*m[4] + v[2]*m[8] + v[3]*m[12],
        v[0]*m[1] + v[1]*m[5] + v[2]*m[9] + v[3]*m[13],
        v[0]*m[2] + v[1]*m[6] + v[2]*m[10] + v[3]*m[14],
        v[0]*m[3] + v[1]*m[7] + v[2]*m[11] + v[3]*m[15],
    }
}

func multiply(data []V4, m M4) {
    for i, v := range data {
        data[i] = M4MultiplyV4(m, v)
    }
}
```

</div>

<div style="background-color:#DBDBDB;border-radius:0 0 6px 6px;margin:6px 0;padding:10px;position:relative;top:-10px;">

<div style="font-family:Monaco, monospace;font-size:16px;">

go get goroutines.com/asm-process-vectors

</div>

</div>

<div style="margin: 15px 0; line-height: 1.5;">

This takes 140ms for 128MB of data on a 2012 Macbook Pro using Go 1.5.3.
What's the fastest this implementation could be? A memory copy seems
like a good benchmark, it's [about 14ms](zip/asm-vectors-copy.zip).

</div>

<div style="margin: 15px 0; line-height: 1.5;">

Here's a version of the function written in assembly using SIMD
instructions to perform the multiplications, which lets us multiply 4
float32s in parallel:

</div>

<div>

[download](zip/asm-process-vectors-asm-simd.zip)

</div>

<div>

``` prettyprint
// func multiply(data []V4, m M4)
//
// memory layout of the stack relative to FP
//  +0 data slice ptr
//  +8 data slice len
// +16 data slice cap
// +24 m[0]  | m[1]
// +32 m[2]  | m[3]
// +40 m[4]  | m[5]
// +48 m[6]  | m[7]
// +56 m[8]  | m[9]
// +64 m[10] | m[11]
// +72 m[12] | m[13]
// +80 m[14] | m[15]

TEXT ·multiply(SB),NOSPLIT,$0
  // data ptr
  MOVQ data+0(FP), CX
  // data len
  MOVQ data+8(FP), SI
  // index into data
  MOVQ $0, AX
  // return early if zero length
  CMPQ AX, SI
  JE END
  // load the matrix into 128-bit wide xmm registers
  // load [m[0], m[1], m[2], m[3]] into xmm0
  MOVUPS m+24(FP), X0
  // load [m[4], m[5], m[6], m[7]] into xmm1
  MOVUPS m+40(FP), X1
  // load [m[8], m[9], m[10], m[11]] into xmm2
  MOVUPS m+56(FP), X2
  // load [m[12], m[13], m[14], m[15]] into xmm3
  MOVUPS m+72(FP), X3
LOOP:
  // load each component of the vector into xmm registers
  // load data[i][0] (x) into xmm4
  MOVSS    0(CX), X4
  // load data[i][1] (y) into xmm5
  MOVSS    4(CX), X5
  // load data[i][2] (z) into xmm6
  MOVSS    8(CX), X6
  // load data[i][3] (w) into xmm7
  MOVSS    12(CX), X7
  // copy each component of the matrix across each register
  // [0, 0, 0, x] => [x, x, x, x]
  SHUFPS $0, X4, X4
  // [0, 0, 0, y] => [y, y, y, y]
  SHUFPS $0, X5, X5
  // [0, 0, 0, z] => [z, z, z, z]
  SHUFPS $0, X6, X6
  // [0, 0, 0, w] => [w, w, w, w]
  SHUFPS $0, X7, X7
  // xmm4 = [m[0], m[1], m[2], m[3]] .* [x, x, x, x]
  MULPS X0, X4
  // xmm5 = [m[4], m[5], m[6], m[7]] .* [y, y, y, y]
  MULPS X1, X5
  // xmm6 = [m[8], m[9], m[10], m[11]] .* [z, z, z, z]
  MULPS X2, X6
  // xmm7 = [m[12], m[13], m[14], m[15]] .* [w, w, w, w]
  MULPS X3, X7
  // xmm4 = xmm4 + xmm5
  ADDPS X5, X4
  // xmm4 = xmm4 + xmm6
  ADDPS X6, X4
  // xmm4 = xmm4 + xmm7
  ADDPS X7, X4
  // data[i] = xmm4
  MOVNTPS X4, 0(CX)
  // data++
  ADDQ $16, CX
  // i++
  INCQ AX
  // if i >= len(data) break
  CMPQ AX, SI
  JLT LOOP
END:
  // since we use a non-temporal write (MOVNTPS)
  // make sure all writes are visible before we leave the function
  SFENCE
  RET
```

</div>

<div style="background-color:#DBDBDB;border-radius:0 0 6px 6px;margin:6px 0;padding:10px;position:relative;top:-10px;">

<div style="font-family:Monaco, monospace;font-size:16px;">

go get goroutines.com/asm-process-vectors-asm-simd

</div>

</div>

<div style="margin: 15px 0; line-height: 1.5;">

This one takes 18ms, so it's pretty close to the speed of the memory
copy. A better optimization might be to run this sort of thing on the
GPU instead of on the CPU because the GPU is really good at it. Here are
the running times for the different programs, including a manually
inlined Go version and a non-SIMD assembly implementation:

</div>

| program                                           | time  | speedup |
| ------------------------------------------------- | ----- | ------- |
| [original](zip/asm-process-vectors.zip)           | 140ms | 1x      |
| [inline](zip/asm-process-vectors-inline.zip)      | 69ms  | 2x      |
| [assembly](zip/asm-process-vectors-asm-plain.zip) | 43ms  | 3x      |
| [simd](zip/asm-process-vectors-asm-simd.zip)      | 17ms  | 8x      |
| [copy](zip/asm-vectors-copy.zip)                  | 15ms  | 9x      |

<div style="margin: 15px 0; line-height: 1.5;">

When optimizing, we can pay the cost of some code complexity to make
things easier for the machine. Assembly is a complicated way to do that,
but sometimes it's the best method available.

</div>

<div style="color:#888888;font-size:24px;margin-top:40px;">

Implementation Notes

</div>

<div style="margin: 15px 0; line-height: 1.5;">

I developed the assembly parts mostly in C and x64 assembly using XCode
and then ported the assembly to the Go format. XCode has a nice debugger
that lets you inspect CPU registers while the program is running. If you
include an assembly .s file in an XCode project, it will build it and
link it to your executable.

</div>

<div style="margin: 15px 0; line-height: 1.5;">

I used the [Intel x64 Instruction Set
Reference](http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf)
and [Intel Intrinsics
Guide](https://software.intel.com/sites/landingpage/IntrinsicsGuide/) to
figure out which instructions to use. Converting to Go assembly is not
necessarily straightforward, but many x64 assembly instructions are
included in
[<span style="background-color:rgba(0, 0, 0, 0.1);border-radius:5px;font-family:Monaco, monospace;font-size:14px;padding:3px 6px;white-space:nowrap;">x86/anames.go</span>](https://github.com/golang/go/blob/release-branch.go1.5/src/cmd/internal/obj/x86/anames.go)
and if they are not, they can be [encoded
directly](https://golang.org/doc/asm#unsupported_opcodes) with the
binary
representation.

-------------------------------------------------

**goroutines** is a series of articles related somehow to the Go programming language
