# Lubn Pricing Block — Design System 檢查報告

來源：`/pages/pricing` 的 PageFly HTML/Liquid element（Buy/Rent 面板 + PMS 方案表 + 硬體規格）。

---

## 1. 顏色 (Colors)

顏色是用 CSS 變數宣告的，但**同一組顏色被重複宣告了三次、而且變數命名不一致**：

| 用途 | 值 | 出現在 |
|---|---|---|
| 主文字色 | `#111` | `--lbn-ink`（`.lbn-rentbuy`, `.lbn-specs-wrap`）／`--i`（`.lbnp`） |
| 次要文字色 | `#6b6b6b` | `--lbn-mut` ／ `--m` |
| 分隔線 | `#e3e3e3` | `--lbn-line` ／ `--l` |
| 淺底色 | `#f6f6f4` | `--lbn-soft` ／ `--soft` |
| 資訊提示底色 | `#eaf2fb` | `--lbn-blue-bg`（只在 `.lbn-rentbuy` 有） |
| 資訊提示文字 | `#0c447c` | `--lbn-blue-ink`（只在 `.lbn-rentbuy` 有） |

**沒有進到變數系統、直接寫死在 CSS 裡的顏色**（這些之後要調色會找不到）：

- 打勾綠色 `#1d9e75`（`.lbn-list li::before`, `.lbnp-y`）
- 錯誤紅色 `#a32d2d`（`.lbn-err`）
- 未選中的灰 `#bdbdbd`（radio 圈）、`#c9c9c9`（disabled 按鈕）、`#c4c4c4`（表格 `—`）、`#f0f0ef`（規格表格線）
- 陰影黑：`rgba(0,0,0,.06)` / `.12` / `.14` / `.22`（各處 box-shadow，深淺不一，沒有統一 elevation 規則）
- Lightbox 遮罩 `rgba(17,17,17,.72)`

**結論**：核心色盤其實很乾淨（1 個墨黑 + 1 個灰階 + 1 個藍 + 1 個綠 + 1 個紅），但因為變數重複宣告、部分顏色又寫死，換色時要改 3 個地方 + 搜尋一堆 hex code，容易漏改。

---

## 2. 字級 (Typography)

`font-family: inherit` — 完全沒指定字體，吃 Shopify 主題的字體，這點是對的（不會跟全站字體打架）。

但字級**沒有統一的 type scale**，是逐一手動指定的 px 值，全部列出來：

```
52px（桌機價格）→ 44px（手機）
24px（方案表標題）→ 21px（手機）
22px（方案價格）→ 14px（手機）
20px（Total 金額、幣別單位）
18px（+/− 按鈕符號）
16px（CTA 按鈕文字）
15px（大部分 UI 文字：分頁、選項名稱、數量標籤、清單）
14px（規格列、說明文字、方案表 desktop 內文）
13px（次要說明、小標籤）
12.5px（極小字註）
12px（方案卡片描述、規格分類標題）
11px（方案表手機版內文）
10.5px（規格分類標題手機版）
10px（方案表最小字）
```

一共出現了 **13 種不同字級**，彼此間距不規則（52→24→22→20→18→16→15→14→13→12.5→12→11→10.5→10），看得出來是「哪裡需要就手動填一個數字」，不是套 scale 算出來的。

**建議**：之後要套用到其他頁面前，先收斂成一組固定 scale，例如：
`11 / 13 / 15 / 18 / 24 / 32 / 44 / 52`（每級對應到固定用途：註腳／內文／UI 標籤／按鈕／小標／大標／價格-手機／價格-桌機），減少到 8 級以內。

字重也只用了 `400` / `500` / `600`（.5 用最多），這點統一，可以保留。

---

## 3. RWD 斷點

一共出現 **3 組斷點，且數值不一致**：

| 斷點 | 用在哪 | 改變什麼 |
|---|---|---|
| `max-width:749px` | `.lbn-rentbuy`（Buy/Rent 面板） | 兩欄變一欄、圖片比例改 8:3、分頁置中、價格字級縮小 |
| `min-width:769px and max-width:1023px` | `.lbnp`（方案表） | 只在這個「窄桌機/平板」區間打開表格橫向捲動 |
| `max-width:768px` | `.lbnp`（方案表）、`.lbn-specs-wrap`（規格） | 字級全面縮小、隱藏描述文字、sticky 表頭 padding 縮小 |

⚠️ **749 跟 768 是兩個不同的手機斷點**，同一頁面裡 Buy/Rent 面板在 749px 就變手機版排版，方案表卻要到 768px 才變。中間這 19px（750–768px）寬度會出現「上面已經是手機版排版，下面表格還是桌機版」的不一致畫面。建議之後統一成同一個斷點（常見會用 `768px`，跟 Shopify 主題預設一致）。

