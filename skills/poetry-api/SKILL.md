---
name: poetry-api
description: 诗泉古诗词 API 调用指南。检索古诗词原文、随机诗词、按作者/朝代/体裁/单字筛选、全文搜索、飞花令。供古诗词赏析专家检索真实诗词原文。
---

# 诗泉古诗词 API

免费开源的中文古诗词 API（https://poetry.palemoky.com），收录唐诗、宋词等 **37 万余首**作品，数据源自 chinese-poetry 开源项目。

## 核心要点

- **Base URL**: `https://poetry.palemoky.com`
- **注意**：路径前缀是 `/api/`（不是 `/api/v1/`，旧版 v1 路径已废弃返回 404）
- 返回统一 JSON 结构：`{"data": ..., "lang": "zh-Hans"}`
- 中文参数需要 URL 编码（curl 用 `--data-urlencode` 或手动编码）

## 端点总览

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/poems/random` | GET | 随机诗词，支持多维筛选 |
| `/api/search?q=关键词` | GET | 全文搜索（**q 至少 3 个字符**） |
| `/api/authors?page=&page_size=` | GET | 作者列表 |
| `/api/dynasties` | GET | 朝代列表 |
| `/api/types` | GET | 体裁列表 |
| `/api/stats` | GET | 数据统计 |
| `/health` | GET | 健康检查 |

## 随机诗词筛选参数

`/api/poems/random` 支持以下查询参数（可组合）：

| 参数 | 示例 | 说明 |
|------|------|------|
| `author` | `李白` | 指定作者 |
| `dynasty` | `唐` | 指定朝代 |
| `type` | `五言绝句` | 指定体裁，**可重复出现**叠加多个值 |
| `char` | `春` | 含指定单字（飞花令场景） |

示例：
```
curl "https://poetry.palemoky.com/api/poems/random?author=李白&type=五言绝句"
curl "https://poetry.palemoky.com/api/poems/random?author=李白&dynasty=唐&type=五言绝句&type=七言绝句"
curl "https://poetry.palemoky.com/api/poems/random?char=春"
```

## 返回格式

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

## 常用调用方式

### Bash / curl（推荐，Windows 用 Git Bash）

```bash
# 随机一首诗
curl -s "https://poetry.palemoky.com/api/poems/random"

# 指定作者+体裁（中文需 URL 编码，或用 --get --data-urlencode）
curl -s --get "https://poetry.palemoky.com/api/poems/random" \
  --data-urlencode "author=李白" --data-urlencode "type=五言绝句"

# 全文搜索（关键词≥3字）
curl -s --get "https://poetry.palemoky.com/api/search" --data-urlencode "q=床前明月光"

# 飞花令：随机含"月"字的诗
curl -s --get "https://poetry.palemoky.com/api/poems/random" --data-urlencode "char=月"
```

### Node.js（内置 fetch）

```js
const res = await fetch(
  "https://poetry.palemoky.com/api/poems/random?author=" + encodeURIComponent("李白")
);
const { data } = await res.json();
```

### Python

```python
import urllib.parse, urllib.request, json
url = "https://poetry.palemoky.com/api/poems/random?" + urllib.parse.urlencode({"author": "李白"})
data = json.load(urllib.request.urlopen(url))["data"]
```

## 注意事项（含实测经验，2026-08 验证）

1. **搜索 q 参数最少 3 个字符**，少于 3 字返回 400 错误
2. **搜索匹配诗句正文，不匹配标题**。搜《静夜思》会返回诗句含"静夜思"三字的诗（如"静夜思鸿宝"），而不是李白那首。**搜篇目名的正确姿势**：用该诗最独特的诗句片段，例如：
   - 《静夜思》→ 搜「疑是地上霜」或「低头思故乡」（精确命中 id 309946）
   - 《春晓》→ 搜「春眠不觉晓」（命中）
3. **通行版本与库内版本可能不同**：数据源为 chinese-poetry，李白《静夜思》存的是宋版「床前看月光／举头望山月」。搜「床前明月光」「举头望明月」返回空属正常现象，改用库内诗句重试即可
4. `type` 参数可重复出现叠加（如同时要绝句和律诗）
5. 返回的 `content` 是**数组**，每行一句，展示时要保持分行
6. 部分生僻字（如「䍠」）是合法字符，不要误删
7. 完整端点字段文档见 `references/api-docs.md`

## 网络搜索兜底（API 检索不到时）

当上述 API 全部检索不到目标篇目（返回空 / 404 / 无匹配）时，**立即启用网络搜索**，不要放弃：

1. **检索关键词**：`{篇目名} {作者} 全诗` 或 `{作者} {篇目名} 原文`
2. **权威来源优先**：古诗文网（gushiwen.cn）、百度汉语、维基文库、国学网、全唐诗/全宋词原文
3. **交叉核验**：同一原文至少两个来源一致才采信；确认后按 API 返回格式整理（title/author/dynasty/type/content[]）继续赏析
4. **无法核实**：多来源都搜不到或来源不可靠时，如实告知用户"未能核实原文"，不编造内容
5. **冲突处理**：网络信息与诗泉 API 版本冲突时（如《静夜思》宋版 vs 通行版），以多来源交叉确认结果为准，并主动说明版本差异
