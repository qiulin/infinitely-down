# 数据结构定义

> 无限向下 - TypeScript 接口与类型定义

## 1. 枚举定义

### 1.1 游戏状态

```typescript
/**
 * 游戏状态枚举
 */
export enum GameState {
    /** 待机状态 - 未开始游戏 */
    IDLE = 'idle',
    /** 游戏进行中 */
    PLAYING = 'playing',
    /** 游戏暂停 */
    PAUSED = 'paused',
    /** 游戏结束 */
    GAMEOVER = 'gameover',
    /** 复活等待中 */
    REVIVING = 'reviving'
}

/**
 * 游戏模式枚举
 */
export enum GameMode {
    /** 无尽模式 */
    ENDLESS = 'endless',
    /** 100层挑战 */
    CHALLENGE_100 = 'challenge_100',
    /** 每日试炼 (后期) */
    DAILY_TRIAL = 'daily_trial'
}
```

### 1.2 平台类型

```typescript
/**
 * 平台类型枚举
 */
export enum PlatformType {
    /** 普通平台 - 稳定不动 */
    NORMAL = 'normal',
    /** 移动平台 - 左右移动 */
    MOVING = 'moving',
    /** 破碎平台 - 踩上后破碎 */
    BREAKABLE = 'breakable',

    // 后期扩展
    /** 弹簧平台 */
    SPRING = 'spring',
    /** 传送带平台 */
    CONVEYOR = 'conveyor',
    /** 冰面平台 */
    ICE = 'ice',
    /** 钉刺平台 */
    SPIKE = 'spike',
    /** 隐藏平台 */
    HIDDEN = 'hidden'
}

/**
 * 死亡原因枚举
 */
export enum DeathReason {
    /** 被推出屏幕顶部 */
    PUSHED_UP = 'pushed_up',
    /** 掉入深渊 */
    FALL_VOID = 'fall_void',
    /** 危险物伤害 */
    HAZARD_DAMAGE = 'hazard_damage',
    /** 超时 */
    TIMEOUT = 'timeout'
}
```

### 1.3 UI 相关枚举

```typescript
/**
 * 场景名称枚举
 */
export enum SceneName {
    LAUNCH = 'LaunchScene',
    HOME = 'HomeScene',
    GAME = 'GameScene'
}

/**
 * 弹窗类型枚举
 */
export enum PopupType {
    /** 结算弹窗 */
    RESULT = 'ResultPopup',
    /** 暂停弹窗 */
    PAUSE = 'PausePopup',
    /** 复活弹窗 */
    REVIVE = 'RevivePopup',
    /** 排行榜弹窗 */
    LEADERBOARD = 'LeaderboardPopup',
    /** 设置弹窗 */
    SETTINGS = 'SettingsPopup'
}

/**
 * 广告类型枚举
 */
export enum AdType {
    /** 激励视频 */
    REWARDED = 'rewarded',
    /** 插屏广告 */
    INTERSTITIAL = 'interstitial',
    /** Banner 广告 */
    BANNER = 'banner'
}
```

---

## 2. 核心数据接口

### 2.1 玩家数据

```typescript
/**
 * 玩家持久化数据
 */
export interface PlayerData {
    /** 用户唯一标识 */
    openid: string;

    /** 昵称 */
    nickname: string;

    /** 头像 URL */
    avatarUrl: string;

    /** 最高层数 */
    highestFloor: number;

    /** 最高分数 */
    highestScore: number;

    /** 总金币 */
    totalCoins: number;

    /** 总钻石 */
    totalDiamonds: number;

    /** 总游戏局数 */
    totalGames: number;

    /** 总游戏时长 (秒) */
    totalPlayTime: number;

    /** 当前使用角色 ID */
    currentCharacterId: string;

    /** 已解锁角色 ID 列表 */
    unlockedCharacters: string[];

    /** 已解锁皮肤 ID 列表 */
    unlockedSkins: string[];

    /** 用户设置 */
    settings: UserSettings;

    /** 新手引导进度 */
    tutorialProgress: number;

    /** 数据版本号 */
    dataVersion: number;

    /** 创建时间 */
    createdAt: number;

    /** 最后更新时间 */
    updatedAt: number;
}

/**
 * 用户设置
 */
export interface UserSettings {
    /** 背景音乐开关 */
    bgmEnabled: boolean;

    /** 背景音乐音量 (0-1) */
    bgmVolume: number;

    /** 音效开关 */
    sfxEnabled: boolean;

    /** 音效音量 (0-1) */
    sfxVolume: number;

    /** 震动反馈开关 */
    vibrationEnabled: boolean;
}

/**
 * 默认用户设置
 */
export const DEFAULT_USER_SETTINGS: UserSettings = {
    bgmEnabled: true,
    bgmVolume: 0.8,
    sfxEnabled: true,
    sfxVolume: 1.0,
    vibrationEnabled: true
};
```

