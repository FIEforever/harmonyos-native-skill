# UIAbility 生命周期与 Stage 模型

## UIAbility 完整生命周期

```
应用启动
    │
    ▼
onCreate() ──── 应用冷启动时调用一次
    │
    ▼
onWindowStageCreate() ──── 关联的 WindowStage 创建
    │
    ├── 页面可见 ────────── 应用在前台
    │
    ▼
onForeground() ──── 从后台切回前台
    │
    ▼
onBackground() ──── 进入后台
    │
    ▼
onWindowStageDestroy() ──── WindowStage 销毁
    │
    ▼
onDestroy() ──── 应用最终销毁
```

### 完整代码示例
```typescript
import UIAbility, { Want } from '@ohos.app.ability.UIAbility'
import window from '@ohos.window'
import hilog from '@ohos.hilog'

const TAG = 'EntryAbility'
const DOMAIN = 0x0001

export default class EntryAbility extends UIAbility {
  storage: LocalStorage | null = null

  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(DOMAIN, TAG, 'onCreate')
    
    // 初始化操作
    this.storage = LocalStorage.getShared()
  }

  onDestroy(): void {
    hilog.info(DOMAIN, TAG, 'onDestroy')
    // 清理资源
    this.storage = null
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(DOMAIN, TAG, 'onWindowStageCreate')
    
    // 加载主页面
    windowStage.loadContent('pages/Index', this.storage, (err, data) => {
      if (err.code) {
        hilog.error(DOMAIN, TAG, '加载失败: %{public}s', JSON.stringify(err))
        return
      }
      hilog.info(DOMAIN, TAG, '加载成功')
    })
  }

  onWindowStageDestroy(): void {
    hilog.info(DOMAIN, TAG, 'onWindowStageDestroy')
    // 保存状态
  }

  onForeground(): void {
    hilog.info(DOMAIN, TAG, 'onForeground')
    // 恢复动画、刷新数据
  }

  onBackground(): void {
    hilog.info(DOMAIN, TAG, 'onBackground')
    // 暂停动画、释放资源
  }

  onNewWant(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(DOMAIN, TAG, 'onNewWant - 新的启动意图')
    // 被再次拉起时调用（如分享）
  }
}
```

## AbilityStage 模块上下文

```typescript
// EntryAbility.ts
import AbilityStage from '@ohos.app.ability.AbilityStage'

// src/main/etc/entryability/EntryAbility.ts
onCreate(): void {
  hilog.info(DOMAIN, TAG, 'Ability onCreate')
}

onAcceptWant(want: Want): string {
  // 启动指定ability（singleton模式）
  return 'EntryAbility'
}
```

### AbilityStage 应用生命周期
```typescript
// EntryAbility.ts
export default class EntryAbility extends AbilityStage {
  onConfigurationUpdate(config: Configuration): void {
    hilog.info(DOMAIN, TAG, '配置更新: %{public}s', JSON.stringify(config))
  }

  onMemoryLevel(level: AbilityConstant.MemoryLevel): void {
    switch (level) {
      case AbilityConstant.MemoryLevel.MEMORY_LEVEL_MODERATE:
        // 清理非必要缓存
        break
      case AbilityConstant.MemoryLevel.MEMORY_LEVEL_LOW:
        // 清理所有缓存
        break
      case AbilityConstant.MemoryLevel.MEMORY_LEVEL_CRITICAL:
        // 紧急释放资源
        break
    }
  }
}
```

## UIAbilityContext 常用 API

```typescript
import common from '@ohos.app.ability.common'
import Want from '@ohos.app.ability.Want'
import particleAbility from '@ohos.ability.particleAbility'

let context = getContext(this) as common.UIAbilityContext

// 启动 UIAbility
startAbility(want: Want): Promise<void>

// 启动Ability并获取结果
startAbilityForResult(want: Want): Promise<AbilityResult>

// 关闭当前Ability
terminateSelf(): Promise<void>

// 带结果关闭
terminateSelfWithResult(result: AbilityResult): Promise<void>

// 获取 Ability 名称
abilityName: string

// 获取应用 Bundle 名称
bundleName: string

// 获取或创建 Caller
callz = context.acquireCaller(abilityName, featureName)

// 获取 Callee
registerCallee(uri: string, callback: CalleeCallback): boolean
unregisterCallee(uri: string): boolean
```

