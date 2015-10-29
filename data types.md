# 数据类型



下面我们要讨论Rust中的数据类型。它们大致对应于C++中的class, struct和enum类型。与C++（Java以及大多数面向对象的语言）相比，Rust的数据和行为分离的更为严格。在Rust中，行为可以在traits或impl中由函数定义，其中traits不能包含数据。他们的意义接近于Java中的interface。我将在后面的章节讨论impl和traits，本章则只讨论数据。

## 结构体


Rust中的结构体(struct)与C中的struct或C++中不带方法的struct相似，只需要列出域的名字。他的语法是：

```rust
struct S {
    field1: int,  
    field2: SomeOtherStruct  
}
```

这里我们定义了一个名叫S的struct，并定义了两个域。域之间用逗号分割。如果你喜欢，你也可以在最后一个域后面加逗号。

结构体定义一个类型。在这个例子中，我们使用S作为类型。并假设SomeOtherStruct是另一个类型并且以值的形式包含在struct中。所谓值的形式，意思是不使用指向其他结构的指针。

要访问结构体中的域，需要使用`.`运算符和域的名字。下面的例子说明了对struct的使用方法

```rust
fn foo(s1: S, s2: &S) {
    let f = s1.field1;
    if f == s2.field1 {
        println!("field1 matches!");
    }
}
```


这里 `s1`是一个按值传递的struct对象，而`s2`是一个引用传递的struct对象。在函数中，我们都使用`.`来访问域，而不需要使用`->`

结构体由结构自面量(struct literals)初始化。需要指定结构体的名字以及每个域的值。下面是一个结构体初始化的例子

```rust
fn foo(sos: SomeOtherStruct) {
    let x = S { field1: 45, field2: sos };  // initialise x with a struct literal
    println!("x.field1 = {}", x.field1);
}
```

由于结构体定义使用值语义，结构体不能递归定义，这意味着struct内域的类型不能包含自己。像`struct R { r: Option<R> }`这样的定义是非法的，并且会触发编译错误（后面将会详细说到Option）。如果你需要递归引用的结构体，你应该使用某种形式的指针，因为指针中允许存在环。下面的例子就是一个合法的自我引用结构体

```rust
struct R {
    r: Option<Box<R>>
}
```

如果上例中没有Option，我们就无法初始化这个结构体，Rust会因此报错。

没有域的结构体无论在定义还是字面量中都不使用大括号。大概是为了加速语法分析，在定义中还是需要有一个分号结尾。下例定义了一个空结构体

```rust
struct Empty;

fn foo() {
    let e = Empty;
}
```

## Tuple

元组(Tuple)是匿名、异构的数据序列。元组由括号中的类型名来定义。因为元组没有名字，他们是由它们的结构定义的。比如类型`(int,int)`是整数而`(i32, f32, S)`是一个三元组。元组值的初始化语法跟声明语法很像，只不过括号里原来放着类型的地方初始化时写的是具体的值。下面是一个例子

```rust
// foo takes a struct and returns a tuple
fn foo(x: SomeOtherStruct) -> (i32, f32, S) {
    (23, 45.82, S { field1: 54, field2: x })
}
```

元组可以用在let表达式之后用于解构。

```rust
fn bar(x: (int, int)) {
    let (a, b) = x;
    println!("x was ({}, {})", a, b);
}
```

我们讲在后面详细说解构的问题。

## 元组结构体

元组结构体是有名字的元组，或者说是没有域名字的结构体。
元组结构体使用`struct`关键字声明，接着一些逗号分割的类型并用括号包裹，最后跟着一个分号。
这样就声明了一个具有指定名字的类型。
其中的域只能像元组一样通过解构操作访问，而不能通过域的名字访问。
元组结构体不是很常用。下面是一个元组结构体的例子。

```rust
struct IntPoint (int, int);

fn foo(x: IntPoint) {
    let IntPoint(a, b) = x;  // Note that we need the name of the tuple
                             // struct to destructure.
    println!("x was ({}, {})", a, b);
}
```

