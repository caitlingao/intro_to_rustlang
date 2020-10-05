## 基本数据类型
### 整数类型
Rust的整数类型主要区分特征是`有符号`和`无符号`，占据空间大小如下图

|整数类型|有符号|无符号|
|------|-----|-----|
|8 bits|i8|u8|
|16 bits|i16|u16|
|32 bits|i32|u32|
|64 bits|i64|u64|
|128 bits|i128|u128|
|pointer size|isize|usize|

所有的数字字面量中，可以在任意地方添加任意的下划线。如：`let var5 = 0x_123_ABCD;`

字面量后面可以跟后缀，代表该数字的具体类型，省略显示类型标记
```Rust
let var6 = 123usize;
let var7 = ox_ff_u8;
let var8 = 32;
```
### 浮点类型
`f32`和`f64`，浮点类型不具备`全序关系`。标准库中`std::num::FpCategory`枚举
```Rust
pub enum FpCategory {
    Nan,
    Infinite,
    Zero,
    Subnormal,
    Normal,
}
```
其中`Zero`表示0值，`Normal` 表示正常状态的浮点数，`Infinite` 代表 "无穷大"，`Nan` 代表 "不是数字"
### Bool 类型
true/false，可以通过 as 操作符将 bool 类型 true 转为 1， false 转为 0.
### 字符类型
由`char`表示，由单引号包围。字符类型代表的是一个 Unicode 标准的字符值，它占用的空间不是1个字节，而是4个字节。
```Rust
let c: char = 'z';
```
### 数组类型
数组类型数组大小固定，数组内元素类型一致，默认不可变。数组类型签名是`[T;N]`。访问数组中的元素，可以通过数字索引。
```Rust
let a = [1, 2, 3, 4];
let a1 = a[0];
```
### 切片类型(slice)
切片类型是对一个数组的引用片段，可以安全有效地访问数组的一部分而不需要拷贝。切片类型`&[T]`或`&mut [T]`
```Rust
fn main() {
    let xs: [i32; 5] = [4, 5, 6, 7, 8];
    let ys = &xs[2..4]; // 数组切片
    assert_eq!(ys, [6, 7]);
}
```
### 字符串
Rust字符串有两种类型，一种是`&str`，它是固定长度字符串不可随便更改其长度。一种是`String`，它是可增长字符串。
#### &str
`str`是Rust的内置类型，`&str`是对`str`的借用，也叫作字符串切片，它对所指向的内存空间没有所有权。 &str 和数组切片行为很相似。
```Rust
fn main() {
    let greeting = "Hello"; // greeting 数据类型是 &str
    let substr: &str = &greeting[2..];
    println!("{}", substr);
}
```
Rust的字符串内部默认是使用utf-8编码格式，而内置的`char`类型是4字节长度的，存储的内容是 Unicode Scalar Value。所以，Rust里面的字符串不能视为 char 类型的数组，而是接近 u8 类型的数组。如果想访问字符串内部的第n个字符，需要使用下面的方式`s.chars().nth(n)`
#### String
`String` 它在堆上动态申请了一块内存空间，有管理内存空间的权力。
```Rust
fn main() {
    let mut greeting = String::from("Hello");
    greeting.push(' ');
    greeting.push_str("World.");
    println!("{}", greeting);
}
```
`&str`类型可以转换为`String`类型，转换语法如下：
```Rust
let s = "hello".to_string();
```
## 复合数据类型
### tuple
元组类型，通过圆括号包含一组表达式构成，tuple内的元素没有名字，元素内元素类型可以不相同，元素没有固定长度。

*元素内只包含一个元素时，应该在后面添加逗号，以区分括号表达式和元组*
```Rust
let a = (0,); // 是一个元组，它有一个元素
let b = (0); // 是一个括号表达式，它是i32类型
```
访问元组内元素有两种方式，一种是`模式匹配`，一种是`数字索引`
```Rust
let t: (i32, f64, char) = (42, 6.12, 'j');

let (x, y, z) = t; // 模式匹配方式访问

// 数字索引方式访问
let x = t.0;
let y = t.1;

let (_, _, z) = t; // 忽略第一个与第二个元素，将第三个元素赋值给z
```
