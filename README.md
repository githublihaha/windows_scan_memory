# windows搜索内存IOC

## 工具特点
+ go语言编写，静态链接，支持64位、32位环境
+ 采用调试模式，可以搜索一些系统的高权限进程
+ 默认输入字符串参数，会以UTF-16和ASCII模式搜索
+ 支持HEX模式搜索
+ 支持指定PID、指定进程名称(包含)搜索
+ 采用Boyer-Moore搜索算法
+ 多线程扫描
+ 只扫描进程的Private、Image内存区域，不扫描Mapped（Mapped表示通过文件映射分配的内存，可能是共享文件的内容或进程间通信使用的区域，此区域会有很多误报）
+ 终端输出、结果文件输出、日志文件输出
+ 输出：进程名称、PID、内存地址、匹配类型、进程路径、上下文数据等
+ 具有特殊保护属性的内存不扫描：PAGE_GUARD、PAGE_NOCACHE、PAGE_WRITECOMBINE
+ 支持输出匹配到的线程信息（获取线程栈地址范围，匹配地址，并不是所有的都能匹配到），输出线程ID、模块路径、线程栈基址、栈限制地址

## 使用方法
```
使用方法:
  字符串搜索: win_scan_memory.exe [选项] <字符串>
选项:
  --hex        使用十六进制模式搜索
  --pid <pid>  只扫描指定PID的进程
  --name <name> 只扫描名称包含指定字符串的进程（不区分大小写）
示例:
  win_scan_memory.exe www.example1.com
  win_scan_memory.exe --pid 1234 www.example1.com
  win_scan_memory.exe --name note www.example1.com
  win_scan_memory.exe --hex 7777772e6578616d706c65312e636f6d
```

## 示例
#### 使用示例

<img width="376" alt="image" src="https://github.com/user-attachments/assets/b19a8d31-f127-4a59-9a6e-f57e4d93bcb0" />

#### 相同目录下生成日志文件和结果文件

<img width="186" alt="image" src="https://github.com/user-attachments/assets/43442b36-cd4f-4c7d-8816-e2158f5c44a6" />


#### 结果文件

<img width="629" alt="image" src="https://github.com/user-attachments/assets/142247be-2bf5-4422-9cbd-1a03582c57d2" />

#### 包含线程信息的结果

<img width="860" alt="image" src="https://github.com/user-attachments/assets/e5426c58-cdc8-40e2-9495-6f4970817ee0" />


#### 在log日志文件中，可以看到获取到的线程ID、起始地址、栈范围、模块文件路径
<img width="1141" alt="image" src="https://github.com/user-attachments/assets/f948ffeb-1312-4e35-b825-e1c8dc3ddb9b" />

(部分环境下运行程序，终端可能会卡住，多按几下回车就正常了...

#### 结果中多出来的进程
一般扫描结果中，会比意想的多出来几个进程，如下图所示，但是这其实是真实的，并不是程序错误

<img width="632" alt="image" src="https://github.com/user-attachments/assets/58cbf551-a949-45a2-b4a5-ff1bc82286ae" />

如果查看这几个程序的命令行分别是如下内容，查下他们的含义，发现扫描结果其实也是合理的

| **进程/命令**                     | **描述**                                                                                     |
|-----------------------------------|---------------------------------------------------------------------------------------------|
| `svchost.exe -k utcsvc -p`        | 用户体验与遥测服务（Connected User Experiences and Telemetry），用于收集系统诊断数据并上传给 Microsoft 改善用户体验和系统稳定性。 |
| `C:\Windows\System32\dwm.exe`     | 桌面窗口管理器（Desktop Window Manager），负责渲染窗口效果、透明效果、任务栏预览等图形界面功能，依赖 GPU 提升渲染性能。              |
| `\??\C:\Windows\system32\conhost.exe 0x4` | 控制台窗口主机（Console Host），用于管理控制台应用（如 `cmd.exe`、`powershell.exe`）的图形界面和输入输出。            |
