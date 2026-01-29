# 核心玩法技术设计

> 无限向下 - 游戏核心机制文档

## 1. 游戏主循环

```
┌─────────────────────────────────────────────────────────────┐
│                      游戏主循环                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐               │
│   │  Input  │───▶│ Update  │───▶│ Render  │               │
│   └─────────┘    └─────────┘    └─────────┘               │
│        │              │              │                      │
│        ▼              ▼              ▼                      │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐               │
│   │ 触摸检测 │    │ 物理更新 │    │ 场景渲染 │               │
│   │ 按键检测 │    │ 碰撞检测 │    │ UI 更新  │               │
│   │ 手势识别 │    │ 状态更新 │    │ 特效播放 │               │
│   └─────────┘    └─────────┘    └─────────┘               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.1 帧更新流程

```typescript
// 每帧执行顺序 (60 FPS, deltaTime ≈ 16.67ms)
update(deltaTime: number) {
    // 1. 输入处理
    this.inputManager.processInput();

    // 2. 玩家更新
    this.player.update(deltaTime);

    // 3. 平台更新
    this.platformManager.update(deltaTime);

    // 4. 相机更新
    this.cameraController.update(deltaTime);

    // 5. 碰撞检测
    this.checkCollisions();

    // 6. 边界检测
    this.checkBoundaries();

    // 7. 分数更新
    this.updateScore();

    // 8. 难度更新
    this.updateDifficulty();
}
```

---

## 2. 玩家控制参数

### 2.1 基础参数表

| 参数名 | 默认值 | 单位 | 说明 |
|--------|--------|------|------|
| `moveSpeed` | 400 | px/s | 水平移动速度 |
| `gravity` | 1200 | px/s² | 重力加速度 |
| `maxFallSpeed` | 800 | px/s | 最大下落速度 |
| `jumpForce` | 600 | px/s | 基础跳跃力 |
| `powerJumpForce` | 900 | px/s | 蓄力跳跃力 |
| `jumpChargeTime` | 0.5 | s | 蓄力跳跃充能时间 |
| `airControlFactor` | 0.8 | - | 空中移动控制系数 |
| `groundFriction` | 0.9 | - | 地面摩擦系数 |

### 2.2 控制参数接口

```typescript
interface PlayerConfig {
    // 移动
    moveSpeed: number;           // 水平移动速度
    airControlFactor: number;    // 空中控制系数 (0-1)
    groundFriction: number;      // 地面摩擦 (0-1)

    // 跳跃
    jumpForce: number;           // 基础跳跃力
    powerJumpForce: number;      // 蓄力跳跃力
    jumpChargeTime: number;      // 蓄力所需时间

    // 物理
    gravity: number;             // 重力加速度
    maxFallSpeed: number;        // 最大下落速度

    // 碰撞
    width: number;               // 碰撞盒宽度
    height: number;              // 碰撞盒高度
}

const DEFAULT_PLAYER_CONFIG: PlayerConfig = {
    moveSpeed: 400,
    airControlFactor: 0.8,
    groundFriction: 0.9,
    jumpForce: 600,
    powerJumpForce: 900,
    jumpChargeTime: 0.5,
    gravity: 1200,
    maxFallSpeed: 800,
    width: 60,
    height: 80
};
```

---

## 3. 相机系统设计

### 3.1 相机行为

```
┌─────────────────────────────────────────┐
│              屏幕可视区域                 │
│  ┌─────────────────────────────────────┐│
│  │         死亡线 (顶部)                ││
│  │═════════════════════════════════════││
│  │                                     ││
│  │                                     ││
│  │            🧍 玩家                   ││
│  │         ─────────────               ││
│  │           目标跟随线                 ││
│  │                                     ││
│  │                                     ││
│  │═════════════════════════════════════││
│  │         深渊线 (底部)                ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

### 3.2 相机参数

```typescript
interface CameraConfig {
    // 跟随设置
    followOffsetY: number;       // 玩家在屏幕中的 Y 偏移 (0=中心, 正=偏上)
    followSmoothing: number;     // 跟随平滑度 (0-1, 越大越平滑)

    // 下落速度
    minScrollSpeed: number;      // 最小滚动速度
    maxScrollSpeed: number;      // 最大滚动速度
    scrollAcceleration: number;  // 滚动加速度

    // 边界
    topDeathOffset: number;      // 顶部死亡线距屏幕顶部距离
    bottomVoidOffset: number;    // 底部深渊线距屏幕底部距离
}

const DEFAULT_CAMERA_CONFIG: CameraConfig = {
    followOffsetY: 100,          // 玩家略微偏上
    followSmoothing: 0.1,        // 平滑跟随
    minScrollSpeed: 50,          // 初始滚动速度
    maxScrollSpeed: 300,         // 最大滚动速度
    scrollAcceleration: 5,       // 每层加速值
    topDeathOffset: 50,          // 顶部死亡线
    bottomVoidOffset: 100        // 底部深渊线
};
```

