## 贪吃蛇游戏
### 简单介绍
小蛇每吃到一个苹果就会长一小截，使用上、下、左、右箭头可以控制小蛇的方向，小蛇每次移动一个小格。蛇不能咬到自己，一但咬到自己就会死掉，游戏重新开始。蛇撞到四周的墙会也死掉，游戏重新开始。
### 源代码
[tensor-programming/snake-tutorial](https://github.com/tensor-programming/snake-tutorial)
### 代码
#### 新建项目并添加相关依赖包
创建一个名为`snake`的项目
```
$ cargo new snake
     Created binary (application) `snake` package
```
打开`Cargo.toml`文件，添加相关依赖包
```toml
[package]
name = "snake"
version = "0.1.0"
authors = ["your name <your email@example.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.7.3"
piston_window = "0.113.0"
```
运行`cargo build`编译通过
```
$ cargo build
    Updating crates.io index
  Downloaded raw-window-handle v0.3.3
  Downloaded rayon v1.4.1
  Downloaded rusttype v0.8.3
  Downloaded bytemuck v1.4.1
  Downloaded crossbeam-channel v0.4.4
  Downloaded weezl v0.1.1
......
```
`main.rs`文件声明引入第三方包
```rust
// src/main.rs
extern crate piston_window;
extern crate rand;

fn main() {
    println!("Hello, world!");
}
```
#### 添加相关辅助方法
##### 添加draw文件
1. 在`src`目录下添加`draw.rs`文件，存放draw相关的方法
``` 
$ touch src/draw.rs
```
2. 在`src/main.rs`文件中使用`mod`关键字定义`draw`模块
```rust
// src/main.rs

#extern crate piston_window;
#extern crate rand;

mod draw;

#fn main() {
#    println!("Hello, world!");
#}
```
3. `src/draw.rs`文件中引入需要的模块
```Rust
// src/draw.rs

use piston_window::types::Color;
use piston_window::{rectangle, Context, G2d};
```
4. `src/draw.rs` 文件中定义常量，用来表示小块的大小。
```rust,ignore,does_not_compile
// src/draw.rs

# use piston_window::types::Color;
# use piston_window::{rectangle, Context, G2d};
#
const BLOCK_SIZE: f64 = 25.0;
```
5. 接下来在`src/draw.rs`文件中添加第一个函数`to_coord`，在游戏坐标值i32类型在乘以BLOCK_SIZE后返回f64类型。同时这个函数使用关键字`pub`，这样它就可以在游戏项目内被调用。
```rust,ignore,does_not_compile
// src/draw.rs

# use piston_window::types::Color;
# use piston_window::{rectangle, Context, G2d};
# 
# const BLOCK_SIZE: f64 = 25.0;
#
pub fn to_coord(game_coord: i32) -> f64 {
    (game_coord as f64) * BLOCK_SIZE
}
```
6. 添加第一个主要的帮助函数`draw_block`，绘制小方块
```rust,ignore,does_not_compile
// src/draw.rs

# use piston_window::types::Color;
# use piston_window::{rectangle, Context, G2d};
# 
# const BLOCK_SIZE: f64 = 25.0;
# 
# pub fn to_coord(game_coord: i32) -> f64 {
#     (game_coord as f64) * BLOCK_SIZE
# }
#
pub fn draw_block(color: Color, x: i32, y: i32, con: &Context, g: &mut G2d) {
    let gui_x = to_coord(x);
    let gui_y = to_coord(y);

    rectangle(
        color,
        [gui_x, gui_y, BLOCK_SIZE, BLOCK_SIZE],
        con.transform,
        g,
    );
}
```
7. 添加第二个函数`draw_rectangle`用来绘制长方形
```rust,ignore,does_not_compile
// src/draw.rs

# use piston_window::types::Color;
# use piston_window::{rectangle, Context, G2d};
# 
# const BLOCK_SIZE: f64 = 25.0;
# 
# pub fn to_coord(game_coord: i32) -> f64 {
#     (game_coord as f64) * BLOCK_SIZE
# }
# 
# pub fn draw_block(color: Color, x: i32, y: i32, con: &Context, g: &mut G2d) {
#     let gui_x = to_coord(x);
#     let gui_y = to_coord(y);
# 
#     rectangle(
#         color,
#         [gui_x, gui_y, BLOCK_SIZE, BLOCK_SIZE],
#         con.transform,
#         g,
#     );
# }
# 
pub fn draw_rectangle(
    color: Color,
    x: i32,
    y: i32,
    width: i32,
    height: i32,
    con: &Context,
    g: &mut G2d,
) {
    let x = to_coord(x);
    let y = to_coord(y);

    rectangle(
        color,
        [
            x,
            y,
            BLOCK_SIZE * (width as f64),
            BLOCK_SIZE * (height as f64),
        ],
        con.transform,
        g,
    );
}
```
##### 添加snake文件
1. src目录下添加snake.rs文件
```
$ touch src/snake.rs
```
2. `src/main.rs` 文件中添加`snake`模块
```rust
// src/main.rs

# extern crate piston_window;
# extern crate rand;

# mod draw;
mod snake;

# fn main() {
#     println!("Hello, world!");
# }
```
3. `src/snake.rs`文件中引入需要的模块
```rust,ignore,does_not_compile
// src/snake.rs

use piston_window::types::Color;
use piston_window::{Context, G2d};
use std::collections::LinkedList;

use crate::draw::draw_block;
```
4. `src/snake.rs` 文件中添加snake_color 常量
```rust,ignore,does_not_compile
// src/snake.rs
#
# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
#
# use crate::draw::draw_block;
#
const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
```
5. `src/snake.rs`文件中定义enum来表示snake移动的方向，并为enum实现方法，在这个方法中去匹配蛇移动的方法，当它向上移动时，按向下方向键使它向下移动
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
 pub enum Direction {
    Up,
    Down,
    Left,
    Right,
}

impl Direction {
    pub fn opposite(&self) -> Direction {
        match *self {
            Direction::Up => Direction::Down,
            Direction::Down => Direction::Up,
            Direction::Left => Direction::Right,
            Direction::Right => Direction::Left,
        }
    }
}
```
6. 使用 struct 定义一下 Block 类型
```rust,ignore,does_not_compile
//src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
struct Block {
    x: i32,
    y: i32,
}
```
7. 使用 struct 定义 Snake 类型，Snake 中元素包括 direction: 蛇当前运动的方向，body: 蛇的身体，是一个包含Blocks的 LinkList，tail: 蛇的尾巴，它是一个Option块，当蛇吃到苹果时，尾巴才会有值，因为当吃到苹果时尾巴不会被删掉。
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
struct Snake {
    direction: Direction,
    body: LinkedList<Block>,
    tail: Option<Block>,
}
```
8. 为 Snake 类型实现方法来初始化一条蛇，每次重新开始游戏时，会初始化一条蛇，初始时蛇有3个水平排列的小方块，第1小方块坐标分别是x, y，第2个小方块的坐标是x+1, y，第2个小方块的坐标是x+2, y，初始时蛇移动的方向是向右
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
    pub fn new(x: i32, y: i32) -> Self {
        let mut body: LinkedList<Block> = LinkedList::new();
        body.push_back(Block { x: x + 2, y });
        body.push_back(Block { x: x + 1, y });
        body.push_back(Block { x: x, y });

        Self {
            direction: Direction::Right,
            body,
            tail: None,
        }
    }
}
```
9. 定义draw方法，并调用 `draw.rs` 文件中的 `draw_block`方法绘制出蛇在屏幕上显示的样子
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
    pub fn draw(&self, con: &Context, g: &mut G2d) {
        for block in &self.body {
            draw_block(SNAKE_COLOR, block.x, block.y, con, g);
        }
    }
}
```
10. 定义`head_position`方法，返回头部坐标的x,y值
```rust,ignore,does_not_compile
// src/snake.rs

# se piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
    pub fn head_position(&self) -> (i32, i32) {
        let head_block = &self.body.front().unwrap();
        (head_block.x, head_block.y)
    }
}
```
11. 定义 `move_forward` 方法，记录蛇向前移动
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
    pub fn move_forward(&mut self, dir: Option<Direction>) {
        match dir {
            Some(d) => self.direction = d,
            None => (),
        }

        let (last_x, last_y): (i32, i32) = self.head_position();
        let new_block = match self.direction {
            Direction::Up => Block {
                x: last_x,
                y: last_y - 1,
            },
            Direction::Down => Block {
                x: last_x,
                y: last_y + 1,
            },
            Direction::Left => Block {
                x: last_x - 1,
                y: last_y,
            },
            Direction::Right => Block {
                x: last_x + 1,
                y: last_y,
            },
        };

        self.body.push_front(new_block);
        let removed_block = self.body.pop_back();
        self.tail = removed_block;
    }
}
```
12. 定义`head_direction`方法，获取蛇移动的方向
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
    pub fn head_direction(&self) -> Direction {
        self.direction
    }
}
```
添加这个方法后，会报错
```
cannot move out of `self.direction` which is behind a shared reference
```
为解决这个问题需要对 `Direction enum` 实现 `Copy trait`和`Clone trait`
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#[derive(Copy, Clone)]
 pub enum Direction {
    Up,
    Down,
    Left,
    Right,
}

# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
    pub fn head_direction(&self) -> Direction {
        self.direction
    }
}
```
13. 定义`next_head`方法
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
# #[derive(Copy, Clone)]
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
#     pub fn head_direction(&self) -> Direction {
#         self.direction
#     }
# 
    pub fn next_head(&self, dir: Option<Direction>) -> (i32, i32) {
        let (head_x, head_y): (i32, i32) = self.head_position();

        let mut moving_dir = self.direction;
        match dir {
            Some(d) => moving_dir = d,
            None => (),
        }

        match moving_dir {
            Direction::Up => (head_x, head_y - 1),
            Direction::Down => (head_x, head_y + 1),
            Direction::Left => (head_x - 1, head_y),
            Direction::Right => (head_x + 1, head_y),
        }
    }
}
```
14. 定义 `resore_tail` 方法，当蛇吃到苹果时，这个方法就会被调用，蛇长度增加1
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
# #[derive(Copy, Clone)]
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
#[derive(Clone)]
struct Block {
    x: i32,
    y: i32,
}
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }

impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
#     pub fn head_direction(&self) -> Direction {
#         self.direction
#     }
# 
#     pub fn next_head(&self, dir: Option<Direction>) -> (i32, i32) {
#         let (head_x, head_y): (i32, i32) = self.head_position();
# 
#         let mut moving_dir = self.direction;
#         match dir {
#             Some(d) => moving_dir = d,
#             None => (),
#         }
# 
#         match moving_dir {
#             Direction::Up => (head_x, head_y - 1),
#             Direction::Down => (head_x, head_y + 1),
#             Direction::Left => (head_x - 1, head_y),
#             Direction::Right => (head_x + 1, head_y),
#         }
#     }
# 
    pub fn restore_tail(&mut self) {
        let block = self.tail.clone().unwrap();
        self.body.push_back(block);
    }
}
```
15. 定义`overlap_tail`方法，判断蛇是否咬到自己
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
# #[derive(Copy, Clone)]
#  pub enum Direction {
#     Up,
#     Down,
#     Left,
#     Right,
# }
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# #[derive(Clone)]
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
#     pub fn head_direction(&self) -> Direction {
#         self.direction
#     }
# 
#     pub fn next_head(&self, dir: Option<Direction>) -> (i32, i32) {
#         let (head_x, head_y): (i32, i32) = self.head_position();
# 
#         let mut moving_dir = self.direction;
#         match dir {
#             Some(d) => moving_dir = d,
#             None => (),
#         }
# 
#         match moving_dir {
#             Direction::Up => (head_x, head_y - 1),
#             Direction::Down => (head_x, head_y + 1),
#             Direction::Left => (head_x - 1, head_y),
#             Direction::Right => (head_x + 1, head_y),
#         }
#     }
# 
#     pub fn restore_tail(&mut self) {
#         let block = self.tail.clone().unwrap();
#         self.body.push_back(block);
#     }
# 
    pub fn overlap_tail(&self, x: i32, y: i32) -> bool {
        let mut ch = 0;
        for block in &self.body {
            if block.x == x && block.y == y {
                return true;
            }

            ch += 1;
            if ch == self.body.len() - 1 {
                break;
            }
        }
        false
    }
}
```
##### 添加game.rs
1. src目录下添加`game.rs`文件
```
$ touch game.rs
```
2. `src/main.rs` 文件中添加`game`模块
```rust
// src/main.rs

# mod draw;
mod game;
# mod snake;
# 
# fn main() {
#     println!("Hello, world!");
# }
```
3. `src/game.rs`文件中引入相关方法
```rust,ignore,does_not_compile
// src/game.rs

use piston_window::types::Color;
use piston_window::*;

use rand::{thread_rng, Rng};

use crate::draw::{draw_block, draw_rectangle};
use crate::snake::{Direction, Snake};
```
4. 添加食物颜色、画面颜色、游戏结束颜色常量
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
```
5. 添加运动周期、重新开始时间常量
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
const MOVING_PERIOD: f64 = 0.1;
const RESTART_TIME: f64 = 1.0;
```
6. 使用 struct 定义 Game 类型，包含 蛇、食物是否存在，食物的坐标、游戏画布宽高、游戏是否结束、等待时间等字段
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
# const MOVING_PERIOD: f64 = 0.1;
# const RESTART_TIME: f64 = 1.0;
# 
pub struct Game {
    snake: Snake,
    food_exists: bool,
    food_x: i32,
    food_y: i32,
    width: i32,
    height: i32,
    game_over: bool,
    waiting_time: f64,
}
```
7. 为 Game 实现 new 方法
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
# const MOVING_PERIOD: f64 = 0.1;
# const RESTART_TIME: f64 = 1.0;
# 
# pub struct Game {
#     snake: Snake,
#     food_exists: bool,
#     food_x: i32,
#     food_y: i32,
#     width: i32,
#     height: i32,
#     game_over: bool,
#     waiting_time: f64,
# }
# 
impl Game {
    pub fn new(width: i32, height: i32) -> Self {
        Self {
            snake: Snake::new(2, 2),
            food_exists: true,
            food_x: 6,
            food_y: 4,
            width,
            height,
            game_over: false,
            waiting_time: 0.0,
        }
    }
}
```
8. 添加`key_pressed`方法，监听当前键盘按下的方向键
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
# const MOVING_PERIOD: f64 = 0.1;
# const RESTART_TIME: f64 = 1.0;
# 
# pub struct Game {
#     snake: Snake,
#     food_exists: bool,
#     food_x: i32,
#     food_y: i32,
#     width: i32,
#     height: i32,
#     game_over: bool,
#     waiting_time: f64,
# }
# 
impl Game {
#    pub fn new(width: i32, height: i32) -> Self {
#        Self {
#            snake: Snake::new(2, 2),
#            food_exists: true,
#            food_x: 6,
#            food_y: 4,
#            width,
#            height,
#            game_over: false,
#            waiting_time: 0.0,
#        }
#    }
#
    pub fn key_pressed(&mut self, key: Key) {
        if self.game_over {
            return;
        }

        let dir = match key {
            Key::Up => Some(Direction::Up),
            Key::Down => Some(Direction::Down),
            Key::Left => Some(Direction::Left),
            Key::Right => Some(Direction::Right),
            _ => Some(self.snake.head_direction()),
        };

        // 如果蛇正在移动，按下与蛇移动方向相反的键，直接return。例如：蛇向上移动，接下向下方向键
        if dir.unwrap() == self.snake.head_direction().opposite() {
            return;
        }
        self.update_snake(dir);
    }

