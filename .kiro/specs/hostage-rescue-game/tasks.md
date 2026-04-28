# 实现计划：床下潜伏解救人质游戏

## 概述

将游戏设计转化为单个 `index.html` 文件的增量实现步骤。每个任务在前一个任务基础上构建，从核心数据结构开始，逐步添加地图渲染、玩家控制、AI、战斗、音频等系统，最终整合为完整可玩的游戏。所有代码内联在一个 HTML 文件中，使用 HTML5 Canvas 渲染，Web Audio API 生成音效。

## 任务

- [ ] 1. 搭建项目骨架与核心数据结构
  - [ ] 1.1 创建 `index.html` 基础结构，包含 HTML 骨架、viewport meta 标签、内联 CSS 布局（HUD 信息栏 + Canvas 游戏区 + 操作控制区的 flex 纵向布局）、内联 `<script>` 标签
    - 包含禁止触屏缩放/滚动的 meta 和 CSS 设置（`touch-action:none`, `overflow:hidden` 等）
    - 包含 HUD 区域（生命值、武器、防具、人质计数、楼层、静音按钮）
    - 包含 Canvas 元素
    - 包含操作控制区（方向键网格布局 + 动作按钮）
    - 包含 CSS media queries 适配 320px-768px 屏幕及横屏布局
    - _需求: 9.1, 9.2, 9.3, 9.5, 9.6, 10.1, 10.2, 10.6_

  - [ ] 1.2 实现 `GAME_CONSTANTS` 常量对象和所有数据模型初始化函数
    - 定义 `GAME_CONSTANTS`（网格大小9、楼层数3、最小格子36px、玩家100HP、恐怖分子50HP/20伤害、检测距离2、人质每层3/总9、恐怖分子总6、手榴弹半径2、火焰持续3回合/30伤害、防具100耐久/50%减伤、通知2秒、目标30FPS）
    - 定义 `TileType` 枚举（FLOOR=0, WALL=1, BED=2, STAIR_UP=3, STAIR_DOWN=4）
    - 定义 `WEAPON_CONFIGS` 武器属性常量
    - 实现 `createInitialGameState()` 函数，返回完整的 `GameState` 对象（phase='start'）
    - 实现 `createPlayerState()`、`createTerroristState()`、`createHostageState()`、`createItemState()` 工厂函数
    - _需求: 7.1, 7.2, 5.4, 5.5, 5.6, 5.7, 6.3, 6.4, 6.5_

- [ ] 2. 实现地图系统
  - [ ] 2.1 硬编码3层9x9预定义地图数据，实现 `MapSystem` 核心函数
    - 定义 `FLOOR_1`、`FLOOR_2`、`FLOOR_3` 二维数组（按设计文档中的地图布局）
    - 实现 `initMaps()` 返回3层地图数据
    - 实现 `getTile(floor, x, y)` 查询地形类型
    - 实现 `isWalkable(floor, x, y)` 检查可通行性（FLOOR、BED、STAIR_UP、STAIR_DOWN 可通行）
    - 实现 `getStaircase(floor, direction)` 获取楼梯位置
    - 实现 `getLineOfSight(floor, fromX, fromY, direction)` 获取射击路径
    - _需求: 1.1, 1.4, 1.7_

  - [ ]* 2.2 编写属性测试：墙壁碰撞阻止移动
    - **Property 1: 墙壁碰撞阻止移动**
    - **验证: 需求 1.4, 2.2**

  - [ ]* 2.3 编写属性测试：有效移动恰好移动一格
    - **Property 2: 有效移动恰好移动一格**
    - **验证: 需求 2.1**

