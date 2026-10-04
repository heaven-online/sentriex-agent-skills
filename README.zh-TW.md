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

到[官方 Releases](https://github.com/heaven-online/sentriex-agent-skills/releases) 下載 `sentriex_v0.1.0_windows_amd64.zip`（x64）或 `sentriex_v0.1.0_windows_arm64.zip`（ARM64），以及 `checksums.txt`。用 `Get-FileHash -Algorithm SHA256` 計算 ZIP 的雜湊，與 `checksums.txt` 對應項目比對。

x64 可在 PowerShell 解壓縮並執行：

```powershell
$sentriexZip = "$env:USERPROFILE\Downloads\sentriex_v0.1.0_windows_amd64.zip"
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

設定 `SENTRIEX_BASE_URL` 為實際 Public API origin，並以環境變數 `SENTRIEX_API_KEY` 提供平台的 `sk_test_` 金鑰，再執行 `sentriex doctor --json`。金鑰應由安全的本機環境或 secret manager 載入，勿貼到對話或寫入 Git。

接著告訴 Agent：「請使用 Sentriex 串接 Skill，協助這個應用程式實作沙盒入金及 callback 驗簽。」

支援 Public API 0.3.0，CLI 最低版本 0.1.0。這裡的安裝目標是 Claude Code 與 Codex，不是一般 Claude 或 ChatGPT 聊天介面。
