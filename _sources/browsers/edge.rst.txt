Edge
==================================================

进程架构
--------------------------------------------------
在较早的版本，Edge由 ``MicrosoftEdge.exe`` ``MicrosoftEdgeCP.exe`` 两个进程组成。而后Edge使用UWP架构，之后其进程主要由 ``RuntimeBroker.exe`` 和其他进程组成，其中 ``RuntimeBroker.exe`` 为中等完整性级别，其他进程都在 ``AppContainer`` 中以低完整性级别运行。

当前版本的Edge运行的进程如下：

- MicrosoftEdge.exe
    - mediumintegrity
    - 主进程
- MicrosoftEdgeCP.exe (Web)
    - AppContainer
    - Web内容处理
- MicrosoftEdgeCP.exe (UI)
    - AppContainer
    - 新Tab的内容处理
- MicrosoftEdgeCP.exe (Extensions)
    - AppContainer
    - 扩展处理
- MicrosoftEdgeCP.exe(Settings)
    - AppContainer
    - 设置相关处理
- MicrosoftEdgeCP.exe(Flash)
    - AppContainer
    - Flash处理
- MicrosoftEdgeCP.exe(OOPJS)
    - AppContainer
    - JIT编译处理
- browser_broker.exe
    - mediumintegrity
    - Broker进程
- RuntimeBroker.exe
    - mediumintegrity
    - 权限管理

完整性级别
--------------------------------------------------
- 高等完整性级别是管理员权限
- 中等完整性级别是普通用户运行进程的权限
- 低完整性级别有以下的限制
    - 大多数的Windows消息和进程Hook都被UIPI(User Interface Privilege Isolation)限制
    - 开进程和CreateRemoteThread被mandatory标签禁止
    - 不允许以写权限打开共享内存
    - 使用比其完整性级别高的进程创建的对象
    - 绑定到运行中的Component Object Model服务
- 低完整性级别可以操作
    - 剪贴板
    - Remote Procedure Calls (RPC)
    - Sockets
    - 被更高级别完整性进程设置为允许的Window Message
    - 被更高级别完整性进程设置为允许的共享内存
    - 被更高级别完整性进程设置为允许的COM
    - 被更高级别完整性进程设置为允许的命名管道
- 不信任级别几乎所有涉及到权限的操作都不被允许

AppContainer
--------------------------------------------------
AppContainer是Windows提出的进程隔离机制。通过将应用程序与不需要的资源和其他应用程序隔离，可以最大限度地减少恶意操纵的机会 基于最小权限授予访问权限可防止应用程序和用户访问超出其权限的资源。

其提供了用户凭证、设备、文件、网络、进程等的隔离。
