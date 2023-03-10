# 插件中心

本程序的插件机制大体参考了Qt Creator的设计，由于Qt Creator过于庞大且采用GPL协议，并且包含一系列文本编辑器的类库，直接照搬并不合适，因此本程序重新实现了一套相似的。

## 插件描述文件

### 主要字段

| 字段名               | 值类型     | 描述                              |
|-------------------|---------|---------------------------------|
| Name              | String  | 插件的显示名字                         |
| ID                | String  | 插件的唯一标识符                        |
| Version           | String  | 版本，需要符合X.Y.Z.W的格式               |
| CompatVersion     | String  | 能向下兼容到的最低版本，默认与Version一致即不兼容    |
| Experimental      | Boolean | 可选，默认为false。如果为true，则宿主不主动加载。   |
| DisabledByDefault | Boolean | 可选，默认为false。如果为true，则宿主不主动加载。   |
| Required          | Boolean | 可选，默认为false。如果为true，则用户不能禁用此插件。 |
| Platform          | String  | 可选，默认为空。值为正则表达式，用以匹配此插件能够运行的平台。 |

### 描述性字段

| 字段名         | 值类型    | 描述     |
|-------------|--------|--------|
| Category    | String | 类别     |
| Author      | String | 作者名    |
| Copyright   | String | 版权信息   |
| License     | String | 许可证    |
| Description | String | 详细描述信息 |
| Url         | String | 相关网页   |

### 依赖信息

| 字段名          | 值类型             | 描述      |
|--------------|-----------------|---------|
| Dependencies | Array Of Object | 此插件的依赖项 |

依赖项的格式如下

| 字段名      | 值类型     | 描述     |
|----------|---------|--------|
| ID       | String  | 依赖项的ID |
| Version  | String  | 依赖项的版本 |
| Required | Boolean | 是否硬依赖  |