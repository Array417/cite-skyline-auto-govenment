# Presidente 內部設定與擴充入口盤點

檢查：2026-11-16，Asia/Hong_Kong。對象為本機已合法載入的 public 內容，平台標示 1.7.4、內容標籤 1.7.2，遊戲 1.6.4f1。

**結論：有足夠具體入口支持繼續驗證獨立 extension，但「存在公開 setter／方法」不代表全部可任意修改或已支援穩定整合。** 本次完成完整 metadata 盤點；只有先前的草案建立已做城市寫入測試，沒有在掃描時改動其他設定。

組件 SHA256：`e60f1dd8a48de950f0d35b4f9a663f12b9ff62f6a70e43ea2e66afdb55424130`。結果僅對此 public 內容成立，不延伸至 beta 或後續更新。

## 掃描範圍與證據級別

以 .NET 標準 metadata API 讀取 DLL，沒有在工具程序執行遊戲／Presidente 程式、建立系統、呼叫 setter 或反編譯複製其實作。共檢查 2,839 個型別記錄，納入 1,773 個具名 HomelandSecurity 型別，其中 1,251 個為有效公開型別。其餘 1,066 個屬非該命名空間、編譯器生成或存取閘門型別，不列入設定盤點；沒有讀取 beta 容器或繞過存取限制。

以下數目均為該型別自行宣告的成員；不包含遊戲基底類別繼承而來的 API，也不是功能數目。159 個非 UI、名稱以 System 結尾的型別作為系統分類。其他幫助類、政策目錄及資料結構仍列入完整公開成員 CSV。

| 項目 | 數量 | 代表甚麼 |
| --- | ---: | --- |
| Mod 選項類屬性 | 43 | 30 項帶可見 UI metadata，13 項標為 UI 隱藏；30 項之中有 3 個動作按鈕 |
| 公開屬性 | 4,077 | 包括大量唯讀統計，不全是設定 |
| 有 public setter 的屬性 | 176 | 43 個選項類屬性，加上 133 個系統屬性 |
| 系統 public setter | 133 | 包含 53 個 Message 結尾欄位，另外 80 個混合規則、名稱與狀態 |
| 公開普通方法 | 6,876 | 包括查詢、操作、序列化與生命週期方法，不全是命令；其中系統方法 5,571 個 |
| 公開非 literal 欄位 | 8,441 | 多為資料記錄、集合與實體狀態，不能當作 8,441 個安全配置 |
| 數字／布林 literal 常數 | 8,820 | 包括 enum 代碼與編譯期平衡值，不能作一般執行期調整 |
| 系統／選項類非公開欄位與屬性 | 7,687 | 用來辨識內部狀態邊界，不是可用寫入 API |

證據分為：metadata 確認、遊戲內讀取成功、草案寫入成功、正式操作／後果／重載尚待驗證。不能將這幾級混為一談。

## 可以調查及使用的入口

