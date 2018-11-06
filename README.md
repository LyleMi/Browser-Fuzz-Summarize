# 浏览器模糊测试综述

对浏览器模糊测试相关知识、论文、工具进行整理。尚有缺漏或未完成的部分，持续更新中，如有错误请不吝指出。在完成的过程中参考了一些blog、文献，皆在最后给出相应链接，感谢blog作者的分享。

## 索引

- 基础知识
    - 浏览器内核简介
    - JavaScript引擎
    - Chrome
    - Webkit
    - Edge
    - ChakraCore
    - WebIDL
    - WebAssembly
- 相关论文
    - one fuzzing template revealed over 100 IE UAF
    - DOM Level Fuzz
    - Fileja
    - GramFuzz Fuzzing
    - MongoDB’s JavaScript Fuzzer
    - Not all bytes are equal Neural byte sieve for fuzzing
    - predicting vulnerable software components
    - Scheduled DOM Fuzzing
    - Skyfire Data-Driven Seed Generation for Fuzzing
- 浏览器防护措施
    - Edge
- 浏览器漏洞类型
    - 信息泄漏漏洞
    - 内存破坏漏洞
    - 沙箱逃逸漏洞
    - 扩展和插件漏洞
    - UI欺骗
    - 特权域 XSS + 特权域 API
- Fuzz方法
    - 白盒fuzz
    - 黑盒fuzz
    - 灰盒fuzz
- Fuzz实现
    - 综述
    - 随机样本的生成
    - 样本保存
    - 其他样本生成方式
    - 浏览器进程监控
    - 浏览器fuzz缺点
- Grinder
    - 客户端配置
    - Node
    - Server
    - metasm
    - Fuzzer编写
    - nduja
    - fileja
- 参考资料
    - 参考文献
    - 工具
    - Blog or Talks
    - 漏洞相关

### 生成 HTML 格式的文档

```shell
git clone https://github.com/LyleMi/Browser-Fuzz-Summarize.git bfs
pip install sphinx
pip install sphinx-rtd-theme
make html
```

### 反馈

欢迎大家提出各种意见和建议，不胜感激。

反馈邮箱 ``lylemi@126.com``
