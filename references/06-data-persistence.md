# 数据持久化

## Preferences 轻量级键值存储

### 基础操作
```typescript
import preferences from '@ohos.data.preferences'

let context = getContext(this)

// 获取实例
let preferences = await preferences.getPreferences(context, 'myPrefs')

// 写入
await preferences.put('username', 'Tom')
await preferences.put('age', 25)
await preferences.put('isVip', true)
await preferences.flush()  // 同步到磁盘

// 读取
let username = preferences.getSync('username', 'default')
let age = preferences.getSync('age', 0)
let isVip = preferences.getSync('isVip', false)

// 删除
await preferences.delete('username')

// 清空
await preferences.clear()
```

### 支持的数据类型
```typescript
// 全部支持
preferences.put('string', 'value')
preferences.put('number', 123.45)
preferences.put('boolean', true)

// 数组 - 转为 JSON 字符串
preferences.put('array', JSON.stringify([1, 2, 3]))
let arr = JSON.parse(preferences.getSync('array', '[]'))

// 对象 - 转为 JSON 字符串
preferences.put('user', JSON.stringify({ name: 'Tom', age: 25 }))
let user = JSON.parse(preferences.getSync('user', '{}'))
```

### 监听变化
```typescript
import emitter from '@ohos.events.emitter'

// 定义事件
let event = {
  eventId: 1,
  priority: emitter.EventPriority.HIGH
}

// 订阅
emitter.on(event, (data) => {
  console.info(`Preferences 变化: ${JSON.stringify(data)}`)
})

// 模拟变化
emitter.emit({ eventId: 1 }, { data: { key: 'username', value: 'Jerry' } })

// 取消订阅
emitter.off(event)
```

## RelationalStore 关系型数据库

### 初始化
```typescript
import relationalStore from '@ohos.data.relationalStore'
import rdb from '@ohos.data.rdb'

const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'mydb.db',
  securityLevel: relationalStore.SecurityLevel.S1
}

// 获取 RDB
let store: relationalStore.RdbStore | null = null

async function getStore(): Promise<relationalStore.RdbStore> {
  if (store) return store
  
  store = await relationalStore.getRdbStore(getContext(this), STORE_CONFIG)
  
  // 创建表
  await store.executeSql(`
    CREATE TABLE IF NOT EXISTS users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      age INTEGER,
      email TEXT,
      created_at INTEGER DEFAULT (strftime('%s', 'now'))
    )
  `)
  
  return store
}
```

### 增删改查
```typescript
async function CRUD() {
  const store = await getStore()
  const valueBucket: relationalStore.ValuesBucket = {
    name: 'Tom',
    age: 25,
    email: 'tom@example.com'
  }
  
  // 插入
  const rowId = await store.insert('users', valueBucket)
  console.info(`插入ID: ${rowId}`)
  
  // 查询
  const predicates = new relationalStore.RdbPredicates('users')
  const result = await store.query(predicates, ['id', 'name', 'age'])
  
  // 遍历结果
  while (result.goToNextRow()) {
    console.info(`用户: ${result.getString(1)}, 年龄: ${result.getLong(2)}`)
  }
  result.close()
  
  // 更新
  const updatePredicates = new relationalStore.RdbPredicates('users')
  updatePredicates.equalTo('name', 'Tom')
  await store.update(updatePredicates, { age: 26 })
  
  // 删除
  const deletePredicates = new relationalStore.RdbPredicates('users')
  deletePredicates.equalTo('id', rowId)
  await store.delete(deletePredicates)
}
```

### 批量操作
```typescript
// 批量插入
async function batchInsert(users: User[]) {
  const store = await getStore()
  const batch: relationalStore.ValuesBucket[] = users.map(u => ({
    name: u.name,
    age: u.age,
    email: u.email
  }))
  
  const promises = batch.map(data => store.insert('users', data))
  await Promise.all(promises)
}

// 事务
async function transaction() {
  const store = await getStore()
  
  await store.executeSql('BEGIN TRANSACTION')
  
  try {
    await store.insert('users', { name: 'A', age: 20 })
    await store.insert('users', { name: 'B', age: 30 })
    await store.executeSql('COMMIT')
  } catch (err) {
    await store.executeSql('ROLLBACK')
    throw err
  }
}
```

## 文件存储

### 读写文本文件
```typescript
import fs from '@ohos.file.fs'

let context = getContext(this)

// 获取文件路径
let filePath = context.filesDir + '/data.txt'

// 写入
let file = fs.openSync(filePath, fs.OpenMode.WRITE_ONLY | fs.OpenMode.CREATE)
fs.writeSync(file.fd, 'Hello World')
fs.closeSync(file)

// 读取
let readFile = fs.openSync(filePath, fs.OpenMode.READ_ONLY)
let buffer = new ArrayBuffer(1024)
let len = fs.readSync(readFile.fd, buffer)
let content = String.fromCharCode.apply(null, new Uint8Array(buffer, 0, len))
fs.closeSync(readFile)
```

### JSON 文件存储
```typescript
interface UserData {
  name: string
  age: number
}

async function saveUser(user: UserData) {
  let filePath = getContext(this).filesDir + '/user.json'
  let file = fs.openSync(filePath, fs.OpenMode.WRITE_ONLY | fs.OpenMode.CREATE)
  fs.writeSync(file.fd, JSON.stringify(user))
  fs.closeSync(file)
}

async function loadUser(): Promise<UserData | null> {
  let filePath = getContext(this).filesDir + '/user.json'
  
  if (!fs.accessSync(filePath)) return null
  
  let file = fs.openSync(filePath, fs.OpenMode.READ_ONLY)
  let stat = fs.fstatSync(file.fd)
  let buffer = new ArrayBuffer(stat.size)
  fs.readSync(file.fd, buffer)
  fs.closeSync(file)
  
  return JSON.parse(String.fromCharCode(...new Uint8Array(buffer)))
}
```

## 分布式数据服务（跨设备）

### 创建分布式表
```typescript
import distributedKVStore from '@ohos.distributed.kvstore'

const KV_MANAGER_CONFIG = {
  context: getContext(this),
  bundleName: 'com.example.myapp'
}

const kvManager = distributedKVStore.createKVManager(KV_MANAGER_CONFIG)

kvManager.getKVStore('storeId', (err, store) => {
  if (err) {
    console.error(`获取失败: ${err.message}`)
    return
  }
  
  // 写入
  store.put('key1', 'value1', (err) => {
    if (err) console.error('写入失败')
  })
  
  // 同步到其他设备
  store.setSyncRange(['key1'], ['deviceId1'])
})
```

## 数据存储选择指南

| 场景 | 方案 | 说明 |
|------|------|------|
| 用户偏好设置 | Preferences | 轻量、快速 |
| 结构化业务数据 | RelationalStore | SQL 查询、事务 |
| 配置文件 | JSON 文件 | 灵活、易读 |
| 临时缓存 | AppStorage | 内存级别 |
| 跨设备同步 | Distributed KVStore | 分布式同步 |

---

> 关联：SKILL.md 核心文件 | 标签：Preferences, RelationalStore, 文件, 持久化
