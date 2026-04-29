# 状态管理 V1 / V2 / 组件通信

## V1 状态管理 (@State/@Prop/@Link等)

### @State 组件状态
```typescript
@Entry
@Component
struct Counter {
  @State count: number = 0  // 简单类型
  @State message: string = 'Hello'
  
  build() {
    Column() {
      Text(`${this.count}`)
      Button('+1').onClick(() => this.count++)
      Button('重置').onClick(() => { this.count = 0 })
    }
  }
}
```

### @Prop 单向数据流（父→子）
```typescript
@Component
struct ChildComponent {
  @Prop title: string  // 父组件传入
  @Prop count: number
  
  build() { Text(`${this.title}: ${this.count}`) }
}

@Entry
@Component
struct Parent {
  @State message: string = '计数'
  @State num: number = 0
  
  build() {
    ChildComponent({ title: this.message, count: this.num })
  }
}
```

### @Link 双向绑定
```typescript
@Component
struct CounterChild {
  @Link count: number  // 必须初始化
  
  build() {
    Button(`计数: ${this.count}`).onClick(() => this.count++)
  }
}

@Entry
@Component
struct Parent {
  @State parentCount: number = 0
  
  build() {
    CounterChild({ count: $parentCount })
  }
}
```

⚠️ **注意**: `@Link` 在 `@Entry` 组件的**直接子组件**中使用有限制，需确保父组件存在。

### @Provide / @Consume 跨层级传值
```typescript
@Entry
@Component
struct Ancestor {
  @Provide message: string = '祖先数据'
  
  build() {
    Parent()  // 中间组件不需传值
  }
}

@Component
struct Parent {
  // 无需任何装饰器，自动获取 @Provide
}

@Component
struct Child {
  @Consume message: string
  
  build() {
    Text(this.message)
  }
}
```

### @Observed / @ObjectLink 对象监听
```typescript
@Observed
class UserInfo {
  name: string = ''
  age: number = 0
  friends: string[] = []
}

@Entry
@Component
struct Parent {
  @State userInfo: UserInfo = new UserInfo()
  
  build() {
    ChildComponent({ userInfo: this.userInfo })
  }
}

@Component
struct ChildComponent {
  @ObjectLink userInfo: UserInfo
  
  build() {
    Column() {
      Text(this.userInfo.name)
      Button('改名字').onClick(() => {
        // ⚠️ 必须解构赋值触发更新
        this.userInfo = new UserInfo({ ...this.userInfo, name: '新名字' })
      })
    }
  }
}
```

## V2 状态管理 (@ComponentV2)

### @Local 本地状态（替代 @State）
```typescript
@Entry
@ComponentV2
struct MyPage {
  @Local count: number = 0
  @Local message: string = ''
  @Local isLoading: boolean = false
  
  build() {
    Column() {
      Text(`${this.count}`)
      Button('+1').onClick(() => this.count++)
      TextInput({ placeholder: '输入' })
        .onChange(v => this.message = v)
    }
  }
}
```

### @Param 参数（替代 @Prop）
```typescript
@ComponentV2
struct UserCard {
  @Param name: string = ''
  @Param avatar: ResourceStr = ''
  @Param isSelected: boolean = false
  
  @Event onSelect: (name: string) => void = () => {}
  
  build() {
    Row() {
      Image(this.avatar).width(40).height(40)
      Text(this.name)
      if (this.isSelected) {
        Text('✓').fontColor('#07C160')
      }
    }
    .onClick(() => this.onSelect(this.name))
  }
}
```

### @Event 事件（替代回调）
```typescript
@Entry
@ComponentV2
struct Parent {
  @Local selectedName: string = ''
  
  build() {
    UserCard({
      name: 'Tom',
      avatar: $r('app.media.avatar'),
      onSelect: (name: string) => {
        this.selectedName = name
      }
    })
  }
}
```

### @Monitor 监听变化
```typescript
@Entry
@ComponentV2
struct FormPage {
  @Local email: string = ''
  @Local password: string = ''
  @Local isValid: boolean = false
  
  @Monitor('email', 'password')  // 监听多个
  onFormChange() {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    this.isValid = emailRegex.test(this.email) && this.password.length >= 6
  }
  
  build() {
    Column() {
      TextInput({ placeholder: '邮箱' })
        .onChange(v => this.email = v)
      TextInput({ placeholder: '密码', type: InputType.Password })
        .onChange(v => this.password = v)
      Button('提交', { enabled: this.isValid })
    }
  }
}
```

