Chrome
==================================================

进程架构
--------------------------------------------------
Chrome涉及到多个进程，在各个进程间都设置了不同的完整性程度

+ 主进程
    + 主框架部分，管理子进程
    + 管理地址栏、书签、前进/后退按钮等
    + 处理浏览器的不可见任务，如网络请求和本地文件访问等
    + Medium Integrity Level
+ 渲染进程
    + 渲染和处理获取到的Web内容
    + Untrusted Integrity Level
+ GPU进程
    + 和显卡交互
    + Low Integrity Level
+ 扩展进程
    + 运行扩展代码
    + Untrusted Integrity Level
+ 插件进程
    + 运行插件代码
    + Untrusted Integrity Level
+ Utility进程
    + 运行特定任务
    + Untrusted Integrity Level

JavaScript引擎
--------------------------------------------------
V8是Chrome的JavaScript语言处理程序（VM）。其引擎由TurboFan、Ignition和Liftoff组成。其中Turbofan是其优化编译器，Ignition则是其解释器，Liftoff是WebAssembly的代码生成器。

渲染与IPC
--------------------------------------------------
在Chrome中，把进程分隔开来，来保护整个进程不受渲染引擎的一些bug干扰。也限制了渲染引擎对整个系统的访问。总的来说，做了内存保护和访问控制的工作。

Chrome用了blink，一个开源的layout engine来解析HTML的layout。

|framework|

渲染进程管理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
每个渲染进程都有一个对应的全局RenderProcess对象管理和浏览器进程之间的通信，并保存一些全局状态。浏览器也相应的保存了一个RenderProcessHost来管理浏览状态并与渲染进程通信。他们之间通信使用了Chromium的IPC system。

视图管理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
每一个渲染进程都有由RenderProcess管理的和浏览器的标签页对应的一个或多个RenderView对象。相应的RenderProcessHost都有一个RenderViewHost和正在渲染的view相对应。

每个view都有一个ID，用来唯一标识。ID在渲染进程中是唯一的，但是在浏览器中是不唯一，所以确定一个view需要view ID和RenderProcessHost。

浏览器和特定标签页之间的通信是通过RenderViewHost对象完成的，RenderViewHost知道如何通过RenderProcessHost传递信息给RenderProcess和RenderView

模块和接口
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在渲染进程中:
+ RenderProcess保持着和相应RenderProcessHost之间的IPC通信。每一个渲染进程都有一个RenderProcess对象。这是浏览器和渲染模块通信的细节。
+ RenderView和RenderViewHost、webkit的嵌入层通信。这个对象表示选项卡或弹出窗口中的一个网页的内容

在浏览器进程中:
+ Browser对象代表了最高level的浏览器窗口
+ RenderProcessHost对象表示浏览器端的单个浏览器<-->渲染器的IPC连接。 在每个渲染过程的浏览器进程中有一个RenderProcessHost。
+ RenderViewHost对象封装了与远程RenderView的通信，RenderWidgetHost在浏览器中处理RenderWidget的输入和渲染

共享渲染过程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
通常，每个新的窗口或选项卡都会在一个新的过程中打开。浏览器将产生一个新的进程，并指示它创建一个单独的RenderView。

有时需要或希望在标签页或窗口之间共享渲染过程。一个Web应用程序打开一个新的窗口，它期望同步进行通信，例如在JavaScript中使用window.open。在这种情况下，当我们创建一个新的窗口或选项卡时，我们需要重新使用窗口打开的进程。如果进程总数太大，或者用户已经有进程打开导航到该域，那么我们还有策略将新的选项卡分配给现有进程。这些策略在过程模型中有所描述。

检测渲染器的Crash或者错误行为
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
每个IPC连接到一个浏览器进程监视进程句柄。如果这些句柄发出信号，则渲染过程已经崩溃，并且通知了标签页。现在，我们显示一个“sad tab”屏幕，通知用户渲染器已经崩溃。可以通过按重新加载按钮或启动新的导航来重新加载页面。当这种情况发生时，我们注意到没有进程并创建一个新进程。

沙盒渲染器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
鉴于渲染器在单独的进程中运行，我们有机会通过沙箱限制其对系统资源的访问。例如，我们可以确保渲染器对网络的唯一访问是通过其父浏览器进程。同样，我们可以使用操作系统的内置权限来限制对文件系统的访问。

除了限制渲染器访问文件系统和网络之外，我们还可以对其访问用户的显示和相关对象设置限制。我们在单独的Windows“桌面”上运行每个渲染过程，该用户不可见。这样可以防止受影响的渲染器打开新窗口或捕获击键。

内存优化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
给定的渲染器在单独的进程中运行，将隐藏的选项卡视为较低优先级变得很简单。通常，Windows上的最小化进程将其内存自动放入“可用内存”池中。在内存不足的情况下，Windows会将内存交换到磁盘，然后才能更换优先级的内存，从而有助于保持用户可见程序的响应。我们可以将这个相同的原则应用于隐藏的标签当渲染过程没有顶级选项卡时，我们可以释放该进程的“工作集”大小作为系统的提示，以便在必要时将该内存交换到磁盘。因为我们发现，当用户在两个选项卡之间切换时，减小工作集大小也会降低标签切换性能，我们逐渐释放此内存。这意味着如果用户切换回最近使用的选项卡，则该选项卡的内存比较少使用的选项卡更有可能被分页。具有足够内存来运行所有程序的用户根本不会注意到这个过程：Windows只有在需要时才会实际收回这样的数据，所以当内存充足时，没有任何性能下降。

这有助于我们在低内存情况下获得更优化的内存占用。与很少使用的背景选项卡相关联的内存可以完全交换出来，而前景选项卡的数据可以完全加载到内存中。相比之下，单进程浏览器将使所有选项卡的数据随机分布在其内存中，并且不可能如此干净地分离已使用和未使用的数据，从而浪费内存和性能。

编译
--------------------------------------------------
::

    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    export PATH="$PATH:/path/to/depot_tools"
    mkdir ~/chromium && cd ~/chromium
    fetch --nohooks chromium
    cd src
    ./build/install-build-deps.sh
    gclient runhooks
    gn gen out/Default

常用命令
--------------------------------------------------
- ``chrome://about`` 查看所有命令
- ``chrome://crashes`` 主动Crash
- ``chrome://version`` 查看版本
- ``chrome://flags`` 查看或关闭chrome的特性
- ``chrome://dns`` DNS状态
- ``chrome://net-internals`` 网络相关信息
- ``chrome://quota-internals`` 磁盘相关信息

.. |framework| image:: ../images/chrome-frame.png
