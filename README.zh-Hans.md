<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>macOS 上的 AI 双窗格文件管理器。</strong><br>
  Total Commander 的深度,ForkLift 的精致,内置 AI 智能。
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
</p>

<p align="center">
  <a href="https://unifyl.app/zh-Hans/">官网</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">文档</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">下载</a>
</p>

<p align="center">
  <strong>语言:</strong>
  <a href="README.md">English</a> &middot;
  <a href="README.ja.md">日本語</a> &middot;
  <strong>简体中文</strong> &middot;
  <a href="README.de.md">Deutsch</a> &middot;
  <a href="README.es.md">Español</a> &middot;
  <a href="README.fr.md">Français</a>
</p>

---

![Unifyl 截图](docs/screenshots/01-dualpane.png)

## 概览

Unifyl 是面向 macOS 的双窗格文件管理器,将专业级强力工具与本地 AI 智能融为一体。你的文件永不离开 Mac。

- **双窗格** — 单、双、三窗格及自由分割布局,标签页、工作区,每个窗格内置内联终端
- **AI 驱动** — 语义搜索、AI 重命名、智能标记、重复检测 — 全部在 Apple Neural Engine 本地运行
- **远程与云** — FTP、SFTP、WebDAV、SMB、S3、Google Drive、Dropbox、OneDrive、Docker、Kubernetes
- **50 多个专业工具** — 批量重命名、文本/二进制/图像比较、目录同步、三方合并、EXIF 批量编辑
- **压缩包作为虚拟文件夹** — ZIP、7z、TAR、GZ、BZ2、XZ、ZSTD、RAR(包含 CJK 旧式 ZIP 自动编码识别)
- **键盘优先** — 命令面板、Vim 风格导航、120 多个可重新绑定的快捷键
- **15 种语言** — 英语、韩语、日语、中文(简体与繁体)、德语、法语、西班牙语、葡萄牙语、意大利语、俄语、土耳其语、阿拉伯语、泰语、越南语
- **CJK 强化** (1.3.0): `.alz` 压缩包提取(内置 `unalz`),`.hwpx` 内联预览,拼音 / 罗马字 / 韩文-汉字多语言文件名搜索
- **CJK 进一步强化** (1.3.1): 通讯/云盘文件夹自动书签(KakaoTalk · LINE · WeChat · iCloud Drive 等 12 种),全角 ↔ 半角搜索折叠,EUC-KR/Shift-JIS/GBK → UTF-8 编码转换(支持撤销),韩国政府表格启发式检测

---

## 安装

从 [unifyl.app/download](https://github.com/goodbug89/Unifyl.app/releases/latest) 获取最新版本。

也可在 [Mac App Store](https://apps.apple.com/app/unifyl/id6773977314?mt=12) 获取（沙盒版 — 功能差异请参阅 FAQ）。

1. 打开 `.dmg` 文件
2. 将 Unifyl 拖到「应用程序」文件夹
3. 启动,并在提示时授予文件访问权限

---

## 价格

| | **Free** | **Pro — $24.99** |
|---|---|---|
| 窗格布局 | 双／三／自由分割（全部） | 单、双、三、自由分割 |
| 搜索 | 基本 + Spotlight | 完整正则 + 内容搜索 |
| AI 工具 | — | 全部 AI 功能 |
| 远程连接 | FTP、SFTP | 全部协议 + 云存储 |
| 压缩包 | ZIP | ZIP、7z、TAR、GZ、BZ2、XZ、ZSTD、RAR、**ALZ** |

> **版本说明。** Mac App Store 版本运行在沙盒中，不包含 Git 面板、Docker/Kubernetes、SSH 隧道、内联终端、端口与进程查看器，以及基于 ffmpeg 的媒体转换 —— macOS 在那里不允许这些功能。其余部分完全相同，Pro 也是两边同样的一次性价格。如果你需要以上功能，请选择官网直接下载版。

一次性买断,无订阅。在 Mac App Store 免费下载(Pro 通过 App 内购买,绑定 Apple ID 并支持家人共享),或直接下载(Pro 通过 LemonSqueezy,最多可在 3 台设备上使用)。首次启动享 14 天 Pro 免费试用。

---

## 主要键盘快捷键

| 快捷键 | 操作 |
|---|---|
| `⌘ + ⇧ + P` | 命令面板 |
| `Tab` | 切换活动窗格 |
| `Return` | 打开文件 / 进入文件夹 |
| `⇧ + Return` | 重命名 |
| `Space` | 快速查看 |
| `⌘ + Delete` | 移到废纸篓 |
| `⌃ + G` | 缩略图网格 |
| `⌘ + T` | 新建标签 |
| `⌘ + 2` | 双窗格 |
| `⌘ + K` | 终端 |

全部 120 多个快捷键可在「设置 > 键盘快捷键」中自定义。

---

## 从源码构建

```bash
git clone https://github.com/goodbug89/Unifyl.app.git
cd unifyl
make doctor    # 工具链检查
make gen       # 生成 Xcode 项目
make build     # 顺次构建全部包
```

完整功能列表、贡献指南、插件开发请参阅 [English README](README.md)。

---

## 许可证

专有。Copyright 2024–2026 Unifyl. 保留所有权利。

<p align="center">
  <a href="https://unifyl.app/zh-Hans/">官网</a> &middot;
  <a href="https://github.com/goodbug89/Unifyl.app/wiki">文档</a>
</p>