- [ ] 3. 实现 Canvas 渲染系统
  - [ ] 3.1 实现 `Renderer` 模块：Canvas 初始化、响应式 tileSize 计算、地图渲染
    - 实现 `calculateTileSize()` 根据 Canvas 可用空间计算格子大小（不小于36px）
    - 实现 `renderMap(ctx, floor, mapData, tileSize)` 绘制当前楼层地图（墙壁、地板、床、楼梯用不同颜色/图标区分）
    - 实现窗口 resize 事件监听，重新计算 tileSize 并重绘
    - _需求: 1.1, 1.2, 1.8, 10.6_

  - [ ] 3.2 实现实体渲染：玩家、恐怖分子、人质、地面物品、火焰区域
    - 实现 `renderEntities(ctx, state, tileSize)` 绘制当前楼层的所有实体
    - 玩家用特定颜色/符号表示，根据朝向显示方向指示
    - 恐怖分子用红色表示，区分 patrol/alert 状态
    - 人质用绿色表示（未解救的）
    - 地面物品用对应图标表示
    - 火焰区域用橙色半透明覆盖表示
    - _需求: 1.1, 10.6_

  - [ ] 3.3 实现 HUD 渲染和通知系统
    - 实现 `updateHUD(state)` 更新 DOM 中的 HUD 元素（生命值、武器名称和弹药、防具状态、人质计数、楼层）
    - 实现 `showNotification(state, text, type)` 添加通知到队列
    - 实现 `renderNotifications(ctx, notifications, canvasWidth, canvasHeight)` 在 Canvas 上绘制通知消息
    - 通知显示2秒后自动消失
    - _需求: 3.3, 3.5, 5.3, 5.10, 6.6, 7.9, 9.4_

- [ ] 4. 实现输入系统与玩家控制
  - [ ] 4.1 实现 `InputSystem`：触屏方向按钮、动作按钮、键盘监听
    - 实现 `setupTouchControls()` 为方向按钮绑定 touchstart/click 事件，产生 move action
    - 实现 `setupKeyboardControls()` 监听键盘方向键，产生 move action
    - 实现开火、躲藏、切换武器按钮的事件绑定
    - 实现 `consumeAction()` 获取并清空当前帧的输入动作
    - 阻止触屏默认行为（touchmove preventDefault、gesturestart/gesturechange preventDefault）
    - _需求: 2.1, 2.6, 2.7, 9.2, 9.3, 9.6_

  - [ ] 4.2 实现 `PlayerSystem`：移动、躲藏、切换武器、楼梯传送
    - 实现 `playerMove(state, direction)` 处理玩家移动（碰撞检测、更新朝向、自动拾取物品、自动使用楼梯）
    - 实现 `toggleHide(state)` 切换躲藏状态（仅在 BED 格上有效）
    - 实现 `switchWeapon(state)` 循环切换已拥有武器
    - 实现 `useStairs(state)` 楼梯传送逻辑（上楼梯传送到上层下楼梯位置，下楼梯传送到下层上楼梯位置）
    - 实现 `pickupItems(state)` 自动拾取当前格子上的物品（武器加入背包、防具装备、手榴弹/燃烧弹堆叠4个）
    - _需求: 1.5, 1.6, 2.1, 2.2, 2.3, 2.5, 5.2, 6.2, 9.7_

  - [ ]* 4.3 编写属性测试：躲藏/取消躲藏往返一致
    - **Property 3: 躲藏/取消躲藏往返一致**
    - **验证: 需求 2.3, 2.5**

  - [ ]* 4.4 编写属性测试：物品拾取添加到背包
    - **Property 10: 物品拾取添加到背包**
    - **验证: 需求 5.2, 6.2**

- [ ] 5. 检查点 - 确保基础系统正常运行
  - 确保所有测试通过，如有问题请向用户确认。

- [ ] 6. 实现恐怖分子AI系统
  - [ ] 6.1 实现 `TerroristAI`：视线检测、巡逻、追击、攻击决策
    - 实现 `canDetectPlayer(terrorist, player, map)` 四方向2格直线视线检测（含墙壁遮挡判定、hidden 状态免疫）
    - 实现 `generatePatrolPath(terrorist, map)` 生成巡逻路径（在可通行格子间随机选择相邻目标点）
    - 实现 `findPath(floor, fromX, fromY, toX, toY, map)` 简化A*寻路
    - 实现 `decide(terrorist, state)` AI决策：patrol→检测到玩家→alert→追击→同格→attack
    - 实现 `updateAll(state)` 所有存活恐怖分子依次执行一步行动
    - _需求: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6_

  - [ ]* 6.2 编写属性测试：躲藏状态下玩家不可被检测
    - **Property 4: 躲藏状态下玩家不可被检测**
    - **验证: 需求 2.4, 4.5**

  - [ ]* 6.3 编写属性测试：恐怖分子巡逻每回合移动一格
    - **Property 6: 恐怖分子巡逻每回合移动一格**
    - **验证: 需求 4.2**

  - [ ]* 6.4 编写属性测试：2格视线内检测触发警戒
    - **Property 7: 2格视线内检测触发警戒**
    - **验证: 需求 4.3**

  - [ ]* 6.5 编写属性测试：同格触发战斗
    - **Property 8: 同格触发战斗**
    - **验证: 需求 4.4**

