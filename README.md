# mmd-npr-render

> 把 MMD（`.pmx`）模型渲成动漫赛璐璐质感的 Blender 三渲二全流程 —— 一个可被 AI 助手直接使用的技能（AgentSkill）。

用一句需求让 AI 助手帮你把 MMD 模型渲成三渲二：硬边明暗、干净描边、贴图完整、肤色不发白、眼睛高光锐利。

## 特性

- Blender 4.2 LTS + EEVEE Next，全程可命令行 / 脚本化
- mmd_tools 正确导入姿势（含「中文路径导致贴图全丢」的坑）
- 卡通材质：`Shader to RGB` + `ColorRamp` 硬边明暗
- 描边：Geometry Nodes 反向壳，支持**逐材质**开关
- 疑难速查：线宽跳变、嘴/眼多余黑线、腿「白袜」、肤色发白、预设打不开……

## 环境要求

- Blender 4.2.x（推荐 4.2 LTS 便携版）
- 扩展 `mmd_tools`（Blender 4.2 扩展仓库：`bl_ext.blender_org.mmd_tools`）

## 安装 / 使用

### 作为 Agent 技能

发布到 ClawHub 后：

```bash
clawhub install mmd-npr-render
```

或直接把本目录放进你的 Agent skills 目录。

### 手动使用

把 `SKILL.md` 交给具备 Blender / 文件操作能力的 AI 助手，或按文中步骤手动操作。

## 目录结构

```
mmd-npr-render/
├── SKILL.md               # 技能正文（Agent 读）
├── README.md
├── CHANGELOG.md
├── LICENSE                # MIT（默认）
└── LICENSE-CC-BY-4.0.txt  # CC BY 4.0 官方法律文本（备选，见下）
```

## 授权

默认 **MIT**（见 `LICENSE`）——最宽松、对含代码的技能最省心。Copyright (c) 2026 SilverlingFox.

如果你想只用 CC BY 4.0 发布（或采用「代码 MIT + 文档 CC BY 4.0」的混合许可），可改用 `LICENSE-CC-BY-4.0.txt`，并同步修改 `SKILL.md` frontmatter 的 `license` 字段。

## 免责声明

- 本项目**不包含**任何 MMD 模型、贴图或渲染预设；文中提及的风格仅为说明。
- MMD 模型普遍「允许改造、禁二次配布 / 禁商用 / 禁 18 禁」，请在模型作者授权范围内使用。
- 文中的数值是实测**起点值**，换模型需要重新校准。

## 致谢

流程来自实际项目的踩坑与调参总结，欢迎 issue / PR 补充新模型的校准经验。
