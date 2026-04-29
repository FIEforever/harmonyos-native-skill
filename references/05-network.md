# HTTP 网络请求与 WebSocket

## @kit.NetworkKit HTTP (推荐)

### 基础 GET 请求
```typescript
import http from '@ohos.net.http'

let httpRequest = http.createHttp()

httpRequest.request(
  'https://api.example.com/users',
  {
    method: http.RequestMethod.GET,
    header: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token'
    },
    connectTimeout: 60000,   // 60秒
    readTimeout: 60000
  }
).then((resp: http.HttpResponse) => {
  // 状态码
  let code = resp.responseCode
  console.info(`状态码: ${code}`)
  
  // 响应体
  let data = resp.result as string
  console.info(`响应: ${data}`)
  
  // 响应头
  let headers = resp.header
  console.info(`Headers: ${JSON.stringify(headers)}`)
  
}).catch((err: Error) => {
  console.error(`请求失败: ${err.message}`)
  
}).finally(() => {
  httpRequest.destroy()  // ⚠️ 必须释放
})
```

### POST 请求
```typescript
let httpRequest = http.createHttp()

let postData = {
  name: 'Tom',
  age: 25
}

httpRequest.request(
  'https://api.example.com/users',
  {
    method: http.RequestMethod.POST,
    header: {
      'Content-Type': 'application/json'
    },
    extraData: JSON.stringify(postData)  // 请求体
  }
).then((resp) => {
  if (resp.responseCode === http.ResponseCode.OK) {
    let result = JSON.parse(resp.result as string)
    console.info(`创建成功: ${result.id}`)
  }
}).finally(() => {
  httpRequest.destroy()
})
```

### 下载文件
```typescript
import fs from '@ohos.file.fs'

let httpRequest = http.createHttp()
let file = fs.openSync('/path/to/save.pdf', fs.OpenMode.WRITE_ONLY)

httpRequest.download('https://example.com/file.pdf',
  (err, data) => {
    if (err) {
      console.error(`下载失败: ${err.message}`)
      return
    }
    // data 是临时文件路径
    fs.copyFileSync(data.tempFilePath, '/path/to/save.pdf')
    httpRequest.destroy()
  }
)
```

### 请求取消
```typescript
let httpRequest = http.createHttp()
let requestTask = httpRequest.request(url, options)

setTimeout(() => {
  // 取消请求
  requestTask.then(task => task.destroy())
}, 5000)
```

## rcp 模块（推荐 RESTful）

### 基础配置
```typescript
import rcp from '@kit.NetworkKit'

// 创建客户端
const client = rcp.createSession({
  baseAddress: 'https://api.example.com',
  timeout: 30000
})

// GET
const users = await client.get('/users', {
  headers: { 'Accept': 'application/json' }
})

// POST
const newUser = await client.post('/users', {
  headers: { 'Content-Type': 'application/json' },
  body: rcp.jsonBody({ name: 'Tom', age: 25 })
})

// PUT
await client.put('/users/123', {
  body: rcp.jsonBody({ name: 'Jerry' })
})

// DELETE
await client.delete('/users/123')
```

### 响应处理
```typescript
const response = await client.get('/users')

// 检查状态
if (response.status === 200) {
  const data = response.toJSON()  // 自动解析 JSON
  console.info(`用户数: ${data.length}`)
}

// 错误处理
if (response.status === 404) {
  console.info('资源不存在')
} else if (response.status >= 400) {
  console.error(`请求错误: ${response.status}`)
}
```

## WebSocket

### 客户端连接
```typescript
import webSocket from '@ohos.net.webSocket'

let ws = webSocket.createWebSocket()

ws.on('open', (err, value) => {
  if (err) {
    console.error(`连接失败: ${JSON.stringify(err)}`)
    return
  }
  console.info('WebSocket 已连接')
  
  // 发送消息
  ws.send('Hello Server')
})

ws.on('message', (err, value) => {
  if (err) {
    console.error(`接收失败: ${JSON.stringify(err)}`)
    return
  }
  console.info(`收到消息: ${value}`)
})

ws.on('close', (err, value) => {
  console.info(`连接关闭: code=${value.code}, reason=${value.reason}`)
})

ws.on('error', (err) => {
  console.error(`错误: ${JSON.stringify(err)}`)
})

// 建立连接
ws.connect('wss://echo.websocket.org').then(() => {
  console.info('连接中...')
}).catch((err) => {
  console.error(`连接失败: ${err.message}`)
})
```

### 心跳保活
```typescript
// 定期发送心跳
let heartbeatTimer = setInterval(() => {
  if (ws.getState() === webSocket.WebSocketState.CONNECTED) {
    ws.send(JSON.stringify({ type: 'ping' }))
  }
}, 30000)

// 清理
clearInterval(heartbeatTimer)
ws.close()
```

## 证书与安全

### 禁用证书验证（仅开发环境）
```typescript
// ❌ 不推荐，仅开发测试用
http.createHttp({ 
  caPath: undefined  // 使用系统默认 CA
})

// ⚠️ 切勿在生产环境禁用证书验证
```

### 客户端证书
```typescript
// 使用 P12 证书
let httpRequest = http.createHttp()

// 复杂场景使用 rcp
const client = rcp.createSession({
  baseAddress: 'https://secure.example.com',
  clientCertificate: {
    certPath: '/path/to/client.p12',
    certType: 'p12',
    // 如需要密码
    // key: 'password'
  }
})
```

## HTTP 代理配置

```typescript
import http from '@ohos.net.http'

let httpRequest = http.createHttp()

httpRequest.request(
  'https://api.example.com',
  {
    method: http.RequestMethod.GET,
    // 代理配置
    usingProxy: true,
    header: {}
  }
)
```

## 常见错误处理

### 网络超时
```typescript
async function requestWithRetry(url: string, retries = 3): Promise<string> {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await http.request(url, {
        method: http.RequestMethod.GET,
        connectTimeout: 10000,
        readTimeout: 10000
      })
      return response.result as string
    } catch (err) {
      if (i === retries - 1) throw err
      await new Promise(r => setTimeout(r, 1000 * (i + 1)))  // 指数退避
    }
  }
  throw new Error('请求失败')
}
```

### 错误码对照
| HTTP Code | 含义 | 处理 |
|-----------|------|------|
| 200 | 成功 | 解析数据 |
| 201 | 创建成功 | 获取新资源 ID |
| 400 | 请求错误 | 检查参数 |
| 401 | 未认证 | 刷新 Token |
| 403 | 无权限 | 提示用户 |
| 404 | 资源不存在 | 友好提示 |
| 500 | 服务器错误 | 重试或反馈 |

---

> 关联：SKILL.md 核心文件 | 标签：HTTP, WebSocket, 网络请求, @kit.NetworkKit
