# WebSSH2 项目 - 快速参考指南

## 核心路径图

### 数据流入口
   ```
   用户请求 → HTTP 路由 → WebSocket 连接 → Socket 事件 → 服务层 → SSH/Telnet 连接
   ```

### 关键文件映射

#### 从请求到 SSH 连接的完整路径
   ```
   1. GET /ssh?host=example.com
      └─> app/routes/routes-v2.ts (HTTP 路由)
          ├─> app/connectionHandler.ts (读取 HTML 模板)
          └─> 返回 client.htm

   2. 浏览器加载页面，建立 WebSocket 连接

   3. Socket 连接建立
      └─> app/socket-v2.ts (socket.io 监听)
          └─> app/socket/adapters/service-socket-adapter.ts (主适配器)
              ├─> ServiceSocketAuthentication (处理认证)
              │   └─> app/auth/auth-pipeline.ts (认证管道)
              │       └─> app/services/ssh/ssh-service.ts (建立 SSH 连接)
              └─> ServiceSocketTerminal (处理终端)
                  └─> app/services/terminal/terminal-service.ts
   ```

## 文件分类速查

### 核心启动和初始化
- `index.ts` - 项目入口
- `app/app.ts` - 应用创建
- `app/server.ts` - HTTP 服务器
- `app/io.ts` - Socket.IO 配置
- `app/socket-v2.ts` - Socket 初始化

### 认证相关
- `app/auth/auth-pipeline.ts` - 认证流程
- `app/auth/auth-utils.ts` - 认证工具
- `app/auth/auth-method-policy.ts` - 认证方法策略
- `app/auth/providers/*.ts` - 认证方法提供者

### HTTP 路由
- `app/routes/routes-v2.ts` - 路由定义
- `app/routes/handlers/ssh-handler.ts` - SSH 路由处理
- `app/routes/handlers/ssh-config-handler.ts` - 配置处理
- `app/routes/adapters/express-adapter.ts` - Express 适配器
- `app/connectionHandler.ts` - 连接处理 (HTML 渲染)

### WebSocket 事件处理
- `app/socket/adapters/service-socket-adapter.ts` - 主适配器
- `app/socket/adapters/service-socket-authentication.ts` - 认证事件
- `app/socket/adapters/service-socket-terminal.ts` - 终端事件
- `app/socket/adapters/service-socket-sftp.ts` - SFTP 事件
- `app/socket/adapters/service-socket-control.ts` - 控制事件
- `app/socket/adapters/service-socket-prompt.ts` - 提示事件
- `app/constants/socket-events.ts` - 事件常量

### 服务层
- `app/services/ssh/ssh-service.ts` - SSH 服务主类
- `app/services/ssh/connection-pool.ts` - 连接池
- `app/services/terminal/terminal-service.ts` - 终端服务
- `app/services/sftp/sftp-service.ts` - SFTP 服务
- `app/services/auth/auth-service.ts` - 认证服务
- `app/services/host-key/host-key-service.ts` - 主机密钥服务
- `app/services/container.ts` - DI 容器
- `app/services/setup.ts` - 服务初始化

### 配置和常量
- `app/config/config-loader.ts` - 配置加载
- `app/config/default-config.ts` - 默认配置
- `app/constants/index.ts` - 常量定义
- `app/constants/socket-events.ts` - Socket 事件
- `app/constants/validation.ts` - 验证消息

### 前端相关
- `app/client-path.ts` - 获取前端路径
- `app/connectionHandler.ts` - HTML 模板处理
- `app/utils/html-transformer.ts` - HTML 配置注入

### 中间件
- `app/middleware.ts` - 中间件配置
- `app/middleware/auth.middleware.ts` - 认证中间件
- `app/middleware/session.middleware.ts` - 会话中间件
- `app/middleware/csrf.middleware.ts` - CSRF 中间件

### 日志和错误
- `app/logger.ts` - 日志系统
- `app/logging/` - 日志模块
- `app/errors.ts` - 错误处理
- `app/errors/webssh2-error.ts` - 自定义错误

### 类型定义
- `app/types/contracts/v1/socket.ts` - Socket 消息类型
- `app/types/contracts/v1/sftp.ts` - SFTP 消息类型
- `app/types/contracts/v1/http.ts` - HTTP 消息类型
- `app/types/config.ts` - 配置类型

## 重要的导出类/函数

### 从 `app/app.ts`
   ```typescript
   createAppAsync(config)     // 创建 Express 应用
   initializeServerAsync()    // 初始化整个服务器
   ```

### 从 `app/services/ssh/ssh-service.ts`
   ```typescript
   class SSHServiceImpl {
     connect(config): Promise<Result<SSHConnection>>
     shell(sessionId): Promise<Result<Stream>>
     exec(sessionId, command): Promise<Result<ExecResult>>
     close(sessionId): Promise<Result<void>>
   }
   ```

### 从 `app/socket/adapters/service-socket-adapter.ts`
   ```typescript
   class ServiceSocketAdapter {
     // 处理所有 Socket 事件
     setupEventHandlers(): void
   }
   ```

