---
name: public-tender-grokbot-orchestrator-skill
description: Autonomous public procurement and tender orchestration for sole traders and SME contractors using native grokbot CLI browser agents. Enforces the ground-truth axiom, pre-filing portal mailbox verification, non-duplicate filing guards, FNMT PAdES/CAdES digital signing, LCSP Regla 17 pricing safeguards, and 3-hour recurring notification sweeps across all major EU and Spanish procurement platforms.
---

# Public Tender Grokbot Orchestrator Skill

## Overview

Use this skill to autonomously manage, bid, monitor, and resolve public procurement tenders across Spanish and European platforms (LCSP, Directives 2014/24/EU, Vortal, PSCP, PLACSP, Regional and Municipal portals) for sole traders (autónomos) and SMEs.

This skill operates strictly using the native **`grokbot` CLI** (`grokbot agent roster`, `prompt`, `tail`, `call`, `ask`, `pool`, `serve`).
**Zero Python relay scripts rule:** Never re-introduce `relay.py` or ad-hoc Python websocket wrappers. All browser agent orchestration is handled by the official `grokbot` package.

---

## 1. The Core Ground-Truth Axiom (Zero Premature Closures)

> **Ground-Truth Axiom:** An initial submission receipt proves *only* that an initial bid or a previous filing was delivered at that specific timestamp. It is **never** proof that the tender is won, finished, or closed.

1. **Portal Mailbox Overrides All Receipts:** The contracting authority's post-submission communications and requirements (*subsanaciones de defectos formales*, *aclaraciones técnicas*, *requerimientos de documentación previa a adjudicación* under art. 150 LCSP, *solicitudes de justificación de bajas temerarias*, *entrega de claves de descifrado*) inside the live portal mailbox override all local state and prior receipts.
2. **Terminal Status Conditions:** A case can ONLY be transitioned to a terminal status (*Formalizado*, *Excluido*, or *Adjudicado a otro licitador*) after direct, authenticated portal verification.
3. **No Blind Receipts:** Storing a PDF receipt (`justificant.pdf`, `recibo.acdr`, `registre.pdf`) simply documents that an action took place; it does not eliminate the duty to continue monitoring the case mailbox until final formalization or definitive closure.

---

## 2. Pre-Filing Reconciliation Guard (Zero Duplicate Filings)

> **Non-Duplication Axiom:** Never assume that local files or prepared draft folders mean an action is currently owed or pending.

Before preparing or submitting any subsanación, response, or document pack:
1. **Live Portal Thread Inspection:** Query the live portal thread, general electronic register (*seu electrònica*), or *Sobre Digital* mailbox via `grokbot agent prompt`.
2. **Timestamp & CSV Matching:** Check if an entry for the exact same requirement or file has already been registered (matching CSV, entry number, or identical signed hash).
3. **Immediate Deduplication:** If already submitted, abort filing immediately, mark status as `YA PRESENTADA`, record the existing receipt timestamp and CSV in tracking records, and alert the team.

---

## 3. The Misattribution Trap Guard (Actas & Tablones Analysis)

When reading *Actas de la Mesa de Contratación*, *Listas de Admitidos*, or public board announcements:
1. **Multiple Bidders Caution:** Official tender minutes evaluate all bidders in a single document.
2. **Explicit NIF Verification:** Always verify that any listed defect, requested subsanación, or low-bid justification specifically names the bidder's Tax ID (NIF / CIF) before treating it as an owed action.
3. **Never Act on Competitor Issues:** If an acta notes that another bidder (e.g., Auren, Indra, etc.) incurred an abnormally low bid or missing certificate, record the competitive advantage but do not prepare any submission.

---

## 4. Autonomous Worker (Autónomo / SME) Eligibility Gates

Every prospective tender must pass strict eligibility filters:
- **Entity Type:** Persona física / Autónomo (Sole Trader, microempresa).
- **Service Scope:** 100% remote services (IT consultancy, software architecture, full-stack engineering, AI/ML engineering, prompt engineering, technical assistance, data processing, systems integration).
- **Hard Disqualifiers:**
  - Mandatory on-site presence or physical guard duty.
  - Physical construction, hardware maintenance, security, or cleaning.
  - Maximum annual turnover requirement exceeding 1.5x of verified annual billing (LCSP art. 87).
  - Excessive team composition quotas exceeding sole-trader / micro-team capacity without sub-contracting approval.
- **Solvency Reliance:** Leverage the European Single Procurement Document (DEUC / ESPD) declaring economic, financial, and technical solvency under standard SME protections.

---

