## 智能指针
智能指针行为与指针类似，但是它拥有额外的元数据和附加功能。 `Box`是最简单直接的一种智能指针，它的类型被写作`Box<T>`。`Box`可以将数据存储在堆上，并在栈上保留一个指向堆数据的指针。
```rust
fn main() {
    let b = Box::new(10);
    println!("b = {}", b);
}
```
上面代码定义了一个持有`Box`的值的变量b，它指向了堆上的值10，与其它拥有所有权的值一样，`Box`会在离开自己的作用域时(b到达main函数的结尾时)被释放。

### 把Box<T>当成引用来操作
```rust
fn main() {
    let y = 4;
    let x = &y;
    let z = Box::new(y);

    if *x == *z {
        println!("true"); 
    }
}
```
## 闭包
闭包也可被叫做匿名函数
```rust
// 普通函数
fn f(i: i32) -> i32 { i + 1 }

fn main() {
    // 闭包
    let f = |i: i32| -> i32 { i + 1 };
    // let f = |i| { i + 1 }; // 对上面的简化
    // let f = |i| i + 1; // 对上一行代码的简化
    let x = 10;
    println!("result is {}", f(x));
}
```
闭包可以捕获外部环境变量。下面代码中变量c的作用域范围在整个main函数内，闭包函数inc获取外部环境变量c
```rust
fn main() {
    let mut c = 0;
    let mut inc = || {
        c += 1;
        println!("incremented by 1: {}", c); 
    };

    inc();
    inc();
    inc();
}
```
### 闭包作为函数签名
```rust
fn run<F>(f: F) where F: Fn() {
    f();
}

fn add3<F>(f: F) -> i32
where F: Fn(i32) -> i32 {
    f(3)
}

struct A<F: Fn(i32) -> i32> {
    f: F,
}


fn main() {
    let p = || println!("run from function");
    run(p);

    let x = |i| i * 10;
    println!("add3 result is: {}", add3(x));

    let a = A { f: x };
}
```
### 闭包作为函数返回值
rust 中函数返回值必须是具体的数据类型不可以是泛型，所以如果使用闭包作为函数返回值需要通过智能指针`Box`
```rust
fn create() -> Box<Fn()> {
    Box::new(|| println!("this is a closure in a box."))
}

fn main() {
    let x = create();
    x();
}
```
如果希望闭包获取环境变量的所有权，那在闭包的参数列表前需要加上`move`关键字
```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;
    println!("can't use x here: {:?}", x);
}
```
## 使用迭代器处理元素序列
Rust的迭代器是惰性的。也就是说创建迭代器后，除非 主动调用方法来消耗并使用迭代器，否则它们不会产生任何的实际效果
```rust
fn main() {
    let v1 = ve![1, 2, 3];
    let v1_iter = v1.iter();
    for val in v1_iter {
        println!("Gog: {}", val); 
    }
}
```
上面代码中`let v1_iter = v1.iter()`通过调用`Vec<T>`的`iter`方法创建了一个用于遍历动态数组`v1`的迭代器，这段代码本身并不会产生任何影响。只有当下面的`for`循环开始执行时，迭代器才开始为第一次循环产生一个元素并将元素打印出来。
### Iterator trait 和 next 方法
所有迭代器都实现了`Iterator trait`，这个`trait`定义如下
```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```
上面代码中`type Item` 是`trait`的关联类型，这个`Item`类型会被用作`next`方法的返回值类型。也就是说`Item`类型将是迭代器返回元素的类型。

`Iterator trait` 要求实现者手动定义一个方法`next`方法，它会在每次被调用时返回一个包裹在`Some`中的迭代器元素，并在迭代结束时返回`None`。
```rust
fn main() {
    let v1 = vec![1, 2];
    let mut v1_iter = v1.iter();
    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), None);
}
```
### 消耗迭代器
```rust
fn is_even(n: u32) -> bool {
    n % 2 == 0
}

fn main() {
    let top = 10000;
    let mut c = 0;

    for n in 0.. {
        let x = n * n;
        if x >= top {
            break; 
        }  else if is_even(x) {
            c += x; 
        }
    }
    println!("{}", c);
}
```
使用迭代器实现
```rust
fn is_even(n: u32) -> bool {
    n % 2 == 0
}

fn main() {
    let top = 10000;
    let c: u32 = (0..).map(|n| n * n).take_while(|&n| n < top).filter(|&n| is_even(n)).fold(0, |s, i| s + i);
    println!("{}", c);
}
```