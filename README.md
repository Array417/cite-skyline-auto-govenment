# AutoGovernment

Cities: Skylines II 虛擬政府 Mod 專案。玩家以國家元首身分輸入施政方向，透過「提出建議」或「強制命令」影響政府決策與城市。

**目前處於開發前驗證，尚未提供可玩成品或安裝包。** 倉庫中的 C# 程式仍是載入／卸載模板；文件描述的是完整產品要求，不能視為已實作功能。

預定功能包括共享本地輕量 LLM 的政府 agents、自主買地／劃區／道路與橋隧建設，以及選舉、議會、司法、股票、社會抗議和獨裁玩法。優先驗證以獨立 Mod 擴充 Presidente；已讀回部分公開系統並在城市成功建立及查詢一份草案，但正式政策生效、城市效果、施工及重載相容性仍待實測，尚未形成正式整合承諾。

## 專案文件

- [完整產品規格](PRD.md)：已確認需求、範圍與完整成品驗收條件。
- [結合 Presidente 的完整玩法設計](docs/GAMEPLAY_DESIGN.md)：開局、元首操作、自治部門、跨系統後果及完整施政例子。
- [玩法決策及相容性風險](docs/GAMEPLAY_DECISIONS.md)：七項已確認選擇、在野復權流程與開發方需要驗證的問題。
- [AI 開發指引](AGENTS.md)：工具鏈、實作原則及外部 Mod 整合規則。
- [開發前驗證紀錄](docs/PREFLIGHT.md)：已取得的證據、失敗案例及尚未通過的項目。
- [Presidente 整合研究](docs/PRESIDENTE_RESEARCH.md)：接口調查、授權與整合決策依據。
- [Presidente 內部設定盤點](docs/PRESIDENTE_SETTINGS_AUDIT.md)：完整選項、公開 setter、方法、常數與可修改邊界。
- [其他開源專案研究](docs/OPEN_SOURCE_RESEARCH.md)：Cities2-MCP、城市控制 Bridge 及道路工具的用途與授權區別。

## 開發環境概覽

以下以 Windows／PowerShell 為準，更新於 2026-11-16。版本表是本機已驗證組合，不是最新版本清單；遊戲、Unity、工具鏈與第三方 Mod 升級後要重新驗證。

| 項目 | 已驗證版本／要求 |
| --- | --- |
| Cities: Skylines II | Steam 版 1.6.4f1；需要本機遊戲檔案 |
| 官方 Modding Toolchain | 從遊戲安裝；包含 Mod.props、Mod.targets、postprocessor、Unity 模組專案及模板 |
| Unity Editor | 本機實際安裝 2022.3.62f2c1，授權已啟用；必須與本機工具鏈／Unity 專案吻合 |
| .NET SDK | 8.0.428；Mod 目標仍為 net48／C# 9，並非 net8.0 |
| Node.js／npm | 22.21.0／10.9.4；供 UI 開發，C# 模板本身不需要 npm |
| IDE | Visual Studio 或 VS Code；本機已安裝兩者，Rider 非必要 |
| Git | 本機 2.55.0.windows.3；SourceTree 可作 GUI |
| 整合測試 Mod | Presidente，Paradox ID 160999；依遊戲依賴提示加入 Custom Chirps，ID 121812 |
| 本地 LLM | 可選技術環境：llama.cpp b12359 CPU ＋ Qwen3-1.7B Q4_K_M；不是建置模板的必要依賴 |

本機硬體為 Ryzen 7 4800H、約 15.4 GiB RAM、RTX 2060 6 GiB。這不是最低配備承諾，也未完成大城市效能測試。

## 1. 安裝與檢查工具鏈

