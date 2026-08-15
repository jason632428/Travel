# 家族東京 9 日遊

2026/8/17 – 8/25 家族旅行的行程網頁：每日行程、景點地圖、交通攻略與行李打包清單。
純靜態網頁，沒有建置流程——用瀏覽器直接開 `index.html` 就能用。

## 檔案

| 檔案 | 內容 |
|---|---|
| `index.html` | 行程總覽 + 地圖與交通（Leaflet） |
| `luggage.html` | 行李打包清單（87 項，可勾選） |
| `tickets.html` | 票券（晴空塔 QR + NEX 換票資訊）**已加密，需密碼** |
| `shared.css` | 兩頁共用樣式：設計 token、標頭、底部導覽、深色模式 |
| `manifest.webmanifest`、`icon-*.png` | 加到手機主畫面用 |
| `hooks/pre-commit` | 擋下明文票券的 git hook（需手動安裝，見下） |

## 改資料的地方

**行程與地圖景點：`index.html` 的 `itinerary` 陣列。**
每天的 `places` 就是地圖上的標記，順序＝走訪順序，也用來畫當天的路線。行程文字和地圖點位放在同一個地方，刪改景點只要動一處。

```js
{
    day: 4, title: '上野・澀谷潮流', meal: '…', budget: '¥5,000',
    area: '上野・澀谷',              // 地圖圖例上顯示的區域名
    items: [ { time: '上午', desc: '…' } ],
    places: [ { name: '上野公園', lat: 35.7146, lng: 139.7734, desc: '…' } ]
}
```

**日期：只有 `tripDayDate()` 一個地方。**

```js
const tripDayDate = (day) => new Date(2026, 7, 16 + day);   // Day 1 = 2026-08-17
```

卡片標題、日期快跳條、地圖彈窗的日期全部從這裡推導，改行程日期只需要改這一行。
（備忘錄與行李提醒的文案裡有寫死的日期，例如「8/24 花火節」，那是敘述的一部分，需要另外改。）

**行李清單：`luggage.html` 的 `packSections` 陣列。**
項目可以是字串，或 `{ t: '主文字', h: '補充說明' }`。有 `tips` 的區塊不含勾選框。

## 票券頁

`tickets.html` 裡是 AES-GCM 加密後的密文，金鑰用 PBKDF2-SHA256 迭代 60 萬次由密碼導出，
解密全部在瀏覽器端做。**repo 裡沒有任何明文票券資料**。

密碼不在這個專案裡，也沒有任何地方存著它——弄丟就打不開，只能用原始 PDF 重新產生一份。

明文票夾與加密工具都已刪除，機器上只剩原始 PDF 與這份加密版。
要重做票券時，跑 `rebuild-tickets.py`（在 Downloads，不在 repo）從 PDF 重新產生明文票夾；
若還需要換密碼重新加密，請照 `## 票券頁` 的規格重做加密工具頁。

### 防止明文票券外流

`.gitignore` 擋得掉 `tickets-lock.html`、`tickets-secure.html`，但**擋不掉同名的 `tickets.html`**，
所以另外裝了一個 pre-commit hook 做把關，它會擋下：

- 任何文字檔內嵌 base64 PNG（明文票夾的 QR 就是這樣存的）
- `tickets.html` 不是加密版（沒有 `PAYLOAD`，或資料不是解密後注入）

hook 放在 `.git/hooks/` 底下**不會被 push**，重新 clone 之後要自己裝回來：

```bash
cp hooks/pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
```

## 狀態儲存

三頁共用 localStorage 的 `tokyo-trip-state-v1`：深色模式、目前分頁、展開的行程卡片、導航交通方式、鐵道圖層開關、行李勾選項目。票券的「已使用」標記另存在 `tokyo-tickets-used-v1`。清除瀏覽器資料就會回到預設。

## 外部依賴

Leaflet 1.9.4、OpenStreetMap 圖磚、OpenRailwayMap 圖磚、Google Fonts，全部走 CDN。
**沒有網路時**：行程總覽與行李清單正常運作（字體會退回系統字體），地圖分頁會偵測到 Leaflet 沒載入，改成列出所有景點的 Google Maps 連結。

## 部署

推到 GitHub 後開 Pages 即可（專案沒有建置步驟，直接指向 `main` 分支根目錄）。
