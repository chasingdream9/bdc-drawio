---
name: bdc-drawio
description: |
  当用户希望创建或修改任何 diagrams.net / draw.io 图时使用本技能：
  包括流程图、架构图、UML 时序图/类图、ER 图、脑图、网络拓扑图，以及其他结构化可视化图。

  触发词包括："画图"、"流程图"、"架构图"、"UML"、"时序图"、"类图"、"ER图"、"脑图"、
  "网络拓扑"、"可视化"、"draw.io"、"drawio"、"diagram"。

  工作流：明确需求 -> 直接编写 draw.io XML -> 自检 -> 保存 .drawio ->
  默认导出 PNG -> 如果无法导出，再回退为仅交付 .drawio。
---

# BDC Draw.io 制图技能

本技能用于根据用户需求生成 draw.io 图。

默认最终交付物是 `PNG`。
`.drawio` 文件是中间可编辑源文件，应始终保留，但除非导出失败，否则它不是主交付物。

默认视觉风格应优先采用“金融风格”：

- 主色：深蓝 / 海军蓝
- 辅色：金色 / 香槟金
- 强调色：墨绿或深青色
- 背景：浅灰蓝或浅米白
- 避免大面积高饱和紫色、荧光色、儿童风配色

## 工作流

1. 理解需求：确认图类型、核心元素、关系、语言和输出格式。
2. 直接编写 draw.io XML。
3. 在保存前重新检查 XML 结构和布局问题。
4. 在用户工作目录中保存 `.drawio` 文件。
5. 默认使用 draw.io CLI 导出 `PNG`。
6. 返回结果时优先给出 `PNG` 路径，再给出 `.drawio` 路径。

## 第一步：理解需求

需要确认：

- 图类型：`flowchart`、`architecture`、`uml-sequence`、`uml-class`、`er`、`mindmap`、`network`
- 核心元素：节点、服务、参与方、实体、角色、模块
- 关系类型：箭头、依赖、数据流、调用链、归属关系
- 输出格式：默认 `PNG`；只有用户明确要求时才使用 `SVG` 或 `PDF`
- 标签语言：默认与用户使用语言保持一致

## 第二步：直接生成 XML

必须直接编写 XML，不要调用其他脚本生成。

需要时再读取 [references/best-practices.md](references/best-practices.md) 中相关部分，不要一次性加载无关内容。

### 基础 XML 结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mxfile host="app.diagrams.net" agent="bdc-drawio-skill" version="21.0.0" type="device">
  <diagram name="DiagramName" id="diagram-1">
    <mxGraphModel dx="1422" dy="762"
                   grid="1" gridSize="10"
                   guides="1" tooltips="1" connect="1"
                   arrows="1" fold="1"
                   page="1" pageScale="1"
                   pageWidth="1600" pageHeight="1200"
                   math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

### 节点模板

```xml
<mxCell id="node-1" value="Label"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#eaf0f6;strokeColor=#1f4e79;fontColor=#0f172a;fontSize=14;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="160" height="60" as="geometry" />
</mxCell>
```

### 连线模板

```xml
<mxCell id="edge-1" value=""
        style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=20;html=1;strokeColor=#1f4e79;strokeWidth=2;endArrow=classic;endFill=1;entryPerimeter=1;exitPerimeter=1;perimeterSpacing=0;exitX=0.5;exitY=1;exitDx=0;exitDy=0;entryX=0.5;entryY=0;entryDx=0;entryDy=0;"
        edge="1" parent="1" source="node-1" target="node-2">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### 文本规则

- 多行文本必须在 `value` 中使用 `&#xa;` 换行
- 不要在标签里写字面量 `\n`
- 所有节点都应保留 `whiteSpace=wrap;html=1;`

### 默认金融配色

除非用户明确指定其他视觉风格，否则优先使用以下配色：

- 普通节点：`fillColor=#eaf0f6; strokeColor=#1f4e79; fontColor=#0f172a`
- 起止节点：`fillColor=#dbe7f3; strokeColor=#153e75; fontColor=#0f172a`
- 决策节点：`fillColor=#f4ecd8; strokeColor=#b38b2d; fontColor=#3b2f0b`
- 容器/分层块：`fillColor=#dce6f1; strokeColor=#335c81; opacity=70`
- 强调/风险/异常节点：`fillColor=#e6f4ea; strokeColor=#2f6b4f; fontColor=#123524`
- 连线：`strokeColor=#1f4e79; strokeWidth=2`

