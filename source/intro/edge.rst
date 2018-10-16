Edge
==================================================

进程架构
--------------------------------------------------
在较早的版本，Edge由 ``MicrosoftEdge.exe`` ``MicrosoftEdgeCP.exe`` 两个进程组成。而后Edge使用UWP架构，之后其进程主要由 ``RuntimeBroker.exe`` 和其他进程组成，其中 ``RuntimeBroker.exe`` 为中等完整性级别，其他进程都在 ``AppContainer`` 中以低完整性级别运行。
