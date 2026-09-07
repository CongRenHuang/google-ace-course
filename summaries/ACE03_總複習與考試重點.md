# ACE03-GETCERT 課程總複習與 ACE 考試重點

涵蓋 Session #2～#6（共 5 堂，Session #1 尚未取得逐字稿）

---

## Session #2（2026-07-06）— IAM 與資源階層

- **Resource Hierarchy**：Organization → Folder → Project → Resource。權限由上往下繼承。
- **IAM 三種角色**：
  - Predefined（Google 預設，建議優先使用）
  - Custom（客製化組合）
  - Basic（Owner/Editor/Viewer，範圍最粗）
  - 優先順序：**Predefined → Custom → Basic（最後手段）**
- BigQuery 權限重點：**檢視資料**要 `dataViewer`，**執行查詢**要另外的 `jobUser`（兩者分開）。
- **全域 / 區域 / 可用區資源**：
  - 全域：VPC、部分 Load Balancer
  - 區域：Regional MIG、Regional Persistent Disk
  - 可用區：VM 實例本身、Cloud SQL（預設）、Persistent Disk（多數情況）
  - 常考陷阱：**VM 永遠是 zonal**，GKE cluster 最多到 regional，GCS bucket 最多 multi-region（非 global）。
- VM 最少要有 **1 個 internal IP**，external/alias IP 都是選配。
- Compute Engine：Spot VM（省 90%，可被收回）、Sole-tenant node（獨佔硬體，較貴）、MIG（stateful vs stateless）。
- **Dataproc**（管理式 Hadoop/Spark，運算與儲存分離）vs **Dataflow**（batch+streaming 統一處理，無自帶儲存）。

## Session #3（2026-07-13）— Cloud Storage 與 Cloud SQL

- 儲存等級（依存取頻率遞減）：**Standard → Nearline → Coldline → Archive**。
- **Lifecycle Policy**：自動依條件搬移/刪除物件。
- **Uniform bucket-level access**：強制只用 IAM，關閉 ACL。
- **加密金鑰三層**：GMEK（Google 管理，預設）／CMEK（客戶管理金鑰，仍存在 Google）／CSEK（客戶自帶金鑰，遺失=資料遺失）。
- **Retention Policy**（bucket 層級鎖定）與 **Object Hold**（物件層級）的差異；**Object Versioning 現已可與 Retention Policy 併用**（近期規則變更，考試以新規則為準）。
- Bucket 地理選項：Region / Dual-region / Multi-region / Zone（新選項）。
- **Storage Transfer Service**：外部資料搬進 GCS 的首選（除非頻寬極低或檔案極小）。
- Cloud SQL：預設 zonal；**HA = 自動多可用區容錯**；**Read Replica = 手動容錯**（不是自動）；**DMS** 支援異質資料庫遷移；刪除/停止 instance **不會自動備份**，刪除後僅保留 4 天。

## Session #4（2026-07-20）— GKE 與 Compute 服務深入

- **NoSQL vs 關聯式資料庫**：NoSQL 走 **BASE**（非 ACID），schema 彈性、邏輯下放到應用層。
- 合規重點：**Google 本身通過的合規認證（PCI-DSS/SOC2/ISO/GDPR/HIPAA）不代表客戶在其上蓋的系統也自動合規**——這是共同責任。考試常考情境題（例如用 **Organization Policy** 強制資料/VM 留在特定地區），而非直接問「你合規嗎」。
- **GKE Autopilot**（全託管、**永遠 regional**、貴）vs **Standard**（可 zonal 或 regional、彈性高、便宜）。
- **IAM（cluster 層級）vs RBAC（namespace 層級）**：建議 IAM 管大方向，RBAC 補足細粒度。
- **Private Cluster**：無外部 IP → 連 Google 服務用 **Private Google Access**，連外部用 **Cloud NAT**。
- **Workload Identity**：K8s Service Account 直接代用 GCP Service Account，免產生/注入金鑰檔（安全性最佳實務）。
- **Cloud Run**（需 stateless）vs **App Engine**（Standard 可縮到零／Flexible 至少 1 台）vs **Cloud Functions**（純事件驅動）。**兩者都適用時優先選 Cloud Run**。
- Terraform 四大檔案（cloudbuild config / backend.tf / terraform.tfstate / main.tf）與四大指令（**init → plan → apply → destroy**）。

