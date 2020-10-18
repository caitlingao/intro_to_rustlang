## 生命周期
### 使用生命周期避免悬垂指针
生命周期最主要的目标在于避免悬垂引用。
```rust
fn main() {
    let r;
    {
        let x = 5;
        r = &x; 
    }
    println!("r: {}", r);
}
```
上面代码运行后会报错
```
   Compiling playground v0.0.1 (/playground)
error[E0597]: `x` does not live long enough
 --> src/main.rs:5:13
  |
5 |         r = &x; 
  |             ^^ borrowed value does not live long enough
6 |     }
  |     - `x` dropped here while still borrowed
7 |     println!("r: {}", r);
  |                       - borrow later used here

error: aborting due to previous error

For more information about this error, try `rustc --explain E0597`.
error: could not compile `playground`.

To learn more, run the command again with --verbose.
```
错误信息指出变量`x`的戚周期不够长。变量`x`存活范围是代码第3~第6行，而变量`r`对于整个作用域都是有效的。在代码第5行，变量`r`指向了变量`x`的引用，
第6行，变量`x`的生命周期结束，变量`x`存储的值就会被销毁，此时就会出现悬垂指针。
### 函数中的泛型生命周期
```rust,ignore,does_not_compile
fn pr(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
上面代码会提示缺少生命周期参数错误。在定义这个函数时，并不知道传入函数的具体值，所以也不能确定哪个分支会被执行，也无法知道传入的引用的具体生命周期，所以无法通过分析作用域来确定返回的引用是否有效。
为了解决这个问题，需要添加一个泛型生命周期参数，并用它定义引用之间的关系。
```rust,ignore,does_not_compile
fn pr<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
上面代码表明，函数所获取的两个字符串切片参数的戚时间，必须不我在玩给你写的生命周期`'a`
```rust,ignore,does_not_compile
fn pr<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
上面代码中函数返回的是第一个字符切片，这时就不需要再为y参数指定生命周期，因为y的生命周期与x和返回值的生命周期没有任何相互关系。
```rust,ignore,does_not_compile
fn pr<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```
上面代码中返回类型指定了生命周期参数`'a`，也无法编译通过，因为返回值的生命周期没有与任何参数的生命周期产生关联。
### 结构体中定义生命周期
```rust
struct A<'a, 'b> {
    x: &'a str,
    y: &'b str,
}

fn main() {
    let x = A{x: "hello", y: "there"};
}
```
### 方法定义中的生命周期标注
```rust,ignore,doesnot_compile
impl<'a> A<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```
