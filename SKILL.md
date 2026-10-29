---
name: public-tender-grokbot-orchestrator-skill
description: Autonomous public procurement and tender orchestration for sole traders and SME contractors using native grokbot CLI browser agents. Enforces the ground-truth axiom, multi-platform authenticated inbox sweeps (DEHú, e-NOTUM, PLACSP, PLYCA, Vortal, SiREC, AWS, PLACE, Dematis), anti-false-expired guards, pre-filing portal mailbox verification, non-duplicate filing guards, replicated parallel horizontal synchronization across grokbot pool accounts (zero single-account bottlenecks), FNMT PAdES/CAdES digital signing, LCSP Regla 17 pricing safeguards, and 3-hour recurring notification sweeps across all major EU and Spanish procurement platforms.
---

# Public Tender Grokbot Orchestrator Skill

## Overview

Use this skill to autonomously manage, bid, monitor, and resolve public procurement tenders across Spanish and European platforms (LCSP, Directives 2014/24/EU, Vortal, PSCP, PLACSP, Regional and Municipal portals) for sole traders (autónomos) and SMEs.

This skill operates strictly using the native **`grokbot` CLI** (`grokbot agent roster`, `prompt`, `tail`, `call`, `ask`, `pool`, `serve`).
**Zero Python relay scripts rule:** Never re-introduce `relay.py` or ad-hoc Python websocket wrappers. All browser agent orchestration is handled by the official `grokbot` package.

---

## 1. The Core Ground-Truth Axiom & Multi-Platform Statutory Inbox Matrix

> **Ground-Truth Axiom:** An initial submission receipt proves *only* that an envelope was delivered at that timestamp. It is **never** proof that the tender is completed, awarded, or closed. The contracting authority's communications inside authenticated portal mailboxes override all prior states and receipts.

### The Mandatory Statutory Notification Hubs:
Public tenders notify through specialized, authenticated legal channels that do NOT appear on public web notice boards:

| Notification Hub | Scope & Jurisdiction | Access Method | Critical Rules |
|---|---|---|---|
| **DEHú** *(Dirección Electrónica Habilitada única)* | Mandatory statutory notification portal for all Spanish Public Administrations (Ley 39/2015). | Certificate login (FNMT) via `grokbot` | Notifications expire after 10 calendar days. Must be checked every 3 hours. |
| **e-NOTUM** *(Consorci AOC)* | Mandatory notification portal for Generalitat de Catalunya, Catalan town councils, and universities. | Cl@ve / FNMT / OTP SMS | Houses esmenes and resolutions for Catalan public sector (e.g. AHC, Cassà, Girona). |
| **PLACSP Mis Comunicaciones** | State platform private contractor inbox (`contrataciondelestado.es`). | PLACSP bidder credentials / FNMT | Houses Art. 150.2 requirements and technical subsanaciones before they ever appear on public boards. |
| **PLYCA** | Mutualities and Social Security collaborating entities (e.g. Mutua Universal, Ibermutua). | PLYCA portal credentials | Closed private procurement portal; requires dedicated credential session. |
| **Vortal Messaging** | Portugal public contracts (SPMS, ANACOM) and international health consortia (FIBHGM). | Vortal account login | Communication happens strictly in closed private message threads (`PT1.MSG.*`). Read recipient lists carefully to avoid false low-bid alarms. |
| **SiREC Buzón** | Junta de Andalucía (Andalucía Directo, SiREC-Portal de Licitación). | Digital certificate | Private contractor notifications. |
| **European Portals** *(AWS, PLACE, Maximilien, Dematis, demat-ampa)* | France, Germany, EU institutions. | Espace entreprise logins | Private messages, negotiation rounds (*tours de négociation*), and *lettres de rejet / demandes de précisions*. Check erratum annexes for required replacement submissions. |

---

## 2. Replicated Parallel Horizontal Synchronization Architecture

