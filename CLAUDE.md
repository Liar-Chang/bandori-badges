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
├── index.html              # 前台(主網頁)~5300 行
├── admin.html              # 後台管理工具 ~3140 行
├── announcement-tool.html  # 公告產生器
├── badge-crop.html         # 徽章截圖/裁切工具
├── badges.json             # 徽章主資料(2700+ 筆)
├── manifest.json           # PWA 設定
├── favicon-*.png           # 各尺寸 icon(16, 32, 180, 192, 512)
└── images/                 # 徽章圖片(由徽章 ID 為檔名)
```

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
54mm / 50mm / 30mm / 25mm / 15.2cm / 15cm
```
**尺寸 — 非圓形**:
```
橢圓形 / 方形 / 心形 / 撥片 / 名牌 / 磁鐵 / 約80mm布偶胸章 / 未確定
```

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
- X 第一/二則日文內容
- 額外 hashtag

輸出:
- Threads 版(繁中)
- X 第一則(日文,新增資料)
- X 第二則(日文,功能調整)

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
| X(Twitter) | 日文 | 跟 Bluesky 同步,Buffer 自動拆 X 串文 |
| Bluesky | 日文 | 跟 X 同步 |
| Threads(脆) | 繁體中文 | 獨立發 |

**不發英文/韓文版本到社群**(雖然徽章資料 4 語言都有翻譯)。

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

## 公告寫法(每日)

寫公告時:

- **四語言**(繁中/日文/英文/韓文)
- **只輸出兩個區塊**:【新增/更新資料】+ 【功能調整】
- **不要加** SITE_URL、日期標頭、`【図鑑更新✨】` 包裝(announcement-tool 會處理)
- 範例格式:
  - 繁中:`(新增 N 張)`
  - 日文:`(N枚追加)`
  - 英文:`(N added)`
  - 韓文:`(N장 추가)`
- `(無)` `(なし)` `(None)` `(없음)` **即使空也保留標題**
- 額外輸出 X 用日文 hashtag 區塊
- **匯出工具偶爾會把已公告過的徽章再列出來**(明羽說是同步沒成功造成),Claude 應主動跟前幾天公告比對,有重複就刪除

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
