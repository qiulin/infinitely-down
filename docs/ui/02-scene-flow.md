# 场景流转设计

> 无限向下 - 场景与弹窗流转文档

## 1. 场景概览

```
┌─────────────────────────────────────────────────────────────┐
│                       场景流转图                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐                                          │
│   │ LaunchScene │  启动场景                                 │
│   │  (微信授权)  │                                          │
│   └──────┬──────┘                                          │
│          │ 授权完成/跳过                                    │
│          ▼                                                  │
│   ┌─────────────┐      开始游戏      ┌─────────────┐       │
│   │  HomeScene  │ ──────────────────▶│  GameScene  │       │
│   │   (主界面)   │                    │  (游戏场景)  │       │
│   └──────┬──────┘◀────────────────── └─────────────┘       │
│          │           返回主界面                              │
│          │                                                  │
│   ┌──────┴──────────────────────────────────┐              │
│   │              主界面功能                   │              │
│   │  ┌────────┐ ┌────────┐ ┌────────┐      │              │
│   │  │排行榜  │ │ 设置   │ │ 商店   │ ...  │              │
│   │  └────────┘ └────────┘ └────────┘      │              │
│   └─────────────────────────────────────────┘              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. LaunchScene (启动场景)

### 2.1 功能说明

- 游戏入口场景
- 显示 Logo/启动画面
- 微信登录授权
- 资源预加载
- 版本检查

### 2.2 节点结构

```
LaunchScene
├── Canvas
│   ├── Background           # 启动背景图
│   ├── Logo                 # 游戏 Logo
│   ├── LoadingBar           # 加载进度条
│   │   ├── ProgressBg       # 进度条背景
│   │   └── ProgressFill     # 进度条填充
│   ├── LoadingText          # 加载提示文字
│   └── VersionLabel         # 版本号显示
└── Scripts
    └── LaunchController     # 启动控制器
```

### 2.3 流程

```
启动
 │
 ▼
┌─────────────┐
│ 显示 Logo   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 检查登录态  │
└──────┬──────┘
       │
   ┌───┴───┐
   │ 已登录?│
   └───┬───┘
       │
  否 ──┴── 是
  │        │
  ▼        │
┌─────────┐│
│微信授权 ││
└────┬────┘│
     │     │
     ▼     │
┌─────────┐│
│获取用户 ││
│信息    ││
└────┬────┘│
     │     │
     └──┬──┘
        │
        ▼
┌─────────────┐
│ 预加载资源  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ 进入主界面  │
└─────────────┘
```

---

## 3. HomeScene (主界面场景)

### 3.1 功能说明

- 游戏主菜单
- 模式选择入口
- 排行榜入口
- 设置入口
- 好友排行榜展示

### 3.2 节点结构

```
HomeScene
├── Canvas
│   ├── Background           # 主界面背景
│   │   └── ScrollingBg      # 滚动背景效果
│   │
│   ├── TopBar               # 顶部栏
│   │   ├── CoinDisplay      # 金币显示
│   │   │   ├── CoinIcon
│   │   │   └── CoinLabel
│   │   ├── DiamondDisplay   # 钻石显示
│   │   │   ├── DiamondIcon
│   │   │   └── DiamondLabel
│   │   └── SettingsBtn      # 设置按钮
│   │
│   ├── CenterArea           # 中心区域
│   │   ├── GameTitle        # 游戏标题
│   │   ├── CharacterDisplay # 角色展示
│   │   └── HighScoreLabel   # 最高纪录
│   │
│   ├── ModeButtons          # 模式按钮区
│   │   ├── EndlessModeBtn   # 无尽模式
│   │   └── ChallengeModeBtn # 100层挑战
│   │
│   ├── BottomBar            # 底部栏
│   │   ├── LeaderboardBtn   # 排行榜按钮
│   │   ├── ShopBtn          # 商店按钮
│   │   └── ShareBtn         # 分享按钮
│   │
│   └── FriendRankPanel      # 好友排行侧边栏
│       ├── Title
│       ├── RankList         # 排行列表
│       └── MoreBtn          # 查看更多
│
├── PopupLayer               # 弹窗层
│   └── (动态加载弹窗)
│
└── Scripts
    └── HomeController       # 主界面控制器
```

### 3.3 交互流程

```
主界面加载完成
       │
       ▼
┌─────────────────┐
│ 加载玩家数据    │
│ 显示金币/钻石   │
│ 显示最高纪录    │
│ 加载好友排行    │
└────────┬────────┘
         │
    用户操作
    ┌────┴────────────────────┐
    │                         │
    ▼                         ▼