## Enum

枚举(Enum)是类似于C++中enum和union的类型，其中可以放置多种类型的值。The simplest kind of enum is just like a C++ enum:
最简单的一种枚举类型就像C++中的enum：

```rust
enum E1 {
    Var1,
    Var2,
    Var3
}

fn foo() {
    let x: E1 = Var2;
    match x {
        Var2 => println!("var2"),
        _ => {}
    }
}
```

然而，Rust中的枚举比C++中的要强大的多：枚举的每个变体都可以包含数据。他们可以像元组一样通过类型列表来定义。在这种情况下enum更像是C++中的union。Rush的enum可使用带标记的union而不像C++中一样不带标记。着意味着你永远不会在运行时把一个变体当作另一个使用。举例来说：

```rust
enum Expr {
    Add(int, int),
    Or(bool, bool),
    Lit(int)
}

fn foo() {
    let x = Or(true, false);   // x has type Expr
}
```

在Rust中使用enum可以实现大多数简单的面向对象多态性。  

我们通常将match表达式于enum一起使用。之前说过match表达式与C++中的switch语句相似。之后我们将深入的讨论match表达式及其与数据解构结合的高级用法。下面是一个例子

```rust
fn bar(e: Expr) {
    match e {
        Add(x, y) => println!("An `Add` variant: {} + {}", x, y),
        Or(..) => println!("An `Or` variant"),
        _ => println!("Something else (in this case, a `Lit`)"),
    }
}
```
match的每一个分支匹配了`Expr`的一个变体，而且必须涵盖所有变体。最后一个`_`分支可以涵盖所有没有列出的变体（这个例子中只有`Lit`）。每个变体中的数据都被绑定在一个变量中，如在`Add`分支中我们绑定了`Add`的两个int到`x`和`y`中。如果我们不关心数据，可以像例子中的`Or`分支一样，使用`..`来匹配任意数据。

## Option

Rust中一个很常用的enum类型是`Option`。它有两个变体:`Some`和`None`。`None`不包含数据，而`Some`有一个类型为`T`的数据。`Option`是一个范型enum，我们将在后面详细说明，他的基本思想来源于C++。Option常常表示一个值可以存在也可以不存在。在C++中使用nullptr的地方（可能用于表示不可用、未定义、未初始化或者false），在Rust中都应使用Option。使用Option将会保证在使用值之前对它进行检查，从而避免出现对null指针解引用之类的操作，因而更加安全。同时它更具一般性，既可以与指针一起使用，也可以与值一起使用。下面是一个例子：

```rust
use std::rc::Rc;

struct Node {
    parent: Option<Rc<Node>>,
    value: int
}

fn is_root(node: Node) -> bool {
    match node.parent {
        Some(_) => false,
        None => true
    }
}
```

这里，`parent`域既可以使`None`也可以是包含一个`Rc<Node>`类型的`Some`。这个例子中通过使用`_`忽略了`Some`的载荷，因为我们并不需要它，在实际工作中有可能需要对其进行绑定。

There are also convenience methods on Option, so you could write the body of
`is_root` as `node.is_none()` or `!node.is_some()`.
Option包含了很多方法方便使用，因此你可以将刚才`is_root`函数的函数体写为`node.is_none()`或者`!node.is_some()`。

## 可变性的继承与Cell/RefCell

在Rust中，本地变量默认是不可变的，可以通过关键字`mut`标记为可变的。Rust中不能标记struct或enum中的域为可变，他们的可变性继承自他们所属的struct和enum。着意味着一个struct中的域是否可变，决定于这个struct本身是否可变。下面是一个例子

```rust
struct S1 {
    field1: int,
    field2: S2
}
struct S2 {
    field: int
}

fn main() {
    let s = S1 { field1: 45, field2: S2 { field: 23 } };
    // s 是深度不可变的，下面所有的变化都是不允许的
    // s.field1 = 46;
    // s.field2.field = 24;

    let mut s = S1 { field1: 45, field2: S2 { field: 23 } };
    // s是可变的，下面的修改是正确的
    s.field1 = 46;
    s.field2.field = 24;
}
```

