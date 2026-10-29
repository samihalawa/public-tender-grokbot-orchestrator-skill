---
name: public-tender-grokbot-orchestrator-skill
description: Autonomous public procurement and tender orchestration for sole traders and SME contractors using native grokbot CLI browser agents. Enforces the ground-truth axiom, multi-platform authenticated inbox sweeps (DEHú, e-NOTUM, PLACSP, PLYCA, Vortal, SiREC, AWS), anti-false-expired guards, pre-filing portal mailbox verification, non-duplicate filing guards, FNMT PAdES/CAdES digital signing, LCSP Regla 17 pricing safeguards, and 3-hour recurring notification sweeps across all major EU and Spanish procurement platforms.
---

# Public Tender Grokbot Orchestrator Skill

## Overview

Use this skill to autonomously manage, bid, monitor, and resolve public procurement tenders across Spanish and European platforms (LCSP, Directives 2014/24/EU, Vortal, PSCP, PLACSP, Regional and Municipal portals) for sole traders (autónomos) and SMEs.

This skill operates strictly using the native **`grokbot` CLI** (`grokbot agent roster`, `prompt`, `tail`, `call`, `ask`, `pool`, `serve`).
**Zero Python relay scripts rule:** Never re-introduce `relay.py` or ad-hoc Python websocket wrappers. All browser agent orchestration is handled by the official `grokbot` package.

---

## 1. The Core Ground-Truth Axiom & Multi-Platform Inbox Matrix

> **Ground-Truth Axiom:** An initial submission receipt proves *only* that an envelope was delivered at that timestamp. It is **never** proof that the tender is completed, awarded, or closed. The contracting authority's communications inside authenticated portal mailboxes override all prior states and receipts.

### The Mandatory Statutory Notification Hubs:
Public tenders notify through specialized, authenticated legal channels that do NOT appear on public web notice boards:

| Notification Hub | Scope & Jurisdiction | Access Method | Critical Rules |
|---|---|---|---|
| **DEHú** *(Dirección Electrónica Habilitada única)* | Mandatory statutory notification portal for all Spanish Public Administrations (Ley 39/2015). | Certificate login (FNMT) via `grokbot` | Notifications expire after 10 calendar days. Must be checked every 3 hours. |
| **e-NOTUM** *(Consorci AOC)* | Mandatory notification portal for Generalitat de Catalunya, Catalan town councils, and universities. | Cl@ve / FNMT / OTP SMS | Houses esmenes and resolutions for Catalan public sector (e.g. AHC, Cassà, Girona). |
| **PLACSP Mis Comunicaciones** | State platform private contractor inbox (`contrataciondelestado.es`). | PLACSP bidder credentials / FNMT | Houses Art. 150.2 requirements and technical subsanaciones before they ever appear on public boards. |
| **PLYCA** | Mutualities and Social Security collaborating entities (e.g. Mutua Universal, Ibermutua). | PLYCA portal credentials | Closed private procurement portal; requires dedicated credential session. |
| **Vortal Messaging** | Portugal public contracts (SPMS, ANACOM) and international health consortia. | Vortal account login | Communication happens strictly in closed private message threads (`PT1.MSG.*`). |
| **SiREC Buzón** | Junta de Andalucía (Andalucía Directo, SiREC-Portal de Licitación). | Digital certificate | Private contractor notifications. |
| **European Portals** *(AWS, PLACE, Maximilien, demat-ampa)* | France, Germany, EU institutions. | Espace entreprise logins | Private messages and *lettres de rejet / demandes de précisions*. |

---

## 2. The Anti-False-Expired Guard (The FPCM EXP. 11.2026 Lesson)

> **Anti-False-Expired Axiom:** An agent is strictly prohibited from classifying any tender as `EXPIRED`, `LOST`, `CLOSED`, or `NO-GO` based on an unread notification or a passed calendar date.

1. **Explicit Exclusion Mandate:** A tender can ONLY be moved to `CLOSED_LOST` or `EXCLUIDO` upon reading a signed official resolution or minutes that explicitly names the bidder's Tax ID (NIF `02558168Q`) as excluded or awards the contract to a competitor.
2. **The Public Status Trap (`Evaluación` vs `Adjudicación`):**
   - On PLACSP, PSCP, and SiREC, the public status remains **`Evaluación`** even after the Mesa de Contratación selects the winning bidder!
   - The status only flips to `Adjudicación` *after* the winning bidder submits the pre-award documentation and the decree is signed.
   - **Mandatory Crawler Rule:** Scrapers must never filter solely by `Estado == "Adjudicada"`. They must enter the `Documentos` / `Actas` tab of every tender in `Evaluación` and inspect newly published PDFs for `Acta`, `Propuesta de Adjudicación`, or `Clasificación`.

