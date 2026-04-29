# DevEco Studio 工具与调试

## DevEco Profiler 性能分析

### CPU Profiler
```typescript
// 在 DevEco Studio 中
// 1. Run > Profile 或点击工具栏 Profiler 图标
// 2. 选择 "CPU" 标签
// 3. 点击 Record 开始录制
// 4. 执行需要分析的操作
// 5. 点击 Stop 停止录制
// 6. 查看 Call Chart / Flame Chart 分析
```

### 内存分析
```typescript
// 1. 选择 "Memory" 标签
// 2. 查看堆内存使用情况
// 3. 点击 "Dump Java heap" 获取堆快照
// 4. 分析内存泄漏和优化点

// 代码中主动触发 GC（仅调试用）
import forceGC from '@ohos.foundation.forceGC'
forceGC()
```

### 帧率分析
```typescript
// 1. 选择 "Frame Performance" 标签
// 2. 查看 UI 渲染帧率
// 3. 目标：稳定 60fps（16.67ms/帧）
// 4. 查看 "Jank" 标记的卡顿帧
```

### 能耗分析
```typescript
// 1. 连接真机（非模拟器）
// 2. 选择 "Power" 标签
// 3. 查看各模块功耗占比
// 4. 优化高功耗操作
```

## DevEco Testing 测试

### 单元测试
```typescript
// entry/src/test/ets/test/List.test.ets
import { describe, it, expect } from '@ohos.hypium'

export default function listTests() {
  describe('list_test_suite', () => {
    it('should_contain_element', 0, () => {
      let list = [1, 2, 3]
      expect(list.contains(2)).assertTrue()
    })
    
    it('should_equal_value', 0, () => {
      let value = 10
      expect(value).assertEqual(10)
    })
  })
}
```

### UI 测试
```typescript
// 使用 UI 测试 API
import driver from '@ohos.testing.driver'

// 启动测试
await driver.create()
// 找到按钮
let button = await driver.findComponent(By.text('点击'))
await button.click()
// 验证结果
let text = await driver.findComponent(By.text('1'))
expect(await text.text()).assertEqual('1')
```

## HAP/HAR/HSP 打包

### HAP 模块类型
```typescript
// module.json5
{
  "module": {
    "type": "entry",      // entry 或 feature
    "name": "entry",
    
    "distro": {
      "moduleName": "entry",
      "moduleType": "entry",      // entry, feature, har
      "installationFree": false    // 是否免安装
    }
  }
}
```

### HAR (Harmony Archive)
```typescript
// 创建 HAR 模块
// 1. New > Module > Harmony Library
// 2. 编写导出内容

// entry/src/main/module.json5
{
  "module": {
    "type": "har"
  }
}

// 导出
export { identity } from './utils/common'
export { UserService } from './service/UserService'
export { default as Config } from './config'
```

### HAR 依赖
```typescript
// 在 entry 或其他模块的 oh-package.json5 中
{
  "dependencies": {
    "my-har": "file:../my-har"
    // 或
    // "@scope/my-har": "^1.0.0"
  }
}
```

### HSP (Harmony Shared Package)
```typescript
// 创建 HSP - 共享代码但不打包到应用中
// 适用于：多个 Entry 共享同一功能

{
  "module": {
    "type": "shared"  // HSP 类型
  }
}
```

## BundleManager 应用管理

```typescript
import bundleManager from '@ohos.bundle.bundleManager'
import bundle from '@ohos.bundle.bundleManager'

// 获取应用信息
const bundleInfo = bundleManager.getBundleInfoForSelfSync(
  bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
)

console.info(`包名: ${bundleInfo.name}`)
console.info(`版本: ${bundleInfo.versionName}`)

// 获取已安装应用列表
bundleManager.getBundleInfos(
  bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION,
  (err, infos) => {
    if (err) {
      console.error(`查询失败: ${err.message}`)
      return
    }
    console.info(`已安装 ${infos.length} 个应用`)
  }
)
```

## AppStartup 应用启动优化

### 配置启动任务
```typescript
// StartupLoader 负责初始化启动任务
// module.json5
{
  "app": {
    "startup": {
      "tasks": [
        {
          "name": "InitTask",
          "srcPath": "InitTask.ets",
          "dependencies": [],
          "runOnNewThread": true
        }
      ]
    }
  }
}
```

### 创建启动任务
```typescript
// entry/src/main/ets/startup/InitTask.ets
import AppStartup from '@ohos.app.ability.AppStartup'

class InitTask extends AppStartup {
  onCreate() {
    // 初始化操作
    console.info('InitTask onCreate')
  }

  onDestroy() {
    console.info('InitTask onDestroy')
  }
}

export default InitTask
```

## 应用签名

### 自动签名（推荐）
```typescript
// DevEco Studio 中
// File > Project Structure > Signing Configs
// 勾选 "Automatically generate signature"
// 选择设备类型 > 点击 Sign In 注册华为账号
```

### 手动签名
```typescript
// 1. 申请华为开发者账号
// 2. 创建应用
// 3. 下载签名证书 (.p12, .cer, .pem)
// 4. 配置签名

// build-profile.json5
{
  "signingConfigs": [
    {
      "name": "default",
      "certificatePath": "./signature/certificate.pem"
    }
  ],
  "products": [
    {
      "name": "default",
      "signingConfig": "default"
    }
  ]
}
```

### 调试证书生成
```typescript
// 使用 keytool 生成调试签名
// keytool -genkey -alias debug -keyalg RSA -keystore debug.keystore -keysize 2048 -validity 10000 -storepass 123456 -keypass 123456 -dname "CN=debug, O=debug, C=CN"
```

---

> 关联：SKILL.md 核心文件 | 标签：DevEco, Profiler, Testing, 打包, 签名
