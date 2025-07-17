# Minigame 插件开发任务清单

## 项目概览
- **插件名称**: minigame
- **包名**: qq.3753926641
- **作者**: RayFilm
- **核心依赖**: Paper 1.21.4 ([API文档](https://jd.papermc.io/paper/1.21.4/index.html))
- **跨服核心**: Velocity ([API文档](https://jd.papermc.io/velocity/3.4.0/))
- **Java版本**: 21

## 核心架构设计

### 1. 基础框架搭建

#### 1.1 主插件类创建
- [ ] 创建主插件类 `MinigamePlugin.java`
  - 继承 `JavaPlugin` 类
  - 实现双模启动检测（Velocity/Paper）
  - **API方法**: `JavaPlugin.getServer().getName()`
  - **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/Server.html#getName()
  - **用法**: 
    ```java
    String serverName = getServer().getName();
    if (serverName.contains("Paper") || serverName.contains("Spigot")) {
        // Paper/Spigot 环境
    }
    ```

#### 1.2 适配器模式架构
- [ ] 创建 `PlatformAdapter` 抽象类
  - 定义统一的平台接口
  - 包含跨平台通用方法
- [ ] 创建 `PaperAdapter` 实现类
  - 实现Paper平台特定功能
- [ ] 创建 `VelocityAdapter` 实现类
  - 实现Velocity平台特定功能

#### 1.3 配置文件系统
- [ ] `config.yml` - 主配置文件
  - **API方法**: `JavaPlugin.saveDefaultConfig()`
  - **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/plugin/java/JavaPlugin.html#saveDefaultConfig()
  - **配套方法**: 
    - `reloadConfig()` - 重新加载配置
    - `getConfig()` - 获取配置对象
- [ ] `messages.yml` - 消息配置
- [ ] `debugmessages.yml` - 调试消息配置

### 2. 游戏核心系统

#### 2.1 匹配系统
- [ ] 创建 `MatchingSystem` 抽象类
  - [ ] 实现跨服匹配逻辑
  - [ ] **Velocity API**: `Player.createConnectionRequest(RegisteredServer server)`
  - [ ] **子URL**: https://jd.papermc.io/velocity/3.4.0/com/velocitypowered/api/proxy/player/package-summary.html
  - [ ] **跨服事件处理**:
    - `ServerPreConnectEvent` - 服务器连接前事件
    - `ServerConnectedEvent` - 服务器连接成功事件
    - `ServerPostConnectEvent` - 服务器连接后事件
    - `KickedFromServerEvent` - 服务器踢出事件
  - [ ] **子URL**: https://jd.papermc.io/velocity/3.3.0/com/velocitypowered/api/event/player/package-summary.html

- [ ] GUI界面设计
  - [ ] **API方法**: `Bukkit.createInventory(InventoryHolder owner, int size, String title)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/Bukkit.html#createInventory(org.bukkit.inventory.InventoryHolder,int,java.lang.String)
  - [ ] 实现队伍编辑功能
  - [ ] **物品自定义**: 
    - `ItemStack` 创建物品
    - `ItemMeta` 设置物品属性
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/inventory/ItemStack.html

- [ ] 匹配队列管理
  - [ ] 创建 `MatchQueue` 类管理等待玩家
  - [ ] 实现最大5个队伍，每队3人的限制

#### 2.2 地图系统
- [ ] 地图克隆生成
  - [ ] **纠正API**: 使用 `Bukkit.createWorld(WorldCreator creator)` 配合文件操作
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/Bukkit.html#createWorld(org.bukkit.WorldCreator)
  - [ ] **实现方式**:
    ```java
    WorldCreator creator = new WorldCreator("game_world_" + gameId);
    creator.copy(templateWorld);
    World gameWorld = Bukkit.createWorld(creator);
    ```
  - [ ] 实现动态世界管理

- [ ] 队伍出生点管理
  - [ ] 创建 `SpawnPoint` 类存储固定点位
  - [ ] **API方法**: `Location` 类管理坐标
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/Location.html

#### 2.3 物资系统
- [ ] 箱子物资刷新
  - [ ] **事件监听**: `PlayerInteractEvent` 处理箱子交互
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/event/player/PlayerInteractEvent.html
  - [ ] 实现物品逐一显示效果（容器翻找功能）
  - [ ] **箱子访问**: 
    ```java
    if (block.getState() instanceof Chest chest) {
        Inventory inventory = chest.getInventory();
    }
    ```
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/block/Chest.html

- [ ] 地面掉落物刷新
  - [ ] **API方法**: `World.dropItem(Location location, ItemStack item)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/World.html#dropItem(org.bukkit.Location,org.bukkit.inventory.ItemStack)
  - [ ] 创建物资刷新调度器

#### 2.4 AI系统
- [ ] 士兵（神话生物）生成
  - [ ] **API方法**: `World.spawnEntity(Location location, EntityType type)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/World.html#spawnEntity(org.bukkit.Location,org.bukkit.entity.EntityType)
  - [ ] **实体类型**: `EntityType` 枚举定义生物类型
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/entity/EntityType.html

- [ ] AI 行为控制系统
  - [ ] **MobGoals接口**: 管理生物AI目标
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/com/destroystokyo/paper/entity/ai/MobGoals.html
  - [ ] **获取MobGoals**: `Bukkit.getMobGoals()`
  - [ ] **AI目标管理**:
    ```java
    MobGoals mobGoals = Bukkit.getMobGoals();
    // 添加AI目标
    mobGoals.addGoal(mob, priority, goal);
    // 移除AI目标
    mobGoals.removeGoal(mob, goalKey);
    // 获取运行中的目标
    Collection<Goal<T>> runningGoals = mobGoals.getRunningGoals(mob);
    ```

- [ ] 原版AI目标使用
  - [ ] **VanillaGoal**: 预定义的原版AI目标
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/com/destroystokyo/paper/entity/ai/VanillaGoal.html
  - [ ] **常用AI目标**:
    - `VanillaGoal.MELEE_ATTACK` - 近战攻击
    - `VanillaGoal.NEAREST_ATTACKABLE` - 寻找最近可攻击目标
    - `VanillaGoal.LOOK_AT_PLAYER` - 看向玩家
    - `VanillaGoal.RANDOM_LOOK_AROUND` - 随机环顾
    - `VanillaGoal.HURT_BY` - 被伤害时反击
    - `VanillaGoal.FLOAT` - 浮水行为
  - [ ] **实现方式**:
    ```java
    // 为僵尸添加攻击玩家的AI
    Zombie zombie = (Zombie) world.spawnEntity(location, EntityType.ZOMBIE);
    MobGoals mobGoals = Bukkit.getMobGoals();

    // 移除默认AI（可选）
    mobGoals.removeAllGoals(zombie, GoalType.TARGET);

    // 添加自定义AI目标
    mobGoals.addGoal(zombie, 1, VanillaGoal.FLOAT);
    mobGoals.addGoal(zombie, 2, VanillaGoal.MELEE_ATTACK);
    mobGoals.addGoal(zombie, 3, VanillaGoal.NEAREST_ATTACKABLE);
    ```

- [ ] AI目标类型管理
  - [ ] **GoalType枚举**: 定义AI目标类型
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/com/destroystokyo/paper/entity/ai/GoalType.html
  - [ ] **目标类型**:
    - `GoalType.MOVE` - 移动类型
    - `GoalType.LOOK` - 观察类型
    - `GoalType.JUMP` - 跳跃类型
    - `GoalType.TARGET` - 目标选择类型
    - `GoalType.UNKNOWN_BEHAVIOR` - 未知行为类型

- [ ] 自定义AI目标创建
  - [ ] **Goal接口**: 创建自定义AI行为
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/com/destroystokyo/paper/entity/ai/Goal.html
  - [ ] **实现自定义Goal**:
    ```java
    public class AttackPlayerGoal implements Goal<Zombie> {
        private final Zombie zombie;
        private Player target;

        public AttackPlayerGoal(Zombie zombie) {
            this.zombie = zombie;
        }

        @Override
        public boolean shouldActivate() {
            // 检查是否应该激活此目标
            target = zombie.getWorld().getNearbyPlayers(zombie.getLocation(), 10.0)
                .stream().findFirst().orElse(null);
            return target != null;
        }

        @Override
        public void start() {
            // 目标开始时的行为
            zombie.setTarget(target);
        }

        @Override
        public void tick() {
            // 每tick执行的行为
            if (target != null && target.isValid()) {
                zombie.getPathfinder().moveTo(target.getLocation());
            }
        }

        @Override
        public void stop() {
            // 目标停止时的行为
            zombie.setTarget(null);
            target = null;
        }

        @Override
        public GoalKey<Zombie> getKey() {
            return GoalKey.of(Zombie.class, new NamespacedKey(plugin, "attack_player"));
        }

        @Override
        public EnumSet<GoalType> getTypes() {
            return EnumSet.of(GoalType.TARGET, GoalType.MOVE);
        }
    }
    ```

- [ ] 特殊AI行为实现
  - [ ] **队伍协作AI**: 士兵之间的协作行为
  - [ ] **巡逻AI**: 在指定区域巡逻
  - [ ] **保护AI**: 保护特定目标或区域
  - [ ] **追击AI**: 追击特定玩家或队伍
  - [ ] **实现示例**:
    ```java
    // 为不同类型的士兵设置不同AI
    if (entityType == EntityType.SKELETON) {
        // 弓箭手AI - 远程攻击
        mobGoals.addGoal(mob, 1, VanillaGoal.RANGED_BOW_ATTACK);
        mobGoals.addGoal(mob, 2, VanillaGoal.AVOID_ENTITY); // 避免近战
    } else if (entityType == EntityType.ZOMBIE) {
        // 近战士兵AI
        mobGoals.addGoal(mob, 1, VanillaGoal.ZOMBIE_ATTACK);
        mobGoals.addGoal(mob, 2, VanillaGoal.MELEE_ATTACK);
    } else if (entityType == EntityType.IRON_GOLEM) {
        // 守护者AI
        mobGoals.addGoal(mob, 1, VanillaGoal.DEFEND_VILLAGE);
        mobGoals.addGoal(mob, 2, VanillaGoal.OFFER_FLOWER);
    }
    ```

### 3. 战斗与生存机制

#### 3.1 倒地机制
- [ ] 实现二条命系统
  - [ ] **事件监听**: `EntityDamageEvent` 处理伤害
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/event/entity/EntityDamageEvent.html
  - [ ] 创建 `DownedState` 管理倒地状态
  - [ ] **游戏模式切换**: `Player.setGameMode(GameMode mode)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/entity/Player.html#setGameMode(org.bukkit.GameMode)

- [ ] 队友救援系统
  - [ ] 实现拉起队友功能
  - [ ] 放弃救援（长按空格）检测

#### 3.2 死亡系统
- [ ] 尸包生成
  - [ ] 玩家死亡后生成箱子
  - [ ] **API方法**: `Block.setType(Material.CHEST)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/block/Block.html#setType(org.bukkit.Material)
  - [ ] 保存玩家物品到箱子

### 4. 撤离系统
- [ ] 拉匝撤离点
  - [ ] 创建倒计时系统
  - [ ] **任务调度**: 继承 `BukkitRunnable` 实现计时器
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/scheduler/BukkitRunnable.html
  - [ ] **使用方式**:
    ```java
    new BukkitRunnable() {
        @Override
        public void run() {
            // 计时器逻辑
        }
    }.runTaskTimer(plugin, 0L, 20L);
    ```
  - [ ] **音效播放**: `Player.playSound(Location location, Sound sound, float volume, float pitch)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/entity/Player.html#playSound(org.bukkit.Location,org.bukkit.Sound,float,float)

- [ ] 野外撤离点
  - [ ] 区域检测系统
  - [ ] 等待时间管理

### 5. 通信与UI系统

#### 5.1 队伍通信
- [ ] 队内聊天系统
  - [ ] **新API推荐**: `AsyncChatEvent` (Paper 1.21.4推荐)
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/io/papermc/paper/event/player/AsyncChatEvent.html
  - [ ] **传统API**: `AsyncPlayerChatEvent` (仍可用但不推荐)
  - [ ] **聊天渲染器**: 使用 `ChatRenderer` 接口自定义聊天显示
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/io/papermc/paper/chat/ChatRenderer.html
  - [ ] **实现方式**:
    ```java
    @EventHandler
    public void onAsyncChat(AsyncChatEvent event) {
        event.renderer(ChatRenderer.viewerUnaware((source, sourceDisplayName, message) -> {
            // 自定义队伍聊天格式
            return Component.text("[队伍] ").color(NamedTextColor.GREEN)
                .append(sourceDisplayName)
                .append(Component.text(": "))
                .append(message);
        }));
    }
    ```
  - [ ] 实现消息过滤和转发
  - [ ] **跨服聊天**: 使用Plugin Messaging转发队伍消息
  - [ ] **Velocity聊天事件**: `PlayerChatEvent` 处理代理层聊天
  - [ ] **子URL**: https://jd.papermc.io/velocity/3.3.0/com/velocitypowered/api/event/player/PlayerChatEvent.html

- [ ] 队伍发光效果
  - [ ] **API方法**: `Player.setGlowing(boolean flag)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/entity/Player.html#setGlowing(boolean)

#### 5.2 观战系统
- [ ] 观察者模式实现
  - [ ] **API方法**: `Player.setGameMode(GameMode.SPECTATOR)`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/GameMode.html#SPECTATOR
  - [ ] 限制潜行键离开
  - [ ] 队伍视角切换

### 6. 数据存储系统

#### 6.1 背包系统
- [ ] 独立匹配背包
  - [ ] 创建 `MatchInventory` 类
  - [ ] **API方法**: `Player.getInventory()` 返回 `PlayerInventory`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/entity/Player.html#getInventory()
- [ ] 安全箱功能
  - [ ] 创建持久化存储
  - [ ] 实现物品保护逻辑
- [ ] 撤离仓库
  - [ ] 独立存储撤离物品

#### 6.2 战绩系统
- [ ] 战绩记录
  - [ ] 创建 `BattleRecord` 数据类
  - [ ] 记录：时间、存活、撤离状态、物资价值、击杀数等
  - [ ] 使用数据库或文件存储
- [ ] 查询命令
  - [ ] 实现 `/minigame stats` 命令
  - [ ] **命令API**: 继承 `CommandExecutor` 接口
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/command/CommandExecutor.html

### 7. 排位系统
- [ ] 段位计算
  - [ ] 创建 `RankSystem` 类
  - [ ] 实现动态加减分算法
  - [ ] 考虑段位差异的分数调整

### 8. 声音增强系统
- [ ] 脚步声增强
  - [ ] **事件监听**: `PlayerMoveEvent`
  - [ ] **子URL**: https://jd.papermc.io/paper/1.21.4/org/bukkit/event/player/PlayerMoveEvent.html
  - [ ] 根据材质播放不同音效
  - [ ] **音效播放**: `Player.playSound()` 扩大范围
- [ ] 头盔声音调整
  - [ ] 检测装备类型
  - [ ] 动态调整声音距离

### 9. 跨服通信系统

#### 9.1 Plugin Messaging 实现
- [ ] 注册插件消息通道
  - [ ] **API方法**: `getServer().getMessenger().registerOutgoingPluginChannel(this, "BungeeCord")`
  - [ ] **接收消息**: `getServer().getMessenger().registerIncomingPluginChannel(this, "BungeeCord", this)`
  - [ ] **实现接口**: `PluginMessageListener`
  - [ ] **子URL**: https://docs.papermc.io/paper/dev/plugin-messaging/

- [ ] 跨服玩家传送
  - [ ] **BungeeCord命令**: `Connect` - 传送玩家到指定服务器
  - [ ] **实现方式**:
    ```java
    ByteArrayDataOutput out = ByteStreams.newDataOutput();
    out.writeUTF("Connect");
    out.writeUTF("game-server-1");
    player.sendPluginMessage(this, "BungeeCord", out.toByteArray());
    ```

- [ ] 跨服数据同步
  - [ ] **Forward命令**: 转发消息到其他服务器
  - [ ] **PlayerCount**: 获取服务器玩家数量
  - [ ] **MessageRaw**: 发送原始聊天组件到玩家
  - [ ] **实现方式**:
    ```java
    // 转发消息到所有服务器
    ByteArrayDataOutput out = ByteStreams.newDataOutput();
    out.writeUTF("Forward");
    out.writeUTF("ALL");
    out.writeUTF("MinigameChannel");

    ByteArrayOutputStream msgBytes = new ByteArrayOutputStream();
    DataOutputStream msgOut = new DataOutputStream(msgBytes);
    msgOut.writeUTF("GAME_START");
    msgOut.writeUTF(gameId);

    out.writeShort(msgBytes.toByteArray().length);
    out.write(msgBytes.toByteArray());
    player.sendPluginMessage(this, "BungeeCord", out.toByteArray());
    ```

#### 9.2 Velocity 事件处理
- [ ] 服务器连接事件
  - [ ] **ServerPreConnectEvent**: 连接前处理（可修改目标服务器）
  - [ ] **ServerConnectedEvent**: 连接成功处理
  - [ ] **ServerPostConnectEvent**: 连接后处理
  - [ ] **子URL**: https://jd.papermc.io/velocity/3.3.0/com/velocitypowered/api/event/player/ServerPreConnectEvent.html

- [ ] 玩家踢出处理
  - [ ] **KickedFromServerEvent**: 处理服务器踢出事件
  - [ ] **重定向选项**: `RedirectPlayer`, `DisconnectPlayer`, `Notify`
  - [ ] **实现方式**:
    ```java
    @Subscribe
    public void onKickedFromServer(KickedFromServerEvent event) {
        // 如果是游戏服务器踢出，重定向到大厅
        if (event.getServer().getServerInfo().getName().startsWith("game-")) {
            Optional<RegisteredServer> lobby = proxyServer.getServer("lobby");
            if (lobby.isPresent()) {
                event.setResult(KickedFromServerEvent.RedirectPlayer.create(lobby.get()));
            }
        }
    }
    ```

### 10. 特殊机制
- [ ] 穿甲系统（待确认）
  - [ ] 与现有枪械插件集成
  - [ ] 实现护甲耐久度计算
  - [ ] 穿透伤害逻辑

## API 方法参考

### Paper API 关键方法
- **插件基础**: `JavaPlugin` 类及其方法
- **世界管理**: `Bukkit.createWorld(WorldCreator)`, `World` 接口
- **玩家交互**: `Player.sendMessage()`, `Player.teleport()`, `Player.setGameMode()`
- **物品管理**: `ItemStack`, `ItemMeta`, `Inventory` 接口
- **事件监听**: `@EventHandler`, 各种 `Event` 类
- **任务调度**: `BukkitRunnable`, `BukkitScheduler`

### Velocity API 关键方法
- **跨服传送**: `Player.createConnectionRequest(RegisteredServer)`
- **消息转发**: `ChannelMessageEvent`
- **服务器管理**: `ProxyServer.getServer()`
- **玩家事件**: `PlayerChatEvent`, `ServerPreConnectEvent`, `KickedFromServerEvent`
- **插件消息**: `PlayerChannelRegisterEvent`

### Plugin Messaging 系统
- **注册通道**: `getServer().getMessenger().registerOutgoingPluginChannel(this, "BungeeCord")`
- **发送消息**: `player.sendPluginMessage(this, "BungeeCord", data)`
- **接收消息**: 实现 `PluginMessageListener` 接口
- **BungeeCord通道**: 支持 `Connect`, `PlayerCount`, `Forward`, `MessageRaw` 等命令
- **子URL**: https://docs.papermc.io/paper/dev/plugin-messaging/

## 开发优先级
1. **高优先级**: 基础框架、匹配系统、地图系统
2. **中优先级**: 战斗机制、撤离系统、数据存储
3. **低优先级**: 声音系统、排位系统、特殊机制

