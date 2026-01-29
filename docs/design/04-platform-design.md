# 平台系统设计

> 无限向下 - 平台组件与生成系统

## 1. 平台组件架构

```
┌─────────────────────────────────────────────────────────────┐
│                    BasePlatform (基类)                       │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  - type: PlatformType                                   ││
│  │  - width: number                                        ││
│  │  - height: number                                       ││
│  │  - isActive: boolean                                    ││
│  │  + onPlayerLand(): void                                 ││
│  │  + onPlayerLeave(): void                                ││
│  │  + reset(): void                                        ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│           ▲               ▲               ▲                 │
│           │               │               │                 │
│  ┌────────┴───┐   ┌──────┴─────┐   ┌─────┴──────┐         │
│  │NormalPlatform│ │MovingPlatform│ │BreakablePlatform│     │
│  │  (普通平台)  │ │  (移动平台)  │ │  (破碎平台)  │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. BasePlatform 基类

```typescript
import { _decorator, Component, Node, Collider2D, Vec3 } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 平台基类
 * 所有平台类型的父类，定义通用行为和接口
 */
@ccclass('BasePlatform')
export class BasePlatform extends Component {

    /** 平台类型 */
    @property
    public type: PlatformType = PlatformType.NORMAL;

    /** 平台宽度 */
    @property
    public width: number = 120;

    /** 平台高度 */
    @property
    public height: number = 20;

    /** 是否激活 */
    protected _isActive: boolean = true;

    /** 碰撞器组件 */
    protected collider: Collider2D | null = null;

    /** 所属层数 */
    public floorIndex: number = 0;

    /**
     * 组件初始化
     */
    onLoad() {
        this.collider = this.getComponent(Collider2D);
    }

    /**
     * 玩家落在平台上时调用
     * @param player 玩家节点
     */
    onPlayerLand(player: Node): void {
        // 子类重写
    }

    /**
     * 玩家离开平台时调用
     * @param player 玩家节点
     */
    onPlayerLeave(player: Node): void {
        // 子类重写
    }

    /**
     * 重置平台状态 (从对象池取出时调用)
     */
    reset(): void {
        this._isActive = true;
        this.node.active = true;
        if (this.collider) {
            this.collider.enabled = true;
        }
    }

    /**
     * 回收平台 (放回对象池时调用)
     */
    recycle(): void {
        this._isActive = false;
        this.node.active = false;
    }

    /**
     * 获取平台是否可碰撞
     */
    get isActive(): boolean {
        return this._isActive;
    }

    /**
     * 获取平台顶部 Y 坐标
     */
    get topY(): number {
        return this.node.position.y + this.height / 2;
    }

    /**
     * 获取平台左边界 X 坐标
     */
    get leftX(): number {
        return this.node.position.x - this.width / 2;
    }

    /**
     * 获取平台右边界 X 坐标
     */
    get rightX(): number {
        return this.node.position.x + this.width / 2;
    }
}
```

---

## 3. 三种 MVP 平台

### 3.1 NormalPlatform (普通平台)

```typescript
import { _decorator } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 普通平台
 * 最基础的平台，稳定不动，无特殊效果
 */
@ccclass('NormalPlatform')
export class NormalPlatform extends BasePlatform {

    onLoad() {
        super.onLoad();
        this.type = PlatformType.NORMAL;
    }

    /**
     * 玩家落在平台上
     */
    onPlayerLand(player: Node): void {
        // 普通平台无特殊逻辑
        // 可以播放落地音效
        AudioManager.instance.playSfx('land');
    }

    /**
     * 重置状态
     */
    reset(): void {
        super.reset();
        // 普通平台无需额外重置
    }
}
```

### 3.2 MovingPlatform (移动平台)

```typescript
import { _decorator, Vec3 } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 移动平台
 * 在水平方向左右移动的平台
 */
@ccclass('MovingPlatform')
export class MovingPlatform extends BasePlatform {

    /** 移动速度 (像素/秒) */
    @property
    public speed: number = 100;

    /** 移动范围 (左右各多少像素) */
    @property
    public moveRange: number = 80;

    /** 当前移动方向 (1: 右, -1: 左) */
    private direction: number = 1;

    /** 初始 X 位置 */
    private initialX: number = 0;

    /** 移动的玩家引用 (用于同步玩家位置) */
    private attachedPlayer: Node | null = null;

    onLoad() {
        super.onLoad();
        this.type = PlatformType.MOVING;
    }

    start() {
        this.initialX = this.node.position.x;
    }

