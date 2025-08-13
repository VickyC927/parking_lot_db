# 停車場資料庫系統設計

本專案為「停車場資料庫系統」課程作業，旨在設計並實作一套具備**車輛管理、停車紀錄、費用計算與報表查詢**功能的資料庫系統，並透過系統化的資料建模提升資料維護與查詢效率。


## 專案目標
- 建立結構化的停車場管理資料庫
- 提供車輛進出紀錄、計費與報表分析功能
- 支援多角色使用者（管理員、收費員等）操作
- 確保資料一致性與安全性


## 系統功能
1. **車輛管理**
   - 新增、修改、刪除車輛資訊
   - 車牌、車主、車種等基本資料維護
2. **停車紀錄管理**
   - 記錄車輛進出時間
   - 自動計算停車費用
3. **費用計算與收費**
   - 依車種與停車時數計算費用
   - 支援多種計費方式（小時制、月租制等）
4. **報表查詢**
   - 每日/每月營收統計
   - 車位使用率分析


## 系統設計圖與流程圖
### ER 圖
erDiagram
    OWNER ||--o{ VEHICLE : 擁有
    VEHICLE ||--o{ PARKING_SESSION : 產生
    RATE_PLAN ||--o{ PARKING_SESSION : 套用
    LOT ||--o{ SPACE : 包含
    SPACE ||--o{ PARKING_SESSION : 佔用
    PARKING_SESSION ||--o{ PAYMENT : 產生
    USER ||--o{ PAYMENT : 建立
    USER ||--o{ PARKING_SESSION : 結帳處理

    OWNER {
      int owner_id PK
      string name
      string phone
      string email
    }

    VEHICLE {
      string plate_no PK
      int owner_id FK
      string type         "car/scooter/other"
      string brand_model
    }

    LOT {
      int lot_id PK
      string name
      string address
    }

    SPACE {
      int space_id PK
      int lot_id  FK
      string zone
      string space_type   "regular/EV/disabled"
      string status       "available/occupied"
    }

    RATE_PLAN {
      int rate_id PK
      string name
      decimal hourly_rate
      decimal daily_cap
      decimal monthly_fee
      string apply_type   "hourly/daily/monthly"
    }

    PARKING_SESSION {
      bigint session_id PK
      string plate_no FK
      int space_id  FK
      int rate_id   FK
      datetime in_time
      datetime out_time
      int duration_min
      decimal fee_calc
      string status       "open/closed"
    }

    PAYMENT {
      bigint payment_id PK
      bigint session_id FK
      datetime paid_at
      decimal amount
      string method       "cash/card/qr"
      string ref_no
      int user_id FK
    }

    USER {
      int user_id PK
      string name
      string role        "admin/cashier/guard"
      string account
      string passwd_hash
    }

### 系統流程圖
> ![系統流程圖範例](docs/system-flowchart.png)
> *(可繪製車輛進出、收費與報表流程)*


## 技術細節
- **資料庫系統**：MySQL / MariaDB
- **開發工具**：MySQL Workbench
- **程式語言**（可選擇性擴充）：Java / Python / PHP
- **資料備份**：每日自動備份，防止資料遺失


## 範例查詢語法
```sql
-- 查詢當日營收
SELECT SUM(費用) AS 今日營收
FROM 停車紀錄
WHERE DATE(離場時間) = CURDATE();

-- 查詢特定車牌的停車紀錄
SELECT * FROM 停車紀錄
WHERE 車牌號碼 = 'ABC-1234';

-- 計算每小時的停車數量
SELECT HOUR(進場時間) AS 小時, COUNT(*) AS 車輛數
FROM 停車紀錄
GROUP BY HOUR(進場時間)
ORDER BY 小時;
