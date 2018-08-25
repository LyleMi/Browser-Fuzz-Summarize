Skyfire Data-Driven Seed Generation for Fuzzing
==================================================

这篇文章主要想解决的问题是语法严格的程序一般有语法解析、语义检查然后执行。想要生成有效语义的case的代价是非常昂贵的。而一般的bug是在执行阶段，这为Fuzz带来了难度。

文章主要的贡献是在AST和CFG的基础上提出了Probabilistic Context Sensitive Grammar (PCSG)。从样本中学习出可以描述语法和语义的PCSG。再使用PCSG生成正确的，多样化的，不常见的Case。

但是这样生成有几个问题，因为一些生成规则是递归的，那么就可以生成很大的样本，因此文中增加了一些启发式的算法。包括：

- Heuristic 1: Favor low-probability production rules
- Heuristic 2: Favor low-frequency production rules
- Heuristic 3: Favor low-complex production rules
- Heuristic 4: Restrict the total number of rule applications