> **Horizontal Scaling Axiom:** Accounts in the pool (`hao`, `h999`, `oulang`, `sami`) are **independent, horizontally replicated peer nodes**, NOT sequential silos or hierarchical dependencies. Workloads are sharded horizontally across all active accounts to smooth quota consumption, eliminate single points of failure, and maximize throughput.

### Why Vertical Account Siloing Fails:
- **Single Point of Failure (SPOF):** Partitioning accounts into fixed roles (e.g. Account A = filer, Account B = notifier) means that if the filer exhausts its quota or gets throttled, all submissions halt.
- **Artificial Quota Imbalance:** One account hits 100% quota rate limits while other accounts sit at 0% utilization.
- **Brittle Pipeline Coupling:** If Account A must wait for Account B to notify it before it can file, the system inherits all the latency, timeouts, and failure modes of a distributed chain.

### The Horizontal Replicated Model:
1. **Node Symmetry (Any Peer Can Execute Any Role):**
   - Every account in `grokbot pool` (`hao`, `h999`, `oulang`, `sami`) is an autonomous, symmetric peer worker capable of performing any operational step (sweeping, scraping, requirement parsing, document assembly, signing, and submission).
   - Any worker can execute an end-to-end task without relying on another account to hand off data.
2. **Horizontal Workload Sharding (Quota Smoothing & Parallel Throughput):**
   - Active tenders and platforms are sharded horizontally across all active pool accounts simultaneously.
   - Example 4-Way Shard Allocation:
     - **Peer Worker 1 (`hao`):** Shard A — PLACSP State Tenders (FPCM, Reina Sofía, JCISAT).
     - **Peer Worker 2 (`h999`):** Shard B — Regional & Municipal Portals (PSCP, Sobre Digital, Madrid, Canarias, Promálaga, Gomera).
     - **Peer Worker 3 (`oulang`):** Shard C — International & Mutuas (Vortal PT, PLYCA, FIBHGM).
     - **Peer Worker 4 (`sami`):** Shard D — French Platforms (PLACE, Dematis, AWSolutions).
   - Sharding divides the token and request load by $N$ accounts, smoothing quota consumption evenly and preventing any single account from hitting weekly caps or rate limits.
3. **Decoupled Horizontal Synchronization via Shared State:**
   - **Zero Inter-Account Dependencies:** Accounts NEVER communicate directly with each other or wait for inter-agent messages.
   - **Decoupled Single Source of Truth:** Coordination occurs strictly through the shared state layer:
     - **Twenty CRM:** Central opportunity pipeline and lifecycle states.
     - **Shared Git Repository (`samihalawa/licitaciones-packs`):** Signed PAdES/CAdES submission packages and SHA-256 manifests.
     - **Google Drive (`AI_REPORTS_MASTER/`):** Permanent audit ledgers, indices, and executive reports.
   - **Atomic Leases / Idempotency Locks:**
     - Before processing an `expediente`, the worker inspects the shared record.
     - If the action is already completed (valid CSV / registration receipt exists), the worker immediately skips to prevent duplicate filings.
     - If in progress, an atomic lease with a 15-minute TTL prevents duplicate concurrent submissions.
   - **Immediate Event Write-Through:**
     - The instant an Acta is discovered or a filing is submitted, the worker commits the official receipt (CSV, timestamp, SHA-256) to the shared state.
     - All other peer workers immediately see the updated status on their next cycle.
4. **Stateless Quota Resilience:**
   - If an account hits a temporary rate limit or cooldown, it idles autonomously without affecting any other worker.
   - Unclaimed tasks in that shard are immediately picked up by the remaining active peer accounts from the shared queue.

---

## 3. The Anti-False-Expired Guard & False-Alarm Defenses

> **Anti-False-Expired Axiom:** An agent is strictly prohibited from classifying any tender as `EXPIRED`, `LOST`, `CLOSED`, or `NO-GO` based on an unread notification, a past calendar date, or a generic low-bid notice.

