# 云函数接口定义

> 无限向下 - 微信云开发云函数规范

## 1. 云函数概览

### 1.1 MVP 云函数列表

| 函数名 | 功能 | 触发时机 |
|--------|------|----------|
| `login` | 用户登录 | 游戏启动 |
| `submitScore` | 提交分数 | 游戏结束 |
| `getLeaderboard` | 获取排行榜 | 排行榜页面 |
| `syncUserData` | 同步用户数据 | 定时/退出时 |
| `getConfig` | 获取远程配置 | 游戏启动 |

### 1.2 目录结构

```
cloudfunctions/
├── login/
│   ├── index.js
│   ├── package.json
│   └── config.json
├── submitScore/
│   ├── index.js
│   └── package.json
├── getLeaderboard/
│   ├── index.js
│   └── package.json
├── syncUserData/
│   ├── index.js
│   └── package.json
└── getConfig/
    ├── index.js
    └── package.json
```

---

## 2. login 登录函数

### 2.1 功能说明

- 接收 code，换取 openid
- 创建或更新用户记录
- 返回用户基础信息

### 2.2 接口定义

**请求参数**:
```typescript
interface LoginRequest {
    code: string;           // wx.login 获取的 code
}
```

**返回结果**:
```typescript
interface LoginResponse {
    success: boolean;
    openid: string;
    isNewUser: boolean;     // 是否新用户
    userData?: {            // 已有用户的数据
        nickname: string;
        avatarUrl: string;
        highestFloor: number;
        totalCoins: number;
    };
}
```

### 2.3 实现代码

```javascript
// cloudfunctions/login/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();

exports.main = async (event, context) => {
    const { OPENID } = cloud.getWXContext();

    try {
        // 查询用户是否存在
        const userResult = await db.collection('users')
            .where({ _openid: OPENID })
            .get();

        if (userResult.data.length === 0) {
            // 新用户，创建记录
            const newUser = {
                _openid: OPENID,
                nickname: '',
                avatarUrl: '',
                highestFloor: 0,
                highestScore: 0,
                totalCoins: 0,
                totalGames: 0,
                createdAt: db.serverDate(),
                updatedAt: db.serverDate()
            };

            await db.collection('users').add({ data: newUser });

            return {
                success: true,
                openid: OPENID,
                isNewUser: true
            };
        }

        // 老用户，返回数据
        const user = userResult.data[0];
        return {
            success: true,
            openid: OPENID,
            isNewUser: false,
            userData: {
                nickname: user.nickname,
                avatarUrl: user.avatarUrl,
                highestFloor: user.highestFloor,
                totalCoins: user.totalCoins
            }
        };

    } catch (err) {
        console.error('Login error:', err);
        return {
            success: false,
            error: err.message
        };
    }
};
```

---

## 3. submitScore 提交分数

### 3.1 功能说明

- 接收游戏结果
- 服务端校验分数合法性
- 更新用户最高分
- 记录游戏日志

### 3.2 接口定义

**请求参数**:
```typescript
interface SubmitScoreRequest {
    sessionId: string;      // 游戏会话 ID
    mode: string;           // 游戏模式
    floor: number;          // 层数
    score: number;          // 分数
    coins: number;          // 金币
    duration: number;       // 游戏时长 (秒)
    deathReason: string;    // 死亡原因
    checksum: string;       // 校验码
}
```

**返回结果**:
```typescript
interface SubmitScoreResponse {
    success: boolean;
    isNewRecord: boolean;   // 是否新纪录
    beatPercent: number;    // 击败百分比
    rank: number;           // 当前排名
    error?: string;
}
```

### 3.3 校验规则

```javascript
// 分数校验规则
const VALIDATION_RULES = {
    maxFloorPerSecond: 2,       // 每秒最多下 2 层
    maxScorePerFloor: 100,      // 每层最高 100 分
    minGameDuration: 10,        // 最短游戏时长 10 秒
    maxCoinsPerFloor: 10        // 每层最多 10 金币
};

function validateScore(data) {
    const { floor, score, coins, duration } = data;

    // 时长校验
    if (duration < VALIDATION_RULES.minGameDuration) {
        return { valid: false, reason: 'duration_too_short' };
    }

    // 速度校验
    if (floor / duration > VALIDATION_RULES.maxFloorPerSecond) {
        return { valid: false, reason: 'too_fast' };
    }

    // 分数校验
    if (score / floor > VALIDATION_RULES.maxScorePerFloor) {
        return { valid: false, reason: 'score_too_high' };
    }

    // 金币校验
    if (coins / floor > VALIDATION_RULES.maxCoinsPerFloor) {
        return { valid: false, reason: 'coins_too_high' };
    }

    return { valid: true };
}
```

