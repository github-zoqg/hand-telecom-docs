---
sidebar_position: 4
sidebar_label: "高级用法"
---

# Fiddler 高级用法

## FiddlerScript 编程

### 1. FiddlerScript 简介

FiddlerScript 是基于 JScript.NET 的脚本语言，允许用户自定义 Fiddler 的行为。通过编写脚本，可以实现复杂的请求处理逻辑。

#### 访问脚本编辑器

1. 菜单：`Rules` → `Customize Rules`
2. 快捷键：`Ctrl + R`
3. 脚本文件位置：`Fiddler2\Scripts\CustomRules.js`

#### 脚本基础结构

```javascript
// FiddlerScript 主要方法

// 请求发送前执行
static function OnBeforeRequest(oSession: Session) {
    // 请求处理逻辑
}

// 响应接收后执行
static function OnBeforeResponse(oSession: Session) {
    // 响应处理逻辑
}

// 主函数执行
static function Main() {
    // 初始化逻辑
}
```

### 2. 常用脚本示例

#### 请求头修改

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 添加自定义请求头
    oSession.oRequest.headers["X-Custom-Header"] = "Fiddler-Modified";

    // 修改 User-Agent
    if (oSession.uriContains("mobile")) {
        oSession.oRequest["User-Agent"] = "Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)";
    }

    // 移除特定请求头
    oSession.oRequest.headers.Remove("Referer");
}
```

#### 响应内容修改

```javascript
static function OnBeforeResponse(oSession: Session) {
    // 只处理 JSON 响应
    if (oSession.oResponse.headers.ExistsAndContains("Content-Type", "application/json")) {
        var responseBody = oSession.GetResponseBodyAsString();

        try {
            // 解析 JSON
            var jsonObj = Fiddler.WebFormats.JSON.JsonDecode(responseBody);

            // 修改 JSON 数据
            if (jsonObj.JSONObject) {
                jsonObj.JSONObject["modified"] = "true";
                jsonObj.JSONObject["timestamp"] = new Date().toISOString();
            }

            // 重新编码响应
            var modifiedBody = Fiddler.WebFormats.JSON.JsonEncode(jsonObj.JSONObject);
            oSession.utilSetResponseBody(modifiedBody);
        } catch (e) {
            FiddlerApplication.Log.LogString("JSON 解析错误: " + e.toString());
        }
    }
}
```

#### 条件断点

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 对特定 API 设置断点
    if (oSession.uriContains("/api/v1/users")) {
        oSession["x-breakrequest"] = "user API breakpoint";
    }

    // 对特定请求方法设置断点
    if (oSession.HTTPMethodIs("DELETE")) {
        oSession["x-breakrequest"] = "DELETE request breakpoint";
    }
}
```

#### 自动重定向

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 将旧 API 重定向到新 API
    if (oSession.uriContains("/api/v1/")) {
        oSession.url = oSession.url.replace("/api/v1/", "/api/v2/");
        FiddlerApplication.Log.LogString("重定向: " + oSession.url);
    }
}
```

#### 模拟网络延迟

```javascript
static function OnBeforeResponse(oSession: Session) {
    // 对特定接口添加延迟
    if (oSession.uriContains("/api/slow")) {
        oSession["response-trickle-delay"] = "3000"; // 3秒延迟
    }

    // 随机延迟（模拟不稳定网络）
    if (oSession.uriContains("/api/unstable")) {
        var delay = Math.random() * 2000 + 500; // 500-2500ms
        oSession["response-trickle-delay"] = delay.toString();
    }
}
```

### 3. 高级脚本技巧

#### 环境变量使用

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 根据环境变量修改请求
    var environment = Environment.GetEnvironmentVariable("FIDDLER_ENV");

    if (environment === "development") {
        oSession.url = oSession.url.replace("prod-api", "dev-api");
    } else if (environment === "testing") {
        oSession.url = oSession.url.replace("prod-api", "test-api");
    }
}
```

#### 外部文件读写

