---
name: trip-map-builder
description: >
  Use when planning a trip from destination selection through itinerary and
  maps, or when the destination is already chosen and the user asks to 细化行程,
  research restaurants, create a 行程地图, or build a trip map page.
---

# Trip Map Builder

完整行程按 `references/旅行规划流程.md` 的五步流程执行；目的地已定时从
Part 4 的细化排程接入下方 **Plan → Research → Build** 三阶段。

The output is a **reference itinerary**, not a script the traveler must obey.
During the trip, weather, current location, fatigue, and hunger can override
the original plan.

## 启动模式

开始规划前，先确定本次模式。用户已指定模式时直接采用；未指定时只问
一次：**“这次选自动走完，还是交互式逐步确认？”** 等待用户选择后启动。
模式只决定过程中的确认频率，不改变事实核查、文件归档和交付物。

- **自动走完**：依据已有画像、本次约束和实时调研，自行完成方向筛选、
  删点、排程和餐饮选择，连续推进五步流程及三阶段交付；在最终结果中
  说明关键取舍及依据。缺少会改变目的地或日期的硬约束、关键地理位置
  无法核实时，集中提问。对外发布、购买或预订仍按实际授权处理。
- **交互式**：五步流程的方向收敛、方案拍板，以及下方各闸门都由用户
  确认后推进。用户已明确给出的选择直接生效，不重复提问。

流程文件和生成资源都在本包内。旅行目录优先使用用户指定位置，其次使用
当前工作区中已有旅行资料的目录。旅行者画像和当次出行 README 是用户
数据，从旅行目录读取；不存在时按 `references/旅行规划流程.md` Part 1
收集必要信息，并在旅行目录中建立。

## 交互式闸门

以下闸门只在交互式模式启用。每道闸门停下来和用户对齐后继续：

- **闸门 0（Phase 1 开头）**：抽完硬约束、分完组后，先把"我理解的约束 +
  初步分组 + 打算砍掉的点"摆给用户确认。此阶段只做轻量判断，深度调研
  等骨架确认后再启动——避免把调研成本花在会被砍掉的点上（与上游 2.2
  的精神一致）。
- **闸门 1（删点确认）**：所有删点走四拍格式确认，明说删了什么、为什么；
  用户有权捞回。
- **闸门 2（Phase 2 前）**：行程骨架（每天主区域 + 锚点）经用户过目后，
  才启动大众点评/小红书重调研。
- **闸门 3（Phase 3 前）**：调研结论压缩成候选摆给用户选（餐厅 2-3 选 1、
  可选项去留），确认定稿后才生成三件交付物。
- 提问参考 `references/trip-planning.md` 的四拍格式；已知信息直接使用。

## 画像体系

偏好记忆由画像文件承担，skill 不另建记忆文件。开始规划前检查当前旅行目录；
存在对应文件时读取：

1. `旅行者画像.md` — 索引，确认本次出行人
2. 每个出行人的 `画像_<人>.md` — 五节画像（基本属性 /
   关系 / 偏好 / 去过的地方 / 原始 fact）
3. 当次出行文件夹的 `README.md` — 本次已拍板的方向、决策理由、硬约束

**多人约束求交**：体能按最弱成员排日程，天气敏感与饮食禁忌取并集，
拥挤容忍按最低者。画像之间冲突时明说冲突、给用户四拍选项。

画像文件不存在或缺人时，只补当前决策所需的信息。画像只收录跨次
稳定的信息；与本次目的地绑定的判断写入当次出行 README。

每次行程定稿后：
- 跨次稳定的新偏好 → 写回对应 `画像_<人>.md`
- 本次产物索引（表格/地图/部署地址）、决策理由 → 写进当次出行 README

## Phase 1: Plan the itinerary

Read `references/trip-planning.md` for the full methodology.

Core sequence:

1. **Extract hard constraints** — dates, flight times, terminals, hotel
   location; cross-check against 当次 README 里已拍板的方向
2. **Group user's wishlist** — city-easy / needs-reservation / far-suburbs / pass-through
3. **Cut high-risk items first** — too far, holiday-crowded, weather-dependent. Say what was cut and why.

   → 交互式：过闸门 0/1。自动走完：记录取舍并继续。
4. **Arrange by area** — one main area per day, first day light, last day close to airport

   → 交互式：过闸门 2。自动走完：进入 Phase 2。
5. **Fill in meals** — daily-area candidates first, 大众点评 + 小红书 signals second, fame last.
6. **Add tickets & transport** — only critical ones (museum tickets, airport transfer)
7. **Write reference doc** — conclusion first, then daily plan, weather-sensitive spots, meal areas, and what was cut

Key principles:
- Not everything the user listed fits. Delete for them.
- One area per day. One reservation-required spot per day max.
- Itineraries should be smooth, not packed.
- The plan gives coordinates for later adjustment; it does not pretend reality will follow the timeline.
- 需要提问时参考 `references/trip-planning.md` § 用户交互，合并独立问题。

