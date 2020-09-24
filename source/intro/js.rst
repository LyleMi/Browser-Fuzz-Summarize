JavaScript 引擎
========================================

简介
----------------------------------------
JavaScript引擎是一个专门处理JavaScript脚本的虚拟机，一般会附带在网页浏览器之中。

常见引擎
----------------------------------------
- V8 (Chrome)
- SpiderMonkey (FireFox)
- ChakraCore (Edge)
- JavaScriptCore (Safari)

执行
----------------------------------------
- JavaScript源代码被Parser解析成AST
- AST经Interpreter解析成Bytecode后执行
- 不断执行Bytecode，收集运行时信息，生成优化后的Bytecode
