# Antigravity 总记忆库 (General Memory)

此文档记录通用的开发经验、协作偏好以及跨项目的技术沉淀，以便在每次开启新会话时能够瞬间找回最佳的开发状态。

---

## 1. 协作与开发习惯 (Development Preferences)

* **上下文快速还原**：由于每次新开会话的 Transcript 可能为空，开启新会话后的第一步必须是检查 `antigravity-memory` 仓库，并运行 `git status` / `git log -n 5` 以分析当前物理工作区的变动，以此精准定位开发进度。
* **架构设计原则**：
  * **高内聚低耦合**：优先将功能包装成独立的节点或组件（例如 `MysticRealmComponent`、`SpellComponent`），通过信号或纯数据接口与主类（如 `player.gd`）交互。
  * **数据驱动 (Data-Driven)**：倾向于使用 JSON、CSV 或本地常量配置表（如物品数据库 `ItemDatabase`、配方数据 `RECIPES`）定义属性与效果，避免将数据硬编码到逻辑代码中。
  * **用户体验至上 (UX & Details)**：增加微调动画（如受击闪烁、攻击突刺 Tween 动画）和人性化细节（如物品悬停 Tooltip 提示、飞行滑行阻力感）。

---

## 2. 核心技术栈与最佳实践 (Tech Stack & Practices)

* **引擎与语言**：Godot Engine 4.x，GDScript。
* **代码风格规范**：
  * 保持原有的 GDScript 变量与方法命名规范（下划线命名法，私有方法以下划线 `_` 开头）。
  * 严格保证文件和代码符号的可点击跳转格式（如 `[player.gd](file:///path/to/player.gd)`），方便查看。
  * 尽可能保留第三方插件与引擎的核心生成配置，只修改业务逻辑和核心自定义脚本。

---

## 3. 记忆维护与同步规则 (Memory Sync Rules)

* **更新时机**：每当完成一个重要的开发里程碑（如新功能跑通、重大架构重构）时，应主动更新本记忆库及对应项目的具体记忆文件。
* **版本打标**：结合 `antigravity-projects-git` 的版本 Tag 机制，在记忆库中同步记录该版本的技术设计方案与避坑指南。
