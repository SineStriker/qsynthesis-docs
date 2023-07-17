# DSPX 格式简介

---

## 根区域

```json
{
    "version": "1.0.0",
    "content": {
        ...
    },
    "workspace": {
        ...
    }
}
```
+ `version`：文件版本号
+ `content`：工程可编辑区域，见下
+ `workspace`：应用程序状态信息

## 可编辑区域

```json
{
    "metadata": {
        "name": "New Project",
        "author": "Admin"
    },
    "master": {
        "control": {
            "gain": 0,
            "mute": false
        }
    },
    "timeline": {
        "timeSignatures": [
            ...
        ],
        "tempos": [
            ...
        ],
        "labels": [
            ...
        ]
    },
    "tracks": [
        ...
    ]
}
```
+ `metadata`: 元信息
    + `name`: 工程名
    + `author`: 作者
+ `master`：总线控制
    + `control`: 全局主控
        + `gain`：增益，`double`
        + `mute`：静音
    <!-- + `loop`: 循环区间
        + `enabled`: 是否启用
        + `start`：开始（tick)，`int`
        + `length`：长度（tick)，`int` -->
+ `timeline`：时间轴
    + `timeSignatures`：拍号列表
    + `tempos`：曲速列表
    + `labels`：标签；列表
+ `tracks`：音轨列表

## 拍号

```json
{
    "index": 0,
    "numerator": 4,
    "denominator": 4
}
```
+ `index`：位置（bar），`int`
+ `numerator`：分子，`int`
+ `denominator`：分母，`int`

## 曲速

```json
{
    "pos": 0,
    "value": 120.0
}
```
+ `pos`：位置（tick），`int`
+ `value`：曲速值，`double`

## 标签

```json
{
    "pos": 480,
    "text": "张三"
}
```
+ `pos`：位置（tick），`int`
+ `text`：标签内容

## 音轨

```json
{
    "name": "主音轨",
    "control": {
        "gain": 0,
        "pan": 0,
        "mute": false,
        "solo": false
    },
    "clips": [
        ...
    ],
    "color": {
        ...
    }
}
```

+ `name`：主控
+ `control`: 音轨主控
    + `gain`：增益，`double`
    + `pan`：声像（-1~1），`double`
    + `mute`：静音
    + `solo`：独奏
+ `clips`：音轨区间列表，暂有波形区间、人声区间
+ `color`: 音轨颜色信息
    + 自定义：
        ```json
        "value": "#3B7FFF"
        ```
    + 内置：
        ```json
        "id": "1"
        ```

## 波形区间

```json
{
    "type": "audio",
    "time": {
        "start": 0,
        "length": 0,
        "clipStart": 1000,
        "clipLen": 20000
    },
    "name": "BGM_1",
    "control": {
        "gain": 1,
        "mute": false
    },
    "path": "/path/to/bgm.wav",
}
```

+ `type`：类型，固定值
+ `time`: 时间信息
    + `start`：区间开始（tick），`int`
    + `length`：固定为0，`int`
    + `clipStart`：实际开始，相对于区间开始的长度（tick），`int`
    + `clipLen`：实际长度，相对于实际开始的长度（tick），`int`
+ `name`：音轨名
+ `control`：与主控一致
+ `path`：波形文件所在路径

## 人声区间

```json
{
    "type": "singing",
    "time": {
        "start": 0,
        "length": 300000,
        "clipStart": 1000,
        "clipLen": 20000
    },
    "name": "样式 1",
    "control": {
        "gain": 1,
        "mute": false
    },
    "notes": [
        ...
    ],
    "params": {
        "pitch": {
            "original": [
                ...
            ],
            "edited": [
                ...
            ]
        },
        "energy": {
            "original": [
                ...
            ],
            "edited": [
                ...
            ],
            "envelope": [
                ...
            ]
        },
        ...
    },
    "sources": {
        ...
    }
}
```

+ `type`：类型，固定值
+ `time`: 时间信息
    + `length`：区间全长（tick）
+ `notes`：音符列表
+ `params`：单线条参数
    + `pitchDelta`：音高偏差
        + `original`：自动参数列表
        + `edited`：已修改的参数列表
        + `envelope`: 在实参上的包络（音高参数没有这个属性）
+ `sources`：使用到的模型，不定长

## 音符

```json
{
    "pos": 480,
    "length": 480,
    "keyNum": 60,
    "lyric": "拉",
    "phonemes": {
        "original": [
            ...
        ],
        "edited": [
            ...
        ]
    },
    "vibrato": {
        "start": 0.1,
        "end": 0.8,
        "freq": 1,
        "phase": 0.5,
        "amplitude": 1.1,
        "offset": 0.1,
        "points": [
            {
                "x": 0.2,
                "y": 0.8
            },
            {
                "x": 0.8,
                "y": 1
            }
        ]
    }
}
```

+ `pos`：位置（tick），`int`
+ `length`：长度（tick），`int`
+ `keyNum`：音阶
+ `lyric`：歌词
+ `phonemes`：音素
    `original`：自动参数列表
    `edited`：已修改的参数列表
+ `vibrato`：颤音
    + `start`：开始（音符比例），`double`
    + `end`：结束（音符比例），`double`
    + `freq`：频率（Hz），`double`
    + `phase`：相位（0~1），`double`
    + `amp`：振幅（半音倍率），`double`
    + `offset`：音高偏移（半音倍率），`double`
    + `points`：控制点列表，一般只有两个点
        + `x`：颤音长度比例（0~1），`double`
        + `y`：颤音振幅比例（0~1），`double`

## 音素

```json
{
    "type": "ahead",
    "token": "ang",
    "start": 5
}
```

+ `type`：类型，`ahead`、`normal`或`final`
+ `token`：音素符号
+ `start`：相对当前音符头的开始时间（ms），`int`

## 单线条参数

### 手绘型

```json
{
    "type": "free",
    "start": 100,
    "step": 5,
    "values": [
        114,
        514
    ]
}
```
+ `type`：类型，固定值
+ `start`：开始（tick）,`int`
+ `step`：采样步长，固定为5，`int`
+ `values`：音高坐标（1/100个半音），`int`

### 锚点型

```json
 {
    "type": "anchor",
    "start": 100,
    "nodes": [
        {
            "x": 114,
            "y": 514,
            "interp": "linear"
        },
        {
            "x": 1919,
            "y": 810,
            "interp": "hermite"
        },
        {
            "x": 0,
            "y": 0,
            "interp": null
        }
    ]
}
```

+ `type`：类型，固定值
+ `start`：开始（tick）,`int`
+ `nodes`：参数列表
    + `x`：时间坐标（Tick），`int`
    + `y`：音高坐标（1/100个半音），`int`
    + `interp`：插值方式

## 其他

### 单线条参数种类

+ `pitch`：音高曲线参数，`y`的值为绝对音高，如`C4`为`6000`
+ `energy`: 力度曲线
+ ...