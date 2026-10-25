# ⚡ FitLog Pro

单文件、纯前端、本地优先的训练与健康记录工具。下载 `index.html`，双击用浏览器打开即可使用，无需安装、无需后端、无需构建。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 1. 项目解决什么问题

日常训练记录通常散落在备忘录、Excel 或聊天记录里，带来几个具体问题：

- **看不到进步**：练了三个月深蹲，说不清到底涨了多少。
- **重复劳动**：每次训练都要重新输入同样的动作名称和组数。
- **训练与恢复割裂**：训练记录在一个地方，睡眠、心率、步数在另一个地方，无法放在一起判断。
- **反馈滞后**：想获得专业分析，但请教练成本高，自己又缺少判断框架。

FitLog Pro 针对这四点做了取舍：

- 用 **e1RM 估算 + PR 检测** 让进步可见
- 用 **偏好学习 + 训练模板** 减少重复输入
- 把 **训练、心率、睡眠、恢复、体重、饮食** 放在同一数据模型里，趋势页可对比
- 用 **可配置的 AI 分析**（用户自备 API Key）按运动科学框架生成反馈，并尝试引入近期训练科学 RSS 资讯作为参考上下文

**明确的边界**：所有数据只存在当前浏览器的 `localStorage` 中，不经过任何服务器；AI 分析由用户主动触发并直连用户填写的 API Endpoint。

---

## 2. 主要功能

### 2.1 训练记录

- **训练分类**：体能训练 / 专项训练 / 恢复放松
- **动作维度**：力量 / 有氧 / 爆发力 / 柔韧 / 平衡
- **动作录入类型**：

  | 类型 | 可填字段 |
  |---|---|
  | 力量 | 组数、次数、重量(kg)、RIR(0–5) |
  | 有氧 | 时长(分钟)、距离(km) |
  | 计时 | 组数、每组秒数 |
  | 自由 | 文本描述 |

- **训练级字段**：日期、主题、备注、平均心率、消耗(kcal)
- **e1RM 实时估算**：力量动作填写重量和次数后，自动用 Epley 公式 `e1RM = weight × (1 + reps/30)` 显示估算值
- **PR 检测**：对比该动作历史最高 e1RM，若新记录更高或为首次记录，显示「🏆 新 PR」徽章；记录列表中破 PR 的动作会带 PR 标记
- **上次记录参考**：动作名称匹配到历史记录时，下方显示「上次：X 天前 · 60kg × 3 组 × 8 次（RIR 2）」；重量/组数/次数输入框在为空时用上次值作为 placeholder

### 2.2 休息计时器

- 每个力量动作旁有「⏱ 休息 90s」按钮，点击后可输入自定义秒数（默认 90）
- 页面底部出现倒计时浮层，可随时停止；倒计时结束时通过 Web Audio 播放提示音

### 2.3 偏好学习

- 每次保存训练，累加动作和维度的使用次数（`fitlog-pref-v1`）
- 训练表单下方和设置页展示最高频的 8 个动作，点击可直接追加到当前训练
- **删除或编辑训练时回滚计数**：每个 session 保存 `learnedEntries`，删除/编辑时先减去旧计数再重新学习，避免计数虚高

### 2.4 自定义动作库

- 设置页可添加自定义动作，并指定所属维度（力量/有氧/柔韧/平衡/爆发力）
- 自定义动作会进入下拉建议、偏好统计和维度归类
- 可逐条删除

### 2.5 训练模板

- 训练表单点「保存为模板」，把当前动作列表（名称、维度、类型）存为模板
- 下次从「从模板创建…」下拉选择，自动填入动作列表
- **模板只保存动作结构，不保存具体重量/组数/次数**

### 2.6 草稿自动保存

- 训练表单变动后 800ms 防抖写入 `sessionStorage`（`fitlog-draft-v1`）
- 重新打开页面时若检测到草稿，询问是否恢复
- 保存成功后自动清除草稿
- **关闭标签页后草稿即丢失**（`sessionStorage` 特性）

### 2.7 删除撤销

