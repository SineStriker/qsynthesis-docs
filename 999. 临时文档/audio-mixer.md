# 混音器的设计

一个简单的混音器，参考了 OpenUTAU 的实现。

## 实时混音器

### 简介

实时混音器是 NAudio 中的接口 `ISampleProvider` 的实现，包含音频格式、音频源、播放指针位置等属性，以及 `Read` 和 `SetPosition` 函数，前者用于将音频源进行混音并填充到缓冲区，后者用于设置播放指针的位置。实时混音器能直接对接到音频后端，以输出实时混合的音频。

### 工作流程
- 用户导入音频或渲染器渲染完毕，得到音频文件（如 wav, mp3, opus 等）
- 解码器将音频文件进行解码，得到音频流 `WaveStream` 对象。
- 用户按下播放键，渲染器开始渲染整个工程，此时渲染器将轨道上的音频片段转换成音频源并进行混合，最后施加音量推子和声像电位器的效果，并返回混合后的音频流。
- 回放管理器 `PlaybackManager` 将混合后的音频流填充到音频后端进行播放，用户就能听到混音后的声音。

### 音频源接口设计

```csharp
/// <summary>
/// 音频信号源接口，可用于实现将音频信号混合到缓冲区的功能，还可以用于信号处理，如实现音量和声像控制。
/// </summary>
interface ISignalSource
{
    /// <summary>
    /// 判断当前区间是否准备好进行混音操作。
    /// </summary>
    /// <param name="position">当前位置</param>
    /// <param name="count">数量</param>
    /// <returns>当前区间是否准备好进行混音操作。</returns>
    bool IsReady(int position, int count);

    /// <summary>
    /// 将当前位置开始的采样数据与给定缓冲区中的数据进行混合。
    /// </summary>
    /// <param name="position">当前位置，即要混合的采样点的起始位置。</param>
    /// <param name="buffer">缓冲区，即要混合的目标数组。</param>
    /// <param name="index">缓冲区中的索引，即要混合的目标数组的起始位置。</param>
    /// <param name="count">数量，即要混合的采样点的个数。</param>
    /// <returns>混合后的结束位置。</returns>
    int Mix(int position, float[] buffer, int index, int count);
}
```

基于音频信号源接口，派生出以下用于实现音频混合和效果处理的信号源接口。

### Wave 音频信号源接口设计（待完善）

```csharp
/// <summary>
/// 单个 Wave 音频信号源，可以是音频片段或者渲染好的演唱片段。
/// </summary>
interface IWaveSource: ISignalSource
{

}
```

### Wave 混音音频信号源接口设计（待完善）

```csharp
/// <summary>
/// 用于实现将多个 Wave 音频信号源混音。
/// </summary>
interface IWaveMix: ISignalSource
{

}
```

### 音量推子音频信号源接口设计（待完善）

```csharp
/// <summary>
/// 音量推子。
/// </summary>
interface IFader: ISignalSource
{

}
```

### 声像电位器音频信号源接口设计（待完善）

```csharp
/// <summary>
/// 声像电位器。可能需要实现多种声像法则。
/// </summary>
interface IPanPot: ISignalSource
{

}
```

### 实时混音器接口设计

```csharp
/// <summary>
/// 实时混音器接口，可用于实现实时混音，还可以跟踪当前的位置和等待状态。
/// </summary>
interface IMasterAdapter : ISampleProvider
{
    /// <summary>
    /// 实现 ISampleProvider 接口的方法。音频格式，包含采样率、声道数等。
    /// </summary>
    // WaveFormat WaveFormat { get; };

    /// <summary>
    /// 表示是否正在等待信号源准备好数据.
    /// </summary>
    bool IsWaiting { get; private set; };

    /// <summary>
    /// 表示已经等待了多少个样本。
    /// </summary>
    int Waited { get; private set; };

    /// <summary>
    /// 实现 ISampleProvider 接口的方法，在此处完成混音。
    /// </summary>
    /// <param name="buffer"></param>
    /// <param name="offset"></param>
    /// <param name="count"></param>
    /// <returns></returns>
    // int Read(float[] buffer, int offset, int count);

    /// <summary>
    /// 设置播放指针的位置。
    /// </summary>
    /// <param name="position">位置。</param>
    void SetPosition(int position);
}
```

## 非实时混音器

### 非实时混音器接口设计

```csharp
/// <summary>
/// 非实时混音器接口，用于导出混音。
/// </summary>
interface IMasterAdapter : ISampleProvider
{
    /// <summary>
    /// 实现 ISampleProvider 接口的方法。音频格式，包含采样率、声道数等。
    /// </summary>
    // WaveFormat WaveFormat { get; };

    /// <summary>
    /// 实现 ISampleProvider 接口的方法，在此处完成混音。
    /// </summary>
    /// <param name="buffer"></param>
    /// <param name="offset"></param>
    /// <param name="count"></param>
    /// <returns></returns>
    // int Read(float[] buffer, int offset, int count);
}
```