# 微信 API 集成规范

> 无限向下 - 微信小游戏 API 接入文档

## 1. API 概览

### 1.1 MVP 阶段使用的 API

| 功能 | API | 说明 |
|------|-----|------|
| 登录 | `wx.login` | 获取登录凭证 |
| 用户信息 | `wx.getUserInfo` | 获取昵称头像 |
| 数据存储 | `wx.setStorageSync` / `wx.getStorageSync` | 本地存储 |
| 分享 | `wx.shareAppMessage` / `wx.onShareAppMessage` | 分享功能 |
| 排行榜 | 开放数据域 | 好友排行 |
| 广告 | `wx.createRewardedVideoAd` 等 | 广告系统 |
| 云开发 | `wx.cloud` | 云函数/云数据库 |

---

## 2. 登录授权

### 2.1 登录流程

```
┌─────────────┐
│  游戏启动   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ wx.login()  │  获取临时登录凭证 code
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 云函数 login│  code 换取 openid
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 存储 openid │  本地缓存
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 获取用户信息│  wx.getUserInfo (可选)
└─────────────┘
```

### 2.2 代码实现

```typescript
// platform/WechatAPI.ts

/**
 * 微信 API 封装
 */
export class WechatAPI {

    private static _openid: string = '';

    /**
     * 获取当前用户 openid
     */
    static get openid(): string {
        return this._openid;
    }

    /**
     * 微信登录
     * @returns Promise<string> openid
     */
    static async login(): Promise<string> {
        // 1. 获取 code
        const code = await this._wxLogin();

        // 2. 调用云函数获取 openid
        const result = await this._callCloudFunction('login', { code });
        this._openid = result.openid;

        // 3. 缓存 openid
        this._saveToStorage('openid', this._openid);

        return this._openid;
    }

    /**
     * wx.login Promise 封装
     */
    private static _wxLogin(): Promise<string> {
        return new Promise((resolve, reject) => {
            wx.login({
                success: (res) => {
                    if (res.code) {
                        resolve(res.code);
                    } else {
                        reject(new Error('wx.login failed: no code'));
                    }
                },
                fail: (err) => reject(err)
            });
        });
    }

    /**
     * 获取用户信息
     */
    static async getUserInfo(): Promise<WechatUserInfo> {
        return new Promise((resolve, reject) => {
            wx.getUserInfo({
                success: (res) => {
                    resolve({
                        nickname: res.userInfo.nickName,
                        avatarUrl: res.userInfo.avatarUrl
                    });
                },
                fail: (err) => reject(err)
            });
        });
    }

    /**
     * 调用云函数
     */
    private static _callCloudFunction(name: string, data: any): Promise<any> {
        return new Promise((resolve, reject) => {
            wx.cloud.callFunction({
                name,
                data,
                success: (res) => resolve(res.result),
                fail: (err) => reject(err)
            });
        });
    }

    /**
     * 本地存储
     */
    private static _saveToStorage(key: string, value: any): void {
        try {
            wx.setStorageSync(key, value);
        } catch (e) {
            console.error('Storage save failed:', e);
        }
    }

    /**
     * 读取本地存储
     */
    static getFromStorage<T>(key: string, defaultValue: T): T {
        try {
            const value = wx.getStorageSync(key);
            return value !== '' ? value : defaultValue;
        } catch (e) {
            console.error('Storage read failed:', e);
            return defaultValue;
        }
    }
}

/**
 * 用户信息接口
 */
interface WechatUserInfo {
    nickname: string;
    avatarUrl: string;
}
```

---

## 3. 分享功能

### 3.1 分享配置

```typescript
/**
 * 配置分享
 */
static setupShare(): void {
    // 右上角分享按钮
    wx.showShareMenu({
        withShareTicket: true,
        menus: ['shareAppMessage', 'shareTimeline']
    });

    // 监听分享事件
    wx.onShareAppMessage(() => this._getShareConfig());
}

/**
 * 主动触发分享
 */
static shareToFriend(score: number, floor: number): void {
    wx.shareAppMessage(this._getShareConfig(score, floor));
}

/**
 * 获取分享配置
 */
private static _getShareConfig(score?: number, floor?: number): WechatShareConfig {
    // 基础分享
    if (!score) {
        return {
            title: '无限向下 - 经典下落挑战！',
            imageUrl: 'share/default.png',
            query: ''
        };
    }

    // 成绩分享
    return {
        title: `我下到了第 ${floor} 层！你能超过我吗？`,
        imageUrl: 'share/score.png',
        query: `inviter=${this._openid}&score=${score}`
    };
}

interface WechatShareConfig {
    title: string;
    imageUrl: string;
    query: string;
}
```

