# Gossamer: a Rust-flavoured language with real goroutines and pause-free memory

富有表现力且清晰的语法
前向管道（|>
）、默认不可变，
以及做事只有一种显而易见的方式。数据自上而下流动，
正如你书写的那样——而不是由内向外地层层嵌套。

一门带有 Rust 风味的系统语言，拥有真正的 goroutine 和自动、无停顿的内存管理——既可以像脚本一样运行，也可以作为单一的二进制文件交付。

开始使用 → 体验导览 GitHub
开源 · Apache-2.0 · v0.19.1（预 1.0 版）

前向管道（|>
）、默认不可变，
以及做事只有一种显而易见的方式。数据自上而下流动，
正如你书写的那样——而不是由内向外地层层嵌套。

确定性引用计数加上 arena { }
区域会在内存用完的那一刻立即回收它。没有借用
检查器，没有生命周期，没有会暂停整个世界的垃圾回收器。

go
以及在 M:N 调度器上的类型化通道。
阻塞调用会挂起 goroutine，而不是线程。没有
async
，没有 await
，没有函数
着色问题。

一个字节码虚拟机和一个 REPL 用于快速迭代；交付时通过 LLVM 生成单一、无依赖的原生二进制文件。同一种语言，分层任你选择。

Result
/ Option
/ ?
、
穷尽式 match
、特征（trait）、泛型，并且没有
null
。如果你了解 Rust、Go 或 F#，那么你
已经能读懂它了。

一个广泛的标准库——HTTP、JSON、加密、SQL、压缩等等——以及在需要时安全地降级到 Rust 的清晰路径。

在任意示例上点击 Run，它就会直接在你的浏览器中运行，无需安装。Gossamer 的虚拟机被编译为 WebAssembly。

fn double(x: i64) -> i64 { x * 2 }
fn add(a: i64, b: i64) -> i64 { a + b }
fn clamp(lo: i64, hi: i64, x: i64) -> i64 {
if x < lo { lo } else if x > hi { hi } else { x }
}
fn main() {
let n = 3 |> double |> add(10) |> clamp(0, 100)
println!("answer: {}", n)
}

// 一个请求路由器——Web 服务的核心，就在这里可运行。
fn route(method: &String, path: &String) -> String {
if method == &"GET" && path == &"/" { "200 Gossamer" }
else if method == &"GET" && path == &"/health" { "200 ok" }
else if method == &"POST" && path == &"/users" { "201 created" }
else { "404 not found" }
}
let requests = [("GET", "/"), ("GET", "/health"), ("POST", "/users"), ("GET", "/x")]
for (m, p) in requests {
println!("{} {} -> {}", m, p, route(&m, &p))
}

fn fib(n: i64) -> i64 {
if n < 2 { n } else { fib(n - 1) + fib(n - 2) }
}
fn main() {
// spawn 在一个 goroutine 上运行 fib；join 收集它的结果。
let h = spawn(|| fib(30))
match h.join() {
Ok(v) => println!("fib(30) on a goroutine = {}", v),
Err(e) => eprintln!("worker failed: {}", e),
}
}

use std::strings
enum Shape {
Circle(f64),
Rect { w: f64, h: f64 },
Triangle(f64, f64),
}
fn area(s: &Shape) -> f64 {
match s {
Shape::Circle(r) => 3.14159 * r * r,
Shape::Rect { w, h } => w * h,
Shape::Triangle(b, h) => 0.5 * b * h,
}
}
// 顶层语句——不需要 fn main()
let shapes = [Shape::Circle(3.0), Shape::Rect { w: 4.0, h: 5.0 }, Shape::Triangle(6.0, 2.0)]
for s in shapes {
println!("area = {:.2}", area(&s))
}

简短、聚焦的指南，把你已经掌握的知识映射到 Gossamer 上。

编写一个 .gos
文件，并用
gos
工具链运行它——在你迭代时
无需构建步骤：

$ gos run hello.gos
hello, gossamer
$ gos build --release hello.gos # 单一原生二进制文件，可随时交付

可在 Linux（x86_64、aarch64、armv7）、macOS（Intel 与 Apple Silicon）以及 Windows（x86_64）上运行。
