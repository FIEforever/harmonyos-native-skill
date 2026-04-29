# ArkUI 组件与布局体系

## 基础布局容器

### Column / Row / Stack
```typescript
// 垂直排列
Column({ space: 12 }) {
  Text('第一行')
  Text('第二行')
}
.width('100%')
.height(200)
.backgroundColor('#F5F5F5')

// 水平排列
Row({ space: 8 }) {
  Image('/assets/icon.png').width(24).height(24)
  Text('标题')
}

// 层叠布局
Stack() {
  Image('/assets/bg.jpg')
  Text('叠加文字').fontSize(24)
}
.alignContent(Alignment.Center)
```

### 相对定位 RelativeContainer
```typescript
RelativeContainer() {
  Text('左侧')
    .id('left')
    .alignRules({
      left: { anchor: '__container__', align: HorizontalAlign.Start },
      center: { anchor: '__container__', align: HorizontalAlign.Center }
    })
    
  Text('右侧')
    .id('right')
    .alignRules({
      left: { anchor: 'left', align: HorizontalAlign.End }
    })
}
.height(200)
```

### 栅格布局 GridRow / GridCol
```typescript
GridRow({
  columns: { sm: 4, md: 8, lg: 12 },  // 响应式列数
  breakpoints: { value: ['320vp', '600vp', '840vp'] },
  gutter: { x: 16, y: 16 }
}) {
  GridCol({ span: 4 }) { Text('1/3宽度') }
  GridCol({ span: 8 }) { Text('2/3宽度') }
  GridCol({ span: { sm: 12, lg: 6 } }) { Text('响应式') }
}
```

## 容器组件

### List / ListItem / ListItemGroup
```typescript
List({ space: 12, initialIndex: 0 }) {
  ListItem() { Text('第一项') }
  ListItem() { Text('第二项') }
  
  ListItemGroup({ header: this.listGroupHeader, footer: this.listGroupFooter }) {
    ForEach(this.groupItems, (item: string) => {
      ListItem() { Text(item) }
    })
  }
}
.width('100%')
.height(300)
.divider({ strokeWidth: 1, color: '#eee' })
.onScrollIndexChange((start, end) => {
  console.info(`可见: ${start}-${end}`)
})
```

### Grid / GridItem
```typescript
Grid(undefined, { irregulartoSize: ['1', '2', '1', '3'] }) {
  GridItem() { Text('0') }
  GridItem() { Text('1') }
  GridItem() { Text('2') }
  GridItem() { Text('3') }
  GridItem() { Text('4') }
}
.columnsTemplate('1fr 1fr 1fr')
.rowsTemplate('1fr 1fr 1fr')
.width(300)
.height(300)
```

### WaterFlow
```typescript
WaterFlow() {
  LazyForEach(this.dataSource, (item: ItemInfo) => {
    FlowItem() {
      Text(item.name).width('100%').height(item.height)
    }
  }, (item: ItemInfo) => item.id.toString())
}
.width('100%')
.height('100%')
.columnsTemplate('1fr 1fr')
.itemConstraintSize({
  minWidth: 150,
  maxWidth: 200,
  minHeight: 100,
  maxHeight: 200
})
```

## 基础组件

### Text
```typescript
Text('Hello')
  .fontSize(20)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333')
  .fontStyle(FontStyle.Italic)
  .textAlign(TextAlign.Center)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })
  .decoration({ type: TextDecorationType.LineThrough, color: 'red' })
```

### Image
```typescript
Image($r('app.media.icon'))
  .width(100)
  .height(100)
  .borderRadius(50)  // 圆形
  .fitMatchContent()
  .objectFit(ImageFit.Contain)
  .alt($r('app.media.placeholder'))
```

### Button / Radio / Checkbox
```typescript
// 普通按钮
Button('点击', { type: ButtonType.Normal, stateEffect: true })
  .onClick(() => {})
  .enabled(true)

// 图标按钮
Button({ type: ButtonType.Circle, stateEffect: false }) {
  Image($r('app.media.icon')).width(24)
}

// 单选
Radio('选项1').checked(false)
  .onChange((isChecked: boolean) => {})

// 多选
Checkbox('同意协议').select(false)
```

## 表单组件

### TextInput / TextArea
```typescript
TextInput({ placeholder: '请输入', text: this.inputText })
  .type(InputType.Number)
  .maxLength(11)
  .inputFilter('[a-zA-Z0-9]')
  .onChange((value: string) => {
    this.inputText = value
  })

TextArea({ placeholder: '多行文本' })
  .height(120)
  .showCounter(true)
```

### Slider / Switch / Rating
```typescript
Slider({
  value: this.sliderValue,
  min: 0,
  max: 100,
  step: 1,
  style: SliderStyle.OutSet
})
.onChange((value: number, mode: SliderChangeMode) => {
  this.sliderValue = value
})

Switch('深色模式')
  .checked(this.isDark)
  .onChange((isChecked: boolean) => {
    this.isDark = isChecked
  })

Rating({ rating: 4.5, indicator: false })
  .stars(5)
  .allowSemiCompleteRating(false)
```