### 3.4 实现代码

```javascript
// cloudfunctions/submitScore/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();
const _ = db.command;

exports.main = async (event, context) => {
    const { OPENID } = cloud.getWXContext();
    const { sessionId, mode, floor, score, coins, duration, deathReason, checksum } = event;

    try {
        // 1. 校验数据
        const validation = validateScore(event);
        if (!validation.valid) {
            console.warn(`Invalid score from ${OPENID}:`, validation.reason);
            return {
                success: false,
                error: 'validation_failed'
            };
        }

        // 2. 校验 checksum (简单示例，实际应更复杂)
        const expectedChecksum = generateChecksum(sessionId, floor, score, duration);
        if (checksum !== expectedChecksum) {
            console.warn(`Checksum mismatch from ${OPENID}`);
            return {
                success: false,
                error: 'checksum_invalid'
            };
        }

        // 3. 获取用户当前数据
        const userResult = await db.collection('users')
            .where({ _openid: OPENID })
            .get();

        if (userResult.data.length === 0) {
            return { success: false, error: 'user_not_found' };
        }

        const user = userResult.data[0];
        const isNewRecord = floor > user.highestFloor;

        // 4. 记录游戏日志
        await db.collection('scores').add({
            data: {
                openid: OPENID,
                sessionId,
                mode,
                floor,
                score,
                coins,
                duration,
                deathReason,
                createdAt: db.serverDate()
            }
        });

        // 5. 更新用户数据
        const updateData = {
            totalCoins: _.inc(coins),
            totalGames: _.inc(1),
            updatedAt: db.serverDate()
        };

        if (isNewRecord) {
            updateData.highestFloor = floor;
            updateData.highestScore = score;
        }

        await db.collection('users')
            .where({ _openid: OPENID })
            .update({ data: updateData });

        // 6. 计算排名
        const rankResult = await db.collection('users')
            .where({
                highestFloor: _.gt(floor)
            })
            .count();

        const totalUsers = await db.collection('users').count();
        const rank = rankResult.total + 1;
        const beatPercent = Math.floor(
            ((totalUsers.total - rank) / totalUsers.total) * 100
        );

        return {
            success: true,
            isNewRecord,
            beatPercent,
            rank
        };

    } catch (err) {
        console.error('Submit score error:', err);
        return {
            success: false,
            error: err.message
        };
    }
};

function validateScore(data) {
    // ... 校验逻辑
}

function generateChecksum(sessionId, floor, score, duration) {
    // 简单校验码生成 (实际应使用更安全的算法)
    const secret = 'your-secret-key';
    const raw = `${sessionId}|${floor}|${score}|${duration}|${secret}`;
    // 实际应使用 crypto 模块生成 hash
    return Buffer.from(raw).toString('base64');
}
```

---

## 4. getLeaderboard 获取排行榜

### 4.1 功能说明

- 获取全服排行榜
- 支持分页
- 返回当前用户排名

### 4.2 接口定义

**请求参数**:
```typescript
interface GetLeaderboardRequest {
    type: 'floor' | 'score';  // 排行类型
    limit: number;            // 获取数量
    offset: number;           // 偏移量
}
```

**返回结果**:
```typescript
interface GetLeaderboardResponse {
    success: boolean;
    entries: LeaderboardEntry[];
    myRank: number;
    myScore: number;
    total: number;
}

interface LeaderboardEntry {
    rank: number;
    nickname: string;
    avatarUrl: string;
    floor: number;
    score: number;
}
```

### 4.3 实现代码

