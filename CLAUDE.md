# CLAUDE.md

這個檔案給 Claude Code(claude.ai/code)使用,提供 repo 工作脈絡。

## 專案概述

**BanG Dream! 徽章圖鑑**(`liar-chang.github.io/bandori-badges`)
日本動漫《BanG Dream!》系列的粉絲罐徽(缶バッジ)資料庫網站。資料量約 2,700+ 筆,持續每日新增。

技術棧:
- 前端:純 HTML / CSS / JavaScript(無框架)
- 資料儲存:Supabase(使用者資料、清單、收藏)+ GitHub `badges.json`(徽章主資料)
- 部署:GitHub Pages
- PWA:有 manifest.json,可加到主畫面

---

## Repo 檔案結構(關鍵檔案)

```
/
├── index.html              # 前台(主網頁)~5400 行
├── blog.html               # 開發日誌(時間軸式,純 HTML 文章,共用圖鑑的深色模式)
├── admin.html              # 後台管理工具 ~3140 行
├── announcement-tool.html  # 公告產生器
├── badge-crop.html         # 徽章截圖/裁切工具(單張+批次,🔍 Hough 自動偵測,🗂 裁剪記錄資料集)
├── badges.json             # 徽章主資料(2700+ 筆)
├── manifest.json           # PWA 設定
├── favicon-*.png           # 各尺寸 icon(180, 192, 512;16/32 走 favicon.ico 內含尺寸)
└── images/                 # 徽章圖片(由徽章 ID 為檔名)
```

### blog.html 開發日誌(2026-05-30 起)

明羽的開發心得永久存放處。緣由:Threads 帳號曾被 Meta 無預警關閉,心血歸零;改放自己 GitHub 上,平台再也關不掉。

- **結構**:時間軸式,每篇 = 一個 `<article class="post">`。新文章複製整個 article 區塊,貼在「新文章貼這行下方」的註解下,最新放最上面
- **視覺**:完全呼應 index.html(同 CSS 變數、Boogaloo logo 字型、粉紅漸層 header、共用 `localStorage['theme']` 深色模式 + `images/bg.png` 深色背景)
- **入口**:index.html footer「分享此網站」按鈕旁的「📝 開發日誌」連結
- **目前**:純 HTML 手寫文章(不是 Markdown 動態載入)。明羽說「先做一個頁面放這篇就好」,未來文章變多再考慮升級成 .md 載入系統
- **文章列表**(最新在上):
  1. 2026.05.24「開站兩個月——關於這個徽章圖鑑的開發心得」→ 配圖 `images/blog-001.jpg`(妃那白底閉眼托腮圖)
  2. 2026.04.21「一切的起點——這個徽章圖鑑是怎麼來的」(起源故事:從蒐集愛音徽章→Google 試算表→VPN 切 9 國轉檔→抽到 TAIPEI LIVE 門票祭品文→Claude Pro 三天架站) → 配圖 `images/blog-002.jpg`(羊寶藍光舞台圖)
- 所有文末 `<img>` 都有 `onerror` 自動隱藏保護,圖未上傳不會破版
- **圖片管理**:blog 配圖命名 `blog-XXX.jpg`,走 GitHub Desktop 推(`.gitignore` 已放行 `!images/blog-*.jpg`);徽章圖仍走 admin 後台。兩套流程互不干擾
- **✅ 已上線(2026-05-30 push)**:兩篇文 + 兩張配圖 + footer 入口 + og 標籤多語言 + Threads 連結換新帳號,全部已推上 GitHub Pages

---

### badge-crop.html 裁剪記錄資料集(2026-05-31 起)

**緣由**:明羽問「能不能透過多次訓練增強裁剪/搜尋工具的準確度」。釐清:現有工具是**傳統演算法(Hough / pHash),不會自學**,重複使用不會變準;真正能讓它變準的是**累積「正確答案」**(human-in-the-loop),之後拿去重調參數或訓練模型。這個功能就是在攢那份 ground truth 資料。

- **掛載點**:單張 `doCrop()`、批次 `cropBatchAll()` —— 每次裁切自動記一筆(fire-and-forget,失敗只 `console.warn`,絕不擋裁切流程)
- **存哪**:IndexedDB(DB `badgeCropLog` / store `records`,自增 id)
- **每筆內容**(座標一律**原圖像素**):
  - `source`:`{name 原始檔名, w/h 原圖尺寸}`
  - `thumb`:來源圖縮圖 JPEG blob(長邊 ≤ 1280,品質 0.82)+ `thumbW/thumbH`
  - `finals`:你最後採用的框(ground truth),`[{shape,cx,cy,hw,hh}]`(圓/方時 hw=hh=r)
  - `autos`:自動偵測「**原始提議框**」,沒跑自動偵測則 `null`。單張取自 `_singleDetectCircles[_singleDetectIdx]`;批次取自 `_batchAutoSnapshot`(在 `runBatchAutoDetect` 推 `batchCrops` 時同步快照,`selectFile`/`clearAllBatchCrops` 時清空)
