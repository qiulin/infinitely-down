# 技术架构设计

> 无限向下 - MVP 技术架构文档

## 1. 系统分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                      表现层 (Presentation)                   │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  Scene  │  │   UI    │  │ Prefab  │  │ Animation│        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
├─────────────────────────────────────────────────────────────┤
│                      业务逻辑层 (Logic)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ GameManager │  │PlayerManager│  │ UIManager   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │PlatformMgr  │  │ InputManager│  │ AudioManager│         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                      数据层 (Data)                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ PlayerData  │  │ GameConfig  │  │  EventBus   │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
├─────────────────────────────────────────────────────────────┤
│                      平台适配层 (Platform)                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  WechatSDK  │  │ CloudAPI    │  │   AdSDK     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. assets/ 目录结构规划

```
assets/
├── scenes/                    # 场景文件
│   ├── LaunchScene.scene      # 启动场景
│   ├── HomeScene.scene        # 主界面场景
│   └── GameScene.scene        # 游戏场景
│
├── scripts/                   # 脚本代码
│   ├── managers/              # 管理器单例
│   │   ├── GameManager.ts     # 游戏状态管理
│   │   ├── InputManager.ts    # 输入控制
│   │   ├── PlatformManager.ts # 平台生成与管理
│   │   ├── AudioManager.ts    # 音效管理
│   │   ├── UIManager.ts       # UI 管理
│   │   └── DataManager.ts     # 数据持久化
│   │
│   ├── components/            # 游戏组件
│   │   ├── Player.ts          # 玩家角色
│   │   ├── CameraFollow.ts    # 相机跟随
│   │   ├── platforms/         # 平台组件
│   │   │   ├── BasePlatform.ts
│   │   │   ├── NormalPlatform.ts
│   │   │   ├── MovingPlatform.ts
│   │   │   └── BreakablePlatform.ts
│   │   └── ui/                # UI 组件
│   │       ├── HomeUI.ts
│   │       ├── GameUI.ts
│   │       └── ResultUI.ts
│   │
│   ├── data/                  # 数据定义
│   │   ├── GameConfig.ts      # 游戏配置常量
│   │   ├── DataTypes.ts       # 接口与类型定义
│   │   └── EventTypes.ts      # 事件类型定义
│   │
│   ├── utils/                 # 工具类
│   │   ├── ObjectPool.ts      # 对象池
│   │   ├── EventBus.ts        # 事件总线
│   │   └── MathUtils.ts       # 数学工具
│   │
│   └── platform/              # 平台适配
│       ├── WechatAPI.ts       # 微信 API 封装
│       ├── CloudAPI.ts        # 云函数调用
│       └── AdAPI.ts           # 广告 API 封装
│
├── prefabs/                   # 预制体
│   ├── Player.prefab          # 玩家预制体
│   ├── platforms/             # 平台预制体
│   │   ├── NormalPlatform.prefab
│   │   ├── MovingPlatform.prefab
│   │   └── BreakablePlatform.prefab
│   └── ui/                    # UI 预制体
│       ├── ResultPopup.prefab
│       ├── PausePopup.prefab
│       └── LeaderboardPopup.prefab
│
├── textures/                  # 图片资源
│   ├── ui/                    # UI 图片
│   ├── player/                # 玩家图片
│   ├── platforms/             # 平台图片
│   └── backgrounds/           # 背景图片
│
├── audio/                     # 音频资源
│   ├── bgm/                   # 背景音乐
│   └── sfx/                   # 音效
│
└── animations/                # 动画资源
    ├── player/                # 玩家动画
    └── platforms/             # 平台动画
```

---

## 3. 核心模块职责

### 3.1 GameManager (游戏管理器)

**职责**: 控制游戏状态机、生命周期

```typescript
enum GameState {
    IDLE,       // 待机
    PLAYING,    // 游戏中
    PAUSED,     // 暂停
    GAMEOVER,   // 游戏结束
    REVIVING    // 复活中
}

class GameManager {
    // 状态管理
    currentState: GameState;

    // 游戏数据
    currentFloor: number;      // 当前层数
    currentScore: number;      // 当前分数
    currentCoins: number;      // 本局金币

    // 生命周期
    startGame(): void;
    pauseGame(): void;
    resumeGame(): void;
    gameOver(): void;
    revive(): void;
}
```

### 3.2 InputManager (输入管理器)

**职责**: 统一处理触摸/按键输入

```typescript
class InputManager {
    // 输入状态
    moveDirection: number;     // -1 左, 0 静止, 1 右
    isJumping: boolean;
    jumpPower: number;         // 0-1 蓄力值

    // 事件
    onMoveStart(direction: number): void;
    onMoveEnd(): void;
    onJumpTap(): void;
    onJumpHold(duration: number): void;
}
```

### 3.3 PlatformManager (平台管理器)

**职责**: 平台生成、回收、难度控制

```typescript
class PlatformManager {
    // 对象池
    platformPools: Map<PlatformType, ObjectPool>;

    // 生成控制
    spawnPlatform(type: PlatformType, y: number): BasePlatform;
    recyclePlatform(platform: BasePlatform): void;

    // 难度控制
    getCurrentDifficulty(): DifficultyConfig;
    updateDifficulty(floor: number): void;
}
```