- [ ] 7. 实现战斗与伤害系统
  - [ ] 7.1 实现 `CombatSystem`：伤害计算、防具减伤、武器开火
    - 实现 `calculateDamage(baseDamage, armor)` 计算防具减伤后的实际伤害和防具耐久消耗
    - 实现 `damageToPlayer(state, damage)` 对玩家造成伤害（考虑盔甲和头盔减伤，耐久度归零时销毁防具）
    - 实现 `damageToTerrorist(state, terroristIndex, damage)` 对恐怖分子造成伤害（生命值归零标记 dead，更新 eliminatedCount）
    - 实现 `playerFire(state)` 玩家开火逻辑：手枪/AK47 沿朝向直线射击（命中路径上第一个恐怖分子）、手榴弹范围爆炸、燃烧弹创建火焰区域
    - 实现 `canFire(weapon)` 检查武器是否可开火（弹药检查）
    - 实现 `consumeAmmo(weapon)` 消耗弹药
    - _需求: 5.4, 5.5, 5.6, 5.7, 5.8, 5.9, 6.3, 6.4, 6.5, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8_

  - [ ] 7.2 实现手榴弹爆炸和燃烧弹火焰区域逻辑
    - 实现 `grenadeExplosion(state, targetX, targetY)` 对2格半径内所有角色造成100点伤害
    - 实现 `incendiaryExplosion(state, targetX, targetY)` 在目标格创建持续3回合的火焰区域
    - 实现 `processFireZones(state)` 每回合处理火焰区域：对区域内角色造成30点伤害，剩余回合减1，回合归零时移除
    - _需求: 5.6, 5.7, 7.6, 7.7_

  - [ ]* 7.3 编写属性测试：接触人质即解救
    - **Property 5: 接触人质即解救**
    - **验证: 需求 3.2**

  - [ ]* 7.4 编写属性测试：消灭的恐怖分子永久移除
    - **Property 9: 消灭的恐怖分子永久移除**
    - **验证: 需求 4.6, 5.9**

  - [ ]* 7.5 编写属性测试：弹药耗尽无法开火
    - **Property 11: 弹药耗尽无法开火**
    - **验证: 需求 5.5**

  - [ ]* 7.6 编写属性测试：手榴弹范围伤害
    - **Property 12: 手榴弹范围伤害**
    - **验证: 需求 5.6**

  - [ ]* 7.7 编写属性测试：直线射击被墙壁阻挡
    - **Property 13: 直线射击被墙壁阻挡**
    - **验证: 需求 5.8**

  - [ ]* 7.8 编写属性测试：防具减伤50%且耐久度消耗
    - **Property 14: 防具减伤50%且耐久度消耗**
    - **验证: 需求 6.3, 6.4, 6.5**

  - [ ]* 7.9 编写属性测试：燃烧弹创建持续火焰区域
    - **Property 15: 燃烧弹创建持续火焰区域**
    - **验证: 需求 5.7, 7.7**

- [ ] 8. 检查点 - 确保战斗和AI系统正常运行
  - 确保所有测试通过，如有问题请向用户确认。

- [ ] 9. 实现音频系统
  - [ ] 9.1 实现 `AudioSystem`：Web Audio API 程序化音效生成
    - 实现 `initAudio()` 初始化 AudioContext（在用户首次交互时调用）
    - 实现 `playBGM()` 使用 OscillatorNode 生成循环背景音乐
    - 实现 `stopBGM()` 停止背景音乐
    - 实现 `playSFX(type)` 根据类型生成不同音效：手枪射击（短促高频）、AK47射击（低频连发）、爆炸（低频衰减噪声）、解救（上升音阶）、受伤（短促低频）、拾取（清脆高频）、脚步（轻微敲击）
    - 实现 `toggleMute()` 切换静音状态，绑定静音按钮
    - _需求: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7_

