# 开发指南

> 无限向下 - MVP 开发环境与工作流程

## 1. 开发环境配置

### 1.1 必需工具

| 工具 | 版本 | 说明 |
|------|------|------|
| Cocos Creator | 3.8.5 | 游戏引擎 |
| Node.js | 18.x LTS | 运行时环境 |
| 微信开发者工具 | 最新稳定版 | 小游戏调试 |
| VS Code | 最新版 | 代码编辑器 |
| Git | 最新版 | 版本控制 |

### 1.2 VS Code 推荐插件

```json
{
    "recommendations": [
        "AntHall.cocos-creator",            // Cocos Creator 支持
        "AntHall.cocos-creator-helper",     // Cocos 辅助工具
        "dbaeumer.vscode-eslint",           // ESLint
        "esbenp.prettier-vscode",           // Prettier
        "ms-vscode.vscode-typescript-next"  // TypeScript
    ]
}
```

### 1.3 项目初始化

```bash
# 1. 克隆项目
git clone <repository-url>
cd infinitely-down

# 2. 用 Cocos Creator 打开项目
# 双击 project.json 或从 Cocos Dashboard 打开

# 3. 配置微信开发者工具路径
# Cocos Creator → 偏好设置 → 外部程序 → 微信开发者工具
```

---

## 2. 项目结构

### 2.1 目录说明

```
infinitely-down/
├── assets/                 # 游戏资源和脚本 (核心开发目录)
│   ├── scenes/            # 场景文件
│   ├── scripts/           # TypeScript 脚本
│   ├── prefabs/           # 预制体
│   ├── textures/          # 图片资源
│   ├── audio/             # 音频资源
│   └── animations/        # 动画资源
│
├── library/               # Cocos 生成的资源库 (勿手动修改)
├── local/                 # 本地配置 (不提交)
├── temp/                  # 临时文件 (不提交)
│
├── settings/              # 项目设置
│   └── v2/
│       ├── packages/      # 各平台构建设置
│       └── project.json   # 项目设置
│
├── build/                 # 构建输出目录 (不提交)
│
├── docs/                  # 技术文档
│
├── project.json           # Cocos 项目配置
├── tsconfig.json          # TypeScript 配置
└── package.json           # 项目元数据
```

### 2.2 .gitignore 配置

```gitignore
# Cocos Creator
/library/
/local/
/temp/
/build/
/profiles/

# Node
node_modules/

# IDE
.idea/
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

---

## 3. 开发工作流

### 3.1 功能开发流程

```
┌─────────────────┐
│ 1. 创建分支     │  git checkout -b feature/xxx
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 2. 编写代码     │  在 assets/scripts/ 中编写
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 3. 场景编辑     │  在 Cocos Creator 中编辑场景和预制体
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 4. 预览测试     │  Cocos Creator 内置预览 (浏览器/模拟器)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 5. 微信真机调试 │  构建到微信开发者工具测试
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 6. 代码审查     │  提交 PR，代码审查
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 7. 合并主分支   │  审查通过后合并
└─────────────────┘
```

### 3.2 分支命名规范

| 分支类型 | 命名格式 | 示例 |
|----------|----------|------|
| 功能开发 | `feature/功能名` | `feature/platform-system` |
| 问题修复 | `fix/问题描述` | `fix/jump-collision` |
| 优化改进 | `improve/优化内容` | `improve/object-pool` |
| 发布版本 | `release/版本号` | `release/v1.0.0` |

### 3.3 提交信息规范

```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型 (type)**:
- `feat`: 新功能
- `fix`: 修复 bug
- `refactor`: 重构
- `style`: 格式调整
- `docs`: 文档
- `test`: 测试
- `chore`: 构建/工具

**示例**:
```
feat(platform): 添加破碎平台组件

- 实现 BreakablePlatform 类
- 添加破碎动画和音效
- 支持配置破碎延迟时间

Closes #12
```

---

## 4. 构建与调试

### 4.1 本地预览

```bash
# 方法 1: Cocos Creator 内预览
# 点击工具栏的"预览"按钮 (浏览器/模拟器)

# 方法 2: 构建后在微信开发者工具预览
# 1. Cocos Creator → 项目 → 构建发布
# 2. 平台选择"微信小游戏"
# 3. 点击"构建"
# 4. 构建完成后点击"运行"或在微信开发者工具打开
```

### 4.2 构建配置

