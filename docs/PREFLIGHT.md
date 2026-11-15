# 開發前驗證紀錄

更新：2026-11-16，Asia/Hong_Kong。對應 [完整成品 PRD](../PRD.md)。

**整體狀態：尚未全部通過，正式產品功能未開始實作。** 已完成下列工具鏈與套件調查；遊戲內政策讀寫、存檔、施工及模型並行測試仍在進行。內部實驗不作分階段交付。

## 已取得的證據

| 項目 | 結果 | 證據與限制 |
| --- | --- | --- |
| 遊戲 | 1.6.4f1，可開啟主選單及 Paradox Mods | 本機 version 檔及遊戲畫面 |
| .NET／Node | SDK 8.0.428、Node 22.21.0 | CLI 實查 |
| 模板編譯 | 通過獨立 net48 測試 | `.validation/ToolchainProbe.csproj` 編譯現有 Mod.cs；非成品 |
| 根專案實際建置 | AutoGovernment.csproj 通過編譯、IL postprocess 與 Windows／macOS／Linux Burst | 2026-11-16：暫時將 User Unity 版本改為實際的 c1，僅以 LocalModsPath 將部署重導工作區，0 警告、0 錯誤，約 30 秒。finally 已還原 f2；模板 Burst 無實際方法，未證明跨平台遊戲可用 |
| 官方 postprocess | 通過 IL 處理與 Windows Burst | 暫時將 User `CSII_UNITYVERSION` 改為實際安裝的 `2022.3.62f2c1` 後成功，0 警告、0 錯誤；finally 已還原原值 `2022.3.62f2` |
| 環境讀取 | 受限工具與完整環境結果不同 | 受限環境讀 User scope 時回傳空值；完整權限唯讀檢查確認 User 路徑實際存在，不能誤判為需要重裝 |
| Unity 安裝 | 2022.3.62f2c1 | Unity Hub editors-v2.json、專案 ProjectVersion.txt、HKLM 安裝路徑三者吻合；工具鏈版本設定為 f2，確有差異 |
| UI 工具鏈與通訊 | 官方範本成功建置；遊戲內雙向通訊通過 | `.validation/UiProbe`，webpack 5.97.1；UI ping → C# ValueBinding ACK → UI confirm → C# 的完整 roundtrip，CS2MCP.log 及 `/preflight/ui` 均為 True；非正式產品介面 |
| 硬體 | Ryzen 7 4800H，8 核 16 執行緒，15.4 GiB RAM，RTX 2060 6144 MiB | CIM 與 nvidia-smi 實查；顯示卡 AdapterRAM 欄位不能正確代表 6 GB，採 nvidia-smi |
| Presidente 安裝 | 已加入獨立 `ag-test` playset | 原 Initial playset 保持空白；同時加入宣告依賴 Custom Chirps（ID 121812） |
| Presidente 版本 | 平台標示 1.7.4／建議遊戲 1.6.* | 遊戲內模組詳情；不能沿用搜尋索引 1.6.0 當最新版本 |
| 實際 public 內容 | public 標籤 1.7.2；識別 `1d59d5a557fd5dd53f633fbd9277b7d9538f31560ca1dc215d85d0f1c06bd976` | 只讀已下載的 public 容器；外層載入器與內容版本分開記錄，沒有處理 beta 內容 |
| Presidente 載入 | 政治、選舉、議會、司法、市場等系統註冊成功 | Modding.log 與 HomelandSecurity.Mod.log；註冊成功不等於所有功能與本 Mod 已整合 |
| Presidente 接口 | 已在遊戲程序讀取 8 個已建立的系統 | 立法、行政命令、議會、選舉、財政、市場、司法與社會系統的可用性及部分屬性已讀回；當時在主選單，多數值為初始值。政策寫入、城市效果及存檔仍待驗證，不能視為穩定 SDK |
| 原生城市接口 | 已確認類型及方法存在 | Game.dll metadata 包括 TaxSystem.SetTaxRate、CityServiceBudgetSystem.SetServiceBudget、MapTilePurchaseSystem.PurchaseSelection、NetCourse／CreationDefinition；尚未實測施工 |
| 道路源碼 | 已下載 Network Tools 作研究 | MIT 授權，尚未重用或部署；仍須驗證橋隧及成本流程 |
| 城市控制 Bridge | 編譯與遊戲載入通過，loopback API 可連線 | 原版及帶技術 probe 的 build 均 0 警告、0 錯誤；實際載入與 `/ping` 成功。具體施工與城市寫入仍待測 |
| API 輸入邊界 | 實測通過 | GET 建立草案被拒 405、跨 Origin 請求被拒 403、主選單無效寫入被拒 409；`.validation/test-bridge.mjs` 留有可重跑檢查 |
| 本地推理 | Qwen3-1.7B Q4_K_M 已載入；六例中五例通過原有窄斷言，不代表完整語意成功率 | 官方 llama.cpp b12359 CPU，4 執行緒、2048 context、0 GPU layers；約 2020 MiB working set，遊戲主選單同時開啟。六例約 1844–3547 ms；斷言只核對 action／mode／value。「命令天空立即落雨」錯誤解析為調稅；另有住屋補助案例自行填入未指定的 target「香港」，原斷言未捕捉。語意可靠性未通過，不代表完整 agent 或大城市壓力測試 |

