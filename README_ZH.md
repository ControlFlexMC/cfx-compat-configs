# cfx-compat-configs

[English](./README.md)

[ControlFlex](https://www.curseforge.com/minecraft/mc-mods/control-flex)（CFX）的**社区模组兼容配置**仓库：通过 JSON 告诉 ControlFlex 如何正确处理第三方模组的按键、摇杆模式、跨层按键等。

将 [`compat/`](./compat/) 下的 `{modid}_keys.json` 复制到**游戏实例**目录中：

```text
config/controlflex/compat/user/
```

ControlFlex 按三层目录加载 compat 配置（优先级 `user` > `mods` > `default`）：
- `default/` — ControlFlex 内置配置
- `mods/` — 桥接模组安装（cfx-compat-* 系列模组）
- **`user/`** — 你的自定义配置（最高优先级，永不被覆盖）

新增或修改配置后需**重启游戏**。在 `latest.log` 中搜索 `[ModCompat]` 确认是否加载成功。

---

## 目录结构

```text
compat/
  dragonminez_keys.json
  epicfight_keys.json   （示例 — 欢迎 PR 补充）
  ...
```

每个文件命名为 **`{modid}_keys.json`**，`{modid}` 与 Forge/Fabric 中的模组 ID 一致。

---

## 如何使用

1. 从 [`compat/`](./compat/) 复制 `{modid}_keys.json`。
2. 放入实例内的 **`config/controlflex/compat/user/`**（没有则新建）。
3. 重启 Minecraft。
4. 打开 ControlFlex 设置，确认该模组按键显示与行为正常。
5. 若异常，在日志中搜索 `[ModCompat] Failed to load compat config`（最常见为 JSON 语法错误）。

ControlFlex 内置配置解压到 `default/` 目录，**不会覆盖** `user/` 中你已有的文件。

### `config/controlflex/compat/user/` 在哪里？

**简便找法：** 在启动器里打开实例文件夹（「打开文件夹」「实例目录」等）。找到包含 **`mods/`** 的那一层，即为游戏根目录。compat 文件路径为：

```text
<游戏根目录>/config/controlflex/compat/user/{modid}_keys.json
```

部分启动器会把游戏根目录放在 `.minecraft` 或 `minecraft` 子文件夹里。若 `mods` 在 `.minecraft` 内，则使用：

```text
<实例>/.minecraft/config/controlflex/compat/
```

若 `mods` 直接在实例文件夹下，则使用：

```text
<实例>/config/controlflex/compat/
```

---

## 如何新增 compat 配置（贡献流程）

1. **Fork** 本仓库。
2. **调研目标模组** — 确认 mod ID、`KeyMapping` 名称（游戏「控制」界面、模组 JAR 或 ControlFlex 日志中的 dynamic action 注册信息）。
3. **新建** `compat/{modid}_keys.json`（见下方[最小模板](#最小模板)）。
4. **填写元数据** — 建议填写 `mod_name`、`home_page`，便于在 GitHub 上识别模组。
5. **按需配置字段** — 大部分数组可留空；根据实测添加 `guiKeys`、`specialActionKeys`、摇杆规则等。
6. **实机测试** — 安装 ControlFlex 与目标模组后验证。
7. **提交 Pull Request**，说明：
   - 模组名称与测试版本
   - 加载器（Forge / Fabric / both）
   - 配置解决的问题（如长按开菜单、GLFW 轮询、GUI 摇杆光标等）

### 如何查找 KeyMapping 名称

- 游戏内：**选项 → 控制** → 对应模组分类 → 如 `key.dragonminez.utility_menu`。
- JSON 中只写 **key 名**，不含 `modid:` 前缀。
- ControlFlex 内部 action ID 格式为 `{modid}:{keyName}`。

### JSON 注意

- 校验 JSON 语法（**不要**在最后一项后加逗号）。
- 通用配置使用 `"version": ""` 匹配所有版本。
- 需要长按开菜单 / GLFW 轮询时，使用 **`specialActionKeys`**（`GLFW_COMPAT`、`PHASE_PERSISTENT`）。

---

## 最小模板

```json
{
  "mod_id": "examplemod",
  "mod_name": "示例模组",
  "home_page": "https://www.curseforge.com/minecraft/mc-mods/examplemod",
  "loader": ["forge"],
  "versions": [
    {
      "version": "",
      "guiKeys": [],
      "ignoreKeys": [],
      "skipForgeKeys": [],
      "skipVanillaKeys": [],
      "specialActionKeys": [],
      "itemSuppressKeys": [],
      "tips": [],
      "leftStick": [],
      "rightStick": []
    }
  ]
}
```

---

## 字段说明

### 顶层字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `mod_id` | string | 是 | 模组 ID，须与已加载模组一致（如 `jei`、`epicfight`）。 |
| `mod_name` | string | 否 | 可读模组名（如 `Just Enough Items`）。便于本仓库阅读；ControlFlex 运行时目前不强制依赖。 |
| `home_page` | string | 否 | 模组主页（CurseForge、Modrinth、GitHub 等）。便于查阅；运行时可选。 |
| `loader` | String[] | 否 | 加载器限制: `["forge"]`, `["fabric"]`, `["forge","fabric"]`；省略 = 全平台。平台不符时跳过该配置。 |
| `versions` | array | 是 | 按版本区分的规则列表；见[版本匹配](#版本匹配)。 |

### `versions[]` 字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `version` | string | 是 | 语义版本。`""` 匹配所有版本（兜底）。也支持区间语法: `"[1.0,2.0)"`, `"[21.1,)"`, `"(,1.20]"`。 |
| `guiKeys` | string[] | 否 | 归属 **GUI 层**的按键（仅打开 Screen 时生效）。 |
| `ignoreKeys` | string[] | 否 | 从 ControlFlex 绑定列表中隐藏。 |
| `skipForgeKeys` | string[] | 否 | 不发送 Forge `InputEvent`（除非同时 skip vanilla）。 |
| `skipVanillaKeys` | string[] | 否 | 不调用 `KeyMapping.setPressed()`。 |
| `specialActionKeys` | object[] | 否 | 特殊标志；见 [specialActionKeys](#specialactionkeys)。 |
| `itemSuppressKeys` | object[] | 否 | 持特定物品时屏蔽动作；见 [itemSuppressKeys](#itemsuppresskeys)。 |
| `tips` | object[] | 否 | 绑定界面中的提示文字。 |
| `leftStick` | object[] | 否 | GUI 下**左摇杆**行为；见[摇杆模式](#摇杆模式-leftstick--rightstick)。 |
| `rightStick` | object[] | 否 | GUI 下**右摇杆**行为。 |

### 版本匹配

`version` 字段支持三种形式，按数组顺序**首个匹配即生效**：

| 格式 | 示例 | 语义 |
|------|------|------|
| 通配符 | `""` | 匹配所有版本 |
| 精确 | `"1.21.1"` | 仅匹配此版本 |
| 区间 | `"[1.0,2.0)"` | `1.0 ≤ v < 2.0` |
| 区间 | `"[21.1,)"` | `v ≥ 21.1` |
| 区间 | `"(,1.20]"` | `v ≤ 1.20` |

> 推荐: 具体版本条目在前，通配符 `""` 在最后作 fallback。

---

### `guiKeys`

归入 GUI 层的按键（打开背包、模组界面等时生效）：

```json
"guiKeys": ["key.jei.showRecipe"]
```

值为 `KeyMapping.getName()`，**不含** `mod_id:` 前缀。

---

### `ignoreKeys`

从绑定 UI 隐藏（调试键、作弊键等）：

```json
"ignoreKeys": ["key.jei.toggleCheatMode"]
```

---

### `skipForgeKeys` 与 `skipVanillaKeys`

默认：ControlFlex 同时走 `KeyMapping.setPressed()` 与 Forge 按键事件（与 MC 键盘一致）。

| skipForgeKeys | skipVanillaKeys | 行为 |
|:-:|:-:|---|
| 否 | 否 | 双通道（默认） |
| 是 | 否 | 仅 KeyMapping |
| 否 | 是 | 仅 Forge 事件 |
| 是 | 是 | 均不发送 |

常见场景：

- **`skipForgeKeys`** — 模组两路都监听 → 双触发。
- **`skipVanillaKeys`** — 模组只监听 Forge；`setPressed()` 有副作用。

---

### `specialActionKeys`

声明跨层 / GLFW 镜像的推荐方式：

```json
"specialActionKeys": [
  {
    "keyName": "key.dragonminez.utility_menu",
    "flags": ["GLFW_COMPAT", "PHASE_PERSISTENT"]
  }
]
```

| 标志 | 含义 |
|------|------|
| `GLFW_COMPAT` | 将手柄按住状态镜像到合成 `InputConstants.isKeyDown()`，供直接轮询 GLFW 的模组使用。 |
| `PHASE_PERSISTENT` | 打开 GUI 后仍保持逻辑按下（避免长按菜单闪退）。须配合 **HOLD** 触发模式。 |

`keyName` 不可与 `guiKeys` / `ignoreKeys` 重复。勿在此配置原版 Shift，请用 ControlFlex 虚拟 Shift。

---

### `itemSuppressKeys`

主手持有特定物品时临时屏蔽按键：

```json
"itemSuppressKeys": [
  {
    "item": "efn:yamato_dmc*",
    "playerState": ["epicfight:battle_mode"],
    "suppressActions": ["use"]
  },
  {
    "tag": "forge:swords",
    "suppressActions": ["attack"]
  }
]
```

| 字段 | 说明 |
|------|------|
| `item` | 物品 ID，支持 `*` 通配；与 `tag` 二选一。 |
| `tag` | 物品标签；与 `item` 二选一。 |
| `playerState` | 可选 `modId:状态` 列表（OR）；空表示仅看物品。 |
| `suppressActions` | 要屏蔽的 action — 内置短名（`attack`、`use`）或完整 ID。 |

---

### `tips`

绑定行提示：

```json
"tips": [
  {
    "key": "key.epicfight.weapon_innate_skill",
    "text": {
      "en_us": "Bind to the same button as Attack; use HOLD mode.",
      "zh_cn": "请绑定为与「攻击」相同的按键，并使用 HOLD 模式。"
    }
  }
]
```

**`text` 语言**

- 键名为 **Minecraft 语言代码**（与「选项 → 语言」一致），如 `en_us`、`zh_cn`、`zh_tw`、`ja_jp`、`fr_fr`。**没有固定白名单**，JSON 里写什么 code 就支持什么。
- 运行时按玩家当前所选语言在 `text` 中查找对应条目。
- 若当前语言不存在，**回退到 `en_us`**；若连 `en_us` 也没有，则不显示 tip。
- **建议始终提供 `en_us`**，保证所有玩家能看到提示；有精力时可补充 `zh_cn` 或其他语言。

| 字段 | 类型 | 说明 |
|------|------|------|
| `key` | string | KeyMapping 名称（不含 `mod_id:` 前缀） |
| `text` | object | 语言代码 → 提示文案 |

---

### 摇杆模式（`leftStick` / `rightStick`）

在 **GUI（有 Screen）** 下定制摇杆。按 Screen 全类名匹配（`screen.getClass().getName()`）。

```json
"leftStick": [
  {
    "screens": ["com.dragonminez.client.gui.UtilityMenuScreen"],
    "mode": "VIRTUAL_MOUSE"
  }
],
"rightStick": [
  {
    "screens": ["com.dragonminez.client.gui.UtilityMenuScreen"],
    "mode": "RADIAL_CURSOR",
    "threshold": 0.5,
    "cursorRadius": 90,
    "returnToCenter": false,
    "angleBias": 1.0
  }
]
```

| `mode` | 左摇杆（GUI） | 右摇杆（GUI） |
|--------|---------------|---------------|
| `DEFAULT` | 增量虚拟鼠标 | 无轴输入（GUI 下本就不转视角） |
| `VIRTUAL_MOUSE` | 增量虚拟鼠标 | 增量虚拟鼠标 |
| `RADIAL_CURSOR` | 以屏幕中心为原点的极坐标光标 | 同上 |
| `DISABLED` | 禁用轴；**按下**摇杆的按键绑定仍有效 | 同上 |

**`RADIAL_CURSOR` 参数**

| 字段 | 默认 | 说明 |
|------|------|------|
| `threshold` | `0.5` | 死区 |
| `cursorRadius` | `100` | 相对屏幕中心的半径（逻辑像素 × gui scale） |
| `returnToCenter` | `false` | 松杆是否回中 |
| `angleBias` | `1.0` | 角度灵敏度 |

数组中**第一个**匹配的 `screens` 规则生效；无匹配则回退 `DEFAULT`。

---

## 常见模组类型参考

| 类型 | 常用字段 |
|------|----------|
| 战斗（Epic Fight） | `guiKeys`、`ignoreKeys`、`skipForgeKeys`、`itemSuppressKeys` |
| 辅助（JEI） | `guiKeys`、大量 `ignoreKeys` 隐藏作弊/调试键 |
| 长按开菜单 | `specialActionKeys`（`GLFW_COMPAT` + `PHASE_PERSISTENT`）、`tips`、摇杆配置 |
| 简单按键模组 | 常只需 `mod_id` 与元数据 |

---

## 相关项目

| 仓库 | 用途 |
|------|------|
| [ControlFlex](https://github.com/ControlFlexMC/control-flex) | 主 mod |
| `cfx-{modid}-compat`（规划） | 纯 JSON 不够时的**代码**适配 mod |

---

## 许可证

见 [LICENSE](./LICENSE)。
