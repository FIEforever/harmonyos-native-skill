# HarmonyOS Native Skill

> 鸿蒙原生应用开发专家 Skill —— 纯 ArkTS + ArkUI，覆盖 API 12/21/22 最新接口

[English](README.md) | 中文

---

## 预览图

### DevEco Studio 开发界面

```
┌─────────────────────────────────────────────────────────────────┐
│  🔍 Search...                              🌐 API 12 │ 中文 ▼    │
├──────────────┬────────────────────────────┬─────────────────────┤
│  Project     │  MainPage.ets              │  Attributes         │
│  ▼ Entry     │                            │  ┌─────────────┐   │
│    ▼ src     │  @Entry                    │  │ Text        │   │
│      ▼ main  │  @Component                │  │   fontSize  │   │
│        ├ Main │  struct MainPage {        │  │   fontWeight│   │
│        │ Page │    @State msg: string     │  │   color     │   │
│        │    . │                            │  └─────────────┘   │
│        │  ets│    build() {               │  ▼ Styles           │
│        └ util│      Column() {            │    width: 100%      │
│              │        Text(this.msg)      │    height: 100%    │
│  ▼ oh_modu   │          .fontSize(20)      │    justifyContent   │
│  > entry     │      }                      │      (FlexAlign)    │
│  > library   │    }                        │                     │
├──────────────┴────────────────────────────┴─────────────────────┤
│  ▶ Preview              [▶ DevEco Device Manager]               │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                                                           │  │
│  │           Hello World                                     │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 状态管理数据流

```
┌─────────────────────────────────────────────────────────────────┐
│                    状态管理架构图                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌──────────────┐                                             │
│   │   @Entry     │  ← 页面入口                                  │
│   │  @Component  │                                             │
│   └──────┬───────┘                                             │
│          │ @State                                               │
│          ▼                                                      │
│   ┌──────────────┐                                             │
│   │   @Component │  子组件接收                                  │
│   │  @Component  │──── @Prop                                   │
│   └──────────────┘                                             │
│                                                                 │
│   V2 架构:                                                      │
│   ┌──────────────┐     ┌──────────────┐                        │
│   │  @ComponentV2│────▶│  @Local      │  响应式状态             │
│   │              │     │  @Param      │  可观测对象             │
│   │  @Trace      │     │  @Monitor   │  主动追踪               │
│   └──────────────┘     └──────────────┘                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### UIAbility 生命周期

```
┌─────────────────────────────────────────────────────────────────┐
│                  UIAbility 生命周期                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐   │
│  │ onCreate│──▶│onWindowSt-│──▶│ onForegr- │◀──│onBackgr- │   │
│  │         │   │ ageCreate │   │ ound      │   │ ound     │   │
│  └─────────┘   └──────────┘   └───────────┘   └──────────┘   │
│       │              │              │              │         │
│       │              │              │              ▼         │
│       │              │              │         ┌──────────┐    │
│       │              │              │         │onTermina-│    │
│       │              │              │         │te         │    │
│       │              │              │         └──────────┘    │
│       │              ▼              │                          │
│       │         ┌──────────┐        │                          │
│       │         │onWindowSt-│◀───────┘                          │
│       │         │ageDestroy │  应用切后台后重新激活              │
│       │         └──────────┘                                    │
│       ▼                                                         │
│  ┌──────────┐                                                   │
│  │ onDestroy│  应用销毁                                          │
│  └──────────┘                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Navigation + NavPathStack 路由

```
┌─────────────────────────────────────────────────────────────────┐
│                  页面导航架构                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Navigation()                         │   │
│   │  ┌───────────────────────────────────────────────────┐  │   │
│   │  │              NavPathStack                         │  │   │
│   │  │  ┌─────┬─────┬─────┬─────┐                        │  │   │
│   │  │  │Home │List │Detail│Settings│                    │  │   │
│   │  │  └──┬──┴──┬─┴──┬──┴──┬──┘                        │  │   │
│   │  │     │     │    │     │                            │  │   │
│   │  │     ▼     ▼    ▼     ▼                            │  │   │
│   │  │  push() push() push() push()                       │  │   │
│   │  │     │     │    │     │                            │  │   │
│   │  │     ▼     ▼    ▼     ▼                            │  │   │
│   │  │  [页面栈模拟: Home → List → Detail → Settings]     │  │   │
│   │  └───────────────────────────────────────────────────┘  │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 简介

本 Skill 是专为鸿蒙原生应用开发打造的 AI 助手指南，基于 DevEco Studio 本地离线文档，收录 **21,000+ API** 条目。

### 核心能力

