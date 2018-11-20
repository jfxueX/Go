
[![Learning the Go Programming
Language](https://cdn-images-1.medium.com/letterbox/301/45/50/50/1*LpnzYkg7-Mp_j8f0xHj3Og.png?source=logoAvatar-lo_bYEJnDVSsXzK---49b3a4577f18)](https://medium.com/learning-the-go-programming-language?source=logo-lo_bYEJnDVSsXzK---49b3a4577f18)

</div>

<div class="u-flexCenter u-height65 u-xs-height56 u-xs-hide">

<div class="buttonSet">

<span class="button-label js-buttonLabel">Follow</span>

</div>

</div>

</div>

<div class="metabar-block u-flex0 u-flexCenter">

<div class="u-flexCenter u-height65 u-xs-height56">

<div class="buttonSet buttonSet--wide u-lineHeightInherit">

[Sign
in](https://medium.com/m/signin?redirect=https%3A%2F%2Fmedium.com%2Flearning-the-go-programming-language%2Fpointing-to-go-the-go-pointer-type-a3c3f587592f&source=--------------------------nav_reg&operation=login)[Get
started](https://medium.com/m/signin?redirect=https%3A%2F%2Fmedium.com%2Flearning-the-go-programming-language%2Fpointing-to-go-the-go-pointer-type-a3c3f587592f&source=--------------------------nav_reg&operation=register)

</div>

</div>

</div>

</div>

<div class="metabar-inner u-marginAuto u-maxWidth1032 js-metabarBottom">

  - [Home](https://medium.com/learning-the-go-programming-language)

  - [Go
    Fundamentals](https://medium.com/learning-the-go-programming-language/fundamentals/home)

  - [IO and
    Networking](https://medium.com/learning-the-go-programming-language/io-networking/home)

  - 

</div>

</div>

<div class="metabar metabar--spacer js-metabarSpacer u-height105 u-xs-height95">

</div>

<div data-role="main">

<div class="uiScale uiScale-ui--regular uiScale-caption--regular u-paddingBottom10 row postMetaHeader">

<div class="col u-size12of12 js-postMetaLockup">

<div class="uiScale uiScale-ui--regular uiScale-caption--regular postMetaLockup postMetaLockup--authorLockupForPost u-flexCenter js-postMetaLockup">

<div class="u-flex0">

[](https://medium.com/@vladimirvivien?source=post_header_lockup)

<div class="u-relative u-inlineBlock u-flex0">

![Go to the profile of Vladimir
Vivien](https://cdn-images-1.medium.com/fit/c/75/75/1*FhusGAyQctbtxjUhNVMwqA.jpeg)

<div class="avatar-halo u-absolute u-textColorGreenNormal svgIcon" style="width: calc(100% + 12px); height: calc(100% + 12px); top:-6px; left:-6px">

</div>

</div>

</div>

<div class="u-flex1 u-paddingLeft15 u-overflowHidden">

<div class="u-lineHeightTightest">

[Vladimir
Vivien](https://medium.com/@vladimirvivien?source=post_header_lockup)

<span class="button-label button-defaultState">Blocked</span><span class="button-label button-hoverState">Unblock</span>

<span class="button-label button-defaultState js-buttonLabel">Follow</span><span class="button-label button-activeState">Following</span>

</div>

<div class="ui-caption ui-xs-clamp2 postMetaInline">

Software Eng • Go Programming • Kubernetes • Author
http://golang.fyi

</div>

<div class="ui-caption postMetaInline js-testPostMetaInlineSupplemental">

Apr 25,
2016<span class="middotDivider u-fontSize12"></span><span class="readingTime" title="4 min read"></span>

</div>

</div>

</div>

</div>

</div>

<div class="postArticle-content js-postField js-notesSource js-trackedPost" data-post-id="a3c3f587592f" data-source="post_page" data-collection-id="49b3a4577f18" data-tracking-context="postPage">

<div class="section section section--body section--first section--last" name="592f">

<div class="section-divider">

-----

</div>

<div class="section-content">

<div class="section-inner sectionLayout--insetColumn">

# Pointing to Go — The Go Pointer Type

![](https://cdn-images-1.medium.com/freeze/max/38/0*XNySepcb8F70SftL.?q=20)

In my introductory write up on Go types, I did a (rather length)
summarized walk-through of the basic and composite types in Go (read it
[**here**](https://medium.com/learning-the-go-porgramming-language/types-in-the-go-programming-language-65e945d0a692)
if you need a refresher). Continuing with this theme, this write up
discusses the pointer type: how to create and initialize pointer types.

Though these are easy concepts in Go, nevertheless, a well-grounded
understanding of these topics is certain to lessen frustrating moments
if you are a newcomer to Go\!

### The Pointer Type

The pointer type in Go is used to point to a memory address where data
is stored. Similar to C/C++, Go uses the **\*** operator to designate a
type as a pointer. The following snippet shows several pointers with
different underlying types.

``` graf graf--pre graf-after--p
var valPtr *float32
var countPtr *int
```

``` graf graf--pre graf-after--pre
type person struct{name string, age int}
var prsn *person
var matrix *[1024]int
var row []*int64
```

The code snippet above declares each variable as a pointer to its
associated type:

  - <span id="1020">variables *valPtr* and *countPtr* are pointers to
    their respective primitive types *float32* and *int*</span>
  - <span id="133a">variables *prsn* and *matrix* are declared as
    pointers to their respective composite types *person* and
    *\[1024\]int*.</span>
  - <span id="6a91">Lastly, variable *row* is a slice of pointers to
    elements to type *int64*.</span>

Each pointer stores an address which points to a memory location where a
value, of the indicated underlying type, is stored. Go does not allow
pointer arithmetic and, as you have come to expect with Go’s strict type
system, each pointer type is unique. Meaning, the following will not
compile.

``` graf graf--pre graf-after--p
var intPtr *int
var int32Ptr *int32
```

``` graf graf--pre graf-after--pre
intPtr = int32Ptr
```

``` graf graf--pre graf-after--pre
$> cannot use int32Ptr (type *int32) as type *int in assignment
```

This is because a pointer to an *int* is not compatible with pointer of
type *int32*, even when both point to types with similar memory layout.

#### The Address Operator

Go uses the ***&*** (ampersand) operator to return the address of a
variable. For instance, the following uses the address operator in
expressions that return memory locations for associated values.

``` graf graf--pre graf-after--p
val := float32(5.5)
var valPtr *float32 = &val
```

``` graf graf--pre graf-after--pre
score := 79
scorePtr := &score
```

``` graf graf--pre graf-after--pre
func printId(id *string) { 
    ... 
}
uid := "abcd-eff-33cc-5534"
printId(&uid)
```

The two variables *valPtr* and *scorePtr*, in the pervious code snippet,
are assigned memory addresses with expressions *\&val* and *\&score*
respectively. The second portion of the code snippet shows function
*printId(id \*string)* that takes a pointer as its sole parameter. It is
invoked with the address value *\&uid* as its argument.

It is important to note that in Go, you cannot use ampersand operator
directly on literal constants for numeric, string, boolean types. For
instance, the following will not compile.

``` graf graf--pre graf-after--p
ptr := &5
```

``` graf graf--pre graf-after--pre
$> cannot take the address of 5
```

You can, however, apply the ampersand operator to composite type literal
expressions. For instance, the following shows the literal expressions
for initializing a struct and an array values that use the ampersand
operators to return their addresses.

``` graf graf--pre graf-after--p
type person struct{name string, age int}
prsn := &person{"Prince", 57}
```

``` graf graf--pre graf-after--pre
pair := &[2]string{"left-sock", "right-sock"}
```

In the previous snippet, variable *prsn* stores the address of a value
of *\&person{“Prince”, 57}*. Variable *pair* is declared and initialized
with the address of value *&\[2\]string{“left-sock”, “right-sock”}*.

#### Pointer Indirection — Dereference Pointer Values

As established, a pointer is simply an address of an actual value in
memory. To access that value at the end of the pointer, simply apply the
\* operator to an address as shown in the following.

``` graf graf--pre graf-after--p
func main() {
    a := 71
    print(&a)
}
func print(val *int) {
    fmt.Println(val)
    fmt.Println(*val * 2)
}
```

In the previous example, *function print(val \*int)* accepts a pointer
as its parameter. It first prints the address with the first
*fmt.Println(val)* . In the second *fmt.Println(\*val \* 2)* statement,
the code uses pointer indirection *\*val* to access the actual value
pointed to, which is then multiplied by 2. When executed, the output
will look something like the following.

``` graf graf--pre graf-after--p
0x10434114
142
```

#### The new() Function

When the built-in function new(\<type\>) is used to initialize a value,
it allocates the appropriate memory for a zero-value of the specified
type. The function then returns the address for the newly created
zero-value.

``` graf graf--pre graf-after--p
intPtr := new(int)
*intPtr = 77
```

``` graf graf--pre graf-after--pre
type person struct{name string, age int}
prsn := new(person)
prsn.first = "Prince"
prsn.age = 57
```

The previous snippet uses the *new()* function to initialize a
zero-value int and assign its address to variable *intPtr*. Then the
code uses pointer indirection to assign it a value with *\*intPtr = 77*.

Similarly, the code initializes variable *prsn* with the address of
zero-value for composite type *person*. To access the value (not the
address) of the pointed composite, the idiom is more forgiving. It is
not necessary to write *\*prsn.first = “Prince”*. We can drop the
indirection and just use *prsn.first = “Prince”*.

#### Conclusion

In this short(er) write up, I explored the Go pointer type. You saw how
to create and initialize pointers. We also explored pointer indirection
to access and update pointed values. Future write ups will continue to
explore the Go type system covering Functions, Methods, and Interfaces,
etc. (Making the write up shorter will hopefully let me get to them
sooner\!)

Happy go(ding)\!

</div>

</div>

</div>

</div>

<div class="container u-maxWidth740">

<div class="row">

<div class="col u-size12of12">

</div>

</div>

<div class="row">

<div class="col u-size12of12 js-postTags">

<div class="u-paddingBottom10">

  - [Programming](https://medium.com/tag/programming?source=post)
  - [Golang](https://medium.com/tag/golang?source=post)
  - [Programming
    Languages](https://medium.com/tag/programming-languages?source=post)

</div>

</div>

</div>

<div class="postActions js-postActionsFooter">

<div class="u-flexCenter">

<div class="u-flex1">

<div class="multirecommend js-actionMultirecommend u-flexCenter" data-post-id="a3c3f587592f" data-is-icon-29px="true" data-is-circle="true" data-has-recommend-list="true" data-source="post_actions_footer-----a3c3f587592f---------------------clap_footer">

<div class="u-relative u-foreground">

<div class="clapUndo u-width60 u-round u-height32 u-absolute u-borderBox u-paddingRight5 u-transition--transform200Spring u-background--brandSageLighter js-clapUndo" style="top: 14px; padding: 2px;">

</div>

</div>

140

</div>

</div>

<div class="buttonSet u-flex0">

2

</div>

</div>

</div>

</div>

<div class="u-maxWidth740 u-paddingTop20 u-marginTop20 u-borderTopLightest container u-paddingBottom20 u-xs-paddingBottom10 js-postAttributionFooterContainer">

<div class="row js-postFooterInfo">

<div class="col u-size6of12 u-xs-size12of12">

<div class="u-marginLeft20 u-floatRight">

<span class="button-label button-defaultState">Blocked</span><span class="button-label button-hoverState">Unblock</span>

<span class="button-label button-defaultState js-buttonLabel">Follow</span><span class="button-label button-activeState">Following</span>

</div>

<div class="u-tableCell">

[](https://medium.com/@vladimirvivien?source=footer_card "Go to the profile of Vladimir Vivien")

<div class="u-relative u-inlineBlock u-flex0">

![Go to the profile of Vladimir
Vivien](https://cdn-images-1.medium.com/fit/c/75/75/1*FhusGAyQctbtxjUhNVMwqA.jpeg)

<div class="avatar-halo u-absolute u-textColorGreenNormal svgIcon" style="width: calc(100% + 12px); height: calc(100% + 12px); top:-6px; left:-6px">

</div>

</div>

</div>

<div class="u-tableCell u-verticalAlignMiddle u-breakWord u-paddingLeft15">

### [Vladimir Vivien](https://medium.com/@vladimirvivien "Go to the profile of Vladimir Vivien")

<div class="ui-caption u-textColorGreenNormal u-fontSize13 u-tintSpectrum u-accentColor--textNormal u-marginBottom7">

Medium member since Jul 2017

</div>

Software Eng • Go Programming • Kubernetes • Author <http://golang.fyi>

</div>

</div>

<div class="col u-size6of12 u-xs-size12of12 u-xs-marginTop30">

<div class="u-marginLeft20 u-floatRight">

<span class="button-label js-buttonLabel">Follow</span>

</div>

<div class="u-tableCell">

[![Learning the Go Programming
Language](https://cdn-images-1.medium.com/fit/c/75/75/1*dYBDKrYcd4IPQDn7-5RNSQ.png)](https://medium.com/learning-the-go-programming-language?source=footer_card "Go to Learning the Go Programming Language")

</div>

<div class="u-tableCell u-verticalAlignMiddle u-breakWord u-paddingLeft15">

### [Learning the Go Programming Language](https://medium.com/learning-the-go-programming-language?source=footer_card)

Short and insightful posts for newcomers learning the Go programming
language

<div class="buttonSet">

</div>

</div>

</div>

</div>

</div>

<div class="js-postFooterPlacements">

</div>

<div class="u-padding0 u-clearfix u-backgroundGrayLightest u-print-hide supplementalPostContent js-responsesWrapper">

</div>

<div class="supplementalPostContent js-heroPromo">

</div>

</div>

<div class="u-foreground u-top0 u-transition--fadeOut300 u-fixed u-sm-hide u-marginLeftNegative12 js-postShareWidget">

  - 
    
    <div class="multirecommend js-actionMultirecommend u-flexColumn u-marginBottom10" data-post-id="a3c3f587592f" data-is-icon-29px="true" data-is-vertical="true" data-is-circle="true" data-has-recommend-list="true" data-source="post_share_widget-----a3c3f587592f---------------------clap_sidebar">
    
    <div class="u-relative u-foreground">
    
    <div class="clapUndo u-width60 u-round u-height32 u-absolute u-borderBox u-paddingRight5 u-transition--transform200Spring u-background--brandSageLighter js-clapUndo" style="top: 14px; padding: 2px;">
    
    </div>
    
    </div>
    
    140
    
    </div>

  - 
  - 
  - 

</div>

<div class="u-fixed u-bottom0 u-width100pct u-backgroundWhite u-boxShadowTop u-borderBox u-paddingTop10 u-paddingBottom10 u-zIndexMetabar u-xs-paddingLeft10 u-xs-paddingRight10 js-stickyFooter">

<div class="u-maxWidth700 u-marginAuto u-flexCenter">

<div class="u-fontSize16 u-flex1 u-flexCenter">

<div class="u-flex0 u-inlineBlock u-paddingRight20 u-xs-paddingRight10">

[![Learning the Go Programming
Language](https://cdn-images-1.medium.com/fit/c/50/50/1*dYBDKrYcd4IPQDn7-5RNSQ.png)](https://medium.com/learning-the-go-programming-language "Go to Learning the Go Programming Language")

</div>

<div class="u-flex1 u-inlineBlock">

<div class="u-xs-hide">

Never miss a story from **Learning the Go Programming Language**, when
you sign up for Medium. [Learn
more](https://medium.com/@Medium/personalize-your-medium-experience-with-users-publications-tags-26a41ab1ee0c#.hx4zuv3mg)

</div>

<div class="u-xs-show">

Never miss a story from **Learning the Go Programming Language**

</div>

</div>

</div>

<div class="u-marginLeft50 u-xs-marginAuto">

<span class="button-label button-defaultState js-buttonLabel">Get
updates</span><span class="button-label button-activeState">Get
updates</span>

</div>

</div>

</div>

</div>

</div>

</div>

<div class="loadingBar">

</div>
