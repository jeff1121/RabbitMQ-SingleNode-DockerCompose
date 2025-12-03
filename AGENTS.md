# 存放庫指引

## 專案結構與模組配置
此存放庫以 `docker-compose.yml` 為核心，該檔定義單節點 RabbitMQ、管理 UI 與具名資料卷。預設環境變數放在 `env/`（同步更新 `.env.example`），自訂 broker 設定放到 `config/`（如 `config/rabbitmq.conf`、TLS 憑證、外掛清單）。日誌追蹤或清理工具等輔助腳本置於 `scripts/`，整合測試與範例則分別放在 `tests/` 與 `docs/`，確保根目錄聚焦於 Compose 資訊。

## 建置、測試與開發指令
使用 `docker compose up --build -d` 作為標準在地部署，啟動前會重建 RabbitMQ 映像。驗證設定時執行 `docker compose logs -f rabbitmq`，並以 `docker compose exec rabbitmq rabbitmqctl status` 確認外掛載入是否成功。需要乾淨節點時改用 `docker compose down -v`，同時移除資料卷。

## 程式風格與命名慣例
Compose YAML 與設定檔採兩格縮排、服務名稱一律小寫（如 `rabbitmq`、`management`）。環境變數以前綴 `RABBITMQ_` 命名以符合官方慣例，檔名使用 kebab-case（`scripts/refresh-certs.sh`）。Bash 腳本需通過 `shellcheck`，並預設 `set -euo pipefail`，同時在檔案頂端註記所需二進位。

## 測試準則
Smoke test 置於 `tests/`，在設定變更前後都要檢查容器健康（管理埠 15672、AMQP 5672）。建議使用 Python（pytest + requests）或 Bats，以便在 CI 中無需額外服務即可執行。所有測試統一由 `./scripts/run-tests.sh` 觸發，該腳本整合 lint、單元 smoke 以及選配壓力測試；若要新增流程，請擴充此腳本而非另建指令。

## 提交與 Pull Request 準則
遵循既有的 conventional commits 前綴（`feat:`、`fix:`、`chore:`），讓釋出版自動產生變更記錄。每次提交只處理單一議題（設定微調、腳本新增或文件更新），修正 broker 問題時在訊息內寫下重現步驟。Pull Request 需說明動機、連結相關議題、貼上關鍵指令輸出（`docker compose ps`、`rabbitmqctl status`），UI 有異動時附上截圖。

## 安全與設定提醒
勿提交真實認證資料，`.env` 只保留遮蔽範例。處理憑證時請存於 `config/certs/` 並紀錄密碼流程。合併前以 `rabbitmq-diagnostics check_running` 驗證新外掛或政策，降低佈署風險。