### 2.2 游戏会话

```typescript
/**
 * 游戏会话数据 (单局游戏)
 */
export interface GameSession {
    /** 会话 ID */
    sessionId: string;

    /** 游戏模式 */
    mode: GameMode;

    /** 开始时间戳 */
    startTime: number;

    /** 结束时间戳 */
    endTime: number;

    /** 当前层数 */
    currentFloor: number;

    /** 当前分数 */
    currentScore: number;

    /** 本局金币 */
    coinsCollected: number;

    /** 复活次数 */
    reviveCount: number;

    /** 最大连击数 */
    maxCombo: number;

    /** 死亡原因 */
    deathReason: DeathReason | null;

    /** 是否已结算 */
    isSettled: boolean;
}

/**
 * 游戏结算结果
 */
export interface GameResult {
    /** 会话 ID */
    sessionId: string;

    /** 游戏模式 */
    mode: GameMode;

    /** 最终层数 */
    floor: number;

    /** 最终分数 */
    score: number;

    /** 获得金币 */
    coins: number;

    /** 游戏时长 (秒) */
    duration: number;

    /** 死亡原因 */
    deathReason: DeathReason;

    /** 是否为新纪录 */
    isNewRecord: boolean;

    /** 击败玩家百分比 */
    beatPercent: number;

    /** 校验码 (反作弊) */
    checksum: string;
}
```

---

## 3. 平台相关接口

### 3.1 平台配置

```typescript
/**
 * 平台基础配置
 */
export interface PlatformConfig {
    /** 平台类型 */
    type: PlatformType;

    /** 宽度 */
    width: number;

    /** 高度 */
    height: number;

    /** 预制体路径 */
    prefabPath: string;

    /** 是否可穿透 (从下方跳上) */
    canPassThrough: boolean;
}

/**
 * 移动平台配置
 */
export interface MovingPlatformConfig extends PlatformConfig {
    /** 移动速度 */
    speed: number;

    /** 移动范围 (左右各多少像素) */
    moveRange: number;

    /** 初始方向 (1: 右, -1: 左) */
    initialDirection: 1 | -1;
}

/**
 * 破碎平台配置
 */
export interface BreakablePlatformConfig extends PlatformConfig {
    /** 破碎延迟时间 (秒) */
    breakDelay: number;

    /** 破碎后恢复时间 (秒, 0 表示不恢复) */
    recoverTime: number;

    /** 破碎前警告时间 (秒) */
    warningTime: number;
}

/**
 * MVP 平台配置表
 */
export const PLATFORM_CONFIGS: Record<PlatformType, PlatformConfig> = {
    [PlatformType.NORMAL]: {
        type: PlatformType.NORMAL,
        width: 120,
        height: 20,
        prefabPath: 'prefabs/platforms/NormalPlatform',
        canPassThrough: true
    },
    [PlatformType.MOVING]: {
        type: PlatformType.MOVING,
        width: 100,
        height: 20,
        prefabPath: 'prefabs/platforms/MovingPlatform',
        canPassThrough: true,
        speed: 100,
        moveRange: 80,
        initialDirection: 1
    } as MovingPlatformConfig,
    [PlatformType.BREAKABLE]: {
        type: PlatformType.BREAKABLE,
        width: 100,
        height: 20,
        prefabPath: 'prefabs/platforms/BreakablePlatform',
        canPassThrough: true,
        breakDelay: 0.5,
        recoverTime: 0,
        warningTime: 0.3
    } as BreakablePlatformConfig
};
```

### 3.2 难度配置