### 3.2 分享回调处理

```typescript
/**
 * 处理分享回调 (被邀请者进入)
 */
static handleShareCallback(): void {
    const launchOptions = wx.getLaunchOptionsSync();
    const query = launchOptions.query;

    if (query.inviter) {
        // 记录邀请关系
        this._recordInvite(query.inviter);
    }
}

private static async _recordInvite(inviterId: string): Promise<void> {
    await this._callCloudFunction('recordInvite', {
        inviterId,
        inviteeId: this._openid
    });
}
```

---

## 4. 排行榜 (开放数据域)

### 4.1 架构说明

```
┌─────────────────────────────────────────┐
│            主域 (Game)                   │
│  - 游戏逻辑                              │
│  - 提交分数到开放数据域                  │
│  - 显示排行榜 Canvas                     │
└───────────────────┬─────────────────────┘
                    │ postMessage
                    ▼
┌─────────────────────────────────────────┐
│         开放数据域 (Open Data)           │
│  - 读取好友数据                          │
│  - 渲染排行榜                            │
│  - 返回渲染结果到共享 Canvas             │
└─────────────────────────────────────────┘
```

### 4.2 主域代码

```typescript
// platform/LeaderboardAPI.ts

export class LeaderboardAPI {

    /**
     * 提交分数到开放数据域
     */
    static submitScore(score: number, floor: number): void {
        wx.setUserCloudStorage({
            KVDataList: [
                { key: 'score', value: String(score) },
                { key: 'floor', value: String(floor) },
                { key: 'updateTime', value: String(Date.now()) }
            ],
            success: () => console.log('Score submitted'),
            fail: (err) => console.error('Submit failed:', err)
        });
    }

    /**
     * 请求开放数据域渲染排行榜
     */
    static requestRenderLeaderboard(type: 'friend' | 'group'): void {
        const openDataContext = wx.getOpenDataContext();
        openDataContext.postMessage({
            action: 'render',
            type,
            width: 600,
            height: 800
        });
    }

    /**
     * 清除排行榜显示
     */
    static clearLeaderboard(): void {
        const openDataContext = wx.getOpenDataContext();
        openDataContext.postMessage({
            action: 'clear'
        });
    }
}
```

### 4.3 开放数据域代码

```typescript
// open-data/index.ts (开放数据域入口)

const sharedCanvas = wx.getSharedCanvas();
const ctx = sharedCanvas.getContext('2d');

wx.onMessage((message) => {
    switch (message.action) {
        case 'render':
            renderLeaderboard(message.type, message.width, message.height);
            break;
        case 'clear':
            clearCanvas();
            break;
    }
});

async function renderLeaderboard(type: string, width: number, height: number): Promise<void> {
    // 获取好友数据
    const data = await getFriendData();

    // 排序
    data.sort((a, b) => {
        const scoreA = parseInt(a.KVDataList.find(kv => kv.key === 'score')?.value || '0');
        const scoreB = parseInt(b.KVDataList.find(kv => kv.key === 'score')?.value || '0');
        return scoreB - scoreA;
    });

    // 渲染
    clearCanvas();
    renderList(data, width, height);
}

function getFriendData(): Promise<WxFriendData[]> {
    return new Promise((resolve) => {
        wx.getFriendCloudStorage({
            keyList: ['score', 'floor', 'updateTime'],
            success: (res) => resolve(res.data),
            fail: () => resolve([])
        });
    });
}

function clearCanvas(): void {
    ctx.clearRect(0, 0, sharedCanvas.width, sharedCanvas.height);
}

function renderList(data: WxFriendData[], width: number, height: number): void {
    // 设置画布大小
    sharedCanvas.width = width;
    sharedCanvas.height = height;

    // 渲染背景
    ctx.fillStyle = '#1a1a2e';
    ctx.fillRect(0, 0, width, height);

    // 渲染列表项
    const itemHeight = 80;
    data.slice(0, 10).forEach((friend, index) => {
        const y = index * itemHeight;
        renderListItem(friend, index + 1, y, width);
    });
}

function renderListItem(friend: WxFriendData, rank: number, y: number, width: number): void {
    const score = friend.KVDataList.find(kv => kv.key === 'score')?.value || '0';

    // 排名
    ctx.fillStyle = rank <= 3 ? '#ffd700' : '#ffffff';
    ctx.font = 'bold 24px Arial';
    ctx.fillText(String(rank), 20, y + 50);

    // 昵称
    ctx.fillStyle = '#ffffff';
    ctx.font = '20px Arial';
    ctx.fillText(friend.nickname, 80, y + 45);

    // 分数
    ctx.fillStyle = '#4ecdc4';
    ctx.font = 'bold 22px Arial';
    ctx.textAlign = 'right';
    ctx.fillText(score, width - 20, y + 45);
    ctx.textAlign = 'left';
}

interface WxFriendData {
    nickname: string;
    avatarUrl: string;
    KVDataList: { key: string; value: string }[];
}
```

