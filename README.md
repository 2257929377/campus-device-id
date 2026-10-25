# 今日校园「扫码获取设备ID」复刻版

## 原理（已逆向验证）

你发的二维码内容是 `https://jrxyapp.zyh365.top/device.html`，这是一个第三方打卡工具的"设备授权"页。
它的完整工作链路：

1. 用**今日校园 App 内置扫一扫**打开该页面（普通浏览器打不开，见第 3 点）
2. 页面按 UA 判断平台，动态加载今日校园**官方公开**的 Cordova 桥接脚本：
   - Android: `https://static.campushoy.com/innerjs/cordova_android.js`
   - iOS: `https://static.campushoy.com/innerjs/cordova_ios.js`
   （实测两文件均公网可访问，华为 OBS 托管，CORS 全开）
3. 今日校园 App 的 WebView 里注入了 Cordova 运行时，脚本加载后触发 `deviceready` 事件
4. 页面调用原生插件：`cordova.exec(success, error, "MAMPDevice", "getDeviceInfo", [])`
   → 返回 `{ uuid, deviceId, ... }`，这就是"设备ID"
5. 兜底途径：`BH_MOBILE.device.getDeviceId()`、`window.device.uuid`
6. 原版页面拿到 ID 后，**静默 POST 到 `https://a.cn/report.php`**（带固定 token）——
   这就是打卡工具厂商收集/绑定你设备授权的过程

## 与原版的区别

- ✅ 保留：全部三种获取途径 + 复制按钮 + 失败提示
- ✅ 增加：展示完整设备信息 payload、环境调试信息（UA / cordova 是否存在）
- ❌ 移除：**不会把设备ID上报给任何第三方**（原版会上报到 a.cn/report.php）
- 🔧 如需上报到自己的后端，改 `device.html` 顶部的 `REPORT_URL` 变量（留空 = 纯本地展示）

## 使用步骤

1. 把 `device.html` 部署到任意 HTTPS 静态托管（GitHub Pages / Cloudflare Pages / 自己的 VPS / 服务器 nginx 都行）
2. 用二维码生成器（如 `https://cli.im/`）把你的部署地址做成二维码
3. **必须用今日校园 App 的扫一扫**打开（App 内置浏览器才有 Cordova 桥）
4. 页面显示设备ID → 点复制 → 填进你的打卡工具配置

## 注意

- 普通浏览器 / 微信扫码打开会显示"获取失败"，这是正常的（没有原生桥）
- 设备ID本质是 App 生成的设备唯一标识，换手机/重装 App 可能变化
- 该功能仅供学习研究，配合打卡工具使用请自行评估学校相关规定
