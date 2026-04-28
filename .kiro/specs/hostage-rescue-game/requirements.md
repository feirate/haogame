# 需求文档：床下潜伏解救人质游戏

## 简介

一款基于网页的战术潜行游戏。玩家扮演特种兵，在一栋3层建筑中潜伏行动，解救被恐怖分子看守的人质。游戏采用俯视角9x9网格地图，玩家可以躲在床下避免被发现，利用武器消灭敌人，收集装备，最终解救全部人质通关。游戏为纯前端实现（HTML + CSS + JavaScript），适配手机触屏操作，可部署到 GitHub Pages 直接游玩。

## 术语表

- **Game（游戏系统）**：整个游戏应用，负责协调所有子系统
- **Map（地图系统）**：管理3层9x9网格地图的数据和渲染
- **Player（玩家角色）**：用户控制的主角，可在地图上移动、拾取物品、使用武器
- **Terrorist（恐怖分子）**：AI控制的敌人，在楼层间巡逻
- **Hostage（人质）**：需要被解救的NPC，玩家接触即可解救
- **Bed（床）**：地图上的安全区域，玩家躲在床下时不会被恐怖分子发现
- **Staircase（楼梯）**：连接不同楼层的出入口，分为上楼梯和下楼梯
- **Weapon（武器）**：可拾取的攻击道具，包括AK47、手枪、手榴弹、燃烧弹
- **Armor（防具）**：可拾取的防御道具，包括盔甲和头盔
- **Collision_System（碰撞系统）**：检测角色与地图元素、角色与角色之间的碰撞
- **Audio_System（音频系统）**：管理游戏音效和背景音乐的播放
- **Input_System（输入系统）**：处理触屏和键盘输入
- **HUD（界面系统）**：显示玩家状态、武器信息、楼层信息等

## 需求

### 需求 1：地图系统

**用户故事：** 作为玩家，我希望在一栋3层建筑中进行游戏，每层都有独特的布局，以便体验丰富的关卡设计。

#### 验收标准

1. THE Map SHALL render a 9x9 grid for each of the 3 floors
2. WHEN the game starts, THE Map SHALL display Floor 1 as the initial view
3. THE Map SHALL contain at least one Bed per floor as a safe zone
4. THE Map SHALL contain wall tiles that block Player and Terrorist movement
5. WHEN the Player moves to a Staircase tile marked as "up", THE Map SHALL transition the view to the next higher floor
6. WHEN the Player moves to a Staircase tile marked as "down", THE Map SHALL transition the view to the next lower floor
7. THE Map SHALL place one Staircase-up and one Staircase-down on each floor, except Floor 1 which has no Staircase-down and Floor 3 which has no Staircase-up
8. THE Map SHALL render all tiles with a minimum size of 36x36 CSS pixels to ensure touch accessibility on mobile devices

### 需求 2：玩家角色控制

**用户故事：** 作为玩家，我希望能流畅地控制角色在地图上移动，以便探索建筑并执行任务。

#### 验收标准

1. WHEN the Player swipes or taps a directional control, THE Input_System SHALL move the Player one grid cell in the corresponding direction (up, down, left, right)
2. WHEN the Player attempts to move into a wall tile, THE Collision_System SHALL prevent the movement and keep the Player at the current position
3. WHEN the Player is on a Bed tile and chooses to hide, THE Game SHALL set the Player state to "hidden"
4. WHILE the Player state is "hidden", THE Game SHALL prevent Terrorist detection of the Player
5. WHEN the Player chooses to leave the Bed, THE Game SHALL set the Player state to "active"
6. THE Input_System SHALL provide on-screen directional arrow buttons (up, down, left, right) for touch-based directional control on mobile devices
7. THE Input_System SHALL support keyboard arrow keys for directional control on desktop

### 需求 3：人质解救

**用户故事：** 作为玩家，我希望能找到并解救所有人质，以便完成游戏目标。

#### 验收标准