```typescript
/**
 * 难度等级配置
 */
export interface DifficultyConfig {
    /** 层数范围 [最小, 最大] */
    floorRange: [number, number];

    /** 平台垂直间距范围 */
    platformGap: {
        min: number;
        max: number;
    };

    /** 平台宽度范围 */
    platformWidth: {
        min: number;
        max: number;
    };

    /** 各类型平台生成权重 */
    platformWeights: {
        [PlatformType.NORMAL]: number;
        [PlatformType.MOVING]: number;
        [PlatformType.BREAKABLE]: number;
    };

    /** 移动平台速度 */
    movingPlatformSpeed: number;

    /** 破碎平台延迟 */
    breakableDelay: number;

    /** 相机滚动速度 */
    cameraScrollSpeed: number;
}

/**
 * MVP 难度配置表
 */
export const DIFFICULTY_CONFIGS: DifficultyConfig[] = [
    {
        floorRange: [1, 10],
        platformGap: { min: 80, max: 100 },
        platformWidth: { min: 120, max: 150 },
        platformWeights: {
            [PlatformType.NORMAL]: 100,
            [PlatformType.MOVING]: 0,
            [PlatformType.BREAKABLE]: 0
        },
        movingPlatformSpeed: 0,
        breakableDelay: 0.5,
        cameraScrollSpeed: 50
    },
    {
        floorRange: [11, 20],
        platformGap: { min: 90, max: 110 },
        platformWidth: { min: 110, max: 140 },
        platformWeights: {
            [PlatformType.NORMAL]: 85,
            [PlatformType.MOVING]: 10,
            [PlatformType.BREAKABLE]: 5
        },
        movingPlatformSpeed: 50,
        breakableDelay: 0.5,
        cameraScrollSpeed: 60
    },
    {
        floorRange: [21, 30],
        platformGap: { min: 100, max: 120 },
        platformWidth: { min: 100, max: 130 },
        platformWeights: {
            [PlatformType.NORMAL]: 75,
            [PlatformType.MOVING]: 15,
            [PlatformType.BREAKABLE]: 10
        },
        movingPlatformSpeed: 70,
        breakableDelay: 0.45,
        cameraScrollSpeed: 70
    },
    {
        floorRange: [31, 50],
        platformGap: { min: 110, max: 140 },
        platformWidth: { min: 90, max: 120 },
        platformWeights: {
            [PlatformType.NORMAL]: 60,
            [PlatformType.MOVING]: 25,
            [PlatformType.BREAKABLE]: 15
        },
        movingPlatformSpeed: 100,
        breakableDelay: 0.4,
        cameraScrollSpeed: 85
    },
    {
        floorRange: [51, 100],
        platformGap: { min: 120, max: 160 },
        platformWidth: { min: 80, max: 110 },
        platformWeights: {
            [PlatformType.NORMAL]: 50,
            [PlatformType.MOVING]: 30,
            [PlatformType.BREAKABLE]: 20
        },
        movingPlatformSpeed: 130,
        breakableDelay: 0.35,
        cameraScrollSpeed: 100
    },
    {
        floorRange: [101, Infinity],
        platformGap: { min: 140, max: 180 },
        platformWidth: { min: 70, max: 100 },
        platformWeights: {
            [PlatformType.NORMAL]: 40,
            [PlatformType.MOVING]: 35,
            [PlatformType.BREAKABLE]: 25
        },
        movingPlatformSpeed: 160,
        breakableDelay: 0.3,
        cameraScrollSpeed: 120
    }
];
```

---

## 4. 排行榜数据

```typescript
/**
 * 排行榜条目
 */
export interface LeaderboardEntry {
    /** 排名 */
    rank: number;

    /** 用户 openid */
    openid: string;

    /** 昵称 */
    nickname: string;

    /** 头像 URL */
    avatarUrl: string;

    /** 分数/层数 */
    score: number;

    /** 记录时间 */
    recordTime: number;
}

/**
 * 排行榜类型
 */
export enum LeaderboardType {
    /** 好友排行 - 最高层数 */
    FRIEND_FLOOR = 'friend_floor',
    /** 好友排行 - 最高分数 */
    FRIEND_SCORE = 'friend_score',
    /** 全服排行 - 最高层数 */
    GLOBAL_FLOOR = 'global_floor',
    /** 全服排行 - 最高分数 */
    GLOBAL_SCORE = 'global_score',
    /** 每日排行 */
    DAILY = 'daily'
}

/**
 * 排行榜数据
 */
export interface LeaderboardData {
    /** 排行榜类型 */
    type: LeaderboardType;

    /** 排行榜条目 */
    entries: LeaderboardEntry[];

    /** 我的排名 */
    myRank: number;

    /** 我的分数 */
    myScore: number;

    /** 更新时间 */
    updateTime: number;
}
```

---

## 5. 事件类型

