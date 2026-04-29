# HarmonyOS Design 设计规范

## 设计理念

HarmonyOS 设计追求「**意图优先，一致性**」：
- 降低认知负担
- 保持视觉一致性
- 符合直觉的交互

## 色彩系统

### 基础色板
| 色彩 | 色值 | 用途 |
|------|------|------|
| #007DFF | 蓝 | 主要操作、链接 |
| #007DFF (10%) | 浅蓝 | 背景色 |
| #F7CE46 | 黄 | 警示、提醒 |
| #E84026 | 红 | 错误、危险 |
| #27C93F | 绿 | 成功、安全 |

### 文字色彩
| 层级 | 色值 | 透明度 | 用途 |
|------|------|--------|------|
| 一级文字 | #000000 | 90% | 标题、重要内容 |
| 二级文字 | #000000 | 60% | 正文 |
| 三级文字 | #000000 | 38% | 辅助说明 |
| 占位文字 | #000000 | 26% | 输入框占位符 |

### 深色模式适配
```typescript
@Styles
function cardStyle() {
  .backgroundColor($r('app.color.card_background'))
  .borderRadius(12)
  .padding(16)
}

// 在 resources/dark/element/color.json
// 定义 dark 模式下的颜色值
```

## 字体规范

### 字号体系
| 名称 | 字号 | 字重 | 行高 | 用途 |
|------|------|------|------|------|
| Title-L | 36vp | Bold | 44vp | 大标题 |
| Title-M | 28vp | Bold | 36vp | 标题 |
| Title-S | 22vp | Bold | 28vp | 小标题 |
| Body-L | 18vp | Regular | 26vp | 正文大号 |
| Body-M | 16vp | Regular | 24vp | 正文 |
| Body-S | 14vp | Regular | 20vp | 正文小号 |
| Caption | 12vp | Regular | 16vp | 辅助文字 |

### 使用方式
```typescript
Text('标题')
  .fontSize($r('sys.float.ohos_id_text_size_title1'))
  .fontWeight(FontWeight.Bold)
  .fontFamily($r('sys.str.ohos_id_card_font_family'))
```

## 间距系统

### 基础间距单位
```
4vp - 最小间距（图标与文字间隙）
8vp - 紧凑间距
12vp - 标准间距
16vp - 宽松间距
24vp - 大间距（模块之间）
32vp - 特大间距
```

### 组件间距
```typescript
// 列表项内间距
ListItem().padding({ left: 16, right: 16 })

// 卡片内边距
Column() {
  // 内容
}
.padding(16)

// 按钮内间距
Button('确定')
  .height(40)
  .padding({ left: 24, right: 24 })
```

## 圆角系统

| 尺寸 | 圆角值 | 适用场景 |
|------|--------|---------|
| 小圆角 | 4vp | 小按钮、标签 |
| 中圆角 | 8vp | 输入框、卡片 |
| 大圆角 | 12vp | 弹窗、浮层 |
| 超大圆角 | 16vp | 底部弹窗 |
| 全圆 | 50% | 头像、圆形按钮 |

```typescript
Button('圆角按钮')
  .borderRadius(8)

Image($r('app.media.avatar'))
  .borderRadius(50)  // 圆形
```

## 阴影规范

### 阴影层级
```typescript
// 浅阴影 - 卡片
.shadow({
  radius: 8,
  color: '#1A000000',
  offsetX: 0,
  offsetY: 2
})

// 中阴影 - 弹窗
.shadow({
  radius: 24,
  color: '#33000000',
  offsetX: 0,
  offsetY: 8
})

// 深阴影 - 浮层
.shadow({
  radius: 32,
  color: '#40000000',
  offsetX: 0,
  offsetY: 16
})
```

## 图标规范

### 尺寸
| 场景 | 尺寸 |
|------|------|
| 底部导航 | 24vp |
| 列表图标 | 24vp |
| 操作按钮 | 24vp |
| 空状态 | 64vp |
| 品牌图标 | 120vp |

### 使用
```typescript
Image($r('sys.symbol.ohos_filled_checkmark'))
  .width(24)
  .height(24)
  .fillColor('#007DFF')

// 使用系统符号
Image($r('sys.symbol.ohos_filled_settings'))
Image($r('sys.symbol.ohos_filled_home'))
Image($r('sys.symbol.ohos_filled_person'))
```

## 动效规范

### 时长规范
| 场景 | 时长 |
|------|------|
| 微交互 | 100-200ms |
| 状态切换 | 200-300ms |
| 页面转场 | 300-400ms |
| 复杂动画 | 400-600ms |

### 曲线规范
```typescript
// 标准曲线
curve: Curve.EaseOut      // 减速（默认）
curve: Curve.EaseIn        // 加速
curve: Curve.EaseInOut    // 先加速后减速
curve: Curve.FastOutSlowIn // 官方推荐

// 弹性曲线
curve: Curve.Spring        // 弹性
curve: Curve.FastOutLinearIn   // 快入
curve: Curve.LinearOutSlowIn   // 慢出
```

### 示例
```typescript
// 按钮点击反馈
Button('点击')
  .onClick(() => {
    animateTo({
      duration: 150,
      curve: Curve.FastOutSlowIn
    }, () => {
      this.scale = 0.95
    })
  })

// 页面元素入场
animateTo({
  duration: 300,
  curve: Curve.EaseOut,
  delay: 100
}, () => {
  this.opacity = 1
  this.offsetY = 0
})
```

## 响应式布局

### 断点系统
| 名称 | 宽度范围 | 列数 |
|------|----------|------|
| xs | < 320vp | 2 |
| sm | 320-600vp | 4 |
| md | 600-840vp | 8 |
| lg | > 840vp | 12 |

### 响应式示例
```typescript
GridRow({
  columns: { sm: 2, md: 4, lg: 8 },
  breakpoints: { value: ['320vp', '600vp', '840vp'] }
}) {
  GridCol({ span: { sm: 2, md: 2, lg: 4 } }) {
    ContentA()
  }
  GridCol({ span: { sm: 2, md: 2, lg: 4 } }) {
    ContentB()
  }
}
```

## 暗色模式

### 适配原则
1. 背景色使用语义化颜色
2. 不使用纯白(#FFFFFF)或纯黑(#000000)
3. 确保文字对比度 ≥ 4.5:1

```typescript
// 使用语义化颜色
.backgroundColor($r('sys.color.ohos_id_color_background'))
.color($r('sys.color.ohos_id_color_text_primary'))

// 浅色模式背景: #F1F3F5
// 深色模式背景: #0A1628
```

### 条件渲染
```typescript
import ConfigurationConstant from '@ohos.app.ability.ConfigurationConstant'

@State colorMode: ConfigurationConstant.ColorMode = ConfigurationConstant.ColorMode.COLOR_MODE_LIGHT

aboutToAppear() {
  // 获取当前配色模式
  let config = getContext(this).getConfiguration()
  this.colorMode = config.colorMode
}

// 条件样式
if (this.colorMode === ConfigurationConstant.ColorMode.COLOR_MODE_DARK) {
  // 深色样式
}
```

---

> 关联：SKILL.md 核心文件 | 标签：Design, 色彩, 字体, 布局, 动效
