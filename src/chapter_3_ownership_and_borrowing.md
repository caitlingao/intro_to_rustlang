## 所有权和数据转移
### 所有权
栈和堆是代码在运行时可以使用的内存空间。栈是"后进先出"策略，所有存储在栈中的数据必须拥有一个已知且固定的大小。对于在编译期无法确定大小的数据，只能存储在堆上。堆空间的管理是松散的。

一般来讲，所有的程序都会需要管理自己在运行时使用的计算机内存空间。一些垃圾回收机制的语言，比如：Java、C#、Python等，会在运行时定期检查并回收那些没有被继续使用的内存；而像C++、C这些语言则需要程序员手动地分配和释放内存。而Rust采用了第三种方式：它使用包含特定规则的所有权系统来管理内存，这套规则允许编译器在编译过程中执行检查工作，而不会产生任何的运行时开销。

Rust 所有权规则包括：
* Rust 中的每一个值都有一个对应的变量作为它的所有者；
* 在同一个时间内，值有且仅有一个所有者；
* 当所有者离开自己的作用域时，它持有的传正就会被释放掉；
```Rust
fn main() {
    let x = 1;
    {
        let a = 10;    
    }
    x + a // cannot find value `a` in this scope
}
```
变量从出生到死亡的整个阶段叫一个变量的`生命周期`，上面代码中变量`a`的生命周期在`{}`这段块语句内，当块语句结束时，变量`a`的生命周期也就结束了，所以在`x+a`这处代码会报错。
#### 移动语义
一个变量可以把它拥有的值转移给另外一个变量，称为`所有权转移`，它是所有类型的**默认语义**
```Rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}", s1);
}
```
执行这段代码会报错
```Rust
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:4:20
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `std::string::String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |     println!("{}", s1);
  |                    ^^ value borrowed here after move
```
在`let s2 = s1;`语句中，原本由s1拥有的字符串已经转移给了s2，在后面继续使用s1便会报错。

这样设计的原理：`String`内存布局是这样的，`String::from`创建出来的字符串会存在堆中，在栈上会存放这个字符串的指针，栈上存放的内容包括3部分，一个指向存放字符串内容的指针（ptr），一个长度（len），一个容量（capacity）。当s1赋值给s2时，便将栈上的内容复制了一份，包括指针、长度、容量字段，但是并没有复制指针指向的堆数据。Rust当一个变量离开当前的作用域时，Rust会自动调用它的drop函数，并将变量使用的堆内存释放回收。现在会有栈上的两个指针指向堆内存的同一条数据，为了确保内存安全，同时避免复制分配的内存，Rust在这种场景下会简单地将s1废弃。

|name|value| 
|----|-----|
|ptr||
|len|5|
|capacity|5|

当确实需要深度拷贝堆上的数据，而不仅仅是栈数据时，可以使用`clone`方法来复制堆上的数据
```Rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);
```
#### 复制语义
任何时候需要复制都去调用`clone`方法会比较烦琐，对于一些简单类型，如：整数、bool、等在赋值时会采用复制语义。这些类型在编译期时可以确定自己的大小，并且能够将自己完整的数据存储在栈中，这些值的复制操作是非常快的。
```Rust
let x = 5; // 将整数5绑定到变量x上
let y = x; // 创建一个x的拷贝，将它绑定到y上。
println!("x = {}, y = {}", x, y);
```
Rust提供了名为`Copy`的`trait`，一旦某种类型拥有了`Copy`这种`trait`，它的变量就可以在赋值时给其它变量后保持可用性。

拥有`Copy trait`的类型包括：
* 所有整数类型
* bool 类型
* 字符类型：char
* 所有浮点类型
* 如果元组包含的所有字段类型都是`Copy`的，那么这个元组也是`Copy`的。

**如果一种类型本身或者这种类型的任意成员实现了`Drop trait`，这种类型不允许实现`Copy trait`**
#### 引用与借用
变量对其管理的内存拥有所有权，这个所有权可以被转移，也可以被借用（borrow）`&`或者`&mut`代表的就是引用语义，它允许在不获取所有权的前提下使用值。
```Rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // &s1 在不转移所有权情况下创建一个指向s1值的引用
    println!("The length of '{}' is {}", s1, len);
}

fn calculate_length(s: &String) -> usize { // s 是一个指向 String 的引用
    s.len()
} // s 离开作用域，但是由于它不持有自己所指向值的所有权，它离开自己的作用域时不会销毁指向的数据
```
可变引用
```Rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s); // 使用 &mut 给函数传入一个可变引用
}

fn change(s: &mut String) { // &mut String 接收一个可变引用作为参数
    s.push_str(", world");
}
```
#### 更多例子
```Rust
fn take(v: Vec<i32>) {
    println!("We took v: {}", v[10] + v[100]);
}

fn main() {
    let mut v = Vec::new();
    for i in 1..1000 {
        v.push(i); 
    }
    take(v); // move 语义
    // println!("{}", v[0]); // 这里无法再使用 v
    println!("Finished!");
}
```
变量v是动态数组，数据存储在堆上，`take(v)`是 move 语义，这段语句下面的 `println!("{}", v[0]);`这句会报错，因为v已被move
```Rust
fn cop(a: i32, b: i32) {
    println!("{}", a + b);
}

fn main() {
    let a = 32; // 整数类型，数据存放在栈上
    let b = 45; // 整数类型，数据存放在栈上
    cop(a, b); // copy 语义，a, b 变量的拷贝传给调用的方法
    println!("We have a: {} and b: {}", a, b); // 变量a, b 仍然可用
}
```
```Rust
fn re(v: Vec<i32>) -> Vec<i32> {
    println!("{}", v[120] + v[111]);
    v
}

fn borrow1(v: &Vec<i32>) {
    println!("{}", (*v)[10] + (*v)[12]); // 通过指针访问v中元素
}

fn borrow2(v: &Vec<i32>) {
    println!("{}", v[10] + v[11]); // 直接访问v中元素
}

fn main() {
    let mut v = Vec::new();
    for i in 1..1000 {
        v.push(i); 
    }
    v = re(v); // move 语义，但是 re 函数返回类型是 Vec<i32>，并将返回值赋值给变量 v，re 函数将所有权又返回给 main 函数，在 main 函数中可以继续使用
    println!("Still own: v: {} {}", v[0], v[1]);
    borrow1(&v); // 借用
    println!("Still own: v: {} {}", v[0], v[1]);
    borrow2(&v); // 借用
    println!("Still own: v: {} {}", v[0], v[1]);
}
```
```Rust
fn main() {
    let v = vec![4, 5, 3, 6, 7, 4, 8, 6, 4, 2, 4, 2, 4, 5, 3, 7, 7];
    for &i in &v {
        let r = count(&v, i); // 将 v 的引用传给函数 count，v 在 main 函数中生命周期没有结束，仍然可用
        println!("{} is repeated {} items", i, r); 
    }
}

fn count(v: &Vec<i32>, val: i32) -> usize {
    v.into_iter().filter(|&&x| x == val).count()
}
```