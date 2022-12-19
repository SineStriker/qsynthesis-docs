# 历史记录功能

+ 参考了OpenUtau，在qvogen的基础上作了一些调整

### 数据结构

+ `LVCommand`（虚类）
    + 待定成员

+ `LVCommandList`
    + `List<LVCommand>`
    + 其他自定义成员

+ `ICommandSubscriber`（接口）

+ `CommandManager`
    + `historyStack`（`ArrayList<LVCommandList>`）
    + `historyIndex`（数字）
        + 表示当前状态下，已经执行过的命令数
        + 在没有执行过重做操作时，等于`historyStack`的长度
    + `savedHistoryIndx`（数字）
        + 表示处于已保存状态下，`historyIndex`的值，初始为0
    + `subscribers`（`List<ICommandSubscriber>`）
        + 与命令相关联的对象，主要是`ViewModel`

### 成员函数

+ `ICommandSubscriber`（接口）
    + `Execute(LVCommandList cmds, bool isUndo)`

+ `CommandManager`
    + `Execute(LVCommand cmd, bool isUndo)`
        + 传入一个`LVCommand`，即执行一次操作
        + 调用后，会广播给所有`subscribers`让它们更新
    + `Execute(LVCommandList cmds, bool isUndo)`
        + 传入一个`LVCommandList`，即执行一串操作，其它与上面一致
    + `Undo`
        + 调用后，`historyIndex`减一
    + `Redo`
        + 调用后，`historyIndex`加一
    + `StartRecord`
        + 调用以后，接下来的`Execute`调用传入的命令将会被依次录制，直到`EndRecord`调用为止
    + `EndRecord`
        + 将自从上次`StartRecord`调用后录制下来的所有命令，整合为一个`LVCommandList`，添加到`historyStack`末尾，`historyIndex`加一（加之前如果`savedHistoryIndx`比`historyIndex`大，说明保存点已经被撤销掉了，要将`savedHistoryIndx`设为-1，因为无论怎么撤消重做都不可能恢复到保存点）
    + `AddSubscriber/RemoveSubscriber`

### 操作说明

+ `LVCommand`（虚类）
    + 不同的操作，命令存储的形式不同，因此各种操作（例如音符操作、工程信息操作、音轨操作等）都要继承`LVCommand`，在子类中添加一定的成员变量来保存操作前后变化的信息，并且保存的内容应当是纯数据，不应存在实例引用

+ `CommandManager`
    + 执行保存时，将`savedHistoryIndx`设为`historyIndex`
    + 在用户执行某个操作前，调用`StartRecord`开始录制，用户操作过程中，调用一次或多次`Execute`，用户本次操作结束后，调用`EndRecord`结束录制，这样本次用户操作就会被记录在历史记录中；也可以在用户完成操作后，依次调用`StartRecord`、`Execute`、`EndRecord`来保存历史记录；
    + 系统执行的某些过程应当作为命令，例如加载工程时，进行初始音符的添加，应该调用`Execute(new AddNoteCommand(...))`来实现，与用户操作的区别是不调用`StartRecord`或`EndRecord`

+ `ICommandSubscriber`（接口）
    + 所有与命令相关联的实例必须实现这个接口，该命令的执行或反执行会引起实例的任意成员变量发生变化，例如以下几种
        + `ViewModel`，比如执行了`AddNoteCommand`，`PianoViewModel`中的音符GUI应当更新
        + 局部的信息副本，比如执行了`ChangeTempoCommand`，`RenderBackend`中还存了一份当前的曲速，那么也应当更新
    + 如果`Execute`传入的命令不匹配，例如`PitchSubscriber`接收到一个`ChangeTempoCommand`，那么应当忽略