## 第三步：布局规则

所有图统一遵守以下规则：

- 坐标尽量使用 `10` 的倍数，保持对齐
- 节点边界之间至少保留 `80px` 间距
- 同类型节点尽量复用相同宽高
- 页面四周至少保留 `60px` 边距
- 任意两个非容器节点不得重叠
- 图过宽或过高时，主动增大 `pageWidth` 或 `pageHeight`
- 优先使用对称、留白均衡的布局
- 导出前必须检查最外层元素是否都在页面范围内，避免 PNG 被裁切
- 对流程图和网络图，主方向连线优先从节点边缘中点出入，不允许箭头终点穿进框体内部

### 各类图的推荐布局

| 类型 | 方向 | 推荐间距 |
|------|------|----------|
| Flowchart | 自上而下 | 纵向 100px |
| Architecture | 分层块状 | 层间 20px |
| UML Sequence | 自左向右 | 横向 200px |
| UML Class | 网格 / 自上而下 | 纵向 100px，横向 80px |
| ER | 网格 | 横纵各 120px |
| Mindmap | 中心向外 | 每层 150px |
| Network | 分层 | 纵向 100px，横向 120px |

对于架构图，优先采用 `references/best-practices.md` 中的分层块状风格。

### 连线锚点规则

- 上下关系默认使用：
  - `exitX=0.5; exitY=1`
  - `entryX=0.5; entryY=0`
- 左右关系默认使用：
  - `exitX=1; exitY=0.5`
  - `entryX=0; entryY=0.5`
- 不要省略 `entryX/entryY` 和 `exitX/exitY`
- 不要依赖 draw.io 自动吸附去决定终点位置，必须显式写锚点
- 连线默认增加：`entryPerimeter=1; exitPerimeter=1; perimeterSpacing=0`
- 如有折线回路，应使用 `mxPoint` 控制转折点，避免连线贴边穿框
- 箭头末端必须落在目标节点边界，不得落入节点内部

### 流程图强模板规则

流程图不只是“尽量”规整，而是必须按以下强模板执行：

- 主链路必须垂直向下
- 条件分支优先向右展开，回归主链后再向下
- 回退路径不得直接斜插回主节点，必须使用折线和中间转折点
- 任意两个有连线关系的节点，如果中心线之间存在其他节点，应强制增加 `mxPoint`
- 决策节点下方只保留一个主出口；侧向出口必须从菱形左右边缘发出
- 如果边会贴着节点边缘走，先拉开节点间距，不要靠缩短线段解决
- 对流程图，优先使用以下三种固定模板，而不是临时拼接：
  - 上下直连模板
  - 右侧分支模板
  - 右侧回环模板

上下直连模板：

```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=20;html=1;strokeColor=#1f4e79;strokeWidth=2;endArrow=classic;endFill=1;entryPerimeter=1;exitPerimeter=1;perimeterSpacing=0;exitX=0.5;exitY=1;entryX=0.5;entryY=0;"
```

右侧分支模板：

```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=20;html=1;strokeColor=#1f4e79;strokeWidth=2;endArrow=classic;endFill=1;entryPerimeter=1;exitPerimeter=1;perimeterSpacing=0;exitX=1;exitY=0.5;entryX=0;entryY=0.5;"
```

右侧回环模板：

```xml
style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=20;html=1;strokeColor=#1f4e79;strokeWidth=2;endArrow=classic;endFill=1;entryPerimeter=1;exitPerimeter=1;perimeterSpacing=0;exitX=0.5;exitY=0;entryX=1;entryY=0.5;"
```

并且必须配合：

```xml
<Array as="points">
  <mxPoint x="..." y="..." />
  <mxPoint x="..." y="..." />
</Array>
```

### 架构图强模板规则

架构图必须采用“标题与容器分离”模式，禁止把容器标题直接写进容器本体中。

强制要求：

- 所有大容器默认 `value=""`
- 层标题必须使用单独的标题 cell
- 子分组标题必须使用单独的标题 cell，或使用顶部标题条，不要直接让容器居中显示文字
- 叶子节点才承载业务文字
- 不允许容器在视觉中心出现孤立文字
- 架构图默认优先无箭头；只有在确实需要表达调用关系时才画线
- 如果图是“分层架构图”，标题左对齐或居中置顶，不允许垂直居中浮在容器中部