## Want 启动参数

### 启动本地 Ability
```typescript
import Want from '@ohos.app.ability.Want'

// 基本启动
let want: Want = {
  deviceId: '',  // 空字符串表示本设备
  bundleName: 'com.example.myapp',
  abilityName: 'FuncAbility'
}

// 带参数启动
let wantWithParams: Want = {
  bundleName: 'com.example.myapp',
  abilityName: 'DetailAbility',
  parameters: {
    id: '12345',
    name: 'Tom',
    score: 95
  }
}

// 带选项启动
let options: common.StartOptions = {
  windowMode: 0,  // 默认窗口
  displayId: 0
}

context.startAbility(wantWithParams, options).then(() => {
  hilog.info(DOMAIN, TAG, '启动成功')
})
```

### 启动跨应用 Ability
```typescript
// 打开浏览器
let want: Want = {
  action: 'ohos.want.action.viewData',
  entities: ['entity.system.browsable'],
  uri: 'https://www.example.com'
}

// 打开地图
let mapWant: Want = {
  action: 'ohos.want.action.viewData',
  uri: 'geo:39.9,116.4?q=北京市'
}

// 发送邮件
let emailWant: Want = {
  action: 'ohos.want.action.send',
  uri: 'mailto:example@email.com'
}
```

### 启动模式
```typescript
// module.json5 中配置
{
  "abilities": [
    {
      "name": "EntryAbility",
      "launchType": "singleton",  // 单例（默认）
      // 或
      "launchType": "standard"   // 标准，每次创建新实例
    },
    {
      "name": "SpecifiedAbility",
      "launchType": "specified"  // 指定模式
    }
  ]
}

// 代码中判断
onAcceptWant(want: Want): string {
  if (want.parameters?.key === 'uniqueId') {
    return 'SpecifiedAbility'
  }
  return ''
}
```

## 页面间数据传递

### 方式1：通过 Want Parameters
```typescript
// 发送方
let want: Want = {
  abilityName: 'DetailAbility',
  parameters: {
    userId: 123,
    userName: 'Tom'
  }
}
context.startAbility(want)

// 接收方 - EntryAbility 中
onCreate(want: Want) {
  let userId = want.parameters?.userId as number
  let userName = want.parameters?.userName as string
}
```

### 方式2：通过 AppStorage 共享
```typescript
// 在 UIAbility.onCreate 中设置
AppStorage.setOrCreate('userId', 123)

// 任意页面读取
@StorageLink('userId') userId: number = 0
```

### 方式3：通过 LocalStorage
```typescript
// EntryAbility.onWindowStageCreate
let storage = new LocalStorage({ 'initData': 'value' })
windowStage.loadContent('pages/Index', storage)

// 页面中
@LocalStorageLink('initData') initData: string = ''
```

## 常见问题

### Q: 应用被系统销毁后如何恢复？
```typescript
// 在 UIAbility.onCreate 中获取启动参数
onCreate(want: Want, launchParam: AbilityConstant.LaunchParam) {
  // launchParam.launchReason 判断启动原因
  // AbilityConstant.LaunchReason.APP_RECOVERY 表示从崩溃恢复
  if (launchParam.launchReason === AbilityConstant.LaunchReason.APP_RECOVERY) {
    this.restoreState(launchParam.lastExitReason)
  }
}
```

### Q: 如何判断是冷启动还是热启动？
```typescript
onCreate(want: Want, launchParam: AbilityConstant.LaunchParam) {
  if (launchParam.lastLaunchReason === AbilityConstant.LaunchReason.INDEX_HOME) {
    // 冷启动
  } else if (launchParam.lastLaunchReason === AbilityConstant.LaunchReason.ABILITY_SHOW) {
    // 热启动
  }
}
```

---

> 关联：SKILL.md 核心文件 | 标签：UIAbility, Stage模型, 生命周期, Want
