Fuzz相关论文
==================================================

Scheduled DOM Fuzzing
--------------------------------------------------

该方法[1-1]是基于一些有联系的浏览器fuzz工具和一个名为BFF的fuzz框架结合后优化得来。这种方法对fuzz做出的主要优化是并不以同等的概率选择种子以及种子变换的概率，而是在fuzz的过程中，计算更容易产生crash的种子以及变换方式，从而动态的变化选择种子/变换方式的概率，以期以更高的概率获得crash。该篇文章的实验部分对比了zzuf、crossfuzz等fuzz方式，获得了倍于这些工具的crash，以此认为该方法是有效的。

GramFuzz Fuzzing
--------------------------------------------------

该方法[1-7]的改进点主要有两个。一个在于不直接生成种子，而是从网上获取已经有的html文档，并对其进行分析后获得一个比较庞大的语法库，然后使用语法库获得种子。变换方法则更基于语法规则进行变化，而不是随机生成一些值。

尝试把其主要流程写成了如下的伪代码：

主流程伪代码
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    webFiles = Crawler.getHTMLfromInternet()
    trainSets = new TrainSets(webFiles)
    initCases = new InitialCases(webFiles, Pocs)
    grammarLibrary = {JsLib, HTMLLib, CSSLib} = new GrammarLibrary(trainSets)
    grammarNode = new GrammarNode(grammarLibrary)
    testCases = new Testcases(grammarNode, mutatePattern(initCases))


Extract the Grammar Node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    currentNode = rootNode
    grammarNodeDB = []

    def searchLeafNode(n):
        tmp = new Node()
        tmp.code = n.code
        tmp.type = n.type
        grammarNodeDB.push(tmp)


    def searchChildNode(n):
        searchLeafNode(n)
        tmp = n.nextLeafNode()
        while tmp is not None:
            searchLeafNode(tmp)
            tmp = n.nextLeafNode()

    def main(rootNode):
        searchChildNode(rootNode)

该文章的主要优点在于测试者不需要对DOM树，CSS，或者Js语法有了解，只要使用代码库就可以了。另外，从真实HTML文件中的代码会对接口有更高的覆盖率，而且生成测试样例使用了语法树的方法，给测试例子触发漏洞以更多的可能性。

DOM Level Fuzz
--------------------------------------------------

这是Nduja[3-5]实现的主要的思路。该方法认为基于DOM的Fuzz可以分为三个层次，第一层是常规的随机生成元素，这种思路是最普遍的，所以效率不高。作者认为应该从更高的层次去fuzz。比如基于DOM的第二（基于逻辑）第三层（基于事件）进行fuzz，以期获得更好的效果。

从第二层出发，更多考虑的是基于DOM逻辑进行变换。举例来说，在一棵DOM树上，存在的逻辑有firstChild/lastChild/nextNode/previousNode/nextSiebling/previousSiebling等，而节点则存在detach/attach/nextNode/previousNode等逻辑，可以基于这相关的逻辑进行变换。

第三层考虑的是DOM之间发生变化时相应触发的事件。利用事件之间的逻辑关系去fuzz，比如节点A发生detach时，会触发让该节点attach到节点B的动作，但是又会触发节点删除子节点的动作。用这样一些逻辑上的边界条件去尝试fuzz。

one fuzzing template revealed over 100 IE UAF
--------------------------------------------------

这篇文章出自black hat Europe 2014的一篇讲稿，感觉比较有启发性

作者提出

::

    – Engineers are not good at repairing
    – Engineers make mistakes taking things apart (undoing) 
    – Engineers made mistakes putting things back together (redoing) 

从这里就延伸出去想到工程师可能会犯错的地方，然后fuzzer就从下面这些地方的思路开始构造

- Explicit Pairings
    – Direct: 'on/off', 'true/false', properties.
    - e.g. 

        + ``display="block"/"none"``
        + ``appendChild/removeChild``
        + addEventListener : ``focusin/focusout``
- Implicit Pairings
    - Indirect: inheritance, nullity, state change.
    - e.g.

        + Content: ``innerText=''/ document.write('')``
        + Relation: swap parent/child node
        + Status: ``window.navigate('') / location.reload()``
- Hybrid Pairings
    - Complexity of mixing explicit and implicit.
    - Script (Dynamic) + HTML (Static)

        + ``<body contentEditable='true'>``
        + ``Document.body.contentEditable='false';``
    - Property + Method
- Pairing Combinations
    – Multiple pairings per page.

Fileja
--------------------------------------------------

fileja在nduja的基础上有两个比较大的改进的地方
一个是增加了一些对时间敏感的操作，这个是针对
所有浏览器的

另外，对于IE支持的多脚本引擎的特性，也特别做了fuzz

这里提到的时间敏感的操作主要是说之前的文章对DOM的操作都是同步的
而在这篇文章中引入了xhr和WebSocket

主要有这三种fuzz的行为：

- 同步/异步server返回
- 通过setTimeout 或 setInterval API同时运行多个call
- 任意时间延迟

这样则将导致

- 在一个API调用的中间会触发一次DOM的变化
- ws/xhr的配置还没有改动但是主页面已经跳转了
- race condition
  
predicting vulnerable software components
--------------------------------------------------

这篇文章的主要贡献在于给出了一个预测在大型软件中，某个模块漏洞出现概率的方法。可以通过该方法预测更容易出现漏洞的模块，从而有针对性的进行审计。
该文章使用的主要方法为从https://bugzilla.mozilla.org中获取了firefox历年来的漏洞，使用机器学习的方法对其进行聚类。

个人觉得这个思路是可以借鉴的，如果白盒符号执行的时候路径过多，可以使用该方式进行启发式的路径选择，从而更高效的测试。

