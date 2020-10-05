## 使用结构体来组织相关联的数据
### struct
结构体与元组类似， 可以把多个类型组合到一起作为新的类型。结构体的每个元素都有自己的名字并且要显示指定类型。使用`struct`关键字定义结构体，结构体名称必须遵循**驼峰式**命名规则。
```rust
// 定义结构体 Object
struct Object {
    width: u32,
    height: u32,
}
fn main() {
    // 创建结构体实例
    let obj = Object {
        width: 35,
        height: 55, 
    };
    println!("width is: {}, height is: {}", obj.width, obj.height);
}
```
结构体可以分为：
* 具名结构体，即结构体里面的字段有名称有类型
```rust, no_run
struct People {
    name: String,
    gender: u32,
}
```
* 元组结构体，结构体内的字段没有名称，只有类型
```rust,no_run
struct Color(i32, i32, i32);
```
* 单元结构体，结构体内没有字段
```rust,no_run
struct Empty;
```
### 创建函数
函数是用关键字`fn`开头，由一个名称组成，包含了一段可执行的代码。可以有一系列的输入参数，还有一个返回类型，函数返回可以使用`return`语句，也可以使用表达式，Rust默认函数内最后一个表达式的值作为函数的返回值。
```rust
# struct Object {
#     width: u32,
#     height: u32,
# }

// 定义函数 area，计算结构体 width * height 的值
fn area(obj: &Object) -> u32 {
    obj.width * obj.height
}

fn main() {
    // 创建结构体实例
    let obj = Object {
        width: 35,
        height: 55, 
    };
    println!("{}*{} with area: {}", obj.width, obj.height, area(&obj));
}
```

### 定义方法
方法与函数类似，与函数不同点是方法被定义在结构体（或者枚举类型、trait对象）的上下文中，并且它们的第一个参数永远是`self`，用于指代调用该方法的结构体实例。
```rust
# struct Object {
#     width: u32,
#     height: u32,
# }

impl Object {
    // 定义方法
    fn area(&self) -> u32 {
        self.width * self.height
    }
    fn show(&self) {
        println!("{}*{} with area: {}", self.width, self.height, self.area());
    }
}

fn main() {
    // 创建结构体实例
    let obj = Object {
        width: 35,
        height: 55, 
    };

    obj.show();
}
```
### 关联函数
`impl`允许不用接收`self`作为参数的函数，这些函数与结构体相互关联，所以它们也被称为关联函数（associated function)。它们被称为关联函数主要是因为它们不会作用于某个具体的结构体实例。关联函数常常被用作构造器来返回一个结构体的新实例。
```rust
# struct Object {
#     width: u32,
#     height: u32,
# }

impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }

    // 定义关联函数
    fn new(width: u32, height: u32) -> Self {
        Self {
            width, // key 与 value 字段名相同时，可简写
            height, 
        }    
    }
}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
    obj.show();
}
```
### 多个 impl 块
每个结构体可以拥有多个 impl 块，可以将方法和关联函数放置在不同的 impl 块中，上面代码优化如下
```rust
 struct Object {
     width: u32,
     height: u32,
 }

// Methods
impl Object {
    // 定义方法
    fn area(&self) -> u32 {
        self.width * self.height
    }
    fn show(&self) {
        println!("{}*{} with area: {}", self.width, self.height, self.area());
    }
}

// Related functions
impl Object {
    // 定义关联函数
    fn new(width: u32, height: u32) -> Self {
        Self {
            width, // key 与 value 字段名相同时，可简写
            height, 
        }    
    }
}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
    obj.show();
}
```
### 通过派生 trait 增加实用功能
如果尝试打印 obj 实例，使用 `println!`宏来打印
```rust
# struct Object {
#     width: u32,
#     height: u32,
# }
#
#// Methods
#impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }
#}
#
#// Related functions
#impl Object {
#    // 定义关联函数
#    fn new(width: u32, height: u32) -> Self {
#        Self {
#            width, // key 与 value 字段名相同时，可简写
#            height, 
#        }    
#    }
#}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
#    obj.show();
    println!("{:?}", obj);
}
```
执行这段代码会提示这样的错误信息，提示 `Object` 没有实现 `std::fmt::Debug` 这个 trait
```
Compiling playground v0.0.1 (/playground)
error[E0277]: `Object` doesn't implement `std::fmt::Debug`
  --> src/main.rs:32:22
   |
32 |     println!("{:?}", obj);
   |                      ^^^ `Object` cannot be formatted using `{:?}`
   |
   = help: the trait `std::fmt::Debug` is not implemented for `Object`
   = note: add `#[derive(Debug)]` or manually implement `std::fmt::Debug`
   = note: required by `std::fmt::Debug::fmt`
   = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: could not compile `playground`.

