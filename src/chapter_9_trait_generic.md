## Trait
trait 是 Rust 对 Ad-hoc 多态的支持。 trait 是在行为上对类型的约束，可以使用 trait 来存储不同类型的值。 
### 实现trait
```rust
// 定义 Shape trait
trait Shape {
    fn area(&self) -> u32;
}

struct Rectangle {
    x: u32,
    y: u32,
}

struct Circle {
    r: f64,
}

// 为 Rectangle 实现 area 方法
impl Shape for Rectangle {
    fn area(&self) -> u32 {
        self.x * self.y
    }
}

// 为 Circle 实现 area 方法
impl Shape for Circle {
    fn area(&self) -> u32 {
        (3.14 * self.r * self.r) as u32
    }
}

fn main() {
    let rectangle = Rectangle { x: 30, y: 25 };
    let r_area = rectangle.area();
    println!("{}", r_area);

    let circle = Circle { r: 45.0 };
    let c_area = circle.area();
    println!("{}", c_area);
}
```
### 使用 derive 注释实现 trait
使用`derive注释`实现不同的`trait`，编译器在本质上可以为特定的trait提供基本实现。
```rust
#[derive(Debug)]
struct A(i32);

#[derive(Debug)]
struct B(f32);

fn main() {
    let a = A(29);
    let b = B(43.5);
    println!("a is: {:?}, b is: {:?}", a, b);
}
```
使用`derive注释`为结构体A和B实现`Debug trait`，这样就可以使用Debug模式打印结构体A或B。通常可以使用 derive注释实现的trait还包括`Clone`, `Copy`, `Eq`, `PartialEq`, `PartialOrd`, `Ord` 等

可以使用`trait`重载操作符，下面例子重载加法操作符
```rust
use std::ops;

struct A;
struct B;
#[derive(Debug)]
struct AB;
#[derive(Debug)]
struct BA;

impl ops::Add<B> for A {
    type Output = AB;

    fn add(self, _rhs: B) -> AB {
        AB
    }
}

impl ops::Add<A> for B {
    type Output = BA;

    fn add(self, _rhs: A) -> BA {
        BA
    }
}

fn main() {
    println!("{:?}", A + B);
    println!("{:?}", B + A);
    // println!("{:?}", A + A); // 报错，因为并没有impl ops::Add<A> for A
}
```
### Drop trait
`Drop trait`不需要手动调用，当变量生命周期结束时，它会被自动调用。
```rust
struct A {
    a: String,
}

impl Drop for A {
    fn drop(&mut self) {
        println!("dropped: {}", self.a);
    }
}

fn main() {
    let a = A { a: "A".to_string() };
    {
        let b = A { a: "B".to_string() };
        {
            let c = A { a: "C".to_string() };
            println!("leaving inner scope 2");
        }
        println!("leaving inner scope 1");
    }
    println!("programming end.");
}
```
运行结果
```
$ cargo run

leaving inner scope 2
dropped: C
leaving inner scope 1
dropped: B
programming end.
dropped: A
```
### Iterator trait
```rust
struct Fib {
    c: u32,
    n: u32,
}

impl Iterator for Fib {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        let n = self.c + self.n;
        self.c = self.n;
        self.n = n;

        Some(self.c)
    }
}

fn fib() -> Fib {
    Fib { c: 1, n: 1 }
}

fn main() {
    for j in fib().take(10) {
        println!("{}", j);
    }
    
    let mut f = fib();
    println!("{}", f.next());
    println!("{}", f.next());
    println!("{}", f.next());
    println!("{}", f.next());
    println!("{}", f.next());
}
```
## 泛型
声明函数签名或结构体等元素时使用泛型，然后搭配不同的具体类型来使用这些元素，使用泛型可以减少代码重复
```rust
struct Square<T> {
    x: T,
}

fn main() {
    let s = Square {x: 10};
    let s = Square {x: 1.0};
    let s = Square {x: "hello"};
    let s = Square {x: 'c'};
}
```
上面代码中使用`T`作为类型参数名称，Square 的字段x的类型可以是整型、浮点型、字符串字面量、字符等
### 泛型在函数中使用
```rust
use std::fmt;

fn p<T: fmt::Debug>(x: T) {
    println!("{:?}", x);
}

fn main() {
    p(10);
    p(String::from("string!"));
}
```
上面代码中函数定义`fn p<T: fmt::Debug>(x: T)` 表示泛型参数`T`必须是已经实现了`Debug trait`。
### 泛型在结构体中使用，并为结构体实现方法
```rust
struct A<T> {
    x: T,
}

impl <T> A<T> {
    fn item(&self) -> &T {
        &self.x 
    }
}

fn main() {
    let a = A{x: "hello"};
    a.item();
}
```
必须紧跟着`impl` 关键字声明`T`，这样Rust能够识别出`A`尖括号内的类型是泛型而不是具体类型。

结构体定义中的泛型参数有时候与方法签名上使用的类型参数会不一致
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point { x: self.x, y: other.y, } 
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "hello", y: 'c' };
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```
上面代码中`Point<T, U>`结构体定义了一个方法`mixup`。这个方法会接收另一个`Point`作为参数，而它与`self`参数所代表的`Point`之间可能拥有不同的类型。方法运行结束后会创建一个新的`Point`实例，
这个实例的`x`值来自`self`所绑定的`Point`（拥有类型T），而`y`值则来自传入的`Point`（拥有类型W）
### 泛型约束
```rust
use std::ops::Mul;

trait Shape<T> {
    fn area(&self) -> T;
}

struct Rectangle<T: Mul> {
    x: T,
    y: T,
}

impl<T> Shape<T> for Rectangle<T>
where
    T: Mul<Output = T> + Copy,
{
    fn area(&self) -> T {
        self.x * self.y
    }
}

fn main() {
    let r = Rectangle { x: 10, y: 20 };
    println!("{}", r.area());
}
```
上面代码使用了`T: Mul<Output = T> + Copy`对泛型进行了约束，表示`area`函数的参数必须实现`Mul trait` 和 `Copy trait`，并且乘号两边的数据类型必须一致。