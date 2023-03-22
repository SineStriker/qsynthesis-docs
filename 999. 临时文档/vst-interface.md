# VST 接口

编辑器需要在一个动态链接库中实现这些函数以供 VST 插件调用。

## 数据类型

### `enum Result`

```c++
enum Result {
    Ok,
    Failed,
}
```

用于表示成功或失败。

~~本来想直接用 `bool` 的，后来想状态可能不是二元的就整了这么个枚举，结果实现到最后发现状态还真就是二元的……~~

### `struct PlaybackParameters`

```c++
struct PlaybackParameters {

    double sampleRate;
    double tempo;

    int64_t projectTimeSamples; //当前位置是第几个采样
    double projectTimeMusic; //当前位置是第几个四分音符
    double barPositionMusic; //当前位置所在小节的小节线位于第几个四分音符处

    int32_t numSamples; //输出块的采样个数

}
```

## 函数列表

### `bool SingletonChecker()`

调用机制：同步。

作用：检查是否已有另一个编辑器正在运行，如果没有返回 `true`，否则返回 `false`。

### `Result Initializer()`

调用机制：同步。

作用：初始化。VST 插件保证在调用本函数前，已经调用 `SingletonChecker` 检查过。

### `Result Terminator()`

调用机制：同步。

作用：在插件终止（例如：用户关闭工程文件）之前触发。编辑器需要在这里释放掉全部资源。

### `void WindowOpener()`

调用机制：异步。

作用：当用户打开 VST 界面，或点击 VST 界面中的横幅时触发。编辑器需要显示窗口或者还原最小化的窗口。请注意：与窗口有关的所有资源需要在初始化时就创建好，这个函数的作用是**显示**窗口而非**创建**窗口。

### `void WindowCloser()`

调用机制：异步。

作用：当用户关闭 VST 界面时触发。用于隐藏编辑器窗口。

### ` Result PlaybackProcessor(const PlaybackParameters *playbackParameters, bool isPlaying, int32_t numOutputs, float *const *outputs)`

调用机制：同步。

作用：处理音频。

参数列表：

| 参数                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `playbackParameters` | 一些参数，详见[定义](#struct PlaybackParameters)。           |
| `isPlaying`          | 当前宿主是否正在回放（也包括录制，缩混）。                   |
| `numOutputs`         | 输出通道的个数。                                             |
| `outputs`            | 一个二阶指针，第一阶的下标为通道索引，第二阶的下标为采样索引。程序需要把生成的音频填入其中。请注意：如果程序此时没有音频需要生成，需要将其全部置为 `0`。 |



备注：

- 由于该函数运行在音频处理线程当中，在处理音频时需要让延迟尽可能低。例如：如果当前需要播放的音频还没有被引擎渲染完毕，那么没有渲染好的部分应该输出空白音频，而不是等待引擎渲染。
- 如果当前宿主并没有正在回放，而编辑器需要播放音频（例如：用户在编辑器中预览），需要编辑器自行使用 `playbackParameters->numSamples` 维护一个递增的计时器。具体的实现可以参考[演示例程](https://github.com/CrSjimo/diffscope-pseudo-editor/blob/master/main.c)。

### `Result StateChangedCallback(uint64_t size, const uint8_t *data)`

调用机制：同步。

作用：当持久化的数据在宿主中被修改时（例如：宿主加载工程文件，用户加载了一个已保存的预设）触发。编辑器需要读取数据并重新加载编辑器中的工程内容。

参数列表：顾名思义。

### `Result StateWillSaveCallback(uint64_t& size, uint8_t*& data)`

调用机制：同步。

作用：当宿主请求存储数据时（例如：保存工程文件，保存预设）触发。编辑器需要自己创建一块空间，把编辑器中的工程内容存储进去，然后把地址返回给 VST。

参数列表：顾名思义。

### `void StateSavedAsyncCallback(uint8_t* dataToFree)`

调用机制：异步。

作用：当宿主存储完数据后触发。编辑器需要释放掉 `dataToFree`。VST 保证 `dataToFree` 是 [`StateWillSaveCallback`](#Result StateWillSaveCallback(uint64_t& size, uint8_t*& data)) 中返回的地址。

### `void DirtySetterBinder(void (*setDirty)(bool state))`

调用机制：异步。

作用：初始化完成后触发。VST 插件传递给编辑器一个 `setDirty` 函数。当编辑器变为脏状态（工程内在上次保存后被编辑）需要调用 `setDirty(true)` 来通知宿主。

备注：`setDirty`**似乎**是线程安全的，还需要进一步测试。

## 关于演示例程

[演示例程](https://github.com/CrSjimo/diffscope-pseudo-editor/blob/master/main.c)实现了下列功能：

- 在初始化时加载 `D:\dstest\test.wav` 与 `D:\dstest\test2.wav`。这两个音频文件必须是 48 kHz 采样率，32-bit深的单声道音频，且头部不应含有标记数据。
- 在触发窗口显示和隐藏事件时弹出对话框。
- 在宿主回放时，向宿主输出 `test.wav` 中的音频。
- 在宿主未进行回放且编辑器窗口打开时，向宿主输出 `test2.wav` 中的音频。
- 把 VST 持久化数据写入 `D:\dstest\state` 文件中，并把 `D:\dstest\state` 文件中的数据写入 VST 持久化储存中。