```javascript
// 读取配置文件
function loadConfig() {
  var configPath = "C:\\fiddler\\config.json";
  try {
    var configText = System.IO.File.ReadAllText(configPath);
    return Fiddler.WebFormats.JSON.JsonDecode(configText).JSONObject;
  } catch (e) {
    FiddlerApplication.Log.LogString("配置文件读取失败: " + e.toString());
    return {};
  }
}

// 保存请求数据到文件
function saveRequestData(oSession: Session) {
  var logPath = "C:\\fiddler\\requests.log";
  var logEntry = {
    url: oSession.url,
    method: oSession.HTTPMethodIs,
    timestamp: new Date().toISOString(),
    headers: oSession.oRequest.headers.ToString(),
  };

  try {
    System.IO.File.AppendAllText(logPath, JSON.stringify(logEntry) + "\n");
  } catch (e) {
    FiddlerApplication.Log.LogString("日志写入失败: " + e.toString());
  }
}
```

## 插件开发

### 1. Fiddler 插件架构

Fiddler 支持通过 .NET 开发自定义插件，扩展其功能。

#### 插件类型

- **Inspector 插件**: 自定义请求/响应检查器
- **Import 插件**: 自定义导入格式支持
- **Transform 插件**: 自定义数据转换
- **Rules 插件**: 自定义规则处理

### 2. 创建 Inspector 插件

#### 项目设置

1. 创建 C# 类库项目
2. 引用 Fiddler 程序集：
   - `Fiddler.exe`
   - `FiddlerCore.dll`

#### 基础插件代码

```csharp
using Fiddler;
using System.Windows.Forms;

public class CustomInspector : Inspector2, IResponseInspector2, IRequestInspector2
{
    private TextBox txtBody;
    private HTTPRequestHeaders requestHeaders;
    private HTTPResponseHeaders responseHeaders;
    byte[] body;

    public override int GetOrder()
    {
        return 0; // 显示顺序
    }

    public string GetTitle()
    {
        return "Custom Inspector"; // 标签页标题
    }

    public void AssignSession(HTTPSessionInfo oSession)
    {
        // 清空之前的内容
        txtBody.Clear();

        if (oSession != null)
        {
            if (oSession.requestBodyBytes != null)
            {
                body = oSession.requestBodyBytes;
                txtBody.Text = System.Text.Encoding.UTF8.GetString(body);
            }
        }
    }

    public void Clear()
    {
        txtBody.Clear();
        body = null;
    }

    public bool bDirty { get { return false; } }

    public HTTPRequestHeaders headers
    {
        get { return requestHeaders; }
        set { requestHeaders = value; }
    }

    public byte[] body
    {
        get { return body; }
        set
        {
            body = value;
            if (body != null)
            {
                txtBody.Text = System.Text.Encoding.UTF8.GetString(body);
            }
        }
    }

    // UI 初始化
    public Control GetControl()
    {
        if (txtBody == null)
        {
            txtBody = new TextBox();
            txtBody.Multiline = true;
            txtBody.ScrollBars = ScrollBars.Both;
            txtBody.Dock = DockStyle.Fill;
        }
        return txtBody;
    }
}
```

### 3. 插件部署

#### 编译和安装

1. 编译插件项目生成 `.dll` 文件
2. 将 `.dll` 文件复制到 Fiddler 安装目录的 `Importers` 文件夹
3. 重启 Fiddler
4. 插件将自动加载

## 性能优化

### 1. 内存管理

#### 会话数量控制

```javascript
// 限制会话数量，防止内存溢出
static function OnBeforeRequest(oSession: Session) {
    if (FiddlerApplication.UI.lvSessions.TotalItemCount() > 10000) {
        FiddlerApplication.UI.lvSessions.RemoveSelected();
    }
}
```

#### 自动清理

```javascript
// 定期清理旧会话
var sessionCleanupTimer = 0;

static function OnBeforeRequest(oSession: Session) {
    sessionCleanupTimer++;

    // 每 1000 个请求清理一次
    if (sessionCleanupTimer >= 1000) {
        FiddlerApplication.UI.actSelectAll();
        FiddlerApplication.UI.actRemoveUnselected();
        sessionCleanupTimer = 0;
    }
}
```