- [ ] 10. 实现游戏流程管理与画面
  - [ ] 10.1 实现游戏阶段状态机和开始/胜利/失败画面
    - 实现 `renderStartScreen(ctx, w, h)` 绘制开始画面（标题、说明、"开始游戏"按钮）
    - 实现 `renderVictoryScreen(ctx, w, h, state)` 绘制胜利画面（通关时间、消灭敌人数、"重新开始"按钮）
    - 实现 `renderGameOverScreen(ctx, w, h, state)` 绘制失败画面（统计信息、"重新开始"按钮）
    - 实现 Canvas 点击/触屏检测，处理画面按钮交互
    - _需求: 11.1, 11.3, 11.4, 11.5_

  - [ ] 10.2 实现 `initGame()` 和 `resetGame()` 函数
    - `initGame()` 初始化地图、随机分布物品/人质/恐怖分子到可通行格子、设置玩家初始位置（Floor 1）、phase 切换为 playing、记录 startTime、初始化音频并播放 BGM
    - `resetGame()` 重置所有游戏状态回到 start phase
    - 物品分布：1把AK47、1把手枪、1个手榴弹物品、1个燃烧弹物品、1件盔甲、1顶头盔，随机放置在3层楼的地板格上
    - 人质分布：每层3个，放置在地板格上
    - 恐怖分子分布：共6个，每层至少1个，放置在地板格上并生成巡逻路径
    - _需求: 3.1, 4.1, 5.1, 6.1, 11.2, 11.5_

  - [ ]* 10.3 编写属性测试：游戏重置产生干净初始状态
    - **Property 16: 游戏重置产生干净初始状态**
    - **验证: 需求 11.5**

- [ ] 11. 实现游戏主循环与系统整合
  - [ ] 11.1 实现 `gameLoop()` 主循环，整合所有系统
    - 使用 `requestAnimationFrame` 驱动渲染循环
    - 每帧：读取输入 → 执行玩家动作 → 触发回合（恐怖分子AI行动、火焰区域处理、碰撞检测）→ 检查胜利/失败条件 → 渲染当前帧
    - 回合制逻辑：玩家执行一次移动/动作后，所有恐怖分子依次行动一步
    - 胜利条件：9名人质全部解救 → phase 切换为 victory
    - 失败条件：玩家生命值归零 → phase 切换为 gameover
    - 根据 phase 渲染对应画面（start/playing/victory/gameover）
    - 在恐怖分子攻击时触发伤害音效和 HUD 伤害指示
    - _需求: 3.4, 7.8, 7.9, 11.2, 11.3, 11.4, 11.6_

  - [ ] 11.2 整合人质解救逻辑和通知系统
    - 玩家移动到人质格子时自动解救：标记 rescued、rescuedCount+1、显示解救通知2秒、播放解救音效
    - 恐怖分子攻击玩家时：显示伤害指示1秒、播放受伤音效
    - 武器拾取时：显示拾取通知、播放拾取音效
    - 所有音效调用通过 AudioSystem 统一管理
    - _需求: 3.2, 3.3, 5.3, 7.9, 8.2, 8.3, 8.4, 8.5_

- [ ] 12. 检查点 - 完整游戏流程验证
  - 确保所有测试通过，如有问题请向用户确认。

- [ ] 13. 搭建测试框架与编写属性测试
  - [ ] 13.1 搭建测试环境，配置 fast-check 属性测试框架
    - 创建 `package.json`，添加 fast-check 和测试运行器（vitest 或 jest）依赖
    - 将 `index.html` 中的核心游戏逻辑函数提取为可导入的模块（通过 `<script>` 中使用模块化导出，或创建独立的 `game-logic.js` 供测试导入）
    - 创建测试目录结构：`tests/game.property.test.js`
    - 配置测试运行命令
    - _需求: 10.1, 10.3_

  - [ ]* 13.2 编写单元测试：初始化验证和数值验证
    - 测试 `createInitialGameState()` 返回正确的初始状态
    - 测试地图3层9x9结构、楼梯位置正确
    - 测试 `initGame()` 后人质9个（每层3个）、恐怖分子6个（每层≥1）、物品6个
    - 测试玩家初始100HP、恐怖分子50HP、各武器伤害值匹配常量
    - _需求: 1.1, 1.7, 3.1, 4.1, 5.1, 6.1, 7.1, 7.2_

- [ ] 14. 最终检查点 - 确保所有测试通过，游戏完整可玩
  - 确保所有测试通过，如有问题请向用户确认。

## 备注

- 标记 `*` 的任务为可选任务，可跳过以加快 MVP 进度
- 每个任务引用了具体的需求编号以确保可追溯性
- 检查点确保增量验证，及时发现问题
- 属性测试验证通用正确性属性（使用 fast-check 库）
- 单元测试验证具体示例和边界条件
- 所有代码实现在单个 `index.html` 文件中（测试文件除外）
