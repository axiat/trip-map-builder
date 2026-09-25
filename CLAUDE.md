# trip-map-builder - 旅行地图技能包
Markdown + HTML + OpenCLI references

<directory>
assets/ - 单文件地图模板 (1文件: template.html)
references/ - 五步旅行流程、调研与规划方法论 (5文件: 旅行规划流程.md, trip-planning.md, dianping-research.md, xhs-research.md, hand-drawn-map-prompt.md)
</directory>

<config>
README.md - 对外说明技能定位、安装、流程和目录结构
SKILL.md - Agent 技能入口，定义启动模式、画像体系、五步流程和三阶段细化
</config>

法则: 行程是参考坐标，旅途中的天气、位置、体力可以覆盖原计划。偏好读用户旅行目录中的画像体系，skill 不另建记忆。餐厅先看当天区域，再看大众点评和小红书。本包覆盖目的地选择到三交付物（行程总表 + 手绘地图 + 交互式地图页面）。启动时选自动走完或交互式；交互式逐闸门对齐，自动走完记录取舍后连续推进。

[PROTOCOL]: 变更时更新此头部，然后检查 CLAUDE.md
