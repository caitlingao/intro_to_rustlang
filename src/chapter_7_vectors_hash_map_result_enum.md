## 动态数组
创建`Vector`和创建`String`方法相似，因为`String`类型的字符串本身就是对`Vec<u8>`类型的包装。

创建一个可变的`Vector`空数组`Vec::new()`或者`vec![]`
```rust
fn main() {
    let mut v: Vec<i32> = Vec::new();

    // 向 vector 中添加元素
    v.push(1);
    v.push(2);
    v.push(3);
    v.push(4);

    // 循环遍历 vector 中元素
    for i in &v {
        println!("{}", i); 
    }
    
    // 打印 vector，vector 长度值及容量大小
    println!("{:?}, {}, {}", &v, v.len(), v.capacity());

    // 弹出末尾元素，pop() 方法返回的是 Option<T>类型
    let p = v.pop();
    println!("{:?}", p);

    // 访问vec 中元素
    let does_not_exist = &v[100]; // 返回元素的引用，越界访问会触发 panic
    let does_not_exist = v.get(100); // get 方法返回一个 Option<T>，越界访问会返回None
}
```
可以使用枚举来存储多个类型的值
```rust
#[derive(Debug)]
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Float(10.12),
        SpreadsheetCell::Text(String::from("blue")),
    ];
    println!("{:?}", &row);
}
```
## HashMap
它存储了K类型键与V类型值之间的映射关系，HashMap要求所有键类型必须相同，所有值类型也必须相同。
```rust
use std::collections::HashMap;

fn main() {
    // 定义hash map
    let mut hm = HashMap::new();

    // 向 hash map 中插入元素
    hm.insert(String::from("random"), 12);
    hm.insert(String::from("strings"), 49);

    // 移除 hash map 中指定的 key
    hm.remove(&String::from("strings"));

    // 使用 for 循环遍历 hash map
    for (k, v) in &hm {
        println!("{}: {}", k, v); 
    }

    // 使用 get 方法访问 hash map 指定的 key
    match hm.get(&String::from("random")) {
        Some(&n) => println!("{}", n),
        _ => println!("no match"),
    }
}
```
## Result enum
```Rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
T代表了Ok变体中包含的值类型，该变体中的值会在执行成功时返回；E代表了Err变体中包含的错误类型，该变体中的值会在执行失败时返回。
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.text");
    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("There was a problem opening the file: {:?}", error), 
    };
}
```
### 错误传播
当编写的函数中包含了一些可能会执行失败的调用时，除了可以在函数中处理这个错误，还可以将这个错误返回给调用者，让他们决定应该如何做进一步处理。这个过程就叫做错误传播
```Rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_userName_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.text");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e), 
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e), 
    }
}
```
上面代码可使用`?运算符`简化
```Rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_userName_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.text")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```