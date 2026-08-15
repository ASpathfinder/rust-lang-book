## 构建单线程 Web 服务器

首先让单线程 Web 服务器运行起来。开始之前，我们先快速概览构建 Web 服务器所涉及的协议。这些协议的细节超出本书范围，但简要概览将为你提供所需信息。

Web 服务器主要涉及两种协议：<em>超文本传输协议(Hypertext Transfer Protocol, HTTP)</em>和<em>传输控制协议(Transmission Control Protocol, TCP)</em>。两者都是<em>请求-响应(request-response)</em>协议，即由<em>客户端(client)</em>发起请求，<em>服务器(server)</em>监听请求并向客户端提供响应。请求和响应的内容由这些协议定义。

TCP 是较低层的协议，它描述信息如何从一台服务器传到另一台服务器的细节，却不指定这些信息是什么。HTTP 构建于 TCP 之上，负责定义请求与响应的内容。从技术上说，HTTP 也能与其他协议结合使用，但绝大多数情况下，HTTP 通过 TCP 发送数据。我们将直接处理 TCP 和 HTTP 请求与响应的原始字节。

### 监听 TCP 连接

Web 服务器需要监听 TCP 连接，因此先处理这一部分。标准库提供了 `std::net` 模块，让我们能够做到这一点。像往常一样创建新项目：

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

现在先把示例 21-1 中的代码输入 <em>src/main.rs</em>。这段代码会在本地地址 `127.0.0.1:7878` 监听传入的 TCP stream。收到传入 stream 时，它会打印 `Connection established!`。

<Listing number="21-1" file-name="src/main.rs" caption="监听传入 stream，并在收到 stream 时打印消息">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

使用 `TcpListener`，可以在地址 `127.0.0.1:7878` 监听 TCP 连接。地址中，冒号前的部分是表示你所用计算机的 IP 地址（每台计算机上都相同，并不是专指作者的计算机），`7878` 则是端口。选择这个端口有两个原因：通常不会在该端口接收 HTTP，所以我们的服务器不太可能与计算机上可能正在运行的其他 Web 服务器冲突；另外，在电话键盘上输入 7878 对应的是 <em>rust</em>。

这里的 `bind` 函数与 `new` 函数作用类似，会返回新的 `TcpListener` 实例。这个函数之所以称为 `bind`，是因为在网络术语中，连接到一个端口进行监听称为“<em>绑定到端口(binding to a port)</em>”。

`bind` 函数返回 `Result<T, E>`，表示绑定可能失败。例如，如果运行程序的两个实例，就会有两个程序监听同一个端口。我们编写的只是用于学习的基本服务器，所以不必费心处理这类错误；发生错误时直接使用 `unwrap` 停止程序。

`TcpListener` 上的 `incoming` 方法返回一个迭代器，它向我们提供一系列 stream（更具体地说，是 `TcpStream` 类型的 stream）。单个 <em>stream</em> 表示客户端与服务器之间的一条开放连接。<em>连接(connection)</em>是整个请求和响应过程的名称：客户端连接到服务器，服务器生成响应，然后服务器关闭连接。因此，我们将从 `TcpStream` 读取客户端发送的内容，再把响应写入 stream，将数据发回客户端。总的来说，这个 `for` 循环会依次处理每条连接，并产生一系列供我们处理的 stream。

目前，对 stream 的处理只是调用 `unwrap`：如果 stream 出现任何错误就终止程序；没有错误时则打印消息。下一个示例会为成功情况添加更多功能。客户端连接到服务器时，我们可能会从 `incoming` 方法收到错误，因为实际上迭代的不是连接，而是<em>连接尝试(connection attempt)</em>。连接可能因多种原因失败，其中很多原因与操作系统有关。例如，许多操作系统对能够同时保持打开的连接数设有限制；超过该数量的新连接尝试都会产生错误，直到一些打开的连接关闭为止。

来尝试运行这段代码！在终端调用 `cargo run`，然后在 Web 浏览器中加载 <em>127.0.0.1:7878</em>。浏览器应显示类似“Connection reset”的错误消息，因为服务器目前没有发回任何数据。但查看终端时，应能看到浏览器连接服务器时打印出的若干消息！

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

有时，一次浏览器请求会打印多条消息；原因可能是浏览器既请求了页面，也请求了其他资源，例如浏览器标签页中显示的 <em>favicon.ico</em> 图标。

