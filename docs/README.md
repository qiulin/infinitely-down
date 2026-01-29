# 无限向下 - 技术文档中心

> MVP 开发技术文档索引

## 文档导航

### 设计文档 (design/)

| 文档 | 说明 | 状态 |
|------|------|------|
| [01-technical-architecture.md](./design/01-technical-architecture.md) | 系统架构、模块划分、目录结构 | MVP |
| [02-game-core-design.md](./design/02-game-core-design.md) | 核心玩法、游戏循环、控制参数 | MVP |
| [03-data-structures.md](./design/03-data-structures.md) | TypeScript 接口定义、枚举、常量 | MVP |
| [04-platform-design.md](./design/04-platform-design.md) | 平台系统、对象池、生成逻辑 | MVP |

### API 规范 (api/)

| 文档 | 说明 | 状态 |
|------|------|------|
| [01-wechat-api-integration.md](./api/01-wechat-api-integration.md) | 微信登录、分享、排行榜 API | MVP |
| [02-cloud-functions.md](./api/02-cloud-functions.md) | 云函数接口、数据校验 | MVP |

### UI/UX 规范 (ui/)

| 文档 | 说明 | 状态 |
|------|------|------|
| [01-ui-specification.md](./ui/01-ui-specification.md) | UI 组件规范、颜色、字体 | MVP |
| [02-scene-flow.md](./ui/02-scene-flow.md) | 场景流转、弹窗逻辑 | MVP |

### 实施文档 (implementation/)

| 文档 | 说明 | 状态 |
|------|------|------|
| [01-development-guide.md](./implementation/01-development-guide.md) | 开发环境、工作流程 | MVP |
| [02-milestone-checklist.md](./implementation/02-milestone-checklist.md) | 12 周里程碑检查清单 | MVP |
| [03-code-conventions.md](./implementation/03-code-conventions.md) | 代码规范、命名约定 | MVP |

---

## 快速链接

- [产品设计文档](../README.md) - 完整产品方案
- [开发路线图](../ROADMAP.md) - 分期开发计划
- [Claude 开发指南](../CLAUDE.md) - AI 辅助开发说明

---

## MVP 范围概览

**目标**: 12 周内完成可上线版本，验证核心玩法 + 基础广告变现

**核心功能**:
- 无尽模式下落玩法
- 3 种平台类型（普通、移动、破碎）
- 微信登录 + 好友排行榜
- 激励视频广告（复活、双倍金币）

**技术约束**:
- 引擎: Cocos Creator 3.8.5
- 首包体积: ≤ 4MB
- 目标帧率: 60 FPS
- 平台: 微信小游戏

---

## 文档维护说明

1. 所有文档使用 Markdown 格式
2. 代码示例使用 TypeScript
3. 接口定义需与实际代码同步更新
4. 版本号遵循语义化版本规范
