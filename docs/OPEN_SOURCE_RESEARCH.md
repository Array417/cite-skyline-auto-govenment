# 開源工具適用性

檢查：2026-11-15。以下是候選與實查結果，並非已完成整合。

| 專案 | 用途 | 授權／證據 | 在本產品的角色 |
| --- | --- | --- | --- |
| [mayor-modder/Cities2-MCP](https://github.com/mayor-modder/Cities2-MCP) | Wiki、研究資料與本機遊戲百科查詢，Mod 範本、建置、檢查與打包 | 程式 MIT；Wiki corpus 另為 CC BY-SA 3.0，研究語料分開註明來源；README、LICENSE 及工具程式已查看 | 開發與知識研究；未提供即時起路、買地或國家政治控制，不能當自治政府的現成執行器 |
| [LancerComet/cities-skylines-2-mcp](https://github.com/LancerComet/cities-skylines-2-mcp) | 遊戲內 C# Bridge，加外部 TypeScript MCP；包含城市讀取、道路、分區、土地、財政與公共交通操作 | Apache-2.0；README、csproj、BridgeSystem、RequestHandlers 與土地操作程式已查看 | 已通過本機編譯、載入與 API 連線；施工及城市寫入仍待測，與 Presidente 的政治接口另行處理 |
| [lucarager/CS2-NetworkTools](https://github.com/lucarager/CS2-NetworkTools) | 道路幾何、連接、坡度及橋隧相關實作 | MIT；原始碼與授權檔已查看 | 工程實作參考；尚未決定是否引入依賴或重用程式碼 |

## 城市控制 Bridge 要驗證的事項

研究副本固定來源：LancerComet `72bd7e0bfa01c8d263d2a069a468b413318e8bb8`、Cities2-MCP `8432c34bbd26fdb793bf4ae4ac48574a8c7829d4`、Network Tools `5a4c8dfef850bb76bfa5c3baf75e9c07f6b3679a`。本機 Bridge 加入的 `/preflight/` 與 UI roundtrip 僅為測試修改，不屬原專案功能。

- 本機遊戲 1.6.4f1 是否能編譯及載入；原始 csproj 與程式常數的版本標示不同，需記錄實際 commit 與產物。
- ECS 讀寫是否確實在正確主執行緒／工具更新階段執行，暫停時的行為如何。
- 施工是否經原生預覽、驗證、套用流程，價格與失敗狀態是否可讀回；橋隧、交匯及道路接駁需實際測試。
- 與 Presidente 同時使用時，土地／工程與政治財政由哪一方處理；不能直接使用兩套調稅入口造成狀態不一致。
- HTTP 只綁本機，保留現有 Host／Origin／方法限制；不擴大到區網或公網。
- MCP 是工具協定，不是政府決策模型。政府 agent 可共享本地 LLM，透過有限、已驗證操作接口行動；不必為每個部門運行一個模型。

## 重用原則

研究副本保留在 `.validation/`。真正選入產品前記錄來源 commit、必要檔案、修改內容與授權通知；未選用的整個專案、語料或二進位不得混入發佈包。

Cities2-MCP 的 MIT 程式授權不涵蓋其所有語料；使用 Wiki corpus 時另外遵守相應署名及授權要求。現階段沒有安裝這個 MCP 插件，也沒有修改 Codex 或其他 AI 客戶端設定。
