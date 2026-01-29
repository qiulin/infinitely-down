# 代码规范

> 无限向下 - TypeScript 与 Cocos Creator 编码规范

## 1. TypeScript 命名规范

### 1.1 基础命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 类/接口 | PascalCase | `GameManager`, `PlayerData` |
| 枚举 | PascalCase | `GameState`, `PlatformType` |
| 枚举值 | UPPER_SNAKE_CASE | `PLAYING`, `GAME_OVER` |
| 函数/方法 | camelCase | `startGame()`, `updateScore()` |
| 变量 | camelCase | `currentScore`, `isPlaying` |
| 常量 | UPPER_SNAKE_CASE | `MAX_SPEED`, `SCREEN_WIDTH` |
| 私有成员 | _camelCase | `_isInitialized`, `_timer` |
| 事件名 | UPPER_SNAKE_CASE | `GAME_START`, `PLAYER_DEATH` |

### 1.2 命名约定

```typescript
// 好的命名
class PlatformManager { }
interface PlayerConfig { }
enum GameState { PLAYING, PAUSED }
const MAX_REVIVE_COUNT = 1;
function calculateScore(floor: number): number { }
let currentFloor: number;
private _isActive: boolean;

// 避免的命名
class platformManager { }    // 类应使用 PascalCase
interface playerconfig { }   // 接口应使用 PascalCase
enum gameState { playing }   // 枚举值应使用 UPPER_SNAKE_CASE
const max_revive = 1;        // 常量应使用 UPPER_SNAKE_CASE
function CalculateScore() { }  // 函数应使用 camelCase
```

### 1.3 布尔变量命名

```typescript
// 使用 is/has/can/should 前缀
let isPlaying: boolean;
let hasRevived: boolean;
let canJump: boolean;
let shouldShowAd: boolean;

// 避免
let playing: boolean;    // 含义不明确
let revived: boolean;    // 不够清晰
```

---

## 2. Cocos Creator 组件规范

### 2.1 组件基础结构

```typescript
import { _decorator, Component, Node } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 组件说明
 */
@ccclass('MyComponent')
export class MyComponent extends Component {

    // ============ 属性装饰器区 ============

    @property(Node)
    public targetNode: Node = null;

    @property
    public speed: number = 100;

    // ============ 私有变量区 ============

    private _isInitialized: boolean = false;
    private _timer: number = 0;

    // ============ 生命周期方法 ============

    onLoad() {
        // 初始化
    }

    start() {
        // 延迟初始化
    }

    update(deltaTime: number) {
        // 每帧更新
    }

    onDestroy() {
        // 清理
    }

    // ============ 公开方法 ============

    public doSomething(): void {
        // 公开接口
    }

    // ============ 私有方法 ============

    private _internalMethod(): void {
        // 内部实现
    }
}
```

### 2.2 @property 装饰器使用

```typescript
// 节点引用
@property(Node)
public playerNode: Node = null;

// 预制体引用
@property(Prefab)
public platformPrefab: Prefab = null;

// 组件引用
@property(Label)
public scoreLabel: Label = null;

// 基础类型
@property
public speed: number = 100;

@property
public isEnabled: boolean = true;

@property
public playerName: string = '';

// 数组
@property([Node])
public targetNodes: Node[] = [];

// 枚举 (显示下拉选择)
@property({ type: Enum(PlatformType) })
public platformType: PlatformType = PlatformType.NORMAL;

// 范围限制
@property({ min: 0, max: 100, step: 1, slide: true })
public volume: number = 80;

// 分组显示
@property({ group: { name: 'Movement' } })
public moveSpeed: number = 100;

// 只读显示
@property({ readonly: true })
public debugInfo: string = '';
```

### 2.3 生命周期方法顺序

```typescript
// 推荐的方法顺序
@ccclass('MyComponent')
export class MyComponent extends Component {

    // 1. 属性声明
    @property(Node) target: Node = null;

    // 2. 私有变量
    private _value: number = 0;

    // 3. 生命周期 (按执行顺序)
    onLoad() { }
    onEnable() { }
    start() { }
    update(dt: number) { }
    lateUpdate(dt: number) { }
    onDisable() { }
    onDestroy() { }

    // 4. 公开方法
    public initialize(): void { }

    // 5. 事件回调
    protected onButtonClick(): void { }

    // 6. 私有方法
    private _doSomething(): void { }
}
```

