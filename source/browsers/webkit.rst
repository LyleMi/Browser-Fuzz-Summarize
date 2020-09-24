Webkit
================================

简介
--------------------------------
Webkit主要由WebCore和JavaScript Core组成，除此之外，WebKit Embedding API是负责浏览器UI与WebKit进行交互的部分，而WebKit Ports则是让Webkit更加方便的移植到各个操作系统、平台上，提供的一些调用Native Library的接口。

JavaScript Core
--------------------------------
JavaScript Core的优化执行分为四个部分，LLInt、Baseline、DFG、FTL。
LLInt是最开始的解释执行部分，Baseline是暂时的JIT，DFG阶段开始做一定的优化，FTL阶段做了充分的优化。

编译
--------------------------------
::

    apt update
    apt install flatpak libseccomp-dev
    git clone https://github.com/projectatomic/bubblewrap
    ./autogen.sh
    ./configure
    make
    make insatll
    git clone --depth=1 git://git.webkit.org/WebKit.git
    cd WebKit
    echo Y | ./Tools/gtk/install-dependencies
    ./Tools/Scripts/update-webkitgtk-libs
    ./Tools/Scripts/build-webkit --release --gtk --cmakeargs=-DCMAKE_CXX_FLAGS="-fsanitize=address -fno-omit-frame-pointer -g -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++"
    add-apt-repository ppa:alexlarsson/flatpak
    ./Tools/Scripts/run-minibrowser --gtk

参考链接
--------------------------------
- `WEBKIT渲染不可不知的这四棵树 <https://mp.weixin.qq.com/s?__biz=MzI0ODA2ODU2NQ==&mid=2651130703&idx=1&sn=3a1aa8a892a1fd1e03a0fa45f25b05ef>`_