---

## 5. 广告系统

### 5.1 广告类型

| 类型 | 使用场景 | API |
|------|----------|-----|
| 激励视频 | 复活、双倍金币 | `wx.createRewardedVideoAd` |
| 插屏广告 | 每 N 局 | `wx.createInterstitialAd` |
| Banner | 主界面 | `wx.createBannerAd` |

### 5.2 广告封装

```typescript
// platform/AdAPI.ts

export class AdAPI {

    private static _rewardedVideoAd: WechatMiniprogram.RewardedVideoAd | null = null;
    private static _interstitialAd: WechatMiniprogram.InterstitialAd | null = null;
    private static _bannerAd: WechatMiniprogram.BannerAd | null = null;

    // 广告单元 ID (替换为实际 ID)
    private static readonly REWARDED_AD_UNIT = 'adunit-xxx';
    private static readonly INTERSTITIAL_AD_UNIT = 'adunit-yyy';
    private static readonly BANNER_AD_UNIT = 'adunit-zzz';

    /**
     * 初始化广告
     */
    static initialize(): void {
        this._initRewardedVideoAd();
        this._initInterstitialAd();
    }

    // ==================== 激励视频广告 ====================

    private static _initRewardedVideoAd(): void {
        this._rewardedVideoAd = wx.createRewardedVideoAd({
            adUnitId: this.REWARDED_AD_UNIT
        });

        // 预加载
        this._rewardedVideoAd.load().catch(() => {});
    }

    /**
     * 显示激励视频广告
     * @returns Promise<boolean> 是否完整观看
     */
    static showRewardedVideo(): Promise<boolean> {
        return new Promise((resolve) => {
            if (!this._rewardedVideoAd) {
                resolve(false);
                return;
            }

            // 监听关闭事件
            const onClose = (res: { isEnded: boolean }) => {
                this._rewardedVideoAd!.offClose(onClose);
                resolve(res.isEnded);

                // 预加载下一个
                this._rewardedVideoAd!.load().catch(() => {});
            };

            this._rewardedVideoAd.onClose(onClose);

            // 显示广告
            this._rewardedVideoAd.show().catch(() => {
                // 显示失败，尝试重新加载后显示
                this._rewardedVideoAd!.load()
                    .then(() => this._rewardedVideoAd!.show())
                    .catch(() => resolve(false));
            });
        });
    }

    // ==================== 插屏广告 ====================

    private static _initInterstitialAd(): void {
        this._interstitialAd = wx.createInterstitialAd({
            adUnitId: this.INTERSTITIAL_AD_UNIT
        });

        // 预加载
        this._interstitialAd.load().catch(() => {});
    }

    /**
     * 显示插屏广告
     */
    static showInterstitial(): void {
        if (!this._interstitialAd) return;

        this._interstitialAd.show().catch(() => {
            // 显示失败，静默处理
        });

        // 预加载下一个
        this._interstitialAd.load().catch(() => {});
    }

    // ==================== Banner 广告 ====================

    /**
     * 创建 Banner 广告
     * @param bottom 距底部距离
     */
    static createBanner(bottom: number = 0): void {
        const systemInfo = wx.getSystemInfoSync();
        const bannerWidth = 300;

        this._bannerAd = wx.createBannerAd({
            adUnitId: this.BANNER_AD_UNIT,
            style: {
                left: (systemInfo.windowWidth - bannerWidth) / 2,
                top: systemInfo.windowHeight - bottom - 100,  // 大致高度
                width: bannerWidth
            }
        });

        // 尺寸变化时重新定位
        this._bannerAd.onResize((res) => {
            this._bannerAd!.style.top = systemInfo.windowHeight - bottom - res.height;
        });
    }

    /**
     * 显示 Banner
     */
    static showBanner(): void {
        this._bannerAd?.show().catch(() => {});
    }

    /**
     * 隐藏 Banner
     */
    static hideBanner(): void {
        this._bannerAd?.hide();
    }

    /**
     * 销毁 Banner
     */
    static destroyBanner(): void {
        this._bannerAd?.destroy();
        this._bannerAd = null;
    }
}
```