### 2. 过滤优化

#### 高效过滤

```javascript
// 使用更高效的过滤条件
static function OnBeforeRequest(oSession: Session) {
    // 避免正则表达式，使用字符串方法
    if (oSession.HostnameIs("api.example.com")) {
        // 处理特定域名的请求
    }

    // 使用 Contains 而不是正则表达式
    if (oSession.uriContains("static")) {
        // 处理静态资源请求
    }
}
```

#### 禁用不必要的功能

```javascript
static function Main() {
    // 禁用时间线视图以提高性能
    FiddlerApplication.Prefs.SetString("fiddler.ui.timeline.show", "false");

    // 禁用自动保存
    FiddlerApplication.Prefs.SetString("fiddler.ui.rules.AutoSave", "false");
}
```

## 网络模拟

### 1. 带宽限制

#### 模拟慢速网络

```javascript
static function OnBeforeResponse(oSession: Session) {
    // 模拟 3G 网络 (1 Mbps)
    if (oSession.uriContains("/api/")) {
        var downloadSpeed = 1024 * 1024 / 8; // 1 MB/s in bytes
        oSession["response-trickle-delay"] = ((oSession.responseBodyBytes.Length / downloadSpeed) * 1000).ToString();
    }
}
```

#### 动态带宽调整

```javascript
// 根据时间调整网络速度
static function OnBeforeResponse(oSession: Session) {
    var hour = new Date().getHours();
    var delay = "100"; // 默认 100ms

    // 模拟高峰期网络拥堵
    if (hour >= 9 && hour <= 11) {
        delay = "500"; // 500ms 延迟
    } else if (hour >= 14 && hour <= 16) {
        delay = "800"; // 800ms 延迟
    }

    oSession["response-trickle-delay"] = delay;
}
```

### 2. 错误模拟

#### 模拟服务器错误

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 50% 概率返回 500 错误
    if (oSession.uriContains("/api/unstable") && Math.random() > 0.5) {
        oSession.utilCreateResponseAndBypassServer();
        oSession.responseCode = 500;
        oSession.responseBody = System.Text.Encoding.UTF8.GetBytes("Internal Server Error");
        oSession.oResponse.headers.HTTPResponseCode = 500;
        oSession.oResponse.headers.HTTPResponseStatus = "500 Internal Server Error";
    }
}
```

#### 模拟网络超时

```javascript
static function OnBeforeRequest(oSession: Session) {
    // 模拟特定接口超时
    if (oSession.uriContains("/api/timeout")) {
        oSession["request-trickle-delay"] = "30000"; // 30秒延迟（会导致超时）
    }
}
```

## 安全测试

### 1. 认证测试

#### 自动添加认证头

```javascript
static function OnBeforeRequest(oSession: Session) {
    if (oSession.uriContains("/api/protected")) {
        // 添加 Bearer Token
        oSession.oRequest.headers["Authorization"] = "Bearer your-jwt-token-here";

        // 添加 API Key
        oSession.oRequest.headers["X-API-Key"] = "your-api-key-here";
    }
}
```

#### 测试不同认证方式

```javascript
static function OnBeforeRequest(oSession: Session) {
    if (oSession.uriContains("/api/auth-test")) {
        // 根据参数选择认证方式
        var authType = oSession.oRequest.headers["X-Auth-Type"];

        switch (authType) {
            case "basic":
                var credentials = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes("username:password"));
                oSession.oRequest.headers["Authorization"] = "Basic " + credentials;
                break;

            case "bearer":
                oSession.oRequest.headers["Authorization"] = "Bearer token123";
                break;

            case "apikey":
                oSession.oRequest.headers["X-API-Key"] = "key456";
                break;
        }
    }
}
```

### 2. 安全头检查

#### 检查安全响应头

```javascript
static function OnBeforeResponse(oSession: Session) {
    var securityHeaders = [
        "Strict-Transport-Security",
        "X-Content-Type-Options",
        "X-Frame-Options",
        "X-XSS-Protection",
        "Content-Security-Policy"
    ];

    for (var i = 0; i < securityHeaders.length; i++) {
        if (!oSession.oResponse.headers.Exists(securityHeaders[i])) {
            FiddlerApplication.Log.LogString("缺失安全头: " + securityHeaders[i] + " 在 " + oSession.url);
        }
    }
}
```

## 自动化测试

### 1. 测试脚本编写

#### API 测试脚本

```javascript
// 测试 API 响应时间
function testAPIPerformance(apiUrl, maxTime) {
  var startTime = new Date().getTime();

  var oSession = new Session(
    RequestBuilder.HTTPRequest("GET", apiUrl),
    ResponseBuilder.HTTPResponse()
  );

  oSession.SendRequest();

  var endTime = new Date().getTime();
  var responseTime = endTime - startTime;

  if (responseTime > maxTime) {
    FiddlerApplication.Log.LogString(
      "API 性能测试失败: " + apiUrl + " 耗时 " + responseTime + "ms"
    );
  } else {
    FiddlerApplication.Log.LogString(
      "API 性能测试通过: " + apiUrl + " 耗时 " + responseTime + "ms"
    );
  }
}