### 3.3 相机跟随逻辑

```typescript
class CameraFollow extends Component {
    private config: CameraConfig;
    private currentScrollSpeed: number;
    private targetY: number;

    update(deltaTime: number) {
        // 1. 计算目标位置 (跟随玩家)
        const playerY = this.player.position.y;
        this.targetY = playerY + this.config.followOffsetY;

        // 2. 平滑移动相机
        const currentY = this.node.position.y;
        const newY = lerp(currentY, this.targetY, this.config.followSmoothing);

        // 3. 应用强制下降 (相机只能向下，不能向上)
        const minY = currentY - this.currentScrollSpeed * deltaTime;
        const finalY = Math.min(newY, minY);

        this.node.setPosition(this.node.position.x, finalY);
    }

    // 根据层数更新滚动速度
    updateScrollSpeed(floor: number) {
        this.currentScrollSpeed = Math.min(
            this.config.minScrollSpeed + floor * this.config.scrollAcceleration,
            this.config.maxScrollSpeed
        );
    }
}
```

---

## 4. 碰撞系统

### 4.1 碰撞层级

```typescript
enum CollisionGroup {
    DEFAULT = 1 << 0,
    PLAYER = 1 << 1,
    PLATFORM = 1 << 2,
    HAZARD = 1 << 3,
    ITEM = 1 << 4
}

// 碰撞矩阵
const COLLISION_MATRIX = {
    [CollisionGroup.PLAYER]:
        CollisionGroup.PLATFORM |
        CollisionGroup.HAZARD |
        CollisionGroup.ITEM
};
```

### 4.2 碰撞检测规则

```typescript
interface CollisionRule {
    // 玩家与平台碰撞
    playerPlatform: {
        // 仅下落时检测 (从上方落下)
        onlyWhenFalling: true;
        // 玩家底部与平台顶部重叠阈值
        overlapThreshold: 10;
        // 最小接触速度
        minContactSpeed: 50;
    };

    // 玩家与危险物碰撞
    playerHazard: {
        // 碰撞即触发
        immediate: true;
        // 无敌时间 (复活后)
        invincibleTime: 2.0;
    };
}
```

### 4.3 平台碰撞检测

```typescript
// 简化的 AABB 碰撞检测
function checkPlatformCollision(player: Player, platform: BasePlatform): boolean {
    // 仅当玩家下落时检测
    if (player.velocity.y > 0) return false;

    const playerBottom = player.position.y - player.height / 2;
    const playerLeft = player.position.x - player.width / 2;
    const playerRight = player.position.x + player.width / 2;

    const platformTop = platform.position.y + platform.height / 2;
    const platformLeft = platform.position.x - platform.width / 2;
    const platformRight = platform.position.x + platform.width / 2;

    // 垂直方向：玩家底部接近平台顶部
    const verticalContact =
        playerBottom <= platformTop &&
        playerBottom >= platformTop - 20; // 20px 容差

    // 水平方向：有重叠
    const horizontalOverlap =
        playerRight > platformLeft &&
        playerLeft < platformRight;

    return verticalContact && horizontalOverlap;
}
```

---

## 5. 难度递增系统

### 5.1 难度曲线设计

以 **10 层为一小段**，逐步提高难度：

| 层数范围 | 平台间距 | 移动平台速度 | 移动平台比例 | 破碎平台比例 |
|----------|----------|--------------|--------------|--------------|
| 1-10 | 80-100 | 0 | 0% | 0% |
| 11-20 | 90-110 | 50 | 10% | 5% |
| 21-30 | 100-120 | 70 | 15% | 10% |
| 31-40 | 110-130 | 90 | 20% | 15% |
| 41-50 | 120-140 | 110 | 25% | 20% |
| 51-70 | 130-150 | 130 | 30% | 25% |
| 71-100 | 140-160 | 150 | 35% | 30% |
| 100+ | 150-180 | 180 | 40% | 35% |

### 5.2 难度配置接口

```typescript
interface DifficultyConfig {
    // 层数范围
    floorRange: [number, number];

    // 平台间距
    platformGapMin: number;
    platformGapMax: number;

    // 平台宽度
    platformWidthMin: number;
    platformWidthMax: number;

    // 移动平台
    movingPlatformSpeed: number;
    movingPlatformRatio: number;

    // 破碎平台
    breakablePlatformRatio: number;
    breakableDelayTime: number;

    // 相机滚动
    cameraScrollSpeed: number;
}

// MVP 难度配置数组
const DIFFICULTY_CONFIGS: DifficultyConfig[] = [
    {
        floorRange: [1, 10],
        platformGapMin: 80,
        platformGapMax: 100,
        platformWidthMin: 120,
        platformWidthMax: 150,
        movingPlatformSpeed: 0,
        movingPlatformRatio: 0,
        breakablePlatformRatio: 0,
        breakableDelayTime: 0.5,
        cameraScrollSpeed: 50
    },
    // ... 更多难度配置
];
```