To learn more, run the command again with --verbose.
```
要解决这个问题，需要添加注解来派生 `Debug trait`
```rust
#[derive(Debug)] // 添加注解来派生 Debug trait
struct Object {
 width: u32,
 height: u32,
}
 
#// Methods
#impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }
#}
#
#// Related functions
#impl Object {
#    // 定义关联函数
#    fn new(width: u32, height: u32) -> Self {
#        Self {
#            width, // key 与 value 字段名相同时，可简写
#            height, 
#        }    
#    }
#}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
#    obj.show();
    println!("{:?}", obj);
}
```
Rust 提供了许多可以通过`derive`注解来派生的`trait`，它们可以为自定义的类型增加许多有用的功能。
### 扩展方法
尝试使用 `Display`的格式化方法打印 `Object`的实例
```rust
struct Object {
 width: u32,
 height: u32,
}
 
#// Methods
#impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }
#}
#
#// Related functions
#impl Object {
#    // 定义关联函数
#    fn new(width: u32, height: u32) -> Self {
#        Self {
#            width, // key 与 value 字段名相同时，可简写
#            height, 
#        }    
#    }
#}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
#    obj.show();
    println!("{}", obj);
}
```
执行上面的代码会提示下面的错误
```
Compiling playground v0.0.1 (/playground)
error[E0277]: `Object` doesn't implement `std::fmt::Display`
  --> src/main.rs:32:20
   |
32 |     println!("{}", obj);
   |                    ^^^ `Object` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Object`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`
   = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
error: could not compile `playground`.

To learn more, run the command again with --verbose.
```
增加注解派生 Display trait 是否可行呢
```rust
#[derive(Display)] // 添加注解来派生 Display trait
struct Object {
 width: u32,
 height: u32,
}
 
#// Methods
#impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }
#}
#
#// Related functions
#impl Object {
#    // 定义关联函数
#    fn new(width: u32, height: u32) -> Self {
#        Self {
#            width, // key 与 value 字段名相同时，可简写
#            height, 
#        }    
#    }
#}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
#    obj.show();
    println!("{}", obj);
}
```
依然不行，添加注解派生 Display trait 会提示下面的错误
```
Compiling playground v0.0.1 (/playground)
error: cannot find derive macro `Display` in this scope
 --> src/main.rs:1:10
  |
1 | #[derive(Display)] // 添加注解来派生 Debug trait
  |          ^^^^^^^

error[E0277]: `Object` doesn't implement `std::fmt::Display`
  --> src/main.rs:33:20
   |
33 |     println!("{}", obj);
   |                    ^^^ `Object` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Object`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`
   = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to 2 previous errors

For more information about this error, try `rustc --explain E0277`.
error: could not compile `playground`.

To learn more, run the command again with --verbose.
```
这时候需要通过自定义行为来实现这个trait
```rust
use std::fmt;

struct Object {
 width: u32,
 height: u32,
}
 
#// Methods
#impl Object {
#    // 定义方法
#    fn area(&self) -> u32 {
#        self.width * self.height
#    }
#    fn show(&self) {
#        println!("{}*{} with area: {}", self.width, self.height, self.area());
#    }
#}
#
#// Related functions
#impl Object {
#    // 定义关联函数
#    fn new(width: u32, height: u32) -> Self {
#        Self {
#            width, // key 与 value 字段名相同时，可简写
#            height, 
#        }    
#    }
#}

// 为 Object 实现 Display trait
impl fmt::Display for Object {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {} and area: {})", self.width, self.height, self.area()) 
    }
}

fn main() {
    // 创建结构体实例
    let obj = Object::new(35, 55);
#    obj.show();
    println!("{}", obj);
}
```
