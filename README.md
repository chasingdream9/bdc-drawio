# bdc-drawio

`bdc-drawio` 是一个用于生成 `draw.io / diagrams.net` 图的本地技能（skill）。

它面向需要用 AI 直接产出流程图、架构图、UML 图、ER 图、脑图、网络拓扑图等场景，核心目标是：

- 让模型直接输出可编辑的 `.drawio` 文件
- 在环境支持时默认导出 `PNG`
- 尽量保证布局规整、节点不重叠、结构清晰

本仓库当前版本主要用于 **BD-C 团队内部测试和迭代**。

## 适用场景

适合以下类型的图：

- 流程图
- 架构图
- UML 时序图 / 类图
- ER 图
- 脑图
- 网络拓扑图

典型触发词包括：

- `画图`
- `流程图`
- `架构图`
- `UML`
- `时序图`
- `ER图`
- `脑图`
- `draw.io`
- `drawio`

## 核心思路

这个 skill 的思路不是“让模型自由发挥画图”，而是给模型一套明确约束：

1. 先识别图类型、节点、关系和输出格式
2. 直接编写 draw.io XML
3. 在保存前做结构和布局自检
4. 先保存 `.drawio` 源文件
5. 默认导出 `PNG`
6. 如果本机缺少 draw.io CLI，则回退为仅交付 `.drawio`

这样做的好处是：

- 输出结果可编辑
- 图形结构更稳定
- 更适合工程化复用
- 更方便继续修改，而不是每次重画

## 仓库结构

```text
bdc-drawio/
├── SKILL.md
└── references/
    └── best-practices.md
```

说明：

- `SKILL.md`：技能主文件，定义触发条件、工作流、交付规则
- `references/best-practices.md`：画图规范、布局建议、架构图模板参考

## 输出规则

默认交付：

- `PNG`

中间源文件：

- `.drawio`

说明：

- `.drawio` 一定会保留，方便继续编辑
- 如果本机安装了 draw.io CLI，则默认导出 PNG
- 如果环境里没有 draw.io CLI，skill 会回退为只输出 `.drawio`

## 环境要求

如果希望直接导出 `PNG`，本机需要安装 draw.io 桌面版 CLI。

常见安装方式：

- macOS：`brew install --cask drawio`
- Windows：`winget install JGraph.Draw`
- Linux：`snap install drawio`

也可以从官方发布页下载：

<https://github.com/jgraph/drawio-desktop/releases>

## 如何在 Codex 中安装使用

如果你使用的是本地 Codex skills 目录模式，可按下面方式安装：

1. 克隆仓库

```bash
git clone https://github.com/chasingdream9/bdc-drawio.git
```

2. 将仓库目录复制到本地 skills 目录，例如：

```bash
mkdir -p ~/.codex/skills
cp -r bdc-drawio ~/.codex/skills/
```

3. 确保目录结构类似：

```text
~/.codex/skills/bdc-drawio/
├── SKILL.md
└── references/
    └── best-practices.md
```

4. 在对话中直接触发，例如：

- `使用 bdc-drawio 帮我画一个用户注册流程图`
- `用 bdc-drawio 生成系统部署架构图`
- `帮我画一个债券一级发行流程图，输出 png`

## 如何在 OpenCode 或其他工具中使用

不同工具对“skill / prompt package / local agent capability”的加载方式不完全一样，但可以按通用思路接入：

1. 保留整个目录结构，不要只拿 `SKILL.md`
2. 确保工具支持：
   - 本地技能目录
   - 或系统提示词 / 角色包 / prompt bundle
   - 或可引用外部 markdown 规则文件
3. 把 `SKILL.md` 作为主说明文件加载
4. 把 `references/best-practices.md` 作为可选参考文件一并放入

通用接入原则：

- 主规则看 `SKILL.md`
- 细节规范看 `references/best-practices.md`
- 如果工具支持“触发式技能”，直接把整个目录作为一个技能包导入
- 如果工具不支持 skills 机制，也可以把 `SKILL.md` 内容作为 system prompt 或 role prompt 使用

## 推荐提示词

你可以直接这样提需求：

```text
使用 bdc-drawio 生成一个一级债券发行流程图，输出 png，中文标签。
```

```text
使用 bdc-drawio 画一个三层系统架构图，包含前端、API 服务、数据库和监控模块。
```

```text
使用 bdc-drawio 修改现有 drawio 文件，把审批节点改成双人复核。
```

## 当前定位

当前仓库主要定位为：

- BD-C 团队内部测试
- 画图类任务的可复用技能沉淀
- 后续可继续扩展为更完整的团队标准化 draw.io 生成规范

因此当前版本更强调：

- 实用性
- 可维护性
- 本地可跑
- 输出稳定

而不是追求非常复杂的自动化封装。

## 注意事项

1. 该 skill 默认以 `.drawio` 为中间源文件
2. 最终是否能直接产出 `PNG`，取决于本机是否安装 draw.io CLI
3. 如果图较复杂，建议先生成 `.drawio` 再二次微调
4. 如果需要团队统一风格，可以继续扩展 `references/` 里的模板规则

## 后续可扩展方向

- 增加更多图类型模板
- 增加泳道图、组织架构图、数据流图规范
- 增加统一配色和品牌化布局
- 增加团队内部示例图库

## License

当前仓库主要用于 BD-C 团队内部测试和使用。
如需对外共享或二次分发，建议由仓库维护者补充正式许可说明。
