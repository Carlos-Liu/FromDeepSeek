## 分类
###  1 应用程序崩溃 dump:

WER LocalDumps

DumpType = 0 / 1 / 2

用于 Kaluza.exe 这种进程崩溃


###  2 系统崩溃 dump:

Windows Startup and Recovery / CrashControl

用于蓝屏、系统 bugcheck

包括 Kernel memory dump

## 应用程序 dump file

1. DumpType = 1：Mini Dump

这是小型用户态 dump。

特点：

- 文件较小
- 生成快
- 便于客户上传
- 通常包含线程、调用栈、异常信息、模块列表等
- 不包含完整进程内存

适用场景：

- 初步判断崩溃模块和异常码
- 只需要调用栈定位
- 客户网络/存储受限
- 崩溃频繁，需要轻量收集

局限：

- 很多对象内容、字符串、托管堆、业务数据可能不在里面
- .NET/SOS 深度分析能力有限
- 对复杂内存破坏、剪贴板对象、业务状态定位不够

2. DumpType = 2：Full Dump

这是完整用户态 dump。

特点：

- 包含崩溃进程的大部分/全部用户态内存
- 包含线程栈、模块、异常上下文、托管堆、native heap 等
- 文件通常很大

适用场景：

- 需要深入分析 .NET 托管对象
- 需要看业务对象状态
- 需要查字符串、集合、当前 UI/模型状态
- 内存破坏、native crash、COM/Clipboard 问题
- Mini dump 无法定位根因时

局限：

- 文件很大
- 可能包含敏感信息
- 上传/保存成本高
- 需要匹配 PDB 和调试环境

3. DumpType = 0：Custom Dump

这是自定义 dump。
需要配合 CustomDumpFlags 使用，指定包含哪些内容。

## 设置
```
reg add "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps\Kaluza.exe" /v DumpFolder /t REG_EXPAND_SZ /d "C:\Temp\KaluzaDump" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps\Kaluza.exe" /v DumpType /t REG_DWORD /d 2 /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\Windows Error Reporting\LocalDumps\Kaluza.exe" /v DumpCount /t REG_DWORD /d 10 /f

```