| 領域 | 已確認入口例子 | 可對應的玩法／限制 |
| --- | --- | --- |
| 遊戲選項 | `Mod.Settings` public getter；`Setting` 的 43 個 setter | 犯罪、罷工、經濟、通知、除錯等。當前值及保存方式未逐項讀取；隱藏選項可能為相容遷移保留 |
| 開局角色 | `MovementSystem.Start`、`StartOpen`、`StartWhy` 唯讀；`ChooseStart(int,int,out string)` | `Classic=0`、`InPower=1`、`Movement=2`。有選擇方法，不代表允許中途重選或可跳過初始化 |
| 建國／制度 | `FoundingSystem.Choose`、`Skip`、`Respond`；`Chosen`、`Pending` 等唯讀 | 聯盟、立場、優先方向與建國事件候選；不能只因方法名便推定已解決主權與元首權限 |
| 國家元首 | `PresidentSystem.SetResidence`、`SetOfficeHours`、`SetGuard`、`Choose`、`SetSalary` | 對應元首實體及辦公操作候選；`President` 等結果屬性無 public setter |
| 名稱與外觀 | `CodeSettingsSystem.PresidentName`、`PresidentTitle`、政黨／部門名稱、`Portrait`；`SymbolsSystem` setter | 可改稱呼與外觀的候選。改名不會自動提升政治權限或令省級行政變成主權國家 |
| 選舉制度 | `ElectionSystem.VotingAge`、`NoRecordVote`、`OwnersOnly`、`MinIncome`、`VisaVote`、`NoUnfitVote`、`MinEducation` | 有規則 setter；應核對 `SetLaw`／`SetRule` 與立法流程，不能繞過程序或直接改總統黨籍當作選舉 |
| 選舉操作 | `SetElectionMonth`、`PostponeElection`、`ForceElection`、`Dissolve`，以及相應 Blocked 查詢 | 延後／提前選舉及解散等候選；日期、合法性、票數及後果未實測 |
| 法案 | `LegislationSystem.NewBill`、`SetText`、`SetArticle`、`SetMeasure`、`AddItem`、`Introduce`、`Withdraw`、`Repeal` | 已實測建立及查詢草案 #1；其餘提出／生效／撤銷／重載未通過 |
| 政策數值目錄 | `LegislativeCatalogue.All`、`Find`、`Open`、`Clamp`；Lever 的 `Min`／`Max`／`Step`、`Read`／`Write` | 可作政策白名單與數值範圍來源；各目錄項 ID、執行期範圍和目前值仍需從已載入遊戲匯出 |
| 緊急權及威權化 | `StandingSystem.AssumePowers`、`RevokePowers`、`UrgentDecree`、`HoldPress`、`ReleasePress`、`BanParty`、`LiftBan`、`AppointMembers` | 雙提交模式與集權流程候選。各 Blocked／權限檢查及後果必須保留，未實測前不宣稱任意命令均可強制 |
| 行政命令 | `ExecutiveOrderSystem.OrderBy`、`Act`、`Rescind`、`RevokeLicence`、`SeizeTitle` 等 | 有實體目標、撤銷與歷史記錄候選；Entity、操作代碼、成本和司法反應均需驗證 |
| 內閣 | `CabinetSystem.Appoint`、`AppointPriced`、`DemandResignation`、`Rename` | 任命與辭職流程候選，避免直接改人事記錄而漏掉成本／政治後果 |
| 財政 | `FiscalSystem.Range`、`FinanceBlocked`、`LayFinance`、`BudgetBlocked`、`LayBudget`、`LayHoliday` | 原有財政流程候選；不能同時直接改原生稅率再讓 Presidente 重複處理 |
| 股票／基金 | `MarketsSystem.FundAct`、`TargetShare`、`Buy`、`Sell`；`ExchangeSystem.SetPayout`、`BuyBackCorporation` 等 | 原有持倉、股息與市場操作候選，必須核對帳戶、報價、成交及重複執行 |
| 司法規則 | `JusticeSystem.ExamAct`、`ContinuingEducationAct`、`ProBonoAct`、`AdmissionFee`、`InspectionMonths`、`PdPositions`、`Sessions`、`PleaCap`、`NoDealsViolent`；`LawsuitSystem.QualifiedImmunity` | 法律、資源和司法制度候選；不代表可任意指定判決 |
| 司法操作 | `JusticeSystem.AppointJudge`、`RemoveJudge`、`MakeChief`、`AppointDa`、`SetToughness`、`SetPleaPolicy` 等 | 人事與檢控流程候選；不能把方法存在當成合法、成功或已付款 |
| 民情／社會 | `StandingSystem.GroupShare`、`GroupApproval`、`GroupTurnout`、`GroupEligible`；`SocietySystem` 公開查詢及操作 | 群體反應及資料來源候選；禁止由 LLM 直接捏造民意或改統計結果冒充模擬 |
| 地方政府 | `LocalGovernmentSystem.SetLanguagePolicy`、`SetRuleBudget`、`AppointAdministrator` 等 | 地方權限及資源流程候選；其地區獨立方法不能直接當作全國玩家主權切換 |
| 其他系統規則 | `GuardSystem`、`CommerceSystem`、`CibSystem`、`FinanceSystem`、`BondSystem` 等 setter | 國民衛隊、商業、電訊、金融、債券等入口；完整欄位見系統 setter 清單，不由名稱推定全部效果 |

以上候選方法的完整簽章及參數名稱見 CSV；整數索引／字串代碼的有效集合、各 Entity 的生命週期及精確語意仍未全部解析。

## 最適合 extension 的公開結構

- `StandingSystem.LawProviders`／`ProvideLaws`：法律提供者，含 Prefix、Count、Get、Write、Blocked，可作新增法律規則的候選。
- `StandingSystem.AfterDivision`：表決後回呼候選，可連接本 Mod 等待中的政策工作。
- `StandingSystem.MoodModifiers`：群體民情修正候選，需保留原因及避免與原有反應重複計算。
- `StandingSystem.ExtraAyes`：額外贊成票計算回呼，語意及何時呼叫未實測；不能當作任意補票工具。
- `JusticeSystem.AddStakeholderSource`、`ExchangeSystem.SetSurveyListener`：其他公開擴充入口候選，介面契約及解除註冊方式待驗證。