## Phase 2: Research via 大众点评 + 小红书

Read `references/dianping-research.md` for the 大众点评 OpenCLI workflow.
Read `references/xhs-research.md` for the 小红书 OpenCLI + CDP workflow.

For restaurants, use 大众点评 as the main Chinese dining signal for taste,
queue risk, value, and obvious traps. Use 小红书 to supplement atmosphere,
recent experience, photo-worthiness, and soft warnings. Do not bend a whole
day around a famous restaurant unless it is already on the route.

小红书 core sequence:

1. Launch Chrome with `--remote-debugging-port=9223`
2. Connect via OpenCLI's `CDPBridge`
3. Navigate to `xiaohongshu.com/search_result?keyword=<encoded>` (never simulate input box)
4. Intercept `POST /api/sns/web/v1/search/notes` response
5. Pick top 2-3 notes by relevance, open detail pages
6. Extract via DOM: `#detail-title`, `#detail-desc`, `.author-container .username`
7. Compress to one decision-useful sentence per store, write back to local `.md`

Filtering rules:
- Keep: specific store name, address, dish, personal experience, repeated keywords
- Drop: generic area roundups, reposts, pure emotion, "氛围很好" x3
- Output: store name + one representative link + 2-3 sentence verdict

## Phase 3: Build the deliverables

→ 交互式先过闸门 3；自动走完模式依据调研自行选定并生成。

交付物固定为三件，数据同源（同一份结构化逐日数据），全部进旅行目录下
的当次出行文件夹 `YYYY_<事件>/`：

1. **行程总表** — 按天排列，覆盖住宿 + 景点 + 重要餐食时间线 + 预期价格，
   作为执行和预算核对的依据。Markdown 表格，写入当次出行文件夹。
2. **手绘旅行地图** — 3:4 竖版蜡笔/彩铅风格单图，用于手机查看和打印。
   生成 prompt 见 `references/hand-drawn-map-prompt.md`。生成前先核对
   地理位置，无法核实的关键地点先向用户要地图截图或定位。
3. **交互式地图页面** — 旅途中查位置、跳转导航/小红书/大众点评用：

   1. Copy `assets/template.html` → `index.html`
   2. Fill `HOTEL` object and `DAYS` array with structured data from Phase 1+2
   3. Each location needs: name, lat/lng, type, time, desc; optional: budget, detail, pay, xhs, reserve, gmap
   4. Fill `overviewContent()` with trip summary, payment warnings
   5. Apply design system — default template uses Apple style, but can switch to any style from awesome-design-md

Location types: `food` | `spot` | `drink` | `hotel` | `transport`

Payment chip values: `1` = confirmed yes (green), `0.5` = maybe (orange), omit = not shown

### Design system (optional)

Default template uses Apple design system (SF Pro, light theme, frosted glass).

To use a different style, grab a `DESIGN.md` from [awesome-design-md](https://github.com/VoltAgent/awesome-design-md):

```bash
# Browse available design systems
# Apple, Vercel, Linear, Stripe, Notion, Airbnb, Nike, Spotify, etc.
curl -O https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md/<brand>/DESIGN.md
```

Then adjust `template.html`'s `:root` CSS variables (colors, fonts, spacing, border-radius) to match the chosen DESIGN.md tokens.

### Deploy (optional)

```bash
git init && git add . && git commit -m "trip map"
gh repo create REPO --public --source=. --push
# Import from vercel.com/new — auto-deploys on push
```

## Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| [OpenCLI](https://github.com/jackwener/OpenCLI) | 大众点评 adapter + 小红书调研 | `npm install -g @jackwener/opencli` |
| Chrome/Chromium | 浏览器 + 远程调试 | 已有 |
| [Leaflet.js](https://leafletjs.com) | 地图渲染（CDN 引入，无需安装） | template.html 内置 |
| [gh CLI](https://cli.github.com) | GitHub 仓库创建（可选） | `brew install gh` |

## Resources

- `references/旅行规划流程.md` — 从画像、目的地选择到交付的完整五步流程；目的地未定时先读
- `references/trip-planning.md` — itinerary planning methodology, input/output templates, selection principles, common pitfalls
- `references/dianping-research.md` — 大众点评 OpenCLI search/shop workflow, dining decision signals, writeback format
- `references/xhs-research.md` — OpenCLI installation, Chrome CDP setup, 小红书 search workflow, API details, filtering criteria
- `references/hand-drawn-map-prompt.md` — 手绘旅行地图生成 prompt
- `assets/template.html` — single-file HTML map template (Leaflet + Apple design system)
- [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) — 60+ brand design systems (Apple, Vercel, Stripe, Linear, etc.) for alternative styling
