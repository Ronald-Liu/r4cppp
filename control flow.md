# 控制流

## `if`

在Rust中，`if`语句的使用与C++几乎完全一致，有一个小区别是，不能省略大括号，而在测试表达式外侧的括号则可以省略。另外一个区别是在Rust中，`if`是一个表达式，如之前说过，大括号中最后一个不以`;`结尾的语句就是`if`表达式的值，所以你可以像C++中的`?`一样使用它（Rust中没有`?`）。下面两个代码做的事情完全相同。

```rust
fn foo(x: i32) -> &'static str {
    let mut result: &'static str;
    if x < 10 {
        result = "less than 10";
    } else {
        result = "10 or more";
    }
    return result;
}

fn bar(x: i32) -> &'static str {
    if x < 10 {
        "less than 10"
    } else {
        "10 or more"
    }
}
```


第一种基本上是C++写法的翻译版，而第二种则更具Rust风格。

你也可以写`let x = if ...`一类的句子


## 循环

Rust跟C++一样有while循环

```rust
fn main() {
    let mut x = 10;
    while x > 0 {
        println!("Current value: {}", x);
        x -= 1;
    }
}
```

在Rust中没有`do..while`循环，不过我们可以使用`loop`语句来写一个死循环

```rust
fn main() {
    loop {
        println!("Just looping");   
    }
}
```

Rust has `break` and `continue` just like C++.
Rust跟C++一样有`break`和`continue`指令。


## For循环

Rust也有`for`循环，不过跟C++中的不太一样。假设现在有一个包含很多整数的向量，而你像把它们都打印出来。在讲vector和数组的那一章中会具体说怎么用迭代器和范型，这里我们只需要知道`Vec<T>`是一个包含`T`类型的序列，而`iter()`函数可以返回一个迭代器用来获取vector中的值。一个简单的`for`循环可以这样写

```rust
fn print_all(all: Vec<i32>) {
    for a in all.iter() {
        println!("{}", a);
    }
}
```

如果你希望用下标来遍历`all`（像在C++中遍历普通数组那样），你可以写成：

```rust
fn print_all(all: Vec<i32>) {
    for i in ..all.len() {
        println!("{}: {}", i, all.get(i));
    }
}
```

`len()`函数的意思你懂的。


## Switch/Match

Rust中有`match`语句，与C++中的`switch`语句相似，不过它更加强大。最简单的`match`用法看起来会很熟悉。

```rust
fn print_some(x: i32) {
    match x {
        0 => println!("x is zero"),
        1 => println!("x is one"),
        10 => println!("x is ten"),
        y => println!("x is something else {}", y),
    }
}
```

这里有一些语法上的区别：我们用`=>`来分割匹配模式与需要执行的表达式，每一个分支之间用`,`分隔。上面的例子中，`y`会匹配未与之前任何值匹配的值。另外有一些不那么明显的语义区别：`match`的分支必须是穷尽的，这意味着匹配模式必须覆盖所有可能的匹配值。可以尝试下如果从在上面的例子中移除`y => ..`分支会发生什么。我们也可以写成

```rust
fn print_some(x: i32) {
    match x {
        x => println!("x is something else {}", x)
    }
}
```

分支中的`x`引入了一个新变量，因此在分支中会隐蔽掉函数参数中的`x`，这很像在内部作用域中变量对外部同名变量的隐蔽。

如果你不希望命名一个变量，可以用`_`来占位。如果我们什么都不需要做，可以写一个空的分支。

```rust
fn print_some(x: i32) {
    match x {
        0 => println!("x is zero"),
        1 => println!("x is one"),
        10 => println!("x is ten"),
        _ => {}
    }
}
```

另外一个语义区别是`match`分支之间不会有C++中那种执行穿透现象。

在之后的章节中我们会发现`match`是非常强大的。目前只介绍几个简单的特性——`or`操作符和`if`从句，它们都可以顾名思义。

```rust
fn print_some_more(x: i32) {
    match x {
        0 | 1 | 10 => println!("x is one of zero, one, or ten"),
        y if y < 20 => println!("x is less than 20, but not zero, one, or ten"),
        y if y == 200 => println!("x is 200 (but this is not very stylish)"),
        _ => {}
    }
}
```

与`if`表达式相同，`match`语句也是一个表达式，所以我们可以将最后一个例子改写成

```rust
fn print_some_more(x: i32) {
    let msg = match x {
        0 | 1 | 10 => "one of zero, one, or ten",
        y if y < 20 => "less than 20, but not zero, one, or ten",
        y if y == 200 => "200 (but this is not very stylish)",
        _ => "something else"
    };

    println!("x is {}", msg);
}
```

注意由于前一句事实上是一个`let`语句，必须有`let msg = ...;`的形式，所以最后一个大括号后的分号必不可少。

Rust中的`match`语句避免了C++中`switch`语句的不少常见Bug：你绝不为因为忘记写`break`得到不想要的执行穿越；如果写出了一些对匹配值的模式，Rust编译器会自动帮你检查是否覆盖了所有可能的情况。


## 方法调用

最终简单介绍一下Rust中的成员方法。与C++中类似，他们总是使用`.`操作符来调用（没有`->`操作符，这会在其他章节中讲到）。刚才我们看到了`len`和`iter`两个例子。之后我们会更深入的讲到如何定义和调用他们，不过在学习之前，你可以认为你在Java和C++中学到的经验都是适用的。