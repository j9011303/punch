# 🍲 初殿門市定位打卡系統 (Punch Clock System)

**線上體驗:** https://liff.line.me/2010314288-Vh9dMEyC (需在 LINE 內開啟)

---

## 🎯 系統概述

員工用 **LINE 登入** → **GPS 定位驗證** → **WiFi IP 驗證(可選)** → **單鍵打卡**。打卡記錄自動寫入 Google Sheet，生成月度報表，數據可即時用於加班費計算。

| 功能 | 說明 |
|---|---|
| **身分驗證** | LINE Login + ID Token（無法冒充） |
| **在店驗證** | GPS Haversine 距離 ≤ 店家半徑 |
| **WiFi 驗證(可選)** | 前端抓 IP 送後端比對 `allowed_ips` |
| **防重複** | 同人同型別 1 分鐘內只能打卡 1 次 |
| **月報自動生成** | 員工首次打卡時或手動觸發自動建立 |
| **月度封存** | 每月 1 號自動封存上月資料到 Google Drive |

---

## 🏗️ 技術棧

| 層級 | 技術 | 備註 |
|---|---|---|
| **前端** | 🌐 HTML + JS + LINE LIFF | 部署在 GitHub Pages (`/punch/Index.html`) |
| **後端 API** | 📜 Google Apps Script | 已部署 v9，JSON 介面 |
| **資料庫** | 📊 Google Sheets | 員工、店家設定、打卡記錄、月報 |
| **驗證** | 🔐 LINE OAuth2 | Channel ID: `2010314288` |
| **定位** | 📍 Geolocation API + Haversine | 前端取得，後端驗算 |

---

## 📂 目錄結構

```
punch/
├── Index.html          # 前端主程式（大寫 I，GitHub Pages 要求）
│   ├── LINE LIFF 初始化
│   ├── 員工綁定流程
│   ├── 打卡表單（GPS + IP）
│   └── 當日打卡記錄展示
└── README.md           # 本文件
```

### 關鍵路由（後端 Apps Script）

| action | 功能 | 回傳 |
|---|---|---|
| `bootstrap` | 初始化 / 判斷綁定狀態 | `{emp, bound, stores, line_name}` |
| `bind` | 首次綁定員工 | `{ok, name}` |
| `punch` | 記錄打卡 | `{ok, name, time, reason}` |
| `today` | 當日打卡記錄 | `{punches: [{type, time}], date}` |

---

## 🚀 快速開始

### 1. 員工側：掃 QR Code 打卡

```
1. 用 LINE 掃描或點擊：https://liff.line.me/2010314288-Vh9dMEyC
2. 首次進入 → 選擇名字 → 綁定
3. 進入打卡頁 → 確認店別 → 點「打卡」
4. 允許定位權限 → 打卡成功/失敗提示
5. 下方顯示「今日打卡記錄」
```

### 2. 管理員側：配置店家

**Google Sheets 分頁：`店家設定`（gid=445307389）**

| 欄位 | 範例 | 說明 |
|---|---|---|
| `store_id` | `CH` / `XZ` / `YC` | 店家代碼（中和/新莊/永春） |
| `store_name` | `中和店` | 店家名稱 |
| `lat`, `lng` | `24.996571, 121.500551` | 店家座標 |
| `radius_m` | `30` | GPS 驗收半徑（公尺） |
| `allowed_ips` | `111.251.71.60` | WiFi 對外 IP（為空=略過 IP 驗證） |
| `verify_mode` | `BOTH` / `EITHER` / `GPS_ONLY` / `IP_ONLY` | 驗證模式 |

**驗證模式說明：**
- `GPS_ONLY`：只看定位，店內就過
- `IP_ONLY`：只看 IP 對不對
- `EITHER`：GPS **或** IP 任一過都行
- `BOTH`：GPS **和** IP 都要過（店內 + 店內 WiFi）

### 3. 管理員側：查看打卡記錄

**Google Sheets 分頁：`打卡記錄`**

每一筆打卡包含：
```
timestamp, emp_id, name, store_id, type, result, 
fail_reason, distance_m, client_ip, lat, lng, accuracy, 
line_user_id, user_agent
```

`result` = `SUCCESS` 的才算有效打卡。

### 4. 管理員側：月度操作

**自動流程（每月 1 號 00:00 Asia/Taipei）：**
```
monthlyArchive()
  ├─ 封存上月 → Google Drive 資料夾 (ID: 1cH-h2RoC66kfqn9kCstMCyOsdbDRWAvo)
  ├─ 檔名：{店家名}_{YYYY-MM}.gs
  ├─ 內含 4 分頁：月報 + 打卡明細 + 員工快照 + 店家設定快照
  └─ 刪除主表的上月打卡記錄與月報分頁（保留員工/店家設定）
```

**手動觸發：**
- `testArchiveThisMonth()`：封存當月（測試用，不刪資料）
- `buildMonthlyReport()`：重建所有店家當月月報
- `installMonthlyTrigger()`：安裝 1 號定時觸發器

---

## 🔗 系統識別碼（複製用）

| 項目 | 值 |
|---|---|
| Google Sheet ID | `1XVBq5z6f7hPEAPWzeJJ8O-pIgPwBGCIgHG2GJzN3OGY` |
| Apps Script 專案 ID | `1F7WQPSmikB_1E9VMPIItR6uwsylIJGq34Zs5zplf2faQr3new3PZwkyU` |
| Web App Exec URL | `https://script.google.com/macros/s/AKfycbzGcQLml4LSMtGpCk14P7QT6Hn2eA4Mz0s-REha5O9IBXoUvgoxQIqnANsFOso7XRy6/exec` |
| LINE Channel ID | `2010314288` |
| LIFF ID | `2010314288-Vh9dMEyC` |
| 員工打卡連結 | `https://liff.line.me/2010314288-Vh9dMEyC` |
| Drive 封存資料夾 ID | `1cH-h2RoC66kfqn9kCstMCyOsdbDRWAvo` |

