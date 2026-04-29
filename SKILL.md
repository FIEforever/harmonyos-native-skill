---
name: harmonyos-native
description: HarmonyOS 鸿蒙原生应用开发（纯 ArkTS + ArkUI，不含 uni-app/跨平台方案）。 基于 DevEco Studio 本地离线文档（API 12 / HarmonyOS NEXT，含 API 21/22 最新接口）。 包含：ArkTS 严格类型系统、ArkUI 组件体系（Column/Row/Grid/List/Navigation/NavPathStack 等）、 状态管理 V1（@State/@Prop/@Link/@Provide/@Consume/@Observed/@ObjectLink）、 状态管理 V2（@ComponentV2/@Local/@Param/@Event/@Monitor/@Computed/@ObservedV2/@Trace）、 UIAbility Stage 模型完整生命周期、AbilityStage、Want、UIAbilityContext API、 页面路由（Navigation + NavPathStack / @ohos/router）、NavDestination、路由拦截、 HTTP 网络请求（@kit.NetworkKit）完整参数、WebSocket、 数据持久化（Preferences/RelationalStore/KV数据库/文件）、 权限申请（user_grant 两步流程）、LazyForEach/IDataSource/Repeat 列表优化、 手势系统（Tap/LongPress/Pan/Pinch/Rotation/Swipe）、显式动画（animateTo）、关键帧动画、 Canvas 2D、XComponent、ArkGraphics 3D、AR Engine Kit、 CameraKit、AVPlayer、AVRecorder、Audio Kit、 Core Vision Kit（OCR/SubjectSegmentation）、Core Speech Kit（TTS/ASR）、 MindSpore Lite 端侧推理、NDK/NAPI C++ 原生模块、TaskPool/Worker 多线程、 DevEco Profiler、HarmonyOS Design 设计规范（API 21 最新版）、 响应式布局（GridRow/GridCol/BreakpointSystem）、暗色模式适配。 
triggers:
  - 鸿蒙
  - HarmonyOS
  - ArkTS
  - ArkUI
  - DevEco
  - UIAbility
  - NavPathStack
  - 状态管理
  - @State
  - @Prop
  - @Link
  - @Provide
  - @Consume
  - @ComponentV2
  - HTTP请求
  - 网络请求
  - http
  - rcp
  - 权限申请
  - 权限
  - 页面路由
  - 路由
  - Navigation
  - LazyForEach
  - 列表
  - 动画
  - animateTo
  - Canvas
  - XComponent
  - 相机
  - 拍照
  - 视频
  - 音频
  - TTS
  - ASR
  - OCR
  - MindSpore
  - NDK
  - NAPI
  - C++
  - 多线程
  - TaskPool
  - Worker
  - DevEco
  - Profiler
  - HAP
  - HAR
  - HSP
  - 打包
  - Preferences
  - RelationalStore
  - KV
  - 数据库
  - 持久化
  - 响应式
  - 暗色模式
  - darkmode
author: FIEforever
version: 2.2.0
tags:
  - harmonyos
  - harmonyos-next
  - arkts
  - arkui
  - huawei
  - 移动开发
  - 鸿蒙开发
agent_created: true
---

# HarmonyOS 鸿蒙原生开发 Skill (v2.2.0)

## 快速导航

| 需求场景 | 参考文档 |
|---------|---------|
| ArkTS 类型/语法/异步/类 | `references/01-arkts-core.md` |
| ArkUI 组件/布局/手势/动画 | `references/02-arkui-components.md` |
| 状态管理 V1/V2/组件通信 | `references/03-state-management.md` |
| 生命周期/UIAbility/AbilityStage | `references/04-lifecycle.md` |
| HTTP/rcp/WebSocket 网络请求 | `references/05-network.md` |
| 文件/Preferences/RelationalStore | `references/06-data-persistence.md` |
| 权限申请/用户授权流程 | `references/07-permissions.md` |
| Canvas 2D/XComponent/ArkGraphics | `references/08-graphics.md` |
| DevEco Profiler/Testing/打包发布 | `references/09-deveco-tools.md` |
| HarmonyOS Design 设计规范 | `references/10-design.md` |
| Context/Want/跨应用跳转 | `references/11-context-ability.md` |

## 核心能力矩阵