- **UI**:header「🗂 裁剪記錄」按鈕(單張+批次都看得到)→ 面板可**預覽縮圖 / 匯出 ZIP / 清空**。單張會顯示「位移 Xpx · 半徑差 Xpx」= 自動 vs 你手調的差距
- **匯出 ZIP**(複用已載入的 JSZip):`manifest.json`(全部中繼資料,含原圖座標)+ `thumbs/<id>.jpg` + `README.txt`(欄位/座標換算說明)。檔名 `crop-dataset-<筆數>-<ts>.zip`
- **關鍵函式**:`cropLogRecord()`(記一筆)、`cropLogOpen/Add/GetAll/Count/Clear()`(IndexedDB)、`openCropLog/renderCropLogList/exportCropLog/clearCropLogConfirm()`(UI)、`updateCropLogBadge()`(角落數字)
- **未來可接**:拿匯出的資料集做離線①評估命中率 ②系統化重調 Hough 參數(解決「合成測試測不出真實失敗」的老問題)③訓練偵測模型(YOLO-tiny / fine-tune)。以圖搜圖功能未來若做,同一份資料也用得上
- ⚠️ 這是**維護工具功能,不寫進公告**(訪客看不到)

---

## 編程規範與偏好

### 修改 code 的工作流(嚴格遵守)

1. **先確認方案再動手**,避免燒 token
2. **SQL/code 一段一段給**,不要一次大量輸出
3. **改前先 grep 影響範圍**(同一個字串可能在多處出現)
4. **改完用 `python3 -c "html.count('{')..."` 驗證大括號平衡**
5. **每次改完輸出檔案 + 簡短改動摘要**,不要長篇贅述

### 不要做的事

- **不要憑印象瞎掰**(欄位名、函式名、行號、聲優、樂團對應、暱稱):不確定就 grep / 看實際 code
- **不要在沒實證下推論「業界趨勢」當建議**(SNS 遷移狀況、hashtag 用法、PWA 行為等需要查證再講)
- **不要過度道歉、過度關懷**:錯了就承認直接修
- **不要主動建議重構或大改**:除非使用者問,否則維持既有架構

### 命名與格式約定

- 中文回應,允許混技術術語但解釋清楚
- 偏好短回應、分點、避免長段落
- 使用 markdown 但不過度堆 emoji
- 改 code 用繁體中文寫註解(對齊既有 code)

---

## 資料模型(badges.json)

每筆徽章物件結構:

```json
{
  "id": "001",                   // 編號(string,通常 4 位數)
  "ext": "png",                  // 圖片副檔名(jpg/png/webp)
  "band": "MyGO!!!!!",           // 樂團(必填)
  "character": "高松燈",          // 角色名(可空,例如「其他」類型)
  "series": "Blu-ray付生産限定盤", // 版本/工藝(自由輸入)
  "year": "2025年11月13日",       // 發售日期(字串,可能只有年/年月/年月日)
  "event": "MyGO!!!!! 1st Single「迷星叫」", // 活動名稱(完整)
  "short_event": "1st Single「迷星叫」",     // 活動簡稱(明羽自己寫,前台卡片顯示用)
  "type": "角色本身",             // 類型,5 選 1
  "size": "56mm圓形",             // 尺寸,固定選項
  "country": "日本",              // 販售/活動地
  "url": "https://...",          // 來源連結
  "note": "...",                 // 前台會顯示的備註(可空)
  "admin_note": "...",           // **後台備註,前台不顯示**(可空)
  "is_lottery": false,           // 抽獎取得標記(2026-05-16 加),前台「隱藏抽獎徽章」toggle 會排除 true 的徽章
  "updated_at": "2026-05-09...", // 任何儲存就更新
  "image_updated_at": null       // 換圖時間戳(尚未實作,規劃中)
}
```

### 固定選項

**樂團(band)** — 12 個 + 「其他」:
```
Poppin'Party / Afterglow / Pastel*Palettes / Roselia / Hello, Happy World!
Morfonica / RAISE A SUILEN / MyGO!!!!! / Ave Mujica / 夢限大MewType
millsage / 一家Dumb Rock! / 其他
```

**類型(type)** — 5 選 1:
```
角色本身 / 角色相關 / 角色聲優 / 角色合影 / 其他
```