┌────────┐              ┌────────┐
│无尽模式│              │100层   │
│  按钮  │              │挑战按钮│
└───┬────┘              └───┬────┘
    │                       │
    └───────────┬───────────┘
                │
                ▼
        ┌─────────────┐
        │ 切换到      │
        │ GameScene   │
        └─────────────┘
```

---

## 4. GameScene (游戏场景)

### 4.1 功能说明

- 核心游戏玩法场景
- 玩家控制
- 平台生成
- HUD 显示
- 游戏结算

### 4.2 节点结构

```
GameScene
├── GameWorld                # 游戏世界 (随相机移动)
│   ├── Background           # 游戏背景
│   │   └── ParallaxLayers   # 视差滚动层
│   │
│   ├── PlatformContainer    # 平台容器
│   │   └── (动态生成平台)
│   │
│   ├── Player               # 玩家角色
│   │   ├── Sprite           # 角色精灵
│   │   ├── Shadow           # 影子
│   │   └── CollisionBox     # 碰撞盒
│   │
│   └── EffectContainer      # 特效容器
│       └── (动态生成特效)
│
├── Camera                   # 游戏相机
│   └── CameraFollow         # 相机跟随脚本
│
├── UICanvas                 # UI 画布 (固定)
│   ├── GameHUD              # 游戏 HUD
│   │   ├── FloorLabel       # 层数显示
│   │   ├── ScoreLabel       # 分数显示
│   │   ├── CoinLabel        # 金币显示
│   │   ├── ComboLabel       # 连击显示
│   │   └── PauseBtn         # 暂停按钮
│   │
│   ├── TouchArea            # 触摸输入区域
│   │
│   └── PopupLayer           # 弹窗层
│       └── (动态加载弹窗)
│
└── Scripts
    ├── GameController       # 游戏主控制器
    ├── InputManager         # 输入管理
    └── PlatformSpawner      # 平台生成器
```

### 4.3 游戏流程

```
进入 GameScene
       │
       ▼
┌─────────────────┐
│ 初始化游戏世界  │
│ - 生成初始平台  │
│ - 放置玩家     │
│ - 重置分数     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 开始倒计时      │
│ 3... 2... 1...  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 游戏循环        │
│ - 处理输入     │
│ - 更新物理     │
│ - 生成平台     │
│ - 检测碰撞     │
└────────┬────────┘
         │
    ┌────┴────┐
    │ 游戏结束?│
    └────┬────┘
         │
    是 ──┴── 否
    │        │
    ▼        └──▶ (继续循环)
┌─────────────┐
│ 显示死亡动画│
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│ 可复活?         │
│ (有广告复活次数)│
└────────┬────────┘
         │
    是 ──┴── 否
    │        │
    ▼        ▼
┌────────┐  ┌─────────┐
│复活弹窗│  │结算弹窗 │
└───┬────┘  └────┬────┘
    │            │
    │ 观看广告    │
    │ 或放弃      │
    │     │      │
    └─────┴──────┘
          │
      返回主界面
```

---

## 5. 弹窗系统

### 5.1 弹窗层级

```
PopupLayer (弹窗层)
├── MaskLayer        # 遮罩层 (点击关闭)
└── PopupContainer   # 弹窗容器
    └── ActivePopup  # 当前弹窗
```

### 5.2 ResultPopup (结算弹窗)

```
ResultPopup
├── Background           # 弹窗背景
├── Title                # "游戏结束"
├── ScoreArea            # 分数区域
│   ├── FloorLabel       # "下到 XX 层"
│   ├── ScoreLabel       # "得分 XXXX"
│   ├── CoinLabel        # "获得 XX 金币"
│   └── NewRecordBadge   # 新纪录徽章 (条件显示)
│
├── RankInfo             # 排名信息
│   └── BeatPercentLabel # "击败了 XX% 的玩家"
│
├── ButtonArea           # 按钮区域
│   ├── DoubleCoinsBtn   # "看广告双倍金币"
│   ├── ShareBtn         # "炫耀一下"
│   ├── RetryBtn         # "再来一局"
│   └── HomeBtn          # "返回主界面"
│
└── CloseBtn             # 关闭按钮
```

**结算弹窗流程**:

```
显示结算弹窗
       │
       ▼
┌─────────────────┐
│ 播放入场动画    │
│ 分数滚动显示    │
└────────┬────────┘
         │
    用户选择
    ┌────┴────────────────────────┐
    │         │         │         │
    ▼         ▼         ▼         ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│双倍  │ │分享  │ │再来  │ │返回  │
│金币  │ │成绩  │ │一局  │ │主界面│
└──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
   │        │        │        │
   ▼        ▼        │        │
播放广告  生成分享卡 │        │
   │      调用微信   │        │
   │      分享API    │        │
   │        │        │        │
   └────────┴────────┴────────┘
                │
                ▼
        关闭弹窗/切换场景