## Session #5（2026-07-27）— BigQuery、Spanner、其他儲存服務

- CI/CD 流程：**Cloud Source Repository → Cloud Build → Artifact Registry → 弱點掃描 → Binary Authorization（守門）→ 部署**。
- App Engine **流量分割（traffic splitting）** 可安全測試新版本上線。
- **`gcloud container`** 建立/管理 cluster；**`kubectl`** 操作 cluster 內部資源。
- Filestore（NFS，非資料庫）vs Firestore（NoSQL 文件資料庫，注意拼字勿混淆）vs Firebase（App 開發平台，非資料庫，可搭配 Firestore）。
- **Cloud Spanner**：唯一支援**水平擴展**的關聯式資料庫，但需將 DB 層邏輯（trigger/預存程序）移到應用層，且會**綁定 GCP（vendor lock-in）**。
- **BigQuery** 重點：
  - 欄式資料庫，查詢**成本與速度取決於掃描的資料量**
  - Dataset 建立時要選對 **physical location**（跨 location 不能直接 copy，需用資料轉移工具）
  - **Query 結果自動快取 24 小時**
  - **Table Expiration**（自動過期刪除）、**Partitioning**（依時間戳記/整數分區）、**Clustering**（依欄位共置資料）——三者都是速度/成本優化
- **Bigtable**：key-value、wide-column、時序資料首選；若分析場景**讀取密集且要求極低延遲**，也可用 Bigtable 取代 BigQuery。
- 快照指令固定為 `gcloud [group] [subgroup] [verb]`（例：`gcloud compute snapshots list`，沒有 `get` 這個動作）。

## Session #6（2026-08-03，最終堂）— HA/DR、服務選型、考試報名

- Cloud Function **Gen 1（隱藏 Pub/Sub topic）vs Gen 2（顯式 Eventarc）**——Gen 2 為了可視性/除錯性。
- PII 資料安全策略：**細粒度存取控制**優於 signed URL（後者到期前任何人持有連結都能存取，不適合敏感資料）。
- 靜態網站「最簡單」架設法：**GCS bucket + Global External Application LB（backend bucket）**，注意跟「最佳」方案（App Engine）是不同答案的考點。
- 無法預測負載的 stateless 應用 → 排除 Compute Engine/GKE（需自管基礎設施）→ 選 **App Engine 或 Cloud Run**（兩者都在時選 Cloud Run）。
- **服務帳號（Service Account）= 機器對機器溝通**，非人類使用者身分。
- **HA（高可用）各服務作法**：
  - Compute Engine → Regional MIG + Load Balancer
  - GKE → Regional Cluster + Load Balancer
  - App Engine / Spanner / Cloud Storage → **預設已高可用，免額外設定**
  - Cloud SQL → 勾選 **Multi-zone（HA）**
- **DR（災難復原）三種等級**（以 Cloud SQL 為例）：
  - **Cold**：靠備份還原（慢，但簡單）
  - **Warm**：跨區 Read Replica（快一些，但**手動**容錯）
  - **Hot**：Cloud SQL **原生不支援**，需改用 **Cloud Spanner** 或自建複雜跨區架構
  - **DR 不像 HA 有全自動選項，一定需要人為介入決策**
- 服務選型速記：容器化 → GKE/Cloud Run/App Engine；特定 OS/授權 → Compute Engine；混合雲/多雲 → **Anthos**；事件驅動 → Cloud Functions；要精準控制/固定計費的資源 → Compute Engine/GKE Standard；全託管只寫 code → Cloud Run/Cloud Functions/App Engine。
- 認證金鑰選擇邏輯：
  - 存取**公開**資料 → API Key
  - 存取私有資料、代表**使用者** → OAuth 2.0 Client
  - 存取私有資料、代表**服務帳號**、資源在 **GCP 內** → 環境提供的服務帳號憑證（免金鑰）
  - 存取私有資料、代表**服務帳號**、資源在 **GCP 外** → **Service Account Key**
