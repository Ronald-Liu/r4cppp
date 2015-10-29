# Rust For C++ Programmers

一个为C/C++程序员写的Rust教程

转到 [目录](#contents).
转到 [Contributing](#contributing).

本教程是为已经熟悉指针、引用和系统编程基本概念的程序员写的。我们将主要介绍Rust和C++中的区别，以帮助你更快的用Rust编程，而不用花时间去学你已经熟知的知识。

对于C++程序员而言，Rust是一个很直观的语言——它们的大部分语法很相似。而在我看来其间最大的不同在于Rust编译器将一些系统编程中容易混淆的概念进行了更强的限制。这起初看起来会有些令人恼火——因为有些你原本能做的事Rust编译器禁止你去做，即使这些事情确实是安全的，只是编译器不相信而已。不过你很快你就可以培养出写可接受代码的直觉。为了在你和编译器之间就内存安全的问题建立共识，你需要学习一些新的，有时看起来过于繁复的类型标示。但当你对对象的生存期有更强的概念，并有一定经验后，它们就不会特别难学。

这一教程起先是[一系列博客文章](http://featherweightmusings.blogspot.co.nz/search/label/rust-for-c)。我撰写它们，一部分原因是想帮助我自己(@nick29581)学习Rust，另一部分原因是我发现往上现存的Rust学习资源很难令人满意：它们花了太多的时间介绍我已经知道的基础知识，并利用了更高层的知识来讨论Rust中的概念，而我认为使用底层知识介绍它们更能说明问题。过了一段时间，Rust的官方文档已经有了很大的进步，然而我仍然认为C++的程序员，作为最适合学习Rust语言的人，却没有被文档编写者重视。

## 安装Rust

安装Rust的简单方法（至少在Linux和OS X上）是运行`rustup`的脚本，只需要在命令行中输入以下命令

### Beta 版本(推荐）
```
curl -s https://static.rust-lang.org/rustup.sh | sudo sh
```

### Nightly 版本
如果你需要尚未更新到rust-beta中的那些包，可以使用Nightly版本
```
curl -s https://static.rust-lang.org/rustup.sh | sudo sh -s -- --channel=nightly
```

你可以从[rust-lang.org](http://www.rust-lang.org/install.html)下载运行于Linux, OS X和Windows的nightly二进制版本。

你也可以从[github.com/rust-lang/rust](https://github.com/rust-lang/rust)中下载或运行`git clone https://github.com/rust-lang/rust.git`来获取最新的源代码。要构建Rust编译器只需要运行`./configure && make rustc`。你可以在[building-from-source](https://github.com/rust-lang/rust#building-from-source)查看更详细的安装指南。


## 其它资源

* [The Rust book/guide](http://doc.rust-lang.org/book/) - 全面学习Rust的最好去处，同时也是学习本文没介绍内容的最佳选择
* [Rust API documentation](http://doc.rust-lang.org/std/index.html) - Rust库的详细文档
* [The Rust reference manual](http://doc.rust-lang.org/reference.html) - 略微过时的文档，但是还不错
* [Discuss forum](http://users.rust-lang.org/) - 讨论使用和学习Rust时各类问题的论坛
* [#rust irc channel](https://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust) - 如果你热爱使用IRC，这可能是你要找到某个答案最快的地方
* [Stackoverflow Rust questions](https://stackoverflow.com/questions/tagged/rust) - 寻找有关Rust的基础和高级问题的地方，不过请注意，Rust在这些年来已经有了*非常大*的变化，有些答案在今天可能已经不再适用。

## Contents

1. [Hello world!](hello%20world.md)
1. [花絮 - 为啥要用Rust?](why%20rust.md)
1. [控制流](control%20flow.md)
1. [基本数据类型和操作符](primitives.md)
1. [Unique指针](unique.md)
1. [借出指针](borrowed.md)
1. [Rc与原始指针](rc%20raw.md)
1. [数据类型](data%20types.md)
1. [解构 I](destructuring.md)
1. [解构 II](destructuring 2.md)
1. [Array与vec](arrays.md)
1. [实例：图与内存预分配](graphs/README.md)


## Contributing

跪求！

如果你发现了错别字或者逻辑错误，赶紧提出Pull Request，千万别害羞！对于更大的变化或者你想看到的其他章节，欢迎提出新[issue](https://github.com/nick29581/r4cppp/issues/new)。如果重新组织已有的文章或者增加新的样例可以让这个教程变的更好，我会非常高兴。

如果你想贡献一个段落，一个部分或者整个章节，非常欢迎！如果你希望对某些方面有更多介绍，请看[list of issues](https://github.com/nick29581/r4cppp/issues)，尤其是那些打了 [new material](https://github.com/nick29581/r4cppp/labels/new%20material)Tag的Issue。如果你不确定，欢迎与我联系（@nick29581)或者在IRC里提出 (nrc, 我在 #rust 和 #rust-internals里活动)


### 文档风格

既然本教程主要针对C++程序员，这里主要介绍的是那些对C++程序员来说不太熟悉的概念（不过并不需要你对最新版本的C++很了解）。我希望能避免介绍太多的基础知识，而且不想和其他资源，尤其是Rust官方文档有过多知识上的重叠。

欢迎对前沿问题的讨论（比如用除了Cargo以外的构建系统，编写语法扩展，使用非稳定的API）。

我希望避免写出像“如何将下面的C++代码写成Rust“之类的例子，不过如果有一小部分也是可以的。

欢迎使用不同的文档格式（比如FAQs或者更大的实例)

我不打算添加对迷你项目的练习或者建议，然而如果你有兴趣，请联系我

我希望能表现出一副很学术的样子，但也不是那么死板。所有的文字要用（英式）英语写成，当然也欢迎翻译为其它语言（你可以直接提交中文给我——译者），并使用GitHub markdown格式。就写作风格、语法、拼写等等的问题欢迎参看牛津风格手册或 [The Economist Style Guide](http://www.economist.com/styleguide/introduction)

不要强求你的作品必须完美，我很欢迎其他人一起来编辑和修改

##译者续貂
很早就想学Rust，苦于没有很好的入门教程。 Rust for C++ Programmers对于像我一样的 C 艹程序员来说还是非常有帮助的。网络上介绍Rust的资源本就稀少，中文资源更不必说的极端稀缺。欢迎所有对Rust有兴趣的朋友来一起完善本文，一起帮助Rust在C++程序员，尤其是中国C++程序员中进一步的传播。