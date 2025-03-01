# 如何把 EPUB 拆分转换为 Markdown（Windows 操作系统）

📌 本文档详细介绍如何在 Windows 操作系统上，使用 `epub2md` 将 EPUB 文件拆分为 Markdown（.md）格式，并配置 **右键菜单** 以便快捷执行转换任务。

## 📌 目录
1. [安装必备环境](#安装必备环境)
2. [下载 & 安装 epub2md](#下载--安装-epub2md)
3. [手动执行 EPUB → Markdown 转换](#手动执行-epub--markdown-转换)
4. [创建 Windows 右键菜单，实现一键转换](#创建-windows-右键菜单实现一键转换)
5. [常见问题解决方案](#常见问题解决方案)

---

## 🚀 1. 安装必备环境

在运行 `epub2md` 之前，需要确保 **Node.js** 和 **epub2md** 已正确安装。

### 🔹 检查 Node.js 是否已安装
在 PowerShell 或命令提示符（CMD）中运行：

```powershell
node -v
```
如果返回类似 `v18.0.0` 这样的版本号，说明 **Node.js** 已安装。

### 🔹 如果 Node.js 未安装
- [点击这里下载](https://nodejs.org/) 并安装 **Node.js 长期支持版（LTS）**。
- 安装完成后，关闭 PowerShell 或 CMD 并重新打开，确保 `node -v` 正常运行。

---

## 📥 2. 下载 & 安装 epub2md
- [epub2md库来源](https://github.com/uxiew/epub2MD?tab=readme-ov-file)

安装 `epub2md`，打开 PowerShell 或 CMD，执行：

```powershell
npm install -g epub2md
```
然后检查是否安装成功：

```powershell
epub2md --version
```
如果成功，会返回版本号。

---

## 📌 3. 手动执行 EPUB → Markdown 转换

安装完成后，你可以在 PowerShell 或 CMD 中运行以下命令，将 EPUB 文件转换为 Markdown：
（我的文件放在D盘，所以先切换D盘，然后进入正确的文件目录）
```powershell
cd "D:\电子书"
cmd /c start powershell -ExecutionPolicy Bypass -NoProfile -File "D:\电子书\Convert_EPUB_to_Markdown.ps1" -EPUBFilePath "D:\电子书\jisuanjishizmpaoqilaide.epub"

```

-Convert_EPUB_to_Markdown.ps1：将 EPUB 拆分并转换为 Markdown 格式的PowerShell 脚本（见下4.1）
- `D:\电子书\example.epub`：你的 EPUB 文件路径

转换成功后，你会在 **相同目录** 下看到 `.md` 文件。

---

## 🖱 4. 创建 Windows 右键菜单，实现一键转换

为了更方便地转换 EPUB，可以在 Windows 右键菜单 **添加 “Convert EPUB to Markdown”** 选项。

### 🔹 4.1 创建 PowerShell 脚本

新建 PowerShell 脚本（`Convert_EPUB_to_Markdown.ps1`），内容如下：

```powershell
param ([string]$EPUBFilePath)

# 确保 epub2md 可用
$epub2mdPath = (Get-Command epub2md -ErrorAction SilentlyContinue).Source
if (-not $epub2mdPath) {
    Write-Output "❌ 错误：epub2md 未安装，请先运行 `npm install -g epub2md` 进行安装！"
    exit 1
}

# 获取 EPUB 文件路径
$OutputDir = "$([System.IO.Path]::GetDirectoryName($EPUBFilePath))\Converted"

# 确保输出目录存在
if (-not (Test-Path $OutputDir)) {
    New-Item -ItemType Directory -Path $OutputDir -Force | Out-Null
}

# 运行转换
Write-Output "🚀 正在转换 EPUB 到 Markdown..."
epub2md -M "$EPUBFilePath"

Write-Output "✅ 转换完成！输出目录: $OutputDir"
```

### 🔹 4.2 创建 VBS 脚本

由于 Windows 右键菜单无法直接运行 PowerShell，我们需要用 **VBS 脚本** 作为桥梁。

新建 `Convert_EPUB.vbs`，填入以下内容：

```vbscript
Set objShell = CreateObject("WScript.Shell")
objShell.Run "powershell -ExecutionPolicy Bypass -NoProfile -File ""D:\电子书\Convert_EPUB_to_Markdown.ps1"" -EPUBFilePath """ & WScript.Arguments(0) & """", 1, False
```

### 🔹 4.3 添加 Windows 右键菜单

新建 **添加右键菜单.reg** 文件，并输入：

```ini
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\SystemFileAssociations\.epub\shell\ConvertEPUBtoMarkdown]
@="Convert EPUB to Markdown"
"Icon"="powershell.exe"

[HKEY_CLASSES_ROOT\SystemFileAssociations\.epub\shell\ConvertEPUBtoMarkdown\command]
@="wscript.exe \"D:\\电子书\\Convert_EPUB.vbs\" \"%1\""
```

### 🔹 4.4 添加到注册表

- **双击** `添加右键菜单.reg`，然后点击 “是” 确认添加。

### 🔹 4.5 右键菜单转换 EPUB

- **右键点击任何 EPUB 文件**，选择 **“Convert EPUB to Markdown”**，即可自动转换并在 EPUB 所在目录下生成 Markdown 文件。

---

## 🛠 5. 常见问题 & 解决方案

### ❓ EPUB 解析失败或只生成少量 Markdown

可能是 EPUB 结构特殊，尝试先 **解压 EPUB**：

```powershell
epub2md -u "D:\电子书\example.epub"
```

然后查看 `textXXX.html` 文件是否包含完整内容。

### ❓ 右键菜单不生效

**重新启动 Windows 资源管理器**

```powershell
taskkill /f /im explorer.exe & start explorer
```

**检查注册表是否成功导入**
- 在 **注册表编辑器（regedit）** 中，查看 `HKEY_CLASSES_ROOT\SystemFileAssociations\.epub\shell\ConvertEPUBtoMarkdown` 是否存在。

**确保 epub2md 正确安装**

```powershell
npm list -g epub2md
```

**尝试直接运行 PowerShell 脚本**

```powershell
powershell -ExecutionPolicy Bypass -File "D:\电子书\Convert_EPUB_to_Markdown.ps1" -EPUBFilePath "D:\电子书\example.epub"
```

---

## 🎯 结论
✅ 你现在可以：

- **手动** 在 PowerShell 运行 `epub2md` 转换 EPUB 到 Markdown。
- **右键 EPUB**，选择 **“Convert EPUB to Markdown”**，一键自动转换。

🚀 这一方法适用于 **Windows 10/11**，支持所有 EPUB 文件格式。