// 执行测试
testAPIPerformance("https://api.example.com/users", 1000);
```

### 2. 批量请求测试

#### 压力测试脚本

```javascript
function performStressTest(url, concurrentCount) {
  for (var i = 0; i < concurrentCount; i++) {
    setTimeout(
      function (index) {
        var oSession = new Session(
          RequestBuilder.HTTPRequest("GET", url + "?testId=" + index),
          ResponseBuilder.HTTPResponse()
        );
        oSession.SendRequest();
      },
      i * 100,
      i
    );
  }
}

// 执行 50 个并发请求
performStressTest("https://api.example.com/load-test", 50);
```

## 日志和分析

### 1. 自定义日志

#### 详细请求日志

```javascript
static function OnAfterSessionComplete(oSession: Session) {
    var logEntry = {
        timestamp: oSession.Timers.ClientDoneRequest.ToString(),
        url: oSession.fullUrl,
        method: oSession.HTTPMethodIs,
        statusCode: oSession.responseCode,
        totalTime: oSession.Timers.ClientDoneResponse - oSession.Timers.ClientBeginRequest,
        requestSize: oSession.requestBodyBytes.Length,
        responseSize: oSession.responseBodyBytes.Length
    };

    var logPath = "C:\\fiddler\\detailed_log.json";
    System.IO.File.AppendAllText(logPath, JSON.stringify(logEntry) + "\n");
}
```

### 2. 性能分析报告

#### 生成性能报告

```javascript
function generatePerformanceReport() {
  var sessions = FiddlerApplication.UI.GetAllSessions();
  var report = {
    totalRequests: sessions.Length,
    averageResponseTime: 0,
    slowestRequests: [],
    errorCount: 0,
    bandwidthUsage: 0,
  };

  var totalTime = 0;
  var slowRequests = [];

  for (var i = 0; i < sessions.Length; i++) {
    var session = sessions[i];
    var responseTime =
      session.Timers.ClientDoneResponse - session.Timers.ClientBeginRequest;

    totalTime += responseTime;
    report.bandwidthUsage += session.responseBodyBytes.Length;

    if (responseTime > 1000) {
      slowRequests.push({
        url: session.url,
        responseTime: responseTime,
      });
    }

    if (session.responseCode >= 400) {
      report.errorCount++;
    }
  }

  report.averageResponseTime = totalTime / sessions.Length;
  report.slowestRequests = slowRequests.slice(0, 10); // 取前10个最慢的请求

  var reportPath = "C:\\fiddler\\performance_report.json";
  System.IO.File.WriteAllText(reportPath, JSON.stringify(report, null, 2));

  FiddlerApplication.Log.LogString("性能报告已生成: " + reportPath);
}
```

## 下一步

掌握了高级用法后，你可以：

1. 深入学习 [FiddlerScript API](https://docs.telerik.com/fiddler/fiddlerscript-reference)
2. 探索 [官方插件库](https://www.telerik.com/forums/fiddler)