- **ArkTS 类型系统**：严格类型、泛型、异步、Sendable 类
- **ArkUI 组件体系**：布局、表单、导航、手势、动画
- **状态管理 V1/V2**：@State/@Prop/@Link → @ComponentV2/@Local/@Param
- **UIAbility Stage 模型**：完整生命周期、Want、跨应用跳转
- **网络请求**：@kit.NetworkKit HTTP / rcp / WebSocket
- **数据持久化**：Preferences / RelationalStore / 文件
- **权限申请**：user_grant 两段式流程
- **图形渲染**：Canvas 2D / XComponent / ArkGraphics
- **DevEco 工具链**：Profiler / Testing / 打包 / 签名
- **HarmonyOS Design**：色彩 / 字体 / 布局 / 动效规范

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v2.2.1 | 2026-04-29 | 新增截图章节、ASCII 架构图、流程图 |
| v2.2.0 | 2026-04-29 | 新增 3 个 references、错误速查表、20+ 触发词 |
| v2.1.0 | 2026-03-21 | 初始版本，8 个 references |

---

## 截图预览

> 以下截图帮助您快速了解 Skill 覆盖的内容范围。点击查看大图。

| 截图 | 说明 |
|------|------|
| ![DevEco Studio](screenshots/deveco-studio-overview.png) | DevEco Studio 开发界面 |
| ![ArkUI Preview](screenshots/arkui-preview.png) | ArkUI 组件实时预览 |
| ![State Flow](screenshots/state-management-flow.png) | 状态管理数据流图 |
| ![Lifecycle](screenshots/ability-lifecycle.png) | UIAbility 生命周期 |
| ![Navigation](screenshots/navigation-flow.png) | Navigation 路由架构 |
| ![HTTP Flow](screenshots/http-request-flow.png) | 网络请求流程 |
| ![Permission](screenshots/permission-flow.png) | 权限申请流程 |

### 懒加载列表示意

```
┌─────────────────────────────────────────────────────────────────┐
│                    LazyForEach 懒加载示意                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Virtualized List (可视区域渲染)                         │    │
│  │  ┌─────────────────────────────────────────────────┐   │    │
│  │  │  [Item 1]  ← 已渲染                              │   │    │
│  │  │  [Item 2]  ← 已渲染                              │   │    │
│  │  │  [Item 3]  ← 已渲染                              │   │    │
│  │  │  [Item 4]  ← 当前可视区                          │   │    │
│  │  │  [Item 5]  ← 当前可视区                          │   │    │
│  │  │  [Item 6]  ← 当前可视区                          │   │    │
│  │  │  [Item 7]  ← 当前可视区                          │   │    │
│  │  │  [Item 8]  ← 已回收待复用                        │   │    │
│  │  │  [Item 9]  ← 已回收待复用                        │   │    │
│  │  └─────────────────────────────────────────────────┘   │    │
│  │                                                         │    │
│  │  totalCount: 100000  ← 数据总量                         │    │
│  │  实际渲染: ~10-20个  ← 内存占用极低                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 网络请求流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP 请求完整流程                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  创建    │───▶│  配置    │───▶│  执行    │───▶│  响应    │   │
│  │  Request │    │  Header  │    │  request │    │  onResp  │   │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘   │
│       │                                            │             │
│       │                                            ▼             │
│       │                                      ┌──────────┐        │
│       │                                      │  解析    │        │
│       │                                      │  JSON    │        │
│       │                                      └──────────┘        │
│       │                                            │             │
│       ▼                                            ▼             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  finally { httpRequest.destroy() }  ← 防止内存泄漏       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 权限申请流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    user_grant 权限申请                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: 声明权限                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  module.json5                                           │   │
│  │  "requestPermissions": [                               │   │
│  │    "ohos.permission.CAMERA"                            │   │
│  │  ]                                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  Step 2: 检查权限                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  atManager.checkAccessControl()                         │   │
│  │    ├─ granted → 直接使用                                │   │
│  │    └─ denied  → 请求授权                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  Step 3: 请求授权                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  atManager.requestPermissionsOnUser()                   │   │
│  │    ┌───────────────────────────────────────────────┐     │   │
│  │    │  系统弹窗: "应用需要访问相机，是否允许？"      │     │   │
│  │    │  [允许]  [拒绝]  [总是允许]                    │     │   │
│  │    └───────────────────────────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  Step 4: 处理结果                                               │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  onRequestResult(grantResults)                         │   │
│  │    ├─ [0] → 授权成功，使用功能                         │   │
│  │    └─ [-1] → 授权失败，提示用户                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 文件结构

```
harmonyos-native-skill/
├── SKILL.md                        # 主文件（快速导航 + 核心模板）
├── screenshots/                    # 截图资源
│   ├── README.md                  # 截图说明
│   ├── deveco-studio-overview.png  # DevEco Studio 界面
│   ├── arkui-preview.png           # ArkUI 预览
│   ├── state-management-flow.png   # 状态流图
│   ├── ability-lifecycle.png       # 生命周期图
│   ├── navigation-flow.png         # 导航架构图
│   ├── http-request-flow.png       # HTTP 流程图
│   ├── permission-flow.png         # 权限流程图
│   └── lazy-foreach-demo.png       # 懒加载示例
└── references/
    ├── 01-arkts-core.md           # ArkTS 核心语法与类型系统
    ├── 02-arkui-components.md     # ArkUI 组件与布局体系
    ├── 03-state-management.md     # 状态管理 V1 / V2
    ├── 04-lifecycle.md            # UIAbility 生命周期与 Stage 模型
    ├── 05-network.md              # HTTP / rcp / WebSocket 网络请求
    ├── 06-data-persistence.md      # 数据持久化
    ├── 07-permissions.md          # 权限申请与用户授权
    ├── 08-graphics.md             # Canvas 2D / XComponent / ArkGraphics
    ├── 09-deveco-tools.md         # DevEco Studio 工具与调试
    ├── 10-design.md               # HarmonyOS Design 设计规范
    └── 11-context-ability.md      # Context / Want / 跨应用跳转