**尺寸(size)** — 圓形類:
```
56mm / 57mm / 58mm / 56-58mm(混合) / 100mm / 76mm / 75mm / 65mm / 60mm / 55mm
54mm / 50mm / 44mm / 32mm / 30mm / 25mm / 15.2cm / 15cm
```
**尺寸 — 非圓形**:
```
橢圓形 / 方形 / 心形 / 撥片 / 名牌 / 金屬 / 約80mm布偶胸章 / 刺繡布章 / 未確定
```
- **「布製」前台篩選群組**(2026-05-30):前台側欄把「約80mm布偶胸章」(立體毛絨 ぬいぐるみ)+「刺繡布章」(平面刺繡 ワッペン)合併成「布製」群組顯示(像「56-58mm圓形」群組那樣)。後台建檔時選具體值。`刺繡布章` 多語言 = ワッペンバッジ / Embroidered Patch / 와펜 뱃지
- 改動位置:index.html(SIZE_FABRIC_VALS / _SIZE_FABRIC_VALS / toggleSizeGroup `__fabric__` / SIZE_TRANS / SIZE_ORDER)、admin.html(9 處 size 選單與陣列 + EXP_SIZE_TRANS)

**販售/活動地(country)**:日本 / 台灣 / 韓國 / 中國 / 新加坡 / 其他

---

## 多語言架構(index.html)

支援 4 種語言:**繁中(預設) / 日文 / 英文 / 韓文**。**絕不加 zh-CN 簡中**(立場決定,不要建議加)。

### 翻譯機制三層

1. **LANGS 物件**(L1660~):介面文字(label / 按鈕 / 提示) — 4 個語言各一份
2. **資料層對照表**:
   - `BAND_NAME_MAP`:樂團名翻譯
   - `CHAR_TRANS`:角色名翻譯
   - `SIZE_TRANS`:尺寸翻譯
   - `COUNTRY_TRANS`:販售/活動地翻譯
   - `TYPE_TRANS`:類型翻譯
3. **取得翻譯函式**:`getBandName(b.band, currentLang)` / `getCharName` / `getSizeName` / `getCountryName` / `getTypeName`

### 加新欄位選項時(例如新尺寸/新類型)的更新清單

新增任一資料層選項,**至少要動以下位置**:

**index.html:**
- 對應 TRANS 表加翻譯(4 語言)
- 側欄篩選對應的常數(例如 `SIZE_ROUND_OTHERS`、`TYPE_ZH_LIST`)
- 渲染端的 ROUND_SIZES_SET 之類分類用的 Set
- 匯出工具的 SIZE_ORDER / TYPE_ORDER

**admin.html:**
- 編輯 modal 的 `<select>` option(L290-)
- 批次模式的 select option(L390-)
- 新增 modal 的 select option(L620-)
- 批次卡片內嵌的 select(L820-)
- L1186 內嵌陣列(編輯模式)
- L1365 `BULK_SELECT_OPTIONS`
- L2593 `EXP_SIZE_TRANS`(匯出工具用)
- L2619 `SIZE_ORDER`(匯出工具排序)

⚠️ **加完務必驗證所有列出位置都有改**,漏一處會出現「後台選了但前台不顯示」之類的問題。

---

## 已知技術慣例

### 1. `data-l-key` 機制(動態翻譯 HTML label)

任何 HTML 元素加 `data-l-key="xxx"` 屬性,語言切換時自動套 `LANGS[currentLang].xxx`:
```html
<label data-l-key="series">版本/工藝</label>
```
語言切換邏輯在 L1817 附近(`document.querySelectorAll('[data-l-key]').forEach`)。

### 2. 卡片簡稱欄位優先序

前台卡片顯示「活動簡稱」時,**永遠先讀 `b.short_event`,空才走 `getEventShortName(b.event)` 截短 fallback**。
不要再回到「直接呼叫 `getEventShortName`」的舊寫法 — 那樣會無視明羽建檔的 short_event。

### 3. 卡片發售日期顯示策略

格式:`2026 (2026年7月15日)` — 主顯示年份,完整日期淺灰色括號輔助。
**只有年(無月日)的資料不顯示括號**。判斷:`/月|日/.test(b.year)`。

### 4. 搜尋欄位

L1179 搜尋對比的欄位包含:
```js
[b.id, b.character, b.series, b.event, b.short_event, b.band, b.note]
```
不含 `admin_note`(後台備註不應出現在前台搜尋)。

### 5. PWA 設定

