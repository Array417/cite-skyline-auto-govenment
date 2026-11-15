# Presidente 選項類完整清單

來源：已安裝 public 1.7.2 的 `HomelandSecurity.Setting` metadata；平台標示 1.7.4。43 項均有 public setter，但以下沒有逐項呼叫或改動。中文用途主要依名稱與 UI metadata 判讀，未實測者不視為功能保證。當前值與一般預設值未讀取。

`Mod.Settings` 有 public getter，可作取得現有設定物件的候選；不能自行 new 一份設定物件就宣稱已改到遊戲。直接呼叫 setter 後的保存、通知及模擬後果仍需驗證。

| 屬性 | 用途 | metadata 範圍／值 | 可見性／類型 |
| --- | --- | --- | --- |
| `Chirper` | Chirper 通知模式 | Off=0、EventsOnly=1、Normal=2 | 有 UI metadata |
| `ChirperReactions` | 舊版／隱藏反應選項；與目前模式的關係待測 | true / false | UI 隱藏 |
| `CityResponses` | 城市反應選項；具體影響待測 | true / false | 有 UI metadata |
| `DashboardTextSize` | 儀表板文字比例 | min=80、max=150、step=5、unit=percentage | 有 UI metadata |
| `NotifyBanners` | 橫幅通知 | true / false | 有 UI metadata |
| `NotifyPopups` | 彈出通知 | true / false | 有 UI metadata |
| `NotifySound` | 通知聲音 | true / false | 有 UI metadata |
| `PauseOnBanner` | 橫幅通知時暫停 | true / false | 有 UI metadata |
| `UnderworldIntensity` | 地下犯罪強度 | Off=0、Quiet=1、Normal=2、Albuquerque=3 | 有 UI metadata |
| `UnderworldDeaths` | 地下犯罪死亡選項 | true / false | 有 UI metadata |
| `HeistCrews` | 劫案團隊選項 | true / false | 有 UI metadata |
| `HackerGroups` | 駭客團體選項 | true / false | 有 UI metadata |
| `UnderworldRealMoney` | 隱藏真實金錢選項；遷移／當前作用待測 | true / false | UI 隱藏 |
| `OfficeDirtyTools` | 政治不當手段工具 | true / false | 有 UI metadata |
| `RiggingTools` | 操控選舉工具 | true / false | 有 UI metadata |
| `OfficeExposureEndsTerm` | 曝光醜聞後結束任期選項 | true / false | 有 UI metadata |
| `StormAssembly` | 衝擊議會選項 | true / false | 有 UI metadata |
| `Independence` | 獨立運動強度 | Off=0、Rare=1、Normal=2、Volatile=3 | 有 UI metadata |
| `Strikes` | 罷工範圍 | Workplace=0、Sector=1、General=2 | 有 UI metadata |
| `BlockadesCloseRoads` | 封鎖是否關閉道路 | true / false | 有 UI metadata |
| `RealEconomy` | 經濟模擬模式 | Off=0、Gentle=1、Full=2 | 有 UI metadata |
| `EconomyCrises` | 經濟危機選項 | true / false | 有 UI metadata |
| `StructuralCollapses` | 結構倒塌選項；實際觸發條件待測 | true / false | 有 UI metadata |
| `RevealUnderworld` | 揭示地下組織的除錯選項 | true / false | 有 UI metadata |
| `PerformanceProbe` | 效能探測 | true / false | 有 UI metadata |
| `VerboseLogging` | 詳細日誌 | true / false | 有 UI metadata |
| `ShowPhraseNumbers` | 顯示語句編號 | true / false | 有 UI metadata |
| `DevSpawnType` | 測試生成類型；項目由動態下拉來源提供 | Int32；範圍待驗證 | 有 UI metadata |
| `DevSpawn` | 測試生成按鈕，setter 會執行動作 | true / false | 有 UI metadata；動作按鈕，非布林開關 |
| `DevGrowOrg` | 測試成長目標；項目由動態下拉來源提供 | Int32；範圍待驗證 | 有 UI metadata |
| `DevGrow` | 測試指定組織成長按鈕 | true / false | 有 UI metadata；動作按鈕，非布林開關 |
| `DevGrowAll` | 測試所有組織成長按鈕 | true / false | 有 UI metadata；動作按鈕，非布林開關 |
| `Language` | 語言 | String；範圍待驗證 | UI 隱藏 |
| `RememberLanguage` | 記住語言 | true / false | UI 隱藏 |
| `ContentNotice14Seen` | 內容提示已讀狀態 | true / false | UI 隱藏 |
| `ResizeTipSeen` | 調整大小提示已讀狀態 | true / false | UI 隱藏 |
| `WhatsNew13Seen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `WhatsNewBoomSeen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `WhatsNew14Seen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `WhatsNew15Seen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `WhatsNew16Seen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `WhatsNew1_6Seen` | 更新說明已讀狀態 | true / false | UI 隱藏 |
| `MigrationReportCities` | 城市遷移報告紀錄；格式待測 | String；範圍待驗證 | UI 隱藏 |

除錯生成／成長按鈕會改動城市，不作一般政府政策入口；已讀旗標與語言等也不應交由政府 agents 任意改動。隱藏選項可能為相容遷移保留，存在 setter 不保證仍影響模擬。