### 5.3 广告使用示例

```typescript
// 复活时使用激励视频
async function onReviveButtonClick(): Promise<void> {
    const watched = await AdAPI.showRewardedVideo();
    if (watched) {
        // 发放复活奖励
        GameManager.instance.revive();
    } else {
        // 未看完，不发放奖励
        showToast('观看完整广告后可获得复活机会');
    }
}

// 双倍金币
async function onDoubleCoinsClick(): Promise<void> {
    const watched = await AdAPI.showRewardedVideo();
    if (watched) {
        DataManager.instance.addCoins(currentCoins);  // 再加一倍
        showToast('金币已翻倍！');
    }
}

// 插屏广告 (每 5 局)
function onGameOver(): void {
    gameCount++;
    if (gameCount % 5 === 0) {
        AdAPI.showInterstitial();
    }
}
```

---

## 6. 数据存储

### 6.1 本地存储

```typescript
// platform/StorageAPI.ts

export class StorageAPI {

    private static readonly PLAYER_DATA_KEY = 'player_data';
    private static readonly SETTINGS_KEY = 'settings';

    /**
     * 保存玩家数据
     */
    static savePlayerData(data: PlayerData): void {
        try {
            wx.setStorageSync(this.PLAYER_DATA_KEY, JSON.stringify(data));
        } catch (e) {
            console.error('Save player data failed:', e);
        }
    }

    /**
     * 读取玩家数据
     */
    static loadPlayerData(): PlayerData | null {
        try {
            const str = wx.getStorageSync(this.PLAYER_DATA_KEY);
            return str ? JSON.parse(str) : null;
        } catch (e) {
            console.error('Load player data failed:', e);
            return null;
        }
    }

    /**
     * 保存设置
     */
    static saveSettings(settings: UserSettings): void {
        try {
            wx.setStorageSync(this.SETTINGS_KEY, JSON.stringify(settings));
        } catch (e) {
            console.error('Save settings failed:', e);
        }
    }

    /**
     * 读取设置
     */
    static loadSettings(): UserSettings | null {
        try {
            const str = wx.getStorageSync(this.SETTINGS_KEY);
            return str ? JSON.parse(str) : null;
        } catch (e) {
            console.error('Load settings failed:', e);
            return null;
        }
    }

    /**
     * 清除所有数据
     */
    static clearAll(): void {
        try {
            wx.clearStorageSync();
        } catch (e) {
            console.error('Clear storage failed:', e);
        }
    }
}
```

### 6.2 存储限制

| 限制项 | 值 |
|--------|-----|
| 单个 key 大小 | 1MB |
| 总存储空间 | 10MB |
| 同步 API 阻塞 | 是 |

**最佳实践**:
- 频繁读写使用内存缓存
- 关键节点才写入 Storage
- 避免存储大型数据

---

## 7. 错误处理

### 7.1 统一错误处理

```typescript
export class WechatErrorHandler {

    /**
     * 处理微信 API 错误
     */
    static handle(err: WechatError, operation: string): void {
        console.error(`[${operation}] Error:`, err);

        // 根据错误码处理
        switch (err.errCode) {
            case 10001:
                // 系统错误
                this._showToast('系统繁忙，请稍后再试');
                break;
            case -1:
                // 网络错误
                this._showToast('网络连接失败');
                break;
            default:
                this._showToast('操作失败，请重试');
        }
    }

    private static _showToast(message: string): void {
        wx.showToast({
            title: message,
            icon: 'none',
            duration: 2000
        });
    }
}

interface WechatError {
    errCode: number;
    errMsg: string;
}
```

### 7.2 API 调用包装

```typescript
/**
 * 安全调用微信 API
 */
async function safeWxCall<T>(
    operation: string,
    fn: () => Promise<T>,
    fallback?: T
): Promise<T | undefined> {
    try {
        return await fn();
    } catch (err) {
        WechatErrorHandler.handle(err as WechatError, operation);
        return fallback;
    }
}

// 使用示例
const userInfo = await safeWxCall(
    'getUserInfo',
    () => WechatAPI.getUserInfo(),
    { nickname: '游客', avatarUrl: '' }
);
```