### Core Safeguards:
1. **Explicit Exclusion Mandate:** A tender can ONLY be moved to `CLOSED_LOST` or `EXCLUIDO` upon reading a signed official resolution or minutes that explicitly names the bidder's Tax ID (NIF `02558168Q`) as excluded or awards the contract to a competitor.
2. **The Public Status Trap (`Evaluación` vs `Adjudicación`):**
   - On PLACSP, PSCP, and SiREC, the public status remains **`Evaluación`** even after the Mesa de Contratación selects the winning bidder!
   - The status only flips to `Adjudicación` *after* the winning bidder submits the pre-award documentation and the decree is signed.
   - **Mandatory Crawler Rule:** Scrapers must never filter solely by `Estado == "Adjudicada"`. They must enter the `Documentos` / `Actas` tab of every tender in `Evaluación` and inspect newly published PDFs for `Acta`, `Propuesta de Adjudicación`, or `Clasificación`.
3. **Abnormally Low Bid Defenses (e.g. Vortal / FIBHGM PA13-2026):**
   - When a platform issues an Article 149 / low-bid notice, download the PDF and check the named tenderer list.
   - If the notice names competitor companies (and the bidder's price is within the safe corridor), it is a **FALSE ALARM**; do NOT file unnecessary duplicate documents.
4. **Already-Filed Verification Defense (e.g. Mutualia 2026/085/01 & Canarias SER/2026/0000108561):**
   - Always query the portal's native `Pasos/acciones realizadas` or `Historial` before filing.
   - If an action shows `Respuesta Subsanación` with an existing timestamp/CSV, log it as **ALREADY COMPLETED** to prevent corrupting the submission with duplicate filings.

---

## 4. Current Proposed Awards & Execution Status

### Award 1: Fundación Parque Científico de Madrid (FPCM EXP.11.2026)
- **Tender:** FPCM EXP.11.2026 (Bioinformática & Soporte Computacional).
- **Amount:** 5.100,00 € + IVA.
- **Rank:** 1st place (100.00 / 100.00 points). Winning proposal (*propuesta de adjudicación*).
- **Status & History:**
  - Initial Art. 150.2 response filed on 28/10 at 18:32:35 Madrid (CSV `2SV69_MJT4X_UW50S_48701`, 9 files).
  - Statutory deadline: **13/11/2026 at 23:59 Madrid**.
  - Remediation: Completed turnkey 10-document pack assembled on GitHub `samihalawa/licitaciones-packs/FPCM_EXP11-2026_150_2/` curing missing DNI and covering both technical roles.

### Award 2: Museo Nacional Centro de Arte Reina Sofía (Exp. 202600000261)
- **Tender:** Exp. 202600000261 (Mantenimiento preventivo, correctivo y evolutivo del sistema de información dinámica).
- **Amount:** 28.000,00 € + IVA (33.880,00 € total).
- **Rank:** 1st place ("Licitador mejor valorado para la Adjudicación").
- **Status & History:**
  - Notice UUID: `2026-NOT-08928843` published on **29/10/2026 at 14:17 CET**.
  - Statutory deadline: **03/11/2026 at 23:59 Madrid** (NOT 29/10).
  - Prior documentary supplement registered on 27/10 at 02:49:01 Madrid via REGAGE (`REGAGE26e00092321786`).
  - Action: Prepare formal pre-award solvency and accounts verification dossier.

---

## 5. Pre-Award Turnkey Protocol (LCSP Article 150.2 & Apartado 19 CCP)

When proposed as winning bidder (*propuesta de adjudicación*), the contractor must submit the pre-award documentation pack within **7 to 10 business days**:

### Standard 10-Document Turnkey Pack:
1. **DNI / Personalidad Física:** Official scan of DNI (NIF `02558168Q`), front and back, valid and unexpired.
2. **ROLECE Certificate:** Official registration agreement (*Acuerdo de Inscripción*) in the Official Register of Tenderers and Classified Companies (`ROLECE2026S000069464`).
3. **AEAT Positive Certificate:** Certificate of being up to date with tax obligations (*al corriente de obligaciones tributarias*), validating Article 71.1.d LCSP.
4. **TGSS Positive Certificate:** Certificate of being up to date with Social Security payments (*al corriente con la Seguridad Social*), validating Article 71.1.d LCSP.
5. **Censal Tax Registration (Modelo 036/037):** Proof of active registration in IAE (Epígrafe 769.9 / 844 / 763) with high antiquity.
6. **Professional Liability Insurance (Seguro de Responsabilidad Civil):** Policy certificate (Hiscox Policy `3642985` covering ≥ 600.000 €).
7. **Declaration of Solvency for Newly Created Enterprises:** When applicable under Art. 86–90 LCSP, declaration under sworn statement for enterprises < 5 years old.
8. **Commitment of Resource Allocation (*Compromiso de Adscripción de Medios*):** Formal designation of material and personnel resources committed to contract execution.
   > **Sole Trader Two-Role Rule:** If the tender requires both a *Jefe de Proyecto* and a *Técnico Especializado*, the sole trader may assume both functions simultaneously. The commitment document must explicitly declare that both roles are assumed by the individual contractor under Spanish Ley 9/2017 LCSP.
9. **Technical CV & Academic Credentials:** Professional CV adhering strictly to authentic experience (no foreign corporate jargon, no unverified academic titles like "ingeniero" or "máster" unless backed by official Spanish homologated degrees).
10. **Definitive Guarantee Retention Request (*Anexo V - Retención en el Precio*):** Under Article 107.1 and 108.1 LCSP, formally request that the definitive guarantee (typically 5% of the award amount ex VAT) be constituted by means of withholding from the first invoice payments (*retención sobre el precio*), requiring ZERO upfront cash or bank guarantees.

### Supplementary Rectification Protocol:
If a pre-award filing is submitted with any minor omission (e.g. missing DNI scan, role clarification), the contractor has the legal right under administrative law to submit a **supplementary rectification pack** (*escrito de complemento/subsanación voluntaria*) into the open expediente before the term expires.

---

## 6. Pre-Filing Reconciliation Guard (Zero Duplicate Filings)

> **Non-Duplication Axiom:** Never assume that local files or prepared draft folders mean an action is currently owed or pending.

Before preparing or submitting any subsanación, response, or document pack:
1. **Live Portal Thread Inspection:** Query the live portal thread, general electronic register (*seu electrònica*), or *Sobre Digital* mailbox via `grokbot agent prompt`.
2. **Timestamp & CSV Matching:** Check if an entry for the exact same requirement or file has already been registered (matching CSV, entry number, or identical signed hash).
3. **Immediate Deduplication:** If already submitted, abort filing immediately, mark status as `YA PRESENTADA`, record the existing receipt timestamp and CSV in tracking records, and alert the team.

---

## 7. Sole Trader (Autónomo) Solvency & Capacity Framework

When bidding as a sole trader (*persona física / trabajador autónomo*), enforce the legal safeguards provided by Spanish Law 9/2017 (LCSP):
1. **ROLECE Primary Proof:** Official registration in ROLECE constitutes full legal proof of personality, legal capacity, absence of prohibitions to contract (Art. 71 LCSP), and financial/technical solvency up to classified limits.
2. **New Business Solvency Protection:** Under Article 89.1.h LCSP, economic operators with less than 5 years of activity are exempt from providing 3-year historical turnover figures. Solvency is accredited via personal professional qualifications, previous relevant contracts, or professional liability insurance.
3. **Professional Title Integrity:** Never claim titles or qualifications not verified by Spanish official registers. Describe technical experience factually with measurable deliverables, project dates, and technologies.
4. **European Single Procurement Document (DEUC / ESPD):** Declare economic, financial, and technical solvency under standard SME protections.

---

## 8. Financial Bidding Rules (Regla 17 Pricing Guard)

To maximize win probability while mathematically preventing disqualification:
1. **The 76%–91% Budget Corridor:** Bids must fall strictly between 76% and 91% of the Maximum Tender Budget (*Presupuesto Base de Licitación*, PBL without VAT).
2. **Abnormally Low Bid Safeguard:** Bidding at or above 76% prevents triggering automated *baja temeraria* thresholds under Article 149 LCSP and standard table rules (*Pliego de Cláusulas Administrativas Particulares*, PCAP).
3. **Effective Yield Floor:** Calculated effective hourly rate must strictly exceed **45 €/hour**.

---

## 9. Digital Signature Standards

All official documents, economic bids, DEUCs, and declarations must be signed with a valid Qualified Electronic Signature (QES) from an authorized Trust Service Provider (e.g. FNMT-RCM, Camerfirma):
- **PDF Documents:** PAdES-B-LTA standard containing cryptographic timestamps.
- **Envelopes & Raw Files:** CAdES-BES (`.p7s`) or XAdES (`.xsig`) where mandated by the specific platform (e.g., Vortal, PLACSP Sobre B).
- **Pre-Submission Signature Check:** Verify signature integrity before submission using native verification tools:
  ```bash
  pdfsig document_signed.pdf
  ```

---

## 10. Horizontal grokbot CLI Commands & Automation

Use the native `grokbot` CLI for parallel horizontal operations:

### Check Status & Multi-Account Pool
```bash
grokbot pool
grokbot agent roster [account]
```

### Dispatch Parallel Sharded Tasks Across Independent Peer Accounts
```bash
# Worker 1 (hao): Sweep Shard A (PLACSP State Tenders)
grokbot agent prompt hao filer-placsp "Check PLACSP Mis Comunicaciones from 01/10 onwards. List open requirements, deadlines, amounts. Return JSON."

# Worker 2 (h999): Sweep Shard B (Regional & Municipal Portals)
grokbot agent prompt h999 notif-placsp "Inspect PSCP, Madrid 01286, Canarias, and Promalaga inboxes for pending subsanaciones. Return JSON."

# Worker 3 (oulang): Sweep Shard C (International & Mutuas)
grokbot agent prompt oulang aee4a151-bf7f-4c29-addd-c79cce2f0a54 "Inspect Vortal PT and PLYCA message threads. Return JSON status."

# Worker 4 (sami): Sweep Shard D (French Portals)
grokbot agent prompt sami 8053fc4f-13ee-40a9-b267-f8a7ca2beb79 "Inspect PLACE and Dematis consultations for modified annexes. Return JSON."
```

### Tail Live Worker Logs
```bash
grokbot agent tail [account] [agent_id] [lines]
```

### Local Model Proxy
Ensure local proxy is running for background tool consumption:
```bash
grokbot serve --port 8000
# Endpoint: http://127.0.0.1:8000/v1/chat/completions
```

---

## 11. 3-Hour Recurring Sweep Protocol

For all active tenders in progress:
1. **Periodic Horizontal Sweep (Every 3 Hours):**
   - Shard the 7 platform families across all available accounts in `grokbot pool` (`hao`, `h999`, `oulang`, `sami`).
   - Query both public boards and authenticated private inboxes (DEHú, e-NOTUM, PLACSP, PLYCA, Vortal, PLACE, Dematis).
   - Scan for:
     - New notifications (*Notificaciones recibidas*).
     - Subsanación requirements with strict deadlines (counting business days under Ley 39/2015).
     - Published *Actas de la Mesa* or candidate rankings inside the `Documentos` tab.
     - Requests for decryption keys (*Claves de descifrado*).
2. **Immediate Action Protocol:**
   - If a valid requirement is addressed to the bidder: immediately acquire an atomic lease in the shared state, compute deadline date/hour, assemble required evidence, sign with FNMT QES, and file before the penultimate day of the term.
   - Commit official receipt (CSV, timestamp, SHA-256) to shared storage and update centralized operational logs (`AI_REPORTS_MASTER/INDEX.md`).