`readonly List<...>` 只限制欄位重新賦值，不代表集合內容不可改。僅註冊與移除自己的回呼，不能 Clear 全部或反覆載入造成重複掛載。這些是已存在的公開結構，不是作者承諾的穩定 SDK。

## 無法直接當成設定的內容

- `Start`、`MandateUntil`、`President`、議席、民情統計等多為唯讀結果；應使用原有操作流程，不以改 backing field 當正式整合。
- `PresidentParty` 等雖有 public setter，仍屬重要狀態。直接賦值未必更新選舉紀錄、職位、議會及其他系統。
- 私人欄位通常是快取、ECS 查詢、城市狀態及序列化資料。反射能找到名字不代表可維護地改動；沒有把它們開放給 LLM。
- literal 常數包括 enum 編碼、容量與機率門檻；編譯期常數可能已內嵌到呼叫處。它們不是一般 runtime setter，不能承諾靠反射改一處就生效。
- `Message` setter 多是狀態文字，修改文字不會執行其描述的政策。
- 公開 `SetDefaults`、`Serialize`、`Deserialize`、更新方法等屬生命週期入口，不當作政府政策工具。
- 動作按鈕 `DevSpawn`／`DevGrow`／`DevGrowAll` 雖以 Boolean setter 表達，賦值可能立即改動城市，不是三個一般開關。

## 完整清單與重現方式

| 檔案 | 內容 |
| --- | --- |
| [SETTINGS.md](presidente/SETTINGS.md) | 全部 43 項 Mod 選項、中文用途、enum／slider 範圍及隱藏標示 |
| [SYSTEM_SETTINGS.md](presidente/SYSTEM_SETTINGS.md) | 80 個非訊息系統 setter；另外 53 個訊息欄位保留在 CSV |
| [settings.csv](presidente/settings.csv) | 43 項原始屬性與 UI metadata |
| [system-setters.csv](presidente/system-setters.csv) | 133 個系統 setter |
| [properties.csv](presidente/properties.csv) | 4,077 個公開屬性及 getter／setter 可見性 |
| [methods.csv](presidente/methods.csv) | 6,876 個公開普通方法的簽章及參數名稱 |
| [system-methods.csv](presidente/system-methods.csv) | 其中 5,571 個系統方法 |
| [fields.csv](presidente/fields.csv) | 8,441 個公開非 literal 欄位及 readonly／static 標示 |
| [constants.csv](presidente/constants.csv) | 8,820 個數字／布林 literal 值，含 enum；不可當成執行期設定 |
| [private-members.csv](presidente/private-members.csv) | 7,687 個系統／選項類非公開欄位及屬性名稱、型別、可見性；非操作白名單 |
| [types.csv](presidente/types.csv) | 1,773 個納入型別與成員數 |
| [inventory-summary.json](presidente/inventory-summary.json) | 雜湊、範圍與可核對統計 |

本機完整 metadata JSON 位於 `.validation/Presidente.inventory.json`，包含上述具名型別的公開／非公開成員；CSV 是按用途篩選的索引，不是全部私人實作。完整 JSON 及研究工具依既有約定保留在本機 `.validation`，不隨 Git 發佈。CSV 僅含接口 metadata，沒有遊戲存檔、模型或第三方 DLL。

本機重跑使用既有 MetadataProbe 的 `--inventory` 模式及 `summarize-presidente.mjs`。工具已成功編譯，並以實際組件檢查掃描總數、43 項選項、Classic enum 值、NewBill 簽章、按鈕讀寫區別、CSV 回讀數目及排除範圍。Custom attribute 解析沒有回報錯誤；外部 enum 的 underlying type 在無外部型別資料時暫按 Int32 解碼，這一限制已記入 JSON，不能將 attribute metadata 當成目前值或一般預設值。

## 尚需遊戲內驗證的部分

執行期政策目錄的每項 ID／範圍／目前值、完整開局與執政權限、原有政府自主決策與 agents 的責任邊界、setter 引起的通知與歷史、實際政治後果、各能力授權與版本限制，以及重載保存，仍未逐項通過。需要有範圍的測試城市實驗，不能自動逐一觸發數千個公開方法來宣稱全部驗證。

目前城市實測只證明八個政治系統可讀，以及草案 #1 建立後可獨立查詢；詳見 [PREFLIGHT](PREFLIGHT.md)。整合方向仍以獨立 extension 為優先，正式執行器只開放逐項驗證過的能力。
