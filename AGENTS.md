# HTML Slides 原始碼

## 建置與檢查

- 用 Python 3 執行 `python build.py`。產物是 `HTML-Slides.html`，瀏覽器透過 `file://` 就能讀取。
- 用 Node.js 22 或更新版本執行 `node --test`。測試涵蓋編輯狀態、匯出、媒體資料和發布檔啟動。
- 修改 `src/` 後再建置。產物包含 HTML、CSS、JavaScript 和 JSON，無需 npm、外部函式庫、網路或本機伺服器。
- 每個 `.mjs` 檔有一個 default export。依賴在 `main.mjs` 組裝並傳入各模組。新增模組時，同步更新 `build.py` 的模組清單。

## 修改入口

| 要修改的內容 | 入口 |
|---|---|
| 從瀏覽器開啟既有簡報 | `open-deck.mjs`、`html-format.mjs` |
| 頁面結構、樣式、範例 | `shell.html`、`style.css`、`demo-deck.json` |
| 資料版本、物件預設值、百分比邊界 | `model.mjs` |
| 預設範本與預覽選單 | `slide-templates.json`、`template-picker.mjs` |
| 編輯與播放的頁碼顯示 | `editor-view.mjs`、`playback.mjs` |
| 選取、修改、群組復原／重做、已儲存狀態 | `store.mjs` |
| 增刪、排序、複製、匯入或替換物件 | `edit-actions.mjs` |
| 畫布雙擊文字、物件選取、拖曳、縮放 | `canvas-input.mjs` |
| 工具列、欄位、按鍵與選單事件 | `editor-input.mjs`、`main.mjs` |
| 投影片清單與右側設定 | `editor-view.mjs` |
| 投影片渲染，含縮圖、編輯區、播放 | `slide-view.mjs` |
| 播放翻頁、全螢幕、鍵盤與控制列自動隱藏 | `playback.mjs` |
| 媒體檔案型別、來源、原資料夾重新連結 | `media.mjs`、`filenames.mjs` |
| 另存介面、資料匯出、ZIP、HTML 自我序列化 | `save-dialog.mjs`、`export.mjs`、`zip.mjs`、`html-format.mjs` |

## v1 資料契約

| 記錄 | JSON 欄位 |
|---|---|
| Deck | `version: 1`、`title: string`、`slides: Slide[]`、`assets: Record<AssetId, Asset>`。至少一張投影片。可含 `presentation: {showPageNumber, showEditorPageNumber}`，兩個布林值缺漏時皆為 `true`。 |
| Slide | `id`、`name`、`color`（十六進位）、`pattern`、`objects: SlideObject[]`。後面的物件顯示在上層。可含 `backgroundAsset: AssetId`，引用 `kind: "image"` 的背景圖，移除後沿用 `color` 和 `pattern`。可含 `mediaBox: [x, y, w, h]`，用於投影片第一個影音物件。 |
| SlideObject | 共用欄位為 `id`、`type`、`x`、`y`、`w`、`h`。位置與尺寸以 1280 × 720 畫布的百分比表示。 |
| TextObject | `type: "text"`，另外有 `text`、`size`（畫布像素）、`weight`（400、700、900）、`color`、`align`（left、center、right）。 |
| MediaObject | `type: "media"`，`asset` 存放 Asset ID。圖片位置可為負值，尺寸可超過畫布的 100%。渲染時由畫布裁掉超出頁面的部分。 |
| Asset | `kind`（image、video、audio）、`mime`、`name`、`path`（相對 HTML 的 `media/` 路徑）、`bytes`。單一 HTML 匯出另有 Base64 Data URL 欄位 `data`。 |

- 文件以 `#deckData` 的 JSON 為準。編輯時由 `store.edit()` 寫入文件狀態。連續打字與拖曳使用群組識別值，使一段連續操作只佔一筆復原紀錄。
- 瀏覽器的 File、Blob、Object URL 和載入時的內嵌 Data URL 由 `media.mjs` 管理。復原紀錄只包含投影片與媒體中繼資料。複製投影片共用 Asset ID，只替換投影片和物件 ID。
- 匯出只打包仍被引用的資產。HTML 序列化要轉義 JSON 中的 `<`。ZIP 的媒體名稱要以 UTF-8 編碼，並保留原始檔案位元組。
- 舊版另存的簡報自帶舊版編輯器。用新版的「開啟簡報」讀取 `#deckData`，可延續內容及使用新版功能。原檔如附外部影音，選取原 HTML 所在的資料夾後可重新連結。開發時也可用 `python build.py --from-html 舊簡報.html --output 新簡報.html` 換成新版編輯器並帶入原資料。外部影音以原相對路徑引用，請把新 HTML 放在原 `media/` 旁。

## 擴充投影片範本

- 在 `src/slide-templates.json` 增加一筆，給它唯一的 `id`、選單上的 `label` 和 `description`、背景 `color`、文字陣列 `text`。可給選用的 `name`、`pattern` 或 `mediaBox`。
- 每個文字物件使用 `text`、`box: [x, y, w, h]`、`size`，可給選用的 `color`、`weight` 或 `align`。座標、樣式與投影片契約同義。
- 選單縮圖與新增投影片使用相同的 `model.slideFromTemplate()`。生成的物件皆有新 ID，匯出後保存實際的物件資料。
- 執行 `node --test` 和 `python build.py`。建置時會把範本資料放進單一 HTML。
