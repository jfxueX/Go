[Modifying a Go slice in-place during iteration][1]
===================================================

July 25, 2016

by Paul Smith

[1]: https://pauladamsmith.com/blog/2016/07/go-modify-slice-iteration.html

![][./images/go-slice-mod.png]

**Update:** See a better way of doing this below.

------------------------------------------------------------------------

I'll often have a slice that I want to filter down on, removing elements based 
on some test, and I would prefer to modify the slice in-place for whatever 
reason, either because I want to retain the reference to the original slice or I 
don't want to allocate a new slice as destination for the desired values.

You might think that modifying a slice in-place during iteration should not be 
done, because while you can modify *elements* of the slice during iteration if 
they are pointers or if you index into the slice, changing the *slice itself* by 
removing elements during iteration would be dangerous.

Here's a straightforward way to accomplish it. The idea is that, when you 
encounter an element you want to remove from the slice, take the beginning 
portion of the slice that has values that have passed the test up to that point, 
and remaining portion of the slice, i.e., after that element to the end, and 
copy them *over* the original slice. Then, assign a slice expression up to the 
number of values that passed the test to the original variable.

Here's an example. Let's say I have a slice of integers, and I only want to 
retain the even ones.


```go
var x = []int{90, 15, 81, 87, 47, 59, 81, 18, 25, 40, 56, 8}

i := 0
l := len(x)
for i < l {
    if x[i] % 2 != 0 {
        x = append(x[:i], x[i+1:]...)
        l--
    } else {
        i++
    }
}
x = x[:i]

fmt.Println(x)
// [90 18 40 56 8]
```

The `i` variable is used to keep track of the number of even values found in the 
slice. When an element is odd, we create a temporary slice using `append` and 
two slice expressions on the original slice, skipping over the current element. 
The temporary smaller slice is copied over the existing, shifting down the 
remaining values. The `l` variable makes sure we make the right number of 
comparisons despite moving things around. It's important to note the memory 
location of the original slice is unchanged with the copy. No new heap 
allocations are performed, even with the temporary slice.

------------------------------------------------------------------------

**Update:** A number of people, including here in comments and on [the golang 
reddit][2], have pointed out that the method I outline here is pretty 
inefficient; it's doing a lot of extra work, due to the way I'm using `append`. 
A *much* better way to go about it is the following, which also happens to have 
already been pointed out in the [official Go wiki][3]:

[2]: https://www.reddit.com/r/golang/comments/4uoqr5/modifying_a_go_slice_inplace_while_iterating_over/
[3]: https://github.com/golang/go/wiki/SliceTricks#filtering-without-allocating

```go
y := x[:0]
for _, n := range x {
    if n % 2 != 0 {
        y = append(y, n)
    }
}
```

This also has the benefit of being simpler and shorter. Use it instead!


#### About

Paul Smith is a software engineer. [More …](https://pauladamsmith.com/about.html)
