# Godot 3D RPG 项目记忆库 (3d_game_godot)

此文档记录 **3d_game**（基于 Godot 4.x 开发的 3D 动作 RPG 游戏）的具体系统架构、开发历程以及核心逻辑细节。

---

## 1. 项目架构大图与核心模块

### 1.1 物品数据库与背包 (Inventory & DB System)
* **设计模式**：模块解耦与数据驱动。
* **物品数据库**：[item_database.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/components/item_database.gd) 读取并解析 JSON/CSV 配置数据，拥有完备的容错降级机制，支持解析 HTML 十六进制颜色代码用以自适应渲染物品 UI 背光。
* **背包交互**：实现类似 Minecraft 的格子拖拽、交换、抓取机制。近期新增了鼠标悬停（Hover）在背包槽或快捷栏槽上时，显示物品名字及描述的 tooltip 优化。

### 1.2 魔法与双持战斗 (Spells & Double-wield Combat)
* **手持系统**：双第一人称体素手臂模型，支持独立控制与两手武器生成（如 procedural swords）。
* **点击分流**：左右键点击分别路由到主副手武器或法术。
* **法术选择轮盘**：通过 Q/E 键呼出 3D 悬浮轮盘，支持快速切换装备的主动魔法，发射 3D 魔法弹投影物。

### 1.3 随身小秘境/小洞天 (Mystic Realm Component & Room)
* **核心组件**：[mystic_realm_component.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/components/mystic_realm_component.gd)
  * 拥有独立的洞天仓库、挂机修为累计（`realm_cultivation_exp`）、灵气浓度增幅倍率。
  * 妖怪/灵兽工人状态机管理：预设了“抱元兔”（长于耕作）、“赤焰狐”（长于炼丹）两个工人，具有精力耗损与休息回复的完整闭环算法。
  * 设施与配方：灵田耕作（`plots` 播种与成熟收获）、九转紫金炉（炼制“小还丹”和“聚灵散”）。
  * 结算逻辑：物理帧 `_process` 或手动调用 `process_tick(delta)` 可以按给定的时间步长快速流逝洞天内的生产状态。
* **3D 房间生成**：[mystic_realm_room.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/scenes/mystic_realm_room.gd)
  * 高空位置（Y = 5000）拼装拼接的 3x3 灵木地板与玄石墙壁包围的修炼密室。
  * 房间正中心注册了名为 `NoItemPlacementArea` 的静态碰撞盒（1.8m x 1.6m x 1.8m），禁止玩家在中心区域放置任何物品，同时也挂载了传送落脚点 [TeleportSpawnPoint](file:///e:/0GD/3d-game-in-godot-main/3d_game/scenes/mystic_realm_room.gd#L93)。
* **沟通道具**：**“小秘境传送牌”**
  * 在 [player.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/entities/player/player.gd) 的 `_use_teleport_badge()` 中实现沟通逻辑。
  * 目前用于验证洞天自洽性：使用时在后台连续执行两次 8 秒的时间加速结算（模拟 16 秒生产流逝），并自动收取成熟的灵草后打印当前全部资产状态。

### 1.4 NPC 动画与行为控制 (Animated NPCs)
* **NPC 基类**：[npc_base.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/entities/npc/npc_base.gd)
  * 支持动态从 `res://fbx/` 加载外部 Mixamo 的 FBX 角色和动画资源。
  * 运动动画机：根据当前重力、速度和状态自动平滑过渡播放 `idle`、`walk`、`run`、`jump`、`fall` 等动画。
  * 攻击行为：近战攻击时通过 Tween 对 NPC 挂载的 `Visuals` 节点执行 -Z 轴突刺并退回，实现真实的挥剑打击感。
  * 受击闪烁：支持在受击或死亡时通过 `_flash_skin_color` 使所有材质瞬间亮橙/红，增加视觉反馈。

---

## 2. 关键设计约定与防坑指南

1. **高空 Y=5000 约定**：所有生成在非主大陆的独立空间（如秘境、洞天、试炼副本）坐标一律设在 Y=5000 以上，以完全防止主大陆的模型或物理碰撞影响洞天房间。
2. **FBX 动态加载**：FBX 导入时其动画轨道名称通常默认为 `"mixamo_com"`，[npc_base.gd](file:///e:/0GD/3d-game-in-godot-main/3d_game/entities/npc/npc_base.gd) 中已编写了专门的映射库将其提取并映射到标准动画库中。
3. **数据安全**：如果增加新物品，需要同步修改 `data/items.json`，确保在 `ItemDatabase` 加载时能被系统正确识别并匹配图标颜色。