---

## 3. 文件与目录规范

### 3.1 脚本目录结构

```
assets/scripts/
├── managers/              # 管理器 (单例)
│   ├── GameManager.ts
│   ├── AudioManager.ts
│   └── UIManager.ts
│
├── components/            # 游戏组件
│   ├── Player.ts
│   ├── CameraFollow.ts
│   └── platforms/
│       ├── BasePlatform.ts
│       ├── NormalPlatform.ts
│       └── MovingPlatform.ts
│
├── ui/                    # UI 组件
│   ├── HomeUI.ts
│   ├── GameUI.ts
│   └── popups/
│       ├── BasePopup.ts
│       ├── ResultPopup.ts
│       └── PausePopup.ts
│
├── data/                  # 数据定义
│   ├── DataTypes.ts       # 接口定义
│   ├── GameConfig.ts      # 配置常量
│   └── EventTypes.ts      # 事件定义
│
├── utils/                 # 工具类
│   ├── ObjectPool.ts
│   ├── EventBus.ts
│   └── MathUtils.ts
│
└── platform/              # 平台适配
    ├── WechatAPI.ts
    ├── CloudAPI.ts
    └── AdAPI.ts
```

### 3.2 文件命名

| 类型 | 命名规范 | 示例 |
|------|----------|------|
| 组件脚本 | PascalCase | `GameManager.ts` |
| 工具类 | PascalCase | `ObjectPool.ts` |
| 类型定义 | PascalCase + Types | `DataTypes.ts` |
| 配置文件 | PascalCase + Config | `GameConfig.ts` |
| 接口文件 | I + PascalCase | `IPlayerData.ts` (可选) |

---

## 4. 注释规范

### 4.1 文件注释

```typescript
/**
 * @file GameManager.ts
 * @description 游戏主管理器，控制游戏状态和生命周期
 * @author Your Name
 * @date 2024-01-01
 */
```

### 4.2 类/接口注释

```typescript
/**
 * 游戏管理器
 * 负责控制游戏的整体状态和生命周期
 *
 * @example
 * GameManager.instance.startGame();
 * GameManager.instance.pauseGame();
 */
@ccclass('GameManager')
export class GameManager extends Component {
    // ...
}
```

### 4.3 方法注释

```typescript
/**
 * 计算玩家得分
 *
 * @param floor - 当前层数
 * @param combo - 连击数
 * @returns 计算后的分数
 */
public calculateScore(floor: number, combo: number): number {
    // ...
}
```

### 4.4 行内注释

```typescript
// 好的注释：解释为什么
// 使用 0.1 而不是 0 是为了避免浮点数精度问题
const threshold = 0.1;

// 避免的注释：解释是什么 (代码本身已经说明)
// 将分数加 10
score += 10;
```

---

## 5. 代码风格

### 5.1 格式化配置

**.prettierrc**:
```json
{
    "semi": true,
    "singleQuote": true,
    "tabWidth": 4,
    "trailingComma": "none",
    "printWidth": 100
}
```

**.eslintrc.json**:
```json
{
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended"
    ],
    "rules": {
        "indent": ["error", 4],
        "quotes": ["error", "single"],
        "semi": ["error", "always"],
        "@typescript-eslint/explicit-function-return-type": "warn",
        "@typescript-eslint/no-unused-vars": "error"
    }
}
```

### 5.2 代码风格示例

```typescript
// 导入语句分组
import { _decorator, Component, Node, Vec3 } from 'cc';
import { GameManager } from '../managers/GameManager';
import { PlatformType } from '../data/DataTypes';
import { GAME_CONSTANTS } from '../data/GameConfig';

const { ccclass, property } = _decorator;

// 类定义
@ccclass('Player')
export class Player extends Component {

    // 属性用空行分隔
    @property(Node)
    public sprite: Node = null;

    @property
    public moveSpeed: number = 400;

    // 私有变量
    private _velocity: Vec3 = new Vec3();
    private _isGrounded: boolean = false;

    // 方法之间空一行
    onLoad(): void {
        this._initialize();
    }

    update(deltaTime: number): void {
        this._updateMovement(deltaTime);
        this._checkBoundary();
    }

    // 公开方法
    public jump(force: number): void {
        if (!this._isGrounded) return;

        this._velocity.y = force;
        this._isGrounded = false;
    }

    // 私有方法
    private _initialize(): void {
        // 初始化逻辑
    }

    private _updateMovement(dt: number): void {
        // 移动更新
    }

    private _checkBoundary(): void {
        // 边界检测
    }
}
```