### 从 `app/auth/auth-pipeline.ts`
   ```typescript
   class UnifiedAuthPipeline {
     isAuthenticated(): boolean
     getCredentials(): Credentials | null
     getAuthMethod(): AuthMethod | null
     requiresAuthRequest(): boolean
   }
   ```

## Socket 事件快速索引

### 认证流程事件
   ```
   客户端 → 服务器
     ↓ 'authenticate' (AUTH)
         + 凭证: { username, password/privateKey, ... }

   服务器 → 客户端
     ↓ 'authentication' (AUTH_SUCCESS)
         + 成功消息
     ↓ 'connection-error' (CONNECTION_ERROR)
         + 错误详情
     ↓ 'prompt' (PROMPT)
         + 提示信息 (密码、键盘交互等)
   ```

### 终端数据流事件
   ```
   客户端 → 服务器
     ↓ 'terminal' (TERMINAL)
         + 终端配置: { rows, cols, term, env }
     ↓ 'resize' (RESIZE)
         + 尺寸: { rows, cols }
     ↓ 'data' (DATA)
         + 终端数据

   服务器 → 客户端
     ↓ 'data' (SSH_DATA)
         + 来自 SSH 的数据
     ↓ 'ssherror' (SSH_ERROR)
         + 错误消息
     ↓ 'ready' (SSH_READY)
         + 连接就绪
   ```

### SFTP 事件
   ```
   客户端 → 服务器
     'sftp-list', 'sftp-stat', 'sftp-mkdir', 'sftp-delete'
     'sftp-upload-start', 'sftp-upload-chunk', 'sftp-upload-cancel'
     'sftp-download-start', 'sftp-download-cancel'

   服务器 → 客户端
     'sftp-directory', 'sftp-stat-result', 'sftp-operation-result'
     'sftp-upload-ready', 'sftp-upload-ack'
     'sftp-download-ready', 'sftp-download-chunk'
     'sftp-progress', 'sftp-complete', 'sftp-error'
   ```

## URL 路由参数

### 路由格式
   ```
   GET /ssh                          # 登录页
   GET /ssh/:host                    # 直接连接主机
   GET /ssh/:host/:port              # 连接指定端口
   GET /host/:host                   # 自动连接
   GET /host/:host/:port             # 自动连接到指定端口
   POST /ssh                         # POST 认证
   GET /ssh/config                   # 获取配置
   ```

### 查询字符串参数
   ```
   ?host=hostname         # 主机地址
   ?port=22              # SSH 端口
   ?sshterm=xterm        # 终端类型
   ```

### POST 请求体参数
   ```json
   {
     "username": "admin",
     "password": "secret",
     "privateKey": "-----BEGIN RSA...",
     "passphrase": "key-password",
     "host": "localhost",
     "port": "22",
     "term": "xterm-256color"
   }
   ```

## 关键数据结构

### 认证凭证
   ```typescript
   interface AuthCredentials {
     username: string
     password?: string
     privateKey?: string
     passphrase?: string
   }
   ```

### SSH 连接配置
   ```typescript
   interface SSHConfig {
     host: string
     port: number
     username: string
     password?: string
     privateKey?: string
     passphrase?: string
     readyTimeout?: number
     keepaliveInterval?: number
     keepaliveCountMax?: number
     tryKeyboard?: boolean
   }
   ```

### 终端配置
   ```typescript
   interface TerminalSettings {
     cols: number
     rows: number
     term: string
     env?: Record<string, string>
   }
   ```

### 终端尺寸
   ```typescript
   interface Dimensions {
     rows: number
     cols: number
   }
   ```

## 常用环境变量

   ```bash
   DEBUG=webssh2:*              # 启用调试日志
   NODE_ENV=development        # 开发环境
   WEBSSH2_SERVER_PORT=2222    # 服务器端口
   WEBSSH2_SERVER_IP=0.0.0.0   # 监听 IP
   SSH_KEY_PATH=/keys/         # SSH 密钥路径
   SESSION_SECRET=...          # 会话密钥
   ```

## 项目编译和运行

   ```bash
   # 编译
   npm run build

   # 运行
   npm start

   # 开发模式 (热重载)
   npm run dev

   # 测试
   npm test
   npm run test:watch
   npm run test:e2e

   # 检查
   npm run check
   npm run lint
   npm run typecheck
   ```

## 依赖项说明

### 核心库
- `express` - HTTP 框架
- `socket.io` - WebSocket 库
- `ssh2` - SSH 客户端
- `better-sqlite3` - 主机密钥存储
- `express-session` - 会话管理
- `body-parser` - 请求体解析

### 前端相关
- `webssh2_client` - 前端客户端库 (包含 xterm.js)

### 验证和工具
- `zod` - 数据验证
- `validator` - 字符串验证
- `jsmasker` - 数据脱敏
- `debug` - 调试日志
- `basic-auth` - HTTP Basic 认证