    update(deltaTime: number) {
        if (!this._isActive) return;

        // 计算新位置
        const currentX = this.node.position.x;
        let newX = currentX + this.speed * this.direction * deltaTime;

        // 边界检测，反向
        if (newX >= this.initialX + this.moveRange) {
            newX = this.initialX + this.moveRange;
            this.direction = -1;
        } else if (newX <= this.initialX - this.moveRange) {
            newX = this.initialX - this.moveRange;
            this.direction = 1;
        }

        // 计算位移差
        const deltaX = newX - currentX;

        // 更新平台位置
        this.node.setPosition(newX, this.node.position.y);

        // 同步移动玩家
        if (this.attachedPlayer) {
            const playerPos = this.attachedPlayer.position;
            this.attachedPlayer.setPosition(
                playerPos.x + deltaX,
                playerPos.y
            );
        }
    }

    /**
     * 玩家落在平台上
     */
    onPlayerLand(player: Node): void {
        this.attachedPlayer = player;
        AudioManager.instance.playSfx('land');
    }

    /**
     * 玩家离开平台
     */
    onPlayerLeave(player: Node): void {
        if (this.attachedPlayer === player) {
            this.attachedPlayer = null;
        }
    }

    /**
     * 重置状态
     */
    reset(): void {
        super.reset();
        this.attachedPlayer = null;
        this.direction = Math.random() > 0.5 ? 1 : -1;  // 随机初始方向
    }

    /**
     * 设置移动参数
     */
    setMovementParams(speed: number, range: number): void {
        this.speed = speed;
        this.moveRange = range;
    }
}
```

### 3.3 BreakablePlatform (破碎平台)

```typescript
import { _decorator, tween, Vec3 } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 破碎平台
 * 玩家踩上后会在延迟时间后破碎
 */
@ccclass('BreakablePlatform')
export class BreakablePlatform extends BasePlatform {

    /** 破碎延迟时间 (秒) */
    @property
    public breakDelay: number = 0.5;

    /** 警告时间 (秒) - 开始抖动提示 */
    @property
    public warningTime: number = 0.3;

    /** 是否正在破碎中 */
    private isBreaking: boolean = false;

    /** 原始位置 (用于抖动) */
    private originalPosition: Vec3 = new Vec3();

    onLoad() {
        super.onLoad();
        this.type = PlatformType.BREAKABLE;
    }

    /**
     * 玩家落在平台上
     */
    onPlayerLand(player: Node): void {
        if (this.isBreaking) return;

        AudioManager.instance.playSfx('land');

        // 开始破碎计时
        this.startBreaking();
    }

    /**
     * 开始破碎流程
     */
    private startBreaking(): void {
        this.isBreaking = true;
        this.originalPosition = this.node.position.clone();

        // 警告阶段 - 抖动
        const warningDuration = this.breakDelay - this.warningTime;
        if (warningDuration > 0) {
            this.scheduleOnce(() => {
                this.startWarningShake();
            }, warningDuration);
        } else {
            this.startWarningShake();
        }

        // 破碎
        this.scheduleOnce(() => {
            this.doBreak();
        }, this.breakDelay);
    }

    /**
     * 警告抖动效果
     */
    private startWarningShake(): void {
        const shakeAmount = 3;
        const shakeDuration = 0.05;

        tween(this.node)
            .repeatForever(
                tween()
                    .to(shakeDuration, { position: new Vec3(
                        this.originalPosition.x + shakeAmount,
                        this.originalPosition.y,
                        0
                    )})
                    .to(shakeDuration, { position: new Vec3(
                        this.originalPosition.x - shakeAmount,
                        this.originalPosition.y,
                        0
                    )})
            )
            .start();
    }

    /**
     * 执行破碎
     */
    private doBreak(): void {
        // 停止抖动
        tween(this.node).stop();

        // 播放破碎音效
        AudioManager.instance.playSfx('break');

        // 播放破碎动画/特效
        this.playBreakEffect();

        // 禁用碰撞
        this._isActive = false;
        if (this.collider) {
            this.collider.enabled = false;
        }

        // 隐藏或播放消失动画
        tween(this.node)
            .to(0.2, { scale: new Vec3(0, 0, 1) })
            .call(() => {
                this.node.active = false;
            })
            .start();
    }

    /**
     * 播放破碎特效
     */
    private playBreakEffect(): void {
        // TODO: 生成碎片特效
        // EffectManager.instance.playEffect('platform_break', this.node.position);
    }