也有可能是服务器没有返回任何数据，所以浏览器多次尝试连接。当 `stream` 离开作用域并在循环末尾被丢弃时，连接会作为 `drop` 实现的一部分关闭。浏览器有时会通过重试处理关闭的连接，因为问题可能只是暂时的。

浏览器有时还会在不发送任何请求的情况下打开与服务器的多条连接，以便以后<em>确实</em>发送请求时能够更快完成。发生这种情况时，无论连接上是否存在请求，我们的服务器都会看到每条连接。例如，许多版本的 Chromium 系浏览器都会这样做；可以使用隐私浏览模式或换用其他浏览器来禁用这项优化。

重要的是，我们已经成功取得了 TCP 连接的句柄！

每次运行完某一版本的代码后，请记得按 <kbd>ctrl</kbd>-<kbd>C</kbd> 停止程序。然后，在每组代码修改完成后调用 `cargo run` 命令重启程序，确保运行的是最新代码。

### 读取请求

下面实现从浏览器读取请求的功能！为了分离“先取得连接”与“再对连接执行操作”这两个关注点，我们会新建一个处理连接的函数。在新的 `handle_connection` 函数中，从 TCP stream 读取数据并将其打印出来，以便查看浏览器发送的数据。把代码改为示例 21-2 所示的样子。

<Listing number="21-2" file-name="src/main.rs" caption="从 `TcpStream` 读取并打印数据">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

我们把 `std::io::BufReader` 和 `std::io::prelude` 引入作用域，从而访问能够读写 stream 的 trait 和类型。在 `main` 函数的 `for` 循环中，现在不再打印表示建立连接的消息，而是调用新的 `handle_connection` 函数并向它传入 `stream`。

在 `handle_connection` 函数中，我们创建一个新的 `BufReader` 实例，包装对 `stream` 的引用。`BufReader` 通过替我们管理对 `std::io::Read` trait 方法的调用来添加缓冲。

我们创建名为 `http_request` 的变量，收集浏览器发送给服务器的请求行。通过添加 `Vec<_>` 类型标注，表明想把这些行收集到向量中。

`BufReader` 实现了提供 `lines` 方法的 `std::io::BufRead` trait。`lines` 方法每遇到换行字节便拆分数据流，返回 `Result<String, std::io::Error>` 的迭代器。为了取得每个 `String`，我们对每个 `Result` 执行 `map` 和 `unwrap`。如果数据不是有效 UTF-8，或者从 stream 读取时出现问题，`Result` 就可能是错误。再次强调，生产程序应更妥善地处理这些错误；但为了简单起见，我们选择在错误情况下停止程序。

浏览器通过连续发送两个换行字符表示 HTTP 请求结束，所以为了从 stream 取得一个请求，我们读取各行，直到遇到空字符串所在的行。把这些行收集到向量后，我们使用美化的调试格式打印它们，以便查看 Web 浏览器向服务器发送的指令。

来试试这段代码！启动程序，再次在 Web 浏览器中发起请求。请注意，浏览器仍会显示错误页面，但程序在终端中的输出现在应类似如下内容：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

根据所用浏览器，输出可能略有不同。现在开始打印请求数据后，可以查看请求第一行 `GET` 之后的路径，了解为什么一次浏览器请求会带来多条连接。如果重复的连接都在请求 <em>/</em>，就知道浏览器之所以反复获取 <em>/</em>，是因为没有收到程序的响应。

下面分解这些请求数据，理解浏览器在向程序请求什么。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### 更仔细地查看 HTTP 请求

HTTP 是基于文本的协议，请求采用以下格式：

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

第一行是<em>请求行(request line)</em>，其中保存客户端请求内容的信息。请求行的第一部分指出所用方法，例如 `GET` 或 `POST`，用于描述客户端如何发起请求。我们的客户端使用了 `GET` 请求，表示它正在请求信息。

请求行的下一部分是 <em>/</em>，它指出客户端请求的<em>统一资源标识符(uniform resource identifier, URI)</em>：URI 与<em>统一资源定位符(uniform resource locator, URL)</em>几乎相同，但并不完全一样。URI 与 URL 的差异对本章目的并不重要；不过 HTTP 规范使用 <em>URI</em> 一词，因此这里可以在心中直接用 <em>URL</em> 替代 <em>URI</em>。

