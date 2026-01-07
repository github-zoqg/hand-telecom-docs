---
sidebar_position: 3
sidebar_label: "常用功能"
---

# Fiddler 常用功能

## 界面概览

Fiddler 的主界面分为几个关键区域：

```
┌─────────────────────────────────────────────────────────────┐
│ 菜单栏 (Menu Bar)                                           │
├─────────────────────────────────────────────────────────────┤
│ 工具栏 (Toolbar)                                           │
├─────────────────────────────────────────────────────────────┤
│ 会话列表 (Session List)        │ 检查器 (Inspectors)        │
│                              │                            │
│ ├──#── URL ──── 状态码 ────┐  │ ├── 请求 (Request)        │
│ │ 1  GET /api/users      200 │  │ ├── Headers              │
│ │ 2  POST /api/login     201 │  │ ├── WebForms             │
│ │ 3  GET /static/app.js  304 │  │ ├── JSON                 │
│ └─────────────────────────┘  │ ├── 响应 (Response)       │
│                              │ │ ├── Headers              │
│                              │ │ ├── TextView             │
│                              │ │ ├── JSON                 │
│                              │ └─────────────────────────┘
├─────────────────────────────────────────────────────────────┤
│ 命令栏 (Command Bar)                                       │
├─────────────────────────────────────────────────────────────┤
│ 状态栏 (Status Bar)                                        │
└─────────────────────────────────────────────────────────────┘
```

## 请求捕获和过滤

### 1. 基本捕获

Fiddler 启动后会自动捕获所有 HTTP/HTTPS 请求。

**控制捕获**：

- **开始/停止捕获**: 点击工具栏的 `Capturing` 按钮
- **快捷键**: `F12`
- **菜单**: `File` → `Capture Traffic`

### 2. 请求过滤

当请求过多时，可以使用过滤功能只显示需要的请求。

#### 使用过滤器

1. 点击 `Filters` 标签页
2. 配置过滤条件：
   - **域名过滤**: 只显示特定域名的请求
   - **状态码过滤**: 只显示特定状态码的请求
   - **进程过滤**: 只显示特定进程的请求
   - **内容类型过滤**: 只显示特定类型的请求

#### 常用过滤器示例

```text
# 只显示 API 请求
Host: api.example.com

# 只显示失败的请求
Status: 4xx, 5xx

# 只显示 POST 请求
Method: POST

# 排除静态资源
!Host: *.css, *.js, *.png, *.jpg
```

#### 快速过滤

在会话列表中点击列标题进行快速排序：

- **URL 列**: 按 URL 字母顺序排序
- **Status 列**: 按状态码排序
- **Process 列**: 按进程名称排序

### 3. 请求搜索

#### 文本搜索

1. 使用 `Ctrl+F` 打开搜索框
2. 输入搜索关键词
3. 选择搜索范围：
   - **Request headers**: 请求头
   - **Request body**: 请求体
   - **Response headers**: 响应头
   - **Response body**: 响应体

#### 正则表达式搜索

勾选 "Regex" 选项可以使用正则表达式：

```regex
# 搜索所有 JSON 响应
Content-Type: application/json

# 搜索所有错误响应
HTTP/[0-9.]+ [45][0-9]{2}
```

## 请求分析和检查

### 1. 检查器面板

检查器面板分为请求和响应两部分，提供了多种查看模式。

#### 请求检查器

**Headers 标签**

- 显示完整的 HTTP 请求头
- 支持编辑和复制
- 高亮显示重要字段

**WebForms 标签**

- 解析表单数据
- 显示键值对
- 支持编辑参数

**JSON 标签**

- 格式化显示 JSON 数据
- 语法高亮
- 支持折叠/展开

**TextView 标签**

- 原始文本查看
- 支持语法高亮
- 适合查看纯文本响应

**HexView 标签**

- 十六进制查看器
- 适合查看二进制数据
- 显示 ASCII 和十六进制

#### 响应检查器

**Headers 标签**

- 显示 HTTP 响应头
- 包含状态码、内容类型等
- 支持缓存策略分析

**JSON/XML 标签**

- 自动格式化 JSON/XML 数据
- 树形结构显示
- 支持数据搜索

**ImageView 标签**

- 直接显示图片
- 支持常见图片格式
- 显示图片尺寸和大小

### 2. 性能分析

#### 时间统计

Fiddler 会为每个请求记录详细的时间信息：

```
请求时间线：
DNS Lookup: 15ms
TCP Connect: 25ms
HTTPS Handshake: 120ms
Server Processing: 200ms
Content Download: 50ms
Total Time: 410ms
```

#### 性能视图

1. 选择请求
2. 点击 `Statistics` 标签
3. 查看详细的性能数据：
   - **请求大小**: 请求头和请求体大小
   - **响应大小**: 响应头和响应体大小
   - **时间分解**: 各阶段耗时
   - **带宽使用**: 上传/下载速度

## 请求修改和重放

### 1. 请求重放

#### 单个请求重放

1. 在会话列表中选择要重放的请求
2. 按 `R` 键或右键选择 `Replay`
3. 请求将被重新发送

#### 批量请求重放

1. 选择多个请求（按住 Ctrl 或 Shift）
2. 右键选择 `Replay` → `Replay Selected Sessions`
3. 所有选中的请求将被依次重放

