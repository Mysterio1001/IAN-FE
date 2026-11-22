# Ian 前端生成規則

適用於依設計圖新增 iMES 頁面。先找行為最相近的範例，再比對外觀：同樣有表格，「編輯一筆主檔」、「點數量看明細」和「掃碼後更新任務」是三種不同的資料流。以下檔案連結是參考樣本的位置，不是產出程式時的 import 路徑。

## 1. 先選模組，再決定拆檔

| 設計圖的主要操作 | View 樣本 | 對應 API |
| --- | --- | --- |
| 查詢、分頁、新增／檢視／編輯／複製／刪除 | [nc-reason-type](references/views/nc-reason-type/index.vue) | [nc-reason-type.js](references/api/nc-reason-type.js) |
| 同類主檔、不同欄位 | [nc-real-error-code](references/views/nc-real-error-code/index.vue) | [nc-real-error-code.js](references/api/nc-real-error-code.js) |
| 刷入識別碼、選取資料列、綁定／解綁 | [link-and-unlink](references/views/link-and-unlink/index.vue) | [link-and-unlink.js](references/api/link-and-unlink.js) |
| 刷碼、啟動任務、持續更新結果 | [l11-auto-screw-driver-operation](references/views/l11-auto-screw-driver-operation/index.vue) | [auto-screw-driver-operation.js](references/api/auto-screw-driver-operation.js)，沒有 `l11-` |
| 條件查詢、點表格數量開不同明細 | [mo-material-management-report](references/views/mo-material-management-report/index.vue) | [mo-material-management-report.js](references/api/mo-material-management-report.js) |

- **檔名與職責**：模組目錄、業務元件及 API 檔案採英文 kebab-case。頁面入口固定為 `index.vue`，同層 `init.js` 以 named export 提供 `initSearch`、`initForm`、`tableOptions` 或業務專用的 `*Options`、選擇視窗設定。只定義該頁用到的項目。頁面可變資料以 `copyJSON(initForm)` 等方式初始化，不直接修改共用初值。
- **主檔**：`index.vue` 持有查詢條件、列表、分頁、目前列及模式。`ui/search-ui.vue` 接收 `searchForm`、綁定欄位並 emit `search`／`clear`，由父層查詢或重設。`dialog/main.vue` 共用一份表單處理 `new`、`view`、`edit`、`copy`；不拆成四份表單。非新增時依 `handle` 取明細，複製時清掉 `handle`。儲存成功 emit `afterSave`，父層重查；關閉 emit `update:visible`，重設表單並在 `$nextTick` 清驗證。
- **報表**：搜尋區仍獨立，但 `searchReset` 清條件、資料與分頁，不等於主檔的清除後重查。不同明細有各自的資料來源與欄位時，才拆 `dialog/material-receipt.vue` 這類業務命名的視窗。父層傳 `currentRow` 和 `visible`，子層監看 `visible` 載入／清除明細。不要為唯讀報表生成 CRUD 按鈕。
- **作業站**：兩個 WIP 樣本都是 `index.vue` 配 `init.js`。刷碼、焦點、選取列、燈號與作業結果共享同一流程時，留在頁面協調；重用現成顯示／輸入元件，不因畫面分成幾個框就拆幾個 `.vue`。有監聽、計時或輪詢才加入對應的停止與卸載清理。

## 2. Template 骨架與元件選擇

