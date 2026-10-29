---
name: public-tender-grokbot-orchestrator-skill
description: Autonomous public procurement and tender orchestration for sole traders and SME contractors using native grokbot CLI browser agents. Enforces the ground-truth axiom, multi-platform authenticated inbox sweeps (DEHú, e-NOTUM, PLACSP, PLYCA, Vortal, SiREC, AWS, PLACE, Dematis), legal telematics doctrines (PLACSP §4.6 retirar y sustituir, Art. 71.1.e dolo de-escalation, inalterability of tenders), anti-false-expired guards, pre-filing portal mailbox verification, non-duplicate filing guards, replicated parallel horizontal synchronization across grokbot pool accounts (zero single-account bottlenecks), FNMT PAdES/CAdES digital signing, LCSP Regla 17 pricing safeguards, and 3-hour recurring notification sweeps across all major EU and Spanish procurement platforms.
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
   - **Decoupled Single Source of Truth:** Coordination occurs strictly through the shared state layer (Twenty CRM, Git submission packages with SHA-256 manifests, Google Drive audit ledgers).
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

## 2b. Post-Filing Watch Built Into Every Bot (owner 29/10)

Misses came from three gaps: bots read only their inbox, while actas, awards and changed pliegos appear on each tender's public page; no bot owned a bid after filing it; and sweep prompts were counted on their first acknowledgement. So every filer, notif and Grok Bot description on every replica account ends with the same «VIGILANCIA POST-PRESENTACIÓN» block:

- A filed bid stays the bot's responsibility until formalised, excluded or awarded to someone else.
- On «BARRIDO», at the start of any work turn and after each filing, the bot covers, per platform: native inbox (incl. e-notifications deemed served after 10 days), its own submitted-offer list, and the public page of every open filed case (actas, admission lists, classification, proposed award, award, requerimientos, key requests, negotiation rounds, changed pliegos/annexes/price forms/Q&A).
- Only items naming NIF 02558168Q are Sami's. Owed acts are executed at once in deadline order with real documents. A material pliego/annex change with the deadline open means replacing the bid only if it would otherwise be invalid. Never pay. Email is never the source of truth.
- Final reply lines, money first: «⚑ PROPUESTO|ADJUDICADO …», «ACTO HECHO …», «REQUERIMIENTO … plazo … falta …», «SUSTITUIR … plazo … qué cambió», «PERDIDO … ganador y precio», per platform «SIN NOVEDADES …» or «BLOQUEO … error literal».

Controller protocol: one «BARRIDO» per bot family on one replica account per wave (≤4 concurrent turns per account); read final lines 20–30 minutes later with `grokbot agent tail <acct> <id> 60 --json`; report every ⚑ line to Sami at once; re-route any act whose bot stays silent (quota) to the same-named bot on another account. Read a bot's tail before prompting it; one controller per bot, never re-send a case another controller already handed it. Keep the block identical across accounts (push to idle bots only).

## 3. Legal Doctrine & Procedural Safeguards (Inalterability, Retractation & "Retirar y Sustituir")

> **Owner rule (29/10, governs this section):** do not withdraw, replace or resubmit filed bids over CV or credential wording; the HKU BEng / UPM course mentions are simply left out of new documents («just remove it going forward, don't resubmit, don't focus on that»). A filed bid stays as is whenever there is no fine risk (the only money risk is the 3 % art. 150.2 penalty). Withdraw-and-replace is used only when a material change to the pliegos or price annexes would make the filed bid invalid while the deadline is open.

> **Legal Telematics Axiom:** In electronic public procurement, ordinary email sent to contracting authorities has **zero legal validity** (Art. 347 LCSP, Ley 39/2015 LPACAP). All rectifications, withdrawals, or subsanaciones MUST be executed through official platform telematics (*Retirar y sustituir*) or the General Electronic Registry (REGAGE / Sede Electrónica).

