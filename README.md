# Telegraph 圖床

這是一個基於 Cloudflare Workers 和 Telegraph (telegra.ph) 的免費無限圖床服務。它利用 Telegram 的伺服器作為儲存後端，並透過 Cloudflare Workers 提供快速的 CDN 加速和管理功能。

## 功能特色
- **免費無限儲存**：使用 Telegram 伺服器儲存圖片。
- **Cloudflare CDN**：全球 CDN 加速，存取速度快。
- **多格式支援**：支援 JPG, PNG, GIF, MP4 等多種媒體格式。
- **管理後台**：提供簡單的管理介面上傳和刪除圖片。
- **認證保護**：可選擇啟用登入驗證保護後台。
- **Bing 每日圖片**：首頁背景自動載入 Bing 每日圖片。
- **快取功能**：瀏覽器端快取上傳記錄。

## 部署教學

### 前置需求
1.  一個 Cloudflare 帳號。
2.  一個 Telegram 帳號（用於建立 Bot）。
3.  Cloudflare D1 資料庫（用於儲存圖片索引）。

### 1. 取得 Telegram Bot Token 和 Chat ID
1.  在 Telegram 中搜尋 `@BotFather`，輸入 `/newbot` 建立一個新的 Bot，並取得 `Bot Token` (例如 `123456789:ABCDefgh12345`)。
2.  建立一個新的頻道 (Channel) 或群組 (Group)，將剛建立的 Bot 加入其中並設為管理員。
3.  向該頻道/群組發送一條訊息，然後轉發給 `@userinfobot` (或其他類似工具) 獲取該頻道/群組的 `Chat ID` (通常以 `-100` 開頭)。

### 2. 設定 Cloudflare D1 資料庫
1.  登入 Cloudflare Dashboard，進入 "Workers & Pages" -> "D1 Integration"。
2.  建立一個新的 D1 資料庫，命名為 `cloudflarepicture` (或自訂)。
3.  在控制台 (Console) 中執行以下 SQL 語句建立資料表：
    ```sql
    CREATE TABLE media (
      url TEXT PRIMARY KEY,
      fileId TEXT,
      groupz TEXT,
      uploader TEXT
    );
    ```
    > [!IMPORTANT]
    > 如果你已經有現存的資料庫，請執行以下指令來新增欄位：
    > ```sql
    > ALTER TABLE media RENAME COLUMN "group" TO groupz;
    > -- 或者如果你是新增加欄位:
    > -- ALTER TABLE media ADD COLUMN groupz TEXT;
    > -- ALTER TABLE media ADD COLUMN uploader TEXT;
    > ```

### 3. 部署 Worker
1.  在 Cloudflare Dashboard 建立一個新的 Worker。
2.  將 `_worker.js` 的內容複製貼上到 Worker 的編輯器中。
3.  **綁定 D1 資料庫**：
    - 在 Worker 的設定 (Settings) -> 變數 (Variables) -> D1 資料庫綁定 (D1 Database Bindings)。
    - 變數名稱設為 `DATABASE`。
    - 選擇剛才建立的 D1 資料庫。
4.  **設定環境變數**：
    在 Worker 的設定 (Settings) -> 變數 (Variables) 部分，新增以下環境變數：

    | 變數名稱        | 說明                                                                 | 範例值                     |
    | :-------------- | :------------------------------------------------------------------- | :------------------------- |
    | `DOMAIN`        | 你的 Worker 網域 (不含 `https://`)                                   | `img.example.com`          |
    | `USERNAME`      | 管理員帳號 (用於登入後台)                                            | `admin`                    |
    | `PASSWORD`      | 管理員密碼                                                           | `password123`              |
    | `ADMIN_PATH`    | 管理後台的路徑 (例如 `admin`，則後台為 `/admin`)                     | `admin`                    |
    | `ENABLE_AUTH`   | 是否啟用首頁上傳驗證 (`true` 表示首頁也需要登入才能上傳; `false` 則公開) | `false`                    |
    | `TG_BOT_TOKEN`  | Telegram Bot Token                                                   | `123456789:ABC...`         |
    | `TG_CHAT_ID`    | Telegram 頻道/群組 ID                                                | `-1001234567890`           |
    | `MAX_SIZE_MB`   | 最大上傳檔案大小 (MB)                                                | `20`                       |

### 4. 綁定網域 (可選)
在 Worker 的設定 (Settings) -> 觸發程序 (Triggers) -> 自訂網域 (Custom Domains) 中綁定你的網域。

## 使用方法

### 首頁上傳
直接訪問你的網域，點擊或拖成圖片進行上傳。支援複製貼上上傳。

### 管理後台
訪問 `https://你的網域/ADMIN_PATH` (例如 `/admin`)，輸入設定的 `USERNAME` 和 `PASSWORD` 登入。
在後台可以查看所有上傳的圖片，並進行刪除操作。

## API
- **上傳圖片**: `POST /upload`
    - `file`: 檔案資料
- **Bing 圖片**: `GET /bing-images`

## 特別說明
本專案僅供學習與技術交流使用，請勿用於非法用途。
圖片儲存於 Telegram 伺服器，因此受 Telegram 的服務條款限制。