- **布局用 `el-row`／`el-col`**：頁面分區、欄位並排及按鈕列都用 `el-row` 包 `el-col`，透過 `gutter`、`span`、`justify` 控制間距、寬度與對齊，不用多層 `div` 模擬行列。例外是有明確用途的容器，例如下述單一根節點，或需要獨立捲動的區域。
- **選元件不限於這五個樣本**：生成前可到目標專案的 `src/components` 尋找合適元件。找到候選時，核對其 props、events、slots 與實際呼叫方式；介面符合需求就重用，不另寫一份。若無法存取該目錄，不臆造元件或 import 路徑。
- 使用 Vue 2 單檔元件及 Options API：`<template>` → `<script>` → 必要時 `<style scoped>`。`export default` 內依用途放 `components`、`props`、`data()`、`computed`、`watch`、生命週期、`methods`；不用為湊齊順序加空區塊。採兩空白縮排，JS 字串用單引號，通常不加分號。多屬性標籤逐行排列。沿用 `.sync`、`slot="label"`、`v-slot:header` 等專案語法，不改成 Vue 3 或 Element Plus。
- 列表頁以 `<div class="execution">` 包住 `<basic-container>`，依閱讀順序放搜尋區 → 操作按鈕（如有）→ `el-table` → 靠右 `el-pagination`；業務 dialog 放在容器外、同一根節點內。作業站改放操作輸入、訊息／燈號、資料與結果區，不硬套搜尋與分頁。
- 表單用 `el-form` 綁 `:model`、`ref`、`@submit.native.prevent`；`el-row`／`el-col` 排版，`el-form-item` 的 `prop` 對應 `v-model` 欄位。搜尋欄常見 `:gutter="15"`、`:span="8"`，依設計調整。標籤用既有 `tooltip`，有語系鍵就用 `$t(...)`，英文業務名稱可沿用已確認文案；不要編造 MES 語系鍵。
- 有選擇視窗的輸入使用 `el-input` 的 suffix 圖示，以 `@click.stop` 開 `input-select-dialog`。傳 `:modal-obj`、`:modal-fetch-fun`、`:modal-show.sync`，於 `inputDialogSelected` 回填指定欄位。不是每個輸入都要開窗。
- 表格欄位在 `init.js` 描述 `{prop, label, width, minWidth}`；template 用 `v-for="col in tableOptions"`、`:key="col.prop"` 產生一般欄，序號、操作鈕或可點擊的數量另用欄位／slot。常見 `border`、表頭與內容置中、長值 `show-overflow-tooltip`。分頁使用 `page.current`、`page.size`、`page.total`（列表初值 `1 / 20 / 0`）；搜尋／換頁大小回第一頁，換頁則保留條件。
- CRUD 新增按鈕常用 `primary`、`small`；列操作用 `mini` 圓形圖示鈕。表單視窗用 `el-dialog`、`v-el-drag-dialog`、footer 按鈕，避免點遮罩或 Esc 誤關。作業站才視需求使用 `text-msg`、`traffic-signal`、`qr-code` 或 `Scanner`。生成前確認目標專案已註冊全域元件／指令，或依樣本 import 並在 `components` 註冊。

## 3. 函式與狀態命名

| 責任 | 命名與呼叫方式 |
| --- | --- |
| UI 事件入口 | camelCase，優先 `handle` + 動作，如 `handleSearch`、`handleSave`、`handleDel`、`handleSelectionChange`。多個按鈕只是切換同一視窗時才共用 `handleClick(type, row)`。 |
| 讀取與更新 | 頁面 `getList()` 組查詢條件、呼叫 API `fetchList(payload)`、更新 `tableData`；`getCurrent(row)` 取編輯明細。站點可用 `loadRackInfo`、`loadTask` 說明載入對象，不把資料取得藏在樣式函式內。 |
| UI 輔助與布林 | `showDialog`、`closeDialog`、`clearSelection`、`dataFormatter`；判斷用 `isRowSelectable`、`isAllCpnCompleted`。表單用 `form`／`searchForm`，顯示狀態用 `customerVisible`、`tableLoading` 等。衍生狀態用 `computed`，有副作用的開窗載入才用 `watch`。 |
| 跨元件介面 | 沿用最接近模組的事件名稱；例如 CRUD 為 `clear`，報表為 `searchReset`。父層監聽名須與 `$emit` 完全一致。樣本同時出現 `SearchUI`、`searchUi`；新頁在同一模組內統一即可，不複製 `dailogVisible` 一類拼字錯誤。 |

## 4. 卡控：哪些在呼叫 API 之前做？

**判準：只靠當前畫面即可確定、而且需求已明示的輸入或互動條件，才在前端擋。需要查資料、可能被其他人改變、或決定操作是否獲准的條件，以後端結果為準。**

| 情境 | 前端行為 | 留給後端 |
| --- | --- | --- |
| 必填、長度、格式 | 在 `el-form` 宣告 `rules`，欄位對應 `prop`，`handleSave` 先 `$refs.form.validate`，有效才送出。僅對有規格的欄位做大寫轉換、`maxlength`。 | 驗證收到的 payload、唯一性、參照資料有效性。 |
| 搜尋條件連動 | 報表樣本只啟用選中的 Line／Operation 欄；搜尋前檢查至少一個條件、需要的起迄日期是否齊全。這是該報表的條件，不套到所有查詢。 | 資料範圍、查詢權限、實際結果。 |
| 唯讀、按鈕與刪除 | `view` 停用表單、不顯示儲存且不開選擇窗；依既有 `permissions` 決定按鈕是否顯示；刪除先 `$confirm`，取消不送出。 | 每次異動重新授權；是否可刪、是否已被引用，不以隱藏按鈕代替。 |
| 刷碼或流程按鈕 | 用 guard clause 依序檢查空值、是否已選列、是否正在執行。例：`handleStart` 缺 CSN／Tool ID 即 `return this.msg(...)`；可用**已載入**的工具選項、CPN 清單提早提示選錯。送出期間鎖定相應操作。 | 現場設備是否仍可用、能否建立／提交任務、綁定衝突及最終狀態。不要從畫面快照自行宣告成功。 |

