# 今日校园「扫码获取设备ID」页面

## 工作原理

1. 用**今日校园 App 内置扫一扫**打开页面（普通浏览器无法获取，页面会引导用户改用今日校园）
2. 页面按 UA 判断平台，动态加载今日校园**官方公开**的 Cordova 桥接脚本：
   - Android: `https://static.campushoy.com/innerjs/cordova_android.js`
   - iOS: `https://static.campushoy.com/innerjs/cordova_ios.js`
   （实测两文件均公网可访问，华为 OBS 托管，CORS 全开）
3. 今日校园 App 的 WebView 里注入了 Cordova 运行时，脚本加载后触发 `deviceready` 事件
4. 页面调用原生插件：`cordova.exec(success, error, "MAMPDevice", "getDeviceInfo", [])`
   → 返回 `{ uuid, deviceId, ... }`，这就是"设备ID"
5. 兜底途径：`BH_MOBILE.device.getDeviceId()`、`window.device.uuid`

## 页面行为

- **在今日校园内打开**：显示设备ID + 复制按钮，不展示其他设备信息
- **在其他环境打开**：显示「请在今日校园中打开」引导页，附正确操作步骤；提供「尝试打开今日校园」按钮（Android 走 `intent://` + 包名 `com.wisedu.cpdaily`；iOS 尝试常见 scheme，未真机实测）
- **微信内打开**：额外显示警告 —— 微信屏蔽所有 scheme 拉起，必须退出微信改用今日校园扫一扫

## 隐私说明

- 页面**不会把设备ID上报到任何第三方**（原版工具页会静默上报到其服务器）
- 如需上报到自己的后端，改 `device.html` 顶部的 `REPORT_URL` 变量（留空 = 纯本地展示）

## 部署与使用

1. `device.html` 已部署在 GitHub Pages，线上地址：
   `https://2257929377.github.io/campus-device-id/device.html`
2. 用 `deploy-qr.png` 二维码（内容即上述地址）
3. **必须用今日校园 App 的扫一扫**打开，页面显示设备ID → 点复制 → 填进打卡工具配置

## 注意

- 普通浏览器 / 微信扫码打开会显示引导页，这是预期行为（没有原生桥）
- 设备ID本质是 App 生成的设备唯一标识，换手机/重装 App 可能变化
- 该功能仅供学习研究，配合打卡工具使用请自行评估学校相关规定
