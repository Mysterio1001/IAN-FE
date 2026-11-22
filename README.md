# ian-fe｜Ian 前端開發 Skill

**版本：1.1.0**

將設計圖或需求轉成符合 Ian 慣例的 iMES 前端程式碼，適用於使用 Vue 2 與 Element UI 的目標專案。安裝只需一個 `SKILL.md`，不需要參考專案的業務程式碼。

## 讓 agent 幫你安裝

先用 coding agent 開啟**要開發的目標專案**，將本 skill 的 `SKILL.md` 附給它（或提供它可讀取的來源路徑），然後貼上這段：

```text
請將我提供的 ian-fe skill 安裝到目前專案，
```

## 如何叫用

| 工具 | 在目標專案的對話中輸入 |
| --- | --- |
| Codex | `$ian-fe 依照附圖建立唯讀查詢頁，API URL 尚未提供。` |
| Claude Code | `/ian-fe 依照附圖建立唯讀查詢頁，API URL 尚未提供。` |
| Cline | `請使用 ian-fe skill，依照附圖建立唯讀查詢頁，API URL 尚未提供。` |

若目前對話未發現新 skill，先在同一專案開新對話。

## 版本更新歷史

### 1.1.0

- 新增可寫入檔案白名單：主代理、子代理與驗證修正皆限於本次頁面模組及專用 API；修改 Axios 等共用程式須先取得明確授權。
- 統一 API 函式版型：僅引用 `request` 與必要的服務前綴，直接 `return request({...})`。缺 URL 時保留 `// IAN TO DO`，由頁面暫不呼叫。
- 加入去識別化的主檔、報表與作業站布局範例。預設不新增自訂 class；必要時使用小駝峰，如 `resultViewport`。

### 1.0.0

- 首版提供 Ian 風格的 Vue 2／Element UI 前端生成規則、六步開發流程與單檔 skill 安裝包。

本 skill 僅供昊聲訊息技術有限公司內部使用。如需回報問題，請寄信至 <wuguanru@semi-tech.com>。

**作者：Ian Wu**