# 诗泉古诗词 API 参考文档

> 官方站点：https://poetry.palemoky.com
> 数据来源：chinese-poetry 开源项目（https://github.com/chinese-poetry/chinese-poetry）
> 收录规模：371,313 首诗词 / 13,577 位作者 / 11 个朝代 / 17 种体裁（经 /api/stats 实测）

**版本注意**：历史文档中的 `/api/v1/poems` 系列路径已废弃（返回 `{"error":{"code":"NOT_FOUND","message":"Route not found"}}`），请使用下述 `/api/` 路径。

---

## 1. 随机诗词

### GET /api/poems/random

获取随机一首诗词，可通过查询参数筛选。

| 参数 | 类型 | 说明 |
|------|------|------|
| `author` | string | 作者名（如 李白、杜甫） |
| `dynasty` | string | 朝代名（如 唐、宋、清） |
| `type` | string | 体裁名（如 五言绝句、七言律诗），可重复叠加多个 |
| `char` | string | 诗词必须包含的单字（飞花令场景） |

**示例**

```
GET /api/poems/random
GET /api/poems/random?author=李白
GET /api/poems/random?type=五言绝句
GET /api/poems/random?char=春
GET /api/poems/random?author=李白&type=五言绝句
GET /api/poems/random?author=李白&type=五言绝句&dynasty=唐
GET /api/poems/random?author=李白&dynasty=唐&type=五言绝句&type=七言绝句&type=五言律诗
```

**响应示例**

```json
{
  "data": {
    "id": 310362,
    "title": "荅友人赠乌纱帽",
    "content": ["领得乌纱帽，全胜白接䍠。", "山人不照镜，稚子道相宜。"],
    "author": { "id": 2045, "name": "李白" },
    "dynasty": { "id": 6, "name": "唐" },
    "type": { "id": 11, "name": "五言绝句" }
  },
  "lang": "zh-Hans"
}
```

---

## 2. 全文搜索

### GET /api/search?q={关键词}

按诗句内容全文搜索。**`q` 至少 3 个字符**，否则返回 400：
`{"error":{"code":"BAD_REQUEST","message":"q must be at least 3 characters"}}`

⚠️ **实测要点（2026-08 验证）**：
- 搜索匹配的是**诗句正文**，不是标题。搜「静夜思」返回的是诗句中含"静夜思"三字的诗（如"静夜思鸿宝"），**不会**直接命中李白的《静夜思》
- 数据源为 chinese-poetry 库，李白《静夜思》存的是宋版「床前**看**月光／举头望**山**月」。所以搜「床前明月光」「举头望明月」返回空是正常现象
- **找篇目的正确姿势**：用该诗最独特的诗句片段搜索，例如：
  - 《静夜思》→ `q=疑是地上霜` 或 `q=低头思故乡`（精确命中 id 309946）
  - 《春晓》→ `q=春眠不觉晓`（命中）

示例：

```
GET /api/search?q=床前明月光        # 返回空（库内为宋版"床前看月光"）
GET /api/search?q=疑是地上霜        # 命中李白《静夜思》
GET /api/search?q=春眠不觉晓        # 命中孟浩然《春晓》
GET /api/search?q=静夜思            # 匹配诗句含"静夜思"三字的诗（非标题）
```

**响应示例**（data 为数组）

```json
{
  "data": [
    {
      "id": 289238,
      "title": "同工部李侍郎适访司马子微",
      "content": ["紫微降天仙，丹地投云藻。", "..."],
      "author": { "id": 1, "name": "某作者" },
      "dynasty": { "id": 6, "name": "唐" },
      "type": { "id": 15, "name": "其他" }
    }
  ],
  "lang": "zh-Hans"
}
```

---

## 3. 作者列表

### GET /api/authors?page={n}&page_size={n}

| 参数 | 默认 | 说明 |
|------|------|------|
| `page` | 1 | 页码 |
| `page_size` | 20 | 每页条数 |

**响应示例**

```json
{
  "data": [
    {
      "id": 1,
      "name": "章天与",
      "description": null,
      "dynasty": { "id": 6, "name": "唐" }
    }
  ],
  "lang": "zh-Hans"
}
```

---

## 4. 朝代列表

### GET /api/dynasties

返回全部 11 个朝代：先秦、两汉、魏晋、南北朝、隋、唐、五代、宋（等），含 `id`、`name`、`name_en`、`start_year`、`end_year`。

```json
{
  "data": [
    { "id": 1, "name": "先秦", "name_en": "Pre-Qin", "start_year": -2070, "end_year": -221 }
  ],
  "lang": "zh-Hans"
}
```

---

## 5. 体裁列表

### GET /api/types

返回 17 种体裁，含 `id`、`name`、`category`（唐诗/宋词/等）、`lines`（句数）、`chars_per_line`（每句字数）、`description`。

常用体裁：唐诗、五言绝句（4句×5字）、七言绝句（4句×7字）、五言律诗（8句×5字）、七言律诗（8句×7字）、词牌名（如 水调歌头、如梦令）。

---

## 6. 数据统计

### GET /api/stats

```json
{ "data": { "poems": 371313, "authors": 13577, "dynasties": 11, "types": 17 }, "lang": "zh-Hans" }
```

---

## 7. 健康检查

### GET /health

返回服务状态，用于确认 API 可用。

---

## 错误处理

| 场景 | 返回 |
|------|------|
| 路径不存在 | `{"error":{"code":"NOT_FOUND","message":"Route not found"}}` HTTP 404 |
| q 少于 3 字符 | `{"error":{"code":"BAD_REQUEST","message":"q must be at least 3 characters"}}` HTTP 400 |
| 无匹配结果 | `data` 为 `null` 或空数组 |

**降级策略**：当按篇目名搜不到时，建议：
1. 换用该诗**最独特的诗句片段**（≥3字）搜索（如《静夜思》→「疑是地上霜」）
2. 注意通行版本与库内版本差异（如《静夜思》库内为宋版「床前看月光」），换用库内诗句重试
3. 用 `/api/poems/random?author=XX` 从作者入手
4. 用 `char` 参数做飞花令随机获取