```
状态管理 ────── V1 (@State/@Prop/@Link/@Provide/@Consume/@Observed/@ObjectLink)
           └─ V2 (@ComponentV2/@Local/@Param/@Event/@Monitor/@Computed/@Trace)
           └─ 通信: EventHub/AppStorage/LocalStorage/AppContext
           └─ 持久化: AppStorage/PersistentStorage → Preferences/RelationalStore
           
页面路由 ────── @ohos/router (FA模型)
           └─ Navigation + NavPathStack (Stage模型)
           └─ 路由拦截/传参/返回监听
           
网络请求 ────── @kit.NetworkKit http (Promise/Callback/Callback+Promise)
           └─ rcp (推荐RESTful)
           └─ WebSocket
           └─ 证书配置/代理/超时/Header
           
数据持久化 ──── AppStorage/LocalStorage (页面级)
           └─ Preferences (键值对)
           └─ RelationalStore (SQLite)
           └─ 分布式数据服务 (跨设备)
           
媒体能力 ────── CameraKit (拍照/录像/预览)
           └─ AVPlayer/AVRecorder (音视频播放录制)
           └─ Core Vision Kit (OCR/主体分割)
           └─ Core Speech Kit (TTS/ASR)
           
图形渲染 ────── Canvas 2D (ArkGraphics)
           └─ XComponent (Surface渲染)
           └─ AR Engine Kit (ARCore)
           
Native ─────── NDK + NAPI (C++模块)
           └─ TaskPool/Worker (多线程)
```

## 常见错误速查表

| 错误现象 | 根本原因 | 解决方案 |
|---------|---------|---------|
| UI不更新 | `@State`对象直接修改属性 | `this.obj = {...this.obj, newProp: value}` |
| 白屏/数据为空 | `aboutToAppear`中获取路由参数 | 改用`NavDestination.onReady()` |
| 内存泄漏 | HTTP请求未`destroy()` | `finally`中调用`httpRequest.destroy()` |
| 列表白屏 | `LazyForEach`未实现`IDataSource` | 必须实现`totalCount()`+`getData()`+`addDataHost()`+`reloadListData()` |
| 状态不同步 | `@Link`在`@Entry`中使用 | `@Entry`无父组件，禁止`@Link`；改用`@ObjectLink` |
| 跳转失败 | `want`参数错误/权限不足 | 检查`parameter`格式；申请目标ability权限 |
| 编译报错 | ArkTS严格类型 | 显式注解；禁止`any`；检查可选链`?.` |
| 真机调试失败 | 签名证书问题 | 检查`build-profile.json5`签名配置；重新生成签名 |

## 权限申请两段式

```typescript
// 1. module.json5 声明
"requestPermissions": [
  {"name": "ohos.permission.INTERNET"},
  {"name": "ohos.permission.CAMERA"}
]

// 2. 代码动态申请
import abilityAccessCtrl from '@ohos.abilityAccessCtrl'
import bundleManager from '@ohos.bundle.bundleManager'

let atManager = abilityAccessCtrl.createAtManager()
let tokenId = bundleManager.getBundleInfoForSelfSync(bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION).appInfo.accessTokenId
let requestResult = atManager.requestGrant(tokenId, 'ohos.permission.CAMERA')
if (requestResult === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
  // 权限已授予
}
```

## HTTP请求标准模板

```typescript
import http from '@ohos.net.http'

let httpRequest = http.createHttp()
httpRequest.request(
  'https://api.example.com/data',
  {
    method: http.RequestMethod.GET,
    header: { 'Content-Type': 'application/json' },
    extraData: {},
    connectTimeout: 60000,
    readTimeout: 60000
  }
).then((resp: http.HttpResponse) => {
  let code = resp.responseCode
  let data = resp.result as string
}).catch((err: Error) => {
  console.error(err.message)
}).finally(() => {
  httpRequest.destroy() // 必须释放
})
```

## Stage模型UIAbility启动

```typescript
import common from '@ohos.app.ability.common'
import Want from '@ohos.app.ability.Want'
import hilog from '@ohos.hilog'

const TAG = 'EntryAbility'
const DOMAIN = 0x0001

// 页面跳转
let context = getContext(this) as common.UIAbilityContext
let want: Want = {
  deviceId: '',
  bundleName: 'com.example.app',
  abilityName: 'FuncAbility', // 目标UIAbility名
  parameters: { key: 'value' }
}
context.startAbility(want).then(() => {
  hilog.info(DOMAIN, TAG, '启动成功')
}).catch((err) => {
  hilog.error(DOMAIN, TAG, '启动失败: %{public}s', err.message)
})
```

## 状态管理 V2 最小模板

```typescript
@Entry
@ComponentV2
struct MyPage {
  @Local count: number = 0
  @Param message: string = ''
  @Event onCountChange: (val: number) => void = () => {}
  @Monitor('count')
  onCountMonitor() {
    this.onCountChange(this.count)
  }
  build() {
    Column() {
      Text(`${this.count}`)
      Button('增加').onClick(() => this.count++)
    }
  }
}
```

## 开发环境

- **IDE**: DevEco Studio (推荐最新版)
- **模拟器**: HarmonyOS Emulator (API 12+)
- **真机**: 需要华为开发者账号 + 应用签名
- **文档**: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/

---

> 上次更新: 2026-03-21 | 版本: v2.2.0 | 状态: 活跃维护