- 考試報名：約 **50 題、2 小時**，可選**考場**或**居家線上監考**（居家版需個人筆電、鏡頭/麥克風、房間全景檢查、考試中禁帶紙筆食物飲水）。

---

## 🎯 ACE 考試整體重點與應試技巧

### 1. 出題邏輯：情境題 > 死背題
考題幾乎都是「情境描述 + 4 選項」，重點在於**找關鍵字、消去法**，而非死記服務定義。整個課程反覆示範的解題套路：
1. 先抓出**限制條件關鍵字**（例如：不可預測負載、highly available、特定日期、只需要 code、跨區、機器對機器…）
2. 用關鍵字直接**排除不可能的選項**（技術上不合、範圍不符、非最佳實務）
3. 剩下選項再比「**最佳化 / 最省事 / Google 推薦**」的那一個——同樣技術正確時，通常有唯一的「更優」答案（如 Cloud Run > App Engine、Managed Service > 自管）

### 2. 常見必考觀念地圖
| 主題 | 必背重點 |
|---|---|
| IAM | Predefined > Custom > Basic；Service Account 用於機器對機器 |
| 資源範圍 | VM/Cloud SQL＝zonal；GKE 最多 regional；VPC＝global |
| 運算選型 | 事件驅動→Functions；容器＋免管理→Cloud Run；需自訂 OS→Compute Engine；混合雲→Anthos |
| 儲存選型 | 結構化+跨區→Spanner；分析+大量資料→BigQuery；NoSQL 文件→Firestore；key-value/時序→Bigtable；檔案共享→Filestore |
| HA vs DR | HA＝自動容錯（zone 層級）；DR＝人工決策（region 層級，Cold/Warm/Hot） |
| 安全 | Uniform bucket-level access；PII 用細粒度權限非 signed URL；GCP 合規≠客戶系統自動合規 |
| Terraform | init → plan → apply → destroy |
| gcloud 語法 | `gcloud [group] [subgroup] [verb]`，如 `gcloud compute snapshots list` |

### 3. 特別容易考錯／混淆的地方（課程中反覆強調）
- **Filestore（NFS）vs Firestore（NoSQL）vs Firebase（App 平台）**——三個名字很像但完全不同服務。
- **App Engine vs Cloud Run**：技術上常常都能用，但 Google 建議**都可用時選 Cloud Run**。
- **HA 不等於 DR**：HA 是自動化、處理 zone 失效；DR 是處理整個 region 失效，且**一定有人工介入**。
- **Object Versioning + Retention Policy** 規則已變（現在可併用）——舊教材可能寫錯，以近期規則為準。
- **`gsutil` 已淘汰**，改用 `gcloud storage`，但考題仍可能出現舊語法。
- Cloud SQL **Read Replica 容錯是手動**，不是自動（跟 HA 的自動容錯要分清楚）。

### 4. 讀書策略建議
- 依「資源歸屬服務」分類記憶（哪個服務原生 zonal/regional/global），比背個別服務定義更有效率，因為很多考題就是繞著這個範圍設計陷阱。
- 對每個服務至少記住：**適用情境（When to use）**＋**不適用情境（When NOT to use）**，因為出題常用排除法設計干擾選項。
- 考前用課程提供的兩份 Google Form 練習題（各 ~30 題，目標正確率 80%+）與自建練習 App 做最後驗收。
- 讀書順序建議跟著本課程模組走：IAM/資源階層 → 運算（Compute/GKE/Serverless）→ 資料儲存（GCS/SQL/Spanner/BigQuery/Bigtable）→ 網路 → 安全/合規 → HA/DR → IaC（Terraform）。
