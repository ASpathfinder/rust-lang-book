# 简体中文翻译规范

## 基准与范围

- 英文源基准：`917544888a55e4da7109bdba8c88c893c0da70f4`
- Rust 版本：1.97.0 或更高版本
- Edition：Rust 2024 Edition
- 译文使用准确、自然、面向初学者的简体中文，不逐句硬译，也不增加原文
  没有的技术结论。

## 术语标注

- 专业术语采用 `中文(English)` 格式，使用半角括号，括号前后不加空格。
- 每章第一次在正文中实质出现时标注，后续只使用统一中文译名。
- 同一章拆分成多个 Markdown 文件时仍视为同一个标注范围。
- 前言和引言各自视为独立标注范围。
- 目录、标题、代码、命令、URL 和文件名不计作术语的首次出现。
- Rust 关键字、标识符、工具名和类型名保持代码格式，不强行翻译。
- 中文正文的斜体使用 `<em>强调内容</em>`，粗体使用
  `<strong>强调内容</strong>`。不要依赖 `*...*`、`**...**` 或 `_..._`；这些
  Markdown 定界符与中文字符相邻时可能无法识别，或产生错误嵌套。

## 内容和格式

- 翻译标题、正文、提示、表格、图片替代文本、示例说明和图注。
- Rust 代码、代码注释、字符串、命令、终端输出和编译器诊断保持原样。
- 保留原文的段落、列表、代码围栏、HTML 锚点和语义标签结构。
- 中文标点使用全角形式；代码、路径和英文原文中的标点保持原样。
- 已翻译页面之间使用内部链接；指向未翻译内容的链接暂时指向 Rust 官方
  英文在线版本，翻译对应页面后再改回内部链接。

## 术语表

