# FackWatch

**让小米手表也能收到「双开微信（手机分身）」的消息提醒。**

手表端微信登录主号之后，手机分身里那个微信的消息在手表上就再也不提醒了——因为手表会按包名 `com.tencent.mm` 把手机转发过去的微信通知丢掉。FackWatch 用一个 LSPosed 模块，在**小米运动健康**进程里把「分身的微信通知」伪装成另一个应用转发出去，手表就不会再丢掉它。

> 只在**小米运动健康 → 手表**这条通知链路上生效，不修改微信、不改通知栏、不联网。

---

## 原理

```
双开微信(user 999) 发通知
        │
        ▼
com.mi.health  NotifySyncService   ← FackWatch 在这里 hook
        │  把 StatusBarNotification 的包名换成身份壳包名
        ▼
蓝牙 → 手表          ← 手表认这个包名，不去丢它
```

- 手表只认**真实安装的应用**的名字和图标，所以需要一个「身份壳」APK（`com.lead.wxwechat3`）挂在手机上，它本身没有后台、不耗电、桌面图标已默认隐藏
- 手表对同一个包名只收一次图标，因此身份壳的包名是固定的，不要随意改

## 功能

- 手表上以「微信双开」+ 双开角标图标显示副号消息
- 桌面 App「FackWatch」只做一件事：一键发测试通知到手表
- 不常驻后台：没有 Service，靠 LSPosed 注入，只在通知产生时执行
- **两种通知模式都支持**：
  - 「同步手机通知」：所有应用通知一律同步
  - 「自定义消息通知」：只同步你勾选的应用。本模块会在小米运动健康读取名单时自动把身份壳补进去，**不需要你手动添加**

## 环境要求

| 项 | 要求 |
|---|---|
| 手机 | 小米 / Redmi，HyperOS，**已 root（KernelSU 或 Magisk）** |
| 框架 | **LSPosed**（Zygisk 版） |
| 手表 | 小米手表 S4（其它 VelaOS / HyperOS 手表理论可用，未验证） |
| 配套 App | 小米运动健康 |
| 微信 | 主号在手机主用户，副号在**手机分身 / 应用双开**（user 999） |

## 安装

1. 安装 `FackWatch-1.0.4.apk`
2. 打开 **LSPosed → 模块**，勾选 **FackWatch**
3. 点进 FackWatch → **作用域**，只勾 **小米运动健康**（`com.mi.health`）
4. 安装 `FackWatch-Identity-1.0.0.apk`（身份壳，桌面不显示图标）
5. 重启手机（LSPosed 换模块后必须重启一次，否则不会加载）
6. 打开桌面上的 **FackWatch**，点「发测试通知到手表」，手表应显示「微信双开」

> v1.0.4 起 App 内不再提供「隐藏桌面图标」——隐藏组件会让 LSPosed 模块页也无法打开。想隐藏请用系统自带：设置 → 应用设置 → 应用锁 → 隐藏应用（LSPosed 仍可打开）。

## 已知限制

- **只对「手机分身 / 应用双开」(user 999) 的微信生效**，主号走手表原生微信
- 手表微信必须登录**主号**；若主副互换，需要改模块里的 `DUAL_USER`
- **小米运动健康升级后模块可能失效**，需要按新版本重新适配
- 手表端按包名缓存图标：换图标必须换一个新的身份壳包名（不要自己改包名）

## 免责声明

- 本项目仅用于个人便利，不修改微信客户端、不破解、不绕过任何账号限制
- 身份壳默认图标含微信官方图标 + MIUI 双开角标，**仅限个人使用**；二次分发请自行替换素材，避免商标问题
- 使用本模块造成的任何问题（含设备异常、通知异常）由使用者自行承担

## 构建

需要 JDK 17、Android SDK build-tools 36、`android.jar`（API 36）。

```powershell
cd projects\wx-hook
powershell -File .\build.ps1 -Release -Version 1.0.4 -VersionCode 5
```

产物：`release\FackWatch-1.0.4.apk`（4096 位 RSA，v1+v2+v3 签名）

身份壳（另一个 APK，仓库未内置构建脚本时请直接使用 release 里的成品）：

```powershell
cd projects\wx-identity
powershell -File .\build.ps1 -Pkg com.lead.wxwechat3 -Name "微信双开" -Icon <图标.png> -Install
```

## License

MIT