失敗時不要走成功分支或關閉待修正的表單。頁面級操作用 `showLoading()`／`hideLoading()`；明細表可用 `v-loading="tableLoading"`。用 `finally` 收回 loading。部分樣本有空 `catch`，不視為必要格式：先確認共用 request 是否已提示錯誤，避免重複彈窗，也避免無聲失敗。

## 5. Style 與註解

- 先用 Element 元件的 `size`、`type`、`gutter`、`span`、`align` 類排版能力。單點寬度、邊距、對齊可用 inline style（如 `style="text-align: right"`）。重複或多層樣式留在該元件的 `<style lang="scss" scoped>`；簡單連結外觀用 `<style scoped>` 即可，不需樣式就省略。不要為還原單頁改全域樣式。
- 搜尋區 `.custInput` 用於收緊表單間距；作業站 `.wrapper`、`.scroll` 處理容器與捲動；報表的可點數量以粗體、藍色、底線辨識。動態表格狀態用 `statusCellStyle` 等 callback；不要把個別頁的固定高度、負 margin 或 `!important` 當成全站預設。
- Template 在區塊或意義不明的欄位前加短註解，如 `<!-- 搜尋欄位 -->`、`<!-- 主表格 -->`、`<!-- Reason Type -->`。JS 用 `//`：可用 `// API`、`// TOOL` 分組，例外流程註明原因（如 `// 複製時清空handle`）。`init.js` 中語系鍵不直觀時，欄位行末補 `// 扭力` 等業務名稱。API 函式前用一行中文說明動作，必要時附英文術語；不逐行翻譯程式。

## 6. API 檔案的邊界與命名

- **未提供 API URL 時仍建立 API 檔與函式骨架**。依已知操作建立對應的 named export，保留 request 設定；每個缺 URL 的函式在設定中寫 `url: '', // IAN TO DO`。不要猜路由、套用樣本 URL 或用服務前綴包空字串。URL 補齊前不對空路徑發送請求；HTTP method 與參數位置也只採用已確認的契約。

API 檔只負責傳輸：從 `@/router/axios` 匯入 `request`，從 `@/api/prefix` 匯入服務前綴。每個 endpoint 對應一個 `export function`，直接回傳 request；loading、提示、表單重設和結果呈現都留給呼叫端。以下是主檔樣本的真實查詢，不是可套用到任何模組的路由公式：

```js
import request from '@/router/axios'
import {iep} from '@/api/prefix'

// 查詢
export function fetchList(params) {
  return request({
    url: iep('/z/nc/reason/type/page'),
    method: 'get',
    params
  })
}
```

- 主檔 CRUD：`fetchList(params)`、`getById(params)`、`saveObj(data)`、`delObj(handle)`；樣本 GET 用 `params`，儲存 POST 用 `data`，刪除 DELETE 用 `params: {handle}`。`getById` 在這裡實際傳 `handle`，不要只憑函式名改成 `id`。
- 報表用 `rpt`，主查詢 `fetchList(query)`，明細如 `getMaterialReceipt(query)`，選擇窗如 `moFetch(query)`；以 `params: query` 送出。主檔與兩個 WIP 樣本用 `iep`。前綴不等於 view 所在的 `prd`／`wip` 分類。
- 作業站用業務動詞，如 `link(data)`、`unlink(data)`、`startScrewTask(data)`、`getScrewTaskResult(params)`。**方法與參數位置依契約**：讀取 Rack 的 `getRackInfo` 是 POST + `data`，`clearScrewTask` 是 PUT + `params`。不能從 `get` 字首或按鈕文案猜 HTTP method。
- View 透過目標專案 `@/api/mes/...` 路徑匯入，先組 `payload` 再 `await`。分頁列表常取 `res?.data?.data` 的 `records`／`total`；單筆與任務結果按各自契約解析，不硬套分頁格式。API 檔通常與模組同名，但以實際 import 為準。

## 7. 收到設計圖時的交付檢查

1. 列出畫面區塊、欄位、按鈕、開窗與操作順序，為每個區塊選一個上表中的參考來源。沒有畫出的功能不要因樣本有就新增。
2. 分清「圖上已知」與「待確認」：顯示名稱不等於 payload key；必填、權限鍵、endpoint、回傳結構、語系鍵與狀態轉移以現有契約或需求為準。查不到便列出要確認的具體問題，不自創後端行為或用假成功代替串接。
3. 核對父子 props／events、`init.js` 欄位、API 入參及回傳對應。逐項試走適用流程：搜尋回第一頁、檢視不能寫、複製不帶原識別碼、取消不送出、失敗解除 loading、關窗再開沒有舊資料；作業站另確認重複觸發與離頁清理。
4. 對照待確認的 API 清單，確認每個缺 URL 的操作已有函式骨架、空白 URL 與 `IAN TO DO` 註解；交付時列為待串接，不列為已通過連線測試。