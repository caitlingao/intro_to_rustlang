## 控制流与模式匹配
### 控制流
#### if 表达式
Rust 中`if`表达式的分支必须返回**同一个类型的值**。这也是 Rust 没有三元操作符`? :`的原因。
```rust
fn main() {
    let number = 3;
    if number < 5 {
        println!("condition was true"); 
    } else {
        println!("condition was false"); 
    }
}
```
##### 在let语句中使用if

因为 `if` 是一个表达式，所以可以在 `let` 语句的右侧使用它来生成一个值。
```rust
fn main() {
    let condition = true;
    let number = if condition {
        5 
    } else { 
        6
    };

    println!("The value of number is: {}", number);
}
```
#### 循环表达式
##### loop 循环
使用`loop`关键字反复执行某一块代码
```rust
fn main() {
    loop {
        println!("again!");
    }
}
```
使用`continue`和`break`控制流程。`continue;`语句表示本次循环内后面的语句不再执行，直接进入下一轮循环。`break;`语句表示跳出循环，不再继续。
```rust
fn main() {
    'a: loop {
        println!("loop a");
        'b: loop {
            println!("loop b");
            'c: loop {
                println!("loop c");
                break 'b; // 结束跳出 loop b
            }
        }
        continue 'a; // 继续执行 loop a
    }
}
```
`loop`也可以作为表达式的一部分
```rust
fn main() {
    let x = loop {
        break 10; 
    };
    println!("x: {}", x);
}
```
##### while 条件循环
```rust
fn main() {
    let mut number = 10;
    while number != 0 {
        println!("number: {}", number);
        number = number - 1; 
    }
    println!("LIFTOFF!!")
}
```
##### for 循环遍历集合
```rust
fn main() {
    let a = vec![10, 20, 30, 40, 50];
    for num in a {
        println!("num: {}", num); 
    }
}
```
```rust
fn main() {
    for num in 1..100 {
        println!("num: {}", num); 
    }
}
```
`for...in`表达式本质上是一个迭代器，`1..101`是一个`Range`类型，`for`的每一次循环都从迭代器中取值，当迭代器中没有值了循环结束。`1..101`这个`Range`类型的值范围是`[1,100)`即不包含最后一值101，若想要包含最后一个值，写法可以是`1..=100`
### 模式匹配
#### match
match用于匹配各种情况，类似其它语言中的switch 或 case，最直观的便是匹配字面量
```rust
fn main() {
    let x = 5;
    match x {
        1 => println!("one"), 
        2 => println!("two"), 
        3 => println!("three"), 
        4 => println!("four"), 
        5 => println!("five"), 
        _ => println!("something else"), 
    }
}
```
match 中也可以有多重模式和使用`...`来匹配区间。match 表达式的分支匹配中使用`|`来表示或，它可以用来一次性匹配多个模式。用`...`匹配闭区间的值。
```rust
fn main() {
    let n = 15;
    match n {
        1 => println!("one"), 
        2 | 3 | 5 | 7| 11 => println!("This is a prime"), 
        13...19 => println!("A teen"), 
        _ => println!("Ain's special"), 
    }
}
```
#### 解析结构体
```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point{x: 0, y: 7};
    match p {
        Point {x, y: 0} => println!("On the x axis at {}", x), 
        Point {x: 0, y} => println!("On the y axis at {}", y), 
        Point {x, y} => println!("On the x axis at ({}, {})", x, y), 
    }
}
```
实例p的x字段值为0，它会匹配到第二个分支，最终打印`On the y axis at 7`
#### 忽略模式中的值
可以使用`_模式、使用下划线开头的名称、或者..`忽略值的剩余部分
##### 使用_忽略整个值
```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}
fn main() {
    foo(3, 4);
}
```
##### 以_开头的名称忽略未使用的变量
```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```
##### 使用..忽略值的剩余部分
```rust
struct Point {
    x: i32,
    y: i32,
    z: i32
}

fn main() {
    let p = Point{x: 0, y: 0, z: 0};
    match p {
        Point {x, ..} => println!("x is {}", x)
    }
}
```
这段代码在分支模式中首先列出了变量x，接着列出了一个`..`模式。这种语法比具体写出`y: _`和`z: _`要便捷一些。

`..`语法会自动展开并填充任意多个所需的值
```rust
fn main() {
    let numbers = (2, 4, 6, 8, 10);
    match numbers {
        (first, .., last) => println!("Some numbers: {}, {}", first, last),
    }
}
```
#### 使用匹配守卫添加额外条件
匹配守卫是附加在match分支模式后的if条件语句，分支中的模式只有在该条件被同时满足时才能匹配成功。
```rust
fn main() {
    let pair = (5, -5);
    match pair {
        (x, y) if x == y => println!("Equal"), 
        (x, y) if x + y == 0 => println!("Equal Zero"), 
        (x, _) if x % 2 == 0 => println!("x is even"), 
        _ => println!("No match"), 
    }
}
```
上面代码中会匹配`(x, y) if x + y == 0`这条分支
#### @绑定
`@`运算符允许在测试一个值是否匹配模式的同时创建存储该值的变量。
```rust
fn main() {
    let p = 5;
    match p {
        n @ 1...12 => println!("n: {}", n), 
        n @ 13...19 => println!("n: {}", n), 
        _ => println!("No match"), 
    }
}
```
#### if let 与 while let
用来只关心某一种匹配而忽略其它匹配的情况
```rust
fn main() {
    let some_u8_value = Some(0u8);
    match some_u8_value {
        Some(3) => println!("three"),
        _ => (), 
    }
}
```
可以用 `if let`更简短的方式来实现。可以把`if let`看作是`match`的语法糖，只在值满足某一特定模式时运行代码，而忽略其它所有可能性。
```rust
fn main() {
    let some_u8_value = Some(0u8);
    if let Some(3) = some_u8_value {
        println!("three"); 
    }
}
```
`while let` 与 `if let`类似
```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    loop {
        match v.pop() {
            Some(x) => println!("{}", x),
            None => break,
        }
    }
}
```
使用 `while let` 简化
```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    while let Some(x) = v.pop() {
        println!("{}", x);
    }
}
```
