# Context / Want / 跨应用跳转

## Context 体系

### Context 类型
```
AppContext           - 应用级别（进程唯一）
ApplicationContext   - 应用生命周期管理
AbilityStageContext  - AbilityStage 上下文
UIAbilityContext     - UIAbility 上下文
WindowStageContext   - 窗口上下文
ExtensionContext     - 扩展能力上下文
```

### 获取 Context
```typescript
// UIAbility 中
let context = getContext(this)  // UIAbilityContext
let appContext = context.getApplicationContext()  // ApplicationContext

// ArkUI 组件中
let context = getContext(this)

// ServiceAbility 中
let context = this.context
```

### ApplicationContext
```typescript
import common from '@ohos.app.ability.common'

const appContext = getContext(this).getApplicationContext()

// 应用生命周期
appContext.on('abilityLifecycle', (data) => {
  console.info(`Ability 生命周期: ${data.type}`)
  // data.type: AbilityLifecycleCallback.INDEX_CREATE / DESTROY / FOREGROUND / BACKGROUND
})

// 应用退出
appContext.terminateSelf()
```

### AbilityStageContext
```typescript
import AbilityStage from '@ohos.app.ability.AbilityStage'

export default class MyAbilityStage extends AbilityStage {
  onAcceptWant(want: Want): string {
    // 返回 Ability 名
    return 'MyAbility'
  }
}
```

## Want 详解

### Want 结构
```typescript
interface Want {
  deviceId?: string           // 目标设备ID
  bundleName?: string        // 目标包名
  abilityName?: string       // 目标 Ability 名
  moduleName?: string        // 目标模块名
  uri?: string               // 资源 URI
  type?: string              // 数据类型
  action?: string            // 动作
  entities?: string[]         // 实体
  flags?: number             // 标记
  parameters?: Record<string, Object>  // 自定义参数
  want?: Want                // 嵌套 Want
}
```

### 常用 Action
```typescript
// 打开链接
want.action = 'ohos.want.action.viewData'
want.uri = 'https://www.example.com'

// 发送邮件
want.action = 'ohos.want.action.send'
want.uri = 'mailto:example@email.com'

// 拨打电话
want.action = 'ohos.want.action.dial'
want.uri = 'tel:13800138000'

// 打开设置
want.action = 'action.settings. APPLICATION_SETTINGS'

// 分享
want.action = 'ohos.want.action.send'
want.parameters = {
  'ability.want.action.send.data': '分享内容'
}
```

### 常用 Entities
```typescript
// 网页
want.entities = ['entity.system.browsable']

// 文件
want.entities = ['entity.system.file']

// 播放器
want.entities = ['entity.system.music']
```

## 启动其他应用

### 打开浏览器
```typescript
import common from '@ohos.app.ability.common'

function openBrowser(url: string) {
  let context = getContext(this) as common.UIAbilityContext
  let want: Want = {
    action: 'ohos.want.action.viewData',
    uri: url
  }
  
  context.startAbility(want).catch((err) => {
    console.error(`打开失败: ${err.message}`)
  })
}

openBrowser('https://www.example.com')
```

### 打开地图
```typescript
function openMap(latitude: number, longitude: number, label?: string) {
  let context = getContext(this) as common.UIAbilityContext
  let uri = `geo:${latitude},${longitude}`
  if (label) {
    uri += `?q=${encodeURIComponent(label)}`
  }
  
  let want: Want = {
    action: 'ohos.want.action.viewData',
    uri: uri
  }
  
  context.startAbility(want)
}

openMap(39.9042, 116.4074, '北京市')
```

### 发送短信
```typescript
function sendSMS(phone: string, content: string) {
  let want: Want = {
    action: 'ohos.want.action.send',
    uri: `sms:${phone}`,
    parameters: {
      'ability.want.constant.parcelable.sms_body': content
    }
  }
  
  startAbility(want)
}
```

### 拨打电话
```typescript
function dialPhone(phone: string) {
  let want: Want = {
    action: 'ohos.want.action.dial',
    uri: `tel:${phone}`
  }
  
  startAbility(want)
}
```

### 分享内容
```typescript
function shareText(text: string) {
  let want: Want = {
    action: 'ohos.want.action.send',
    parameters: {
      'ability.want.constant.parcelable.shared': text
    }
  }
  
  startAbility(want)
}
```

### 打开文件
```typescript
import filePicker from '@ohos.file.fileExtensionAbility'

async function openFile(fileUri: string) {
  let context = getContext(this) as common.UIAbilityContext
  
  let want: Want = {
    action: 'ohos.want.action.viewData',
    uri: fileUri
  }
  
  try {
    await context.startAbility(want)
  } catch (err) {
    console.error(`无法打开文件: ${err.message}`)
  }
}
```

## 跨应用跳转回调

### startAbilityForResult 获取返回
```typescript
import AbilityResult from '@ohos.ability.abilityResult'

// 启动并等待结果
let want: Want = {
  bundleName: 'com.example.otherapp',
  abilityName: 'ResultAbility'
}

let context = getContext(this) as common.UIAbilityContext

try {
  const result = await context.startAbilityForResult(want)
  
  if (result.resultCode === 0) {
    let data = result.want?.parameters?.['data']
    console.info(`收到数据: ${data}`)
  } else {
    console.info('用户取消')
  }
} catch (err) {
  console.error(`启动失败: ${err.message}`)
}
```

### 带参数传递
```typescript
// 发送方
let want: Want = {
  abilityName: 'PickerAbility',
  parameters: {
    'multiSelect': true,
    'maxCount': 9,
    'type': 'image/*'
  }
}

let result = await context.startAbilityForResult(want)

// 接收方 (PickerAbility)
onCreate(want: Want) {
  let multiSelect = want.parameters?.multiSelect as boolean
  let maxCount = want.parameters?.maxCount as number
  
  // 选择完成后返回结果
  let resultWant: Want = {
    parameters: {
      'selectedPaths': ['/path/1', '/path/2']
    }
  }
  
  let result: AbilityResult = {
    resultCode: 0,
    want: resultWant
  }
  
  terminateSelfWithResult(result)
}
```

## 常见问题

### Q: 如何判断目标应用是否安装？
```typescript
import bundleManager from '@ohos.bundle.bundleManager'

async function isAppInstalled(bundleName: string): Promise<boolean> {
  try {
    await bundleManager.getBundleInfo(
      bundleName,
      bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    )
    return true
  } catch {
    return false
  }
}

const installed = await isAppInstalled('com.example.otherapp')
```

### Q: 如何获取第三方应用的包名？
```typescript
// 查看设备上已安装的应用
bundleManager.getAllInstalledBundleInfo(
  bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
).then((infos) => {
  infos.forEach(info => {
    console.info(`包名: ${info.name}, 名称: ${info.label}`)
  })
})
```

---

> 关联：SKILL.md 核心文件 | 标签：Context, Want, 跨应用, 跳转, 分享
