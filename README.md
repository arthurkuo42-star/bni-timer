# bni-timer

BNI 富鼎分會的舞台計時器：iPad 全螢幕黑底黃字倒數 + 手機/電腦遙控。

## 檔案

- `index.html` — 入口頁，兩個按鈕分別進入顯示頁、遙控頁
- `display.html` — 顯示頁（iPad 全螢幕用）
- `remote.html` — 遙控頁（手機/電腦用）

三個都是單檔 HTML，無 build step（跟富鼎其他 repo 風格一致）。

## 即時同步

走 Firebase Realtime Database（亞洲節點 asia-southeast1）。資料路徑 `/timer`：

```
state              : 'idle' | 'running' | 'paused'
durationMs         : 預設時長（毫秒）
endTime            : running 狀態下的「結束絕對時間戳」
pausedRemainingMs  : paused 狀態下的「剩餘毫秒」
updatedAt          : serverTimestamp
```

關鍵設計：running 時存 `endTime`（絕對時間戳），不存 `remainingMs`。這樣 Display 端用 `Date.now()` 自己算剩餘時間，不受網路延遲影響——按下開始的瞬間，所有裝置算出來的「剩餘時間」幾乎一致。

## 顯示行為

| 狀態 | 畫面 |
|:--|:--|
| idle / paused | 黑底、黃字 |
| running（時間 > 0） | 黑底、黃字 |
| running（超時 0~5 秒） | 黑紅閃爍（每 500ms 切換）|
| running（超時 ≥ 5 秒） | 白底、紅字、負數計時（顯示 `-mm:ss`） |
| 負數上限 | `-99:59` 後停止累加 |

## Firebase 設定

`firebaseConfig` 直接寫死在三個 HTML 裡（Firebase apiKey 設計上就是可公開的，真正的防護是 Database Rules）。

⚠️ **目前 Rules 是 Test Mode（30 天內全開放）**，正式長期使用建議改為：
- 只允許讀寫 `/timer` 節點
- 用 Anonymous Auth 或加密 token 限制寫入者
- 或仍維持「事務所內部分享網址、信任不會被外人亂改」的低安全模式（畢竟也只是計時器，被改了重設就好）

## 部署

打算放上 GitHub Pages（同 `futding-chains`、`bni-speaker-tracker` 模式）。

## iPad 全螢幕設定

1. iPad Safari 開啟 `display.html`
2. 點底部分享 → **加入主畫面**
3. 之後從桌面開啟，會自動全螢幕（無網址列、無時間列）
4. 程式內已啟用 Wake Lock API，螢幕不會自動鎖

## 雙擊全螢幕

桌機 / 平板瀏覽器直接開時，雙擊畫面也能進入全螢幕。

## 維護筆記

- Firebase 連線狀態：兩個頁面右上角有小圓點（綠=連線 / 紅=斷線）。斷線時遙控按鈕還是能按，但要等網路恢復才會同步。
- 改設定：所有 Firebase config 重複出現在三個 HTML，要改時記得三個一起改。
- 改快選按鈕：只改 `remote.html` 的 `PRESETS` 陣列。