### 3.4 UIManager (UI 管理器)

**职责**: UI 层级管理、弹窗栈

```typescript
class UIManager {
    // 弹窗管理
    showPopup(name: string, data?: any): void;
    hidePopup(name: string): void;
    hideAllPopups(): void;

    // HUD 更新
    updateScore(score: number): void;
    updateFloor(floor: number): void;
    updateCoins(coins: number): void;
}
```

### 3.5 DataManager (数据管理器)

**职责**: 本地存储、云端同步

```typescript
class DataManager {
    // 本地数据
    loadLocalData(): PlayerData;
    saveLocalData(data: PlayerData): void;

    // 云端同步
    syncToCloud(): Promise<void>;
    fetchFromCloud(): Promise<PlayerData>;
}
```

---

## 4. 服务端架构

### 4.1 云函数清单

| 函数名 | 功能 | 调用时机 |
|--------|------|----------|
| `login` | 用户登录、获取 openid | 游戏启动 |
| `submitScore` | 提交分数（含校验） | 游戏结束 |
| `getLeaderboard` | 获取排行榜 | 排行榜页面 |
| `getFriendRank` | 获取好友排行 | 主界面 |
| `syncUserData` | 同步用户数据 | 定时/退出时 |
| `getConfig` | 获取远程配置 | 游戏启动 |

### 4.2 云数据库集合设计

```
collections/
├── users/                     # 用户数据
│   ├── _openid: string        # 微信 openid (主键)
│   ├── nickname: string       # 昵称
│   ├── avatarUrl: string      # 头像
│   ├── highestFloor: number   # 最高层数
│   ├── totalCoins: number     # 总金币
│   ├── totalGames: number     # 总局数
│   ├── characters: array      # 已解锁角色
│   ├── settings: object       # 用户设置
│   ├── createdAt: date        # 创建时间
│   └── updatedAt: date        # 更新时间
│
├── scores/                    # 分数记录
│   ├── _id: string            # 记录 ID
│   ├── openid: string         # 用户 openid
│   ├── floor: number          # 层数
│   ├── score: number          # 分数
│   ├── coins: number          # 金币
│   ├── mode: string           # 游戏模式
│   ├── duration: number       # 游戏时长(秒)
│   ├── deathReason: string    # 死亡原因
│   ├── checksum: string       # 校验码
│   └── createdAt: date        # 创建时间
│
└── configs/                   # 远程配置
    ├── _id: string            # 配置 ID
    ├── type: string           # 配置类型
    ├── data: object           # 配置数据
    └── version: number        # 版本号
```

---

## 5. 性能约束

### 5.1 首包体积要求

| 类别 | 限制 | 说明 |
|------|------|------|
| 首包总大小 | ≤ 4MB | 微信小游戏硬性要求 |
| 代码体积 | ≤ 1MB | 压缩后的 JS |
| 图片资源 | ≤ 2MB | 首包必要资源 |
| 音频资源 | ≤ 500KB | 仅保留关键音效 |
| 预留空间 | ≥ 500KB | 用于后续热更新 |

### 5.2 运行时性能目标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 帧率 | 60 FPS | 稳定不低于 55 FPS |
| 内存占用 | ≤ 150MB | 避免低端机崩溃 |
| 启动时间 | ≤ 3s | 从点击到可交互 |
| DrawCall | ≤ 50 | 合理合批 |
| 节点数量 | ≤ 200 | 场景内活跃节点 |

### 5.3 优化策略

1. **对象池**: 平台、特效必须使用对象池
2. **图集合并**: 同类资源打包成图集
3. **按需加载**: 非首屏资源延迟加载
4. **LOD**: 远处平台使用简化渲染
5. **事件节流**: 触摸事件限制处理频率

---

## 6. 安全设计

### 6.1 反作弊策略

| 威胁 | 防护措施 |
|------|----------|
| 分数篡改 | 服务端校验 + 行为校验 |
| 金币刷取 | 服务端计算 + 频率限制 |
| 存档篡改 | 数据加密 + 云端备份 |
| 协议伪造 | 请求签名 + 时间戳校验 |

### 6.2 数据校验规则

```typescript
// 分数校验公式
interface ScoreValidation {
    maxFloorPerSecond: 2;      // 每秒最多下 2 层
    maxScorePerFloor: 100;     // 每层最高 100 分
    minGameDuration: 10;       // 最短游戏时长 10 秒
}

// 校验逻辑 (云函数)
function validateScore(data: GameResult): boolean {
    const { floor, score, duration } = data;

    // 时长校验
    if (duration < 10) return false;

    // 速度校验
    if (floor / duration > 2) return false;

    // 分数校验
    if (score / floor > 100) return false;

    return true;
}
```

### 6.3 敏感数据处理

- **openid**: 仅在服务端使用，不暴露给前端
- **金币/钻石**: 关键操作必须经过服务端
- **排行榜数据**: 服务端聚合计算，防止注入