最后一部分是客户端使用的 HTTP 版本，随后请求行以 CRLF 序列结束。（<em>CRLF</em> 代表<em>回车(carriage return)</em>和<em>换行(line feed)</em>，这些术语来自打字机时代！）CRLF 序列也可以写作 `\r\n`，其中 `\r` 是回车，`\n` 是换行。<em>CRLF 序列</em>把请求行与其余请求数据分隔开。请注意，打印 CRLF 时，我们看到的是新行开始，而不是 `\r\n`。

查看目前运行程序收到的请求行数据，可以看到 `GET` 是方法，<em>/</em> 是请求 URI，`HTTP/1.1` 是版本。

请求行之后，从 `Host:` 开始的其余各行都是标头。`GET` 请求没有主体。

尝试从另一个浏览器发出请求，或者请求不同地址（例如 <em>127.0.0.1:7878/test</em>），观察请求数据如何变化。

现在已经知道浏览器在请求什么，下面发回一些数据！

### 写入响应

我们将实现向客户端请求返回数据的功能。响应具有以下格式：

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

第一行是<em>状态行(status line)</em>，其中包含响应所用的 HTTP 版本、概括请求结果的数字状态码，以及对状态码进行文字说明的原因短语。CRLF 序列之后是任意标头、另一个 CRLF 序列，以及响应主体。

下面是一个示例响应：它使用 HTTP 1.1 版，状态码为 200，原因短语为 OK，没有标头，也没有主体：

```text
HTTP/1.1 200 OK\r\n\r\n
```

状态码 200 是标准的成功响应。这段文本是一个极小的成功 HTTP 响应。让我们把它写入 stream，作为对成功请求的响应！在 `handle_connection` 函数中删除打印请求数据的 `println!`，用示例 21-3 中的代码替换它。

<Listing number="21-3" file-name="src/main.rs" caption="向 stream 写入极小的成功 HTTP 响应">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

新增的第一行定义变量 `response`，用于保存成功消息的数据。然后对 `response` 调用 `as_bytes`，将字符串数据转换成字节。`stream` 上的 `write_all` 方法接收 `&[u8]`，并沿连接直接发送这些字节。由于 `write_all` 操作可能失败，我们像以前一样对任何错误结果使用 `unwrap`。同样，实际应用应在这里添加错误处理。

完成这些更改后，运行代码并发出请求。现在不再向终端打印任何数据，所以除了 Cargo 的输出以外什么也看不到。在 Web 浏览器中加载 <em>127.0.0.1:7878</em> 时，应看到空白页面，而不是错误。你刚刚手工编写了接收 HTTP 请求并发送响应的代码！

### 返回真正的 HTML

下面实现返回非空白页面的功能。在项目目录根部而不是 <em>src</em> 目录中新建文件 <em>hello.html</em>。可以输入任意 HTML；示例 21-4 给出了一种写法。

<Listing number="21-4" file-name="hello.html" caption="响应中返回的示例 HTML 文件">

