---
name: trip-map-builder
description: >
  Trip execution detailing after the destination is decided: extract hard
  constraints, cut high-risk items, arrange by area, research locations and
  dining signals via 大众点评 + 小红书, then produce three deliverables —
  行程总表, 手绘旅行地图, and an interactive mobile-first map page (Leaflet +
  timeline, optionally deployed to Vercel). Use when the destination/方向 is
  already settled (typically via the upstream 旅行规划流程 Part 1-3 funnel)
  and the user asks to 细化行程, research restaurants on 大众点评/小红书,
  build a trip map page, or says "行程细化", "行程地图", "trip map", "做个行程图".
  If the destination is still undecided, do NOT use this skill for
  destination selection — that belongs to 旅行规划流程.md Part 2-3.
---

# Trip Map Builder

Three-phase pipeline: **Plan → Research → Build**.

The output is a **reference itinerary**, not a script the traveler must obey.
During the trip, weather, current location, fatigue, and hunger can override
the original plan.

## 适用范围

本 skill 承接**目的地已定之后**的执行细化。选目的地的漏斗（宏观扫描 →
大方向 → 定向深挖 → 思路发散 → 拍板）在 `~/Qbsidian/Travel/旅行规划流程.md`
Part 1-3，默认先走那边；那边的结论沉淀在当次出行文件夹
`~/Qbsidian/Travel/YYYY_<事件>/README.md`。

用户直接要求做行程且目的地未定时，先回到旅行规划流程 Part 2 完成方向
收敛，再进 Phase 1。

## 入口提醒（首次调用时，建议不强制）

skill 被调用时，先判断本次上下文是否来自统一入口
`~/Qbsidian/Travel/旅行规划流程.md`：当次出行文件夹已有拍板结论
（README 里记录了已选方向/思路），或用户明确表示是接着流程走来的，
视为来自入口，**跳过提醒**。

否则（例如用户在其他对话/目录里直接说"帮我规划行程"），先给一句轻量
提醒再开工：

> 建议从统一入口 `~/Qbsidian/Travel/旅行规划流程.md` 开始规划（画像 +
> 方向漏斗 + 当次出行文件夹都在那边沉淀），本次要继续在这里做也可以。

提醒只出现一次、一句话，用户说继续就直接进 Phase 1，不追问、不阻断。

## 交互闸门（不确定先讨论）

三阶段流水线逐闸门推进。每道闸门停下来和用户对齐，得到确认后再往下
走；拿不准的一律先摆出来讨论，由用户拍板：

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
- 全程提问用四拍格式（Re-ground → Simplify → Recommend → Options），
  用户指令已含的信息按 smart skip 跳过。

## 画像体系

偏好记忆由画像文件承担，skill 不另建记忆文件。开始 Phase 1 前必读：

1. `~/Qbsidian/Travel/旅行者画像.md` — 索引，确认本次出行人
2. 每个出行人的 `~/Qbsidian/Travel/画像_<人>.md` — 五节画像（基本属性 /
   关系 / 偏好 / 去过的地方 / 原始 fact）
3. 当次出行文件夹的 `README.md` — 本次已拍板的方向、决策理由、硬约束

**多人约束求交**：体能按最弱成员排日程，天气敏感与饮食禁忌取并集，
拥挤容忍按最低者。画像之间冲突时明说冲突、给用户四拍选项。

画像文件不存在或缺人时，通过对话补齐并写回画像文件。画像只收录跨次
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

   → **停：闸门 0/1**。把约束理解、分组、砍点清单摆给用户确认后再继续。
4. **Arrange by area** — one main area per day, first day light, last day close to airport

   → **停：闸门 2**。骨架给用户过目后才进 Phase 2 调研。
5. **Fill in meals** — daily-area candidates first, 大众点评 + 小红书 signals second, fame last.
6. **Add tickets & transport** — only critical ones (museum tickets, airport transfer)
7. **Write reference doc** — conclusion first, then daily plan, weather-sensitive spots, meal areas, and what was cut

Key principles:
- Not everything the user listed fits. Delete for them.
- One area per day. One reservation-required spot per day max.
- Itineraries should be smooth, not packed.
- The plan gives coordinates for later adjustment; it does not pretend reality will follow the timeline.
- All user-facing questions follow the **4-beat format**: Re-ground → Simplify → Recommend → Options. See `references/trip-planning.md` § 用户交互 for examples and anti-patterns.

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

→ **先过闸门 3**：调研结论已压缩成候选并经用户选定、行程定稿后，才生成。

交付物固定为三件，数据同源（同一份结构化逐日数据），全部进当次出行
文件夹 `~/Qbsidian/Travel/YYYY_<事件>/`：

1. **行程总表** — 按天排列，覆盖住宿 + 景点 + 重要餐食时间线 + 预期价格，
   作为执行和预算核对的依据。Markdown 表格，写入当次出行文件夹。
2. **手绘旅行地图** — 3:4 竖版蜡笔/彩铅风格单图，用于手机查看和打印。
   生成 prompt 固化在 `references/hand-drawn-map-prompt.md`（与
   旅行规划流程.md Part 5 同步，改一边要同步另一边）。生成前先核对
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

- `references/trip-planning.md` — itinerary planning methodology, input/output templates, selection principles, common pitfalls
- `references/dianping-research.md` — 大众点评 OpenCLI search/shop workflow, dining decision signals, writeback format
- `references/xhs-research.md` — OpenCLI installation, Chrome CDP setup, 小红书 search workflow, API details, filtering criteria
- `references/hand-drawn-map-prompt.md` — 手绘旅行地图固化生成 prompt（与旅行规划流程.md Part 5 双向同步）
- `assets/template.html` — single-file HTML map template (Leaflet + Apple design system)
- [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) — 60+ brand design systems (Apple, Vercel, Stripe, Linear, etc.) for alternative styling
