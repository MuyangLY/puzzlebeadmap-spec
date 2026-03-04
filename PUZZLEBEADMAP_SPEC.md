# PuzzleBeadMap 格式规范

**版本**：1.0
**状态**：草案（Draft）
**文件扩展名**：`.txt`（推荐），`.json` 亦可
**字符编码**：UTF-8
**规格仓库**：https://github.com/MuyangLY/puzzlebeadmap-spec
---

## 概述

PuzzleBeadMap 是一种用于描述拼豆（熔珠/Perler Bead）图案的开放文件格式，基于 JSON 编码。

设计目标：

- **可读性**：人类可直接阅读和编辑
- **完整性**：单个文件包含图案还原所需的全部信息，不依赖外部色卡数据库
- **多品牌**：原生支持多品牌混搭，每颗豆保留品牌信息
- **可移植性**：任何支持 JSON 的程序均可读写此格式
- **可扩展性**：通过版本字段保证向后兼容

---

## 文件结构

一个合法的 PuzzleBeadMap 文件是一个 JSON 对象，包含以下顶层字段：

```
{
  "format":  string,   // 必填
  "version": string,   // 必填
  "meta":    object,   // 可选
  "canvas":  object,   // 必填
  "palette": array,    // 必填
  "grid":    array     // 必填
}
```

---

## 字段详细规范

### `format`

**类型**：`string`
**必填**：是
**固定值**：`"PuzzleBeadMap"`

用于识别文件类型，读取方应首先检查此字段。

```json
"format": "PuzzleBeadMap"
```

---

### `version`

**类型**：`string`
**必填**：是
**当前值**：`"1.0"`

格式版本号，采用 `主版本.次版本` 格式。

- **主版本**变更：不向后兼容（字段结构发生破坏性变化）
- **次版本**变更：向后兼容（新增可选字段）

读取方应拒绝无法识别的主版本号，并对未知的次版本保持宽容。

```json
"version": "1.0"
```

---

### `meta`

**类型**：`object`
**必填**：否

图案的元数据，所有子字段均为可选。省略整个 `meta` 对象时文件仍合法。

| 字段 | 类型 | 说明 |
|------|------|------|
| `title` | string | 图案名称 |
| `author` | string | 创作者昵称 |
| `created` | string | 创建日期，推荐 ISO 8601 格式（`"2026-03-04"`） |
| `description` | string | 备注说明，自由文本 |
| `tags` | string[] | 标签列表，便于分类检索 |
| `app` | string | 生成此文件的应用名称 |
| `appVersion` | string | 生成应用的版本号 |

```json
"meta": {
  "title": "小猫图案",
  "author": "豆豆妈",
  "created": "2026-03-04",
  "description": "女儿最爱的小猫，适合 Hama 大豆板",
  "tags": ["动物", "可爱", "入门"],
  "app": "拼豆转换器",
  "appVersion": "2.5.0"
}
```

---

### `canvas`

**类型**：`object`
**必填**：是

描述图案画布的尺寸，单位为格（颗豆）。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `cols` | integer | 是 | 列数（宽度），最小值 1 |
| `rows` | integer | 是 | 行数（高度），最小值 1 |

`grid` 数组的实际尺寸必须与 `canvas` 声明一致：外层数组长度等于 `rows`，每个内层数组长度等于 `cols`。

```json
"canvas": {
  "cols": 30,
  "rows": 40
}
```

---

### `palette`

**类型**：`array`
**必填**：是

调色板，列出图案中所有用到的颜色。`grid` 中的格子通过 `id` 引用此数组中的条目。

每个调色板条目的字段：

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `id` | integer | 是 | 从 0 开始的唯一整数索引，供 `grid` 引用 |
| `brand` | string | 是 | 品牌标识符，小写英文（见附录） |
| `brandLabel` | string | 是 | 品牌显示名称（如 `"Hama"`、`"MARD"`） |
| `code` | string | 是 | 品牌官方色号（如 `"H01"`、`"A5"`） |
| `name` | string | 是 | 颜色名称 |
| `rgb` | [r,g,b] | 是 | RGB 值，三个整数，范围 0–255 |

> **为什么同时保留 `brand+code` 和 `rgb`？**
> `brand+code` 是购买和施工所需的精确信息；`rgb` 是颜色的物理值，支持在不依赖原品牌色卡数据库的情况下完成品牌转换（通过 ΔE 色差匹配到其他品牌最近色）。

```json
"palette": [
  {
    "id": 0,
    "brand": "hama",
    "brandLabel": "Hama",
    "code": "H01",
    "name": "White",
    "rgb": [229, 236, 241]
  },
  {
    "id": 1,
    "brand": "hama",
    "brandLabel": "Hama",
    "code": "H10",
    "name": "Black",
    "rgb": [30, 30, 35]
  },
  {
    "id": 2,
    "brand": "mard",
    "brandLabel": "MARD",
    "code": "A5",
    "name": "粉红",
    "rgb": [255, 182, 193]
  }
]
```

`id` 值应从 0 开始连续递增，不应存在跳号或重复。

---

### `grid`

**类型**：`array`
**必填**：是

