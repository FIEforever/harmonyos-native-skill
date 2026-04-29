# ArkTS 核心语法与类型系统

## 严格类型系统

### 类型推断
```typescript
// ArkTS 要求显式类型注解
let name: string = 'Tom'
let age: number = 25
let isActive: boolean = true
let arr: number[] = [1, 2, 3]
let tuple: [string, number] = ['name', 18]
```

### 联合类型与交叉类型
```typescript
type Status = 'pending' | 'success' | 'error'
type Readonly<T> = { readonly [P in keyof T]: T[P] }
```

### 类型守卫
```typescript
function isString(val: string | number): val is string {
  return typeof val === 'string'
}
```

## 接口与类

### 接口
```typescript
interface Person {
  name: string
  age?: number  // 可选属性
  readonly id: string  // 只读属性
}

interface Config {
  [key: string]: string  // 索引签名
}
```

### 类与继承
```typescript
class Animal {
  name: string
  constructor(name: string) {
    this.name = name
  }
  speak(): void {
    console.log('sound')
  }
}

class Dog extends Animal {
  breed: string
  constructor(name: string, breed: string) {
    super(name)
    this.breed = breed
  }
  override speak(): void {  // override 关键字
    console.log('bark')
  }
}
```

## 泛型

### 基础泛型
```typescript
function identity<T>(arg: T): T {
  return arg
}

let output = identity<string>('hello')
let num = identity(42)  // 类型推断
```

### 约束泛型
```typescript
interface Lengthwise {
  length: number
}

function logLength<T extends Lengthwise>(arg: T): void {
  console.log(arg.length)
}
```

### 多约束
```typescript
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 }
}
```

## Sendable 类（跨线程通信）

### 定义 Sendable 类
```typescript
import { Sendable, Trait } from '@kit.ArkData'

// 使用装饰器
@Sendable
class UserData {
  name: string = ''
  age: number = 0
  
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }
}

// 不使用装饰器 - 需实现 Trait 接口
class Product implements Trait {
  id: string = ''
  price: number = 0
  
  // 必须实现这些方法
  setFrom(source: Product): void {
    this.id = source.id
    this.price = source.price
  }
  equalTo(other: Product): boolean {
    return this.id === other.id
  }
}
```

### 使用场景
```typescript
import taskpool from '@kit.ArkTS'

// TaskPool 任务传参
@Sendable
class TaskParams {
  url: string = ''
  page: number = 0
}

@Concurrent
function fetchData(params: TaskParams): string {
  // 只能在 Sendable 类中访问
  return `Data from ${params.url}`
}

let params = new TaskParams()
params.url = 'https://api.example.com'
params.page = 1

taskpool.execute(fetchData, params).then(result => {
  console.log(result)
})
```

## 异步编程

### Promise 链式调用
```typescript
function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function fetchUser(): Promise<User> {
  await delay(1000)
  return { id: '1', name: 'Tom' }
}

fetchUser()
  .then(user => console.log(user.name))
  .catch(err => console.error(err))
```

### async/await
```typescript
async function loadData(): Promise<Data[]> {
  try {
    const res = await fetch('https://api.example.com/data')
    return res.json()
  } catch (err) {
    console.error('加载失败:', err)
    return []
  }
}
```

### 并行执行
```typescript
async function parallelFetch() {
  const [users, posts] = await Promise.all([
    fetch('/users').then(r => r.json()),
    fetch('/posts').then(r => r.json())
  ])
  return { users, posts }
}
```

## 常见错误处理

### 类型错误
```typescript
// ❌ 错误：any 类型
let data: any = getData()

// ✅ 正确：使用 unknown
let data: unknown = getData()
if (typeof data === 'string') {
  console.log(data.toUpperCase())
}
```

### 空值检查
```typescript
// ❌ 错误：可能为 null
let name = user.name.toString()

// ✅ 正确：可选链 + 空值合并
let name = user?.name?.toString() ?? 'Unknown'
```

### 异步错误处理
```typescript
// ✅ 标准模式
async function safeRequest() {
  try {
    const result = await fetchData()
    return result
  } catch (err) {
    // 明确错误类型
    if (err instanceof NetworkError) {
      // 处理网络错误
    } else if (err instanceof ServerError) {
      // 处理服务器错误
    }
    throw err  // 重新抛出
  }
}
```

## 装饰器

### 基础装饰器
```typescript
function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with`, args)
    return original.apply(this, args)
  }
  return descriptor
}

class MyClass {
  @log
  myMethod(arg: string): string {
    return `Hello ${arg}`
  }
}
```

## 模块导出

```typescript
// 导出
export { identity, Person }
export default class DefaultClass { }

// 导入
import MyClass, { identity } from './module'
```

---

> 关联：SKILL.md 核心文件 | 标签：ArkTS, 类型系统, 异步
