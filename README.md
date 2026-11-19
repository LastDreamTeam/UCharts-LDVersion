# UCharts LDVersion

LastDreamTeam 维护的 Unity UGUI 图表，从 [iMemento/UCharts](https://github.com/iMemento/UCharts) 整理而来。在 Canvas 上画雷达图、环形饼图和半环图，源码在 `Assets/UCharts`。

## 先打开演示场景

1. 在 Unity Hub 中将仓库根目录添加为项目，使用 [Unity 2022.3.60f1](ProjectSettings/ProjectVersion.txt) 打开。
2. 在 Project 窗口打开 [DemoScene.unity](Assets/UCharts/DemoScene/DemoScene.unity)，按 Play 看图表展开。要把图表拖入自己的界面，从[预制件目录](Assets/UCharts/Prefabs)选一个，放在 Canvas 下。
3. 选中雷达图，在 Inspector 调整 **Data Color**。数据多边形的填充色和网格的描边色可以分别设置。

下面是上游演示，供识别图表类型。

![雷达图](https://raw.githubusercontent.com/iMemento/UCharts/master/Images/0.gif)
![饼图与半环](https://raw.githubusercontent.com/iMemento/UCharts/master/Images/1.gif)

## 和上游的差别

| | 上游 | 本仓库 |
| --- | --- | --- |
| 工程 | Unity 5.x，目录嵌在 `UCharts/` 下 | **2022.3.60f1**，根目录可直接打开 |
| 雷达数据颜色 | 写死 `new Color32(244, 12, 12, 100)` | Inspector 字段 `m_DataColor`，默认值不变 |
| 脚本编码 | UTF-8 BOM | 已去掉 BOM |
| `.gitignore` | 到 `Library/` 为止 | 补了 `Logs/`、`UserSettings/`、`.vs/`、`.vscode/` |
| 文档与演示素材 | 含原版 README、LICENSE 和本地 GIF | 当前使用本仓 README；演示 GIF 引用上游，仓内未随附上游 LICENSE |

`BarChart` 上下游都是空类，不能当功能用。

## 环境

- Unity **2022.3.60f1**（见 `ProjectSettings/ProjectVersion.txt`）
- 依赖 `com.unity.ugui` 1.0.0，已写在 `Packages/manifest.json`
- 命名空间 `UCharts`
- 组件继承 `MaskableGraphic`，必须放在 `Canvas` 下面

整仓当工程打开，或只把 `Assets/UCharts` 拷进已有工程。已有工程需要 UGUI。编辑器脚本用了 `UnityEditor.UI.GraphicEditor`。

## 目录

```text
Assets/UCharts/
  Scripts/UCharts/   ChartBase、RadarChart、PieChart、HalfPieChart、BarChart
  Editor/            RadarChart 的 Inspector
  Prefabs/           三个预制。半环那个文件名中间有空格
  DemoScene/         DemoScene.unity
  Fonts/Bowlby_One/  演示字体，许可见同目录 OFL.txt
  readme.pdf         上游图解，三页图片，没有可复制的文字
```

## 雷达图

预制：`Assets/UCharts/Prefabs/RadarChart.prefab`。这是旧版 Unity 的二进制预制；可用上面的演示场景检查导入与播放效果。里面的示例轴是十二星座。

物体下必须有一个名为 `Text` 的 `UnityEngine.UI.Text`。`Awake` 里是 `transform.Find("Text").gameObject`，没有这个子物体会直接空引用。播放时按轴复制成 `Text0`、`Text1`……非播放模式不生成标签。

Inspector 由 `RadarChartEditor` 画：

| 字段 | 作用 |
| --- | --- |
| Sides | 边数，3 到 36 |
| Indicators | 每条轴的文字。旁边的 Max 只显示，绘制不用 |
| Data | 每条轴的数值，直接乘半径。`1` 顶到外圈，不是按 Max 归一化 |
| Color0 / Color1 | 背景网格两层交替色 |
| Border Color | 网格线、轴线，以及数据多边形的描边 |
| Data Color | 数据多边形填充。本分支新增，默认 `RGBA(244, 12, 12, 100)` |

播放后约 1 秒展开。按住拖动能旋转整张图。

没有公开的 `SetData`。`m_Data` 和 `m_Indicators` 都是私有序列化字段，运行时改数要自己加接口。

绘制时的下标判断是：

```csharp
var index = i % (vertices - 1);
var value = index < m_Data.Count - 1 ? m_Data[index] : 0;
```

`vertices` 是边数加 1。所以数据条数要比边数多 1，多出来的那条只是占位，否则最后一条轴会被画成 0。5 边就放 6 个数。

## 饼图

`PieChart` 画的是环，不是实心饼。`thickness` 是环边厚度，运行时夹在 `0` 和宽度的一半之间。填充大约在 60% 半径到外径之间，内外圈用 `m_BorderColor`。

```csharp
public struct PieChartDataNode
{
    public string Text;
    public float Value;
}
```

`Text` 目前不会画出来，`DrawValueLines` 是空的。颜色来自 Inspector 里的 `m_Colors`，按段循环。颜色列表也不能是空的，网格生成时会读 `m_Colors[0]`。

```csharp
pie.SetData(new List<PieChartDataNode> {
    new PieChartDataNode { Text = "A", Value = 30 },
    new PieChartDataNode { Text = "B", Value = 70 },
});
```

`SetData` 只换数据，不换颜色。`m_Data` 为空时会读 `m_Data[0]`，至少留一条。`Awake` 会放一段约 1 秒的填充动画，也可以再调 `PlayAnimation()`。

## 半环图

`HalfPieChart` 和饼图同一套画法，角度是 180 度。没有 `SetData`，只能在 Inspector 里配数据和颜色。同样不能空列表。

预制文件名是 `HalfPieChart .prefab`，`Chart` 和 `.prefab` 之间有一个空格。按名字查找时别漏。

## 柱状图

`BarChart` 只有空的 `Start` / `Update`，没有网格。不要挂上去期待出图。

## English — try the charts

Open this repository as a Unity **2022.3.60f1** project, open the [demo scene](Assets/UCharts/DemoScene/DemoScene.unity), then press Play. To reuse a chart, place a radar, pie or half-pie [prefab](Assets/UCharts/Prefabs) under a Canvas in a project with UGUI. The radar fill color is editable in the Inspector. The bar chart is a placeholder.

## 来源

- 上游：[iMemento/UCharts](https://github.com/iMemento/UCharts)，原始代码作者王在京
- 原 Asset Store 页面：[UCharts](https://assetstore.unity.com/packages/tools/gui/ucharts-95073)
- 上游 `LICENSE` 是 MIT，版权声明为 Copyright (c) 2023 PeakCoder。本仓库整理工程时没有带上这个文件
- 字体 Bowlby One 另见 `Assets/UCharts/Fonts/Bowlby_One/OFL.txt`