### Core Doctrines (LCSP & Directives 2014/24/EU):
1. **The Inalterability Principle & Phase Bifurcation:**
   - Under Art. 132 LCSP and binding TJUE jurisprudence (*SAG ELV Slovensko*, C-599/10; *Manova*, C-336/12), tenders are binding and inalterable once the submission window closes.
   - **Plazo Abierto (Submission Window Open):** Bidders have the statutory right under PLACSP Guide v9.0 §4.6 (and regional/EU equivalents: SILEX, Sobre Digital, PLACE, eSubmission) to **withdraw and replace** (*retirar y sustituir*) an offer. The withdrawal completely invalidates the previous plica and receipt, allowing a fully sanitized proposal to be evaluated.
   - **Plazo Cerrado / Plicas Abiertas (Envelopes Opened):** Subsanación under Arts. 141 and 145 LCSP is strictly limited to formal administrative defects in Sobre 1. Technical proposals cannot be altered. If a non-accreditable qualification was scored, the proper strategy is formal withdrawal (*desistimiento*) before the award proposal to avoid forfeiture of bond.
2. **De-escalation of Sancioning Risk (Article 71.1.e LCSP):**
   - Prohibition to contract under Art. 71.1.e LCSP requires **dolo o culpa grave** (deliberate deceit to gain an advantage; Tribunal Supremo & JCCPE Informe 45/2018).
   - Spontaneous written retraction before the award proposal deactivates the intentional element (*dolo*), shielding the contractor from administrative penalties or exclusion from public procurement.
3. **Harmless Retraction on Non-Scoring Qualifications:**
   - Where qualifications were not part of the mandatory pliego requirements or technical scoring criteria, their retraction is legally harmless (*materialmente inocua*) to bid evaluation (e.g. CNIO 30/26SPF, Fomento de San Sebastián 058/26).

### Peremptory Execution Table & Urgent Deadlines:
| Expediente | Authority | Portal | Statutory Deadline | Required Action |
|---|---|---|---|---|
| **392026** | **SEGITTUR** (Min. Industria y Turismo) | **PLACSP** | **30/10/2026 10:00 h CET (URGENT)** | Leave as is unless the pliegos changed materially (owner 29/10: no resubmission over credential wording). |
| **AB-SER1-26-068** | **SERGAS** (Sanidade Galicia) | **SILEX** | **31/10/2026 23:59 h (3 days)** | Subsanación Sobre A: Anexo I DEUC, economic solvency (accounts/books), and technical solvency certificates in PDF PAdES. |
| **CSG-2026-0039** | **CRTVG** (Grafismo Electoral 29N) | **Notifica Xunta** | **Immediate (Urgencia Electoral)** | Access Sede Electrónica Xunta, open emergency notification, and fulfill Sobre A subsanación. |
| **AB-SER1-26-066** | **SERGAS** (Demanda Dinámica) | **SILEX** | **02/11/2026 14:00 h CET** | Leave as is unless the pliegos changed materially (owner 29/10: no resubmission over credential wording). |
| **SDA31/33/34** | **EMACSA** (Aguas de Córdoba) | **VORTAL** | **Active (Phone desk)** | Direct telephone communication with Begoña Maldonado (957 222 617 ext 2617 / 683 193 095) to unblock portal verification. |
| **CE18196HO01/25**| **Cetursa Sierra Nevada** | **Sede / Email** | **03/11/2026 (Debit date)** | Provide official bank debit receipt for Hiscox premium (333,10 €) once charged on 03/11. |
| **2026/08 (C15181)**| **Fundación Canal** | **PLACSP / Portal** | **Plazo abierto** | Re-submit complete offer in official portal explicitly replacing prior proposal. |
| **MNTB26-097** | **Fundación Museo Thyssen** | **PLACSP** | **Plazo abierto** | Leave as is unless the pliegos changed materially (owner 29/10: no resubmission over credential wording). |
| **ACE/9999.09** | **ACUAES** | **PLACSP** | **Plazo abierto** | Leave as is unless the pliegos changed materially (owner 29/10: no resubmission over credential wording). |
| **2026-01339** | **ISDEFE** | **PLACSP** | **11/11/2026** | Leave as is unless the pliegos changed materially (owner 29/10: no resubmission over credential wording). |

---

## 3b. Lessons Filtered From the 29/10 Evaluation (what holds, what does not)