```typescript
/**
 * 游戏事件类型
 */
export enum GameEventType {
    // 游戏流程
    GAME_START = 'game_start',
    GAME_PAUSE = 'game_pause',
    GAME_RESUME = 'game_resume',
    GAME_OVER = 'game_over',
    GAME_REVIVE = 'game_revive',

    // 玩家事件
    PLAYER_JUMP = 'player_jump',
    PLAYER_LAND = 'player_land',
    PLAYER_DEATH = 'player_death',

    // 平台事件
    PLATFORM_TOUCH = 'platform_touch',
    PLATFORM_BREAK = 'platform_break',

    // 进度事件
    FLOOR_CHANGE = 'floor_change',
    SCORE_CHANGE = 'score_change',
    COIN_COLLECT = 'coin_collect',

    // UI 事件
    POPUP_SHOW = 'popup_show',
    POPUP_HIDE = 'popup_hide',

    // 广告事件
    AD_SHOW = 'ad_show',
    AD_CLOSE = 'ad_close',
    AD_REWARD = 'ad_reward'
}

/**
 * 事件数据基类
 */
export interface GameEventData {
    type: GameEventType;
    timestamp: number;
}

/**
 * 层数变化事件
 */
export interface FloorChangeEvent extends GameEventData {
    type: GameEventType.FLOOR_CHANGE;
    floor: number;
    previousFloor: number;
}

/**
 * 分数变化事件
 */
export interface ScoreChangeEvent extends GameEventData {
    type: GameEventType.SCORE_CHANGE;
    score: number;
    delta: number;
}
```

---

## 6. 游戏常量

```typescript
/**
 * 游戏全局常量
 */
export const GAME_CONSTANTS = {
    // 屏幕相关
    DESIGN_WIDTH: 750,
    DESIGN_HEIGHT: 1334,

    // 玩家相关
    PLAYER_WIDTH: 60,
    PLAYER_HEIGHT: 80,
    PLAYER_START_Y: 500,

    // 物理相关
    GRAVITY: 1200,
    MAX_FALL_SPEED: 800,
    MOVE_SPEED: 400,
    JUMP_FORCE: 600,
    POWER_JUMP_FORCE: 900,

    // 相机相关
    CAMERA_FOLLOW_OFFSET_Y: 100,
    CAMERA_MIN_SCROLL_SPEED: 50,
    CAMERA_MAX_SCROLL_SPEED: 300,
    DEATH_MARGIN_TOP: 50,
    DEATH_MARGIN_BOTTOM: 100,

    // 平台相关
    PLATFORM_SPAWN_BUFFER: 500,  // 屏幕外生成缓冲区
    PLATFORM_RECYCLE_BUFFER: 300,  // 屏幕外回收缓冲区
    MIN_PLATFORM_WIDTH: 70,
    MAX_PLATFORM_WIDTH: 150,

    // 分数相关
    BASE_SCORE_PER_FLOOR: 10,
    COMBO_THRESHOLD: 3,
    MAX_SCORE_MULTIPLIER: 3.0,

    // 金币相关
    BASE_COINS_PER_FLOOR: 1,
    BONUS_COIN_CHANCE: 0.1,
    BONUS_COIN_AMOUNT: 5,
    DOUBLE_COIN_MULTIPLIER: 2,

    // 广告相关
    MAX_REVIVE_PER_GAME: 1,
    INTERSTITIAL_INTERVAL: 5,  // 每 5 局一次插屏

    // 复活相关
    REVIVE_INVINCIBLE_TIME: 2.0,  // 复活无敌时间

    // 数据相关
    LOCAL_STORAGE_KEY: 'infinitely_down_data',
    DATA_VERSION: 1
} as const;

/**
 * 资源路径常量
 */
export const RESOURCE_PATHS = {
    // 场景
    SCENES: {
        LAUNCH: 'scenes/LaunchScene',
        HOME: 'scenes/HomeScene',
        GAME: 'scenes/GameScene'
    },

    // 预制体
    PREFABS: {
        PLAYER: 'prefabs/Player',
        PLATFORMS: {
            NORMAL: 'prefabs/platforms/NormalPlatform',
            MOVING: 'prefabs/platforms/MovingPlatform',
            BREAKABLE: 'prefabs/platforms/BreakablePlatform'
        },
        UI: {
            RESULT_POPUP: 'prefabs/ui/ResultPopup',
            PAUSE_POPUP: 'prefabs/ui/PausePopup',
            REVIVE_POPUP: 'prefabs/ui/RevivePopup'
        }
    },

    // 音频
    AUDIO: {
        BGM_HOME: 'audio/bgm/home',
        BGM_GAME: 'audio/bgm/game',
        SFX_JUMP: 'audio/sfx/jump',
        SFX_LAND: 'audio/sfx/land',
        SFX_BREAK: 'audio/sfx/break',
        SFX_DEATH: 'audio/sfx/death',
        SFX_COIN: 'audio/sfx/coin',
        SFX_BUTTON: 'audio/sfx/button'
    }
} as const;
```