---

## 6. 单例模式

### 6.1 管理器单例

```typescript
@ccclass('GameManager')
export class GameManager extends Component {

    private static _instance: GameManager | null = null;

    public static get instance(): GameManager {
        return this._instance!;
    }

    onLoad(): void {
        if (GameManager._instance) {
            this.destroy();
            return;
        }
        GameManager._instance = this;
    }

    onDestroy(): void {
        if (GameManager._instance === this) {
            GameManager._instance = null;
        }
    }
}
```

### 6.2 纯 TypeScript 单例

```typescript
export class EventBus {

    private static _instance: EventBus | null = null;

    public static get instance(): EventBus {
        if (!this._instance) {
            this._instance = new EventBus();
        }
        return this._instance;
    }

    private constructor() {
        // 私有构造函数
    }
}
```

---

## 7. 事件系统使用

### 7.1 事件定义

```typescript
// EventTypes.ts
export const GameEvents = {
    GAME_START: 'game:start',
    GAME_PAUSE: 'game:pause',
    GAME_OVER: 'game:over',
    SCORE_CHANGE: 'game:score-change',
    FLOOR_CHANGE: 'game:floor-change'
} as const;
```

### 7.2 事件发送与监听

```typescript
import { EventBus } from '../utils/EventBus';
import { GameEvents } from '../data/EventTypes';

// 发送事件
EventBus.instance.emit(GameEvents.SCORE_CHANGE, { score: 100, delta: 10 });

// 监听事件
EventBus.instance.on(GameEvents.SCORE_CHANGE, this._onScoreChange, this);

// 移除监听
EventBus.instance.off(GameEvents.SCORE_CHANGE, this._onScoreChange, this);

// 回调处理
private _onScoreChange(data: { score: number; delta: number }): void {
    console.log('Score changed:', data.score);
}
```

---

## 8. 异步处理

### 8.1 Promise 使用

```typescript
// 异步方法
public async loadResources(): Promise<void> {
    try {
        await this._loadTextures();
        await this._loadAudio();
        console.log('Resources loaded');
    } catch (error) {
        console.error('Load failed:', error);
    }
}

// 私有异步方法
private _loadTextures(): Promise<void> {
    return new Promise((resolve, reject) => {
        resources.loadDir('textures', SpriteFrame, (err, assets) => {
            if (err) {
                reject(err);
            } else {
                resolve();
            }
        });
    });
}
```

### 8.2 回调转 Promise

```typescript
// 封装微信 API
public static wxLogin(): Promise<string> {
    return new Promise((resolve, reject) => {
        wx.login({
            success: (res) => resolve(res.code),
            fail: (err) => reject(err)
        });
    });
}

// 使用
const code = await WechatAPI.wxLogin();
```

---

## 9. 类型安全

### 9.1 避免 any

```typescript
// 避免
function processData(data: any): void { }

// 推荐
function processData(data: PlayerData): void { }

// 如果类型不确定，使用 unknown
function processUnknown(data: unknown): void {
    if (typeof data === 'string') {
        // 现在 TypeScript 知道 data 是 string
    }
}
```

### 9.2 类型断言

```typescript
// 避免非空断言 (!)
const node = this.getComponent(Label)!;

// 推荐空值检查
const node = this.getComponent(Label);
if (!node) {
    console.error('Label component not found');
    return;
}

// 或使用可选链
this.getComponent(Label)?.string = 'text';
```

### 9.3 泛型使用

```typescript
// 对象池泛型
class ObjectPool<T> {
    private _pool: T[] = [];
    private _createFn: () => T;

    constructor(createFn: () => T) {
        this._createFn = createFn;
    }

    public get(): T {
        return this._pool.pop() ?? this._createFn();
    }

    public put(item: T): void {
        this._pool.push(item);
    }
}

// 使用
const platformPool = new ObjectPool<BasePlatform>(() => {
    return instantiate(this.prefab).getComponent(BasePlatform);
});
```
