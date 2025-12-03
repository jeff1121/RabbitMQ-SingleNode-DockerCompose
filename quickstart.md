# 快速上手

依照下列步驟即可在數分鐘內啟動 RabbitMQ + Prometheus 堆疊。

## 1. 取得原始碼並檢視
```bash
git clone <repo-url> rabbitmq-single-node
cd rabbitmq-single-node
ls
```
確認 `docker-compose.yml`、`config/` 與 `AGENTS.md` 均已存在。

## 2. 設定（可選）
- 將 `.env.template` 複製為 `.env`，並調整 `RABBITMQ_DEFAULT_USER/PASS`。
- 於 `config/rabbitmq/` 放入自訂設定（例如 `rabbitmq.conf`、`definitions.json`）。
- 若新增 exporter，請修改 `config/prometheus/prometheus.yml` 的抓取目標。

## 3. 啟動堆疊
```bash
docker compose up -d
```
Compose 會抓取 RabbitMQ 3.13 management 映像、啟用 Prometheus 外掛，並使用內建設定啟動 Prometheus。

## 4. 驗證服務
```bash
docker compose ps
open http://localhost:15672     # RabbitMQ UI（admin/admin）
open http://localhost:9090      # Prometheus UI
curl http://localhost:15692/metrics | head
```
若 broker 未成功啟動，可使用 `docker compose logs -f rabbitmq` 解析，並以 `docker compose exec rabbitmq rabbitmqctl status` 確認叢集狀態。

## 5. 持續迭代
- 變更設定後，可針對特定服務執行 `docker compose up -d rabbitmq` 或 `... prometheus` 進行滾動更新。
- 若新增 `tests/`、`scripts/`，建議在此執行 smoke test 或監控腳本。

## 6. 清理
```bash
docker compose down
# 若需移除持久化佇列／交換器：
docker compose down -v
```
資料卷儲存在 `rabbitmq_data`，刪除後 broker 會回到初始狀態。