1. 安裝遊戲並透過 Steam 啟動一次，建立使用者資料。直接啟動 Cities2.exe 的本機實驗曾遇到平台初始化失敗。
2. 在遊戲的 Modding Toolchain 安裝畫面完成依賴：Unity、有效授權、Unity 模組專案、.NET SDK、C# 與 UI 模板。已有且正常的依賴不用重裝。
3. 安裝 Git 及所需 IDE；做 UI 開發時再確認 Node.js。官方下載：[.NET SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)、[Node.js](https://nodejs.org/en/download)。版本以工具鏈相容性為準。
4. 完成安裝後重新開啟 PowerShell／IDE，從 GitHub 的 Code 選單複製實際倉庫 URL，clone 或用 SourceTree 加入本機倉庫。以下命令均在倉庫根目錄執行。

本專案為私人倉庫，clone 前需以獲授權的 GitHub 帳戶登入：

```powershell
git clone https://github.com/Array417/cite-skyline-auto-govenment.git
Set-Location cite-skyline-auto-govenment
```

```powershell
git --version
dotnet --list-sdks
dotnet new list
node --version
npm.cmd --version
```

模板列表應有 `csiimod`。本倉庫已包含 C# 專案，**不要在根目錄再次執行 dotnet new 覆寫它**。PowerShell 若阻擋 npm.ps1，使用 `npm.cmd`／`npx.cmd`，不用修改全機執行政策。

## 2. 確認 CSII 環境變數

根專案與官方工具鏈讀取 **User scope**。只設定目前程序的 `$env:` 不保證 postprocessor 取得相同值。

```powershell
$csiiNames = @(
    'CSII_TOOLPATH', 'CSII_MANAGEDPATH', 'CSII_USERDATAPATH',
    'CSII_LOCALMODSPATH', 'CSII_UNITYMODPROJECTPATH',
    'CSII_MODPOSTPROCESSORPATH', 'CSII_MSCORLIBPATH',
    'CSII_ENTITIESVERSION', 'CSII_UNITYVERSION'
)
foreach ($name in $csiiNames) {
    [pscustomobject]@{
        Name = $name
        UserValue = [Environment]::GetEnvironmentVariable($name, 'User')
        ProcessValue = [Environment]::GetEnvironmentVariable($name, 'Process')
    }
}
```

檢查路徑中的檔案，而不只看變數有沒有字串：

| 變數／位置 | 應存在的內容 |
| --- | --- |
| CSII_TOOLPATH | Mod.props、Mod.targets |
| CSII_MANAGEDPATH | Game.dll、Unity.Entities.dll 等遊戲組件 |
| CSII_USERDATAPATH | 遊戲使用者資料目錄 |
| CSII_LOCALMODSPATH | 本機開發 Mod 部署目錄，通常為使用者資料下 Mods |
| CSII_UNITYMODPROJECTPATH | 工具鏈的 UnityModsProject；Library／PackageCache 需已完成初始化 |
| CSII_MODPOSTPROCESSORPATH | ModPostProcessor.exe |
| CSII_MSCORLIBPATH | 對應的 mscorlib.dll |
| CSII_ENTITIESVERSION | 必須對應實際 Unity Entities 套件及 SourceGenerators |
| CSII_UNITYVERSION | 必須對應實際安裝／註冊的 Unity 版本 |

```powershell
$toolchainPath = [Environment]::GetEnvironmentVariable('CSII_TOOLPATH', 'User')
if (-not $toolchainPath) { throw 'CSII_TOOLPATH 未設定，先檢查官方工具鏈。' }
Test-Path -LiteralPath (Join-Path $toolchainPath 'Mod.props')
Test-Path -LiteralPath (Join-Path $toolchainPath 'Mod.targets')
```

本機預設資料位置為 `%USERPROFILE%\AppData\LocalLow\Colossal Order\Cities Skylines II`；Steam 遊戲通常在 `C:\Program Files (x86)\Steam\steamapps\common\Cities Skylines II`，其他 Steam Library 請用實際位置。不要把另一台電腦的路徑寫死進專案。

受限工具可能讀不到 User 變數，但一般 PowerShell／IDE 可以；本次已遇過此差異。先核對執行帳戶與權限，不直接判定需要重裝。不要修改官方共享 Mod.props／Mod.targets。

## 3. 建置 C# 專案

先確認 Unity 版本設定一致；本機的特殊情況見下一節。首次 restore 需要網路取得所需 NuGet 參考組件，包含 SDK 使用的 .NET Framework 參考組件。

```powershell
dotnet restore AutoGovernment.csproj
dotnet build AutoGovernment.csproj -c Debug
```

輸出通常在 `bin\Debug\net48`。**官方 DeployWIP 會在建置後取代本機 Mods\AutoGovernment 目錄內容**，不要在該輸出目錄保存手動維護的檔案。此步驟不等於發布到 Paradox Mods。

若只想在工作區驗證建置，將部署重導至已忽略的目錄：

```powershell
$localDeploy = Join-Path (Get-Location).Path '.validation\local-mods'
dotnet build AutoGovernment.csproj -c Debug ('-p:LocalModsPath=' + $localDeploy)
```

已在本機驗證根專案 `AutoGovernment.csproj`：編譯、官方 IL postprocess、Windows／macOS／Linux Burst 及工作區內部署均成功，0 警告、0 錯誤。模板只有載入／卸載日誌，Burst 當時沒有實際待編譯方法；這不是三個平台已可遊玩或正式產品完成的證明。

### 本機 Unity f2／f2c1 差異

本機實際安裝及 Unity 專案是 `2022.3.62f2c1`，User `CSII_UNITYVERSION` 卻是 `2022.3.62f2`。僅改 Process scope 無法修正 postprocessor 的 User scope 查找。

以下是本機已驗證的暫時 workaround：**只在確認實際安裝版本確為 c1 時使用**；建置後無論成功或失敗都還原原值。其他電腦應使用其實際版本，並避免同時執行其他依賴這個 User 變數的建置。

```powershell
$previousUnityVersion = [Environment]::GetEnvironmentVariable('CSII_UNITYVERSION', 'User')
$localDeploy = Join-Path (Get-Location).Path '.validation\local-mods'
try {
    [Environment]::SetEnvironmentVariable('CSII_UNITYVERSION', '2022.3.62f2c1', 'User')
    dotnet build AutoGovernment.csproj -c Debug ('-p:LocalModsPath=' + $localDeploy)
    if ($LASTEXITCODE -ne 0) { throw '建置失敗，請檢查上方錯誤。' }
}
finally {
    [Environment]::SetEnvironmentVariable('CSII_UNITYVERSION', $previousUnityVersion, 'User')
}
```

請以一般使用者權限與官方工具鏈設定排查，將本機版本差異記錄於 PREFLIGHT。

## 4. UI 開發環境

本機已用官方 UI 模板完成獨立 probe：React 18.3.1、TypeScript 4.8.4、webpack 5.97.1，且通過 UI → C# → UI → C# 通訊。

**正式 UI 專案尚未納入此倉庫，根目錄沒有 package.json**，因此目前不要在根目錄執行 npm install 或 npm run build。UI 模板與試驗檔位於未追蹤的 `.validation/UiProbe`，fresh clone 不會取得它們。

建立正式 UI 時使用遊戲安裝的官方模板；在生成的 UI 專案目錄依其 package.json 操作：

```powershell
npm.cmd install
npm.cmd run build
```

該官方模板的 webpack 讀取 Process `CSII_USERDATAPATH`，輸出至其下 `Mods\<mod.json 的 id>`。若終端尚未繼承 User 值，可在該終端設定：

```powershell
$env:CSII_USERDATAPATH = [Environment]::GetEnvironmentVariable('CSII_USERDATAPATH', 'User')
```

整合時須對齊 UI mod id、C# binding 名稱及實際載入位置。技術通訊成功不代表已完成元首辦公室或雙按鈕介面。

## 5. 遊戲內開發測試

1. 建立獨立測試 playset；本機名稱為 `ag-test`。Presidente 與 Custom Chirps 經 Paradox Mods 安裝，不把其 DLL 放入 Git。
2. 本機平台標示 Presidente 1.7.4，但 active public 內容標籤是 1.7.2；整合要同時記錄外層版本、內容識別及接口簽章。不要將 beta 當作依賴或繞過其存取限制。
3. 另建或另存測試城市。政治／施工寫入前先保存獨立副本，保留原城市。
4. 建置並部署至開發 Mods 目錄後，重新啟動遊戲確認新組件載入。工作區重導建置本身不會將 Mod 裝入遊戲。
5. 檢查使用者資料下的 Logs；分別確認載入、UI 通訊、實際狀態變動、成本及重載，而不是只看 build 成功。

目前沒有正式產品自動測試套件或可安裝成品。已完成的實驗及失敗案例見 [PREFLIGHT](docs/PREFLIGHT.md)。新 clone 不會包含歷史實驗資料、已安裝 Mod 或測試存檔。

## 6. 可選：本地 Qwen 推理環境

此環境用於開發驗證，**目前不會自動接入 AutoGovernment 模板或 Presidente**。本機採 CPU、4 執行緒、2048 context、單一並行請求與 0 GPU layers；正式模型選型尚未確立。

先在工作區準備目錄，從官方固定 release 及模型發佈來源下載；這兩個下載不是每次建置都要執行。

```powershell
New-Item -ItemType Directory -Force .validation\llm | Out-Null
Invoke-WebRequest -Uri 'https://github.com/ggml-org/llama.cpp/releases/download/b12359/llama-b12359-bin-win-cpu-x64.zip' -OutFile '.validation\llm\llama-cpu.zip'
Invoke-WebRequest -Uri 'https://huggingface.co/lmstudio-community/Qwen3-1.7B-GGUF/resolve/main/Qwen3-1.7B-Q4_K_M.gguf' -OutFile '.validation\Qwen3-1.7B-Q4_K_M.gguf'

$runtimeHash = (Get-FileHash '.validation\llm\llama-cpu.zip' -Algorithm SHA256).Hash
$modelHash = (Get-FileHash '.validation\Qwen3-1.7B-Q4_K_M.gguf' -Algorithm SHA256).Hash
if ($runtimeHash -ne '837c1655e87df3b66989f4be33a792a1399c785bf259038dc75c900096683e69') { throw 'Runtime SHA256 不符。' }
if ($modelHash -ne 'e0801cbda7e2f3fd00bea4d73b53b422b14b13aa130e778f6414b6b641920b7e') { throw '模型 SHA256 不符。' }
Expand-Archive -LiteralPath '.validation\llm\llama-cpu.zip' -DestinationPath '.validation\llm\runtime'
```

如果目錄已安裝並有服務執行，先檢查現有版本，不覆寫其檔案。模型約 1.28 GB，runtime 啟動後本機實測記憶體約 2 GiB。

在獨立 PowerShell 啟動模型服務；保留該終端以便 Ctrl+C 停止：

```powershell
& '.\.validation\llm\runtime\llama-server.exe' --model '.\.validation\Qwen3-1.7B-Q4_K_M.gguf' --host 127.0.0.1 --port 18641 --threads 4 --ctx-size 2048 --parallel 1 --n-gpu-layers 0 --jinja
```

另一個終端檢查服務：

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:18641/health'
```

推理 API 為 `POST /v1/chat/completions`，測試採 `chat_template_kwargs.enable_thinking=false` 及受限 JSON schema。其格式與端點參考 [llama.cpp server 文件](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md)；模型資料見 [Qwen 原模型](https://huggingface.co/Qwen/Qwen3-1.7B) 及 [Community 量化發佈](https://huggingface.co/lmstudio-community/Qwen3-1.7B-GGUF)。下載來源及授權各自適用，不隨 Git 提供模型。

六例短政策測試中，五例只通過 action／mode／value 的窄斷言；另有錯誤調稅分類及未指定 target 幻覺。不能將此數字當成完整語意成功率。大型城市負載、長政策、否定條件與成品可靠性仍未通過。

## 7. 開發服務與本機實驗邊界

| 服務 | 本機位置 | 狀態 |
| --- | --- | --- |
| 城市 Bridge | http://127.0.0.1:8642 | 外部 CS2MCP 的研究副本；不是此 C# 模板內建服務 |
| 本地模型 | http://127.0.0.1:18641 | 依上節另外啟動，與遊戲分開執行 |

Bridge 基礎來源及固定 commit 見 [開源研究](docs/OPEN_SOURCE_RESEARCH.md)。本機 `/preflight/presidente`、`/preflight/ui` 等是自訂實驗入口，**上游原版與本倉庫 fresh clone 沒有這些功能**；不要把測試副本當正式產品依賴。服務僅綁 loopback，不開放至 LAN／公網；保留原有 Host、Origin 與寫入方法檢查。

API、日誌與可重跑測試優先用於狀態驗證；UI 外觀、橋隧及街頭事件再作畫面檢查。Computer Use 不是每次驗證的必要條件。

## 8. 常見問題

| 現象 | 排查方向 |
| --- | --- |
| Import 找不到 Mod.props | 檢查 User CSII_TOOLPATH、終端帳戶及實際檔案，不只看 Process 值 |
| 找不到 Game.dll／SourceGenerators | 核對遊戲路徑、Entities 版本及 Unity 模組專案初始化 |
| 找不到 Unity Editor／postprocess 失敗 | 核對實際安裝、Unity 專案、User CSII_UNITYVERSION；本機有 f2／f2c1 差異 |
| .NET Framework 參考組件錯誤 | 先完成 restore，核對 NuGet 網路與工具鏈；SDK 8 不等於 net48 參考組件已可用 |
| 編譯後遊戲沒有新 Mod | 檢查是否使用工作區重導、實際部署目錄、遊戲重新載入及 Logs |
| npm.ps1 被阻擋 | 使用 npm.cmd，不修改全機執行政策 |
| Git／SourceTree dubious ownership | 核對倉庫路徑與擁有者；只對已確認可信的這一個倉庫加入 safe.directory，不使用萬用字元信任 |
| Bridge／模型連不上 | 確認相應程序確實已啟動、端口及日誌；clone 本倉庫不會自動安裝這兩個服務 |

SourceTree 擁有者問題可在確認可信的倉庫根目錄執行：

```powershell
$trustedRepository = (Get-Location).Path.Replace('\', '/')
git config --global --add safe.directory $trustedRepository
git status
```

## 9. 倉庫、提交與發佈

倉庫保留規格、研究紀錄、接口 CSV 與專案源碼。`.validation/`、bin／obj、node_modules、Unity Library、日誌及本機 IDE 檔案已忽略。遊戲組件、Presidente DLL／容器、模型、存檔、登入憑證不隨 Git 發佈。

根目錄沒有正式 UI 或政府功能實作；`.validation` 中的歷史 probe／第三方研究副本亦不在 Git，不能假設 clone 後存在。接口 CSV 只含 metadata 與研究結果，第三方來源與授權見研究文件。

提交前檢查：

```powershell
git status --short
git diff --check
git diff --stat
git remote -v
```

將源碼與文件提交至既定遠端分支；模型及遊戲二進位保持排除。不要用 force push 解決未知遠端分歧。

`Properties/PublishConfiguration.xml` 與 PublishProfiles 仍為官方模板，名稱以外多項內容未完成。此處的 Git push 不等於 Paradox 發佈，開發建置亦不應執行 dotnet publish。