    fn check_eating(&mut self) {
        let (head_x, head_y): (i32, i32) = self.snake.head_position();

        if self.food_exists && self.food_x == head_x && self.food_y == head_y {
            self.food_exists = false;
            self.snake.restore_tail();
        }
    }

    fn check_if_snake_alive(&self, dir: Option<Direction>) -> bool {
        let (next_x, next_y) = self.snake.next_head(dir);

        if self.snake.overlap_tail(next_x, next_y) {
            return false;
        }

        next_x > 0 && next_y > 0 && next_x < self.width - 1 && next_y < self.height - 1
    }

    fn update_snake(&mut self, dir: Option<Direction>) {
        if self.check_if_snake_alive(dir) {
            self.snake.move_forward(dir);
            self.check_eating();
        } else {
            self.game_over = true;
        }
        self.waiting_time = 0.0;
    }
}
```
在这里会提示错误，原因是 `snake::Direction` 没有实现`PartialEq trait`
```
binary operation `==` cannot be applied to type `snake::Direction`
```
通过 `dreiver` 为 `snake::Direction` 添加 `PartialEq`
```rust,ignore,does_not_compile
// src/snake.rs

# use piston_window::types::Color;
# use piston_window::{Context, G2d};
# use std::collections::LinkedList;
# 
# use crate::draw::draw_block;
# 
# const SNAKE_COLOR: Color = [0.00, 0.80, 0.00, 1.0];
# 
#[derive(Copy, Clone, PartialEq)]
pub enum Direction {
    Up,
    Down,
    Left,
    Right,
}
# 
# impl Direction {
#     pub fn opposite(&self) -> Direction {
#         match *self {
#             Direction::Up => Direction::Down,
#             Direction::Down => Direction::Up,
#             Direction::Left => Direction::Right,
#             Direction::Right => Direction::Left,
#         }
#     }
# }
# 
# #[derive(Clone)]
# struct Block {
#     x: i32,
#     y: i32,
# }
# 
# pub struct Snake {
#     direction: Direction,
#     body: LinkedList<Block>,
#     tail: Option<Block>,
# }
# 
# impl Snake {
#     pub fn new(x: i32, y: i32) -> Self {
#         let mut body: LinkedList<Block> = LinkedList::new();
#         body.push_back(Block { x: x + 2, y });
#         body.push_back(Block { x: x + 1, y });
#         body.push_back(Block { x: x, y });
# 
#         Self {
#             direction: Direction::Right,
#             body,
#             tail: None,
#         }
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         for block in &self.body {
#             draw_block(SNAKE_COLOR, block.x, block.y, con, g);
#         }
#     }
# 
#     pub fn head_position(&self) -> (i32, i32) {
#         let head_block = &self.body.front().unwrap();
#         (head_block.x, head_block.y)
#     }
# 
#     pub fn move_forward(&mut self, dir: Option<Direction>) {
#         match dir {
#             Some(d) => self.direction = d,
#             None => (),
#         }
# 
#         let (last_x, last_y): (i32, i32) = self.head_position();
#         let new_block = match self.direction {
#             Direction::Up => Block {
#                 x: last_x,
#                 y: last_y - 1,
#             },
#             Direction::Down => Block {
#                 x: last_x,
#                 y: last_y + 1,
#             },
#             Direction::Left => Block {
#                 x: last_x - 1,
#                 y: last_y,
#             },
#             Direction::Right => Block {
#                 x: last_x + 1,
#                 y: last_y,
#             },
#         };
# 
#         self.body.push_front(new_block);
#         let removed_block = self.body.pop_back();
#         self.tail = removed_block;
#     }
# 
#     pub fn head_direction(&self) -> Direction {
#         self.direction
#     }
# 
#     pub fn next_head(&self, dir: Option<Direction>) -> (i32, i32) {
#         let (head_x, head_y): (i32, i32) = self.head_position();
# 
#         let mut moving_dir = self.direction;
#         match dir {
#             Some(d) => moving_dir = d,
#             None => (),
#         }
# 
#         match moving_dir {
#             Direction::Up => (head_x, head_y - 1),
#             Direction::Down => (head_x, head_y + 1),
#             Direction::Left => (head_x - 1, head_y),
#             Direction::Right => (head_x + 1, head_y),
#         }
#     }
# 
#     pub fn restore_tail(&mut self) {
#         let block = self.tail.clone().unwrap();
#         self.body.push_back(block);
#     }
# 
#     pub fn overlap_tail(&self, x: i32, y: i32) -> bool {
#         let mut ch = 0;
#         for block in &self.body {
#             if block.x == x && block.y == y {
#                 return true;
#             }
# 
#             ch += 1;
#             if ch == self.body.len() - 1 {
#                 break;
#             }
#         }
#         false
#     }
# }
```
9. 添加`draw`方法，绘制蛇、食物、游戏画布、游戏结束时样子
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
# const MOVING_PERIOD: f64 = 0.1;
# const RESTART_TIME: f64 = 1.0;
# 
# pub struct Game {
#     snake: Snake,
#     food_exists: bool,
#     food_x: i32,
#     food_y: i32,
#     width: i32,
#     height: i32,
#     game_over: bool,
#     waiting_time: f64,
# }
# 
impl Game {
#     pub fn new(width: i32, height: i32) -> Self {
#         Self {
#             snake: Snake::new(2, 2),
#             food_exists: true,
#             food_x: 6,
#             food_y: 4,
#             width,
#             height,
#             game_over: false,
#             waiting_time: 0.0,
#         }
#     }
# 
#     pub fn key_pressed(&mut self, key: Key) {
#         if self.game_over {
#             return;
#         }
# 
#         let dir = match key {
#             Key::Up => Some(Direction::Up),
#             Key::Down => Some(Direction::Down),
#             Key::Left => Some(Direction::Left),
#             Key::Right => Some(Direction::Right),
#             _ => Some(self.snake.head_direction()),
#         };
# 
#         // 如果蛇正在移动，按下与蛇移动方向相反的键，直接return。例如：蛇向上移动，接下向下方向键
#         if dir.unwrap() == self.snake.head_direction().opposite() {
#             return;
#         }
# 
#         self.update_snake(dir);
#     }
# 
    pub fn draw(&self, con: &Context, g: &mut G2d) {
        self.snake.draw(con, g);

        if self.food_exists {
            draw_block(FOOD_COLOR, self.food_x, self.food_y, con, g);
        }

        draw_rectangle(BORDER_COLOR, 0, 0, self.width, 1, con, g);
        draw_rectangle(BORDER_COLOR, 0, self.height - 1, self.width, 1, con, g);
        draw_rectangle(BORDER_COLOR, 0, 0, 1, self.height, con, g);
        draw_rectangle(BORDER_COLOR, self.width - 1, 0, 1, self.height, con, g);

        if self.game_over {
            draw_rectangle(GAME_OVER_COLOR, 0, 0, self.width, self.height, con, g);
        }
    }
# 
#     fn check_eating(&mut self) {
#         let (head_x, head_y): (i32, i32) = self.snake.head_position();
# 
#         if self.food_exists && self.food_x == head_x && self.food_y == head_y {
#             self.food_exists = false;
#             self.snake.restore_tail();
#         }
#     }
# 
#     fn check_if_snake_alive(&self, dir: Option<Direction>) -> bool {
#         let (next_x, next_y) = self.snake.next_head(dir);
# 
#         if self.snake.overlap_tail(next_x, next_y) {
#             return false;
#         }
# 
#         next_x > 0 && next_y > 0 && next_x < self.width - 1 && next_y < self.height - 1
#     }
# 
#     fn update_snake(&mut self, dir: Option<Direction>) {
#         if self.check_if_snake_alive(dir) {
#             self.snake.move_forward(dir);
#             self.check_eating();
#         } else {
#             self.game_over = true;
#         }
#         self.waiting_time = 0.0;
#     }
}
```
10. 添加 `update` 方法
```rust,ignore,does_not_compile
// src/game.rs

# use piston_window::types::Color;
# use piston_window::*;
# 
# use rand::{thread_rng, Rng};
# 
# use crate::draw::{draw_block, draw_rectangle};
# use crate::snake::{Direction, Snake};
# 
# const FOOD_COLOR: Color = [0.80, 0.00, 0.00, 1.0];
# const BORDER_COLOR: Color = [0.00, 0.00, 0.00, 1.0];
# const GAME_OVER_COLOR: Color = [0.90, 0.00, 0.00, 0.5];
# 
# const MOVING_PERIOD: f64 = 0.1;
# const RESTART_TIME: f64 = 1.0;
# 
# pub struct Game {
#     snake: Snake,
#     food_exists: bool,
#     food_x: i32,
#     food_y: i32,
#     width: i32,
#     height: i32,
#     game_over: bool,
#     waiting_time: f64,
# }
# 
impl Game {
#     pub fn new(width: i32, height: i32) -> Self {
#         Self {
#             snake: Snake::new(2, 2),
#             food_exists: true,
#             food_x: 6,
#             food_y: 4,
#             width,
#             height,
#             game_over: false,
#             waiting_time: 0.0,
#         }
#     }
# 
#     pub fn key_pressed(&mut self, key: Key) {
#         if self.game_over {
#             return;
#         }
# 
#         let dir = match key {
#             Key::Up => Some(Direction::Up),
#             Key::Down => Some(Direction::Down),
#             Key::Left => Some(Direction::Left),
#             Key::Right => Some(Direction::Right),
#             _ => Some(self.snake.head_direction()),
#         };
# 
#         // 如果蛇正在移动，按下与蛇移动方向相反的键，直接return。例如：蛇向上移动，接下向下方向键
#         if dir.unwrap() == self.snake.head_direction().opposite() {
#             return;
#         }
# 
#         self.update_snake(dir);
#     }
# 
#     pub fn draw(&self, con: &Context, g: &mut G2d) {
#         self.snake.draw(con, g);
# 
#         if self.food_exists {
#             draw_block(FOOD_COLOR, self.food_x, self.food_y, con, g);
#         }
# 
#         draw_rectangle(BORDER_COLOR, 0, 0, self.width, 1, con, g);
#         draw_rectangle(BORDER_COLOR, 0, self.height - 1, self.width, 1, con, g);
#         draw_rectangle(BORDER_COLOR, 0, 0, 1, self.height, con, g);
#         draw_rectangle(BORDER_COLOR, self.width - 1, 0, 1, self.height, con, g);
# 
#         if self.game_over {
#             draw_rectangle(GAME_OVER_COLOR, 0, 0, self.width, self.height, con, g);
#         }
#     }
# 
    pub fn update(&mut self, delta_time: f64) {
        self.waiting_time += delta_time;

        if self.game_over {
            if self.waiting_time > RESTART_TIME {
                self.restart();
            }
            return;
        }

        if !self.food_exists {
            self.add_food();
        }

        if self.waiting_time > MOVING_PERIOD {
            self.update_snake(None);
        }
    }
# 
#     fn check_eating(&mut self) {
#         let (head_x, head_y): (i32, i32) = self.snake.head_position();
# 
#         if self.food_exists && self.food_x == head_x && self.food_y == head_y {
#             self.food_exists = false;
#             self.snake.restore_tail();
#         }
#     }
# 
#     fn check_if_snake_alive(&self, dir: Option<Direction>) -> bool {
#         let (next_x, next_y) = self.snake.next_head(dir);
# 
#         if self.snake.overlap_tail(next_x, next_y) {
#             return false;
#         }
# 
#         next_x > 0 && next_y > 0 && next_x < self.width - 1 && next_y < self.height - 1
#     }
# 
#     fn update_snake(&mut self, dir: Option<Direction>) {
#         if self.check_if_snake_alive(dir) {
#             self.snake.move_forward(dir);
#             self.check_eating();
#         } else {
#             self.game_over = true;
#         }
#         self.waiting_time = 0.0;
#     }

    fn add_food(&mut self) {
        let mut rng = thread_rng();

        let mut new_x = rng.gen_range(1, self.width - 1);
        let mut new_y = rng.gen_range(1, self.height - 1);
        while self.snake.overlap_tail(new_x, new_y) {
            new_x = rng.gen_range(1, self.width - 1);
            new_y = rng.gen_range(1, self.height - 1);
        }

        self.food_x = new_x;
        self.food_y = new_y;
        self.food_exists = true;
    }

    fn restart(&mut self) {
        self.snake = Snake::new(2, 2);
        self.waiting_time = 0.0;
        self.food_exists = true;
        self.food_x = 6;
        self.food_y = 4;
        self.game_over = false;
    }
}
```
##### 完善main.rs文件
```
mod draw;
mod game;
mod snake;

use piston_window::types::Color;
use piston_window::*;

use crate::draw::to_coord_u32;
use crate::game::Game;

const BACK_COLOR: Color = [0.5, 0.5, 0.5, 1.0];

fn main() {
    let (width, height) = (30, 30);

    let mut window: PistonWindow =
        WindowSettings::new("Snake", [to_coord_u32(width), to_coord_u32(height)])
            .exit_on_esc(true)
            .build()
            .unwrap();
    let mut game = Game::new(width, height);

    while let Some(event) = window.next() {
        if let Some(Button::Keyboard(key)) = event.press_args() {
            game.key_pressed(key);
        }
        window.draw_2d(&event, |c, g, _| {
            clear(BACK_COLOR, g);
            game.draw(&c, g);
        });

        event.update(|arg| {
            game.update(arg.dt);
        });
    }
}
```
`src/draw.rs`文件中增加 `to_coord_us`方法
```rust,ignore,does_not_compile
// src/draw.rs

# use piston_window::types::Color;
# use piston_window::{rectangle, Context, G2d};
# 
# const BLOCK_SIZE: f64 = 25.0;
# 
# pub fn to_coord(game_coord: i32) -> f64 {
#     (game_coord as f64) * BLOCK_SIZE
# }
# 
pub fn to_coord_u32(game_coord: i32) -> u32 {
    to_coord(game_coord) as u32
}
# 
# pub fn draw_block(color: Color, x: i32, y: i32, con: &Context, g: &mut G2d) {
#     let gui_x = to_coord(x);
#     let gui_y = to_coord(y);
# 
#     rectangle(
#         color,
#         [gui_x, gui_y, BLOCK_SIZE, BLOCK_SIZE],
#         con.transform,
#         g,
#     );
# }
# 
# pub fn draw_rectangle(
#     color: Color,
#     x: i32,
#     y: i32,
#     width: i32,
#     height: i32,
#     con: &Context,
#     g: &mut G2d,
# ) {
#     let x = to_coord(x);
#     let y = to_coord(y);
# 
#     rectangle(
#         color,
#         [
#             x,
#             y,
#             BLOCK_SIZE * (width as f64),
#             BLOCK_SIZE * (height as f64),
#         ],
#         con.transform,
#         g,
#     );
# }
```