---

## 3. Pre-Award Turnkey Protocol (LCSP Article 150.2 & Apartado 19 CCP)

When proposed as winning bidder (*propuesta de adjudicación*), the contractor must submit the pre-award documentation pack within **7 to 10 business days**:

### Standard 9-Document Turnkey Pack:
1. **DNI / Personalidad Física:** Official DNI copy of the contractor.
2. **Situación Censal e IAE:** Certificado de situación censal AEAT (Modelo 036/037) and declaration of IAE exemption under Art. 82.1.c TRLRHL (< 1.000.000 € turnover).
3. **Certificado Corriente AEAT:** Positive tax clearance certificate for public contracting.
4. **Certificado Corriente TGSS:** Positive Social Security clearance certificate.
5. **Acreditación de Solvencia (ROLECE):** Official certificate of registration in the *Registro Oficial de Licitadores y Empresas Clasificadas del Estado* (Art. 96 LCSP).
6. **Compromiso de Adscripción de Medios:** Declaration committing technical personnel. In sole-trader setups, the freelancer formally covers both management and specialized technical execution.
7. **Compromiso de Acuerdos de Nivel de Servicio (SLA):** Ratification of response times (Level 1 ≤ 2h, Level 2 ≤ 8h).
8. **Curriculum Vitae y Titulaciones:** Detailed technical CV and university qualifications.
9. **Garantía Definitiva (Anexo V - Retención en Precio):**
   - **The Zero-Cash Standard:** Whenever permitted by the pliegos, always execute **Anexo V (Solicitud de retención en las primeras facturas)** under Art. 107.2 LCSP.
   - Eliminates bank guarantee (*aval bancario*) delays, fees, and frozen capital.

### The Supplementary Rectification Right:
If a pre-award filing is submitted with any minor omission (e.g. missing DNI scan, role clarification), the contractor has the legal right under administrative law to submit a **supplementary rectification pack** (*escrito de complemento/subsanación voluntaria*) into the open expediente before the term expires.

---

## 4. Pre-Filing Reconciliation Guard (Zero Duplicate Filings)

> **Non-Duplication Axiom:** Never assume that local files or prepared draft folders mean an action is currently owed or pending.

Before preparing or submitting any subsanación, response, or document pack:
1. **Live Portal Thread Inspection:** Query the live portal thread, general electronic register (*seu electrònica*), or *Sobre Digital* mailbox via `grokbot agent prompt`.
2. **Timestamp & CSV Matching:** Check if an entry for the exact same requirement or file has already been registered (matching CSV, entry number, or identical signed hash).
3. **Immediate Deduplication:** If already submitted, abort filing immediately, mark status as `YA PRESENTADA`, record the existing receipt timestamp and CSV in tracking records, and alert the team.

---

## 5. The Misattribution Trap Guard (Actas & Tablones Analysis)

When reading *Actas de la Mesa de Contratación*, *Listas de Admitidos*, or public board announcements:
1. **Multiple Bidders Caution:** Official tender minutes evaluate all bidders in a single document.
2. **Explicit NIF Verification:** Always verify that any listed defect, requested subsanación, or low-bid justification specifically names the bidder's Tax ID (NIF / CIF) before treating it as an owed action.
3. **Never Act on Competitor Issues:** If an acta notes that another bidder (e.g., Auren, Indra, etc.) incurred an abnormally low bid or missing certificate, record the competitive advantage but do not prepare any submission.

---

## 6. Autonomous Worker (Autónomo / SME) Eligibility Gates

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

## 7. Financial Bidding Rules (Regla 17 Pricing Guard)

To maximize win probability while mathematically preventing disqualification:
1. **The 76%–91% Budget Corridor:** Bids must fall strictly between 76% and 91% of the Maximum Tender Budget (*Presupuesto Base de Licitación*, PBL without VAT).
2. **Abnormally Low Bid Safeguard:** Bidding at or above 76% prevents triggering automated *baja temeraria* thresholds under Article 149 LCSP and standard table rules (*Pliego de Cláusulas Administrativas Particulares*, PCAP).
3. **Effective Yield Floor:** Calculated effective hourly rate must strictly exceed **45 €/hour**.