二维数组，描述图案中每一格的颜色。

- 外层数组：行，长度等于 `canvas.rows`
- 内层数组：列，长度等于 `canvas.cols`
- 每个格子的值为：
  - **integer**：`palette` 中对应 `id` 的颜色
  - **`null`**：空白格（无豆，常见于图纸分割时末块补齐）

```json
"grid": [
  [0, 0, 1, 1, 0, 0],
  [0, 1, 2, 2, 1, 0],
  [1, 2, 2, 2, 2, 1],
  [null, 1, 1, 1, 1, null]
]
```

---

## 完整示例文件

```json
{
  "format": "PuzzleBeadMap",
  "version": "1.0",

  "meta": {
    "title": "小猫图案",
    "author": "豆豆妈",
    "created": "2026-03-04",
    "description": "女儿最爱的小猫，适合 Hama 大豆板",
    "tags": ["动物", "可爱"],
    "app": "拼豆转换器",
    "appVersion": "2.5.0"
  },

  "canvas": {
    "cols": 6,
    "rows": 4
  },

  "palette": [
    {
      "id": 0,
      "brand": "hama",
      "brandLabel": "Hama",
      "code": "H01",
      "name": "White",
      "rgb": [229, 236, 241]
    },
    {
      "id": 1,
      "brand": "hama",
      "brandLabel": "Hama",
      "code": "H10",
      "name": "Black",
      "rgb": [30, 30, 35]
    },
    {
      "id": 2,
      "brand": "mard",
      "brandLabel": "MARD",
      "code": "A5",
      "name": "粉红",
      "rgb": [255, 182, 193]
    }
  ],

  "grid": [
    [0, 0, 1, 1, 0, 0],
    [0, 1, 2, 2, 1, 0],
    [1, 2, 2, 2, 2, 1],
    [null, 1, 1, 1, 1, null]
  ]
}
```

---

## 合法性校验规则

读取方在处理文件前应执行以下校验，校验失败应向用户报告明确的错误原因：

| 编号 | 规则 |
|------|------|
| V1 | `format` 字段值必须为 `"PuzzleBeadMap"` |
| V2 | `version` 字段必须存在且为字符串 |
| V3 | `canvas.cols` 和 `canvas.rows` 必须为正整数 |
| V4 | `palette` 必须为数组，每个条目的 `id`、`brand`、`brandLabel`、`code`、`name`、`rgb` 字段必须存在 |
| V5 | `palette` 中所有 `id` 值必须唯一，且从 0 开始连续 |
| V6 | `palette[].rgb` 必须为长度为 3 的数组，每个值为 0–255 的整数 |
| V7 | `grid` 必须为数组，外层长度等于 `canvas.rows`，每个内层数组长度等于 `canvas.cols` |
| V8 | `grid` 中的非 `null` 值必须为整数，且在 `palette` 的 `id` 范围内 |

---

## 未知品牌处理建议

当读取方不支持文件中某个 `brand` 值时（例如第三方工具生成了本地色卡库中不存在的品牌），推荐以下处理流程：

1. **提示用户**：告知图案中包含未知品牌 `X`，本地暂无该品牌色卡
2. **让用户选择目标品牌**：从本地已有品牌中选择一个进行转换
3. **ΔE 匹配**：对未知品牌的每个颜色，使用 `rgb` 字段在用户选定的目标品牌色卡中，通过 CIELAB ΔE76 色差算法找到最近色号
4. **应用替换**：将 `palette` 中该条目的 `brand`、`brandLabel`、`code`、`name` 替换为目标品牌的匹配结果，`rgb` 字段可选择保留原值或更新为目标色号的 RGB 值
5. **告知转换结果**：展示替换摘要（如"已将 12 种颜色从未知品牌转换为 Hama，平均色差 ΔE = 8.3"）

> 此流程为建议，各实现可根据实际情况调整。

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 1.0 | 2026-03-04 | 初始草案发布 |

---

## 附录：已知品牌标识符

以下为「我家公主们爱拼豆」小程序中使用的品牌标识符，供参考。第三方实现可使用自定义 `brand` 值，只要在文件内部保持一致即可。

| `brand` 值 | `brandLabel` | 备注 |
|------------|--------------|------|
| `hama` | Hama | 丹麦，92色 |
| `artkal` | Artkal | 国产，174色 |
| `perler` | Perler | 美国，102色 |
| `mard` | MARD | 国产，295色 |
| `coco` | COCO | 国产，291色 |
| `kaka` | 卡卡 | 国产，154色 |
| `manman` | 漫漫 | 国产，290色 |
| `panpan` | 盼盼 | 国产，291色 |
| `mixiaowo` | 咪小窝 | 国产，291色 |
| `youken` | 优肯 | 国产，141色 |
| `xiaowu` | 小舞 | 国产，295色 |
| `dodo` | DODO | 国产，277色 |
| `huangdoudou` | 黄豆豆 | 国产，295色 |

---

## 开源协议

本格式规范以 [CC0 1.0 通用](https://creativecommons.org/publicdomain/zero/1.0/deed.zh) 协议发布，任何人可自由使用、实现、修改和分发，无需署名，无需许可。