```

---

## 快速导航

### 按场景查找

| 场景需求 | 推荐文档 |
|---------|---------|
| 类型报错、语法问题 | `01-arkts-core.md` |
| UI 不更新、白屏 | `02-arkui-components.md` + `SKILL.md` 错误速查表 |
| 页面跳转失败 | `04-lifecycle.md` + `11-context-ability.md` |
| HTTP 请求报错 | `05-network.md` |
| 数据无法持久化 | `06-data-persistence.md` |
| 权限被拒绝 | `07-permissions.md` |
| 列表性能问题 | `02-arkui-components.md` (LazyForEach) |
| 签名打包失败 | `09-deveco-tools.md` |

### 常见错误速查

| 错误现象 | 根本原因 | 解决方案 |
|---------|---------|---------|
| UI 不更新 | `@State` 对象直接修改属性 | `this.obj = {...this.obj, prop: value}` |
| 白屏/数据为空 | `aboutToAppear` 中获取路由参数 | 改用 `NavDestination.onReady()` |
| 内存泄漏 | HTTP 请求未 `destroy()` | `finally` 中调用 `httpRequest.destroy()` |
| 列表白屏 | `LazyForEach` 未实现 `IDataSource` | 实现 `totalCount()` + `getData()` |
| 状态不同步 | `@Link` 在 `@Entry` 中使用 | 改用 `@ObjectLink` |

---

## 使用方法

### 在 WorkBuddy 中使用

1. 安装本 Skill 到 `~/.workbuddy/skills/`
2. 描述你的需求，Skill 将自动加载相关文档

```
示例提示词：
- "帮我写一个登录页面，使用 Navigation + NavPathStack"
- "HTTP 请求报错 500 怎么解决"
- "@State 和 @Local 有什么区别"
```

### 触发词

本 Skill 会自动响应以下关键词：

```
鸿蒙、HarmonyOS、ArkTS、ArkUI、DevEco、UIAbility、NavPathStack
状态管理、@State、@Prop、@Link、@ComponentV2
HTTP请求、网络请求、权限申请、页面路由、LazyForEach
Canvas、XComponent、相机、拍照、视频、音频
TTS、ASR、OCR、MindSpore、NDK、NAPI
HAP、HAR、HSP、打包、DevEco Profiler
Preferences、RelationalStore、KV数据库
```

---

## 开发环境

| 工具 | 版本 | 说明 |
|------|------|------|
| DevEco Studio | 4.x+ | 推荐最新版本 |
| HarmonyOS SDK | API 12/21/22 | 支持 NEXT |
| Node.js | 18+ | 仅部分工具需要 |
| JDK | 17+ | DevEco 内置 |

### 真机调试

1. 华为开发者联盟注册账号
2. 创建应用、申请签名证书
3. 配置 `build-profile.json5`
4. 设备开启开发者模式 + USB 调试

---

## 贡献指南

欢迎提交 Issue 和 Pull Request！

### 报告问题

- API 错误或过时 → 提交 Issue 并注明 API 名称
- 文档缺失 → 提交 Issue 描述需求场景
- 代码示例报错 → 附上完整错误信息和环境

### 提交代码

1. Fork 本仓库
2. 创建分支 `git checkout -b feature/your-feature`
3. 提交更改 `git commit -m 'feat: add xxx'`
4. 推送 `git push origin feature/your-feature`
5. 创建 Pull Request

---

## 参考资源

- [HarmonyOS 开发者官网](https://developer.huawei.com/consumer/cn/)
- [API 参考文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/)
- [ArkTS 语言规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-overview/)
- [ArkUI 组件参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/)
- [WorkBuddy Skill 文档](https://www.workbuddy.cn/)

---

## 许可证

[MIT License](LICENSE)

---

## 维护者

- **FIEforever** - [GitHub](https://github.com/FIEforever)

---

> ⚠️ 本 Skill 内容基于 DevEco Studio 内置离线文档整理，如有出入请以官方文档为准。
