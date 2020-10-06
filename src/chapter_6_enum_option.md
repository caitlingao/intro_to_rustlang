## enum
`enum`可以嵌入任意类型的数据，包括字符串、数值、结构体或者另外一个枚举。
```Rust
enum Message {
    Quit,
    Move {x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
enum的一个例子
```rust
enum Shape {
    Rectangle {width: u32, height: u32},
    Square(u32),
    Circle(f64),
}

impl Shape {
    fn area(&self) -> f64 {
        match *self { 
            Shape::Rectangle {width, height} => (width * height) as f64,
            Shape::Square(ref s) => (s * s) as f64,
            Shape::Circle(ref r) => 3.14 * (r * r), 
        } 
    }
}

fn main() {
    let r = Shape::Rectangle {width: 10, height: 70};
    let s = Shape::Square(10);
    let c = Shape::Circle(4.5);

    let ar = r.area();
    println!("{}", ar);

    let aq = s.area();
    println!("{}", aq);

    let ac = c.area();
    println!("{}", ac);
}
```
如果需要绑定的是被匹配对象的引用，可以使用`ref`关键字。使用`ref`是因为在模式匹配的时候有可能发生变量的所有权转移，使用`ref`就是为了避免出现所有权转移。
```Rust
let ref x = 5;
let x = &5_i32;
```
这两条语句中变量`x`是同样类型`&i32`

enum另一个更加复杂的例子
```rust
# #![allow(dead_code)]
#[derive(Debug)]
enum Direction {
    Up(Point),
    Down(Point),
    Left(Point),
    Right(Point),
}

#[derive(Debug)]
enum Keys {
    UpKey(String),
    DownKey(String),
    LeftKey(String),
    RightKey(String),
}

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Direction {
    fn match_direction(&self) -> Keys {
        match *self {
            Direction::Up(_) => Keys::UpKey(String::from("Pressed w")),
            Direction::Down(_) => Keys::DownKey(String::from("Pressed s")),
            Direction::Left(_) => Keys::LeftKey(String::from("Pressed a")),
            Direction::Right(_) => Keys::RightKey(String::from("Pressed d")),
        } 
    }
}

fn main() {
    let u = Direction::Up(Point{x: 0, y: 1});
    let k = u.match_direction();
    println!("{:?}", k);
}
```
运行上面的代码结果：
```
UpKey("Pressed w")
```
对于上面的代码，需要返回 Keys 这个 enum 中 UpKey 这个字段的具体的值（Pressed w），需要为 Keys 定义方法
```rust
# #![allow(dead_code)]
# #[derive(Debug)]
# enum Direction {
#     Up(Point),
#     Down(Point),
#     Left(Point),
#     Right(Point),
# }
# 
# #[derive(Debug)]
# enum Keys {
#     UpKey(String),
#     DownKey(String),
#     LeftKey(String),
#     RightKey(String),
# }
# 
# #[derive(Debug)]
# struct Point {
#     x: i32,
#     y: i32,
# }
# 
# impl Direction {
#     fn match_direction(&self) -> Keys {
#         match *self {
#             Direction::Up(_) => Keys::UpKey(String::from("Pressed w")),
#             Direction::Down(_) => Keys::DownKey(String::from("Pressed s")),
#             Direction::Left(_) => Keys::LeftKey(String::from("Pressed a")),
#             Direction::Right(_) => Keys::RightKey(String::from("Pressed d")),
#         } 
#     }
# }
impl Keys {
    fn destruct(&self) -> &String {
        match *self {
            Keys::UpKey(ref s) => s, 
            Keys::DownKey(ref s) => s, 
            Keys::LeftKey(ref s) => s, 
            Keys::RightKey(ref s) => s, 
        } 
    }
}


fn main() {
#    let u = Direction::Up(Point{x: 0, y: 1});
#    let k = u.match_direction();
    let x = k.destruct();
    println!("{:?}", x);
}
```
运行这段代码返回结果
```
Pressed w
```
### 可选项 Option<T>
Rust 没有空值，提供了一个拥有类似概念的枚举，可以用它来标识一个值无效或缺失，这个枚举就是`Option<T>`
```Rust
enum Option<T> {
    Some(T),
    None,
}
```
当有了一个Some值时，就确定值是存在的，并且被Some所持有，如果有了一个None值时，就知道当前并不存在一个有效的值。
```rust
fn division(x: f64, y: f64) -> Option<f64> {
    if y == 0.0 {
        None 
    } else {
        Some(x / y) 
    }
}

fn main() {
    let res = division(5.0, 7.0);
    match res {
        Some(x) => println!("{:.7}", x),
        None => println!("cannot divide by 0"),
    }
}
```