```javascript
// cloudfunctions/getLeaderboard/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();
const _ = db.command;

exports.main = async (event, context) => {
    const { OPENID } = cloud.getWXContext();
    const { type = 'floor', limit = 50, offset = 0 } = event;

    try {
        const sortField = type === 'floor' ? 'highestFloor' : 'highestScore';

        // 1. 获取排行榜数据
        const leaderboardResult = await db.collection('users')
            .orderBy(sortField, 'desc')
            .skip(offset)
            .limit(limit)
            .field({
                nickname: true,
                avatarUrl: true,
                highestFloor: true,
                highestScore: true
            })
            .get();

        // 2. 格式化数据
        const entries = leaderboardResult.data.map((user, index) => ({
            rank: offset + index + 1,
            nickname: user.nickname || '匿名玩家',
            avatarUrl: user.avatarUrl || '',
            floor: user.highestFloor,
            score: user.highestScore
        }));

        // 3. 获取当前用户排名
        const myUserResult = await db.collection('users')
            .where({ _openid: OPENID })
            .get();

        let myRank = 0;
        let myScore = 0;

        if (myUserResult.data.length > 0) {
            const myUser = myUserResult.data[0];
            myScore = type === 'floor' ? myUser.highestFloor : myUser.highestScore;

            // 计算排名
            const higherCount = await db.collection('users')
                .where({
                    [sortField]: _.gt(myScore)
                })
                .count();

            myRank = higherCount.total + 1;
        }

        // 4. 获取总数
        const totalResult = await db.collection('users').count();

        return {
            success: true,
            entries,
            myRank,
            myScore,
            total: totalResult.total
        };

    } catch (err) {
        console.error('Get leaderboard error:', err);
        return {
            success: false,
            error: err.message
        };
    }
};
```

---

## 5. syncUserData 同步用户数据

### 5.1 功能说明

- 同步本地数据到云端
- 处理数据合并冲突
- 返回最新数据

### 5.2 接口定义

**请求参数**:
```typescript
interface SyncUserDataRequest {
    localData: {
        nickname: string;
        avatarUrl: string;
        settings: UserSettings;
        localVersion: number;   // 本地数据版本
    };
}
```

**返回结果**:
```typescript
interface SyncUserDataResponse {
    success: boolean;
    serverData: PlayerData;    // 服务器最新数据
    merged: boolean;           // 是否发生合并
}
```

### 5.3 实现代码

```javascript
// cloudfunctions/syncUserData/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();

exports.main = async (event, context) => {
    const { OPENID } = cloud.getWXContext();
    const { localData } = event;

    try {
        // 1. 获取服务器数据
        const serverResult = await db.collection('users')
            .where({ _openid: OPENID })
            .get();

        if (serverResult.data.length === 0) {
            return { success: false, error: 'user_not_found' };
        }

        const serverData = serverResult.data[0];

        // 2. 合并数据 (服务端数据优先，但保留本地的 nickname/avatar/settings)
        const mergedData = {
            nickname: localData.nickname || serverData.nickname,
            avatarUrl: localData.avatarUrl || serverData.avatarUrl,
            settings: localData.settings || serverData.settings,
            updatedAt: db.serverDate()
        };

        // 3. 更新服务器
        await db.collection('users')
            .where({ _openid: OPENID })
            .update({ data: mergedData });

        // 4. 返回最新数据
        const finalResult = await db.collection('users')
            .where({ _openid: OPENID })
            .get();

        return {
            success: true,
            serverData: finalResult.data[0],
            merged: true
        };

    } catch (err) {
        console.error('Sync user data error:', err);
        return {
            success: false,
            error: err.message
        };
    }
};
```

---

## 6. getConfig 获取远程配置

### 6.1 功能说明

- 获取游戏远程配置
- 支持热更新参数
- 版本控制

### 6.2 接口定义

**请求参数**:
```typescript
interface GetConfigRequest {
    configType: 'game' | 'ad' | 'all';
    clientVersion: string;      // 客户端版本号
}
```

**返回结果**:
```typescript
interface GetConfigResponse {
    success: boolean;
    config: {
        game?: GameRemoteConfig;
        ad?: AdRemoteConfig;
    };
    needUpdate: boolean;        // 是否需要强制更新
    latestVersion: string;
}

interface GameRemoteConfig {
    difficultyMultiplier: number;   // 难度系数
    coinMultiplier: number;         // 金币系数
    maxRevivePerGame: number;       // 每局最大复活次数
}

interface AdRemoteConfig {
    interstitialInterval: number;   // 插屏间隔局数
    bannerEnabled: boolean;         // Banner 是否启用
    rewardedVideoEnabled: boolean;  // 激励视频是否启用
}
```

### 6.3 实现代码

