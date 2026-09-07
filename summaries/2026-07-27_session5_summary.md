# ACE03-GETCERT Session #5 — 2026-07-27 (~1h32m)

Source transcript: `../recordings/2026-07-27_ACE03-GETCERT_session5_transcript.txt`

## Admin Q&A (0:00–4:53)
- Skill-badge progress not counted correctly → contact **Google Skills support team** directly (issue lives in their tracking system).
- Non-starred action-plan items (courses) are optional — useful for learning (400 free credits available) but **not required** for the voucher; only starred/"please complete" items are mandatory.
- No labs in the actual certification exam — exam is **MCQ only**, so AI tool restrictions applying to labs don't apply to the exam itself.
- Exam logistics (physical vs. online, booking) to be covered in a later session.
- Quiz sheet (Google Forms) is already linked in the shared drive.

## Quiz block (4:59–20:15) — CI/CD & Kubernetes basics
- CI/CD pipeline flow: **source code (Cloud Source Repository) → Cloud Build (compiles/debugs/creates artifact) → Artifact Registry** (renamed from the older "Container Registry") **→ vulnerability scanning → Binary Authorization → deploy to production cluster**.
- **Binary Authorization** acts as a gatekeeper: checks whether an artifact passed vulnerability scanning before allowing deployment to a GKE/production cluster; blocks and audit-logs untrusted images, denying deployment until developer fixes the code and restarts the pipeline.
- Testing a risky App Engine update in production: use **App Engine traffic splitting/versioning** — deploy new version alongside the old, gradually shift a percentage of traffic (e.g. 2–3%) to the new version, monitor, then ramp up to 100% and retire the old version. (MIGs can do similar canary/rolling updates but require more manual effort than App Engine's built-in traffic-splitting bar.)
- Basic unit of Kubernetes deployment → **Pod**. Concept used to expose an application to endpoints → **Service** (in GKE this manifests as a Load Balancer).
- Command to create/manage a Kubernetes **cluster** → **`gcloud container ...`** (not `gcloud kubernetes`, which doesn't exist, and not `kubectl`). Once the cluster exists, use **`kubectl`** to operate *inside* it (deploy pods, manage objects via YAML manifests).

## Symbol Superstore — operations (20:15–28:15)
- Ongoing ACE responsibility: monitor resources, understand GCP monitoring/logging services, and manage operational health post-deployment.
- **MIG** ensures the requested instance count is maintained via **health checks** — auto-replaces failed instances. Regional MIGs distribute instances across multiple zones within a region, surviving a zonal outage (vs. zonal MIGs, which don't).
- **Supply-chain management app** (Compute Engine, near HQ in Minneapolis): recommendation to migrate to a **MIG** for automatic scaling, zonal-outage resilience, and easier OS/version updates (AB/canary testing) versus managing bare VMs individually.
- **E-commerce app** (GKE-based, containerized): recommend **external HTTPS Load Balancer** with a single global IP — reduces user latency by serving content closer to users and gives global visibility over the app/DB.
- **Transportation/logistics app** (Pub/Sub → Cloud Function → Dataflow → BigQuery/Bigtable): ACE's job is to document pipeline source/sink details for data engineers. Tip: create a **BigQuery permanent external table** pointing at Bigtable data so analysts can run BigQuery SQL against Bigtable-stored data without duplicating/moving it.

## Additional data storage services (28:15–1:01:10)
- **Filestore**: managed **NFS** solution (not a database) — gives a single IP + share name, mountable on VMs for shared read/write file access.
- **Firestore** (careful: distinct from Filestore): serverless, fully managed **NoSQL document database**, scales from zero to global automatically. Not suited for OLTP/full-SQL-relational needs or unstructured blobs (images/video). Two modes: **Datastore mode** (older) and **Native mode** (newer) — Native mode uniquely supports **offline data persistence** (client-side caching for offline app access).
- **Firebase**: not a database and not technically part of core GCP — it's a **mobile/app development platform** (backed by Google) covering the whole app lifecycle (build, analytics, integrations, auth). Can use Firestore underneath as its NoSQL storage layer, but the two are otherwise unrelated services that just share a similar name.
- **Memorystore**: managed **Redis** — used to cache data in memory, reducing latency/speeding up repeated reads versus round-tripping to a backing database.
- **Cloud Spanner**: scales **horizontally** by adding nodes (unlike traditional RDBMS vertical scaling) — a unique property for a relational database. Config options: **Regional** (4 nines availability) vs. **dual/multi-region** (5 nines) — dual/multi-region spreads read-write replicas across regions plus a witness replica for quorum. Tradeoffs: not ideal for latency-sensitive use cases; requires **offloading DB-level logic (triggers, stored procedures) to the application layer** since Spanner doesn't support them; and migrating to Spanner creates **vendor lock-in** (Spanner is GCP-only, unlike open-source-compatible engines).

## BigQuery deep-dive (39:25–1:01:10)
- BigQuery is a **columnar** database — every query scans full columns, so **cost and speed both scale with the amount of data scanned**.
- Dataset **physical location** must be chosen at creation and is sensitive to matching: to query/copy across tables, the datasets/tables **must reside in the same location** — cross-region copy requires a **data transfer tool**, not a plain copy command.
- IAM pattern for cross-project querying: the querying project needs **`bigquery.jobUser`** role (queries are treated as "jobs" in BigQuery); the project holding the target dataset needs to grant the querying identity **Data Viewer** (or Data Owner) on that dataset.
- **BigQuery Data Transfer Service**: analogous to GCS's Storage Transfer Service — moves data into BigQuery from external sources, supports recurring/scheduled loads (daily/hourly), not just one-off transfers.
- Upload cost model: **batch loads are free**; **streaming inserts incur additional cost**.
- **Data sharing via BigQuery**: share the dataset (with query permissions) rather than exporting/copying the raw data to a partner — partner pays only for their own queries, owner retains the data. **Saved/shared queries** let teams reuse a well-built query instead of rewriting from scratch.
- **Scheduled queries**: built-in feature to run a query automatically on a recurring schedule, avoiding hand-rolled cron/webhook triggering for report generation.
- **Query result caching**: results of a SELECT query are automatically cached for **24 hours** — re-running the identical query against unchanged data costs nothing and returns faster (subject to some caching constraints).
- **Table expiration**: set automatic data expiration (e.g., "keep for 60 days") to auto-delete stale data and stop storage billing — no manual tracking/deletion needed.
- **Partitioning**: splits a table by **timestamp or integer value**; lets a query scan only the relevant partition (e.g., last month) instead of the full dataset — major cost/speed win for warehouse-style, ever-growing tables.
- **Clustering**: co-locates related rows based on one or more column values (e.g., cluster by `country`) to speed up filtered queries; partitioning and clustering can be combined on the same table.
- Instructor noted a comparison-slide deck and decision flowcharts exist (structured/unstructured, transactional/analytical, single-region/multi-region, etc.) covering all ~7-8 storage services (GCS, Datastore, Firestore, Bigtable, Cloud SQL, Spanner, BigQuery) — recommended for exam prep, not walked through live in full.

## Bigtable (55:29–58:00)
- Use when you need: **key-value pairs**, a **wide-column** database, or **time-series data** storage.
- Primarily used for OLTP-style/operational workloads, but can also serve **read-heavy, latency-sensitive analytical workloads** requiring near-real-time decisions — an exception to the default rule of "use BigQuery for analytics."

## Closing diagnostic quiz (1:01:10–end)
- Command to list VM snapshots → **`gcloud compute snapshots list`** — reinforced the `gcloud [group] [subgroup] [verb]` pattern; `gcloud compute get` is invalid (`get` isn't a valid snapshot verb; valid ones are **list, describe, delete**).
- Scheduled snapshot won't delete → scheduled snapshots form an **incremental chain** (each depends on the prior) and may carry a **retention policy** — must **detach the snapshot schedule first**, then delete.
- Which two settings live in a **Managed Instance Group** config (not the Instance Template)? → **health checks** and **autoscaling/load-balancing settings** — machine type, persistent disk, and OS are defined in the **Instance Template** (the VM "blueprint"), not the MIG itself.
- Exposing an application to endpoints (repeat concept) → **Service**.
- **Declarative vs. imperative**: declarative describes the **desired end state** ("what," not "how") — `kubectl apply` is Kubernetes' declarative command; `create`/`replace`/etc. are imperative.
- Limiting the number of connections from a **Cloud Run** service to its backend database → set **max instances** on the Cloud Run service (not "concurrency," which controls users-per-instance, not instances-per-database) — since each Cloud Run instance opens its own DB connection(s), capping total instances caps total DB connections.
- Lifecycle rule to move objects from Standard to Nearline **after a specific date**: use conditions **"Storage class matches Standard"** + **"Created before [date]"** — not "Age," which is for relative day-counts (e.g., 90 days), not an absolute calendar date.

## Wrap-up
One week remaining in the program. Next session (#6) will cover additional technical topics plus non-technical logistics: booking the exam slot after receiving the voucher, and the pros/cons of different exam delivery modes.
