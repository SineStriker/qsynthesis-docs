# 日志工具

应用在各种不同的环境中运行，一些不符合预期的行为可能无法在开发者的设备上复现。通过应用的日志可以帮助定位和解决问题。日志工具以 DiffScope 插件提供。

## 日志的存储

应用每次启动都创建新的日志文件，所有插件的日志存储在同一日志文件内。推荐保存在 AppData 下，文件名包含应用启动的日期和时间。命名示例：
```
log_2023_06_30_11_45_14_191.txt
```

## 日志的级别

| 级别 | 缩写 | 说明 | 优先级 |
| --- | --- | ---| --- |
| Debug | D | 调式信息 | 最低 |
| Info | I | 一般信息 | 较低 |
| Warning | W | 警告信息 | 较高 |
| Error | E | 错误信息 | 最高 |

## 日志的标签（TAG）
一个插件的功能可能会非常复杂，为了方便区分不同功能模块的消息，这里引入了类似于 Android 日志系统的 TAG。如果想在某个类输出一条日志，推荐填入类名作为标签。也可以使用任何便于区分的名称。


## 环境日志消息格式

在应用开始运行时，调用日志工具输出一份包含系统版本、插件列表等环境信息的日志消息。

操作系统和硬件的日志消息格式如下：
```
OS Name: 操作系统名称
OS Type: 系统类型
OS Version: 版本
CPU: 处理器名称
GPU: 所有显卡名称
Total Memory: 物理内存大小
```

示例：
```
OS Name: Microsoft Windows 11 Professional 
OS Type: x64
OS Version: 10.0.25393
CPU: Intel Core i9-13900K
GPU: Intel UHD Graphics 770, NVIDIA RTX 4090
Total Memory: 32.00 GiB
```

编辑器和插件的日志消息格式如下：
```
Name        Version
插件 A      插件 A 的版本
...         ...
```

示例：
```
Name        Version
DiffScope   v1.1.4
CorePlugin  v1.2.0
IEMgr       v1.0.0
ScriptMgr   v1.3.2
```

## 普通日志消息格式

日志的消息格式如下：

```
日期 时间 插件/缩写级别/标签: 消息
```

示例：
```
2023-06-30 11:45:14.191 UpdateChecker/E/Api: Failed to fetch xxx
2023-06-30 12:19:19:810 PitchExtractor/I/pyin: sonic-annotator.exe exited with code 0.
```