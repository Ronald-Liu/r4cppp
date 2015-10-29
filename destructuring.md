# 解构

我们在前一章当中重点关注了Rust中的数据结构，当有了一个数据结构，你可能会像将数据从里面取出来。对于struct而言，你可以像在C++中一样使用域名进行访问。对tuple,enum和元组结构体，则必须使用解构(Destructuring)来进行数据提取。在C++中不存在数据解构的问题，但是Python或者很多函数式语言都有解构的概念（在有的语言中它被称为元组拆包)。它的中心思想是，使用类似于对象创建的语句结构，将已有对象的数据填入到指定的本地变量中。换句话说，解构就是一种模式匹配和本地变量赋值的结合。解构已经成为了Rust最有力的语言特性。

解构通常使用`let`和`match`语句完成。当数据结构有多重变体时(如enum)，你需要使用`match`语句。`let`表达式将各个变量放入到当前作用域中，而`match`语句则会引入新的作用域。如下面的代码所示

```rust
fn foo(pair: (int, int)) {
    let (x, y) = pair;
    // 我们可以在foo中的任何地方使用x和y

    match pair {
        (x, y) => {
            // 只能在这个作用域内使用x和y
        }
    }
}
```

模式的语法（上例中`let`之后和`=>`之前的部分)在两个例子中是（几乎）完全一样的。你也可以在函数参数中使用相同的模式语法。

```rust
fn foo((x, y): (int, int)) {
}
```
(这种用法对struct和元组结构体更用，而如果参数是tuple就没那么有用了)

大多数初始化表达式都可以在解构模式中出现，而且它们可以任意的复杂。模式中也可以包含引用和原始类型的字面值。举个例子。

```rust
struct St {
    f1: int,
    f2: f32
}

enum En {
    Var1,
    Var2,
    Var3(int),
    Var4(int, St, int)
}

fn foo(x: &En) {
    match x {
        &Var1 => println!("first variant"),
        &Var3(5) => println!("third variant with number 5"),
        &Var3(x) => println!("third variant with number {} (not 5)", x),
        &Var4(3, St { f1: 3, f2: x }, 45) => {
            println!("destructuring an embedded struct, found {} in f2", x)
        }
        &Var4(_, x, _) => {
            println!("Some other Var4 with {} in f1 and {} in f2", x.f1, x.f2)
        }
        _ => println!("other (Var2)")
    }
}
```

需要注意的是我们需要在模式中使用`&`对引用进行解构。另外可以看到允许混合使用字面值(`5`, `3`, `St { ... }`)，下划线`_`，以及任意变量名（即使跟要匹配的值同名也可以）。

当有一个变量会出现而你又想忽略它时，可以使用`_`。比如在上面的例子中`&Var3(_)`表示匹配Var3变体，但是并不关心其中的具体值。在第一个`Var4`分支中我们将整个结构体都进行了绑定。当然也可以使用`..`来表示tuple和struct的所有域。因此当你想对enum的变体做某些事情而又不想管这个变体的具体结构时，可以这样写：

```rust
fn foo(x: En) {
    match x {
        Var1 => println!("first variant"),
        Var2 => println!("second variant"),
        Var3(..) => println!("third variant"),
        Var4(..) => println!("fourth variant")
    }
}
```

When destructuring structs, the fields don't need to be in order and you can use
`..` to elide the remaining fields. E.g.,
当解构struct时，可以不以声明时的顺序进行解构，同时也可以使用`..`忽略剩下的所有域。

```rust
struct Big {
    field1: int,
    field2: int,
    field3: int,
    field4: int,
    field5: int,	
    field6: int,
    field7: int,
    field8: int,
    field9: int,
}

fn foo(b: Big) {
    let Big { field6: x, field3: y, ..} = b;
    println!("pulled out {} and {}", x, y);
}
```

struct解构的一个缩写版本可以直接创建域名字对应的本地变量，比如在上面的例子中命名了两个新的本地变量`x`和`y`，你也可以这样写：

```rust
fn foo(b: Big) {
    let Big { field6, field3, .. } = b;
    println!("pulled out {} and {}", field3, field6);
}
```

这样就会自动创建两个跟域名字相同的本地变量，上面的例子中就是`field6`和`field3`。

还有几个Rust中解构的小问题。假如你想要得到模式中一个变量的引用，不能直接使用`&`符号，因为它们将匹配一个引用而非创建一个引用（而且会对对象进行一个解引用操作)。例如

```rust
struct Foo {
    field: &'static int
}

fn foo(x: Foo) {
    let Foo { field: &y } = x;
}
```

这里`y`的类型是`int`而且时一个`x`的拷贝而非引用。

要创建一个到某个模式的引用，你需要使用`ref`关键字，如下所示


```rust
fn foo(b: Big) {
    let Big { field3: ref x, ref field6, ..} = b;
    println!("pulled out {} and {}", *x, *field6);
}
```

这里`x`和`field6`的类型都是`&int`，而且它们都是对`b`对象中相应域的引用。

还有最后一个小问题，当你需要对复杂对象进行解构时，可能不仅对某个具体域，而对某个中间对象也感兴趣。在前面的例子中有一个模式`&Var4(3, s,45)`，在这个模式中我们将结构体的一个域命名为`x`。假如你想对整个`St`结构体进行命名，你可以写成`&Var4(3, s,
45)`，从而将结构体对象命名为`s`，然而如果之后还需要访问其中的一个域就需要再写一个域访问语句，另外如果只想匹配某个域为某个值的对象，就需要再写一个嵌套的`match`语句。这一点也不好玩。Rust允许在模式中使用`@`语法来命名模式中的一部分，比如我们可以写`&Var4(3, s @ St{f1: 3, f2: x }, 45)`，这样可以同时将特定域（`x`对应`f2`）和整个结构体(`s`)同时命名。

上面介绍了在Rust中使用模式匹配的几个方式，但愿能让你了解使用`match`和`let`能实现哪些强大的特性。另外还有一些特性（如向量匹配）还没有提到。下一章我会介绍一些匹配和借出之间微妙的相互关系。在我学Rust的时候它们让我困惑了好一阵子。