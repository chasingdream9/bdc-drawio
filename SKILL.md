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
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=14;"
        vertex="1" parent="1">
  <mxGeometry x="100" y="100" width="160" height="60" as="geometry" />
</mxCell>
```

### 连线模板

```xml
<mxCell id="edge-1" value=""
        style="edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;"
        edge="1" parent="1" source="node-1" target="node-2">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

### 文本规则

- 多行文本必须在 `value` 中使用 `&#xa;` 换行
- 不要在标签里写字面量 `\n`
- 所有节点都应保留 `whiteSpace=wrap;html=1;`

## 第三步：布局规则

所有图统一遵守以下规则：

- 坐标尽量使用 `10` 的倍数，保持对齐
- 节点边界之间至少保留 `80px` 间距
- 同类型节点尽量复用相同宽高
- 页面四周至少保留 `60px` 边距
- 任意两个非容器节点不得重叠
- 图过宽或过高时，主动增大 `pageWidth` 或 `pageHeight`
- 优先使用对称、留白均衡的布局

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
"$DRAWIO" -x -f png --scale 2 --border 20 -o output.png diagram.drawio
```

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