可变性的继承止步于引用。这有点像C++中你可以通过一个const对象中的指针修改非const对象。如果希望一个引用域可变，
你需要在域类型前面加`&mut`：

```rust
struct S1 {
    f: int
}
struct S2<'a> {
    f: &'a mut S1   // mutable reference field
}
struct S3<'a> {
    f: &'a S1       // immutable reference field
}

fn main() {
    let mut s1 = S1{f:56};
    let s2 = S2 { f: &mut s1};
    s2.f.f = 45;   // legal even though s2 is immutable
    // s2.f = &mut s1; // illegal - s2 is not mutable
    let s1 = S1{f:56};
    let mut s3 = S3 { f: &s1};
    s3.f = &s1;     // legal - s3 is mutable
    // s3.f.f = 45; // illegal - s3.f is immutable
}
```

(在`S2`和`S3`中的`'a`参数是生命周期参数，我们将在后面详细讨论。)

有时，虽然一个对象逻辑上不可变，但其中的某些部分内部是可变的，如缓存或引用计数数据（即使是一个const对象的引用计数也需要在其析构时进行修改）。在C++中，可以使用`mutable`关键字来表明一个域即使在const对象中也可以被修改。在Rust中，我们有Cell和RefCell结构体。他们允许不可变对象的某一部分可变。虽然这很有用，但你应该注意当你在Rust中看到一个不可变的对象，它的某些部分可能是可变的。

RefCell和Cell允许你绕过Rust的可变性与别名限制。虽然编译器不能保证它们的静态不变形，但能保证Rust的不变量在运行时时不变的，因此使用它们时安全的。Cell和RefCell都是单线程对象。

Cell是用于拥有拷贝语义的类型（也就是原始类型）。Cell有`get`和`set`两个方法用于修改存储的值，同时还有`new`方法用来使用心得值初始化cell。Cell时非常简单的对象，因为在Rust中具有拷贝语义的类型不需要保存引用信息，而Cell又不能在线程间共享，所以使用它多半不会出错。

RefCell是用于拥有移动语义的类型（基本上包括了Rust中所有的非简单类型）的，最常见的就是struct对象。RefCell也使用`new`来创建并使用`set`函数修改。但是如果要获取RefCell中的值，你必须使用借出函数(`borrow`, `borrow_mut`, `try_borrow`, `try_borrow_mut`)来借出具体的值。这些函数的使用规则与静态借出相同——只能拥有一个可变借出，不能同时存在可变借出和不可变借出。不同的是，当出现违反上述规则的情况时，静态借出会导致编译错误，而RefCell的借出函数将返回运行时错误。`try_`系列函数会返回Option——当能正常借出时将返回`Some`，而不能正常借出时则返回`None`。如果值已经被借出，调用`set`方法也同样会失败。

下面是一个对RefCell的使用计数指针的代码（这是非常常见的使用场景）

```rust
use std::rc::Rc;
use std::cell::RefCell;

Struct S {
    field: int
}

fn foo(x: Rc<RefCell<S>>) {
    {
        let s = x.borrow();
        println!("the field, twice {} {}", s.f, x.borrow().field);
        // let s = x.borrow_mut(); // 错误 —— 前面已经有对X的借出
    }

    let s = x.borrow_mut(); // 正常 —— 前一个借出已经退出作用域
    s.f = 45;
    // println!("The field {}", x.borrow().field); // 错误 —— 不能同时存在可变和不可变借出
    println!("The field {}", s.f);
}
```

如果需要使用Cell或RefCell，你要尽量保证它们的起作用的范围尽量小，也就是说，尽量将它们应用在struct中的几个尽量少的域上，而不要应用在整个struct上。你可以把它们想象成一个单线程锁，更精细的控制锁会有更好的效果，因为它可以避免过多的锁冲突。