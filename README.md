# PuzzleBeadMap

一种用于描述拼豆（熔珠 / Perler Bead）图案的开放文件格式，基于 JSON 编码。

---

## 为什么需要这个格式？

拼豆软件之间无法互相读取图纸，颜色匹配结果无法保存和分享，每次换软件都要重新转换。
PuzzleBeadMap 提供一个通用中间格式，让任何工具都能读写同一份图纸文件。

---

## 设计目标

- **可读性** — 人类可直接阅读和编辑
- **完整性** — 单个文件包含图案还原所需的全部信息，不依赖外部色卡数据库
- **多品牌** — 原生支持多品牌混搭，每颗豆保留品牌信息
- **可移植性** — 任何支持 JSON 的程序均可读写
- **可扩展性** — 通过版本字段保证向后兼容

---

## 最小示例

```json
{
  "format": "PuzzleBeadMap",
  "version": "1.0",
  "canvas": { "cols": 3, "rows": 2 },
  "palette": [
    { "id": 0, "brand": "hama", "brandLabel": "Hama", "code": "H01", "name": "White", "rgb": [229, 236, 241] },
    { "id": 1, "brand": "hama", "brandLabel": "Hama", "code": "H10", "name": "Black", "rgb": [30, 30, 35] }
  ],
  "grid": [
    [0, 1, 0],
    [1, 0, 1]
  ]
}
```

`grid` 中的整数是 `palette` 的 `id` 引用，`null` 表示空白格。

---

## 完整规范

详见 [PUZZLEBEADMAP_SPEC.md](./PUZZLEBEADMAP_SPEC.md)，包含：

- 所有字段定义与约束
- 合法性校验规则（V1–V8）
- 未知品牌处理流程（基于 CIELAB ΔE76 色差匹配）
- 已知品牌标识符列表

---

## 支持此格式的工具

| 工具 | 平台 | 读 | 写 |
|------|------|----|----|
| [我家公主们爱拼豆](https://github.com/example) | 微信小程序 | ✅ | ✅ |

> 如果你的工具支持 PuzzleBeadMap，欢迎提 PR 添加到此列表。

---

## 许可证

本格式规范以 [CC0 1.0 通用](https://creativecommons.org/publicdomain/zero/1.0/deed.zh) 协议发布。
任何人可自由使用、实现、修改和分发，无需署名，无需许可。