- 训练记录删除采用软删除（标记 `deletedAt`），toast 中提供 6 秒内「撤销」按钮
- **健康数据的删除没有撤销**，确认后直接删除

### 2.8 健康数据

- 每日可记录：体重、静息心率、平均心率、最高心率、活动消耗、总消耗、步数、睡眠时长、深睡时长、恢复放松分钟数、饮食摄入 kcal、数据来源、备注
- 数据来源字段为标记用途（手动 / Apple 健康 / 小米 / 智能设备），**不执行任何设备同步**
- 健康记录列表支持编辑与删除

### 2.9 趋势图

- **指标（10 种）**：静息心率、平均心率、步数、睡眠时长、活动消耗、体重、训练次数、训练组数、训练容量、**动作 e1RM**
- **范围**：7 / 14 / 30 / 90 天
- Canvas 折线图 + 平均/最高/最低统计
- 选择「动作 e1RM」时出现动作选择器，展示该动作在所选区间内每天的最高 e1RM

### 2.10 AI 分析

- 用户填写 **API Endpoint（OpenAI Chat Completions 兼容）、API Key、模型名**，保存在本机 `localStorage`（`fitlog-ai-settings`）
- 选择分析范围（7/14/30 天）与自然语言「分析焦点」
- 程序自动组装提示词，包含：
  - 该区间内全部训练记录（含分类、主题、动作明细、平均心率、消耗）
  - 该区间内全部健康数据
  - 高频动作 Top 10
  - 近期训练科学 RSS 摘要（最多 5 条，每条截取 120 字）
  - 用户的分析焦点
- 系统提示词要求 AI 从训练量与频率、强度与恢复、结构平衡性、偏好分析、可操作建议、知识依据六个角度输出
- 分析结果保存到历史（本机最多 50 条，界面显示最近 10 条，可展开对比）

### 2.11 每日训练科学资讯

- 从 3 个 RSS 源拉取：
  - Stronger by Science
  - PubMed Sports Med（resistance training 检索）
  - ACSM
- 通过公共 CORS 代理 `api.allorigins.win` 访问（**依赖该第三方服务可用**）
- 每个源最多取 3 条 7 天内的条目，去重后缓存到 `localStorage`（最多 200 条）
- 知识页展示标题、来源、摘要、原文链接；摘要去 HTML 后截取 300 字
- 手动「刷新」按钮触发重新拉取

### 2.12 导入导出

- **导出**：JSON 备份，包含 `sessions`、`daily`、`pref`、`customExercises`、`templates`、`aiHistory`
- **导入**：
  - 若 JSON 含 `sessions` 或 `daily` 字段，视为完整备份，确认后覆盖当前数据
  - 若 JSON 是数组，视为健康数据数组，按日期合并 `steps / hrRest / hrAvg / hrMax / sleep / calActive / rest / weight / calIntake` 字段
- **不支持 Apple 健康 `export.xml`、CSV 或蓝牙设备直连**（见「已知限制」）

### 2.13 其他

- **移动端适配**：窗口 ≤720px 时顶部标签隐藏，改为底部导航（今日 / 训练 / 健康 / 更多），「更多」弹出趋势、AI、知识、设置的入口
- **数据版本迁移**：主数据带 `version: 4`，加载时对旧版本数据补充 `learnedEntries` 字段
- **URL hash 直达**：`#train`、`#trend`、`#ai` 等可直接打开对应视图
- **PWA 支持**：`index.html` 中已引用 `manifest.json` 并注册 `sw.js`，通过 HTTP(S) 访问时可安装到桌面/主屏并离线缓存

---

## 3. 安装方法

本项目是**单文件应用**，没有依赖、没有构建步骤。

### 方式 A：本地直接使用（推荐）

1. 下载 `index.html`
2. 双击用浏览器打开（Chrome / Edge / Safari / Firefox 均可）
3. 开始使用。数据自动保存在当前浏览器的 `localStorage`

因为是单文件且不使用 ES Module，`file://` 协议下可直接运行，不需要启动 HTTP 服务器。

> 在 `file://` 协议下 `sw.js` 不会注册，PWA 安装和离线缓存不可用；其余功能全部正常。