```html
{{#include ../../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

这是一个带有标题和一些文本的最小 HTML5 文档。为了在收到请求时从服务器返回它，我们会像示例 21-5 那样修改 `handle_connection`：读取 HTML 文件，把它作为主体添加到响应，然后发送。

<Listing number="21-5" file-name="src/main.rs" caption="把 <em>hello.html</em> 的内容作为响应主体发送">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

我们在 `use` 语句中添加 `fs`，把标准库的文件系统模块引入作用域。读取文件内容为字符串的代码应该很熟悉；在示例 12-4 的 I/O 项目中读取文件内容时，我们使用过它。

接下来使用 `format!` 把文件内容添加为成功响应的主体。为了确保 HTTP 响应有效，我们添加 `Content-Length` 标头，并将它设为响应主体的大小——这里就是 `hello.html` 的大小。

用 `cargo run` 运行这段代码，然后在浏览器中加载 <em>127.0.0.1:7878</em>；应该能看到 HTML 渲染出的内容！

目前，我们忽略了 `http_request` 中的请求数据，只是无条件发回 HTML 文件的内容。这意味着，即使尝试在浏览器中请求 <em>127.0.0.1:7878/something-else</em>，仍会收到相同的 HTML 响应。目前的服务器非常有限，没有完成大多数 Web 服务器会做的事情。我们希望根据请求自定义响应，并且只对格式正确、指向 <em>/</em> 的请求发回 HTML 文件。

### 验证请求并有选择地响应

现在，无论客户端请求什么，Web 服务器都会返回文件中的 HTML。我们来添加功能：返回 HTML 文件之前，检查浏览器是否正在请求 <em>/</em>；如果浏览器请求其他内容，则返回错误。为此，需要像示例 21-6 那样修改 `handle_connection`。新代码把收到的请求内容与已知的 <em>/</em> 请求形式进行比较，并添加 `if` 与 `else` 块，用不同方式处理请求。

<Listing number="21-6" file-name="src/main.rs" caption="以不同于其他请求的方式处理对 <em>/</em> 的请求">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

我们只会查看 HTTP 请求的第一行，所以不再把整个请求读入向量，而是调用 `next` 从迭代器取得第一项。第一个 `unwrap` 处理 `Option`，如果迭代器没有任何项就停止程序。第二个 `unwrap` 处理 `Result`，效果与示例 21-2 中添加到 `map` 里的 `unwrap` 相同。

接着检查 `request_line`，看看它是否等于对 <em>/</em> 路径发出的 GET 请求行。如果相等，`if` 块就返回 HTML 文件的内容。

如果 `request_line` <em>不</em>等于对 <em>/</em> 路径发出的 GET 请求，就表示收到了其他请求。稍后会在 `else` 块中添加代码来响应所有其他请求。

现在运行这段代码并请求 <em>127.0.0.1:7878</em>；应该会收到 <em>hello.html</em> 中的 HTML。如果发出任何其他请求，例如 <em>127.0.0.1:7878/something-else</em>，会得到与运行示例 21-1 和 21-2 的代码时类似的连接错误。

现在把示例 21-7 的代码添加到 `else` 块，返回状态码为 404 的响应，表示没有找到请求的内容。我们还会返回一些 HTML，让浏览器渲染一个向最终用户说明响应的页面。

<Listing number="21-7" file-name="src/main.rs" caption="如果请求的内容不是 <em>/</em>，以状态码 404 和错误页面响应">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

这里的响应状态行包含状态码 404 和原因短语 `NOT FOUND`。响应主体是文件 <em>404.html</em> 中的 HTML。接下来需要在 <em>hello.html</em> 旁边创建 <em>404.html</em> 文件作为错误页面；同样，可以任意使用自己想要的 HTML，也可以使用示例 21-8 中的示例 HTML。

<Listing number="21-8" file-name="404.html" caption="随任何 404 响应发回的示例页面内容">

```html
{{#include ../../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

完成这些更改后，再次运行服务器。请求 <em>127.0.0.1:7878</em> 应返回 <em>hello.html</em> 的内容；任何其他请求（例如 <em>127.0.0.1:7878/foo</em>）都应返回 <em>404.html</em> 中的错误 HTML。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### 重构

目前，`if` 与 `else` 块中有大量重复：它们都读取文件，并把文件内容写入 stream。唯一的区别是状态行与文件名。我们来提取这些差异，把它们放入单独的 `if` 和 `else` 行中，为状态行与文件名变量赋值；然后便可以在读取文件和写入响应的代码中无条件使用这些变量。示例 21-9 展示了替换较大的 `if` 和 `else` 块后得到的代码。

<Listing number="21-9" file-name="src/main.rs" caption="重构 `if` 与 `else` 块，使其只包含两种情况中不同的代码">

```rust,no_run
{{#rustdoc_include ../../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

现在，`if` 与 `else` 块只在元组中返回适当的状态行和文件名值；然后，正如第 19 章讨论的那样，使用 `let` 语句中的模式进行解构，把这两个值赋给 `status_line` 和 `filename`。

之前重复的代码现在位于 `if` 与 `else` 块之外，并使用 `status_line` 和 `filename` 变量。这让两种情况之间的差异更容易看清，也意味着如果想改变文件读取和响应写入的方式，只需更新一处代码。示例 21-9 中代码的行为与示例 21-7 相同。

太棒了！现在我们用大约 40 行 Rust 代码实现了一个简单 Web 服务器：它会用内容页面响应一种请求，并对所有其他请求返回 404 响应。

目前，服务器运行在单个线程中，意味着它一次只能服务一个请求。下面通过模拟一些缓慢请求，看看这会带来什么问题。然后，我们会修复它，让服务器能够同时处理多个请求。