```javascript
// cloudfunctions/getConfig/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const db = cloud.database();

// 默认配置
const DEFAULT_GAME_CONFIG = {
    difficultyMultiplier: 1.0,
    coinMultiplier: 1.0,
    maxRevivePerGame: 1
};

const DEFAULT_AD_CONFIG = {
    interstitialInterval: 5,
    bannerEnabled: true,
    rewardedVideoEnabled: true
};

exports.main = async (event, context) => {
    const { configType = 'all', clientVersion = '1.0.0' } = event;

    try {
        const result = {
            success: true,
            config: {},
            needUpdate: false,
            latestVersion: '1.0.0'
        };

        // 1. 获取版本信息
        const versionResult = await db.collection('configs')
            .where({ type: 'version' })
            .get();

        if (versionResult.data.length > 0) {
            const versionConfig = versionResult.data[0];
            result.latestVersion = versionConfig.data.latestVersion;
            result.needUpdate = compareVersion(
                clientVersion,
                versionConfig.data.minVersion
            ) < 0;
        }

        // 2. 获取游戏配置
        if (configType === 'game' || configType === 'all') {
            const gameResult = await db.collection('configs')
                .where({ type: 'game' })
                .get();

            result.config.game = gameResult.data.length > 0
                ? gameResult.data[0].data
                : DEFAULT_GAME_CONFIG;
        }

        // 3. 获取广告配置
        if (configType === 'ad' || configType === 'all') {
            const adResult = await db.collection('configs')
                .where({ type: 'ad' })
                .get();

            result.config.ad = adResult.data.length > 0
                ? adResult.data[0].data
                : DEFAULT_AD_CONFIG;
        }

        return result;

    } catch (err) {
        console.error('Get config error:', err);
        return {
            success: false,
            error: err.message,
            config: {
                game: DEFAULT_GAME_CONFIG,
                ad: DEFAULT_AD_CONFIG
            }
        };
    }
};

/**
 * 比较版本号
 * @returns -1: v1 < v2, 0: v1 = v2, 1: v1 > v2
 */
function compareVersion(v1, v2) {
    const parts1 = v1.split('.').map(Number);
    const parts2 = v2.split('.').map(Number);

    for (let i = 0; i < 3; i++) {
        const p1 = parts1[i] || 0;
        const p2 = parts2[i] || 0;
        if (p1 < p2) return -1;
        if (p1 > p2) return 1;
    }
    return 0;
}
```

---

## 7. 云数据库集合

### 7.1 users 集合

```javascript
// 用户数据结构
{
    _id: "自动生成",
    _openid: "用户openid",
    nickname: "昵称",
    avatarUrl: "头像URL",
    highestFloor: 0,           // 最高层数
    highestScore: 0,           // 最高分数
    totalCoins: 0,             // 总金币
    totalGames: 0,             // 总局数
    settings: {                // 用户设置
        bgmEnabled: true,
        sfxEnabled: true,
        vibrationEnabled: true
    },
    createdAt: "创建时间",
    updatedAt: "更新时间"
}

// 索引
- _openid: 唯一索引
- highestFloor: 普通索引 (排行榜查询)
- highestScore: 普通索引 (排行榜查询)
```

### 7.2 scores 集合

```javascript
// 游戏记录结构
{
    _id: "自动生成",
    openid: "用户openid",
    sessionId: "会话ID",
    mode: "endless",           // 游戏模式
    floor: 100,                // 层数
    score: 5000,               // 分数
    coins: 50,                 // 获得金币
    duration: 120,             // 游戏时长(秒)
    deathReason: "fall_void",  // 死亡原因
    createdAt: "创建时间"
}

// 索引
- openid: 普通索引 (用户查询)
- createdAt: 普通索引 (时间查询)
```

### 7.3 configs 集合

```javascript
// 配置数据结构
{
    _id: "自动生成",
    type: "game",              // 配置类型: game, ad, version
    data: {                    // 配置数据
        // ... 具体配置
    },
    version: 1,                // 配置版本
    updatedAt: "更新时间"
}

// 索引
- type: 唯一索引
```

---

## 8. 安全规则

### 8.1 数据库安全规则

```json
// users 集合
{
    "read": "auth.openid == doc._openid || doc._openid == 'public'",
    "write": "auth.openid == doc._openid"
}

// scores 集合
{
    "read": "auth.openid == doc.openid",
    "write": false  // 只允许云函数写入
}

// configs 集合
{
    "read": true,
    "write": false  // 只允许管理员修改
}
```

### 8.2 云函数安全

- 所有写入操作通过云函数
- 云函数内校验数据合法性
- 敏感操作记录日志
- rate limiting 防止滥用
