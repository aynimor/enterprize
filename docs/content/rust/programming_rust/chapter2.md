### 2.1 下载和安装

安装：[https://www.rust-lang.org/learn/get-started](https://www.rust-lang.org/learn/get-started)

``` shell
$ cargo --version
cargo 1.59.0 (49d8809dc 2022-02-10)
$ rustc --version
rustc 1.59.0 (9d1b2106e 2022-02-23)
$ rustdoc --version
rustdoc 1.59.0 (9d1b2106e 2022-02-23)
$ 
```

- cargo 是 Rust 的编译管理工具、包管理工具以及通用工具。可以使用 Cargo 来创建新项目、构建和运行程序，以及管理代码所依赖的外部库
- rustc 是 Rust 编译器。但一般不直接使用。
- rustdoc 是 Rust 文档工具。

使用 cargo 创建新的项目：

``` shell
$cargo new hello-world
    Created binary (application) `hello-world` package
```

目录下包含以下文件

``` 
total 0
drwxrwxrwx 1 root root 4096 Feb 27 17:07 .git
-rwxrwxrwx 1 root root    8 Feb 27 17:07 .gitignore
-rwxrwxrwx 1 root root  180 Feb 27 17:07 Cargo.toml
drwxrwxrwx 1 root root 4096 Feb 27 17:07 src
```

Cargo 会自动创建 git 仓库，并生成 Cargo.toml 依赖库配置文件，以及存放源码的文件夹 ./src，其中包含一个 main.rs文件，该文件包含以下内容：

``` rust
fn main() {
    println!("Hello, world!");
}
```

使用 `cargo run` 命令开始构建程序，并运行程序，shell 中将会输出：`Hello, world!`。

### 2.2 一个简单的函数

```rust
fn gcd(mut n: u64, mut m: u64) -> u64 {
	assert!(n != 0 && m != 0);
	while m != 0 {
        if m < n {
            let t = m;
            m = n;
            n = m;
        }
        m = m % n;
    }
    n
}
```

这个 Rust 的一个函数的定义，用于计算两个整数的最大公约数。关键字 `fn` 定义了一个名为 `gcd` 的函数，其接受两个参数 `n` 和 `m`，类型均为 `u64`（无符号64为整数），符号 `->` 后面是函数的返回值类型。Rust 缩进一般约定 4 个空格。

`mut` 为特殊关键字，后续将会学到，此处需要使用 `mut` 来标识，说明该变量是可变的，否则程序将无法被编译。

`assert!()` 为 Rust 中的一个 **宏**，宏也是 Rust 中一个重要的部分，后续还会用到，`assert` 宏的含义与 `C` 中的 assert 类型。

`while` 关键字则说明一个循环体，其后的表达式 `m != 0` 是循环继续进行的条件

`if` 关键字是一个标准的控制流，与 C 类似

`let` 用于指定声明一个变量

#### Rust 基本数据类型

> 在 Rust 中，每一个值都属于某一个 **数据类型**（*data type*），这告诉 Rust 它被指定为何种数据，以便明确数据处理方式。我们将看到两类数据类型子集：标量（scalar）和复合（compound）。
>
> Rust 是 **静态类型**（*statically typed*）语言，也就是说在编译时就必须知道所有变量的类型。

Rust 整数类型共分为：`i8`，`i16`，`i32`，`i64`，`i128`，`isize`，`u8`，`u16`，`u32`，`u64`，`u128`，`usize`。

`i` 和 `u` 表示整数的符号位，`i` 为有符号整数，`u` 为无符号整数。数字则表示该类型所占的位数，`i8` 则为  8 位有符号整数。`isize` 和 `usize` 为特殊情况，其依赖于运行该程序的计算机架构：64 位架构上是 64 位整数，32 位架构上则是 32 位整数。

使用方式大致如下：

```rust
fn main() {
    let a: i32 = 10;
    let b: i32 = 100_000;	// 使用 `_` 分隔，可以方便读数
    let c = 100i32;
    let d = 10;				// 若不指定类型，Rust 会自动推导类型，整型默认 i32，浮点默认 f64
    let e = 10.0;			// f64
}
```

除整数类型外，Rust 还包含浮点数类型、布尔类型（`bool`）、字符类型（`char`）、元组类型、数组类型。其中整数类型、浮点数类型、布尔类型、字符类型又被称为标量类型；元组类型、数组类型又被称为复合类型。

浮点类型分为：`f32`，`f64`，可以将其简单的对应到 C 的 `float` 和 `double`

布尔类型分为：`true`，`false`

字符类型是由 `'` 单引号包围起来的单个字符，如：`let a: char = 'c';`，由于 Rust 的编码是 `UTF-8` 编码，所以 `let heart_eyed_cat = '😻';` 也是成立的。

元组类型指由小括号包含的值，其中每个值由逗号分隔，元组中每个类型可以不一致。元组长度固定，一旦声明，其长度将无法被修改

```rust
fn main() {
    let tuple: (i32, f64, char) = (10, 2.3, 'R');
    let (x, y, z) = tuple;							// 此方法为 Rust 中的模式匹配语法，后续还会说到
    let a = tuple.0;								// 通过索引取指定值
    let b = tuple.1;
    let c = tuple.2;
}
```

数组类型由中括号包含，其中的值由逗号分隔。与元组不同，数组中的每个元素类型必须是相同的，Rust 中的数组不同于其他编程语言，Rust 中的数组长度是固定的，不能增加或减少（如果想要获得和其他编程语言一样的数组，应该使用 Rust 中的 vector 类型）。

数组一般用于知道固定长度的元素集合如：一年十二个月，四个季节等。

``` rust
// 数组声明
fn main() {
    let month: [i32; 12] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12];		// i32 指定类型，12 指定数组长度
    let january = month[0];		// 索引从 0 开始
    let unkwon = month[13];		
    // 抛出异常：thread 'main' panicked at 'index out of bounds: the len is 11 but the index is 13'
}
```

当访问无效的数组元素时，Rust 编译不会报错，程序将会出现一个 **运行时（*runtime*）** 错误，Rust 中对此类错误使用 panic 术语来描述。

### 2.3 编写和运行单元测试

``` rust
#[test]
fn test_gcd() {
    assert_eq!(gcd(14, 15), 1);
    
    assert_eq!(gcd(2 * 3 * 5 * 11 * 17,
    			   3 * 7 * 11 * 13 * 19),
    		   3 * 11);
}
```

在 `shell` 中运行命令 `cargo test`，cargo 将会寻找整个包中的所有包含 `#[test]` 的函数并运行，`#[test]` 标记是 **属性** 的一个例子。

### 后续

我们将跳过第二章后三个小节的学习，直接开始学习第三章的内容，在学习完本书内容之后将会对这三小节的内容有深刻的理解

\>\> Next: Rust 程序设计 第 3 章 基本类型