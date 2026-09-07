# ACE03-GETCERT Session #4 — 2026-07-20 (~1h34m)

Source transcript: `../recordings/2026-07-20_ACE03-GETCERT_session4_transcript.txt`

## Quiz warm-up (0:00–13:47)
- NoSQL's key differentiator vs. relational DBs: **flexible schema** (logic moves to the application layer instead of living in the DB); NoSQL follows **BASE** properties, not ACID.
- Product providing a managed Hadoop cluster → **Dataproc** (BigQuery = warehouse, Pub/Sub = messaging service, both distractors).
- Mandate requiring all bucket access to go through IAM only, with no object-level ACLs → **Uniform bucket-level access** (enabling it auto-disables ACLs and forces IAM-only enforcement).
- Trick question: SSH access to a VM is deliberately blocked (via firewall rules — even over VPN/Cloud Shell you still can't SSH in), but the VM can still be **managed (start/stop/resize) via `gcloud` commands** without ever SSHing in. Managing a VM and SSHing into a VM are separate permissions/capabilities.
- Globally consistent, horizontally scalable RDBMS requirement → **Cloud Spanner**.

## Symbol Superstore — security & compliance (13:47–24:26)
- Reinforces **principle of least privilege**: expose services minimally; grant users only what they need.
- Key concept: **Google's own compliance certifications (PCI-DSS, SOC2, ISO, GDPR, HIPAA, etc.) do NOT automatically make anything a customer builds on top of GCP compliant.** Compliance is a shared responsibility — Google provides the tools (backups, PITR, key management options, granular IAM, private VMs, etc.) but configuring and using them correctly is the customer's job.
- Exam framing note: expect scenario-based questions ("ensure a VM or dataset stays in a specific region") rather than direct "is your system GDPR compliant" questions. The tool to enforce region/location constraints across an org, folder, or project is **Organization Policies (org policy constraints)**.
- There is **no single central "security service"** in GCP — security must be built piece by piece across many independent controls: Binary Authorization, DLP, encryption choices, VPC design (shared VPC or not), IAM, firewall rules, public IP restrictions, etc.

## GKE deep dive (24:26–39:29)
- **Autopilot mode**: default/recommended, fully managed (Google handles node/cluster management), **always regional** (cannot be zonal), higher cost, less granular control.
- **Standard mode**: manual cluster/node management, can be either **zonal or regional**, more granular control, generally lower cost.
- **Pod Disruption Budget**: keeps an application healthy/available during rolling updates/version replacements.
- **Release channels**: let Google automatically handle GKE upgrades for both the control plane and worker nodes.
- Access control approaches: **IAM** (project-level, coarse-grained) vs. **RBAC** (Kubernetes-native, fine-grained, namespace-level). Best practice: use IAM for cluster-level access management, supplement with RBAC when different teams need scoped access to specific namespaces without touching each other's resources.
- **GKE Sandbox**: an additional isolation layer for running untrusted third-party container images (e.g., an unverified Docker image from an external vendor) — a preventive security mechanism at the container runtime level.
- **CIDR/IP planning for GKE**: exam expects you to calculate appropriate subnet sizes for nodes, pods, and services based on cluster size (e.g., "N nodes with M pods each") using `2^(32 - mask)` — using a default range wastefully can burn through thousands of IPs.
- **Private clusters**: nodes have no external IP (analogous to VMs without external IPs). To reach Google APIs/services from a private cluster, use **Private Google Access**; to reach external (non-Google) services/resources, use **Cloud NAT**.
- **Workload Identity**: lets a Kubernetes service account impersonate/use a GCP service account directly, eliminating the need to generate and inject long-lived GCP service account key files into pods — the modern, more secure replacement for manually managed SA keys.

## Cloud Run vs. App Engine vs. Cloud Functions (39:29–53:39)
- **Cloud Run**: "serverless Kubernetes" — deploy a container image, no cluster to manage; the workload must be **stateless** (move state to Cloud SQL, Memorystore, or other external storage before deploying).
- **App Engine** (GCP's first PaaS, launched 2008, predating Kubernetes ~2014): **Standard environment** (scales to zero, supports a limited set of languages) vs. **Flexible environment** (requires at least one instance always running, supports more languages). Considered increasingly legacy relative to Cloud Run.
- Cloud Run's advantages over App Engine — scale-to-zero **plus** any language **plus** container support — are why market trend/usage is shifting away from App Engine toward Cloud Run.
- **Cloud Functions**: strictly event-driven serverless compute (sleeps until a trigger fires, executes, goes back to sleep); billed per ~100ms; not suited for hosting a full web application.
- Exam guidance: when a scenario could use either App Engine or Cloud Run with no other differentiating requirement, **default to recommending Cloud Run**.

## Terraform / Infrastructure as Code (53:39–59:33)
- Four key files:
  - **Cloud Build config file** — instructs what infrastructure to build.
  - **backend.tf** — stores remote Terraform state configuration.
  - **terraform.tfstate** — stores local Terraform state.
  - **main.tf** — contains the main/overall Terraform configuration.
- Four core commands, in order:
  1. **`terraform init`** — downloads the latest provider version.
  2. **`terraform plan`** — verifies syntax, confirms required files exist, previews resources to be created.
  3. **`terraform apply`** — actually provisions the resources per the plan.
  4. **`terraform destroy`** — tears down the resources for a specific configuration file (must specify the file explicitly).

## Closing diagnostic quiz (59:33–end)
- Medium-sized MySQL DB containing **user-defined functions**, needs the most timely/economical migration:
  - Eliminate **Cloud SQL** directly — Cloud SQL does **not support user-defined functions**.
  - Eliminate **Cloud Marketplace MySQL image** — requires heavy manual configuration, treated as a last resort.
  - Between two Compute Engine lift-and-shift options, chose **N2 machine type** (balanced price/performance) over **E2** (low-cost tier) — "medium-sized" implies a workload that shouldn't be squeezed onto the cheapest tier; technical fit is prioritized before optimizing for cost.
- MIG requiring **automatic** OS updates → **proactive update type** (vs. **opportunistic**, which is selective/manually-triggered).
- Small pilot GKE cluster that explicitly does **not** need to be highly available → **zonal Standard cluster** — Autopilot is automatically eliminated from consideration since **Autopilot clusters are always regional, never zonal**. A secondary elimination pattern: an option pairing "default node pool" with "container-optimized image" is inconsistent (default vs. customized intent don't match) and should be eliminated on that basis too.
- Quickly deploy a containerized web application, pay only per request, no infrastructure management → **Cloud Run** (not App Engine due to its technical debt vs. Cloud Run; not Cloud Functions, which is event-driven only).
- GCS trigger needed to "analyze and act on files being added" → **Finalize** event type (fires on new object creation or overwrite) — not Delete, Archive, or Metadata Update.
- Creating a bucket with no plan to use ACLs, needing geo-redundancy across a single US metro area (New York City) → use the modern **`gcloud storage`** command (not the deprecated `gsutil`/`gs://` syntax) with **dual-region** (not multi-region, which spans an entire continent/country) and must **explicitly disable ACL evaluation**, since ACLs are enabled by default.

## Housekeeping note
`gsutil` / `gs://` commands are deprecated in favor of **`gcloud storage`**, though `gsutil` syntax may still appear in exam questions since exam content hadn't been fully updated at time of session.
