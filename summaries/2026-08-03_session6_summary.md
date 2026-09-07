# ACE03-GETCERT Session #6 — 2026-08-03 (~1h33m, final session)

Source transcript: `../recordings/2026-08-03_ACE03-GETCERT_session6_transcript.txt`

## Quiz block (0:37–24:04)
- GCS bucket upload triggering a function → **Pub/Sub** is the mechanism in between.
  - **Cloud Functions Gen 1**: Google auto-creates a **hidden, Google-managed Pub/Sub topic** behind the scenes — GCS publishes to it, it invokes the function. Fully hidden; no visible topic, no IAM to manage, but hard to monitor/debug if triggering fails.
  - **Cloud Functions Gen 2**: introduces **Eventarc** as an explicit intermediary — GCS publishes an event, Eventarc receives and routes it to the function. Google's rationale: full visibility/debuggability of the event pipeline vs. Gen 1's opaque black box. (Noted as beyond exam scope, background context only.)
- Best practice for storing PII (credit card numbers, patient data, etc.) in Cloud Storage → **fine-grained/very granular access control** (not public access, not simple read-only access, and **not** a signed URL with expiration — signed URLs are fine for sharing *non-sensitive* content but are inappropriate for sensitive data since anyone holding the URL can access it until expiry with no further gatekeeping).
- **Easiest** way to host a **static** website → **GCS bucket + Global External Application Load Balancer** with a **backend bucket** (not backend service) pointing at the GCS-stored site content; CDN can be layered on top. (Note: distinguishes from the *best* way, which would be App Engine — the question asked for *easiest*, a deliberate trick.)
- Stateless Python web app, **unpredictable load** → eliminate Compute Engine and GKE (both require you to manage infrastructure/scaling yourself) → use a **PaaS**: **App Engine** (or **Cloud Run**). When both App Engine and Cloud Run appear as options with no other differentiator, **prefer Cloud Run** (reinforces the rule from session #4).
- App Engine Standard's advantage over Flexible → **scales to zero** (Flexible always keeps ≥1 instance running).

## Symbol Superstore — service accounts (24:04–29:11)
- **Service accounts** exist for **machine-to-machine / application-to-application** communication — not for human/user identity.
- Example: Symbol Superstore's supply-chain LAMP-stack app (Compute Engine VM) talks to its Cloud SQL backend via a **service account attached to the VM**.
- Creation flow: IAM & Admin → Service Accounts → Create Service Account → name it (note its auto-generated email) → optionally add description → **Manage Permissions** to grant it the needed role (e.g., **Cloud SQL Instance User** for this case) → attach it to the VM under **Identity and API access**.

## High Availability (HA) deep dive (29:38–44:08)
Core idea: HA = automated failure detection + automated failover, minimizing downtime, protecting **business continuity** — not just "system uptime."

Per-service HA patterns (walked through as a group exercise):
- **Compute Engine**: a single VM is always zonal → combine a **Regional MIG** (survives zonal failure) with a **Load Balancer** to distribute traffic across zones.
- **GKE**: use a **Regional cluster** + **Load Balancer** (same pattern as Compute Engine).
- **App Engine**: already regional by default and fully managed — autoscaling/autohealing built in, **no manual HA configuration needed**.
- **Cloud SQL**: zonal by default — enable the **"multi-zone" / High Availability checkbox** at instance creation to get automatic zonal failover.
- **Cloud Spanner**: regional by default, can be configured **dual-region or multi-region** — inherently highly available (4 nines regional, 5 nines dual/multi-region) with no extra HA configuration needed.
- **Cloud Storage**: same pattern as Spanner — regional → dual-region → multi-region (zone-level bucket location also exists now, per earlier sessions).
- **Takeaway**: the more fully-managed a service is (App Engine, Spanner, Cloud Storage), the less you personally need to configure for HA/fault tolerance — HA is increasingly automatic as you move up the abstraction stack.

## Disaster Recovery (DR) (44:08–55:16)
- DR ≠ HA: HA protects against zonal/component failure with full automation; **DR protects against a whole-region failure** and **always requires some manual intervention** — there's no fully automated end-to-end DR "checkbox" like there is for HA.
- Two governing metrics: **RPO** (Recovery Point Objective — how much data loss is acceptable) and **RTO** (Recovery Time Objective — how much downtime is acceptable). Not explained in depth (assumed prior knowledge).
- DR "flavors" illustrated via Cloud SQL:
  - **Cold DR**: rely on standard **backups**, restore to another region when needed — slow, but adequate if RTO/RPO tolerance is loose. Note: even frequent backups / point-in-time recovery still count as "cold" because **restore time doesn't improve**, only data-loss window does.
  - **Warm DR**: use **cross-region read replicas** — faster recovery, but failover is **manual/human-triggered**, not automatic.
  - **Hot DR**: Cloud SQL **has no native hot-DR option** — the only way to get near-zero-outage DR for a relational workload is to **migrate to Cloud Spanner** (natively multi-region with automatic failover), or build a fully custom multi-region VM + load-balancer architecture yourself (Google gives infrastructure, but you manage everything — complex and atypical).
- Cloud SQL does not offer instance-level "failure checks" outside of the HA (multi-zone) feature; broader monitoring is handled via Cloud Monitoring, a separate concern from DR/HA config.

## Optimal service selection review game (55:16–1:05:04)
Rapid-fire "which GCP service fits this requirement" drill:
- Containerized workloads → **GKE, Cloud Run, App Engine** (not VMs; App Engine's container handling is Google-managed, not literal Kubernetes).
- Need a specific OS / custom licensing / custom backup approach → **Compute Engine**.
- Hybrid or multi-cloud infrastructure (on-prem + Azure + GCP under one pane of glass) → **Anthos**.
- Web application hosting (optimal, not just "possible") → **App Engine or Cloud Run** (repeat of a session-long theme).
- Event-based processing → **Cloud Functions**.
- "Squeeze every drop from provisioned resources" (i.e., you want fine control over exact infrastructure sizing, paying for fixed capacity regardless of load) → **Compute Engine** (or GKE Standard) — not Cloud Run/App Engine, which abstract away infrastructure control.
- Fully managed serverless, focus only on code → **Cloud Run, Cloud Functions, App Engine** (not GKE — GKE isn't fully serverless/PaaS).
- Billing strictly tied to resources consumed → same logic as the "squeeze every drop" case — **Compute Engine / GKE Standard** territory, i.e., you provision and pay for defined resources rather than a managed abstraction.

## Non-technical: booking the certification exam (1:05:41–1:12:51)
- Book via the certification portal (log in / create account) → **Schedule an Exam** → select **Associate Cloud Engineer** → choose delivery mode: **Testing center** vs. **Online/at-home proctored** (a third option shown but told to ignore) → select language, agree to terms, pick a nearby center (based on your profile address) or skip location for at-home → pick date/time → checkout, apply **voucher code** to zero out cost → confirm.
- **Testing center**: straightforward — bring government photo ID + booking confirmation, lock up belongings, take the exam on-site.
- **At-home online proctoring** requires more prep: must use a **personal laptop** (no corporate laptop) with a working camera and mic; a live remote proctor inspects your room (walls, desk, chair) and any suspicious objects before starting; **no pens, paper, food, or water** allowed during the exam once it begins. Certification validity/credentialing is identical regardless of delivery mode.
- Exam format: **~50 questions, 2-hour time limit**.

## Bonus practice content (1:12:51–1:16:03)
- Instructor shared **two practice quiz sets** (Google Forms, ~30 questions each) — recommended to use these **at the very end** of the learning journey, right before booking the exam, targeting **80%+ accuracy** as a readiness signal.
- Also shared a **custom practice-question web app** (trainer-built, not an official Google resource) — log in with email, select difficulty/competency level, choose topic(s) or "all topics," and choose number of questions to generate a custom practice quiz. Explicitly **not** real exam questions, but similar complexity/style.

## Closing diagnostic quiz (1:19:30–1:30:28)
- GKE workload needing to access **Cloud Spanner** (machine-to-machine) → **Service Account** (not a Google user account — that's human/password-based; not Cloud Identity — that's for importing/managing human users from external directories like Active Directory, unrelated to app-to-app resource access).
- Permission needs to apply across an entire **folder** (e.g., "e-commerce" folder containing multiple projects) → assign the role **at the folder level** so it flows down to all child projects, rather than assigning it project-by-project (echoes the org-level pattern from session #2).
- Updating an existing **custom IAM role** → follow the **read-modify-write pattern**: read the role's current state, modify it locally, then write back the modified version. Instructor flagged the documentation's warning about potential conflicts when multiple processes attempt this read-modify-write sequence concurrently — worth reading directly (linked in the deck) rather than memorizing secondhand.
- Mobile app needs to access **Pub/Sub** truck-location data using Google-recommended credentials, and access is **on behalf of a service account** (not an end user) accessing **resources outside Google Cloud** → **Service Account Key**. General decision rule given:
  - Accessing **public** data → **API key**.
  - Accessing **private** data on behalf of an **end user** → **OAuth 2.0 client**.
  - Accessing private data on behalf of a **service account**, resource **inside** GCP → **environment-provided service account credentials** (no key needed, e.g., attached SA on a GCE/GKE workload).
  - Accessing private data on behalf of a **service account**, resource **outside** GCP → **Service Account Key** (this question's case).

## Program wrap-up
This was the final (6th) session of the ACE03-GETCERT cohort. Instructor's closing advice: rewatch the session recordings to internalize the keyword-spotting/elimination technique demonstrated throughout (rather than memorizing answers), use the shared practice quiz sets and custom quiz app before the exam, and check the learning-plan deck for exact voucher-application deadlines (to be confirmed by Aisha/support separately).