1. WHEN the game starts, THE Game SHALL place exactly 9 Hostages across the 3 floors, with exactly 3 Hostages per floor
2. WHEN the Player moves to a tile occupied by a Hostage, THE Game SHALL mark that Hostage as "rescued" and increment the rescued count
3. WHEN the Player rescues a Hostage, THE HUD SHALL display a rescue notification for 2 seconds
4. WHEN all 9 Hostages are rescued, THE Game SHALL display a victory screen with the completion time
5. THE HUD SHALL display the current rescued Hostage count out of 9 at all times

### 需求 4：恐怖分子AI

**用户故事：** 作为玩家，我希望恐怖分子能在楼层间巡逻，以便增加游戏的挑战性和紧张感。

#### 验收标准

1. WHEN the game starts, THE Game SHALL place exactly 6 Terrorists distributed across the 3 floors, with at least one Terrorist per floor
2. THE Terrorist SHALL patrol by moving one grid cell per turn along a predefined or random path within the current floor
3. WHEN a Terrorist is within 2 grid cells line-of-sight distance of the Player and the Player state is "active", THE Terrorist SHALL change state to "alert" and move toward the Player
4. WHEN a Terrorist occupies the same tile as the Player and the Player state is "active", THE Collision_System SHALL trigger a combat encounter
5. WHILE the Player state is "hidden", THE Terrorist SHALL ignore the Player and continue patrolling
6. IF a Terrorist is eliminated, THEN THE Game SHALL remove that Terrorist from the map permanently

### 需求 5：武器系统

**用户故事：** 作为玩家，我希望能收集和使用多种武器，以便消灭恐怖分子保护自己。

#### 验收标准

1. WHEN the game starts, THE Game SHALL randomly distribute Weapons across the 3 floors: one AK47, one Pistol, 4 Grenades, and 4 Incendiary_Bombs
2. WHEN the Player moves to a tile containing a Weapon, THE Game SHALL add that Weapon to the Player inventory
3. WHEN the Player picks up a Weapon, THE HUD SHALL display a pickup notification
4. THE Pistol SHALL have unlimited ammunition
5. THE AK47 SHALL have a magazine of 30 rounds, and WHEN the magazine is empty, THE Game SHALL prevent the AK47 from firing
6. WHEN the Player uses a Grenade, THE Game SHALL deal damage to all characters within a 2-tile radius of the target tile and decrement the Grenade count by 1
7. WHEN the Player uses an Incendiary_Bomb, THE Game SHALL create a fire zone on the target tile lasting 3 turns that damages any character entering the zone, and decrement the Incendiary_Bomb count by 1
8. WHEN the Player fires the AK47 or Pistol, THE Game SHALL apply damage along a straight line from the Player position in the facing direction until hitting a wall or the edge of the map
9. WHEN a Terrorist receives damage equal to or exceeding the Terrorist health, THE Game SHALL eliminate that Terrorist
10. THE HUD SHALL display the currently equipped Weapon and remaining ammunition count

### 需求 6：防具系统

**用户故事：** 作为玩家，我希望能收集防具来减少受到的伤害，以便提高生存能力。

#### 验收标准

1. WHEN the game starts, THE Game SHALL randomly distribute Armor items across the 3 floors: one Body_Armor and one Helmet
2. WHEN the Player moves to a tile containing an Armor item, THE Game SHALL equip that Armor item on the Player
3. WHILE the Player has Body_Armor equipped, THE Game SHALL reduce incoming damage to the Player body by 50 percent
4. WHILE the Player has a Helmet equipped, THE Game SHALL reduce incoming damage to the Player head by 50 percent
5. WHEN an Armor item absorbs a total of 100 damage points, THE Game SHALL destroy that Armor item and remove its protection
6. THE HUD SHALL display the current Armor status including remaining durability

### 需求 7：战斗与伤害系统

**用户故事：** 作为玩家，我希望战斗系统公平且有反馈，以便我能做出战术决策。

#### 验收标准