## 5. Financial Bidding Rules (Regla 17 Pricing Guard)

To maximize win probability while mathematically preventing disqualification:
1. **The 76%–91% Budget Corridor:** Bids must fall strictly between 76% and 91% of the Maximum Tender Budget (*Presupuesto Base de Licitación*, PBL without VAT).
2. **Abnormally Low Bid Safeguard:** Bidding at or above 76% prevents triggering automated *baja temeraria* thresholds under Article 149 LCSP and standard table rules (*Pliego de Cláusulas Administrativas Particulares*, PCAP).
3. **Effective Yield Floor:** Calculated effective hourly rate must strictly exceed **45 €/hour**.

---

## 6. Digital Signature Standards

All official documents, economic bids, DEUCs, and declarations must be signed with a valid Qualified Electronic Signature (QES) from an authorized Trust Service Provider (e.g. FNMT-RCM, Camerfirma):
- **PDF Documents:** PAdES-B-LTA standard containing cryptographic timestamps.
- **Envelopes & Raw Files:** CAdES-BES (`.p7s`) or XAdES (`.xsig`) where mandated by the specific platform (e.g., Vortal, PLACSP Sobre B).
- **Pre-Submission Signature Check:** Verify signature integrity before submission using native verification tools:
  ```bash
  pdfsig document_signed.pdf
  ```

---

## 7. Platform Architecture & Filer Roster

Procurement platforms are segmented into 7 operational families, managed via dedicated browser agents:

| Agent ID | Platform Family | Scope & Portals |
|---|---|---|
| `filer-pscp` | Catalunya | Plataforma de Serveis de Contractació Pública (PSCP), Sobre Digital, e-Licita, AHC, Generalitat, Consorcis. |
| `filer-es-1` | PLACSP & State | Plataforma de Contratación del Sector Público (PLACSP), Ministerios, DGT, CNMC, BOE. |
| `filer-es-2` | Regional Autonomous | Contratación Pública en Euskadi, SiREC Andalucía, Portal de Contratación de Madrid, Contratos de Galicia. |
| `filer-eu-1` | European Union | SIMAP, eNotices2, Tenders Electronic Daily (TED). |
| `filer-eu-2` | International & Enterprise | Vortal (Portugal SPMS, etc.), Jaggaer, BravoSolution, Achilles. |
| `filer-local` | Municipal & Local | Seu Electrònica municipal, e-TRAM (AOC), Diputació de Barcelona, Diputació de Girona. |
| `filer-general` | Ad-Hoc & General Support | Ad-hoc tender platforms, fallback filing, document rendering, and PAdES/CAdES signature validation. |

---

## 8. grokbot CLI Commands & Automation

Use the native `grokbot` CLI for all agent operations:

### Check Status & Active Agents
```bash
grokbot agent roster [account]
grokbot pool
```

### Dispatch Task to Specialized Filer
```bash
# Example: Inspect mailbox for pending subsanaciones
grokbot agent prompt default filer-pscp "Check mailbox for tender AHC-2027-9. Inspect tab 'Notificacions' and 'Esmenes'. Do not submit anything. Return JSON with any pending requirements."

# Example: Check Vortal message thread for prior submissions
grokbot agent prompt default filer-eu-2 "Open tender SPMS 20260322 on Vortal. Inspect thread PT1.MSG.5963058. Return status of message 348160161 and confirm if PAdES/CAdES files are already registered."
```

### Tail Live Filer Logs
```bash
grokbot agent tail default filer-pscp 50
```

### Direct WebSocket Method Execution
```bash
grokbot agent call default getAgentStatus '{"agentId":"filer-es-1"}'
```

### Local Model Proxy
Ensure local proxy is running for background tool consumption:
```bash
grokbot serve --port 8000
# Endpoint: http://127.0.0.1:8000/v1/chat/completions
```

---

## 9. 3-Hour Recurring Sweep Protocol

For all active tenders in progress:
1. **Periodic Sweep (Every 3 Hours):**
   - Query all 7 platform families via their specialized filers.
   - Scan for:
     - New notifications (*Notificaciones recibidas*).
     - Subsanación requirements with strict deadlines (counting business days under Ley 39/2015).
     - Published *Actas de la Mesa* or candidate rankings.
     - Requests for decryption keys (*Claves de descifrado*).
2. **Immediate Action Protocol:**
   - If a valid requirement is addressed to the bidder: immediately alert, compute deadline date/hour, assemble required evidence, sign with FNMT QES, and file before the penultimate day of the term.
   - Update centralized operational logs and master indices (`AI_REPORTS_MASTER/INDEX.md`).
