---
layout:     post
title:      "【译】【图文】标准化中的 WASI：在 web 之外运行 WebAssembly 的系统接口"
date:       2019-04-04 12:52:27 +0800
categories: wasm
---
> 本文已获得翻译授权，译自 [Standardizing WASI: A system interface to run WebAssembly outside the web](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/)，由作者 [Lin Clark](https://twitter.com/linclark) 于当地时间 2019-03-27 发布。

今天（当地时间 2019-03-27），我们宣布开始进行一项新的标准化工作——WASI，WebAssembly 系统接口（WebAssembly System Interface）。

<!--more-->

**起因（Why）：** 开发人员开始将 WebAssembly 向浏览器之外推进，因为它提供了一种快速、可扩展、安全的方式来在所有计算机上运行相同的代码。

但是我们尚未建立一个坚实的基础。
浏览器之外的代码需要一种与系统交互的方式——系统接口。
而 WebAssembly 平台还没有系统接口。

**事物（What）：** WebAssembly 是概念机的汇编语言，而不是物理机的汇编语言。
这就是它可以在各种不同计算机体系结构上运行的原因。

正因为 WebAssembly 是概念机的汇编语言，所以 WebAssembly 需要一个概念操作系统的系统接口，而不是任何单一操作系统的系统接口。
这样，它就可以在所有不同操作系统中运行。

这就是 WASI——WebAssembly 平台的系统接口。

我们的目标是创建一个系统接口，它会成为 WebAssembly 的真正伴侣，并经受起时间的考验。
这意味着坚持 WebAssembly 的关键原则——可移植性与安全性。

**人物（Who）：** 我们正在组建一个 WebAssembly 的一个子工作组，专注于 [WASI](https://wasi.dev/) 标准化。
我们已经聚集了一些志同道合的合作伙伴，并且还在寻找更多伙伴加入。

以下是我们、我们的合作伙伴与支持者认为这很重要的一些原因：

#### Sean White，Mozilla 的首席研发官
“WebAssembly 已经在改变 web 给人们带来新的引人入胜的内容的方式，并使开发者与创作者能在 web 上尽显身手。
到目前为止，已经通过浏览器实现了，不过有了 WASI，我们可以将 WebAssembly 与 web 的益处传递给更多用户、更多场合、更多设备，以及作为更多体验的一部分。”

#### Tyler McMullen，Fastly 的 CTO
“我们将 WebAssembly 带到浏览器之外，在我们的边缘云中作为快速、安全地执行代码的平台。
虽然我们的边缘与浏览器之间存在环境差异，但是 WASI 意味着 WebAssembly 开发者无需将代码移植到每个不同的平台。”

#### Myles Borins，Node 技术指导委员会主任
“WebAssembly 可以解决 Node 中最大的问题之一——如何获得接近原生速度，并像使用原生模块一样复用其他语言（如 C 与 C++）编写的代码，同时仍保持可移植性与安全性。
标准化这个系统接口是实现这一目标的第一步。”

#### Laurie Voss，npm 的联合创始人
“npm 极为感兴趣的是，WebAssembly 潜在具有扩展 npm 生态系统、并同时极大地简化在服务端 JavaScript 应用程序中运行原生代码过程的能力。
我们期待这个过程的结果。”

所以说这是个大新闻！🎉

WASI 目前有三份实现:

- [wasmtime](https://github.com/CraneStation/wasmtime)，Mozilla 的 WebAssembly 运行时
- [Lucet](https://www.fastly.com/blog/announcing-lucet-fastly-native-webassembly-compiler-runtime)，Fastly 的 WebAssembly 运行时
- [浏览器垫片（polyfill）](https://wasi.dev/polyfill/)

（如果你能访问 YouTube）可以在这个视频中看个 WASI 实战：

<iframe width="560" height="315" src="https://www.youtube.com/embed/ggtEJC0Jv8A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

如果你希望了解关于这个系统接口应该如何工作的提案的更多信息，请继续阅读。

### 系统接口是什么？
很多人说像 C 这样的语言可以直接访问系统资源。
但事实并非如此。

这些语言无法直接在大多数系统上进行打开或创建文件等操作。
为什么不能呢？

因为这些系统资源（诸如文件、内存以及网络连接）对于稳定性与安全性来说太重要了。

如果一个程序无意中搅乱了另外一个程序的资源，那么它可能会使另一个程序崩溃。
更糟的是，如果一个程序（或用户）故意干扰另一个程序的资源，那么它可能会窃取敏感数据。

![表示崩溃的皱着眉头的终端窗口，以及表示数据泄露的带有坏锁的文件](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/01-01_crash-data-leak-1.png)

因此，我们需要一种方式来控制哪些程序与用户可以访问哪些资源。
人们很早就发现了这一点，并想出了一个提供这种控制的方式：保护环安全。

有了保护环安全，操作系统基本上在系统资源外围设置了保护屏障。
这就是内核。
内核是唯一可以进行创建新文件、打开文件或者打开网络连接等操作的地方。

用户的程序在称之为用户模式的内核以外运行。
如果程序想做打开文件这样的事，它必须请求内核为它打开。

![左侧是一个文件目录结构，中间有一个包含操作系统内核的保护屏障，右侧是一个敲门访问的应用程序](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/01-02-protection-ring-sec-1.png)

这就是系统调用的概念所在。
当程序需要让内核执行这其中某一项操作时，它会使用系统调用来请求。
这让内核有机会找出是哪个用户在请求。
然后就可以在打开文件之前分辨该用户是否有权访问该文件。

在大多数设备上，这是代码可以访问系统资源的唯一方式——通过系统调用。

![请求操作系统将数据放入已打开文件的应用程序](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/01-03-syscall-1.png)

操作系统让系统调用可用。
但是如果每个操作系统都有自己的系统调用，那岂不是需要为每个操作系统编写不同版本的代码？
幸运的是，并不用。

这个问题是如何解决的呢？——抽象。

大多数语言都提供了标准库。
在编码时，程序员并不需要知道他们所面向的系统。
他们只是使用相应接口。

然后在编译时，工具链会根据所面向的目标系统来选择使用该接口的哪个实现。
这个实现会使用操作系统 API 中的函数，因此它是平台相关的。

这就是系统接口的用武之地。
例如，为 Windows 计算机编译的 `printf` 可以使用 Windows API 与该计算机进行交互。
而为 Mac 或者 Linux 编译，则会改用 POSIX。

![putc 的接口会翻译为为两种不同的实现，一种使用 POSIX 实现，另一种使用 Windows API 实现](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/02-01-implementations-1.png)

然而这却给 WebAssembly 带来了一个问题。

对于 WebAssembly 来说，即使在编译时也无从知晓所面向的是哪种操作系统。
因此，无法在标准库的 WebAssembly 实现中使用任何单一操作系统的系统接口。

![putc 的空实现](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/02-02-implementations-1.png)

我之前说过 WebAssembly 是[一种概念机的汇编语言](https://hacks.mozilla.org/2017/02/creating-and-working-with-webassembly-modules/)，而不是真实计算机的汇编语言。
同样，WebAssembly 也需要一套概念操作系统（而不是真实操作系统）的系统接口。

不过已经存在可以在浏览器之外运行 WebAssembly 的运行时了，即便没有这个系统接口。
他们是怎么做到的呢？我们来看一看。

### 如今 WebAssembly 是如何在浏览器之外运行的？
生成 WebAssembly 的第一个工具是 Emscripten。
它在 web 上模拟一个特定操作系统的系统接口 POSIX。
这意味着程序员可以使用 C 标准库（libc）中的函数。

为此，Emscripten 创建了自己的 libc 实现。
这个实现分为两部分——一部分编译成 WebAssembly 模块，另一部分用 JS 胶水代码实现。
这个 JS 胶水层会调用浏览器，进而与操作系统交互。

![一个 Rube Goldberg 机展示了一个调用如何从 WebAssembly 模块进入到 Emscripten 的 JS 胶水代码，再进入到浏览器，再进入到内核](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/03-01-emscripten-1.png)

大多数早期的 WebAssembly 代码都是使用 Emscripten 编译的。
因此，当人们开始想在没有浏览器的情况下运行 WebAssembly 代码时，他们会从让 Emscripten 所编译的代码能运行入手。

于是，这些运行时需要为 JS 胶水代码中的所有函数创建自己的实现。

不过，这里有个问题。
这个 JS 胶水代码所提供的接口并没有设计成标准接口，甚至并非面向公众的接口。
这并不是它所解决的问题。

例如，对于一个在设计成公开接口的 API 中名为 `read` 的函数，其 JS 胶水代码使用的是 `_system3(which, varargs)`。

![一个清晰的 read 接口，对比一个令人困惑的 system3](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/03-02-system3-1.png)

第一个参数 `which` 是一个整数，它始终与名称中的数字相同（在本例中是 3）。

第二个参数 `varargs` 是用到的参数。
它之所以称为 `varargs`，是因为可以有可变数量的参数。
但是 WebAssembly 并没有提供将可变数量参数传给函数的方式。
于是，这些参数通过线性内存传递。
这不是类型安全的做法，而且也比使用寄存器传递参数（如果可能的话）慢。

对于在浏览器中运行 Emscripten 来说，这没有问题。
但是现在运行时将其视为事实上的标准，实现了自己的一版 JS 胶水代码。
他们是在模拟 POSIX 仿真层的内部细节。

这意味着他们正在重新实现那些对于 Emscripten 的约束有意义的选择（例如将参数作为堆中值传递），即便这些约束并不适于他们的环境。

![一个更复杂的 Rube Goldberg 机，其中 JS 胶水与浏览器都是由 WebAssembly 运行时模拟的](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/03-03-emulation-1.png)

如果我们要构建一个持续数十年的 WebAssembly 生态系统，我们就需要坚实的基础。
这意味着我们的事实标准不能是仿真的仿真。

不过我们应该采用什么原则呢？

### WebAssembly 系统接口需要坚持什么原则？
有两项重要的原则已经融入到 WebAssembly 中：

- 可移植性
- 安全性

当我们转向浏览器之外的应用场景时，我们需要坚持这些关键原则。

实际上，POSIX 与 Unix 的安全访问控制方式并没有帮我们达到目的。
我们来看下它们的不足之处。

#### 可移植性
POSIX 提供了源代码级的可移植性。
相同的源代码可以与不同版本的 libc 一起编译来面向不同的计算机。

![一个 C 源文件编译成多个二进制文件](https://hacks.mozilla.org/files/2019/03/04-01-portability-1-768x576.png)

但是 WebAssembly 需要超越它一步。
我们需要能够编译一次就能在一系列不同的计算机上运行。
我们需要可移植的二进制文件。

![一个 C 源文件编译成单个二进制文件](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/04-02-portability-1.png)

这种可移植性让用户分发代码更容易。

例如，Node 的原生模块如果是用 WebAssembly 编写的，那么当用户安装带有原生模块的应用时就不需要运行 node-gyp 了，开发人员也无需配置并分发几十个二进制文件了。

#### 安全性
当一行代码请求操作系统执行某些输入或输出时，操作系统需要确定该代码所请求的操作是否安全。

操作系统通常使用基于所有权与组的访问控制来处理这个问题。

例如，程序可能会请求操作系统打开一个文件。
一个用户具有他有权访问的特定一组文件。

当该用户启动程序时，该程序代表用户运行。
如果这个用户有权访问某文件（要么因为他是所有者，要么因为他在有权访问的组里），那么该程序也能访问。

![请求打开与其所执行操作相关的文件的应用程序](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/04-03-access-control-1.png)

这在用户之间提供了保护。
在早期操作系统开发出来时很有意义。
系统通常是多用户的，而管理员控制安装什么软件。
所以最突出的威胁是其他用户偷看你的文件。

情况已经变了。
现在系统通常是单个用户，但是他们会运行引入了许多未知可信度的其他第三方代码的代码。
现在最大的威胁是你自己运行的代码会对你不利。

例如，假设你在应用程序中使用的库来了一个新的维护者（在开源项目中经常发生）。
该维护者可能会对你的兴趣很上心……也可能是个坏人。
如果这些代码有权在你的系统上做任何事（例如，打开你的任何文件并通过网络发送出去），那么其代码会造成很大的损害。

![An evil application asking for access to the users bitcoin wallet and opening up a network connection](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/04-04-bitcoin-1.png)

这就是为什么使用可以直接与系统交互的第三方库可能是危险的。

WebAssembly 实现安全性的方式与此不同。
WebAssembly 采用了沙箱。

这意味着代码不能直接与操作系统交互。
那么它是如何利用系统资源的呢？
宿主机（可能是浏览器，也可能是 wasm 运行时）将函数放入代码可以使用的沙箱中。

这意味着宿主机可以逐一限制每个程序可以做什么。
它并不是仅仅让程序代表用户、使用该用户的完整权限调用任何系统调用。

只是拥有沙箱机制并不会使系统本身变安全（宿主机仍然可以将所有能力都放入到沙箱中，若是这种情况则并没有变好），不过它至少让宿主机能够选择创建更安全的系统。

![A runtime placing safe functions into the sandbox with an application](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/04-05-sandbox-1.png)

在我们设计的任何系统接口中，我们都需要坚持这两项原则。
可移植性让开发与分发软件更容易，而为宿主机提供保护自身或用户的工具更是绝对必需。

### 这个系统接口应该是什么样的？
鉴于这两项关键原则，WebAssembly 系统接口应该设计成什么样的？

这正是我们要通过标准化过程得出的成果。
不过我们确实有个提案要启动：

- 创建模块化的一组标准接口
- 开始标准化最基本的模块 wasi-core

![Multiple modules encased in the WASI standards effort](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-01-wasi-1.png)

wasi-core 里会有什么？

wasi-core 会包含所有程序都需要的基本接口。
它会覆盖与 POSIX 近乎相同的领域，包括诸如文件、网络连接、时钟以及随机数。

并且其中很多会采用与 POSIX 非常类似的方式。
例如，它会使用 POSIX 的面向文件的方式，其中有诸如 open、close、read 以及 write 这样的系统调用，而所有其他内容基本都是在此之上提供的扩充。

不过 wasi-core 并不会覆盖所有 POSIX 的内容。
例如，进程概念并没有清晰映射到 WebAssembly 上。
更进一步，让每个 WebAssembly 引擎都需要支持像 `fork` 这样的进程操作并无意义。
当然我们也希望标准化 `fork` 成为可能。

这就模块化方式的用武之地。
通过这种方式，我们可以获得良好的标准化覆盖率，同时仍然让一些平台能够只使用对其有意义的 WASI 部分。

![Modules filled in with possible areas for standardization, such as processes, sensors, 3D graphics, etc](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-02-wasi-1.png)

像 Rust 这样的语言会直接在其标准库中使用 wasi-core。
例如，Rust 的 `open` 在编译到 WebAssembly 时会通过调用 `__wasi_path_open` 来实现。

对于 C 与 C++，我们创建了一个 [wasi-sysroot](https://github.com/CraneStation/wasi-sysroot)，它根据 wasi-core 实现了 libc。

![The Rust and C implementations of openat with WASI](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-03-open-imps-1.png)

我们期望像 Clang 这样的编译器准备好与 WASI API 交互，并完成像 Rust 编译器与 Emscripten 这样的工具链，将 WASI 作为其系统实现的一部分。

用户代码如何调用这些 WASI 函数？

运行用户代码的运行时会将 wasi-core 函数作为导入传入。

![A runtime placing an imports object into the sandbox](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-04-imports-1.png)

这为我们提供了可移植性，因为每个宿主机都可以有专为其平台（从像 Mozilla’s wasmtime 与 Fastly 的 Lucet，到 Node 乃至浏览器）编写的自己的 wasi-core 实现。

它还为我们提供了沙箱，因为宿主机可以逐个程序选择哪些 wasi-core 函数可以传入（即允许哪些系统调用）。
这保持了安全性。

![Three runtimes—wastime, Node, and the browser—passing their own implementations of wasi_fd_open into the sandbox](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-05-sec-port-2.png)

WASI 为我们提供了进一步扩展这种安全性的方式。
它从基于能力的安全性中引入了更多概念。

传统方式中，如果代码需要打开一个文件，那么它会用一个路径名字符串调用 `open`。
然后操作系统检验该代码是否有权限（基于启动该程序的用户）。

对于 WASI，调用一个需要访问文件的函数必须传入一个附加了权限的文件描述符。
可以是用于该文件自身的描述符，也可以是用于包含该文件的目录的描述符。

这样，就不会有随机请求打开 `/etc/passwd` 的代码。
相反，代码只能对传给它的目录进行操作。

![Two evil apps in sandboxes. The one on the left is using POSIX and succeeds at opening a file it shouldn't have access to. The other is using WASI and can't open the file.](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-06-openat-path-1.png)

这让为沙箱中的代码安全地提供更多不同系统调用的访问控制成为可能——因为这些系统调用的能力是受限的。

并且这是发生在逐个模块基础上的。
默认情况下，模块没有对任何文件描述符的访问权限。
但是如果一个模块中的代码拥有文件描述符，那么它可以选择将该文件描述符传给其他模块中它所调用的函数。
也可以创建更受限版本的文件描述符来传给其他函数。

因此运行时可以将应用可用的文件描述符传到顶层代码，然后这些文件描述符就可以按需传播到系统的其余部分。

![The runtime passing a directory to the app, and then then app passing a file to a function](https://2r4s9p1yi1fa2jd7j43zph8r-wpengine.netdna-ssl.com/files/2019/03/05-07-file-perms-1.png)

这让 WebAssembly 更接近最小权限原则——一个模块只能访问完成其工作所需的确切资源。

这些概念来自于能力导向系统（capability-oriented systems），例如 CloudABI 与 Capsicum。
能力导向系统的一个问题是通常很难向它们移植代码。
但我们认为这个问题可以解决。

如果代码已经使用文件相对路径调用 `openat`，那么只要编译该代码就能用。

如果代码使用的是 `open` 并且迁移到 `openat` 风格的前期开销太高，WASI 可以提供一个增量解决方案。
使用 [libpreopen](https://github.com/musec/libpreopen)，可以为应用程序创建一个合理需要访问的文件路径列表。
然后就可以使用 `open` 了，不过只能使用列表中的路径。

### 下一步呢？
我们认为 wasi-core 是一个良好的开端。
它保持了 WebAssembly 的可移植性与安全性，为生态系统提供了坚实的基础。

不过，在 wasi-core 完全标准化之后，我们还需要解决一些问题。这些问题包括：

- 异步 I/O
- 文件监视
- 文件锁定

这只是开始，所以如果你有解决这些问题的想法，就请[加入我们](https://wasi.dev/)吧！

> #### 关于英文原文作者 [Lin Clark](https://twitter.com/linclark)
> Lin 在 Mozilla 的高级开发部门工作，专注于 Rust 与 WebAssembly。
> - [https://twitter.com/linclark](https://twitter.com/linclark)
> - [@linclark](https://twitter.com/linclark)
> 
> [Lin Clark 的更多文章……](https://hacks.mozilla.org/author/lclarkmozilla-com/)