---

## ⚙️ 常數設定（Code.gs）

```javascript
const SHEET_ID = '1XVBq5z6f7hPEAPWzeJJ8O-pIgPwBGCIgHG2GJzN3OGY';
const SHEET_EMP = '員工';
const SHEET_STORE = '店家設定';
const SHEET_LOG = '打卡記錄';
const LINE_CHANNEL_ID = '2010314288';
const MIN_INTERVAL_MIN = 1;           // 防重複秒數
const DEFAULT_RADIUS_M = 60;          // 預設店家半徑
const MAX_ACCURACY_M = 150;           // GPS 精度閾值
const ARCHIVE_FOLDER_ID = '1cH-h2RoC66kfqn9kCstMCyOsdbDRWAvo';
```

---

## 🧪 測試員工

| emp_id | name | store_id | status | 用途 |
|---|---|---|---|---|
| E001 | 吳孟騏 | XZ | ✅ 已綁定 | 新莊測試帳號 |
| E002~E007 | 李文少 等 | CH | ⏳ 未綁 | 中和員工 |
| E008~E014 | 蔡鈺豪 等 | XZ | ⏳ 未綁 | 新莊員工 |
| E015~E020 | 77 等 | YC | ⏳ 未綁 | 永春員工 |

---

## 🚨 常見問題

### Q: 員工登入後看到「400 Bad Request」

**原因：** LINE channel 在 `Developing` 狀態（只有有角色的人能登）

**解決：** 管理者進 LINE Developers → Channel Settings → **改為 `Published`**

### Q: 打卡失敗「距離店家太遠」

**原因：** GPS 讀到的位置超過 `radius_m` 半徑

**檢查項目：**
1. 員工手機 GPS 有無開啟？
2. 店家座標是否正確？
3. `radius_m` 是否設得太小？

### Q: 打卡失敗「IP 不符」

**原因：** 員工 IP 不在 `allowed_ips` 白名單

**檢查項目：**
1. 店內 WiFi 對外 IP 是否穩定？
2. `verify_mode` 是否設為 `BOTH`（要求 IP）？
3. 員工是否連到店內 WiFi？

### Q: 月報沒有出現

**原因：** 該店還沒有任何員工打卡成功

**解決：** 手動跑 `buildMonthlyReport()`

### Q: 我換手機登 LINE，打卡還認得我嗎？

**答：** 是的。系統綁的是 LINE **帳號**（`line_user_id`），不是手機，換手機/裝置直接認。

---

## 🔐 安全機制

1. **身分無法冒充**
   - 前端無法指定他人的 ID
   - 後端用 LINE 官方 API 驗 ID Token，無法偽造
   - 綁定時用 LockService 防重複

2. **防重複打卡**
   - 同人同型別（上班/下班）1 分鐘內最多 1 次
   - 跨型別（上班→下班）不受限制

3. **地理位置驗證**
   - Haversine 公式精確到公尺級
   - GPS accuracy > 150m 會被拒絕

4. **IP 驗證（可選）**
   - 前端向 api.ipify.org 抓對外 IP
   - 後端比對 `allowed_ips` 白名單
   - 無法通過伪造（server-side 驗證）

---

## 📋 API 呼叫範例

### 初始化（bootstrap）
```javascript
fetch('https://script.google.com/macros/s/.../exec', {
  method: 'POST',
  headers: { 'Content-Type': 'text/plain;charset=utf-8' },
  body: JSON.stringify({
    action: 'bootstrap',
    idToken: '(LINE ID Token)'
  })
})
.then(r => r.json())
.then(data => {
  // 回傳：{ok, emp: {name, store_id}, bound, stores, line_name}
})
```

### 打卡（punch）
```javascript
fetch('...', {
  method: 'POST',
  headers: { 'Content-Type': 'text/plain;charset=utf-8' },
  body: JSON.stringify({
    action: 'punch',
    idToken: '...',
    store_id: 'XZ',
    type: '打卡',
    lat: 25.044, lng: 121.453,
    accuracy: 12,
    client_ip: '111.251.71.60',
    user_agent: navigator.userAgent
  })
})
.then(r => r.json())
.then(data => {
  // 回傳：{ok, name, time, reason}
  // reason 例：'距離店家 45 公尺，超出 30 公尺半徑' / 'IP 不符'
})
```

---

## 📞 接手事項（給下一位工程師）

- [ ] **後端部署**：Code.gs 含「月度封存+清除邏輯」已完成，需貼回編輯器並部署新版本
- [ ] **月度觸發器**：執行 `installMonthlyTrigger()` 安裝每月 1 號 00:00 自動觸發
- [ ] **永春月報**：等第一位永春員工打卡會自動生成，或手動跑 `buildMonthlyReport()`
- [ ] **新莊 IP 穩定性**：確認 `111.251.71.60` 是固定 IP；若變動改回 `GPS_ONLY` 或 `EITHER`
- [ ] **整合加班費工具**：後端新增 `getAttendance(storeId, month)` action，回傳 `{rows: [{name, date, time}]}`，前端加「從打卡系統讀取」按鈕

---

## 📚 延伸閱讀

本 GitHub repo 的 `outputs/` 目錄內還有詳細設計文件：
- **打卡系統詳細設計書**
- **加班費工具整合指南**
- **進度交接備忘錄**

---

## 📝 授權與維護

- **擁有者**：Chi (j9011303@gmail.com)
- **最後更新**：2026-06-08（前端改單鍵打卡 + GitHub Pages 部署）
- **版本**：v1（Code.gs v9）
