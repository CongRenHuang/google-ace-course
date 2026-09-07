# ACE03-GETCERT Session #2 — 2026-07-06 (~1h33m)

Source transcript: `../recordings/2026-07-06_ACE03-GETCERT_session2_transcript.txt`

## Voucher claim form (0:00–6:53)
- Form linked from Drive → Resources → learning plan (slides 4–5 list required skill badges).
- Must complete **5 skill badges total**, including 1 mandatory: "Create your first Gemini Enterprise Application."
- Form fields: name, registered email (usually corporate — confirm with instructor if unsure), personal email (must be able to receive mail), completion status, cohort ID (found on learning-plan page 1).
- Google Skills profile must be set to **public visibility** (Settings → scroll to public visibility → save) before submitting the link.
- Certification field must be **Associate Cloud Engineer** — choosing PCA/other forfeits the voucher.
- **Must finish all skill badges before submitting the form**, or voucher is forfeited.
- Distribution starts end of that week; late submitters get theirs the following week.

## Q&A block (7:00–27:35)
- Passing requires meeting the **minimum score threshold in every exam section individually**, not just an overall average.
- Lab technical issues → email support team / use the in-console help chat if no email response; cohort Google Group not yet active (opens "next week").
- No hard deadline forcing labs to be done before session 6; instructor to confirm exact skill-badge completion date (mentioned ~2026-07-17 in passing, not confirmed as the actual deadline).

## Symbol Superstore — IAM (27:45–38:09)
- Case setup: org migrating on-prem infra to GCP; ACE resource decides resource hierarchy, org policies, project/quota management, IAM, billing.
- **IAM = who (principal) can do what (role) on which resource.**
- Example: marketing data analyst needing read access to e-commerce BigQuery data needs **both** `roles/bigquery.dataViewer` (view data) **and** `roles/bigquery.jobUser` (run queries) — viewing and querying are separate permissions.
- Three IAM role types:
  - **Predefined** — Google-managed, granular, recommended first choice (Google knows its own service permission boundaries best).
  - **Custom** — org-defined combination of permissions, for atypical needs.
  - **Basic** — Owner/Editor/Viewer applied broadly across all resources. Viewer = read-only; Editor = viewer + modify; Owner = editor + manage roles/permissions + manage billing. Note: some orgs want to split "manage roles" from "manage billing" into separate people, but in GCP's model both fall under the single Owner role — no native split.
- Ways to interact with GCP: Console (UI), CLI (gcloud), mobile app, REST APIs.

## Quiz block (27:35–45:14) — global/regional/zonal resources
- IaaS = customer manages the OS (vs PaaS/managed services).
- Custom-permission role → **Custom role** (not predefined/basic).
- All GCP resources must belong to a **project** — no other container option.
- **Global resources**: VPC, some Load Balancers (global HTTP(S) LB). GCS buckets are NOT global — max scope is multi-region (still within a continent, e.g. "US", "EU", "ASIA"), not the whole globe. Cloud SQL is zonal (optionally multi-zonal for HA) but never regional/global. Persistent disks are normally zonal, can sometimes be made regional, never global. Subnets are always regional. GKE clusters are at most regional.
- **Regional resources** (pick two from a list): Regional Persistent Disk + Managed Instance Groups (regional MIG). Compute Engine (VM) instances themselves are always zonal — a recurring point of confusion the instructor stressed repeatedly. IP addresses are usually regional (tied to a subnet) but can become global if attached to a global load balancer — ambiguous, so excluded when the question caps the answer count at exactly two.
- Minimum IPs required for a VM: **1 (internal IP only)** — external and alias IPs are optional add-ons.

## Compute Engine deep dive (45:23–1:13:53)
- Matching workload → compute product: event-driven → Cloud Functions; on-prem VM lift-and-shift → Compute Engine; on-prem containers → GKE/Cloud Run; code-only focus (no infra) → Cloud Run/App Engine.
- VM boot troubleshooting checklist: check boot disk full, check serial port output, enable interactive serial console access, or detach disk and attach as a non-boot disk on another VM to inspect.
- Machine family selection: compute-optimized / memory-optimized / network-optimized / storage-optimized families exist for matching workload profile; custom machine types let you pick exact vCPU/memory, but **Google enforces a fixed vCPU-to-memory ratio** even in custom configs.
- **Network bandwidth scales with vCPU count**, capped at 32 Gbps by default; enabling **VM Tier-1 networking performance** (extra cost) raises the cap to 100 Gbps.
- Persistent disks are network disks attached to VMs and consume part of that bandwidth budget — factor this in when sizing for bandwidth-sensitive workloads.
- **Metadata server**: per-VM self-describing store (name, IPs, attached disks, etc.).
- **Spot VMs / preemptible VMs**: ~90–91% cheaper, usable to "borrow" idle capacity to speed up batch jobs that can tolerate interruption/reclaim; not for long-running or interruption-sensitive workloads.
- **Sole-tenant nodes**: opposite of spot — dedicated physical hardware for a single customer (compliance/security driver), at a higher price; Google maintains a separate hardware layer for isolation.
- **Managed Instance Groups (MIGs)**: manage VMs as a group (autoscaling, autohealing, availability); can be regional (unlike a single VM, which is always zonal).
  - **Stateful vs. stateless MIG**: stateless scales up/down freely; stateful preserves per-instance state and resists scale-down. A stateless MIG requires the application itself to be stateless.
  - **Unmanaged instance groups**: for legacy/non-cloud-native or OS-level clustering lift-and-shift cases — a minority use case; stateless managed MIGs are the primary recommended pattern.
  - Autoscaling tuning (CPU-based vs. other metrics, cooldown periods) and rollout strategy (new VM templates, canary updates/rollback) are documented in linked exam-tip material, not covered in depth live.
- Networking: single NIC/VPC is standard for ~95–97% of use cases; multiple NICs support hub-and-spoke architectures where traffic must be validated by a centralized VM before reaching backend apps.

## Dataproc vs. Dataflow (1:14:36–1:18:22)
- **Dataproc**: managed Spark/Hadoop service for lift-and-shift of on-prem Hadoop/Spark infra; decouples compute from storage — clusters can be torn down when idle while data persists in GCS (replacing HDFS) or Bigtable (replacing HBase), cutting cost during idle periods.
- **Dataflow**: unified batch + streaming data processing service; no built-in storage — pulls from multiple sources (Pub/Sub for streaming/IoT, GCS for batch) and writes to multiple sinks (BigQuery, Bigtable, etc.) using the same transformation logic for both batch and streaming.

## Closing diagnostic quiz (1:18:35–1:31:34)
- New team member needs to **monitor VMs only** → best practice: **add the user to a group, then grant the group a Viewer-level predefined role** — "assign a policy to a user" is wrong phrasing (policies are bindings created when a role is granted, not directly "assigned" as an object) and no group was specified in the distractor options.
- Resource hierarchy order: **Organization → Folder → Project → Resource**.
- Project attributes that can be changed: **at creation time**, both project name and project ID are editable; **after creation**, project ID is permanently fixed — only the project name remains editable. (Project number and creation timestamp are never editable.)
- User needs a role that must apply across every project in the org → grant the role **at the organization level** so it inherits down to all folders/projects, rather than assigning it project-by-project; for object management in Cloud Storage, the fitting role is **Storage Object Admin**.
- Google's IAM role-selection order of preference: **1) Predefined roles → 2) Custom roles → 3) Basic roles (last resort)**.