Holds:
- **Channel:** buyer email changes nothing legally. Window open → withdraw/replace on the platform; window closed → the bid is frozen until the Mesa issues a formal requerimiento.
- **Euskadi Contratación:** a receipted bid cannot be withdrawn by the bidder. Withdrawal goes through platform support (kontratazioa@euskadi.eus) and the buyer's approval on the system, then re-upload before the deadline.
- **Deferred verification:** many pliegos check CVs and qualifications only for the proposed awardee (e.g. Donostia 058/26, Carátula cl. 17, ten business days). Nothing is owed before that requerimiento.
- **German procedures:** the procedure language (Verfahrenssprache Deutsch) and the Bietercockpit are exclusive. An English email or an off-portal filing is void.
- **Deemed-served e-notifications:** they are rejected 10 calendar days after being made available (art. 43.2 Ley 39/2015). Every certificate sede needs an owner. Grok Bot owns the inboxes no filer covers: DEHú, Junta de Castilla y León notifica (ae.jcyl.es/notifica), Xunta Notifica, e-NOTUM, Euskadi Mi Carpeta. An unowned sede is how a Registro de Formadores inclusion (JCyL EXP 1263, accept by 08/11) went unseen.
- **Negotiation rounds** (e.g. RRTP 66, Dematis 1182895): check whether each deposit must be e-signed, and watch for BAFO requests until the round closes.

Does not hold:
- **Replacing bids over CV or credential wording.** The owner rule in §3 governs.
- **Treating contract value as net cash margin.** Labour time is the main cost; rank by expected profit per effort.
- **Treating a rule as a fact without checking it (owner 29/10: «never assume, rules may not be true»).** Verified in the BOE text: art. 150.2 LCSP charges 3 % of the base budget excluding VAT, plus the art. 71.2.a ban risk, only when the best-ranked bidder fails to complete the requerimiento in time. It applies at any value, so the €80k cap is an owner preference, not a legal threshold. Its application to non-AAPP contracting entities (state companies, some foundations and consortia) depends on the pliego. Before any withdrawal (irreversible), read the lot's actual solvency, guarantee and penalty clauses. Withdraw only when the bid would demonstrably fail the award documentation and the pliego exposes a penalty, citing the clause. Outside Spain there is no equivalent penalty.
- **Private retainers, npm packages and cloud-credit alerts.** Not tender work; keep them out of tender sweeps.

## 4. The Anti-False-Expired Guard & False-Alarm Defenses

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

## 4b. False-Limitation Guard: «no account», «blocked», «not possible» (owner 30/10)

A bot or Keeper saying an account is missing, a portal is blocked or a step is impossible is a claim to verify, not a result.
- **Credentials:** before accepting «sin cuenta / not in vault», search EVERY 1Password vault through Connect (`GET /v1/vaults`, `/v1/vaults/<id>/items`). Match on title, `urls[].href` domain, `additionalInformation` and platform alias (Jaggaer, Atexo, Solaere, In-tend, Proactis, Panacea, GOV.UK One Login). On 29–30/10, nine accounts reported missing were in the Master Vault.
- **Tool limits:** «HTTP-only login impossible», «captcha» or «DCE needs identification» means switching to the bot's browser session and retrying once logged in, before reporting BLOQUEO.
- **Scout labels:** «account needed / registration needed» are hints. Run the vault search before routing a lot as blocked.
- **Reports:** a BLOQUEO must quote the search or literal error it rests on. The controller re-dispatches every false one at once and notes the item IDs.

## 5. Current Proposed Awards & Execution Status

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

## 6. Pre-Award Turnkey Protocol (LCSP Article 150.2 & Apartado 19 CCP)

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

---

## 7. Pre-Filing Reconciliation Guard (Zero Duplicate Filings)

> **Non-Duplication Axiom:** Never assume that local files or prepared draft folders mean an action is currently owed or pending.

