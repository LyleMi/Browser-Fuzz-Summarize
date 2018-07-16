浏览器防护措施
==================================================

Edge
--------------------------------------------------

ACG (Arbitrary Code Guard)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- ACG防止进程生成动态代码或者修改已经存在的可执行文件。这个是用ProcessDynamicCodePolicy来配置的。

- 有两个 W^X 策略
    - 存在的代码页不可写
    - 没有签名的代码页不能被创建
- 使用内存保护的手段来加固内核函数
    - NtProtectVirtualMemory
    - NtAllocateVirtualMemory
    - NtMapViewOfSection
    - 其他

CIG (Code Integrity Guard)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- CIG禁止进程载入没有签名的镜像，使用ProcessSignaturePolicy来配置
- 在创建section的内核函数NtCreateSection中强制
- 使用ProcessImageLoadPolicy防止载入不受信任的镜像

CFG (Control Flow Guard) 强制控制流完整性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- CFG防止程序的控制流被劫持，使用ProcessControlFlowGuardPolicy来配置。
- 每次跳转的目标都是受控的，检查写在ntdll.dll的LdrpValidateUserCallTarget / LdrpDispatchUserCallTarget等函数
- CFG不能保护基于ret的跳转