---

## 8. Digital Signature Standards

All official documents, economic bids, DEUCs, and declarations must be signed with a valid Qualified Electronic Signature (QES) from an authorized Trust Service Provider (e.g. FNMT-RCM, Camerfirma):
- **PDF Documents:** PAdES-B-LTA standard containing cryptographic timestamps.
- **Envelopes & Raw Files:** CAdES-BES (`.p7s`) or XAdES (`.xsig`) where mandated by the specific platform (e.g., Vortal, PLACSP Sobre B).
- **Pre-Submission Signature Check:** Verify signature integrity before submission using native verification tools:
  ```bash
  pdfsig document_signed.pdf
  ```

---

## 9. Platform Architecture & Filer Roster

Procurement platforms are segmented into 7 operational families, managed via dedicated browser agents:

| Agent ID | Platform Family | Scope & Portals |
|---|---|---|
| `filer-pscp` | Catalunya | Plataforma de Serveis de Contractació Pública (PSCP), Sobre Digital, e-Licita, e-NOTUM, AHC, Generalitat, Consorcis. |
| `filer-es-1` | PLACSP & State | Plataforma de Contratación del Sector Público (PLACSP), DEHú, Ministerios, DGT, CNMC, BOE. |
| `filer-es-2` | Regional Autonomous | Contratación Pública en Euskadi, SiREC Andalucía, Portal de Contratación de Madrid, Contratos de Galicia. |
| `filer-eu-1` | European Union | SIMAP, eNotices2, Tenders Electronic Daily (TED). |
| `filer-eu-2` | International & Enterprise | Vortal (Portugal SPMS, etc.), Jaggaer, BravoSolution, Achilles, PLYCA. |
| `filer-local` | Municipal & Local | Seu Electrònica municipal, e-TRAM (AOC), Diputació de Barcelona, Diputació de Girona. |
| `filer-general` | Ad-Hoc & General Support | Ad-hoc tender platforms, fallback filing, document rendering, and PAdES/CAdES signature validation. |

---

## 10. grokbot CLI Commands & Automation

Use the native `grokbot` CLI for all agent operations:

### Check Status & Active Agents
```bash
grokbot agent roster [account]
grokbot pool
```

### Dispatch Task to Specialized Filer
```bash
# Example: Inspect PLACSP and DEHú inboxes for pending notifications
grokbot agent prompt default filer-es-1 "Check PLACSP Mis Comunicaciones and DEHú inboxes for active tenders. Inspect Documentos tabs for all tenders in 'Evaluación'. Return JSON with any pending requerimientos or actas."

# Example: Inspect e-NOTUM and Sobre Digital inboxes
grokbot agent prompt default filer-pscp "Check e-NOTUM and Sobre Digital mailboxes for any new notifications or esmenes. Do not submit anything. Return JSON status."

# Example: Check Vortal message thread for prior submissions
grokbot agent prompt default filer-eu-2 "Open tender SPMS 20260322 on Vortal. Inspect thread PT1.MSG.5963058. Return status of message 348160161 and confirm if PAdES/CAdES files are already registered."
```

### Tail Live Filer Logs
```bash
grokbot agent tail default filer-es-1 50
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

## 11. 3-Hour Recurring Sweep Protocol

For all active tenders in progress:
1. **Periodic Sweep (Every 3 Hours):**
   - Query all 7 platform families via their specialized filers across both public boards and authenticated private inboxes (DEHú, e-NOTUM, PLACSP, PLYCA, Vortal).
   - Scan for:
     - New notifications (*Notificaciones recibidas*).
     - Subsanación requirements with strict deadlines (counting business days under Ley 39/2015).
     - Published *Actas de la Mesa* or candidate rankings inside the `Documentos` tab.
     - Requests for decryption keys (*Claves de descifrado*).
2. **Immediate Action Protocol:**
   - If a valid requirement is addressed to the bidder: immediately alert, compute deadline date/hour, assemble required evidence, sign with FNMT QES, and file before the penultimate day of the term.
   - Update centralized operational logs and master indices (`AI_REPORTS_MASTER/INDEX.md`).
