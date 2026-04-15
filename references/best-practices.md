# Draw.io XML 最佳实践

## 通用规则

### ID 管理

- 使用有含义的 ID，例如 `start`、`process-1`、`decision-quote`、`edge-api`
- 整个文件中的 ID 必须全局唯一
- 推荐格式：`{type}-{name}` 或 `{type}-{number}`
- `0` 和 `1` 是保留给根节点的 ID

### 样式要点

- 所有节点都应包含 `whiteSpace=wrap;html=1;`，保证文本自动换行
- 默认使用 `fontSize=14;` 或更大字号
- 需要加粗标题时可用 `fontStyle=1`
- 强调边框时可用 `strokeWidth=2;`
- 对大多数图，连线默认使用 `edgeStyle=orthogonalEdgeStyle;rounded=1;`
- 连线应尽量显式设置 `entryX/entryY` 与 `exitX/exitY`，避免箭头插入框体内部

### 金融风格默认配色

除非用户明确指定其他视觉方向，默认采用更偏金融风格的低饱和稳重配色：

- 主节点：`fillColor=#eaf0f6; strokeColor=#1f4e79; fontColor=#0f172a`
- 起止节点：`fillColor=#dbe7f3; strokeColor=#153e75; fontColor=#0f172a`
- 决策节点：`fillColor=#f4ecd8; strokeColor=#b38b2d; fontColor=#3b2f0b`
- 容器：`fillColor=#dce6f1; strokeColor=#335c81`
- 风险/异常/提示节点：`fillColor=#e6f4ea; strokeColor=#2f6b4f; fontColor=#123524`
- 连线：`strokeColor=#1f4e79; strokeWidth=2`

避免：

- 高饱和紫色
- 霓虹色和荧光色
- 过于卡通化的红黄蓝大色块
- 同一张图中出现过多互相竞争的颜色

### 文本与换行

- 多行文本要在 `value` 中使用 `&#xa;`
- 不要写字面量 `\n`

### 布局规则

- 坐标尽量对齐到 `10` 的倍数
- 同一行或同一列的同级元素，除非内容特殊，否则应保持等宽、等高、等间距
- 外部边距应与内部间距大致协调，不要出现一边留白特别大
- 最后一行未铺满时，应居中，而不是挤在左边留下大块空白
- 容器大小应贴合内容加均衡内边距，不要留出大量无意义空白
- 导出前检查所有元素的最外边界，确保页面右侧和底部仍有安全边距，避免 PNG 导出缺图

### 连线锚点规则

- 上下连接优先使用：
  - `exitX=0.5; exitY=1`
  - `entryX=0.5; entryY=0`
- 左右连接优先使用：
  - `exitX=1; exitY=0.5`
  - `entryX=0; entryY=0.5`
- 回路、支线、跨层连接应增加折线控制点，不要让线段直接贴着节点边缘乱拐
- 箭头终点必须停在目标框体边界，不要进入目标框内部
- 如果节点较密，先拉开节点间距，再调锚点和折线

### 常见错误

1. ID 重复
2. `parent` 指错
3. 缺少 `as="geometry"`
4. 节点重叠
5. 忘记写 `whiteSpace=wrap;html=1;`
6. 边引用了不存在的节点 ID
7. 页面尺寸过小
8. 标签里直接写 `\n`
9. 留白严重失衡
10. 箭头末端进入框体内部
11. 元素超出页面，导致 PNG 导出不完整

## 架构图模板

架构图默认推荐使用“分层块状”风格。

### 设计原则

1. 使用浅色背景，例如 `#f5f5f5`
2. 左侧放分层标签列，颜色优先使用深蓝金融风格
3. 右侧放半透明层容器，避免高饱和亮色
4. 一层中如果内容较多，可继续拆成子分组容器
5. 最终叶子节点使用白底、浅边框
6. 横跨多层的能力，例如安全、监控，可放在最右侧边栏
7. 如果空间分组已经足够表达结构，优先不画箭头
8. 整体布局应紧凑、对称、留白均衡

### 布局常量

```text
MARGIN = 40
LABEL_X = 60
LABEL_W = 100
CONTAINER_X = 180
CONTAINER_W = 790
SIDEBAR_X = 990
SIDEBAR_W = 110
LAYER_GAP = 20
ITEM_GAP_H = 10
ITEM_GAP_V = 10
ITEM_H = 35
```

### 背景示例

```xml
<mxCell id="background" value=""
        style="rounded=0;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=none;"
        vertex="1" parent="1">
  <mxGeometry x="40" y="40" width="1080" height="700" as="geometry" />
</mxCell>
```

### 分层标签示例

```xml
<mxCell id="layer-app" value="应用层"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dce6f1;strokeColor=#1f4e79;
               fontSize=18;fontStyle=1;verticalAlign=middle;align=center;"
        vertex="1" parent="1">
  <mxGeometry x="60" y="60" width="100" height="110" as="geometry" />
</mxCell>
```

### 层容器示例

```xml
<mxCell id="app-container" value=""
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dce6f1;strokeColor=#335c81;opacity=70;"
        vertex="1" parent="1">
  <mxGeometry x="180" y="60" width="790" height="110" as="geometry" />
</mxCell>
```

### 叶子节点示例

```xml
<mxCell id="service-api" value="API服务"
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f8fafc;strokeColor=#1f4e79;fontColor=#0f172a;fontSize=14;"
        vertex="1" parent="1">
  <mxGeometry x="200" y="85" width="140" height="60" as="geometry" />
</mxCell>
```

### 右侧横切边栏示例

```xml
<mxCell id="security-frame" value=""
        style="rounded=1;whiteSpace=wrap;html=1;fillColor=#f8cecc;strokeColor=#b85450;
               opacity=30;dashed=1;"
        vertex="1" parent="1">
  <mxGeometry x="990" y="60" width="110" height="660" as="geometry" />
</mxCell>
```
