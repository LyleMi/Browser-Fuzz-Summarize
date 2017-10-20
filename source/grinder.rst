Grinder
==================================================

客户端配置
--------------------------------------------------

- 32位系统
    - ``.\grinder\node\data\x86\grinder_logger.dll`` 复制到 ``c:\windows\system32\``
- 64位系统 
    - ``.\grinder\node\data\x86\grinder_logger.dll`` 复制到 ``c:\windows\syswow64\``
    - ``.\grinder\node\data\x64\grinder_logger.dll`` 复制到 ``c:\windows\system32\``
- 创建保存符号文件的目录 ``c:\symbols\``
- 编辑``config.rb``
- ``ruby grinder.rb [--config=c:\path\to\alternative\config.rb] --browser=BROWSER``
- ps: **目前grinder只支持ruby 1.9-2.0，并不支持高于2.0的版本**


Node
--------------------------------------------------

主要文件说明
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- browser: hook相应浏览器的parseFloat

    + chrome.rb
    + firefox.rb
    + internetexplore.rb
    + safari.rb
      
- core: 核心代码

    + debug

        * debugger.rb 浏览器所用到的debugger基类，继承自Metasm
        * logger.rb 记录日志
    + configuration.rb 配置需要的变量，检查运行环境
    + crypt.rb openssl 加解密库函数
    + debugger.rb
    + logging.rb 控制台输出
    + server.rb 运行web服务器，浏览器访问该服务器以获得样本
    + webstats.rb
    + xmlcrashlog.rb 用xml的格式记录crashlog

- crashes：生成的crash样本存放的文件夹
- data: 测试用的图片/pdf/js等文件，以及需要用到的dll

    + continue.exe => 运行在后台，点击因为crash生成的弹窗
    + logging.js
- fuzzer：用户实现自己fuzzer的位置
- lib： grinder用到的三方库主要是metasm，这是一个ruby的用来做二进制的库
- source

    + continue.exe 源代码
    + logger.dll 源代码
- config.rb 配置文件，若一台电脑运行多个节点时，需要多个配置文件
- crypto.rb 生成密钥，加密生成的crash
- grinder.rb 主文件
- reduction.rb 
- testcase.rb 从日志中恢复样本文件

运行流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

入口文件为grinder.rb

- 初始化过程
    - 初始化需要fuzz的浏览器
    - 初始化环境相关变量
    - 检查dll加载情况
- 运行过程    
    - 检查web server状态，若停止或未启动，则启动
    - 启动浏览器相应的debugger

debugger运行过程

- 初始化logger
- hook heap
- 运行目标浏览器


Server
--------------------------------------------------

服务器端采用典型的PHP+MySQL的结构，设计思路是比较老的非MVC结构，总体功能比较单一，主要用来查看fuzzer节点的情况

部分文件功能说明如下

- account.php 用户账户管理相关
- crash.php 管理crash
- crashes.php 管理crash
- fuzzers.php fuzz节点状态
- install.php 数据库初始化

metasm
--------------------------------------------------

Metasm是一个ruby实现的汇编器、反编汇器以及编译器。

Fuzzer编写
--------------------------------------------------

库函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
grinder提供的库函数主要有下面这些

- ``rand( x )`` Pick a random number between 0 and X
- ``rand_bool()`` Pick either true or false
- ``rand_item( arr )`` Pick an item from an array
- ``tickle(obj)`` 触发一个obj的所有属性
- ``LOGGER( name )`` Logger类的构造函数

SimpleExample
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
SimpleExample.html 是grinder给出来的示例代码，展示了Fuzzer所需要的基本元素
编写logger必须的代码如下

::

    var logger = new LOGGER( "SimpleExample" );
    logger.starting();
    /* other code here */
    logger.finished();          
    window.location.href = window.location.protocol + '//' + window.location.host + '/grinder';

上面部分代码主要用于生成/初始化logger，以及完成fuzz之后回到grinder server

fuzzer主要的代码主要还是生成随机元素的部分，继续以SimpleExample.html为例