### 2. 请求编辑

#### 编辑请求

1. 选择要编辑的请求
2. 切换到 `Composer` 标签
3. 修改请求内容：
   - **Method**: HTTP 方法
   - **URL**: 请求地址
   - **Headers**: 请求头
   - **Body**: 请求体
4. 点击 `Execute` 发送修改后的请求

#### 创建新请求

1. 切换到 `Composer` 标签
2. 选择 HTTP 方法
3. 输入完整的 URL
4. 添加请求头和请求体
5. 点击 `Execute`

### 3. 断点调试

#### 设置断点

1. **全局断点**:

   - 菜单：`Rules` → `Automatic Breakpoints`
   - 选择 `Before Requests` 或 `After Responses`

2. **条件断点**:

   ```javascript
   // 在 FiddlerScript 中设置
   if (oSession.uriContains("api")) {
     oSession["x-breakrequest"] = "breaking on api calls";
   }
   ```

3. **快速断点**:
   - 命令栏输入：`bpu api.example.com` (请求断点)
   - 命令栏输入：`bpafter api.example.com` (响应断点)

#### 断点操作

当触发断点时：

- **继续执行**: 按 `F5` 或点击 `Run to Completion`
- **修改请求**: 在检查器中编辑请求内容
- **终止请求**: 按 `Shift+F5` 或点击 `Abort`

### 4. 自动响应

#### 创建自动响应规则

1. 选择要自动响应的请求
2. 右键选择 `Save` → `Response`
3. 保存响应到文件
4. 切换到 `AutoResponder` 标签
5. 点击 `Add Rule`
6. 配置规则：
   - **Match**: 匹配条件（URL 或正则表达式）
   - **Action**: 响应文件或状态码

#### 常用自动响应场景

```text
# 返回模拟数据
Match: http://api.example.com/users
Action: C:\mock\users.json

# 返回 404 错误
Match: http://api.example.com/old-endpoint
Action: *404:File Not Found

# 延迟响应
Match: http://api.example.com/slow
Action: *200:OK Latency:3000
```

## 会话管理

### 1. 会话操作

#### 会话选择

- **单个选择**: 单击选择
- **多个选择**: `Ctrl + 单击` 或 `Shift + 单击`
- **全选**: `Ctrl + A`

#### 会话删除

- **删除单个**: 选中后按 `Delete`
- **删除所有**: `Ctrl + X`
- **删除非选中**: `Shift + Delete`
- **清空会话**: `Edit` → `Remove` → `All Sessions`

### 2. 会话保存

#### 保存会话

1. 选择要保存的会话
2. `File` → `Save` → `Selected Sessions`
3. 选择保存位置和文件名
4. 保存为 `.saz` 格式

#### 加载会话

1. `File` → `Load`
2. 选择之前保存的 `.saz` 文件
3. 会话将加载到当前会话列表

### 3. 会话比较

#### 比较两个会话

1. 选择两个会话（按住 Ctrl）
2. 右键选择 `Compare`
3. 在比较窗口中查看差异
4. 支持请求头、响应头和响应体比较

#### 比较功能

- **高亮差异**: 用颜色标记不同之处
- **并排显示**: 两个会话并排显示
- **导出报告**: 将比较结果导出为报告

## 命令栏使用

### 1. 常用命令

#### 过滤命令

```text
# 只显示特定域名的请求
bpu api.example.com

# 排除特定域名的请求
bpu =api.example.com

# 只显示特定状态码的请求
status 200

# 只显示特定方法的请求
method POST
```

#### 操作命令

```text
# 清空会话
cls

# 开始/停止捕获
capture

# 切换到特定标签页
select json
```

#### 搜索命令

```text
# 搜索包含特定文本的请求
find text

# 搜索响应中包含特定文本的请求
findresponse text

# 搜索请求中包含特定文本的请求
findrequest text
```

### 2. 命令组合

```text
# 组合使用多个命令
bpu api.example.com && status 200

# 使用正则表达式搜索
find regex /api/v[0-9]/

# 显示特定进程的请求
process chrome.exe
```

## 实用技巧

### 1. 快捷键

| 功能          | 快捷键     |
| ------------- | ---------- |
| 开始/停止捕获 | `F12`      |
| 清空会话      | `Ctrl + X` |
| 查找          | `Ctrl + F` |
| 重放请求      | `R`        |
| 删除请求      | `Delete`   |
| 全选          | `Ctrl + A` |
| 复制 URL      | `Ctrl + U` |

### 2. 右键菜单

选中会话后右键可以访问常用功能：

- **Copy**: 复制各种格式的数据
- **Save**: 保存请求/响应
- **Replay**: 重放请求
- **Unlock**: 解锁会话（防止被过滤）
- **Filter**: 应用过滤条件

### 3. 自定义列

可以添加自定义列来显示额外信息：

1. 右键点击列标题
2. 选择 `Customize Columns`
3. 添加需要的列
4. 设置列宽和位置

## 下一步

掌握了常用功能后，你可以：

1. 学习 [高级用法](./advanced-usage.md)
2. 了解 [实战案例](./examples.md)
3. 探索 [脚本编程](./scripting.md)