- `manifest.json` 位於 repo 根目錄,**theme_color 是 `#ff69b4`**(Poppin'Party 粉紅)
- 沒有 service-worker(無離線快取)— 加 service-worker 是規劃中項目

---

## Supabase 整合

使用者登入後可使用:
- **collections**:逐顆標記蒐集
- **lists**:自訂清單(手動加入徽章)
- **smart_lists**:智慧清單(存篩選條件,動態套用)

### 表 user_id 型別不一致(注意)

```
lists.user_id        = text
smart_lists.user_id  = uuid
collections.user_id  = uuid
```

跨型別 join 要 `::text` 轉換。

---

## 公告產生流程

### announcement-tool.html

獨立工具,輸入:
- 日期、新增/更新資料(繁中)、功能調整(繁中)
- X 第一/二則**日文**內容
- X 第一/二則**韓文**內容
- 額外 hashtag(日韓共用)

輸出:
- Threads 版(繁中,單一 post)
- X 日文 第一則(新增資料)+ 第二則(機能調整)
- X 韓文 第一則(추가 데이터)+ 第二則(기능 조정)

**自動分格(splitForX):**
- 任何單則內容超過 280 字 → 按 `\n` (通常 = `・` 列點邊界)greedy pack 自動切多則
- 第一則 包含 `URL + 標題 + 內容片段 + (1/N) + hashtags`,**字數計入 hashtags**
- 後續則 純內容片段 + `(N/M)` 標記,**不再加 URL/hashtags**
- 切多則時每則 UI 都有獨立 textarea + 字數顯示 + 複製鈕

**固定 hashtag:**
- 日文:`#バンドリ #BanGDream #缶バッジ図鑑`
- 韓文:`#뱅드림 #BanGDream #뱃지도감`

### 字數計算(關鍵)

X 字數規則跟 grapheme 不同,要用 **`countX()` 函式**:
- 半形 ASCII / 半形片假名 = 1 字
- 全形 CJK = 2 字
- URL 固定 23 字(X 自動短縮)
- 上限 280

Threads / Bluesky 用 grapheme 計數(`text.length`),上限分別 500 / 300。

---

## 跨平台發文策略(明羽既定)

| 平台 | 語言 | 同步方式 |
|---|---|---|
| X(Twitter) | 日文 / 韓文 | 日文跟 Bluesky 同步;韓文獨立發(2026-05-24 起加入韓文 X) |
| Bluesky | 日文 | 跟 X 同步 |
| Threads(脆) | 繁體中文 | 獨立發 |

**目前未發英文版本到社群**(雖然徽章資料 4 語言都有翻譯)。

### Hashtag 規範(實證確認)

| 樂團 | Hashtag |
|---|---|
| Poppin'Party | `#ポピパ` |
| Afterglow | `#afterglow` |
| Pastel*Palettes | `#パスパレ` |
| Roselia | `#Roselia` |
| Hello, Happy World! | `#ハロハピ` |
| Morfonica | `#Morfonica` |
| RAISE A SUILEN | `#RAS` 或 `#RAISEASUILEN` |
| MyGO!!!!! | `#MyGO` |
| Ave Mujica | `#AveMujica` |
| 夢限大MewType | `#ゆめみた` |
| 萬用 | `#バンドリ` `#BanGDream` `#缶バッジ図鑑` |

**Hashtag 規則**:後面遇到符號(`*` `/` `!` `'` 空格 `,`)即切斷,所以 `#MyGO!!!!!` 實際只是 `#MyGO`,要直接寫成 `#MyGO`。

---

## 文案寫作通則

### 一律用繁體字,絕對不要用簡體字

跟明羽對話、寫任何文案、改 code 註解、寫 commit message —— **全部用繁體中文,不准出現簡體字**。

- ❌ 不要寫「资料」「网页」「图片」「档案」「确认」「这个」「资料夹」
- ✅ 要寫「資料」「網頁」「圖片」「檔案」「確認」「這個」「資料夾」
- 即使在講解技術、回答問題時也一樣,**任何情況都不切簡體**
- 明羽自己用簡中工具(小紅書/微博)做研究是另一回事,但 Claude 的輸出**永遠繁體**
- (網站介面本來就不加 zh-CN,見下方「設計決策 #1」)

### 繁體中文一律用全形標點

寫**繁體中文**文案時(公告、社群貼文、bio 等),標點符號**全部用全形**,不要用半形。

| 用這個(全形)✓ | 不要用(半形)✗ |
|---|---|
| `，` | `,` |
| `。` | `.` |
| `！` | `!` |
| `？` | `?` |
| `：` | `:` |
| `；` | `;` |
| `「」『』` | `""''` |
| `（）` | `()` |

**例外:**
- **程式碼 / 檔名 / URL**:維持原樣,不要把 `console.log` 改成 `console。log`
- **數字後面的中文標點**:照樣全形 — `共 36 張，三個產品`
- **括號內全是英數**:可以維持半形 `(BOX)`、`(Soweit)`,但**括號裡有中文就要全形**
- **日文/英文/韓文文案**:照各語言自己的標點規範,不適用本規則

---

## 公告寫法(每日)

寫公告時:

- **四語言**(繁中/日文/英文/韓文),分別輸出
- **只輸出兩個區塊**:【新增/更新資料】+ 【功能調整】(直接貼進 `changelog.json` 用)
- **不要加** SITE_URL、日期標頭、`【図鑑更新✨】` 包裝(announcement-tool 會處理)
- 範例格式:
  - 繁中:`(新增 N 張)`
  - 日文:`(N枚追加)`
  - 英文:`(N added)`
  - 韓文:`(N장 추가)`
- `(無)` `(なし)` `(None)` `(없음)` **即使空也保留標題**
- 額外輸出 X 用日文 hashtag 區塊
- 🚫 **跨日去重(鐵則,明羽 2026-05-31 交代)**:【新增】**絕對不能含已公告過的徽章編號**。匯出工具會因同步問題把昨天已發的徽章再列一次。**每次寫公告前必做去重**:
  - ① 比對下方「**已公告徽章 ID 帳本**」+ 前一兩天的 `changelog.json` / 已發社群貼文的產品名
  - ② 交叉驗:查 `badges.json` 的 `created_at`,若落在已公告日期 → 已發過
  - **除非是「更新資料 / 更新圖檔」**,否則【新增】出現重複編號**一律排除**
  - ③ 寫完公告後,**務必把這次新公告的 ID 範圍補進帳本**(否則下次又會漏抓)
  - 實例:5/31 匯出把 5/30 已發的 Poppin'Party Fan Meeting Tour 2019!(#3626–3655,created_at 2026-05-30)又列一次 → 已排除
- **後台/維護工具的功能調整不寫進公告**:`admin.html` 內部按鈕、`badge-crop.html` 批次裁切、`announcement-tool.html` 介面等都是給明羽自己用的,**不寫在面向使用者的【功能調整】內**。判斷準則:**改的東西使用者(訪客)會看到/用到嗎?** 看不到的就不寫。
  - ✅ 該寫:前台篩選邏輯、側欄分類、樂團名翻譯、卡片行為、尺寸選單、隱藏抽獎徽章 toggle
  - ❌ 不寫:badge-crop.html 任何改動、admin.html 編輯 UI / 批次上傳 / 同步流程、announcement-tool.html 任何改動、CLAUDE.md 改動
  - ⚠️ **新增尺寸/分類「即使這批還沒對應徽章資料」也要寫進【功能調整】**——它是告知性的(讓使用者知道以後有這類就歸這)。前例:5/15 寫了「尺寸新增『44mm 圓形』」、5/30 該寫 32mm 卻一度漏掉。凡 size 選單 / 樂團分類 / 篩選群組有新增,一律寫。

### 已公告徽章 ID 帳本(去重用,2026-05-31 起記錄)

寫公告前查這張表;新清單的 ID 已在表內 → 從【新增】排除(更新類不在此限)。**每寫完一篇,就把新範圍補到表底。** 新徽章一般都拿新的(較大)ID,所以「最高已公告 ID 以下」原則上都發過;但後台有跳號補洞功能,偶爾會有小 ID 的新徽章,故仍以實際範圍 + `created_at` 為準。

| 公告日期 | 已公告 ID 範圍 | 張數 |
|---|---|---|
| 2026/05/30 | #3579–3655(RAS アーティスト ライブver. 3579–3598、其餘 5/30 產品 3599–3625、Poppin'Party Fan Meeting Tour 2019! ×2 3626–3655) | 77 |
| 2026/05/31 | #3656–3742(劇場版 FILM LIVE 3656–3665、ナゾトキ 3666–3675、Roselia vol.3 3676–3695、RAS vol.3 3696–3715、Roselia×RAS 合同ライブ 3716–3735、記念ワッペン 3736–3737、SHOW BY ROCK 3738–3742) | 87 |

**目前最高已公告 ID = #3742(截至 2026/05/31)。** 下次新清單若出現 **≤ #3742** 的編號,先當重複嫌疑、逐一查證再決定排除。

### 每月一號:推薦歌曲更新通知(明羽既定)

明羽每月 1 號會換 `song.json` 的推薦曲(前台右下角浮動播放器,欄位 `ytId` / `title` / `artist` / `spotify`)。**每月 1 號的公告要多加一行「本月推薦歌曲已更新」**:
- 這是**內容更新通知**,不歸【新增資料】也不歸【功能調整】
- 放在公告**最上面**(header 之下、【新增】之上),四語並列,一行帶過
- 想更吸睛可帶出當月曲名 / 歌手(問明羽當月選曲),否則用通用版「本月推薦歌曲已更新」
- 範例(六月、通用版):
  - 繁中:`🎵 六月推薦歌曲已更新!點右下角播放器聽聽看~`
  - 日文:`🎵 6月のおすすめ曲を更新しました!右下のプレイヤーからどうぞ~`
  - 英文:`🎵 June's featured song is now live — tap the player at the bottom-right to listen!`
  - 韓文:`🎵 6월 추천곡이 업데이트됐어요! 오른쪽 아래 플레이어에서 들어보세요~`

### 輸出版型(明羽指定,以下是預設格式,不要簡化)

每次公告都按這個版型輸出 — **5 個獨立的 code block**,各自一個語言/區塊:

````
依今天 N 新增/更新寫公告。<本日有無特別說明>

【zh-TW】
```
【新增】
・XXX(新增 N 張)
【功能調整】
(無)
```

【ja】
```
【新規】
・XXX(N枚追加)
【機能調整】
(なし)
```

【en】
```
【New】
・XXX (N added)
【Feature Adjustments】
(None)
```

【ko】
```
【추가】
・XXX (N장 추가)
【기능 조정】
(없음)
```

【X 用日文 hashtag】
```
#<額外的樂團/角色 tag>
```
````

**hashtag 規則:**
- 固定 3 個 `#バンドリ #BanGDream #缶バッジ図鑑` **公告產生器 announcement-tool.html 內建會自動加**,Claude 寫公告時**不要再寫進「X 用日文 hashtag」區塊**,只列當天額外的樂團/角色標籤即可
- 如果當天沒有特別的樂團/角色標籤要加,「X 用日文 hashtag」區塊可以省略不寫,或寫「(無)」

重點:
- 開頭一行是給明羽看的 meta 說明 (例「依今天 36 新增寫公告。三個產品都是新項目,無跨日重複。」)
- **不要**輸出 Threads 版整段(SITE_URL+日期),那是 announcement-tool.html 的事
- **不要**輸出 𝕏 第一/第二則的完整成品,那也是 announcement-tool.html 的事
- 只輸出 4 語言原文 + X 日文 hashtag,讓明羽直接複製貼進 changelog.json 對應欄位

---

## 設計決策(不要動的事)

### 1. 不加 zh-CN 簡體中文

立場決定。明羽自己使用簡中工具(小紅書/微博)做研究是另一回事,**網站介面不加**。

### 2. 不重寫成原生 App

明羽是公務員,業餘時間有限。原生 App(iOS/Android/Flutter)的工程量幾個月起跳,不適合。
**目前路線是 PWA**(網頁 + 加到主畫面),已上線。

### 3. 不過度設計

明羽偏好「**穩定運作**」勝過「**新潮花俏**」。除非有實際使用者反饋或數據支持,**不主動提案改架構**。

### 4. 列表檢視(table view)留著但不擴充

實證:幾乎沒人用列表(明羽自己也不用),但保留供少數使用者。**不為列表檢視做新功能,但也不刪**。
追蹤機制已加(L2362 setView 時 localStorage 累計),日後決定是否砍。

### 5. detail modal 缺欄位的設計

detail modal 只在「列表檢視點某列」會開啟。已補齊 d-size / d-type / d-country 三個欄位(原本 setText 對不存在元素 throw 錯誤)。

---

## 已知技術債(尚未處理)

### 1. image_updated_at 機制(待實作)

需求:後台換圖時要能識別,寫入 `image_updated_at` 欄位,讓公告能區分「新增 / 改資料 / 換圖」。

設計:
```
- onModalImgSelect 觸發時 → modalImgChanged = true(flag)
- 批次模式換圖時也標 flag
- saveSingle / saveBatch 偵測 flag → 寫入 image_updated_at = now,清除 flag
- 沒換圖則不更新 image_updated_at
```

### 2. announcement-tool 三類分欄(待實作)

把現有「新增/更新資料」textarea 拆成:
- 【新增】
- 【更新資料】
- 【更新圖檔】

### 3. admin 列表視覺標記(待實作)

最近 7 天內換過圖的徽章,在管理列表顯示視覺標記(例如相機圖示)。

### 4. search placeholder 多語言(尚未做)

`L.searchPlaceholder` 還沒做,目前 placeholder 寫死繁中「角色、版本、活動...」,日韓英切換不變。

### 5. ~~卡片視圖開啟 detail modal 入口~~（**已評估,不做**,2026-05-11）

2026-05-11 曾實作 🔍 放大鏡按鈕（commit 已 push 到 production），但同日 revert。

**原因:**
- 就地展開（點卡片）跟 detail modal 顯示的欄位高度重疊
- 當初考量的「日期格式不夠完整」問題,已透過 `2026 (2026年5月3日)` 灰字格式在**展開**與 **modal 都統一**了
- fav/collect 入口卡片上本來就有 ★ / + 按鈕,modal 重複入口反而冗餘
- 缺乏明顯增值,屬於過度設計

**未來不要主動建議加**。除非使用者明確說「卡片視圖點不到 detail modal 很不方便」,才重新評估。

### 6. ~~PWA 階段 2:離線快取~~（**已決定不做**，2026-05-11）

明羽決策:**跳過,不主動實作**。理由:「這年頭誰手機會沒網路啊」。
PWA 安裝到主畫面/全螢幕/icon/theme_color 等不靠 SW 的功能已具備。離線快取的維護成本(快取版本管理、使用者卡舊版、debug 痛點)大於實際效益。

未來不要主動建議加 service-worker。除非有具體使用者反饋「我需要離線瀏覽」,才重新評估。

### 7. 夢ノ結唱(框架已實作 2026-05-11,首批 10 顆已入 badges.json #2775-#2784)

採方案 C+:資料層當第 13 樂團、前台獨立呈現。

**已完成:**
- `BAND_LIST` / `BAND_COLORS` / `BAND_ORDER` 加入「夢ノ結唱」(顏色 `#a0c8bc` 薄荷灰青,呼應官方視覺)
- `CHAR_MAP['夢ノ結唱']` = [POPY, ROSE, Pastel, Halo, Aver](全部 5 個,即使後 3 個尚無徽章)
- `CHAR_TO_BAND` 加入 5 個角色→夢ノ結唱
- `CHAR_COLORS` 加 5 個角色顏色
- **前台側欄獨立呈現**:`INDEPENDENT_BANDS = new Set(["夢ノ結唱"])` + 新 `<div id="independent-section">` 區塊,跟「樂團」section 分開顯示。標題「✨ 獨立計畫」4 語言皆有翻譯
- **後台列表 ID 旁加 ✨ icon** 視覺區分(`b.band === '夢ノ結唱'` 時)
- **admin_note 自動填衍生來源**:`autoFillYumeAdminNote()` 在 saveSingle/saveBatch/saveTextBatch 都會呼叫(admin_note 為空才填)
  - POPY ← 戶山香澄 (Poppin'Party)
  - ROSE ← 湊友希那 (Roselia)
  - Pastel ← 丸山彩 (Pastel*Palettes)
  - Halo ← 弦卷心 (Hello, Happy World!)
  - Aver ← 三角初華 (Ave Mujica)

**待處理(使用者操作):**
- 補入 `year` 欄位(發售日期,xlsx 沒提供)
- 透過 admin 上傳 10 張圖(對應檔名 2775.png-2784.png)然後同步到 GitHub

**未來新增徽章:** 只要 band 填「夢ノ結唱」、character 填 5 個拉丁名之一,admin_note 會自動填衍生來源,前台側欄會出現在「✨ 獨立計畫」section。

---

### 8. 重要背景樂團 + 跨團合照(2026-05-21 起)

**重要背景樂團:** `BACKGROUND_BANDS = new Set(["Glitter*Green","CRYCHIC","Sumimi"])`
- 前台側欄獨立 section「🎬 重要背景樂團」(`background-section`),跟主要 12 樂團、獨立計畫平行
- 樂團顏色:Glitter\*Green `#01DFA5`、CRYCHIC `#8cb3d4`、Sumimi `#BBFF77`
- 目前只支援「全團」單一角色項 (`CRYCHIC全團` 等),個別角色徽章未來再加

**跨團合照(`band="跨團合照"`):**
- 用於跨越多個樂團的合照徽章(例:CRYCHIC 五人合照、各團聯動)
- 顏色 `#9e9e9e` 灰色
- **不算進** character 衍生樂團(`badgeBands()` 短路) — 例如 band=跨團合照 + character=CRYCHIC全團 的徽章**只**會出現在「跨團合照」section,不會跑進 CRYCHIC section
- 理由:跨團合照已有獨立分類,再灌進每個相關樂團會造成 sidebar 計數失準 + 重複出現

**跨企劃合照(`band="跨企劃合照"`,2026-05-31 起):**
- 用於 **BanG Dream 角色 × 其他企畫(他作品 / IP)角色**的合照徽章。跟「跨團合照」(BanG Dream 內部跨樂團)刻意分開
- 顏色 `#78909c` 藍灰;四語 `跨企劃合照 / コラボフォト / Cross-Project Photo / 콜라보 사진`
- **行為完全比照跨團合照**:`badgeBands()` 一併短路(`b.band === '跨團合照' || b.band === '跨企劃合照'`),只出現在自己的 section、不展開 character 衍生樂團;sidebar 由 buildSidebar 的 catch-all 自動帶出;admin 自動預設複數角色模式
- **合作(非 BanG Dream)人物怎麼存**(明羽既定:跨企劃極少見,能搜尋就好):**只記名稱當純文字**,直接寫進 `character` 欄(例 `戶山香澄 × 初音ミク`),搜尋找得到即可。**後台輸入方式(2026-05-31 起)**:在**複數角色模式**下,BanG Dream 角色照常勾選,合作人物打進新增的「**其他企劃人物**」自由輸入欄(`#f-char-multi-extra`,用 `、` 或 `x` 分隔,如 `シアン、雪平宇宙`);存檔時自動合併進 `character`(實際分隔符是小寫 `x`,例 `戶山香澄xシアンx雪平宇宙`),編輯回填時勾不到的名字會自動回到自由輸入欄、不會被吃掉(`updateMultiCharValue` 合併 + `openModal` 回填)。⚠️**絕對不要**把合作人物加進 `CHAR_TO_BAND`(index)/ `CHAR_MAP`(admin)/ `CHAR_COLORS` —— 否則會冒出幽靈角色 chip、空樂團 section。BanG Dream 角色才是建檔範圍,合作角色只是可搜尋的附帶文字
- **新增同類「特殊 band」的鏡像清單**:index.html = `BAND_COLORS` + 兩份 `badgeBands` 短路 + `BAND_NAME_MAP`;admin.html = `BAND_COLORS` + `BAND_LIST` + `CHAR_MAP`(填 `[]`)+ 複數模式條件 + 兩份 `BAND_ORDER_LIST` + `BULK_SELECT_OPTIONS.band` + `EXP_BAND_COLORS` + 3 個 `<option>` 下拉

**badgeBands() 函式:** 集中決定「一個徽章該歸在哪些樂團 section」,index.html 的 buildSidebar 計數、applyFilters、buildCharList 都用同一個邏輯,確保 sidebar 數字和點擊後篩出來的結果一致。

---

## 資料建檔協作:日期查證分工(明羽 2026-06-01 定)

明羽自己找資料、建檔。**分工:**
- **資料頁本身有明寫日期 → 明羽自己處理**,不用丟 Claude。
- **日期沒明寫(要跨頁交叉、追到活動 / news 頁、或頁面已死)→ 丟 Claude 查證**。明羽通常已找到資料、只缺日期。

**Claude 查證 SOP:**
- 對**官方來源**交叉確認:`bang-dream.com/events/<slug>`、`/news/<id>`、prtimes 開催報告、ブシロード EC SHOP 通販頁;官方頁已死就用 Wayback(先打 `archive.org/wayback/available?url=...` 確認有無存檔)。
- 回覆:「✅ 對」或「⚠️ 應是 X」+ **附來源連結**讓明羽能信。
- ⚠️ **必盯的雷:會場販售日 ≠ 線上通販期間**(常差幾天。例 Afterglow いつも通りの放課後デイズ:会場 2020/2/2、通販 2020/1/30~2/17)。**每次都把兩個日期分清楚標出**,讓明羽照自己習慣挑(他傾向用會場日)。
- 工具限制:Claude 的 WebFetch **查得到 Wayback「有無存檔」的 API,但抓不到 web.archive.org / archive.ph 的存檔內容本身** —— 那部分給明羽網址、他自己在瀏覽器開。

---

## 開發環境工作流

### 改 code 步驟
1. 從 GitHub 拉最新版到本地
2. 改 index.html / admin.html / announcement-tool.html
3. 用 admin.html 後台「同步到 GitHub」按鈕推送
4. **等 30 秒~幾分鐘 GitHub Pages 部署**(不是即時)
5. 強制重新整理(Ctrl+Shift+R)看到新版

### 圖片管理
- 徽章圖檔以 ID 為檔名(`{id}.{ext}`),放 `images/` 資料夾
- `ext` 欄位記錄副檔名(jpg/png/webp)
- 圖片透過後台上傳到 GitHub

### 資料同步
- 後台直接讀寫 `badges.json`(透過 GitHub Contents API)
- Supabase 部分(使用者清單)透過 supabase-js client

---

## 備註

如果 Claude Code 看到這份文件後仍對某些設計或選擇有疑問,**問過再動**,不要假設。
