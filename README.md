# DeepCleaner

> 一款单文件、纯 Win32 API、零依赖的 Windows 深度清理工具。  
> 深度清理 C 盘、Windows Update 缓存、**按你实际安装的软件**智能定位 AppData 缓存、以及注册表残留。深色现代 UI，开箱即用。

![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D6?logo=windows)
![Language](https://img.shields.io/badge/language-C%2B%2B17-blue)
![Framework](https://img.shields.io/badge/framework-Win32-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)
![Version](https://img.shields.io/badge/version-1.0.0-brightgreen)

---

## ✨ 特性

### 🔍 智能软件缓存扫描
不是拍脑袋写死一堆 App 名单，而是**先读注册表枚举这台机器上真正安装的软件**（`HKCU/HKLM/WOW6432Node` 三处 `Uninstall` 键 + 开始菜单 `.lnk` 补充），再对 `%LOCALAPPDATA%` / `%APPDATA%` 下每一个目录做**名称规范化 + 双向包含匹配**，只有匹配到已安装软件的目录才会被深入查找缓存子目录。

- 支持 Chromium 系（Chrome / Edge / Brave / 各类 Electron 应用）的 `User Data\<Profile>\Cache` 结构
- 支持 Firefox 的 `Profiles\<xxx>\cache2`
- 支持 `<App>\<版本号或 profile>\Cache` 深层结构
- 只删已知缓存目录名（`Cache` / `Code Cache` / `GPUCache` / `CachedData` / `Cache2` / `ShaderCache` / `DawnCache` / `Crashpad` / `blob_storage` 等），**绝不删除目录本身、绝不删除 `User Data` 根、绝不删除非缓存内容**

### 🗑 Windows 系统清理
- `SoftwareDistribution\Download` — Windows Update 下载缓存
- `SoftwareDistribution\DeliveryOptimization` — 传递优化缓存
- `SoftwareDistribution\DataStore` — 更新数据库（默认不勾，敏感）
- `C:\Windows\Temp`、`%TEMP%` — 系统 / 用户临时文件
- `Logs\WindowsUpdate`、`Logs\CBS` — 更新与组件日志
- `WER\ReportArchive`、`WER\ReportQueue` — 错误报告
- `Windows.old` — 旧系统安装残留（默认不勾，需接管所有权）
- `thumbcache_*.db` — 缩略图缓存
- `Prefetch` — 预读取文件（默认不勾）

### 🧹 注册表残留
- 遍历 `Uninstall` 键，判定 `InstallLocation` / `DisplayIcon` 指向路径**是否实际存在**
- `MUICache` 应用名缓存
- 所有注册表项**默认不勾选**，需要你手动确认

### 🔐 严谨的权限接管
- 清理前自动启用 `SeTakeOwnership` / `SeRestore` / `SeBackup` 特权
- 文件 / 目录：设置 Owner 为 `Administrators`，重建 DACL 授予完全控制
- 注册表：`SetNamedSecurityInfoW(SE_REGISTRY_KEY, "MACHINE\..." / "CURRENT_USER\...")` 接管所有权
- 递归遍历**跳过所有重解析点**（符号链接 / 连接点），防止循环与越界删除

### 🛡 多层安全防线
- **路径白名单**：每次删除前校验目标必须在 `LocalAppData`、`RoamingAppData`、`Temp`、`Windows`、`ProgramData` 之下，且比根目录更深
- **停用词过滤**：`update / setup / runtime / common / app` 等通用词不参与软件匹配，避免误删
- **系统目录排除**：`Microsoft`、`Packages`、`Temp`、`CrashDumps` 等不参与软件缓存匹配
- **默认保守**：敏感项（DataStore、Prefetch、Windows.old、全部注册表项）默认不勾
- **删除前确认**：弹窗列出数量并要求确认
- **清理报告**：完成后弹窗展示成功/失败项数与磁盘可用空间变化

### 🎨 现代 UI
- 深色主题（`#181822` 背景 / `#222230` 卡片 / `#4CC2FF` 强调色）
- Win11 圆角窗口 + 深色标题栏（`DwmSetWindowAttribute`）
- Segoe UI 字体、Owner-draw 圆角按钮
- 深色 ListView + 复选框多选
- 实时显示"已选中 N 项 · X MB"
- 底部进度条 + 状态文字
- 高 DPI 感知（`PER_MONITOR_AWARE_V2`）

### 📊 清理前后磁盘报告
点击清理前记录 `C:\` 可用空间，清理完成后弹窗：

```
清理完成

成功: 42 项    失败: 1 项

清理前 C 盘可用: 12.34 GB
清理后 C 盘可用: 15.67 GB
C 盘总容量:     256.00 GB

本次实际释放: 3.33 GB
```

---

## 📸 截图

> _（欢迎提交 PR 补充截图）_

```
┌────────────────────────────────────────────────────────────┐
│  DeepCleaner 1.0                                           │
│  C 盘 · Windows 更新缓存 · AppData 智能匹配 · 注册表残留     │
│                                                            │
│  [  扫描  ]  [  清理选中  ]           3.42 GB · 已选中 42 项│
│  ┌──────────────────────────────────────────────────────┐  │
│  │ ☑ 类别      名称                        大小          │  │
│  │ ☑ 更新      Windows Update 下载缓存     1.82 GB       │  │
│  │ ☑ 软件缓存  Chrome / Default / Cache    412 MB        │  │
│  │ ☑ 软件缓存  VS Code / Code Cache        88 MB         │  │
│  │ ☐ 注册表    FooApp（InstallLocation…）  —             │  │
│  └──────────────────────────────────────────────────────┘  │
│  扫描完成 — 已安装软件 137 个 · 发现 43 项可清理内容         │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓      │
└────────────────────────────────────────────────────────────┘
```

---

## 🖥 系统要求

| 项目 | 要求 |
|---|---|
| 操作系统 | Windows 10 1809+ / Windows 11 |
| 编译器 | Visual Studio 2026（v145 工具集）或更高 |
| 语言标准 | C++17 及以上 |
| 字符集 | Unicode |
| 目标平台 | x64（x86 未测试） |
| 运行库 | 无（静态链接 CRT 可选） |
| 权限 | **建议以管理员身份运行**（部分系统目录 / HKLM 注册表需要） |

---

## 🔨 编译

### 方式一：Visual Studio 2026

1. 新建 **空 C++ 项目** → 模板选择 **Windows 桌面应用程序**
2. 将 `CppDeepCleaner.cpp` 加入项目
3. 右键项目 → **属性**：
   - **C/C++ → 语言 → C++ 语言标准**：`ISO C++17` 或更高
   - **高级 → 字符集**：`使用 Unicode 字符集`
   - **链接器 → 系统 → 子系统**：`Windows (/SUBSYSTEM:WINDOWS)`
4. 选择 `x64` + `Release` → 生成
5. 输出：`x64\Release\CppDeepCleaner.exe`

### 方式二：命令行（MSVC）

```bat
cl /std:c++17 /EHsc /DUNICODE /D_UNICODE /O2 ^
   /link /SUBSYSTEM:WINDOWS ^
   user32.lib gdi32.lib shell32.lib shlwapi.lib comctl32.lib ^
   advapi32.lib ole32.lib dwmapi.lib uxtheme.lib ^
   CppDeepCleaner.cpp
```

---

## 🚀 使用

1. **右键 → 以管理员身份运行** `CppDeepCleaner.exe`（推荐）
2. 点击 **扫描**，程序会：
   - 枚举已安装软件
   - 扫描 Windows 更新缓存 / 系统临时目录
   - 匹配 AppData 下已安装软件的缓存目录
   - 扫描注册表残留
3. 在列表中**勾选**要清理的项目（默认已勾选安全项）
4. 点击 **清理选中** → 确认弹窗 → 等待进度条完成
5. 查看**清理报告**（成功/失败项数、磁盘空间变化）

---

## 🛡 安全设计说明

本工具会**删除文件与注册表项**，请在使用前了解以下设计，确保你的数据安全：

1. **只删白名单根路径下的内容**  
   `LocalAppData` / `RoamingAppData` / `Temp` / `Windows` / `ProgramData` 之外的一律跳过，并在失败信息里注明。
2. **绝不跟随重解析点**  
   所有递归遍历遇到符号链接 / 连接点立即跳过，防止越界或循环。
3. **绝不删除目录本身**  
   只清理缓存目录的**内容**，缓存目录容器保留，避免应用重建失败。
4. **敏感项默认不勾**  
   `DataStore`、`Prefetch`、`Windows.old`、全部注册表项默认不勾选，需你主动确认。
5. **软件匹配有阈值**  
   规范化后长度 < 3 的词、通用词（`update`、`runtime`、`app` 等）不参与匹配；未匹配到已安装软件的 AppData 目录一律不动。
6. **先接管权限再删除**  
   文件/目录/注册表项在删除前会先接管所有权与 DACL，失败会如实记录，不会静默。
7. **管理员权限非必须但强烈建议**  
   非管理员运行时，`Windows\Temp`、`SoftwareDistribution\Download`、`HKLM` 项会因权限失败，报告里会写明。

> ⚠️ **强烈建议**：首次使用前创建系统还原点，或在虚拟机 / 非工作机上先验证。

---

## ⚠️ 免责声明

本软件按 **MIT 协议** 提供，**"AS IS" 无任何担保**。清理类工具天然具有破坏性，作者**不对因使用本工具导致的任何数据丢失、系统故障或业务中断负责**。

- 请勿在**生产服务器**或**关键工作机**上盲测
- 清理 `DataStore` 后首次检查更新可能变慢，属正常现象
- 清理 `Windows.old` 后将**无法回退到旧版本 Windows**
- 删除注册表项前请确认列表中显示的名称与理由合理

**使用即代表你理解并接受上述风险。**

---

## 🗺 路线图

- [x] v1.0 — 核心清理 + 智能软件扫描 + 现代 UI
- [ ] v1.1 — 清理前创建系统还原点
- [ ] v1.2 — 支持导出扫描结果为 JSON / CSV
- [ ] v1.3 — 清理历史记录
- [ ] v1.4 — 多语言（English / 简体中文）
- [ ] v2.0 — 插件式清理规则（用户可自定义）

---

## 🤝 贡献

欢迎 Issue 和 PR！

1. Fork 本仓库
2. 新建分支：`git checkout -b feature/your-feature`
3. 提交：`git commit -am 'feat: add xxx'`
4. 推送：`git push origin feature/your-feature`
5. 发起 Pull Request

**提交 PR 前请确认：**
- 代码保持**单文件、零第三方依赖**
- 新增的清理规则必须附带**白名单根路径**与**匹配条件**
- 不要引入会对非缓存目录进行删除的逻辑

---

## 📄 License

[MIT](LICENSE) © 2026 Your Name

---

## 🙏 致谢

- 界面参考 Windows 11 Fluent Design 深色配色
- 智能匹配思路参考各类开源清理工具的注册表扫描逻辑
- 感谢所有提交 Issue 与 PR 的朋友

---

## English Summary

**DeepCleaner** is a single-file, zero-dependency Win32 C++ deep-cleanup utility for Windows 10/11.

Instead of hardcoding a list of apps, it **enumerates the software actually installed on your machine** (via the registry `Uninstall` keys) and **smart-matches** `%LOCALAPPDATA%` / `%APPDATA%` directories to their owners before touching any cache. It also cleans Windows Update caches, system temp, `Windows.old`, WER reports, thumbnails, and dangling registry entries.

Safety-first: path whitelist, reparse-point skipping, ownership takeover with privilege escalation, conservative defaults, and a post-clean report showing freed disk space.

**License:** MIT · **Build:** VS 2026 / C++17 / Unicode / x64