    /**
     * 重置状态
     */
    reset(): void {
        super.reset();

        // 停止所有动画和定时器
        tween(this.node).stop();
        this.unscheduleAllCallbacks();

        // 重置状态
        this.isBreaking = false;
        this.node.setScale(1, 1, 1);
    }

    /**
     * 设置破碎参数
     */
    setBreakParams(delay: number, warning: number): void {
        this.breakDelay = delay;
        this.warningTime = warning;
    }
}
```

---

## 4. PlatformSpawner 生成逻辑

### 4.1 生成器类

```typescript
import { _decorator, Component, Prefab, instantiate, Node } from 'cc';
const { ccclass, property } = _decorator;

/**
 * 平台生成器
 * 负责按难度曲线生成平台
 */
@ccclass('PlatformSpawner')
export class PlatformSpawner extends Component {

    /** 平台预制体 */
    @property(Prefab)
    public normalPlatformPrefab: Prefab = null;

    @property(Prefab)
    public movingPlatformPrefab: Prefab = null;

    @property(Prefab)
    public breakablePlatformPrefab: Prefab = null;

    /** 平台容器节点 */
    @property(Node)
    public platformContainer: Node = null;

    /** 对象池 */
    private pools: Map<PlatformType, ObjectPool<BasePlatform>> = new Map();

    /** 当前生成的最低 Y 坐标 */
    private lowestSpawnedY: number = 0;

    /** 当前层数 */
    private currentFloor: number = 0;

    /** 屏幕尺寸 */
    private screenWidth: number = 750;
    private screenHeight: number = 1334;

    onLoad() {
        this.initPools();
    }

    /**
     * 初始化对象池
     */
    private initPools(): void {
        // 每种平台类型初始化池
        this.pools.set(
            PlatformType.NORMAL,
            new ObjectPool(() => this.createPlatform(PlatformType.NORMAL), 20)
        );
        this.pools.set(
            PlatformType.MOVING,
            new ObjectPool(() => this.createPlatform(PlatformType.MOVING), 10)
        );
        this.pools.set(
            PlatformType.BREAKABLE,
            new ObjectPool(() => this.createPlatform(PlatformType.BREAKABLE), 10)
        );
    }

    /**
     * 创建平台节点
     */
    private createPlatform(type: PlatformType): BasePlatform {
        let prefab: Prefab;
        switch (type) {
            case PlatformType.MOVING:
                prefab = this.movingPlatformPrefab;
                break;
            case PlatformType.BREAKABLE:
                prefab = this.breakablePlatformPrefab;
                break;
            default:
                prefab = this.normalPlatformPrefab;
        }

        const node = instantiate(prefab);
        node.parent = this.platformContainer;
        node.active = false;

        return node.getComponent(BasePlatform);
    }

    /**
     * 生成初始平台
     */
    initializePlatforms(startY: number): void {
        this.lowestSpawnedY = startY;
        this.currentFloor = 0;

        // 生成一屏平台
        this.spawnPlatformsToY(startY - this.screenHeight - 500);
    }

    /**
     * 生成平台直到指定 Y 坐标
     */
    spawnPlatformsToY(targetY: number): void {
        while (this.lowestSpawnedY > targetY) {
            this.spawnNextPlatform();
        }
    }

    /**
     * 生成下一个平台
     */
    private spawnNextPlatform(): void {
        this.currentFloor++;

        // 获取当前难度配置
        const difficulty = this.getDifficultyConfig(this.currentFloor);

        // 计算平台位置
        const gap = this.randomRange(
            difficulty.platformGap.min,
            difficulty.platformGap.max
        );
        const y = this.lowestSpawnedY - gap;
        const width = this.randomRange(
            difficulty.platformWidth.min,
            difficulty.platformWidth.max
        );
        const x = this.randomRange(
            -this.screenWidth / 2 + width / 2 + 20,
            this.screenWidth / 2 - width / 2 - 20
        );

        // 选择平台类型
        const type = this.selectPlatformType(difficulty.platformWeights);

        // 从对象池获取平台
        const platform = this.pools.get(type).get();
        platform.reset();
        platform.floorIndex = this.currentFloor;
        platform.width = width;
        platform.node.setPosition(x, y);
        platform.node.active = true;

        // 设置类型特定参数
        if (type === PlatformType.MOVING) {
            (platform as MovingPlatform).setMovementParams(
                difficulty.movingPlatformSpeed,
                80
            );
        } else if (type === PlatformType.BREAKABLE) {
            (platform as BreakablePlatform).setBreakParams(
                difficulty.breakableDelay,
                0.3
            );
        }

        this.lowestSpawnedY = y;
    }

