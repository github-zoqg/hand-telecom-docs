---
sidebar_position: 2
sidebar_label: "安装和配置"
---

# Fiddler 安装和配置

## 系统要求

### Fiddler Classic (Windows)

- **操作系统**: Windows 7 SP1 或更高版本
- **内存**: 至少 2GB RAM
- **磁盘空间**: 100MB 可用空间
- **.NET Framework**: 4.6.1 或更高版本

### Fiddler Everywhere (跨平台)

- **操作系统**: Windows 10+, macOS 10.14+, Ubuntu 18.04+
- **内存**: 至少 4GB RAM
- **磁盘空间**: 500MB 可用空间

## 下载和安装

### 1. 下载 Fiddler

#### 官方网站下载

1. 访问 [Fiddler 官方网站](https://www.telerik.com/fiddler)
2. 选择合适的版本：
   - **Fiddler Classic** (免费，仅 Windows)
   - **Fiddler Everywhere** (付费，跨平台)

#### 直接下载链接

- [Fiddler Classic 下载](https://www.telerik.com/download/fiddler)
- [Fiddler Everywhere 下载](https://www.telerik.com/products/fiddler/fiddler-everywhere.aspx)

### 2. 安装步骤

#### Windows (Fiddler Classic)

1. 运行下载的 `.exe` 安装文件
2. 选择安装路径（建议使用默认路径）
3. 选择安装组件：
   - ✅ Fiddler Core (必需)
   - ✅ Fiddler 主程序
   - ✅ 桌面快捷方式
   - ✅ 开始菜单项
4. 点击 "Install" 开始安装
5. 等待安装完成
6. 启动 Fiddler

#### 跨平台安装 (Fiddler Everywhere)

1. 下载对应平台的安装包：

   - Windows: `.exe` 文件
   - macOS: `.dmg` 文件
   - Linux: `.AppImage` 或 `.deb` 文件

2. 安装步骤：

   ```bash
   # Linux Ubuntu/Debian
   sudo dpkg -i fiddler-everywhere_*.deb

   # 修复依赖问题（如果需要）
   sudo apt-get install -f

   # Linux AppImage
   chmod +x fiddler-everywhere*.AppImage
   ./fiddler-everywhere*.AppImage
   ```

## 初始配置

### 1. 首次启动设置

启动 Fiddler 后，会看到以下配置界面：

#### 语言设置

- 默认语言：英语
- 可选择中文（如果可用）

#### 主题设置

- **Light Theme**: 浅色主题
- **Dark Theme**: 深色主题
- **System Theme**: 跟随系统主题

#### 更新设置

- ✅ 自动检查更新
- ✅ 参与用户体验改进计划（可选）

### 2. 网络配置

#### 代理设置

Fiddler 默认会自动配置系统代理：

1. **检查代理状态**

   - 点击 `Tools` → `Options` → `Connections`
   - 确认 "Allow remote computers to connect" 已勾选

2. **代理端口设置**
   - 默认端口：8888
   - 可根据需要修改（避免与其他服务冲突）

#### HTTPS 解密配置

这是 Fiddler 最重要的配置之一：

1. **启用 HTTPS 解密**

   - 点击 `Tools` → `Options` → `HTTPS`
   - ✅ 勾选 "Decrypt HTTPS traffic"
   - ✅ 勾选 "Ignore server certificate errors"

2. **安装根证书**

   - 点击 "Actions" → "Trust Root Certificate"
   - 在弹出的安全警告中点击 "是"
   - 证书将安装到 "受信任的根证书颁发机构"

3. **浏览器证书配置**
   - Chrome/Edge: 自动使用系统证书
   - Firefox: 需要手动导入证书

### 3. 浏览器配置

#### Chrome/Edge

- 自动使用系统代理设置
- 无需额外配置

#### Firefox

1. 打开 Firefox 设置
2. 搜索 "代理"
3. 选择 "手动代理配置"
4. 设置：
   - HTTP 代理：127.0.0.1
   - 端口：8888
   - ✅ 勾选 "对此代理使用 HTTPS"

## 移动设备配置

### iOS 设备

1. 确保 iOS 设备和运行 Fiddler 的电脑在同一网络
2. 获取电脑 IP 地址：
   ```cmd
   ipconfig
   ```
3. 在 iOS 设备上：
   - 设置 → 无线局域网 → 点击当前 WiFi 的 (i)
   - HTTP 代理 → 配置代理 → 手动
   - 服务器：电脑 IP 地址
   - 端口：8888
4. 安装证书：
   - 在 Safari 中访问 `http://电脑IP:8888`
   - 点击 "FiddlerRoot certificate"
   - 安装证书
   - 设置 → 通用 → 关于本机 → 证书信任设置
   - 启用 Fiddler 根证书

### Android 设备

1. 获取电脑 IP 地址
2. 在 Android 设备上：
   - 长按 WiFi 网络 → 修改网络
   - 高级选项 → 代理 → 手动
   - 代理主机名：电脑 IP 地址
   - 代理端口：8888
3. 安装证书：
   - 浏览器访问 `http://电脑IP:8888`
   - 下载 FiddlerRoot.cer
   - 设置 → 安全 → 加密与凭据 → 安装证书 → Wi-Fi 证书

## 常见问题解决

### 1. 无法捕获 HTTPS 流量

**问题**: Fiddler 只显示 HTTP 请求，不显示 HTTPS 请求

**解决方案**:

1. 检查 HTTPS 解密是否启用
2. 重新安装 Fiddler 根证书
3. 重启浏览器和 Fiddler
4. 检查目标应用是否使用了证书固定

### 2. 代理连接失败

**问题**: 浏览器无法通过 Fiddler 访问网络

**解决方案**:

1. 检查 Fiddler 是否正在运行
2. 确认代理设置正确（127.0.0.1:8888）
3. 检查防火墙设置
4. 重启 Fiddler 和浏览器

### 3. 证书安装失败

**问题**: 无法安装 Fiddler 根证书

**解决方案**:

1. 以管理员身份运行 Fiddler
2. 检查系统权限设置
3. 手动导入证书到信任存储
4. 在组策略中允许用户安装证书

### 4. 移动设备无法连接

**问题**: 手机无法通过 Fiddler 上网

**解决方案**:

1. 确认设备和电脑在同一网络
2. 检查 IP 地址和端口设置
3. 在 Fiddler 中启用远程连接
4. 检查防火墙是否阻止连接

## 高级配置

### 1. 自定义规则

通过 `FiddlerScript` 可以自定义 Fiddler 行为：

1. 点击 `Rules` → `Customize Rules`
2. 在打开的编辑器中编写脚本
3. 保存脚本后立即生效

### 2. 插件安装

Fiddler 支持第三方插件扩展功能：

1. 下载插件文件
2. 放置在 Fiddler 安装目录的 `Importers` 文件夹
3. 重启 Fiddler

### 3. 性能优化

对于大量流量处理，可以进行以下优化：

1. **增加缓冲区大小**

   - `Tools` → `Options` → `Tools`
   - 调整 "Maximum request/response body size"

2. **禁用不必要的功能**
   - 关闭自动保存
   - 减少日志保留时间
   - 禁用流量统计

## 配置备份

### 导出配置

1. `File` → `Export` → `All Sessions`
2. 选择保存位置
3. 导出的 `.saz` 文件包含所有配置

### 导入配置

1. `File` → `Import` → `Sessions`
2. 选择之前导出的 `.saz` 文件
3. 配置将自动恢复

## 下一步

完成安装和配置后，你可以：

1. 学习 [常用功能](./common-features.md)
2. 了解 [高级用法](./advanced-usage.md)
3. 查看 [实战案例](./examples.md)
