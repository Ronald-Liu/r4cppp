# 引用计数和原始指针

目前我们已经介绍了独占指针和借出指针。独占指针与C++中的`std::unique_ptr`很相似，而当你在C++中用了指针或引用的地方，在Rust中你可以默认考虑借出指针。Rust还有少数几个更罕见的内建和库中实现的指针类型。它们很像你在C++中用到的各种智能指针。

我花了不少时间来写这一章，然而仍然不太满意。不论在我的文章中还是在Rust的实现上，这部分都有很多虎头蛇尾，我希望这在之后的写作中和Rust语言开发的过程中能有所改善。你可以首先跳过这一部分，因为你大概也不怎么会用到它们。

看起来Rust中有很多指针类型，其实这就像C++在标准库中有很多智能指针一样。只不过在Rust中，你刚开始就会遇到他们。Rust中的指针是编译器支持的，因此相比C++，使用它们更不容易出错。

I'm not going to cover these in as much detail as unique and borrowed references
because, frankly, they are not as important. I might come back to them in more
detail later on.
我不想像介绍独占指针那样介绍它们，因为说实话它们真的不那么重要。也许之后我会回头来完善他们。

## `Rc<T>`

引用计数指针是Rust标准库德一部分，它位于`std::rc`模块中（我们将会在不远的将来说到模块，本章的`use`指令就是为了引入模块的）。一个指向`T`类型的引用计数指针的类型为`Rc<T>`。你可以使用静态方法（可以暂时理解为C++ 中的静态函数，不过之后会讲它们之间的差异）——`Rc::new(...)`来创建一个引用计数指针，它可以用一个值来初始化这个指针所指向的位置。这一构造函数遵循Rust的移动/拷贝语义（如我们之前讨论独占指针所说）——不论何时，在调用Rc::new之后，你将只能通过这个指针来访问那个值。

像其他指针类型一样，`.`操作符可以自动地进行解引用，也可以用`*`来手动解引用。


如果需要传递引用计数指针，你需要使用`clone`函数。这有点难看，但愿之后可以修复它，但是也一定能做到。因为可以获得对于它的借出指针，所以通常你并不需要经常使用`clone`。Rust的类型系统保证引用计数所指向的变量在所有引用退出作用域之前不会被删除。取得一个引用计数指针的借出引用不需要增加或减少引用计数，从而得到更好的性能（不过性能提高很有限，因为引用计数对象被限制在一个单独的线程内，因此引用计数操作不需要是原子操作）。

一个Rc的例子
```rust
use std::rc::Rc;

fn bar(x: Rc<i32>) { }
fn baz(x: &i32) { }

fn foo() {
    let x = Rc::new(45);
    bar(x.clone());   // Increments the ref-count
    baz(&*x);         // Does not increment
    println!("{}", 100 - *x);
}  // Once this scope closes, all Rc pointers are gone, so ref-count == 0
   // and the memory will be deleted.
```

引用计数指针总是不可变的。如果你需要一个可变的引用计数对象，你必须用`RefCell`或`Cell`来包裹`Rc`内的类型

## Cell and RefCell

Cell and RefCell are structs which allow you to 'cheat' the mutability rules.
This is kind of hard to explain without first covering Rust data structures and
how they work with mutability, so I'm going to come back to these slightly
tricky objects later. For now, you should know that if you want a mutable, ref
counted object you need a Cell or RefCell wrapped in an Rc. As a first
approximation, you probably want Cell for primitive data and RefCell for objects
with move semantics. So, for a mutable, ref-counted int you would use
`Rc<Cell<int>>`.
Cell和RefCell是一个可以帮你绕过可变性规则的结构。在介绍Rust数据结构之前可能有点难以理解，所以我会之后再介绍这个比较tricky的对象。现在你只需要知道如果你需要一个可变的引用计数对象，你需要用`Cell`或`RefCell`来包裹Rc中的类型，`Cell`一般用于基本类型数据，而`RefCell`用于有引用语义的对象。所以，要得到一个可变的有引用计数的`int`你可以写成`Rc<Cell<int>>`。

## `*T` - 原始指针

Rust有两类原始指针（也叫非安全指针）：`*const T`用于不可变原始指针；`*mut T`用于可变的原始指针。可以通过`&`和`&mut`来分别创建它们，由于`&`操作符默认用来创建借出指针，因此你需要显式指定类型来得到一个`*T`而不是`&T`。原始指针就像C中的指针，它只是一个指向内存区域的值，并且对于它的使用没有任何限制（你不能在不做转换的情况下进行指针算数，不过如果你非要这么做的话也是可以做到的）。原始指针是Rust中唯一可以是null的指针类型。原始指针没有自动地解引用，因此你需要用`(*x).foo()`来访问具体的数据。最重要的一个限制是，在普通的Rust代码中原始指针只能被传递，它们不能在`unsafe`块之外解引用，因此也就不能在那之外被使用。

那么究竟什么是`unsafe`块呢？Rust有很强的安全性承诺，而且它在很罕见的情况下会禁止你作一些你必须做的事。由于Rust致力于成为一门系统编程语言，因此即使编译器无法验证代码的安全性，也必须允许做任何可能做的事。为了做到这一点，Rust有一个`unsafe`块的概念。在`unsafe`代码中，你可以做一些不安全的事——对原始指针作解引用，对数组作没有边界检查的访问，通过FFI调用其它语言的代码，或者做变量类型转换。很显然，在编写非安全代码时你需要加倍小心。事实上，你应该在非常罕见的情况下才会用到`unsafe`代码，主要是用在库的非常小段的代码中，而不是客户端代码。在`unsafe`块中你必须做C++中你经常做的事来保证安全性。另外，你必须手动保证通常由编译器保证的不变性。`unsafe`块要求你手动的保证Rust的不变性，如果你没有做到，你将会在`unsafe`代码内外同时引入错误。

下面是一个使用原始指针的例子

```rust
fn foo() {
    let mut x = 5;
    let x_p: *mut i32 = &mut x;
    println!("x+5={}", add_5(x_p));
}

fn add_5(p: *mut i32) -> i32 {
    unsafe {
        if !p.is_null() { // 注意：原始指针没有自动解引用，所以is_null是一个*i32中的函数，而不是i32中的
            *p + 5
        } else {
            -1            // 这不是推荐的错误处理模式
        }
    }
}
```

以上介绍了所有的Rust指针。下一章将介绍Rust中的数据类型，之后再回到借出指针的问题。
