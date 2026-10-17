# Gemini in Chrome Toolkit

> Enterprise-grade local configuration, launcher automation, and diagnostic toolkit for Gemini in Chrome on Windows.

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%7C%2011-blue?logo=windows)](https://microsoft.com)
[![Chrome](https://img.shields.io/badge/Chrome-Official%20Stable-orange?logo=googlechrome)](https://www.google.com/chrome/)
[![Runtime](https://img.shields.io/badge/Runtime-PowerShell%207%20%7C%20Python%203.10%2B-brightgreen)](https://github.com/PowerShell/PowerShell)
[![License](https://img.shields.io/badge/License-MIT-purple)](LICENSE)

---

## 📌 项目概述 (Overview)

`gemini-in-chrome-toolkit` 是一套面向 Windows 平台的工程化工具集，旨在解决 **Gemini in Chrome (Project Glic)** 本地环境诊断、特征准入覆盖与稳定启动。工具通过**进程隔离守护**、**原子配置注入**与**环境快照备份**，为开发者和高级用户提供开箱即用的一键调试与配置恢复能力。

---

## ⚡ 核心修改内容提取 (Core Modifications)

本工具对本地 Chrome 环境所做的核心修改分为**运行时启动参数注入**与**本地配置文件持久化修改**两个维度：

### 1. 快捷方式与运行时启动参数 (Launch Entrypoint)

桌面生成的专用快捷方式 `Chrome - Gemini US` 调用隔离启动器，核心执行命令行如下：

```cmd
"C:\Program Files\Google\Chrome\Application\chrome.exe" --variations-override-country=us --disable-features=GlicCountryFiltering,GlicUseSessionCountryForFiltering,GlicLocaleFiltering
```

#### 启动参数与特征标志说明

| 参数 / 特征标识 | 类型 | 功能解析与技术作用 |
| :--- | :--- | :--- |
| `--variations-override-country=us` | Variations Switch | 强制将 Chromium 运行时的 Variations 区域环境覆盖为美国 (`us`)。 |
| `--disable-features=...` | Feature Switch | 关闭 Chrome 内部对 Gemini in Chrome (Glic) 的多重准入过滤特性： |
| ↳ `GlicCountryFiltering` | Feature Flag | 禁用持久化国家代码与 IP 归属地过滤策略。 |
| ↳ `GlicUseSessionCountryForFiltering` | Feature Flag | 禁用基于服务端下发 Session 动态国家的过滤拦截。 |
| ↳ `GlicLocaleFiltering` | Feature Flag | 禁用客户端系统与界面语言区域限制，防止非 en-US 语言环境被判定不符。 |

### 2. Chrome 本地配置注入项 (Profile Mutations)

修改前自动完成原子备份。工具对 Chrome 数据目录中的核心配置文件应用如下确定性修改：

| 配置文件路径 | 修改字段 (JSON Path) | 写入目标值 | 机制说明 |
| :--- | :--- | :--- | :--- |
| **`Local State`** | `variations_country` | `"us"` | 设定本地 Variations 判定国家 |
| | `variations_permanent_consistency_country` | `["<ChromeVersion>", "us"]` | 固化当前 Chrome 版本的长期一致性国家 |
| | `intl.app_locale` | `"en-US"` | 同步 Chrome 应用界面区域语言 |
| | `*.is_glic_eligible` | `true` | 递归更新配置树中已存在的 Glic 资格标记 |
| **`Preferences`**<br>*(Default 及所有 Profile)* | `intl.selected_languages` | `"en-US,en"` | 设置首选语言列表 |
| | `intl.accept_languages` | `"en-US,en"` | 设置 HTTP/内部接收语言优先序 |
| | `*.is_glic_eligible` | `true` | 同步对应 Profile 的资格状态 |

---

## 🚀 快速上手 (Quick Start)

### 1. 启动入口
双击项目根目录的 **`start.cmd`**（脚本会自动检测并优先调用 PowerShell 7，次选 Python 3）。

### 2. 交互菜单
```text
==================================================
  Gemini in Chrome Toolkit
==================================================
  [1] 启用并启动 Chrome (自动备份 -> 注入配置 -> 生成快捷方式 -> 启动)
  [2] 恢复最近一次备份配置 (一键回滚)
  [3] 查看当前环境诊断状态
  [4] 重新生成桌面专用快捷方式
  [5] 移除桌面专用快捷方式
  [0] 退出
```

> **日常使用**：完成初次启用后，日常直接双击桌面生成的 **`Chrome - Gemini US`** 快捷方式启动即可（启动器会自动处理残留进程并带参数拉起）。

---

## 🛠️ CLI 命令行接口 (CLI Reference)

支持在自动化脚本或 CI/CD 环境中通过参数无交互静默调用：

### PowerShell (7.4+)
```powershell
# 启用、备份并启动 Chrome (自动确认)
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Enable -Yes

# 仅修改配置并刷新快捷方式 (不启动 Chrome)
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Enable -Yes -NoLaunch

# 回滚到最新备份 / 回滚到指定备份目录
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Restore -Yes
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Restore -Backup "C:\path\to\backup" -Yes

# 状态诊断 / 快捷方式管理
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Status
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action Shortcut
pwsh -NoProfile -File .\scripts\gemini_chrome.ps1 -Action RemoveShortcut
```

### Python (3.10+)
```powershell
# 启用与启动
python .\scripts\gemini_chrome.py enable --yes [--no-launch]

# 备份回滚
python .\scripts\gemini_chrome.py restore --yes [--backup "C:\path\to\backup"]

# 状态检查与快捷方式管理
python .\scripts\gemini_chrome.py status
python .\scripts\gemini_chrome.py shortcut
python .\scripts\gemini_chrome.py remove-shortcut
```

---

## 🔍 验证与状态诊断 (Verification)

启动后可通过 Chrome 内置页面验证生效状态：

1. **核验启动参数**：在地址栏打开 `chrome://version`，确认「命令行」中包含：
   ```text
   --variations-override-country=us --disable-features=GlicCountryFiltering,GlicUseSessionCountryForFiltering,GlicLocaleFiltering
   ```
2. **核验 Glic 状态位**：在地址栏打开 `chrome://glic/internals`，重点确认：
   - `Passed country filter` / `Passed locale filter`：均显示通过
   - `Account exists and has the Gemini in Chrome capability`：账号能力有效
   - `Server side allows this feature`：服务端下发许可

---

## 🛡️ 安全机制与系统边界 (Safety & Boundaries)

- **原子化版本快照**：每次执行启用操作时，均会在 `%USERPROFILE%\Documents\GeminiInChromeToolkit\backups\<Timestamp>\` 自动生成包含 `manifest.json` 的独立快照，支持无损回滚。
- **纯用户态无侵入**：不修改 Windows 注册表，不绕过或破坏 Chrome 企业组策略 (Enterprise Policies)。
- **服务端条件说明**：本地启动参数和配置修改负责解除客户端的地域与语言限制，Google 账号本身的 Gemini 服务端授权仍取决于 Google 官方灰度条件。

---

## 📄 许可证

本项目采用 [MIT License](LICENSE)。
