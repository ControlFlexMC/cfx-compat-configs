# cfx-compat-configs

[中文](./README_ZH.md)

Community **mod compatibility configs** for [ControlFlex](https://www.curseforge.com/minecraft/mc-mods/control-flex) (CFX): JSON files that tell the mod how to handle third-party key bindings, stick modes, cross-layer keys, and related behavior.

> **⚠ Version requirement:** The configs on the `main` branch target **ControlFlex >= 0.8.5**.
> If you are on ControlFlex **0.8.4 or lower**, use the [`0.8.4`](https://github.com/ControlFlexMC/cfx-compat-configs/tree/0.8.4) branch instead — config formats differ between these versions.

Copy `{modid}_keys.json` from [`compat/`](./compat/) into your **game instance**:

```text
config/controlflex/compat/user/
```

ControlFlex loads compat configs from three layers (priority: `user` > `mods` > `default`):
- `default/` — built-in configs shipped with ControlFlex
- `mods/` — installed by bridge mods (via `cfx-compat-*` mods)
- **`user/`** — your custom configs (highest priority, never overwritten)

Restart the game after adding or updating configs. Check `latest.log` for `[ModCompat]` lines to confirm loading.

---

## Repository layout

```text
compat/
  dragonminez_keys.json
  epicfight_keys.json   (example — add via PR)
  ...
```

Each file is named **`{modid}_keys.json`**, where `{modid}` matches the mod’s ID in Forge/Fabric.

---

## How to install a config

1. Download or copy `{modid}_keys.json` from [`compat/`](./compat/).
2. Place it in **`config/controlflex/compat/user/`** inside your instance (create the folder if needed).
3. Restart Minecraft.
4. Open ControlFlex settings → confirm the mod’s keys appear and behave as expected.
5. If something fails, search logs for `[ModCompat] Failed to load compat config` (JSON syntax errors are the most common cause).

Configs shipped inside the ControlFlex JAR are extracted on first launch and **do not overwrite** files you already have in `config/controlflex/compat/` (they are extracted to `default/`).

### Where is `config/controlflex/compat/user/`?

**Rule of thumb:** open your instance folder in the launcher (“Open folder”, “Instance folder”, etc.). Go to the directory that contains **`mods/`** — that is the game root. Compat files go in:

```text
<game root>/config/controlflex/compat/user/{modid}_keys.json
```

Some launchers wrap the game root in a `.minecraft` or `minecraft` subfolder. If you see `mods` inside `.minecraft`, use:

```text
<instance>/.minecraft/config/controlflex/compat/user/
```

If `mods` is directly under the instance folder, use:

```text
<instance>/config/controlflex/compat/user/
```

---

## How to add a new compat config (contribution)

1. **Fork** this repository.
2. **Inspect the target mod** — find its mod ID and `KeyMapping` names (Controls screen, mod JAR, or logs when ControlFlex registers dynamic actions).
3. **Create** `compat/{modid}_keys.json` (see [minimal template](#minimal-template) below).
4. **Fill in metadata** — `mod_name` and `home_page` help others identify the mod on GitHub.
5. **Configure only what you need** — most fields can stay empty arrays; add `guiKeys`, `specialActionKeys`, stick rules, etc. based on testing.
6. **Test in-game** with ControlFlex installed and the target mod loaded.
7. **Open a pull request** with:
   - Mod name and version tested
   - Loader (Forge / Fabric / both)
   - Brief notes on what the config fixes (e.g. hold-to-open menu, GLFW polling, stick cursor)

### Finding KeyMapping names

- In-game: Minecraft **Controls** → mod’s category → key names like `key.dragonminez.utility_menu`.
- In config/JSON: use the **key name only**, without the `modid:` prefix (e.g. `key.dragonminez.utility_menu`, not `dragonminez:key.dragonminez.utility_menu`).
- Action IDs inside ControlFlex use the full form: `{modid}:{keyName}`.

### JSON tips

- Validate JSON (no trailing commas).
- Use `"version": ""` as a catch-all until you need version-specific blocks.
- Use **`specialActionKeys`** with `GLFW_COMPAT` / `PHASE_PERSISTENT` when hold-to-open or GLFW polling is required.

---

## Minimal template

```json
{
  "mod_id": "examplemod",
  "mod_name": "Example Mod",
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

## Field reference

### Top-level fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mod_id` | string | Yes | Mod ID; must match the loaded mod (e.g. `jei`, `epicfight`). |
| `mod_name` | string | No | Human-readable mod name for this repo (e.g. `Just Enough Items`). Not required by ControlFlex at runtime today; strongly recommended for contributors. |
| `home_page` | string | No | Mod project page (CurseForge, Modrinth, GitHub, etc.). For documentation and review; optional at runtime. |
| `loader` | String[] | No | Loader filter: `["forge"]`, `["fabric"]`, or `["forge","fabric"]`. Omit = all platforms. Config is skipped on the wrong loader. |
| `versions` | array | Yes | Version-specific rule sets; see [version matching](#version-matching). |

### `versions[]` fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | Yes | Semantic version for this block. `""` matches all versions (fallback). Also supports interval notation: `"[1.0,2.0)"`, `"[21.1,)"`, `"(,1.20]"`. |
| `guiKeys` | string[] | No | Key names assigned to the **GUI layer** (active only while a Screen is open). |
| `ignoreKeys` | string[] | No | Keys hidden from ControlFlex binding UI. |
| `skipForgeKeys` | string[] | No | Do **not** send Forge `InputEvent` for these keys (`KeyMapping` path still used unless also skipped). |
| `skipVanillaKeys` | string[] | No | Do **not** call `KeyMapping.setPressed()` for these keys (Forge events still sent unless skipped). |
| `specialActionKeys` | object[] | No | Special flags per key; see [specialActionKeys](#specialactionkeys). |
| `itemSuppressKeys` | object[] | No | Suppress actions while holding specific items/tags; see [itemSuppressKeys](#itemsuppresskeys). |
| `tips` | object[] | No | Hint text shown next to bindings in ControlFlex settings. |
| `leftStick` | object[] | No | Per-screen **left stick** behavior in GUI; see [stick modes](#stick-modes-leftstick--rightstick). |
| `rightStick` | object[] | No | Per-screen **right stick** behavior in GUI. |

### Version matching

The `version` field supports three forms. Entries are evaluated **in array order**; the **first match wins**.

| Format | Example | Meaning |
|--------|---------|---------|
| Wildcard | `""` | Matches all versions |
| Exact | `"1.21.1"` | Exact match only |
| Interval | `"[1.0,2.0)"` | `1.0 ≤ v < 2.0` |
| Unbounded | `"[21.1,)"` | `v ≥ 21.1` |
| Unbounded | `"(,1.20]"` | `v ≤ 1.20` |

> Tip: Put specific version entries first, wildcard `""` last as a fallback.

---

### `guiKeys`

Keys that belong on the GUI layer (inventory open, mod screens, etc.):

```json
"guiKeys": ["key.jei.showRecipe"]
```

Values are `KeyMapping.getName()` **without** the `mod_id:` prefix.

---

### `ignoreKeys`

Hide keys from the binding list (debug keys, cheat keys, duplicates):

```json
"ignoreKeys": ["key.jei.toggleCheatMode"]
```

---

### `skipForgeKeys` vs `skipVanillaKeys`

Default: ControlFlex sends **both** `KeyMapping.setPressed()` and Forge key events (like vanilla keyboard).

| skipForgeKeys | skipVanillaKeys | Result |
|:-:|:-:|---|
| no | no | Both channels (default) |
| yes | no | KeyMapping only |
| no | yes | Forge events only |
| yes | yes | Neither |

Typical cases:

- **`skipForgeKeys`** — mod listens on both paths → double fire.
- **`skipVanillaKeys`** — mod only uses Forge events; `setPressed()` causes side effects.

---

### `specialActionKeys`

Modern way to declare cross-layer / GLFW mirror behavior:

```json
"specialActionKeys": [
  {
    "keyName": "key.dragonminez.utility_menu",
    "flags": ["GLFW_COMPAT", "PHASE_PERSISTENT"]
  }
]
```

| Flag | Meaning |
|------|---------|
| `GLFW_COMPAT` | Mirror controller hold into synthetic `InputConstants.isKeyDown()` state for mods that poll GLFW instead of `KeyMapping.isDown()`. |
| `PHASE_PERSISTENT` | Keep the binding logically pressed when a Screen opens (prevents hold menus from flickering closed). Use with **HOLD** trigger mode only. |

`keyName` must not appear in `guiKeys` or `ignoreKeys`. Do not list vanilla Shift here — use ControlFlex virtual shift.

---

### `itemSuppressKeys`

Suppress actions while the player holds matching items:

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

| Field | Description |
|-------|-------------|
| `item` | Item ID; `*` wildcard supported. Mutually exclusive with `tag`. |
| `tag` | Item tag. Mutually exclusive with `item`. |
| `playerState` | Optional `modId:state` list (OR). Empty = always when item matches. |
| `suppressActions` | Action IDs to block — built-in short names (`attack`, `use`) or full IDs (`epicfight:key.epicfight.guard`). |

---

### `tips`

Hints in the binding UI:

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

**`text` languages**

- Keys are **Minecraft language codes** (same as Options → Language), e.g. `en_us`, `zh_cn`, `zh_tw`, `ja_jp`, `fr_fr`. There is **no fixed whitelist** — any code you add in JSON can be used.
- At runtime ControlFlex reads the player’s selected language and looks up that key in `text`.
- If the current language is missing, **`en_us` is used as fallback**. If `en_us` is also missing, no tip is shown.
- **Always include `en_us`** so all players see a hint. Add `zh_cn` or other locales when you can provide translations.

| Field | Type | Description |
|-------|------|-------------|
| `key` | string | KeyMapping name (without `mod_id:` prefix) |
| `text` | object | Language code → hint string |

---

### Stick modes (`leftStick` / `rightStick`)

Customize stick behavior **while a GUI Screen is open**. Match by fully qualified screen class name (`screen.getClass().getName()`).

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

| `mode` | Left stick (GUI) | Right stick (GUI) |
|--------|------------------|-------------------|
| `DEFAULT` | Incremental virtual mouse | No axis action (camera already disabled in GUI) |
| `VIRTUAL_MOUSE` | Incremental virtual mouse | Incremental virtual mouse |
| `RADIAL_CURSOR` | Polar cursor from screen center | Polar cursor from screen center |
| `DISABLED` | Axis ignored; stick **click** bindings still work | Same |

**`RADIAL_CURSOR` parameters**

| Field | Default | Description |
|-------|---------|-------------|
| `threshold` | `0.5` | Stick deadzone before cursor moves |
| `cursorRadius` | `100` | Distance from screen center (logical pixels × gui scale) |
| `returnToCenter` | `false` | Snap cursor to center when stick released |
| `angleBias` | `1.0` | Angle sensitivity multiplier |

First matching entry in the array wins; if no screen matches, behavior falls back to `DEFAULT`.

---

## Mod-type cheat sheet

| Mod type | Typical fields |
|----------|----------------|
| Combat (Epic Fight) | `guiKeys`, `ignoreKeys`, `skipForgeKeys`, `itemSuppressKeys` |
| Utility (JEI) | `guiKeys`, large `ignoreKeys` for cheat/debug keys |
| Hold-to-open menu | `specialActionKeys`: `GLFW_COMPAT` + `PHASE_PERSISTENT`, `tips`, `leftStick` / `rightStick` |
| Simple key mods | Often empty arrays; only `mod_id` + metadata |

---

## Related projects

| Repository | Purpose |
|------------|---------|
| [ControlFlex](https://github.com/ControlFlexMC/control-flex) | Main controller mod |
| `cfx-{modid}-compat` (future) | Optional **code** bridge mods when JSON alone is not enough |

---

## License

See [LICENSE](./LICENSE).
