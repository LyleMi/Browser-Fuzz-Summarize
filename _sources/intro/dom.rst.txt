DOM
========================================

简介
----------------------------------------
DOM是文档对象模型(Document Object Model)的缩写，标准由 W3C 组织指定。DOM树是把 HTML 文档呈现为带有元素、属性和文本的树结构。

DOM标准
----------------------------------------
DOM标准也在不断在演进，到目前有三个级别。

- DOM Level 1
    - Core: 可表示任何结构化文档的基础性底层接口集合
    - HTML: 基于Core接口，提供更适合的HTML的高层接口
- DOM Level 2
    - Core: 扩展Level 1 Core
    - Views: 程序或脚本动态访问和更新文档表示的内容
    - Events: 程序和脚本可使用的通用事件系统
    - Styles: 程序或脚本动态访问和更新样式数据的内容
    - Traversal and Range: 程序或脚本动态遍历或标识文档中的一段内容
    - HTML: 程序或脚本动态访问和更新HTML文档的结构和内容
- DOM Level 3
    - Core: 扩展Level 1 和 2 的 Core 
    - Load and Save：程序或脚本动态将XML文档加载到DOM文档中，将DOM序列化为XML
    - Validation： 程序或脚本在保证文档合法的基础上动态更新文档的内容
    - Events： 扩展Level 2 Events，主要增加键盘事件
    - Xpath: 使用Xpath访问DOM树