另外還有一個**跟這個 code block 完全無關但被寫死進來的全站設定**：

```css
body{overflow-x:clip !important}
```

這是為了讓 `.lbnp` 表頭的 `position:sticky` 生效，強制覆蓋主題原本的 `body{overflow-x:hidden}`。註解裡也寫了「最好在主題 CSS 修，這裡只是讓這個 block 獨立可用時也正常」。**如果你要把這個 pattern 套到其他頁面**，這行只需要在全站 CSS 設一次即可，不要每個頁面的 block 都重複寫一次 `!important` 覆蓋 body，容易互相蓋來蓋去、之後很難查是誰動了 `body` 的 overflow。

---

## 4. 重複的容器 / reset 樣式

這份 code 有 3 個獨立區塊（`.lbn-rentbuy`／`.lbnp`／`.lbn-specs-wrap`），**每一個都各自重新宣告一次**：

```css
*{box-sizing:border-box}
max-width:1080px;
margin:0 auto;
font-family:inherit;
color:...;
```

功能上沒問題（各自獨立、互不影響），但如果要開始套用到「其他頁面」，建議把這一段抽成**一個共用的 base class**，例如 `.lbn-block`，所有區塊都繼承它，色票變數也只在最外層宣告一次，內層不用各自重複宣告、也不會有變數命名兩套並存的問題。

---

## 5. Clarity 放在哪裡？

**這份 code 裡完全沒有 Microsoft Clarity 的追蹤碼**，我在整個檔案裡搜尋過，沒有任何 `clarity.ms` 或 `clarity(` 相關字樣。

這裡實際存在的分析追蹤是另一條線，寫在最下面的 `track()` function：

```js
function track(name, params){
  window.dataLayer.push(...)   // 推進 GTM 的 dataLayer
  window.gtag(...)             // GA4
  window.fbq(...)              // Meta Pixel
  window.ttq(...)              // TikTok Pixel
}
```

程式碼註解裡有明講：這段**不會**重複載入 GTM／GA4／Pixel，因為全站已經有一個 GTM 容器 `GTM-5VBL9XL2`（在最上面的 nav 附近你之前貼的頁面原始碼裡也看得到這個 ID）。這個 block 只是把 `view_item`、`select_item`、`add_to_cart`、`lubn_plan_toggle` 這些自訂事件，推進**同一個**已經存在的 dataLayer，讓 GTM 容器接手。

**所以 Clarity 該裝在哪：**

Clarity 是**全站層級**的追蹤碼，跟這個 GTM 容器一樣，只需要裝「一次」，裝在 Shopify 後台的 `theme.liquid` 的 `<head>` 區塊（或是透過 GTM 新增一個 Custom HTML 標籤、觸發條件設 All Pages），**不是**每個頁面的 PageFly block 裡各裝一次。

如果你想比照這個 GTM 的做法（單一 dataLayer、各頁面 block 只負責推事件），Clarity 也支援用 `clarity('event', 'xxx')` 或 `clarity('set', 'xxx', 'yyy')` 手動推自訂事件/標籤，可以比照這個 `track()` function 的模式，在 Clarity 裝好之後，同一個 function 裡再加一行 `try{ if(window.clarity) window.clarity('event', name); }catch(e){}`，這樣未來任何頁面用同一套 block，事件會同時進 GTM 也進 Clarity，不用兩邊分開埋。

---

## 6. 建議：抽成共用 tokens（給其他頁面套用用）

```css
:root{
  /* colors */
  --lbn-ink:#111;
  --lbn-mut:#6b6b6b;
  --lbn-line:#e3e3e3;
  --lbn-soft:#f6f6f4;
  --lbn-blue-bg:#eaf2fb;
  --lbn-blue-ink:#0c447c;
  --lbn-green:#1d9e75;
  --lbn-red:#a32d2d;

  /* type scale */
  --lbn-fs-xs:11px;
  --lbn-fs-sm:13px;
  --lbn-fs-base:15px;
  --lbn-fs-md:18px;
  --lbn-fs-lg:24px;
  --lbn-fs-xl:32px;
  --lbn-fs-2xl:44px;  /* 手機大價格 */
  --lbn-fs-3xl:52px;  /* 桌機大價格 */

  /* layout */
  --lbn-container:1080px;
  --lbn-radius-sm:8px;
  --lbn-radius-md:12px;
  --lbn-radius-lg:16px;
}

/* 統一斷點：手機 ≤768px */
@media (max-width:768px){ ... }
```

把這段放進主題的全站 CSS（或每個 PageFly HTML block 最上面都 `include` 同一段），之後新頁面直接用這些變數，不用每次重新定義一次色票和字級。
