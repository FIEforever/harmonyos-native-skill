# Canvas 2D / XComponent / ArkGraphics 图形渲染

## Canvas 2D 绑定模式

### 基础画布
```typescript
@Entry
@Component
struct CanvasDemo {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Column() {
      Canvas(this.context)
        .width(300)
        .height(300)
        .backgroundColor('#f0f0f0')
        .onReady(() => {
          this.draw()
        })
    }
  }

  private draw() {
    const ctx = this.context
    
    // 绘制矩形
    ctx.fillStyle = '#FF5733'
    ctx.fillRect(20, 20, 100, 80)
    
    // 绘制圆形
    ctx.beginPath()
    ctx.arc(200, 150, 50, 0, Math.PI * 2)
    ctx.fillStyle = '#33FF57'
    ctx.fill()
    
    // 绘制文字
    ctx.font = '24px sans-serif'
    ctx.fillStyle = '#333'
    ctx.fillText('Hello Canvas', 30, 250)
    
    // 绘制路径
    ctx.beginPath()
    ctx.moveTo(20, 200)
    ctx.lineTo(100, 150)
    ctx.lineTo(180, 200)
    ctx.strokeStyle = '#3357FF'
    ctx.lineWidth = 3
    ctx.stroke()
  }
}
```

### 渐变与阴影
```typescript
private drawGradient() {
  const ctx = this.context
  
  // 线性渐变
  const gradient = ctx.createLinearGradient(0, 0, 200, 0)
  gradient.addColorStop(0, '#FF0000')
  gradient.addColorStop(0.5, '#00FF00')
  gradient.addColorStop(1, '#0000FF')
  
  ctx.fillStyle = gradient
  ctx.fillRect(20, 20, 200, 100)
  
  // 阴影
  ctx.shadowColor = '#40000000'
  ctx.shadowBlur = 10
  ctx.shadowOffsetX = 5
  ctx.shadowOffsetY = 5
  
  ctx.fillStyle = '#FF5733'
  ctx.fillRect(50, 50, 100, 80)
}
```

### 图片渲染
```typescript
private drawImage() {
  const ctx = this.context
  const img = new ImageBitmap('/common/images/photo.png')
  
  img.onload = () => {
    ctx.drawImage(img, 0, 0, 200, 150)
    
    // 裁剪圆形
    ctx.save()
    ctx.beginPath()
    ctx.arc(100, 75, 50, 0, Math.PI * 2)
    ctx.clip()
    ctx.drawImage(img, 50, 25, 100, 100)
    ctx.restore()
  }
}
```

### 动画循环
```typescript
@Entry
@Component
struct AnimatedCanvas {
  private settings: RenderingContextSettings = new RenderingContextSettings(true)
  private context: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)
  private x: number = 0
  private animationId: number = 0

  aboutToAppear() {
    this.startAnimation()
  }

  aboutToDisappear() {
    cancelAnimationFrame(this.animationId)
  }

  startAnimation() {
    const animate = () => {
      this.context.clearRect(0, 0, 300, 300)
      
      // 移动方块
      this.x = (this.x + 2) % 280
      this.context.fillStyle = '#FF5733'
      this.context.fillRect(this.x, 130, 20, 20)
      
      this.animationId = requestAnimationFrame(animate)
    }
    animate()
  }

  build() {
    Canvas(this.context)
      .width(300)
      .height(300)
  }
}
```

## XComponent 离屏渲染

### 基础用法
```typescript
@Entry
@Component
struct XComponentDemo {
  @Local surfaceId: string = ''
  private eglState: drawing.GraphicContext | null = null

  build() {
    Column() {
      XComponent({ type: XComponentType.SURFACE, id: 'surface' })
        .width(300)
        .height(300)
        .onReady((event) => {
          this.surfaceId = event.surfaceId
          this.initGraphics()
        })
    }
  }

  private initGraphics() {
    // 使用 OH_Drawing 或 ArkGraphics 进行渲染
    // 参考官方图形库文档
  }
}
```

## AR Engine Kit

### AR 场景基础
```typescript
import arengine from '@kit.CoreVisionKit'

@Entry
@Component
struct ARScene {
  @Local arSession: arengine.ARSession | null = null

  async aboutToAppear() {
    // 检查设备支持
    if (!arengine.isArSupported()) {
      console.error('设备不支持AR')
      return
    }

    // 创建 AR 会话
    this.arSession = await arengine.createARSession(getContext(this))
    
    // 配置
    await this.arSession.configure({
      planeFindingMode: arengine.PlaneFindingMode.HORIZONTAL_AND_VERTICAL,
      lightEstimation: true
    })
  }

  build() {
    Column() {
      if (this.arSession) {
        // AR 场景内容
        Text('AR 已启动')
      } else {
        Text('正在初始化AR...')
      }
    }
  }
}
```

## 图形渲染选择指南

| 场景 | 方案 | 说明 |
|------|------|------|
| 简单 2D 图形 | Canvas 2D | 绑定模式，易上手 |
| 复杂 2D/3D | ArkGraphics | 高性能 |
| 视频/摄像头 | XComponent | Surface 渲染 |
| AR 应用 | AR Engine Kit | 空间识别 |

---

> 关联：SKILL.md 核心文件 | 标签：Canvas, XComponent, ArkGraphics, 渲染