    /**
     * 按权重选择平台类型
     */
    private selectPlatformType(
        weights: Record<PlatformType, number>
    ): PlatformType {
        const total = Object.values(weights).reduce((a, b) => a + b, 0);
        let random = Math.random() * total;

        for (const [type, weight] of Object.entries(weights)) {
            random -= weight;
            if (random <= 0) {
                return type as PlatformType;
            }
        }

        return PlatformType.NORMAL;
    }

    /**
     * 获取难度配置
     */
    private getDifficultyConfig(floor: number): DifficultyConfig {
        for (const config of DIFFICULTY_CONFIGS) {
            const [min, max] = config.floorRange;
            if (floor >= min && floor <= max) {
                return config;
            }
        }
        return DIFFICULTY_CONFIGS[DIFFICULTY_CONFIGS.length - 1];
    }

    /**
     * 回收超出屏幕的平台
     */
    recyclePlatformsAboveY(thresholdY: number): void {
        this.platformContainer.children.forEach(node => {
            if (!node.active) return;

            if (node.position.y > thresholdY) {
                const platform = node.getComponent(BasePlatform);
                if (platform) {
                    platform.recycle();
                    this.pools.get(platform.type).put(platform);
                }
            }
        });
    }

    /**
     * 重置生成器
     */
    reset(): void {
        // 回收所有平台
        this.platformContainer.children.forEach(node => {
            const platform = node.getComponent(BasePlatform);
            if (platform) {
                platform.recycle();
                this.pools.get(platform.type).put(platform);
            }
        });

        this.lowestSpawnedY = 0;
        this.currentFloor = 0;
    }

    /**
     * 工具函数：随机范围
     */
    private randomRange(min: number, max: number): number {
        return min + Math.random() * (max - min);
    }
}
```

---

## 5. 对象池管理

```typescript
/**
 * 通用对象池
 */
export class ObjectPool<T> {
    /** 对象池 */
    private pool: T[] = [];

    /** 创建函数 */
    private createFn: () => T;

    /** 最大容量 */
    private maxSize: number;

    constructor(createFn: () => T, initialSize: number = 10, maxSize: number = 50) {
        this.createFn = createFn;
        this.maxSize = maxSize;

        // 预创建对象
        for (let i = 0; i < initialSize; i++) {
            this.pool.push(this.createFn());
        }
    }

    /**
     * 获取对象
     */
    get(): T {
        if (this.pool.length > 0) {
            return this.pool.pop()!;
        }
        return this.createFn();
    }

    /**
     * 归还对象
     */
    put(obj: T): void {
        if (this.pool.length < this.maxSize) {
            this.pool.push(obj);
        }
    }

    /**
     * 获取当前池大小
     */
    get size(): number {
        return this.pool.length;
    }

    /**
     * 清空池
     */
    clear(): void {
        this.pool = [];
    }
}
```

---

## 6. 平台视觉规范

### 6.1 尺寸规范

| 平台类型 | 默认宽度 | 宽度范围 | 高度 |
|----------|----------|----------|------|
| 普通平台 | 120px | 70-150px | 20px |
| 移动平台 | 100px | 70-130px | 20px |
| 破碎平台 | 100px | 70-130px | 20px |

### 6.2 视觉样式

```
普通平台 (Normal):
┌─────────────────────────────────────────┐
│  颜色: #4A90D9 (蓝色)                   │
│  边框: 2px #2D5A87                      │
│  圆角: 4px                              │
│  阴影: 无                                │
└─────────────────────────────────────────┘

移动平台 (Moving):
┌─────────────────────────────────────────┐
│  颜色: #7ED321 (绿色)                   │
│  边框: 2px #5BA011                      │
│  圆角: 4px                              │
│  动画: 左右箭头指示                      │
└─────────────────────────────────────────┘

破碎平台 (Breakable):
┌─────────────────────────────────────────┐
│  颜色: #F5A623 (橙色)                   │
│  边框: 2px #C48000 (虚线)               │
│  圆角: 4px                              │
│  状态: 裂纹纹理                          │
│  破碎前: 抖动 + 变红                    │
└─────────────────────────────────────────┘
```

### 6.3 状态表现

| 状态 | 普通 | 移动 | 破碎 |
|------|------|------|------|
| 默认 | 静止 | 左右移动 | 静止+裂纹 |
| 玩家站立 | 无变化 | 带动玩家 | 开始计时 |
| 警告 | - | - | 抖动+变色 |
| 触发 | - | - | 破碎+碎片 |
