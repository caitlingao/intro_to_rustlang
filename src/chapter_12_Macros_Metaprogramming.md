## 宏
宏是Rust中某一组相关功能的集合。包括使用`macro_rules!`构造的声明宏和3种过程宏。
* 用于结构体或枚举的自定义`#[derive]`宏，它可以指定随`derive`属性自动添加的代码。
* 用于为任意条目添加自自定义属性的属性宏。
* 函数宏，它可以接收并处理一段标记序列。
```rust
macro_rules! a_macro {
    () => {
        println!("This is a macro"); 
    }
}

fn main() {
    a_macro!();
}
```
宏是使用`macro_rules!`定义，它后面紧跟着宏的名称`(a_macro)`，上面代码存在一个`() => {}`的模式分支，这些代码会在模式匹配成功是触发。
宏要求编写出类似`match`表达式的东西。但是它与`match`不同，宏不会编译任何东西，只是对传入的Rust源代码进行模式匹配。
```rust
macro_rules! six_or_print {
    (6) => {6};
    () => {println!("You didn't give 6.")};
}
fn main() {
    six_or_print!(6);
    six_or_print!();
}
```
```rust
macro_rules! might_print {
    ($input:expr) => {
        println!("You gave me: {}", $input); 
    };
}

fn main() {
    might_print!(6);
    might_print!(20 + 90);
}
```
上面代码中`$input:expr`表示可以匹配任意Rust表达式，并将它命名为`$input`。在宏中变量名是以`$`开头的。

Rust中还会使用`ident`，它的含义是指标识符，它可以指代变量或函数名称
```rust
macro_rules! check {
    ($input1:ident, $input2:expr) => {
        println!(
            "Is {:?} equal to {:?}? {:?}",
            $input1,
            $input2,
            $input1 == $input2
        );
    };
}

fn main() {
    let x = 6;
    let my_vec = vec![7, 8, 9];
    check!(x, 6);
    check!(my_vec, vec![7, 8, 9]);
    check!(x, 10);
}
```
```rust
macro_rules! build_fn {
    ($func_name: ident) => {
        fn $func_name() {
            println!("You called {:?}()", stringify!($func_name)); 
        }
    };
}

fn main() {
    build_fn!(hello_world);
    hello_world();
}
```
宏也可以进行重载
```rust
macro_rules! exame {
    ($l:expr; and $r:expr) => {
        println!("{:?} and {:?} is {:?}", stringify!($l), stringify!($r), $l && $r); 
    };
    ($l:expr; or $r:expr) => {
        println!("{:?} or {:?} is {:?}", stringify!($l), stringify!($r), $l || $r); 
    };
}

fn main() {
    exame!(1==1; and 1+1==2);
    exame!(true; or false);
}
```
另一个复杂的宏
```rust
macro_rules! compr {
    ($id1:ident | $id2:ident <- [$start:expr; $end:expr], $cond: expr) => {
        {
            let mut vec = Vec::new();
            for num in $start..$end + 1 {
                if $cond(num) {
                    vec.push(num); 
                } 
            } 
            vec
        } 
    };
}

fn even(x: i32) -> bool {
    x % 2 == 0
}

fn odd(x: i32) -> bool {
    x % 2 != 0
}

fn main() {
    let evens = compr![x | x <- [1;10], even];
    println!("{:?}", evens);
    let odds = compr![y | y <- [1;10], odd];
    println!("{:?}", odds);
}
```
宏可以重复多次
```rust
use std::collections::HashMap;

macro_rules! new_map {
    ($($key: expr => $val: expr),+) => {
        {
            let mut map = HashMap::new();
            $(
                map.insert($key, $val);
            )* 
            map
        }
    };
}

fn main() {
    let m = new_map!{
        "one" => 1,
        "two" => 2,
        "three" => 3
    };
    println!("{:?}", m);
}
```
`$()`后面的`,`表示可能会出现在捕获代码的后面，`+`意味着这个模式能够匹配多个`+`之前的东西