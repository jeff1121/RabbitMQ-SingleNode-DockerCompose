# RabbitMQ 單節點（含 Prometheus）

## 概述
此存放庫提供一套具意見的 Docker Compose 堆疊，用於啟動可使用管理介面並導出 Prometheus 指標的單節點 RabbitMQ broker，適合在地開發、展示與需要 AMQP 及輕量觀測能力的整合測試。

## 堆疊組成
- **RabbitMQ 3.13（management 映像）**：暴露 AMQP（`5672`）、管理 UI（`15672`），並在 `15692` 啟用 `rabbitmq_prometheus` 外掛。
- **Prometheus 2.53**：每 15 秒抓取 broker 指標並在 `9090` 提供 UI。
- **共享服務預設**：`x-common` 集中 DNS、類 swarm 的 deploy 設定與 json-file 日誌，讓所有服務繼承一致的執行政策。

## 目錄結構
```
config/
├── prometheus/
│   └── prometheus.yml   # Prometheus 抓取設定
└── rabbitmq/            # 放置 rabbitmq.conf、definitions.json、憑證等
docker-compose.yml       # 主要堆疊定義
AGENTS.md                # 貢獻者指南
```

## 先決條件
- 安裝 Docker Desktop 或 Docker Engine 24+，並可使用 Compose V2（`docker compose` 指令）
- 主機需保留 `5672`、`15672`、`15692` 與 `9090` 連接埠

## 使用方式
1. 複製 `.env.template` 為 `.env` 並調整憑證；若要修改外掛／設定，可同步更新 `docker-compose.yml` 或 `config/rabbitmq/`。
2. 啟動堆疊：`docker compose up -d`
3. 驗證服務：
   - RabbitMQ UI：<http://localhost:15672>（預設 `admin/admin`）
   - Prometheus UI：<http://localhost:9090>
   - 指標端點：<http://localhost:15692/metrics>
4. 完成後關閉：`docker compose down`（加上 `-v` 可一併清掉資料卷）。

## 可觀測性與指標
Prometheus 預設載入 `config/prometheus/prometheus.yml`。若需新增警示或抓取工作，編輯該檔後執行 `docker compose up -d prometheus` 以熱更新。Grafana 可在 Compose 網路內以 `http://prometheus:9090` 連向 Prometheus。

## 自訂建議
- 透過 `config/rabbitmq/definitions.json` 預先載入 exchange、queue 與 policy，並在 compose 中掛載。
- SSL 憑證置於 `config/rabbitmq/certs/`，並在 `rabbitmq.conf` 參照。
- 若要加入其他 exporter（如 node-exporter），可重複使用 `x-common` 區塊以維持一致配置。

更精簡的步驟請參考 `quickstart.md`。
