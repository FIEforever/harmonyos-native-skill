# 权限申请与用户授权

## 权限类型

### 系统权限
| 权限名 | 说明 |
|--------|------|
| ohos.permission.INTERNET | 访问网络 |
| ohos.permission.CAMERA | 使用相机 |
| ohos.permission.RECORD_AUDIO | 录音 |
| ohos.permission.LOCATION | 位置 |
| ohos.permission.WRITE_MEDIA | 写入媒体 |
| ohos.permission.READ_MEDIA | 读取媒体 |
| ohos.permission.GET_NETWORK_INFO | 获取网络信息 |
| ohos.permission.BLUETOOTH | 蓝牙 |

### 权限级别
```
system_basic - 系统核心权限（需华为审核）
system_grant - 系统授权（用户无感知自动授予）
user_grant - 用户授权（需要用户手动同意）
```

## module.json5 声明

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:reason_internet",
        "usedScene": {
          "abilities": ["EntryAbility", "ServiceAbility"],
          "when": "always"
        }
      },
      {
        "name": "ohos.permission.CAMERA",
        "reason": "$string:reason_camera"
      },
      {
        "name": "ohos.permission.RECORD_AUDIO",
        "reason": "$string:reason_microphone"
      }
    ]
  }
}
```

## 权限申请两段式

### 方式1：使用 abilityAccessCtrl

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl'
import bundleManager from '@ohos.bundle.bundleManager'

async function checkAndRequestPermission(permission: string): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager()
  
  // 获取 Token
  const bundleInfo = bundleManager.getBundleInfoForSelfSync(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
  )
  const tokenId = bundleInfo.appInfo.accessTokenId
  
  // 检查权限状态
  const grantStatus = atManager.checkAccessTokenSync(
    tokenId,
    permission
  )
  
  if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
    return true  // 已有权限
  }
  
  // 请求权限
  return new Promise((resolve) => {
    atManager.requestGrant(tokenId, permission, {
      onRequestResult: (result) => {
        resolve(result.authResult === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED)
      }
    })
  })
}

// 使用
async function useCamera() {
  const hasPermission = await checkAndRequestPermission('ohos.permission.CAMERA')
  if (hasPermission) {
    // 正常调用相机功能
  } else {
    console.info('用户拒绝了相机权限')
  }
}
```

### 方式2：使用 PermissionsRequestCode

```typescript
import Ability from '@ohos.app.ability.UIAbility'
import abilityAccessCtrl from '@ohos.abilityAccessCtrl'
import PermissionRequestResult from '@ohos.abilityAccessCtrl'

export default class EntryAbility extends Ability {
  private static readonly PERMISSION_REQUEST_CODE = 100

  onForeground() {
    // 在需要时请求权限
    this.requestPermissionsForCamera()
  }

  private requestPermissionsForCamera() {
    let atManager = abilityAccessCtrl.createAtManager()
    
    atManager.requestPermissionsFromUser(
      getContext(this),
      ['ohos.permission.CAMERA'],
      EntryAbility.PERMISSION_REQUEST_CODE,
      (result: PermissionRequestResult) => {
        if (result.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
          console.info('权限申请成功')
        } else {
          console.info('权限被拒绝')
        }
      }
    )
  }
}
```

## 权限状态检查

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl'

function hasPermission(permission: string): boolean {
  try {
    const atManager = abilityAccessCtrl.createAtManager()
    const bundleInfo = bundleManager.getBundleInfoForSelfSync(
      bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    )
    
    const grantStatus = atManager.checkAccessTokenSync(
      bundleInfo.appInfo.accessTokenId,
      permission
    )
    
    return grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
  } catch {
    return false
  }
}

// 使用
if (!hasPermission('ohos.permission.CAMERA')) {
  // 提示用户开启权限
} else {
  // 使用相机
}
```

## 常见权限场景

### 相机拍照
```typescript
import camera from '@kit.CameraKit'

async function takePhoto() {
  // 1. 检查权限
  if (!hasPermission('ohos.permission.CAMERA')) {
    const granted = await checkAndRequestPermission('ohos.permission.CAMERA')
    if (!granted) return
  }
  
  // 2. 获取相机实例
  const cameraManager = camera.getCameraManager(getContext(this))
  const cameras = cameraManager.getSupportedCameras()
  
  if (cameras.length === 0) {
    console.error('没有可用相机')
    return
  }
  
  // 3. 创建输入
  const cameraInput = cameraManager.createCameraInput(cameras[0].cameraId)
  await cameraInput.open()
  
  // 4. 创建预览和拍照
  // ...
}
```

### 位置服务
```typescript
import geolocation from '@ohos.geoLocationManager'

async function getLocation() {
  // 检查权限
  const hasLocation = hasPermission('ohos.permission.LOCATION')
  
  if (!hasLocation) {
    await checkAndRequestPermission('ohos.permission.LOCATION')
  }
  
  // 获取位置
  try {
    const location = await geolocation.getCurrentLocation({
      priority: geolocation.LocationRequestPriority.FIRST_FIX,
      scenario: geolocation.LocationRequestScenario.UNSET
    })
    console.info(`位置: ${location.latitude}, ${location.longitude}`)
  } catch (err) {
    console.error(`获取失败: ${err.message}`)
  }
}
```

## 权限被拒绝处理

```typescript
@CustomDialog
struct PermissionDialog {
  controller: CustomDialogController
  title: string = ''
  message: string = ''
  confirm: () => void = () => {}
  cancel: () => void = () => {}

  build() {
    Column() {
      Text(this.title)
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      Text(this.message)
        .fontSize(16)
        .margin({ top: 12, bottom: 24 })
      
      Row({ space: 24 }) {
        Button('取消')
          .onClick(() => {
            this.cancel()
            this.controller.close()
          })
        
        Button('去设置')
          .onClick(() => {
            this.confirm()
            this.controller.close()
          })
      }
    }
    .padding(24)
    .width('80%')
  }
}

// 调用
if (!hasPermission) {
  const dialog = new CustomDialogController({
    builder: PermissionDialog({
      title: '权限申请',
      message: '需要相机权限才能拍照，请在设置中开启',
      confirm: () => {
        // 打开应用设置页面
        app.startAbility({
          action: 'action.settings. APPLICATION_SETTINGS'
        })
      }
    })
  })
  dialog.open()
}
```

## 敏感权限理由配置

在 `resources/base/element/string.json` 中：

```json
{
  "string": [
    {
      "name": "reason_camera",
      "value": "需要使用相机功能来拍摄照片和录制视频"
    },
    {
      "name": "reason_microphone",
      "value": "需要使用麦克风来录制音频"
    },
    {
      "name": "reason_location",
      "value": "需要获取您的位置信息来提供定位服务"
    },
    {
      "name": "reason_storage",
      "value": "需要访问存储来保存和读取文件"
    },
    {
      "name": "reason_internet",
      "value": "需要网络连接来同步数据"
    }
  ]
}
```

---

> 关联：SKILL.md 核心文件 | 标签：权限, 授权, user_grant, 隐私
