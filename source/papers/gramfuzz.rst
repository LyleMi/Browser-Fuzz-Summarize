GramFuzz Fuzzing
==================================================

该方法[1-7]的改进点主要有两个。一个在于不直接生成种子，而是从网上获取已经有的HTML文档，并对其进行分析后获得一个比较庞大的语法库，然后使用语法库获得种子。变换方法则更基于语法规则进行变化，而不是随机生成一些值。

尝试把其主要流程写成了如下的伪代码：

主流程伪代码
--------------------------------------------------

::

    webFiles = Crawler.getHTMLfromInternet()
    trainSets = new TrainSets(webFiles)
    initCases = new InitialCases(webFiles, Pocs)
    grammarLibrary = {JsLib, HTMLLib, CSSLib} = new GrammarLibrary(trainSets)
    grammarNode = new GrammarNode(grammarLibrary)
    testCases = new Testcases(grammarNode, mutatePattern(initCases))


Extract the Grammar Node
--------------------------------------------------

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
