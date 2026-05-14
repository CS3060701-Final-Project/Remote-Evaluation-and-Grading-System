# REGS - Remote Evaluation and Grading System

**REGS** 是一個現代化的線上程式評測系統（Online Judge），採用前後端分離架構，並結合 Docker Sandbox、CMake/Ninja 編譯流程與 JWT 權限控管。

---

## 主要功能

- 使用者註冊、登入、登出與 JWT 權限驗證
- 題目列表瀏覽、題目詳情、題目統計
- `.zip` 程式碼上傳與提交評測
- 非同步評測佇列與 `operatorId` 查詢
- 管理員題目管理：建立、更新、刪除題目，並上傳測資
- Docker 隔離編譯與執行，提升安全性
- Swagger API 文件與互動式測試頁面

## 技術棧

- 後端：Go + Gin
- 資料庫：PostgreSQL + GORM
- 前端：React + TypeScript + Vite
- 編譯工具：CMake + Ninja
- 認證方式：JWT (ECDSA token)
- Docker：PostgreSQL 容器與評測隔離環境

## 專案結構

```
REGS-Backend/
├── backend/                 # Go 後端服務
│   ├── cmd/                 # 服務與種子資料程式入口
│   ├── internal/            # 後端內部實作
│   │   ├── api/             # HTTP handlers 與 middleware
│   │   ├── database/        # PostgreSQL 連線與 AutoMigrate
│   │   ├── judge/           # 評測沙盒與執行邏輯
│   │   └── models/          # GORM 資料模型
│   ├── docs/                # Swagger / OpenAPI 定義
│   ├── pkg/                 # 可重用套件（JWT、工具等）
│   ├── scripts/             # 管理員註冊、重設資料庫等輔助腳本
│   ├── docker-compose.yml   # Postgres 容器編排
│   └── go.mod               # Go 模組
├── frontend/                # React 前端應用
│   ├── public/
│   ├── src/
│   ├── package.json
│   └── README.md
├── start_all.bat            # 一鍵啟動 backend + frontend (Windows)
├── data1.md                 # 題目測試架構說明
├── data2.md                 # 期末專案設計說明
└── README.md                # 本檔案
```

## 快速啟動

### 需求

- Go
- Node.js / npm
- Docker
- Windows (目前提供 `.bat` 啟動腳本)

### 後端：啟動 PostgreSQL

```bat
cd backend
docker compose up -d
```

> PostgreSQL 會啟動在 `localhost:5433`，對應 `backend/internal/database/db.go` 中的預設 DSN。

### 後端：建立管理員帳號

```bat
cd backend
scripts\register_admin.bat
```

### 後端：啟動伺服器

```bat
cd backend
go run ./cmd/server
```

伺服器預設監聽： `http://localhost:8081`

### 前端：設定 API 位置

```bat
cd frontend
copy .env.example .env
```

確認 `.env` 中的內容：

```env
VITE_API_BASE_URL=http://localhost:8081/api
```

### 前端：啟動開發伺服器

```bat
cd frontend
npm install
npm run dev
```

前端預設頁面： `http://localhost:5173`

### 一鍵啟動（Windows）

若已安裝 Go、Node.js 與 Docker，可在專案根目錄直接執行：

```bat
start_all.bat
```

此腳本會在兩個獨立視窗中啟動：

- `backend`：執行 `go run ./cmd/server`
- `frontend`：執行 `npm run dev`

---

## API 文件

後端啟動後，請開啟：

* http://localhost:8081/swagger/index.html

此頁面提供完整的 OpenAPI 文件與互動式測試介面。

## 核心 API 概覽

- `POST /api/users/register`：註冊帳號
- `POST /api/users/login`：登入並取得 JWT
- `POST /api/users/logout`：登出
- `GET /api/users/me`：取得當前使用者資訊
- `GET /api/problems`：取得題目列表
- `GET /api/problems/{id}`：取得題目詳情
- `PUT /api/problems`：新增或更新題目（Admin）
- `DELETE /api/problems/{id}`：刪除題目（Admin）
- `POST /api/submissions`：提交評測（User）
- `GET /api/submissions`：查詢提交紀錄
- `GET /api/submissions/{operatorId}`：查詢單筆提交結果
- `GET /api/stats/problems/{problem_id}`：題目統計資料

> 需要授權的請求請帶入 `Authorization: Bearer <token>`。

---

## 使用者角色與權限

- `Guest`: 可瀏覽題目列表、題目詳情與統計
- `User`: 可註冊、登入、提交程式、查看自己的提交紀錄
- `Admin`: 具備題目管理與測資上傳權限

## JWT 密鑰生成

後端使用 ECDSA JWT 簽署機制，若您需要重新生成金鑰，可於 `backend` 目錄執行：

```bash
openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out private.pem
openssl pkey -in private.pem -pubout -out public.pem
```

- `private.pem`：用於簽發 JWT token
- `public.pem`：用於驗證 JWT token

請確保這兩個檔案放在後端可讀取的位置，並與後端設定相符。

## 重要檔案與資料夾

- `backend/cmd/server/main.go`：後端伺服器啟動程式
- `backend/cmd/seed/main.go`：建立 admin 種子資料
- `backend/internal/database/db.go`：PostgreSQL 連線設定與自動遷移
- `backend/internal/api/handlers/`：API 邏輯處理
- `backend/internal/api/middleware/auth.go`：JWT 驗證與權限控管
- `backend/internal/judge/`：評測沙盒與執行流程
- `backend/pkg/jwt/jwt.go`：JWT token 產生與驗證
- `backend/docs/swagger.yaml`：Swagger 文件定義
- `frontend/src/`：React 前端程式碼
- `frontend/.env.example`：前端 API 基本設定範本

## 開發與測試注意事項

- 若要重設資料庫，可使用 `backend/scripts/reset_database.bat`（僅限開發環境）。