模型檔案大小 `1282439328` bytes，SHA256 `e0801cbda7e2f3fd00bea4d73b53b422b14b13aa130e778f6414b6b641920b7e`，與 LM Studio Community 發佈 metadata 相符。早期未約束測試中，模型曾替未指定金額的法案填入數字；技術測試已改成依操作類型約束參數、提交模式由可信 UI 值鎖定，並驗證結果，不依賴模型口頭遵守。

最新結果保存在 `.validation/llm/validation.results.json`。六例僅是探索性測試；不支援要求仍會被誤分類，正式執行器必須另外核對操作對象、有效範圍及城市狀態，不能將模型分類直接當作執行授權。

2026-11-16 覆核更正：`test-local-model.mjs` 的 passed 沒有驗證 target 或完整條件，因此先前「5／6」僅是窄斷言結果。住屋補助案例的 target「香港」並未由該次輸入或系統提示提供，不能視為正確。正式評測需覆蓋對象與地區的來源、缺省範圍、多項目標、支出上限、否定與禁止條件，不應直接用六例推算成品可用率。

後續已載入城市「丹斯塔布」；API 回報 `Game`、`cityLoaded: true`、人口 0、資金 2,000,000，八個 Presidente 系統仍可讀取。使用者當時畫面停在開局身份選擇，之後的開局紀錄及草案寫入結果見下節。建立／載入城市可由使用者手動操作，不以 Computer Use 作為必要前提。

### 後台草案寫入實測（2026-11-16，香港時間）

- 本機日誌記錄 `MovementSystem: the city begins as start 0, party 0.`，確認已有開局選擇事件；目前 API 尚未暴露完整職位／權限快照，不能只靠開局編號推定所有行政權。
- 寫入前已透過遊戲保存 `AG-Preflight-before-2c43fac07a`，並確認 `.cok` 存在且 ZIP 容器可完整開啟。
- 透過已載入 public 組件的 `LegislationSystem.NewBill` 建立 `AG-Preflight validation draft`，回傳 id `1`；BillsCount 由 `0` 變 `1`，獨立查詢 `Find(1)` 回報存在。
- 沒有呼叫 Introduce、執行措施或撥款；測後人口仍為 `0`、資金仍為 `2,000,000`，不等於通過正式政策生效與財政驗收。
- 已保存第二份 `AG-Preflight-after-2c43fac07a` 並確認容器可讀。兩個 SaveGameData 的 UTF-8 原文搜尋均未找到草案標題；這個搜尋不能判斷序列化內容是否保留，仍須實際重載後查詢 id，不能宣稱存檔持久化已通過。
- API 沒有現成讀取完整執政權限或載入存檔的路由；這是目前驗證工具的缺口，並非已證明 Presidente 沒有相關能力。沒有注入任意執行代碼或改寫原 Mod DLL。
- 本機完整回應及另存路徑記於 `.validation/presidente-write-run.json`；模型／第三方二進位及城市存檔不納入 Git。

## 尚未通過的必要驗證

完整設定與接口 metadata 盤點已完成，見 [內部設定盤點](PRESIDENTE_SETTINGS_AUDIT.md)。43 個選項類屬性及 133 個系統 public setter 均有列表；其中大量欄位為狀態、訊息或除錯入口，不能因此縮減以下操作／效果／重載驗證。

1. 正式選擇整合／部分整合／自建：須讀回政治狀態，提交可追蹤政策，核對結果與重複執行，再保存、重載。
2. 正式產品 UI 的外觀、雙提交按鈕與互動驗收；技術用 C#／UI roundtrip 已通過，不能代替產品介面驗收。
3. 遊戲內實際買地、劃區、建路、升級、建橋／隧道，核對幾何、造價、連通及失敗處理。
4. 自建與整合狀態的存檔、版本限制及跨城市隔離。
5. 模型的更多政策、模糊／惡意輸入、逾時／中斷與重試、大城市並行資源量測；目前只通過短政策解析與主選單並行測試。
6. 完整社會、政治、金融與司法規則的操作映射及可測試範圍，不能只因找到方法名便宣稱已驗證全部功能。

## 實驗產物及邊界

- `.validation/` 僅放工具鏈測試、官方 UI 範本、metadata 工具、第三方研究程式及模型；已列入 `.gitignore`，不是成品或可發佈依賴清單。
- MetadataProbe 使用 .NET 標準 metadata API 讀取組件，不在工具程序載入執行 Game.dll 或 Presidente 組件。
- 只為檢查接口讀取 public 容器，未繞過 beta／付費存取，也未將第三方二進位內容納入產品源碼。
- 研究與驗證可以使用測試城市及一次性 probe；正式交付仍須達成 PRD 的全部完成條件。