推荐结构：

- 背景层
- 左侧层标签
- 右侧层容器
- 容器内部标题条
- 标题条下方叶子模块

## 第四步：自检

不要跳过自检。保存前必须重新检查：

- 每个 `id` 是否唯一
- 每个 `mxGeometry` 是否都带有 `as="geometry"`
- 每条边的 `source` 和 `target` 是否都指向存在的节点
- XML 是否闭合完整、结构合法
- 是否存在节点重叠
- 坐标是否基本按网格对齐
- 多行文本是否使用 `&#xa;`
- 字号是否可读，通常不低于 `14`
- 除非图类型明确需要其他样式，否则连线默认使用 `edgeStyle=orthogonalEdgeStyle`
- 每条边是否明确设置了进入点和离开点，箭头是否停在边框而不是穿入框内
- 所有节点、容器、边标签和折线拐点是否都落在页面范围内
- 计算最右侧和最下侧内容边界后，`pageWidth/pageHeight` 是否仍保留至少 `80px` 安全边距
- 流程图中的分支线和回路线是否使用了固定模板，而不是任意自动路由
- 架构图容器是否全部 `value=""`，标题是否已拆到独立标题 cell
- 架构图中是否存在容器中心漂浮文字；如果存在，必须重构为标题条模式

发现问题后，先修正 XML，再保存或导出。

## 第五步：保存和导出

### 保存规则

- 源文件使用小写英文加中划线命名，例如 `order-processing-flow.drawio`
- 文件名中不要出现空格、中文和特殊字符
- 即使最终交付 `PNG`，也始终保留 `.drawio` 源文件

### 检测 draw.io CLI

按以下顺序检查：

```bash
which draw.io 2>/dev/null || which drawio 2>/dev/null
```

如果未命中，再检查常见安装路径。

macOS：

```bash
ls /Applications/draw.io.app/Contents/MacOS/draw.io 2>/dev/null
```

Windows：

```bash
ls "/c/Program Files/draw.io/draw.io.exe" 2>/dev/null || \
ls "$LOCALAPPDATA/Programs/draw.io/draw.io.exe" 2>/dev/null
```

Linux：

```bash
ls /usr/bin/drawio 2>/dev/null || ls /snap/bin/drawio 2>/dev/null
```

### 导出命令

```bash
"$DRAWIO" -x -f png --scale 2 --border 30 --crop -o output.png diagram.drawio
```

导出时的默认原则：

- 优先使用 `--crop`，减少页面设置不当导致的空白或裁切异常
- 导出前先确认 `pageWidth/pageHeight` 足够大，不要完全依赖 `--crop`
- 如果图较复杂、包含回环和侧向分支，优先把页面放大后再导出
- 如果第一次 PNG 仍有内容缺失，应先增大页面尺寸和边距，再重新导出

推荐命名：

- 源文件：`order-processing-flow.drawio`
- 最终图片：`order-processing-flow.png`

如果本机没有安装 draw.io，提示用户安装，但不要自动安装。

可建议：

- macOS：`brew install --cask drawio`
- Windows：`winget install JGraph.Draw`
- Linux：`snap install drawio`
- 通用下载地址：<https://github.com/jgraph/drawio-desktop/releases>

### 输出策略

- 默认最终输出为 `PNG`
- 只有用户明确要求时才使用 `SVG` 或 `PDF`
- 如果成功导出，应把 `PNG` 作为主结果返回
- 如果由于环境缺少 draw.io CLI 无法导出，应保留 `.drawio` 文件，并明确告诉用户 PNG 导出被环境阻塞

## 第六步：交付结果

导出完成后：

- 先返回导出的 `PNG` 文件路径
- 再返回可编辑的 `.drawio` 文件路径
- 明确说明 `PNG` 是默认输出格式
- 告知用户 `.drawio` 文件可用 <https://app.diagrams.net> 打开和继续编辑

## 修改已有图时

当用户要求修改已有图时：

1. 先读取现有 `.drawio` 文件
2. 在原 XML 上修改，不要整张图重建
3. 除非用户要求重构，否则保留原有风格和整体布局
4. 重新执行同样的自检和导出步骤

## 参考文件

使用 [references/best-practices.md](references/best-practices.md) 获取：

- draw.io XML 编写规范
- 常见布局错误
- 分层架构图模板