## 导航组件

### Navigation + NavPathStack（推荐）
```typescript
// 主页面
@Entry
@Component
struct Index {
  pathStack: NavPathStack = new NavPathStack()
  
  @Builder
  pageMap(name: string) {
    if (name === 'Detail') {
      DetailPage()
    } else if (name === 'Settings') {
      SettingsPage()
    }
  }
  
  build() {
    Navigation(this.pathStack) {
      List() {
        ListItem() {
          Text('详情页').onClick(() => {
            this.pathStack.pushPath({ name: 'Detail', params: { id: 1 } })
          })
        }
      }
    }
    .title('首页')
    .navDestination(this.pageMap)
  }
}

// 详情页
@Component
struct DetailPage {
  pathStack: NavPathStack = NavPathStack.getContext(this)
  
  aboutToAppear() {
    // ❌ 不要在这里获取参数
  }
  
  onReady() {
    // ✅ 在 onReady 中获取
    let params = this.pathStack.getParamByName('Detail')
    console.info('参数:', JSON.stringify(params))
  }
  
  build() {
    NavDestination() {
      Column() {
        Text('详情页')
        Button('返回').onClick(() => this.pathStack.pop())
      }
    }
    .title('详情')
  }
}
```

### Tabs / TabBar
```typescript
Tabs({ barPosition: BarPosition.End }) {
  TabContent() { HomeContent() }.tabBar('首页')
  TabContent() { MineContent() }.tabBar('我的')
}
.animationDuration(300)
.onTabBarClick((index: number) => {
  console.info(`切换到 ${index}`)
})
```

## 手势系统

### 基础手势
```typescript
.gesture(
  TapGesture({ count: 2 })
    .onAction(() => { console.info('双击') })
)

.gesture(
  LongPressGesture({ duration: 500 })
    .onAction(() => { console.info('长按') })
)
```

### PanGesture 拖拽
```typescript
.gesture(
  PanGesture()
    .onActionStart((event: GestureEvent) => {
      console.info(`开始: ${event.localX}, ${event.localY}`)
    })
    .onActionUpdate((event: GestureEvent) => {
      // 实时更新位置
    })
    .onActionEnd((event: GestureEvent) => {
      console.info(`结束: 速度 ${event.velocity}`)
    })
)
```

### 组合手势
```typescript
.gesture(
  GestureGroup(GestureMode.Parallel,
    PinchGesture()
      .onActionUpdate((event) => {
        this.scale = event.scale
      })
      .onActionEnd(() => {
        animateTo({ duration: 300 }, () => {
          this.scale = 1
        })
      }),
    RotationGesture()
      .onActionUpdate((event) => {
        this.angle = event.angle
      })
  )
)
```

## 动画

### 显式动画 animateTo
```typescript
animateTo({
  duration: 300,
  curve: Curve.EaseInOut,
  iterations: 1,
  playMode: PlayMode.Normal,
  delay: 0
}, () => {
  this.offsetX = 100
  this.offsetY = 50
  this.opacityValue = 0.5
})
```

### 关键帧动画
```typescript
animateTo({
  duration: 1000,
  keyframes: [
    { duration: 0, scale: 1, translateX: 0 },
    { duration: 500, scale: 1.2, translateX: 50 },
    { duration: 1000, scale: 1, translateX: 100 }
  ]
}, () => {
  this.transformContent = true
})
```

### transition 转场动画
```typescript
// 页面内元素
Button('Click')
  .transition(
    TransitionEffect.OPACITY.animation({ duration: 500 })
      .combine(TransitionEffect.scale({ x: 0.5, y: 0.5 }))
  )

// NavDestination
NavDestination()
  .transition(
    TransitionEffect.translate({ x: 300, y: 0 })
      .animation({ duration: 300 })
  )
```

### 属性动画
```typescript
// 监听属性变化自动动画
.width(this.isExpanded ? 200 : 100)
.animation({
  duration: 300,
  curve: Curve.EaseOut
})
```

## 通用属性

### 尺寸与位置
```typescript
.width('100%')           // 撑满父容器
.height(200)             // 固定高度
.padding(16)             // 内边距
.margin({ top: 8 })      // 外边距
.constraintSize({ maxWidth: 300 })  // 约束尺寸
```

### 背景与边框
```typescript
.backgroundColor('#FFFFFF')
.backgroundImage('/assets/bg.png', ImageRepeat.XY)
.border({
  width: 1,
  color: '#eee',
  radius: 8,
  style: BorderStyle.Solid
})
.shadow({
  radius: 10,
  color: '#40000000',
  offsetX: 0,
  offsetY: 4
})
.opacity(0.8)
```

### 触摸状态
```typescript
.enabled(true)
.focusable(true)
.focusOnTouch(true)
.responseRegion({
  x: 0,
  y: 0,
  width: '100%',
  height: '100%'
})
```

---

> 关联：SKILL.md 核心文件 | 标签：ArkUI, 组件, 布局, 手势, 动画
