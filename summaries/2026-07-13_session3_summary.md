# ACE03-GETCERT Session #3 — 2026-07-13 (~1h31m)

Source transcript: `../recordings/2026-07-13_ACE03-GETCERT_session3_transcript.txt`

## Admin note (0:00)
- Cohort Google Group now open — post technical questions there going forward.

## Quiz block (2:18–38:38) — Cloud Storage focus
- Storage classes by access frequency: **Standard** (frequent) → **Nearline** (~monthly) → **Coldline** (~quarterly) → **Archive** (~yearly); cost decreases as access frequency decreases.
- **Lifecycle policy**: automates moving/deleting objects on rule-based conditions (age, etc.) — no manual tracking needed.
- Cheapest storage tier overall: **Cloud Storage** < **BigQuery** (2nd cheapest) < other services; NoSQL (Bigtable/Firestore) costs more; Spanner is priciest.
- Multi-petabyte scale + SQL + analysis requirement → **BigQuery** (Cloud SQL tops out ~64TB, not built for analytics).
- Security auditor needing read-only access across every project in an org → grant **Viewer at the org level** (flows down); org-level Viewer alone is insufficient for resource-level detail — also need Project Viewer.
- **Encryption at rest** is mandatory and non-optional across every GCP storage service — can't be disabled. (Encryption in transit is a separate, distinct concern.)
- **Retention Policy** (bucket-level lock): blocks delete/edit for a fixed duration. **Retention lock** = permanent/irreversible until the retention period expires — no admin override once locked.
- **Object Versioning + Retention Policy**: used to be mutually exclusive. **Google changed this rule ~7–8 months prior to this session** — retention now permits object updates, so both features can be combined. Flagged explicitly as the rule to use for the exam (supersedes older material).
- **Object Hold** = retention control applied at the object level (vs. bucket-level Retention Policy). **ACL** = access control at the object level (vs. bucket-level IAM).
- **Object Change Notification**: push-based alerting on bucket changes (vs. polling for changes).
- Encryption key tiers: **GMEK** (Google-managed, default) vs. **CMEK** (customer-managed key, key still stored within Google) vs. **CSEK** (customer-supplied key — customer stores/manages the key itself; losing the key means losing the data).
- Bucket location types: **Region** (lowest latency, single region) / **Dual-region** (2 regions, DR) / **Multi-region** (continent-wide, e.g. "US" spans multiple US regions) / **Zone** (newer option, ~April prior, lowest latency, exam-relevance status unclear at time of session).
- **Autoclass**: Google automatically moves objects between storage classes based on observed access patterns — rarely used in practice but exists as an option.
- Storage class selection must weigh **both access frequency and retention period together**, not frequency alone — picking a class you'll delete from early can overpay.
- Bulk-moving external data into GCS: use **Storage Transfer Service** in ~90–95% of cases (exceptions: very low bandwidth links or extremely small/numerous files).

## Symbol Superstore case study (38:48–1:07:00)
Mapped 3 on-prem applications to GCP compute/service choices:
1. **E-commerce app** (containerized, needs global low latency) → **GKE** + **Cloud Spanner** (global relational DB) + **external HTTPS Load Balancer** + **BigQuery** for analytics.
2. **Transportation/logistics tracking** (IoT truck sensors) → **Pub/Sub** (ingest) → **Cloud Function** (trigger/transform hook) → **Dataflow** (transformation, no persistent storage of its own) → **BigQuery or Bigtable** (storage/analysis, chosen per query pattern).
3. **Supply-chain app** (VM-based LAMP stack, HQ-only usage) → straightforward **lift-and-shift** to **Compute Engine** + **Cloud SQL** + a **regional** (not global) HTTPS Load Balancer, since traffic is localized to HQ.

## Cloud SQL (1:07:00–1:18:00)
- Managed relational database service — **zonal by default**.
- **High Availability** = automatic multi-zone failover, single checkbox to enable.
- **Read Replicas** = cross-region resilience option, but failover from primary to replica is **manual**, not automatic; also used to offload analytics read traffic or support region migration.
- **DMS (Database Migration Service)**: Google's replication-based migration tool; supports heterogeneous source migrations (e.g., Oracle → Cloud SQL).
- Backups: automatic daily + manual snapshots supported; **deleting or stopping an instance does NOT create a backup**; a deleted instance's data is retained only **4 days**; point-in-time recovery is available.
- **Data export** (SQL dump / CSV) is distinct from a backup — used for non-native or cross-platform migration scenarios (e.g., Cloud SQL → on-prem Postgres), not for disaster recovery.

## Closing diagnostic quiz (1:18:00–end)
- Tool for estimating infrastructure cost → **Pricing Calculator**.
- Scenario: 10TB needing immediate access + 30TB accessed every 30 days, must cover the entire US → **multi-region Standard + Nearline** combination (a single region only covers a couple of US regions, not the whole country).
- Requirement for specific OS-level control → **IaaS / Compute Engine**.
- Requirement: portable code, focus only on the application code (not infrastructure) → **Cloud Run** (not Cloud Function, which is strictly event-driven).
- Requirement: highly customized OS + lift-and-shift with minimal changes → **Compute Engine with a custom image**.
- Requirement: full control over containers but no desire to manage the control plane/OS → **GKE**.