```

### 5.3 RevivePopup (复活弹窗)

```
RevivePopup
├── Background           # 弹窗背景
├── Title                # "复活机会"
├── Countdown            # 倒计时显示
│   └── TimerLabel       # "5... 4... 3..."
│
├── ReviveBtn            # "观看广告复活"
├── GiveUpBtn            # "放弃"
│
└── ReviveInfo           # 复活信息
    └── RemainingLabel   # "剩余 1 次复活机会"
```

**复活弹窗流程**:

```
玩家死亡
   │
   ▼
┌─────────────────┐
│ 检查复活次数    │
│ (本局是否已用)  │
└────────┬────────┘
         │
    有 ──┴── 无
    │        │
    ▼        ▼
┌──────────┐ 直接显示
│显示复活  │ 结算弹窗
│弹窗      │
└────┬─────┘
     │
┌────┴────┐
│5秒倒计时│
└────┬────┘
     │
观看广告 ──┬── 放弃/超时
     │     │
     ▼     ▼
┌──────┐ ┌──────┐
│复活  │ │结算  │
│继续  │ │弹窗  │
└──────┘ └──────┘
```

### 5.4 PausePopup (暂停弹窗)

```
PausePopup
├── Background           # 弹窗背景
├── Title                # "游戏暂停"
├── ButtonArea
│   ├── ResumeBtn        # "继续游戏"
│   ├── RestartBtn       # "重新开始"
│   ├── SettingsBtn      # "设置"
│   └── HomeBtn          # "返回主界面"
│
└── VolumeControls       # 音量控制
    ├── BGMSlider        # 背景音乐
    └── SFXSlider        # 音效
```

### 5.5 LeaderboardPopup (排行榜弹窗)

```
LeaderboardPopup
├── Background           # 弹窗背景
├── Title                # "排行榜"
├── TabBar               # 标签栏
│   ├── FriendTab        # "好友"
│   └── GlobalTab        # "全服" (后期)
│
├── RankList             # 排行列表
│   └── RankItem (循环)
│       ├── RankNumber   # 排名
│       ├── Avatar       # 头像
│       ├── Nickname     # 昵称
│       └── Score        # 分数
│
├── MyRank               # 我的排名
│   ├── RankLabel        # "我的排名: XX"
│   └── ScoreLabel       # "最高 XX 层"
│
└── CloseBtn             # 关闭按钮
```

---

## 6. 场景切换

### 6.1 切换管理器

```typescript
/**
 * 场景管理器
 */
class SceneManager {
    /**
     * 切换场景
     * @param sceneName 场景名称
     * @param transition 过渡效果
     */
    static async loadScene(
        sceneName: SceneName,
        transition: TransitionType = TransitionType.FADE
    ): Promise<void> {
        // 1. 显示过渡效果
        await TransitionManager.showTransition(transition);

        // 2. 加载新场景
        await new Promise<void>((resolve) => {
            director.loadScene(sceneName, () => resolve());
        });

        // 3. 隐藏过渡效果
        await TransitionManager.hideTransition(transition);
    }
}

/**
 * 过渡效果类型
 */
enum TransitionType {
    NONE = 'none',
    FADE = 'fade',
    SLIDE_LEFT = 'slide_left',
    SLIDE_RIGHT = 'slide_right'
}
```

### 6.2 过渡效果

| 场景切换 | 过渡效果 | 时长 |
|----------|----------|------|
| Launch → Home | 淡入淡出 | 0.5s |
| Home → Game | 左滑入 | 0.3s |
| Game → Home | 右滑出 | 0.3s |
| 弹窗显示 | 缩放弹入 | 0.2s |
| 弹窗关闭 | 缩放弹出 | 0.15s |

---

## 7. 数据传递

### 7.1 场景间数据

```typescript
/**
 * 场景数据管理
 */
class SceneData {
    // 进入游戏场景时的数据
    static gameSceneData: {
        mode: GameMode;           // 游戏模式
        characterId: string;      // 使用的角色
    };

    // 游戏结束时的数据
    static gameResultData: {
        result: GameResult;       // 游戏结果
        showDoubleCoins: boolean; // 是否显示双倍金币
    };
}
```

### 7.2 弹窗数据

```typescript
/**
 * 弹窗数据接口
 */
interface PopupData {
    // 结算弹窗
    ResultPopup: {
        result: GameResult;
        isNewRecord: boolean;
    };

    // 复活弹窗
    RevivePopup: {
        remainingRevives: number;
        countdownSeconds: number;
    };

    // 排行榜弹窗
    LeaderboardPopup: {
        type: LeaderboardType;
        initialTab: 'friend' | 'global';
    };
}
```
