# cfx-compat-configs

[English](./README.md)

[ControlFlex](https://www.curseforge.com/minecraft/mc-mods/control-flex)（CFX）的**社区模组兼容配置**仓库：通过 JSON 告诉 ControlFlex 如何正确处理第三方模组的按键、分发通道、跨阶段按键、界面/悬浮层中的摇杆与光标行为等。

> **⚠ 版本要求：** 本分支（`0.8.7`）的配置面向 **ControlFlex ≥ 0.8.7**，使用当前配置格式（`mod_versions` + `inGameKeys`/`screenKeys`/`overlayKeys` 三段通道模型）。
> ControlFlex 每个发布版本会把内置源固定到本仓库**对应分支**，因此游戏内同步拉到的永远是与你的模组版本兼容的配置。旧格式分支：[`0.8.5`](https://github.com/ControlFlexMC/cfx-compat-configs/tree/0.8.5) · [`0.8.4`](https://github.com/ControlFlexMC/cfx-compat-configs/tree/0.8.4)。

---

## 工作方式

**玩家**无需接触本仓库：ControlFlex 已把它作为内置适配源。打开 **设置 → 模组适配 → 「更多」→「同步最新配置」**（启动后也会自动静默同步一次），在模组详情页按横幅提示选择社区方案 → 预览 → **应用** —— 写入 `user/<mod_id>.json` 并立即生效，无需重启；「重置配置」或删除该文件即可回落内置默认。

手动安装同样有效：把 [`<MC版本>/<加载器>/mods/`](./1.20.1/forge/mods/) 下的配置复制到 `config/controlflex/compat/user/<mod_id>.json` 后重启。ControlFlex 按三层目录加载 compat 配置，优先级 `user` > `mods` > `default`：

- `default/` —— ControlFlex 内置配置（从 JAR 解压，永不覆盖已有文件）
- `mods/` —— 桥接模组（`cfx-compat-*` / 插件模组）安装
- **`user/`** —— 你的自定义配置（最高优先级；「应用」也是写入这里）

旧格式的 `{modid}_keys.json` 会在启动时自动迁移。

## 仓库结构

```text
<仓库根>/
├── 1.20.1/                  # MC 版本（与 SharedConstants 报告的名字完全一致）
│   ├── forge/
│   │   ├── manifest.json    # 由脚本生成的配置清单
│   │   └── mods/
│   │       ├── epicfight.json
│   │       ├── irons_spellbooks.json
│   │       └── ...
│   └── fabric/
│       ├── manifest.json
│       └── mods/
├── 1.21.1/
│   ├── neoforge/
│   └── fabric/
└── ...
```

客户端拉取 `{baseUrl}/{mcVersion}/{loader}/manifest.json`，再按需下载各配置文件并校验 `sha256`。清单由 [`generate_manifest.py`](./generate_manifest.py) 生成。

## 配置格式一览

每个文件（`mods/<mod_id>.json`）声明按**模组版本**匹配的 `mod_versions` 规则集，**首条匹配即生效**。规则集按上下文声明按键 —— `inGameKeys` / `screenKeys` / `overlayKeys` —— 每个按键可配置**分发通道**（`keyMapping`、`eventBus`、`glfwPoll`、`screenInput`、`virtualKbm`），另有 `phasePersistentKeys`、`ignoreKeys`、`itemSuppressKeys`、`tips` 与按界面/悬浮层的摇杆光标配置。

最小模板（每个字段的完整说明见[字段参考](./docs/01.config-field-reference_ZH.md)）：

```json
{
  "mod_id": "examplemod",
  "contributors": ["yourname"],
  "config_version": "1.0.0",
  "mod_versions": [
    { "version": "", "inGameKeys": {}, "screenKeys": {}, "overlayKeys": {},
      "ignoreKeys": [], "tips": [], "screen": {}, "overlay": {} }
  ]
}
```

---

## 文档

| 文档 | 内容 |
|------|------|
| [01.config-field-reference](./docs/01.config-field-reference_ZH.md) | **配置字段参考** —— 每个字段、通道与行为的完整说明（EN: [English](./docs/01.config-field-reference.md)） |
| [02.build-your-own-compat-source](./docs/02.build-your-own-compat-source_ZH.md) | **构建自己的模组适配源** —— 含 GitHub Fork 六步快速上手（EN: [English](./docs/02.build-your-own-compat-source.md)） |
| [03.community-config-repo-design](./docs/03.community-config-repo-design_ZH.md) | 社区源系统设计文档（已按实现复核修订；EN: [English](./docs/03.community-config-repo-design.md)） |

## 贡献流程

1. **Fork** 本仓库，切到与目标 ControlFlex 版本对应的分支（如 `0.8.7`）。
2. **新建** `<MC版本>/<加载器>/mods/<mod_id>.json`（裸键名来自游戏「控制」界面或 `[ModCompat]` 日志行），并复制到适用的每个加载器目录。
3. **重新生成清单**：`python3 generate_manifest.py` —— 更新过的 `manifest.json` 一并提交。
4. **实机测试**（可将你的 fork 作为[源](./docs/02.build-your-own-compat-source_ZH.md#9-本地测试)添加，或复制到 `user/`），然后**提交 Pull Request**，附：模组名称与测试版本、加载器、配置解决的问题。

也可以不走 PR、直接运营自己的适配源 —— 见[文档 02](./docs/02.build-your-own-compat-source_ZH.md)。

---

## 相关项目

| 仓库 | 用途 |
|------|------|
| [ControlFlex](https://github.com/ControlFlexMC/control-flex) | 主 mod |
| `cfx-{modid}-compat` | 纯 JSON 不够时的**代码**适配 mod |

---

## 许可证

见 [LICENSE](./LICENSE)。