::

    // 定义将要fuzz的元素
    var elements = [ 'div', 'input', 'textarea', 'a', 'img', 'form' ];

    // 生成相关元素
    var elementA  = rand_item( elements );
    var elementB  = rand_item( elements );

    // 创建元素并记录，注意因为这里的fuzzer样本是随机生成的，所以需要详细的记录每一步的动作以保证可以复现
    logger.log( "id_0 = document.createElement( '" + elementA + "' );", "grind", 1 );
    var id_0 = document.createElement( elementA );

    logger.log( "document.body.appendChild( id_0 );", "grind", 1 );
    document.body.appendChild( id_0 ); 

    // 在随后的代码对之前的元素进行变化，主要是对其属性值进行变换、调用、对换等操作
    /* ... */

编写
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在简单看过相关的库函数和一个简单的样本之后，编写fuzzer的基础知识就已经足够了。
其必要的逻辑是在生成/修改等操作前后都调用log记录下足够的信息以便调试crash。另外就主要是设计好fuzzer种子生成的算法以及随机元素变换的算法以获取更多的crash。


nduja
--------------------------------------------------

SimpleExample.html更多的只是一个简单的样本，一个高效的fuzzer编写需要更精巧的思路，所以选择nduja进一步进行分析

其简单的函数调用树如下

- step1
    + buildElementsTree1
        * createStyleSheet
        * createMultipleElement
            - genRandomElement
            - appendToRandomElement
            - initialize => add multiply random listener
            - tweakattributes1 => set attr random vaule
            - tweakstyle1 => set random style sheet
    + initialize
    + boom
        - createNodeIterator
            + ElementChecker
                * Random - case 1
                    - createElemRange1
                    - deleteRange1
                * Random - case 2
                    - random skip / accept
        - createTreeWalker
            + ElementChecker
        - createTagAggregation1
        - createElemRange
        - createTxtRange
        - alterRange => random action
            + deleteContents
            + collapse
            + extractContents
            + surroundContents
            + selectNode
            + selectNodeContents
            + insertNode
            + setStartAfter
            + setStartBefore
            + setEndAfter
            + setEndBefore
        - spray
        - moveIterator
            + random
                * nextNode
                * previousNode
            + fuzzAttributes
        - moveTreeWalker
            + random
                * nextNode
                * previousNode
                * previousSibling
                * nextSibling
                * firstChild
                * lastChild
                * parentNode
            + fuzzAttributes
        - tagCrawler1
            + fuzzAttributes

可以看到，正和之前文章讲述的原因，nduja的fuzz逻辑是比较复杂的，而其fuzz的主要操作是moveIterator，moveTreeWalker。即利用节点间的联系来变换，以期找出漏洞。

fileja
--------------------------------------------------

在之前提到了fileja也是基于grinder编写的fuzzer，相对于nduja，在grinder的基础上做了更多的扩展，其流程更复杂

fileja除了自己编写了testcase之外，也使用了来自于W3C的testcase

fileja的在函数执行之前，定义了比较多的全局变量和常量，对比较多的html元素进行了fuzz，这样能获得的crash也会比较多，定义的主要的值有下面这些

- 全局参数
    + 元素数量
    + Listener数量
    + 是否使用svg
    + ...
- 全局变量
    + websocket
    + xhr
    + ...
- 全局常量
    + object_blacklist
    + interesting_vals
    + commands
    + events
    + mutationEvents
    + HTMLElements
    + MMLElements
    + SVGElements
    + standardAttributes
    + SVGCommonAttributes
    + SVGAttributes
    + ...


在定义相关的值之后，进行了fuzz，这里fuzz就更多的针对xhr，websocket和html事件，在fuzzer中各种同步和异步的事件分别触发来进行尝试

+ getList - get test case list from server
    * buildDocumentTree
        - random
            + buildFromFile
        - go
            * buildDOM
            * initFrame
                - initF => create meta, add script
            * initXhr => add random listener to xhr
            * initWs => add listener to xhr
            * initXhrJS => add random listener to xhr
            * createSupportStructures
                - createNodeIterator
                - createTreeWalker
            * createRanges
            * cycle
                - tick
                - startFuzzing
                    + crawl_properties
                    + call_methods
                    + mutateDOM
                    + CollectGarbage
            * reload