| 英文 | 统一译名 | 备注 |
| --- | --- | --- |
| ahead-of-time compiled | 预先编译 | 首次出现时标注 |
| abstraction | 抽象 |  |
| alias | 别名 |  |
| arm | 分支 | `match` 和 `if` 语境 |
| application programming interface (API) | 应用程序编程接口(API) | 保留缩写 |
| array | 数组 |  |
| associated type | 关联类型 | 特征语境 |
| attribute | 属性 | Rust 标注语法语境 |
| backtrace | 回溯 | panic 与调试语境 |
| bad state | 错误状态 | 错误处理语境 |
| binary | 二进制文件 | 按上下文也可译为“二进制程序” |
| binary target | 二进制目标 | Cargo 语境 |
| binding | 绑定 |  |
| borrowing | 借用 | 所有权语境 |
| borrow checker | 借用检查器 |  |
| build system | 构建系统 | Cargo 语境 |
| buffer overread | 缓冲区过度读取 | 安全语境 |
| command line | 命令行 |  |
| closure | 闭包 |  |
| compiler | 编译器 |  |
| coherence | 一致性 | 特征实现语境 |
| configuration | 配置 | `cfg` 语境 |
| compound type | 复合类型 |  |
| concurrency | 并发 |  |
| contract | 契约 | API 与调用方约定 |
| concern / separation of concerns | 关注点 / 关注点分离 | 程序结构语境 |
| consuming adapter | 消耗适配器 | 迭代器语境 |
| crate | crate | Rust 的编译与分发单元，不译为“箱” |
| crate root | crate 根 |  |
| dangling pointer / reference | 悬垂指针 / 悬垂引用 |  |
| data race | 数据竞争 |  |
| deep copy / shallow copy | 深拷贝 / 浅拷贝 |  |
| dependency | 依赖 |  |
| dereferencing | 解引用 |  |
| double free | 二次释放 |  |
| documentation comment | 文档注释 | `///` 与 `//!` |
| enumeration / enum | 枚举 / enum |  |
| error propagation / propagating errors | 错误传播 / 传播错误 |  |
| executable | 可执行文件 |  |
| expression | 表达式 |  |
| field | 字段 | 结构体语境 |
| field init shorthand | 字段初始化简写 |  |
| filtering | 筛选 | 测试选择语境 |
| function | 函数 |  |
| functional programming | 函数式编程 |  |
| getter | 获取器 |  |
| generic type parameter | 泛型类型形参 |  |
| generics | 泛型 |  |
| glob operator | glob 运算符 |  |
| grapheme cluster | 字形簇 | Unicode 语境 |
| hash map | 哈希表 |  |
| hasher | 哈希器 |  |
| hashing function | 哈希函数 |  |
| integrated development environment (IDE) | 集成开发环境(IDE) | 保留缩写 |
| integration test | 集成测试 |  |
| iterator | 迭代器 |  |
| iterator adapter | 迭代器适配器 |  |
| iterator pattern | 迭代器模式 |  |
| lazy | 惰性 | 迭代器求值语境 |
| linker | 链接器 |  |
| lifetime | 生命周期 |  |
| lifetime annotation | 生命周期标注 |  |
| lifetime elision rule | 生命周期省略规则 |  |
| macro | 宏 |  |
| metadata | 元数据 |  |
| memory safety | 内存安全 |  |
| method | 方法 |  |
| module | 模块 |  |
| module system | 模块系统 |  |
| module tree | 模块树 |  |
| move | 移动 | 所有权转移语境 |
| monomorphization | 单态化 |  |
| mutable reference | 可变引用 |  |
| ownership | 所有权 |  |
| orphan rule | 孤儿规则 | 特征实现的一致性规则 |
| package | 包 | Cargo 语境 |
| package manager | 包管理器 | Cargo 语境 |
| parameter | 形参 | 与 argument 区分 |
| argument | 实参 | 与 parameter 区分 |
| pattern | 模式 |  |
| pointer | 指针 |  |
| privacy | 私有性 |  |
| reference | 引用 |  |
| recoverable / unrecoverable error | 可恢复错误 / 不可恢复错误 |  |
| re-exporting | 重新导出 |  |
| release profile | 发布配置 | Cargo 构建配置语境 |
| scalar | 标量 |  |
| scope | 作用域 |  |
| shadowing | 遮蔽 |  |
| slice | 切片 |  |
| stack / heap | 栈 / 堆 |  |
| statement | 语句 |  |
| standard output / standard error | 标准输出 / 标准错误 | `stdout` / `stderr` |
| struct | 结构体 |  |
| struct update syntax | 结构体更新语法 |  |
| syntactic sugar | 语法糖 |  |
| string slice | 字符串切片 |  |
| source file | 源文件 |  |
| systems programming | 系统编程 |  |
| test | 测试 |  |
| test-driven development (TDD) | 测试驱动开发(TDD) | 保留缩写 |
| trait | 特征 | Rust 类型系统语境 |
| trait bound | 特征约束 |  |
| blanket implementation | 毯式实现 |  |
| tuple struct | 元组结构体 |  |
| unit-like struct | 类单元结构体 |  |
| unit test | 单元测试 |  |
| unwinding / aborting | 展开 / 中止 | panic 处理方式 |
| variant | 变体 | 枚举语境 |
| vector | 向量 |  |
| workspace | 工作区 | Cargo 语境 |
| yanking | 撤回 | crates.io 版本管理语境 |
| zero-cost abstraction | 零成本抽象 |  |
| box | 箱子 | `Box<T>` 智能指针语境 |
| deref coercion | 解引用强制转换 |  |
| destructor / constructor | 析构函数 / 构造函数 |  |
| indirection | 间接寻址 | 递归类型语境 |
| interior mutability | 内部可变性 |  |
| memory leak | 内存泄漏 |  |
| reference counting | 引用计数 |  |
| reference cycle | 引用循环 |  |
| smart pointer | 智能指针 |  |
| strong reference / weak reference | 强引用 / 弱引用 | `Rc<T>` 与 `Weak<T>` 语境 |
| test double / mock object | 测试替身 / 模拟对象 | 测试语境 |
| atomic | 原子 | 并发语境 |
| blocking | 阻塞 | 线程语境 |
| channel | 通道 | 消息传递语境 |
| concurrent / parallel programming | 并发编程 / 并行编程 |  |
| deadlock | 死锁 |  |
| fearless concurrency | 无畏并发 |  |
| lock / mutex | 锁 / 互斥锁 | 并发语境 |
| marker trait | 标记 trait |  |
| message passing | 消息传递 |  |
| process / thread | 进程 / 线程 |  |
| race condition | 竞态条件 |  |
| shared state / shared memory | 共享状态 / 共享内存 | 并发语境 |
| asynchronous programming | 异步编程 |  |
| await point | 等待点 |  |
| cooperative multitasking | 协作式多任务 |  |
| executor | 执行器 | 异步运行时语境 |
| future | future | 保留英文小写；代码类型写作 `Future` |
| I/O-bound / CPU-bound | I/O 密集型 / CPU 密集型 |  |
| parallelism / concurrency | 并行 / 并发 |  |
| pin / pinning | 固定 | `Pin` 类型语境 |
| polling | 轮询 | Future 与 Stream 语境 |
| postfix | 后缀 | 语法语境 |
| promise | promise | 与其他语言的 future 类比时保留英文 |
| runtime | 运行时 | 异步语境 |
| serial | 串行 |  |
| state machine | 状态机 |  |
| stream | stream | 保留英文小写；代码类型写作 `Stream` |
| task | 任务 | 异步运行时语境 |
| work stealing | 工作窃取 | 运行时调度语境 |
| bounded parametric polymorphism | 有界参数多态 |  |
| duck typing | 鸭子类型 |  |
| dynamic / static dispatch | 动态分派 / 静态分派 |  |
| dyn compatibility | dyn 兼容性 |  |
| encapsulation | 封装 | 面向对象语境 |
| inheritance | 继承 |  |
| object-oriented programming (OOP) | 面向对象编程(OOP) | 保留缩写 |
| polymorphism | 多态 |  |
| single inheritance | 单继承 |  |
| state object / state pattern | 状态对象 / 状态模式 | 设计模式语境 |
| trait object | 特征对象 |  |
| at operator | at 运算符 | 模式中的 `@` |
| catch-all pattern | 全匹配模式 |  |
| exhaustive / exhaustiveness | 穷尽的 / 穷尽性 | 模式匹配语境 |
| irrefutable / refutable pattern | 不可反驳模式 / 可反驳模式 |  |
| match arm | 匹配分支 |  |
| match guard | 匹配守卫 |  |
| refutability | 可反驳性 |  |
| wildcard pattern | 通配符模式 |  |
| application binary interface (ABI) | 应用二进制接口(ABI) | 保留缩写 |
| attribute-like macro | 类属性宏 | 过程宏语境 |
| declarative macro | 声明宏 |  |
| diverging function | 发散函数 | 返回 `!` 的函数 |
| dynamically sized type (DST) | 动态大小类型(DST) | 保留缩写 |
| Foreign Function Interface (FFI) | 外部函数接口(FFI) | 保留缩写 |
| function-like macro | 类函数宏 | 过程宏语境 |
| function pointer | 函数指针 | `fn` 类型 |
| invariant | 不变条件 | 安全契约语境 |
| mangling | 名称混淆 | 符号名称语境 |
| metaprogramming | 元编程 |  |
| never type | never 类型 | `!` 类型 |
| newtype pattern | newtype 模式 | 保留 newtype |
| opaque type | 不透明类型 | `impl Trait` 语境 |
| operator overloading | 运算符重载 |  |
| procedural macro | 过程宏 |  |
| raw pointer | 原始指针 |  |
| static variable | 静态变量 |  |
| supertrait | 超 trait |  |
| type alias / type synonym | 类型别名 / 类型同义词 |  |
| undefined behavior | 未定义行为 |  |
| unsafe Rust | 不安全 Rust |  |
| unsized type | 无大小类型 | 亦称动态大小类型 |
| client / server | 客户端 / 服务器 | 网络语境 |
| compiler-driven development | 编译器驱动开发 |  |
| connection attempt | 连接尝试 | 网络语境 |
| denial-of-service (DoS) attack | 拒绝服务(DoS)攻击 | 保留缩写 |
| graceful shutdown | 优雅停机 |  |
| Hypertext Transfer Protocol (HTTP) | 超文本传输协议(HTTP) | 保留缩写 |
| poisoned mutex | 中毒的互斥锁 |  |
| producer / consumer | 生产者 / 消费者 | 通道语境 |
| request line / status line | 请求行 / 状态行 | HTTP 语境 |
| request-response protocol | 请求-响应协议 |  |
| socket | 套接字 | 网络语境 |
| thread pool | 线程池 |  |
| throughput | 吞吐量 |  |
| Transmission Control Protocol (TCP) | 传输控制协议(TCP) | 保留缩写 |
| uniform resource identifier (URI) | 统一资源标识符(URI) | 保留缩写 |
| uniform resource locator (URL) | 统一资源定位符(URL) | 保留缩写 |
| web server | Web 服务器 |  |
| derivable trait | 可派生的 trait | `derive` 语境 |
| edition | 版本 | Rust 2015/2018/2021/2024 语境；必要时保留 `Edition` |
| end of life (EOL) | 生命周期终点(EOL) | 保留缩写 |
| feature flag / feature gate | 功能标志 / 功能门 | 不稳定功能语境 |
| identifier / raw identifier | 标识符 / 原始标识符 |  |
| Language Server Protocol (LSP) | 语言服务器协议(LSP) | 保留缩写 |
| release channel | 发布通道 | nightly/beta/stable 语境 |
| Request For Comments (RFC) | 征求意见(RFC) | 保留缩写 |
| toolchain | 工具链 |  |
| train model | 列车模型 | Rust 发布流程语境 |
| turbofish | 涡轮鱼 | `::<...>` 语法俗称 |
