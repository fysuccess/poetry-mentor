# Poetry Mentor - 古诗词趣味工坊

海量古典诗词，五维赏析与分龄教学。一键生成互动赏析页，支持朗读、飞花令、诗词消消乐，让读诗成趣。

作者：飞月（feiyue.xk@qq.com）

## 类型

Agent 型（单个 AI 专家）

## 功能

- **诗词检索**：通过诗泉 API 按作者/朝代/体裁/单字检索真实原诗，或全文搜索
- **生动赏析**：从画面、字词、情感、背景、名句五个维度赏析
- **分龄解读**：适配学龄前/小学/初中/高中/成人 6 个年龄段
- **学习指导**：背诵技巧、飞花令玩法、仿写创作、考试考点、亲子互动
- **诗词游戏**：飞花令接龙（char 参数随机取诗）

## 使用示例

- 「帮我赏析《静夜思》，并指导小学低年级的孩子学习」
- 「推荐一首适合初中生的励志诗并赏析」
- 「用飞花令"月"字找几首诗词来玩接龙」

## 头像

头像已自动生成在 `avatars/` 目录下。如需替换为自定义头像，要求：
- 格式：PNG（推荐）或 JPG
- 尺寸：512×512 px
- 大小：单张不超过 500KB

## 依赖

- 内置 skill：`skills/poetry-api`（诗泉 API 调用指南 + 完整端点文档 `references/api-docs.md`）
- 外部服务：诗泉古诗词 API（https://poetry.palemoky.com），免费开源，数据源自 chinese-poetry

## 安装

将专家包目录放到专家目录下：

```
C:\Users\feiyu\.workbuddy\plugins\marketplaces\my-experts\plugins/poetry-mentor/
```

然后运行注册命令使其可见：

```bash
python3 scripts/register_expert.py <expert-dir>
```

## 打包分享

```bash
zip -r poetry-mentor.zip poetry-mentor/
```