**微信小游戏构建设置** (`settings/v2/packages/wechatgame.json`):

```json
{
    "appid": "your-appid",
    "orientation": "portrait",
    "separateEngine": true,
    "subpackages": [],
    "wasmSubpackage": false
}
```

### 4.3 调试技巧

**1. Cocos 调试面板**
- 打开方式: Cocos Creator → 开发者 → 调试面板
- 功能: 查看节点树、组件属性、性能数据

**2. 微信开发者工具调试**
```javascript
// 在代码中添加断点
debugger;

// 或使用 console
console.log('当前层数:', this.currentFloor);
console.warn('警告信息');
console.error('错误信息');
```

**3. 远程真机调试**
- 微信开发者工具 → 真机调试
- 扫码后可在电脑端调试手机上的游戏

---

## 5. 资源管理

### 5.1 资源命名规范

```
textures/
├── ui/
│   ├── btn_start_normal.png      # 按钮_开始_正常状态
│   ├── btn_start_pressed.png     # 按钮_开始_按下状态
│   └── icon_coin.png             # 图标_金币
├── player/
│   ├── player_idle_0.png         # 玩家_待机_帧0
│   ├── player_idle_1.png         # 玩家_待机_帧1
│   └── player_jump.png           # 玩家_跳跃
└── platforms/
    ├── platform_normal.png       # 平台_普通
    ├── platform_moving.png       # 平台_移动
    └── platform_breakable.png    # 平台_破碎
```

### 5.2 资源优化

**图片优化**:
- 格式: PNG (透明) / JPG (背景)
- 尺寸: 2 的幂次方 (便于压缩)
- 图集: 同类图片合并成图集

**音频优化**:
- 格式: MP3 (背景音乐) / WAV (短音效)
- 采样率: 22050Hz
- 位深: 16bit

### 5.3 首包体积控制

**4MB 限制分配建议**:

| 类别 | 大小限制 | 内容 |
|------|----------|------|
| 代码 | ≤1MB | 压缩后的 JS |
| 核心图片 | ≤1.5MB | UI、玩家、MVP平台 |
| 音频 | ≤500KB | 必要音效 |
| 场景/预制体 | ≤500KB | 三个场景+必要预制体 |
| 预留 | ≥500KB | 热更新/扩展 |

**分包策略** (后期):
```json
{
    "subpackages": [
        {
            "name": "extra-platforms",
            "root": "assets/platforms-extra/"
        },
        {
            "name": "shop",
            "root": "assets/shop/"
        }
    ]
}
```

---

## 6. 常见问题

### 6.1 TypeScript 编译错误

**问题**: 找不到模块或类型定义

**解决**:
```bash
# 刷新 Cocos Creator 的 TypeScript 声明
# Cocos Creator → 开发者 → 刷新脚本
```

### 6.2 场景丢失引用

**问题**: 预制体或节点引用丢失

**解决**:
1. 检查 meta 文件是否被正确提交
2. 在 Cocos Creator 中重新关联引用
3. 避免直接重命名/移动 assets 中的文件，使用 Cocos Creator 的资源管理器

### 6.3 微信小游戏构建失败

**问题**: 构建报错或包体过大

**解决**:
1. 检查 AppID 配置是否正确
2. 清理 `build/` 目录后重新构建
3. 检查是否有未使用的大资源
4. 开启资源压缩选项

### 6.4 真机显示异常

**问题**: 模拟器正常但真机显示不对

**解决**:
1. 检查适配方案 (Canvas 设置)
2. 测试不同分辨率的设备
3. 检查是否有平台相关的代码分支

---

## 7. 性能监控

### 7.1 内置性能面板

```typescript
// 开启性能统计 (开发环境)
if (CC_DEBUG) {
    profiler.showStats();
}
```

### 7.2 关键指标监控

```typescript
// 自定义性能监控
class PerformanceMonitor {
    static logFrame() {
        const stats = {
            fps: game.frameRate,
            drawCalls: director.root.batcher2D.drawCallCount,
            nodeCount: director.getScene().children.length
        };
        console.log('Performance:', stats);
    }
}
```

### 7.3 性能优化检查清单

- [ ] DrawCall 是否 ≤ 50
- [ ] 帧率是否稳定在 55-60 FPS
- [ ] 内存占用是否 ≤ 150MB
- [ ] 对象池是否正常工作
- [ ] 是否有内存泄漏
- [ ] 是否有不必要的 update() 调用