### @Computed 计算属性
```typescript
@ComponentV2
struct PriceDisplay {
  @Param price: number = 0
  @Param quantity: number = 1
  
  @Computed
  get total(): number {
    return this.price * this.quantity
  }
  
  @Computed
  get discount(): number {
    return this.total >= 100 ? this.total * 0.1 : 0
  }
  
  build() {
    Column() {
      Text(`总价: ¥${this.total.toFixed(2)}`)
      Text(`优惠: -¥${this.discount.toFixed(2)}`)
    }
  }
}
```

### @Trace 属性追踪
```typescript
@ComponentV2
struct MonitoredComponent {
  @Trace traceProperty: string = ''
  
  build() {
    Text(this.traceProperty)
  }
}
```

## V1 → V2 迁移检查清单

| V1 | V2 | 注意事项 |
|----|----|---------|
| `@State` | `@Local` | 组件内状态 |
| `@Prop` | `@Param` | 单向传入 |
| `@Link` | `@Param` + `@Event` | 双向绑定 |
| `@Provide/@Consume` | `@Local` + `@Event` | 跨层级 |
| `@ObjectLink` | `@Param` + `@Trace` | 对象属性 |
| `@Watch` | `@Monitor` | 监听变化 |
| — | `@Computed` | 新增计算属性 |

```typescript
// 迁移示例
// V1
@State list: string[] = []
ListItem() {
  Text(this.list[index])
}

// V2
@Local list: string[] = []
ForEach(this.list, (item: string, index: number) => {
  ListItem() { Text(item) }
})
```

## 组件通信

### EventHub 事件中心
```typescript
// 发布者
import common from '@ohos.app.ability.common'

let context = getContext(this) as common.UIAbilityContext
context.eventHub.emit('eventName', { data: 'value' })
context.eventHub.emit('eventName')  // 无参

// 订阅者
context.eventHub.on('eventName', (data) => {
  console.info('收到:', JSON.stringify(data))
})

// 单次订阅
context.eventHub.once('eventName', (data) => {})

// 取消订阅
context.eventHub.off('eventName')
```

### AppStorage 全局状态
```typescript
// 存储
AppStorage.setOrCreate('theme', 'dark')
AppStorage.setOrCreate('userId', 123)

// 读取
let theme = AppStorage.get<string>('theme')

// 与 @StorageLink 双向绑定
@StorageLink('theme') theme: string = 'light'

// 持久化
AppStorage.SetOrCreate('settings', JSON.stringify(settings))
PersistentStorage.persistProp('theme', 'light')
```

### LocalStorage 页面级存储
```typescript
let storage = new LocalStorage({
  'pageTitle': '默认标题',
  'count': 0
} as Record<string, number | string>)

@Entry({ storage: storage })
@Component
struct MyPage {
  @LocalStorageLink('pageTitle') title: string = ''
  
  aboutToAppear() {
    let value = storage.get<string>('pageTitle')
  }
}
```

## 状态持久化流程

```
用户交互 → @State 更新
     ↓
AppStorage/PersistentStorage (可选持久化)
     ↓
Preferences/RelationalStore (持久化存储)
```

```typescript
// 完整持久化示例
import preferences from '@ohos.data.preferences'

@Entry
@Component
struct Settings {
  @Local isDark: boolean = false
  
  private context = getContext(this)
  private dataPreferences: preferences.Preferences | null = null
  
  async aboutToAppear() {
    this.dataPreferences = await preferences.getPreferences(this.context, 'settings')
    this.isDark = this.dataPreferences.getSync('darkMode', false)
  }
  
  async toggleTheme() {
    this.isDark = !this.isDark
    if (this.dataPreferences) {
      await this.dataPreferences.put('darkMode', this.isDark)
      await this.dataPreferences.flush()
    }
  }
}
```

## 边界情况处理

### 状态初始化顺序
```typescript
// ✅ 正确：@State 初始化
@State list: string[] = []
@State map: Map<string, number> = new Map()

// ❌ 错误：避免在初始化时使用未定义的变量
@State data = getData()  // getData 可能依赖未初始化的服务
```

### 循环引用问题
```typescript
// ❌ 错误：对象循环引用
@State obj: any = { ref: null }
obj.ref = obj  // 禁止

// ✅ 正确：使用 @Observed 包装
@Observed
class Container {
  child: Child | null = null
}
```

### 大数组性能优化
```typescript
// ❌ 避免：频繁重建大数组
@State items: number[] = []
onUpdate() {
  this.items = [...this.items, ...newItems]  // 每次创建新数组
}

// ✅ 推荐：使用 LazyForEach + IDataSource
// 参考 02-arkui-components.md 的 List 相关章节
```

---

> 关联：SKILL.md 核心文件 | 标签：状态管理, V1, V2, @State, @ComponentV2