### 5.3 难度选择逻辑

```typescript
function getDifficultyForFloor(floor: number): DifficultyConfig {
    for (const config of DIFFICULTY_CONFIGS) {
        const [min, max] = config.floorRange;
        if (floor >= min && floor <= max) {
            return config;
        }
    }
    // 超出最大层数，返回最难配置
    return DIFFICULTY_CONFIGS[DIFFICULTY_CONFIGS.length - 1];
}
```

---

## 6. 分数计算规则

### 6.1 基础分数

```typescript
interface ScoreRule {
    // 每层基础分
    baseScorePerFloor: 10;

    // 连续下落加成 (连续 N 层不停留)
    comboBonus: {
        threshold: 3;          // 3 层触发
        multiplier: 1.5;       // 1.5 倍加成
        maxMultiplier: 3.0;    // 最大 3 倍
    };

    // 时间奖励
    timeBonus: {
        enabled: true;
        bonusPerSecond: 1;     // 每秒 1 分
    };

    // 金币掉落
    coinDrop: {
        baseCoins: 1;          // 每层基础金币
        bonusChance: 0.1;      // 10% 概率额外金币
        bonusAmount: 5;        // 额外金币数量
    };
}
```

### 6.2 分数计算公式

```typescript
function calculateFloorScore(floor: number, combo: number): number {
    const rule = SCORE_RULE;

    // 基础分
    let score = rule.baseScorePerFloor;

    // 连击加成
    if (combo >= rule.comboBonus.threshold) {
        const multiplier = Math.min(
            1 + (combo - rule.comboBonus.threshold + 1) * 0.5,
            rule.comboBonus.maxMultiplier
        );
        score *= multiplier;
    }

    // 层数加成 (越深分数越高)
    const depthBonus = 1 + Math.floor(floor / 50) * 0.1;
    score *= depthBonus;

    return Math.floor(score);
}
```

---

## 7. 死亡判定

### 7.1 死亡条件

```typescript
enum DeathReason {
    PUSHED_UP = 'pushed_up',       // 被推出屏幕顶部
    FALL_VOID = 'fall_void',       // 掉入深渊
    HAZARD_DAMAGE = 'hazard',      // 危险物伤害 (MVP暂不实现)
    TIMEOUT = 'timeout'            // 超时 (100层模式)
}

function checkDeath(player: Player, camera: CameraFollow): DeathReason | null {
    const screenTop = camera.position.y + SCREEN_HEIGHT / 2;
    const screenBottom = camera.position.y - SCREEN_HEIGHT / 2;

    // 被推出顶部
    if (player.position.y > screenTop - DEATH_MARGIN_TOP) {
        return DeathReason.PUSHED_UP;
    }

    // 掉入深渊
    if (player.position.y < screenBottom + DEATH_MARGIN_BOTTOM) {
        return DeathReason.FALL_VOID;
    }

    return null;
}
```

### 7.2 死亡流程

```
玩家死亡
    │
    ▼
┌─────────────┐
│ 暂停游戏    │
│ 播放死亡动画│
└─────────────┘
    │
    ▼
┌─────────────┐     是      ┌─────────────┐
│ 是否有复活  │────────────▶│ 显示复活弹窗 │
│ 机会？      │             │ (观看广告)   │
└─────────────┘             └─────────────┘
    │ 否                          │
    ▼                             ▼ 复活
┌─────────────┐             ┌─────────────┐
│ 显示结算界面│◀────────────│ 原地复活    │
│ 保存分数    │   放弃复活   │ 2秒无敌     │
└─────────────┘             └─────────────┘
```

---

## 8. 游戏模式

### 8.1 MVP 模式

| 模式 | 说明 | 结束条件 |
|------|------|----------|
| 无尽模式 | 无尽下落，越深分数越高 | 死亡 |
| 100层挑战 | 经典致敬，尽快到达100层 | 死亡或到达100层 |

### 8.2 模式配置

```typescript
interface GameModeConfig {
    mode: GameMode;
    name: string;
    description: string;

    // 目标
    targetFloor?: number;        // 目标层数 (100层模式)
    timeLimit?: number;          // 时间限制 (秒)

    // 难度修正
    difficultyMultiplier: number;

    // 奖励修正
    scoreMultiplier: number;
    coinMultiplier: number;
}

const GAME_MODES: GameModeConfig[] = [
    {
        mode: GameMode.ENDLESS,
        name: '无尽模式',
        description: '下到最深处',
        difficultyMultiplier: 1.0,
        scoreMultiplier: 1.0,
        coinMultiplier: 1.0
    },
    {
        mode: GameMode.CHALLENGE_100,
        name: '100层挑战',
        description: '经典致敬',
        targetFloor: 100,
        difficultyMultiplier: 0.9,  // 略低难度
        scoreMultiplier: 1.2,       // 完成奖励更高
        coinMultiplier: 1.5
    }
];
```
