# Sentriex 串接 Skill

提供 Claude Code 與 OpenAI Codex 使用的串接流程，包含 Public API 契約、CLI 操作、callback 驗簽與請求範例。

第一個 CLI 版本發布後，可依作業系統安裝，不需要 Go 或 Node.js。

## macOS 與 Linux

透過 Homebrew 安裝：

```sh
brew install heaven-online/tap/sentriex
sentriex --version
```

## Windows

到[官方 Releases](https://github.com/heaven-online/sentriex-agent-skills/releases) 下載 `sentriex_<version>_windows_amd64.zip`（x64）或 `sentriex_<version>_windows_arm64.zip`（ARM64），以及 `checksums.txt`；`<version>` 是 Releases 頁面上的版本 tag，例如 `v0.2.0`。用 `Get-FileHash -Algorithm SHA256` 計算 ZIP 的雜湊，與 `checksums.txt` 對應項目比對。

x64 可在 PowerShell 解壓縮並執行：

```powershell
$sentriexVersion = "v0.2.0" # 換成實際下載的版本
$sentriexZip = "$env:USERPROFILE\Downloads\sentriex_${sentriexVersion}_windows_amd64.zip"
$sentriexInstallDir = "$env:USERPROFILE\Tools\sentriex"
Expand-Archive -Path $sentriexZip -DestinationPath $sentriexInstallDir -Force
& "$sentriexInstallDir\sentriex.exe" --version
$env:Path = "$sentriexInstallDir;$env:Path"
```

ARM64 請改用對應 ZIP 檔名。以上 `Path` 設定只適用於目前終端機；若要永久使用，可在 Windows「環境變數」將安裝資料夾加入使用者的 `Path`，再開新的終端機。也可以直接使用執行檔的完整路徑。

## 安裝 Skill

在自己的應用程式專案裡安裝 Skill：

```sh
sentriex skills install --agent claude-code
# 或：
sentriex skills install --agent codex
```

預設安裝到目前專案；加 `--global` 安裝到使用者目錄。`--ref <tag-or-commit>` 可指定版本，`--force` 可更新既有 Skill。CLI 的安裝流程不需要 Node.js 或 Git。

也能獨立使用：

```sh
npx skills add heaven-online/sentriex-agent-skills --skill sentriex-integration
```

## 從沙盒開始

在自己的終端機執行一次（Windows 請使用 PowerShell 或 Windows Terminal）：

```sh
sentriex configure
```

base URL 提示直接按 Enter 採用預設值 `https://api.sentriex.io`，再貼上商戶後台取得的平台 `sk_test_` API key；輸入時不會顯示。callback secret 為選填，只用於在本機驗證擷取的 callback。設定會保存在目前使用者的 `~/.config/sentriex/config.json`，macOS 與 Linux 上僅本人可讀取。Agent 無法代為執行這個指令；金鑰勿貼到對話或寫入 Git。

接著檢查設定：

```sh
sentriex doctor --json
```

CI 或臨時覆寫時，改用環境變數 `SENTRIEX_API_KEY`、`SENTRIEX_BASE_URL`、`SENTRIEX_ENVIRONMENT` 或 `SENTRIEX_CALLBACK_SECRET`。優先順序為指令參數、環境變數、設定檔，最後才是內建預設值；`doctor` 會顯示每個值的來源。

接著告訴 Agent：「請使用 Sentriex 串接 Skill，協助這個應用程式實作沙盒入金及 callback 驗簽。」

支援 Public API 0.3.0，CLI 最低版本 0.2.0。這裡的安裝目標是 Claude Code 與 Codex，不是一般 Claude 或 ChatGPT 聊天介面。
