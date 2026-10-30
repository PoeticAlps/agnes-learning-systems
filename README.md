# 教学系统原型集（agnes-learning-systems）

AI + 教育方向的三个教学系统原型。每个原型都是**自包含的单文件 HTML**，无构建步骤、无外部依赖（不引用任何 CDN），可直接双击打开，或经本地 HTTP 服务访问：

```bash
cd agnes-learning-systems
python3 -m http.server 8080
# 浏览器打开 http://localhost:8080/
```

入口页 `index.html` 汇总了三个原型的卡片与说明。

## 目录结构

```
agnes-learning-systems/
├── index.html                     # 入口页：三张原型卡片（自包含，无 CDN）
├── A3_多智能体学习系统.html         # 原型 1：多智能体个性化学习系统（静态交互原型）
├── A5_景区导览数字人.html           # 原型 2：景区导览数字人“小桂”（桂林山水 AI 导游原型）
├── 知声_语音知识图谱.html           # 原型 3：语音知识图谱学习伴侣（演示模式，内置 Mock）
└── README.md
```

## 三个原型简介

### 1. A3 · 多智能体学习系统 —— 给谁用：想体验“多 Agent 协作辅导”的教学演示场景

- **是什么**：AI 学习伴侣聊天界面。内置知识库 + 关键词路由，模拟导师、知识图谱、评估、规划四类 Agent 的协作回答，侧边栏有能力雷达图、学习路径、推荐资源。
- **怎么打开**：直接用浏览器打开 `A3_多智能体学习系统.html`，或从 `index.html` 点击“打开原型”。
- **性质**：静态交互原型，纯前端，无后端依赖。快捷按钮或输入包含“函数 / 思维导图 / 练习 / 路径 / 评估”等关键词可触发对应 Agent。

### 2. A5 · 景区导览数字人“小桂” —— 给谁用：智慧文旅 / 景区导览场景的教学演示

- **是什么**：桂林山水 AI 导游数字人。支持景点介绍、半日/一日/两日路线规划、美食与摄影攻略问答，数字人形象带情绪表情与动作反馈。
- **怎么打开**：直接用浏览器打开 `A5_景区导览数字人.html`，或从 `index.html` 进入。
- **性质**：静态交互原型，纯前端。语音按钮依赖浏览器麦克风权限（可选功能，拒绝授权不影响演示）。

### 3. 知声 · 语音知识图谱 —— 给谁用：课堂教学记录 → 知识点沉淀 → 复习巩固的学习者

- **是什么**：语音知识图谱学习伴侣，五个页签：录音管理（上传→转写→提取知识点）、知识图谱（节点/边）、卡片复习（间隔重复）、智能问答（引用录音原文）、学习统计（含掌握度趋势图）。
- **怎么打开**：直接用浏览器打开 `知声_语音知识图谱.html`，或从 `index.html` 进入。默认处于**演示模式**，顶部有橙色“演示模式 · Mock 数据”徽章，无需任何后端即可走通全流程。
- **演示模式下的体验**：
  - 上传任意音频文件 → 界面插入“转写中…”条目，约 1.5 秒后变为“已转写”并展示固定转写文本（《旅游心理学·旅游动机》课堂片段）；
  - “提取知识点”按钮返回 4 个知识点，与知识图谱页数据一致；
  - 知识图谱页内置《旅游心理学》《导游业务》《酒店管理概论》三门课的节点与关系边（切换课程下拉框查看）；
  - 智能问答按关键词命中返回带“录音来源”的答案（试试问“老师上课说的推力因素包括哪些？”）；
  - 复习卡反馈会实时更新学习统计（复习次数、掌握率、今日待复习）。

## 知声：Mock / 真实后端切换

Mock 逻辑全部位于 `知声_语音知识图谱.html` 内联 `<script>` 顶部，原理：`USE_MOCK = true` 时用 `mockFetch` 包装 `window.fetch`，拦截所有指向 `API` 的请求并返回内置数据；其余请求原样放行。**原有 fetch 调用逻辑未做任何改写。**

接入真实后端：

1. 打开 `知声_语音知识图谱.html`，将脚本顶部 `const USE_MOCK = true` 改为 `false`（顶部“演示模式”徽章会自动隐藏）；
2. 将 `const API = 'http://localhost:8000/zhisheng'` 改为真实后端地址；
3. 后端需实现以下接口（请求/响应结构与 mock 返回一一对应，也可直接参考 `mockRoute` 函数）：

| 接口 | 方法 | 请求 | 响应 |
| --- | --- | --- | --- |
| `/upload_audio` | POST | FormData: `file`, `student_id`, `course_name` | `{recording_id, status, course_name, transcript}` |
| `/extract_knowledge/{recording_id}` | POST | - | `{recording_id, points: [{name, type, summary}]}` |
| `/knowledge_graph/{course}` | GET | - | `{nodes: [{id, label, type}], edges: [{source, target, relation}]}` |
| `/review_cards/{student_id}` | GET | - | `[{card_id, card_type, front, back}]` |
| `/review` | POST | JSON: `card_id`, `student_id`, `feedback` | `{ok, ...}` |
| `/ask` | POST | JSON: `student_id`, `question` | `{answer, sources: [{title, snippet}]}` |
| `/stats/{student_id}` | GET | - | `{total_reviews, mastery_rate, streak_days, cards_due_today}` |

> 其中 `/upload_audio`、`/review_cards/{id}`、`/review`、`/ask`、`/stats/{id}`、`/knowledge_graph/{course}` 为页面原有的 6 个接口；`/extract_knowledge/{id}` 是本次修复的按钮缺失函数，与后端约定接口时需一并实现。

## 备注

- 三个原型相互独立、互不依赖，可单独摘出部署。
- 页面数据（课程内容、景点信息等）均为编造的教学演示内容。