Before preparing or submitting any subsanación, response, or document pack:
1. **Live Portal Thread Inspection:** Query the live portal thread, general electronic register (*seu electrònica*), or *Sobre Digital* mailbox via `grokbot agent prompt`.
2. **Timestamp & CSV Matching:** Check if an entry for the exact same requirement or file has already been registered (matching CSV, entry number, or identical signed hash).
3. **Immediate Deduplication:** If already submitted, abort filing immediately, mark status as `YA PRESENTADA`, record the existing receipt timestamp and CSV in tracking records, and alert the team.

---

## 8. Sole Trader (Autónomo) Solvency & Capacity Framework

When bidding as a sole trader (*persona física / trabajador autónomo*), enforce the legal safeguards provided by Spanish Law 9/2017 (LCSP):
1. **ROLECE Primary Proof:** Official registration in ROLECE constitutes full legal proof of personality, legal capacity, absence of prohibitions to contract (Art. 71 LCSP), and financial/technical solvency up to classified limits.
2. **New Business Solvency Protection:** Under Article 89.1.h LCSP, economic operators with less than 5 years of activity are exempt from providing 3-year historical turnover figures. Solvency is accredited via personal professional qualifications, previous relevant contracts, or professional liability insurance.
3. **Professional Title Integrity:** Never claim titles or qualifications not verified by Spanish official registers. Describe technical experience factually with measurable deliverables, project dates, and technologies.
4. **European Single Procurement Document (DEUC / ESPD):** Declare economic, financial, and technical solvency under standard SME protections.

---

## 9. Financial Bidding Rules (Regla 17 Pricing Guard)

To maximize win probability while mathematically preventing disqualification:
1. **The 76%–91% Budget Corridor:** Bids must fall strictly between 76% and 91% of the Maximum Tender Budget (*Presupuesto Base de Licitación*, PBL without VAT).
2. **Abnormally Low Bid Safeguard:** Bidding at or above 76% prevents triggering automated *baja temeraria* thresholds under Article 149 LCSP and standard table rules (*Pliego de Cláusulas Administrativas Particulares*, PCAP).
3. **Effective Yield Floor:** Calculated effective hourly rate must strictly exceed **45 €/hour**.

---

## 10. Digital Signature Standards

All official documents, economic bids, DEUCs, and declarations must be signed with a valid Qualified Electronic Signature (QES) from an authorized Trust Service Provider (e.g. FNMT-RCM, Camerfirma):
- **PDF Documents:** PAdES-B-LTA standard containing cryptographic timestamps.
- **Envelopes & Raw Files:** CAdES-BES (`.p7s`) or XAdES (`.xsig`) where mandated by the specific platform (e.g., Vortal, PLACSP Sobre B).
- **Pre-Submission Signature Check:** Verify signature integrity before submission using native verification tools:
  ```bash
  pdfsig document_signed.pdf
  ```

---

## 11. Horizontal grokbot CLI Commands & Automation

Use the native `grokbot` CLI for parallel horizontal operations:

### Check Status & Multi-Account Pool
```bash
grokbot pool
grokbot agent roster [account]
```

### Dispatch Parallel Sharded Tasks Across Independent Peer Accounts
```bash
# Worker 1 (hao): PLACSP State Tenders
grokbot agent prompt hao filer-placsp "Barrido PLACSP Mis Comunicaciones desde 01/10: una línea por expediente con plazo; ejecuta los actos debidos."

# Worker 2 (h999): Regional Portals (PSCP, Galicia, Euskadi)
grokbot agent prompt h999 notif-placsp "Inspect SILEX Galicia and PSCP inboxes for urgent subsanaciones. Return JSON."

# Worker 3 (oulang): International & Mutuas
grokbot agent prompt oulang aee4a151-bf7f-4c29-addd-c79cce2f0a54 "Inspect Vortal PT and PLYCA message threads. Return JSON status."

# Worker 4 (sami): French Portals
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

## 12. 3-Hour Recurring Sweep Protocol

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
   - If a valid requirement names the bidder's NIF: check the platform's own history for an earlier reply (no duplicate), compute the deadline date/hour, assemble the evidence, sign with FNMT QES and file before the penultimate day of the term.
   - Save the opened native receipt in the pack's `justificantes/` and `samihalawa/licitaciones-packs`, and report every proposed award or award to Sami in plain words at once.