1. THE Player SHALL have an initial health of 100 points
2. THE Terrorist SHALL have an initial health of 50 points
3. WHEN a Terrorist attacks the Player, THE Game SHALL deal 20 damage points to the Player per attack
4. WHEN the Player fires the Pistol and hits a Terrorist, THE Game SHALL deal 25 damage points to that Terrorist
5. WHEN the Player fires the AK47 and hits a Terrorist, THE Game SHALL deal 35 damage points to that Terrorist
6. WHEN a Grenade explodes, THE Game SHALL deal 100 damage points to all characters within the blast radius
7. WHEN a character enters an Incendiary_Bomb fire zone, THE Game SHALL deal 30 damage points per turn to that character
8. IF the Player health reaches 0, THEN THE Game SHALL display a game-over screen with an option to restart
9. WHEN damage is dealt to the Player, THE HUD SHALL display a damage indicator for 1 second

### 需求 8：音频系统

**用户故事：** 作为玩家，我希望游戏有音效和背景音乐，以便增强沉浸感。

#### 验收标准

1. WHEN the game starts, THE Audio_System SHALL play background music in a loop
2. WHEN the Player fires a Weapon, THE Audio_System SHALL play the corresponding weapon sound effect
3. WHEN a Grenade or Incendiary_Bomb explodes, THE Audio_System SHALL play an explosion sound effect
4. WHEN the Player rescues a Hostage, THE Audio_System SHALL play a rescue success sound effect
5. WHEN the Player takes damage, THE Audio_System SHALL play a damage sound effect
6. THE Audio_System SHALL generate all sounds programmatically using the Web Audio API without requiring external audio files
7. THE HUD SHALL provide a mute toggle button to enable or disable all audio

### 需求 9：用户界面与移动端适配

**用户故事：** 作为手机用户，我希望游戏界面适配触屏操作，以便我能在手机上流畅游玩。

#### 验收标准

1. THE Game SHALL render the complete game interface within the viewport without requiring scrolling
2. THE Input_System SHALL display directional arrow buttons (up, down, left, right) in the bottom-left area of the screen for movement control
3. THE Input_System SHALL display action buttons (fire, hide, switch weapon) in the bottom-right area of the screen
4. THE HUD SHALL display Player health, equipped Weapon, ammunition count, Armor status, and rescued Hostage count
5. THE Game SHALL use CSS media queries to adapt layout for screen widths between 320px and 768px
6. THE Game SHALL prevent default touch behaviors such as zooming and scrolling during gameplay
7. WHEN the Player needs to switch Weapons, THE HUD SHALL provide a weapon selection interface accessible via a single tap

### 需求 10：部署与技术约束

**用户故事：** 作为开发者，我希望游戏能以纯前端形式部署到 GitHub Pages，以便任何人都能通过链接直接游玩。

#### 验收标准

1. THE Game SHALL be implemented using only HTML, CSS, and JavaScript without any server-side dependencies
2. THE Game SHALL consist of a single HTML file or a minimal set of static files that can be served by GitHub Pages
3. THE Game SHALL load and run without requiring any build tools, package managers, or compilation steps
4. THE Game SHALL function correctly in mobile Safari and mobile Chrome browsers
5. THE Game SHALL NOT contain any hardcoded user credentials, API keys, or personally identifiable information in the source code
6. THE Game SHALL use the HTML5 Canvas API or DOM-based rendering for the game display

### 需求 11：游戏流程管理

**用户故事：** 作为玩家，我希望游戏有清晰的开始、进行和结束流程，以便我能完整地体验游戏。

#### 验收标准

1. WHEN the game is loaded, THE Game SHALL display a start screen with a "开始游戏" button
2. WHEN the Player taps the "开始游戏" button, THE Game SHALL initialize the map, place all entities, and begin gameplay
3. WHEN all 9 Hostages are rescued, THE Game SHALL transition to a victory screen displaying total time and enemies eliminated count
4. IF the Player health reaches 0, THEN THE Game SHALL transition to a game-over screen with a "重新开始" button
5. WHEN the Player taps the "重新开始" button, THE Game SHALL reset all game state and return to the start screen
6. THE Game SHALL run a game loop at a consistent frame rate of at least 30 frames per second
