# VST

本文介绍了一些通过 vst 插件在宿主中调用 diffscope 的设计思路，以供参考

## 概述

当 VST 插件在宿主中被首次加载时（如：宿主启动，用户添加软音源），需要加载编辑器，加载引擎，并合成工程音频。但是这时不弹出编辑器窗口。

当用户在宿主中打开软音源界面时，将会弹出 VST 插件窗口， VST 插件会再调用程序中的接口显示编辑器窗口，类似 Vocaloid 的实现。

![image-20230310142909067](https://pic.imgdb.cn/item/640ace77f144a010077ca4fb.png)

如果程序以 VST 模式运行，将会：

- 在回放时与宿主的时钟同步。但是，如果宿主没有开始回放，则程序使用自身的时钟控制回放。由于 VST 插件的类别为软音源，所以时钟为小节节拍的形式（这个我也不太明白，还得看看文档）

- 混音器在回放时，不会把音频输出到音频设备，而是输出到宿主。在这种情况下，所有与音频设备相关的设置将不可用。

  ![](https://pic.imgdb.cn/item/640acf0cf144a010077e426d.jpg)

- 程序会从 VST 储存中读写工程文件，这会导致工具栏选项发生改变。

  ![](https://pic.imgdb.cn/item/640acf7bf144a010077f7f3e.jpg)

## VST 与编辑器本体通信的策略

首先，当 VST 被宿主加载时，VST 读取一个全局配置文件，从这个配置文件中找到编辑器的 apploder 动态库位置，设置环境变量 `USE_VST`，并动态加载这个动态库。这个动态库会通知 CorePlugin 以 External 模式运行。插件可以判断程序是 Standalone 模式还是 External 模式，并决定自身的行为。CorePlugin 会加载插件 VstBridge，并由 VstBridge 提供一个用于让 VST 控制编辑器的模块（以下称之为 ControlInterface）。apploader 需要为 VST 插件提供可以访问的接口，来让 VST 插件获取编辑器状态信息以及 ControlInterface 实例的地址等数据。VST 可以通过 ControlInterface 来 toggle 编辑器窗口，让编辑器本体访问 VST 中存储的工程文件，控制回放，获取编辑器合成的音频波形等功能。