### 方式 B：部署到 GitHub Pages

1. 把本仓库（含 `index.html`、`manifest.json`、`sw.js`、`icons/`）推送到 GitHub
2. 仓库 Settings → Pages → Source 选 `Deploy from a branch` → `main` / `root` → 保存
3. 约 1 分钟后访问 `https://<用户名>.github.io/<仓库名>/`
4. 首次加载后 Service Worker 会缓存静态资源，之后离线可打开

### 方式 C：本地 HTTP 服务

```bash
# Python
python3 -m http.server 8080

# Node.js
npx serve .
访问 http://localhost:8080，可完整体验 PWA 安装与离线缓存。

环境要求
现代浏览器（支持 localStorage、fetch、canvas、AudioContext、DOMParser、Service Worker）

AI 分析和 RSS 拉取需要联网

AI 分析需要自备 OpenAI 兼容的 API Key

4. 使用方法
4.1 首次使用流程
打开页面，默认在「今日」页

切到「训练」页，填写日期、选择训练分类

在动作条目里输入动作名称（可从下拉建议中选择，或直接输入自定义名称）

按动作性质填写字段：

力量动作填组数、次数、重量，可选 RIR；系统实时显示 e1RM 和 PR 状态

有氧动作填时长、距离

计时动作填组数和每组秒数

自由动作填文字描述

点「保存训练」

4.2 日常记录流程
重复上一次训练：训练页动作名称输入后，下方会显示上次记录；重量/组数/次数输入框会预填上次值作为 placeholder，直接 Tab 确认即可

使用模板：点「从模板创建…」选择已保存模板，自动填入动作列表，只需填数值

快速添加常用动作：点击表单下方的偏好芯片，直接追加该动作条目

组间休息：做完一组后点「⏱ 休息 90s」，倒计时结束会响铃

记录健康数据：切到「健康」页填写当天数据；或在「今日」页点「编辑今日数据」跳转

4.3 AI 分析
切到「AI 分析」页

填写 Endpoint（如 https://api.openai.com/v1/chat/completions）、API Key、模型名，点「保存 Key」

选择分析范围，可选填分析焦点（如「我的下肢力量是否有进步？」）

点「生成分析」，等待返回；结果会显示在输出区，并自动存入「历史分析」

历史条目点击标题可展开/收起

4.4 每日知识
切到「知识」页

首次打开或点击「刷新」会通过 allorigins 代理拉取 3 个 RSS 源

每条显示来源、标题、摘要和「阅读原文」链接

拉到的最新条目会作为上下文注入后续的 AI 分析

4.5 数据备份
顶部「导出」或设置页「导出全部数据」：下载 JSON 备份

顶部「导入」：选择之前导出的 JSON，确认后覆盖

设置页「清空全部数据」：确认后清空训练、健康、偏好、自定义动作、模板和 AI 历史

5. 输入输出示例
以下示例数据均为虚构，仅用于演示数据结构。

5.1 训练记录：输入 → 存储
界面输入

字段	值
日期	2026-10-25
分类	体能训练
主题	下肢力量
动作 1	深蹲 / 力量 / 4 组 / 8 次 / 80 kg / RIR 2
动作 2	罗马尼亚硬拉 / 力量 / 3 组 / 10 次 / 60 kg / RIR 3
平均心率	128
消耗	420
备注	状态不错
保存后 session 对象（示意）

json
{
  "id": "m1abc2def",
  "createdAt": 1761436800000,
  "date": "2026-10-25",
  "category": "体能训练",
  "title": "下肢力量",
  "note": "状态不错",
  "hrAvg": 128,
  "calBurned": 420,
  "entries": [
    {
      "id": "e1", "name": "深蹲", "type": "strength", "dim": "力量",
      "sets": 4, "reps": 8, "weight": 80, "rir": 2
    },
    {
      "id": "e2", "name": "罗马尼亚硬拉", "type": "strength", "dim": "力量",
      "sets": 3, "reps": 10, "weight": 60, "rir": 3
    }
  ],
  "learnedEntries": [
    { "name": "深蹲", "dim": "力量" },
    { "name": "罗马尼亚硬拉", "dim": "力量" }
  ]
}
界面输出

动作条目实时显示：e1RM ≈ 101 kg（深蹲，80 × (1 + 8/30) ≈ 101.3）

若该动作历史最高 e1RM 低于 101.3，显示「🏆 新 PR」

保存后记录列表中该动作带 🏆 PR 标记

5.2 健康数据：输入 → 存储
界面输入

字段	值
日期	2026-10-25
体重	70.5 kg
静息心率	56 bpm
平均心率	72 bpm
最高心率	145 bpm
活动消耗	420 kcal
总消耗	2100 kcal
步数	8600
睡眠	7.5 h
深睡	1.8 h
恢复放松	30 min
饮食摄入	2200 kcal
数据来源	手动记录
存储对象（示意）

json
{
  "date": "2026-10-25",
  "hrRest": 56, "hrAvg": 72, "hrMax": 145,
  "calActive": 420, "calTotal": 2100, "steps": 8600,
  "sleep": 7.5, "sleepDeep": 1.8, "rest": 30,
  "weight": 70.5, "calIntake": 2200,
  "source": "manual", "note": "",
  "meals": { "b": "", "l": "", "d": "", "s": "" }
}
meals 字段在当前版本中保留于数据结构里，但界面未提供输入入口，因此始终为空对象。

5.3 偏好学习：输入 → 输出
输入：连续 3 次训练都记录了「深蹲」，2 次记录了「卧推」。

输出：训练表单下方偏好面板显示

text
深蹲 (3)   卧推 (2)
点击「深蹲」芯片，直接向当前训练追加一个「深蹲 / 力量 / strength」空条目。

回滚示例：删除那 3 次训练中的 1 次后，「深蹲」计数变回 2。

5.4 AI 分析：输入 → 输出
用户输入

Endpoint：https://api.openai.com/v1/chat/completions

模型：gpt-4o-mini

分析范围：14 天

分析焦点：我的下肢力量是否有进步？

程序自动组装（节选，真实发送的 user message 结构）

text
请分析以下数据（2026-10-12 至 2026-10-25，共 14 天）。

【训练记录】
- 2026-10-15 体能训练 / 下肢力量：深蹲（4 组 × 8 次 @ 80 kg · RIR 2）；罗马尼亚硬拉（3 组 × 10 次 @ 60 kg · RIR 3） [平均心率 128 bpm] [消耗 420 kcal]
- 2026-10-22 体能训练 / 下肢力量：深蹲（4 组 × 8 次 @ 82.5 kg · RIR 2）；...

【健康数据】
- 2026-10-15：静息心率 56 bpm，睡眠 7.5 h，步数 8600，...
- ...

【高频动作】深蹲(4次)、卧推(3次)、罗马尼亚硬拉(2次)

【近期训练科学动态（供参考）】
- [Stronger by Science] ...：...
- ...

【用户特别关注】我的下肢力量是否有进步？

请生成分析报告。
输出：AI 返回的中文分析文本，显示在输出区，并存入历史。历史条目可展开查看完整内容。

5.5 每日知识：输入 → 输出
输入：点击「刷新」

过程：依次请求

text
https://api.allorigins.win/raw?url=https%3A%2F%2Fwww.strongerbyscience.com%2Ffeed%2F
https://api.allorigins.win/raw?url=https%3A%2F%2Fpubmed.ncbi.nlm.nih.gov%2F%3Fterm%3Dsports%2Bmedicine%2Bresistance%2Btraining%26format%3Drss
https://api.allorigins.win/raw?url=https%3A%2F%2Fwww.acsm.org%2Ffeed
输出：解析 XML 后按 7 天内过滤、每源最多 3 条、去重，每条渲染为：

text
[Stronger by Science] 标题
摘要（去 HTML，最多 300 字）
2026-10-24    阅读原文 →
若代理不可用或所有源都失败，toast 提示「未获取到新资讯，可稍后重试」。

5.6 导出 JSON 结构
json
{
  "app": "FitLogPro",
  "version": 4,
  "exportedAt": "2026-10-25T08:00:00.000Z",
  "sessions": [ /* 训练记录数组 */ ],
  "daily": { "2026-10-25": { /* 健康数据 */ } },
  "pref": { "dimCount": {}, "exCount": {}, "lastUsed": {} },
  "customExercises": { "地雷管推举": "力量" },
  "templates": [ /* 模板数组 */ ],
  "aiHistory": [ /* AI 分析历史 */ ]
}
5.7 导入的两种格式
格式 A：完整备份（含 sessions 或 daily 字段）→ 覆盖当前数据。

格式 B：健康数据数组

json
[
  { "date": "2026-10-24", "steps": 10200, "hrRest": 58, "sleep": 8.0 },
  { "date": "2026-10-25", "steps": 8600,  "hrRest": 56, "sleep": 7.5 }
]
导入时按 date 合并到健康数据，只取 steps / hrRest / hrAvg / hrMax / sleep / calActive / rest / weight / calIntake 字段。

6. 已知限制
为避免误用，以下功能当前版本不包含：

不支持设备直连：没有 Web Bluetooth 心率连接，也不解析 Apple 健康 export.xml 或小米/华为的 CSV 导出。数据来源字段仅作标记。

知识拉取依赖第三方代理：api.allorigins.win 不可用、被墙或超时时，知识页为空，AI 提示词中也不包含知识上下文。

AI 分析需自备 Key：不内置任何 API 额度或代理，Key 保存在本机 localStorage，存在 XSS 场景下被读取的理论风险。

数据不跨设备自动同步：只能通过导出/导入 JSON 手动迁移。

PR 基于 e1RM 估算：使用 Epley 公式，未实测 1RM，高次数（>12 次）时估算偏差会增大。

健康数据删除不可撤销：训练删除有 6 秒撤销窗口，健康数据没有。

草稿不跨会话：使用 sessionStorage，关闭标签页即清除。

模板不保存数值：只保存动作结构，加载后需要重新填写重量/组数/次数。

趋势图无日历热力图：只有折线图。

PR 检测未排除软删除记录：理论上，6 秒撤销窗口内被软删除的记录仍可能参与 PR 对比。

饮食宏量营养素未实现：只有总热量摄入字段。

file:// 协议下 PWA 不可用：Service Worker 不会注册，需要通过 HTTP(S) 访问。

7. 数据与隐私
所有训练、健康、偏好数据保存在浏览器 localStorage，不经过任何服务器

导出的 JSON 含个人健康数据，请勿上传到公开仓库或分享给不受信任的第三方

AI 请求会把你选择日期范围内的训练与健康数据摘要发送到你填写的 Endpoint。请勿使用你不信任的第三方 Endpoint

API Key 存于本机 localStorage，存在 XSS 场景下被读取的理论风险

8. 训练分类依据（文献）
训练维度与强度参考以下资料：

Caspersen CJ, Powell KE, Christenson GM. Physical activity, exercise, and physical fitness: definitions and classifications for health-related research. Public Health Rep. 1985.

Bull FC, et al. World Health Organization 2020 guidelines on physical activity and sedentary behaviour. Br J Sports Med. 2020.

American College of Sports Medicine. ACSM's Guidelines for Exercise Testing and Prescription.

National Strength and Conditioning Association. NSCA's Essentials of Strength Training and Conditioning.（力量强度分级：高 >85%1RM，中 65–85%1RM，低 <65%）

ACSM 2026 抗阻训练指南相关结论：RIR 作为强度锚点优于 1RM 百分比；80% 1RM 区间对力量增长最有效。程序中的 RIR 字段与系统提示词措辞参考了该结论。

9. 文件结构
text
fitlog-pro/
├── index.html          # 主程序（单文件，含全部 HTML/CSS/JS）
├── manifest.json       # PWA 清单
├── sw.js               # Service Worker
├── README.md           # 本文件
├── LICENSE             # MIT
├── .gitignore
└── icons/
    ├── icon.svg
    └── icon-maskable.svg
License
本项目基于 MIT License 发布。你可以自由使用、修改、分发，包括商业用途，但需保留版权声明和许可声明。

完整条款见 LICENSE 文件。

text
