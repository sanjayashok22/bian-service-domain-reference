# Bian-service-domain-reference
Practical BIAN service domain reference for Core Banking, Payments &amp; Financial Crime platform design
ISO 20022 Payments Guide
> A practitioner's guide to ISO 20022 migration — message structures, payment rail mapping, SEPA SCT Inst readiness, SWIFT CBPR+ coexistence, and compliance-by-design architecture.
Built from 20+ years of hands-on delivery across Tier-1 global banks, payment hubs, and FinTech platforms spanning Europe, Southeast Asia, GCC, and the US.
---
Who this is for
Product Owners and Product Managers building or migrating payment platforms
Business Analysts translating ISO 20022 requirements into user stories and acceptance criteria
Solution Architects designing API-first, microservices-based payment hubs
Compliance and Operations teams preparing for regulatory cutover deadlines
FinTech founders building greenfield platforms on modern payment rails
---
What is ISO 20022?
ISO 20022 is the global standard for financial messaging — a single, rich data language replacing dozens of legacy formats (SWIFT MT, EDIFACT, proprietary formats) across payment rails worldwide.
It uses structured XML messages with far richer data than legacy formats, enabling:
Richer remittance data — structured creditor/debtor information, purpose codes, regulatory identifiers
Straight-Through Processing (STP) — machine-readable fields reduce manual intervention and repair queues
Regulatory compliance — FATF Recommendation 16 (Travel Rule), AML screening, sanctions data embedded in the message
Interoperability — one standard across SWIFT, SEPA, domestic high-value systems, and real-time rails
The migration timeline at a glance
Rail	Go-Live	Notes
SWIFT CBPR+ (cross-border)	Nov 2022	Coexistence with MT until Nov 2025
SEPA SCT Inst (Europe)	Jan 2025 mandatory	EPC rulebook update
UK CHAPS	Apr 2023	Bank of England mandated
Fedwire (US)	Mar 2025	Full migration
TARGET2 / T2 (ECB)	Mar 2023	Replaced with T2 platform
HVPS+ (global)	Ongoing	Country-by-country
---
Message architecture
The pacs family — payment clearing and settlement
The `pacs` (Payments Clearing and Settlement) namespace covers the core money movement messages. These are the messages your payment hub sends and receives most frequently.
```
pacs.008  —  Customer Credit Transfer (FI to FI)
pacs.009  —  Financial Institution Credit Transfer (bank-to-bank, covers)
pacs.002  —  Payment Status Report (acknowledgement / rejection)
pacs.003  —  Direct Debit (FI to FI)
pacs.004  —  Payment Return
pacs.007  —  Payment Reversal
pacs.028  —  Payment Status Request (investigation)
```
The pain family — payment initiation
The `pain` (Payments Initiation) namespace covers messages between a corporate/customer and their bank.
```
pain.001  —  Customer Credit Transfer Initiation (corporate to bank)
pain.002  —  Customer Payment Status Report
pain.007  —  Customer Payment Reversal
pain.008  —  Customer Direct Debit Initiation
```
The camt family — cash management
```
camt.052  —  Bank to Customer Account Report (intraday)
camt.053  —  Bank to Customer Statement (end of day)
camt.054  —  Bank to Customer Debit/Credit Notification
camt.056  —  FI to FI Payment Cancellation Request
camt.029  —  Resolution of Investigation
```
---
pacs.008 deep dive — customer credit transfer
The `pacs.008` is the workhorse of cross-border payments. Understanding its structure is essential for any payment product build.
```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:pacs.008.001.08">
  <FIToFICstmrCdtTrf>

    <!-- Group Header: applies to all transactions in this message -->
    <GrpHdr>
      <MsgId>MSGID-20240115-001</MsgId>           <!-- Unique message ID -->
      <CreDtTm>2024-01-15T09:30:00</CreDtTm>
      <NbOfTxs>1</NbOfTxs>                         <!-- Transaction count -->
      <SttlmInf>
        <SttlmMtd>INGA</SttlmMtd>                  <!-- Settlement method -->
      </SttlmInf>
    </GrpHdr>

    <!-- Credit Transfer Transaction Information -->
    <CdtTrfTxInf>
      <PmtId>
        <InstrId>INSTR-001</InstrId>               <!-- Instruction ID (sender ref) -->
        <EndToEndId>E2E-CUST-REF-001</EndToEndId>  <!-- End-to-end reference — preserved across entire chain -->
        <UETR>f47ac10b-58cc-4372-a567-0e02b2c3d479</UETR> <!-- Unique End-to-end Transaction Ref (UUID) -->
      </PmtId>

      <IntrBkSttlmAmt Ccy="EUR">10000.00</IntrBkSttlmAmt>  <!-- Interbank settlement amount -->
      <IntrBkSttlmDt>2024-01-15</IntrBkSttlmDt>             <!-- Value date -->

      <!-- Debtor — the customer sending the payment -->
      <Dbtr>
        <Nm>Acme Corporation GmbH</Nm>
        <PstlAdr>
          <Ctry>DE</Ctry>
          <AdrLine>Friedrichstrasse 100, 10117 Berlin</AdrLine>
        </PstlAdr>
      </Dbtr>
      <DbtrAcct>
        <Id><IBAN>DE89370400440532013000</IBAN></Id>
      </DbtrAcct>
      <DbtrAgt>
        <FinInstnId><BICFI>DEUTDEDB</BICFI></FinInstnId>  <!-- Debtor's bank BIC -->
      </DbtrAgt>

      <!-- Creditor — the beneficiary -->
      <Cdtr>
        <Nm>Global Supplies Ltd</Nm>
        <PstlAdr>
          <Ctry>GB</Ctry>
          <AdrLine>10 King Street, London EC2V 8BB</AdrLine>
        </PstlAdr>
      </Cdtr>
      <CdtrAcct>
        <Id><IBAN>GB29NWBK60161331926819</IBAN></Id>
      </CdtrAcct>
      <CdtrAgt>
        <FinInstnId><BICFI>NWBKGB2L</BICFI></FinInstnId>  <!-- Creditor's bank BIC -->
      </CdtrAgt>

      <!-- Remittance information — the ISO 20022 game-changer -->
      <RmtInf>
        <Strd>
          <CdtrRefInf>
            <Tp><CdOrPrtry><Cd>SCOR</Cd></CdOrPrtry></Tp>
            <Ref>INV-2024-8891</Ref>                <!-- Structured invoice reference -->
          </CdtrRefInf>
        </Strd>
      </RmtInf>

    </CdtTrfTxInf>
  </FIToFICstmrCdtTrf>
</Document>
```
Key fields for product owners and BAs
Field	Path	Why it matters
UETR	`PmtId/UETR`	UUID that tracks the payment end-to-end across all banks in the chain. Essential for investigations and gpi tracking.
End-to-end ID	`PmtId/EndToEndId`	Customer's reference — must be passed through unchanged. A common defect source in migrations.
Structured remittance	`RmtInf/Strd`	Enables automated reconciliation. Absent in SWIFT MT103 — one of the biggest ISO 20022 benefits.
Purpose code	`Purp/Cd`	Regulatory classification (e.g. SALA = salary, TAXS = tax payment). Required for some jurisdictions.
Regulatory reporting	`RgltryRptg`	AML/sanctions data, FATF Travel Rule fields. Compliance-by-design — embedded in the message, not a side-channel.
---
SEPA SCT Inst migration — lessons from delivery
What is SEPA SCT Inst?
SEPA Instant Credit Transfer (SCT Inst) is the European real-time payment rail — payments settled in under 10 seconds, 24/7/365, across 36 SEPA countries. From January 2025, PSPs in the Eurozone are mandatorily required to receive SCT Inst payments. Send capability follows in October 2025.
The EPC rulebook change — what moved to ISO 20022
The European Payments Council (EPC) migrated SCT Inst from a proprietary XML format to full ISO 20022 `pacs.008` / `pacs.002` / `pacs.004` messaging. This affected every payment hub, clearing connection, and settlement interface in Europe.
Migration checklist — lessons from a live European bank programme
Technical readiness
[ ] Message schema validation — implement XSD validation for all inbound/outbound pacs messages before processing
[ ] Character set migration — ISO 20022 uses UTF-8; legacy SWIFT MT uses SWIFT character set (no special characters). Map and validate all name/address fields
[ ] Field length changes — debtor/creditor name: 70 chars (MT) → 140 chars (MX). Truncation logic required for legacy systems receiving translated messages
[ ] UETR generation — every outbound payment needs a UUID4 UETR. Implement at payment initiation, not at the hub
[ ] End-to-end ID preservation — validate end-to-end ID passes through every system in the chain unchanged (a common integration defect)
[ ] Structured address migration — MT uses unstructured address lines; ISO 20022 has structured `StrtNm`, `BldgNb`, `PstCd`, `TwnNm`. Define mapping rules per corridor
Processing and SLA
[ ] 10-second SLA — end-to-end from debtor PSP to creditor PSP. Map your internal processing steps and identify bottlenecks (sanctions screening is the most common bottleneck)
[ ] Sanctions screening latency — integrate screening in-message, not as a separate queue. Target < 500ms for automated matches; define exception handling for manual review
[ ] Reject vs. return logic — pacs.002 (status report) for pre-settlement rejects; pacs.004 (return) for post-settlement. Define which errors trigger which response
[ ] Timeout handling — what happens if you don't receive pacs.002 within SLA? Define retry, timeout, and investigation flows
[ ] 24/7 operations — SCT Inst has no settlement windows. Ensure operations, liquidity management, and incident response cover all hours
Compliance and AML
[ ] Regulatory reporting fields — map FATF Travel Rule data (originator/beneficiary name, account, address) to `RgltryRptg` block
[ ] Purpose codes — agree with compliance on mandatory vs. optional purpose codes per corridor
[ ] Sanctions screening on ISO 20022 structured data — screening engines trained on MT free text may miss structured ISO 20022 name variants; recalibrate models
[ ] AML narrative automation — ISO 20022 structured remittance data enables GenAI-powered SAR narrative generation. Define data retention for audit trail
Cutover planning
[ ] Coexistence period — during SWIFT MT/MX coexistence (until Nov 2025), maintain translation layer for counterparties not yet on MX
[ ] Bilateral testing — agree test message sets with key correspondent banks and clearing houses (EBA Clearing, STET)
[ ] Cutover weekend — define go/no-go criteria, rollback triggers, and post-cutover monitoring thresholds (STP rate, reject rate, latency P95)
[ ] Legacy system decommission — plan MT message processing decommission only after 90+ days of stable MX operation
---
SWIFT CBPR+ — cross-border payments and coexistence
MT to MX translation — the key differences
Concept	SWIFT MT103	ISO 20022 pacs.008
Message format	FIN (proprietary)	XML
Remittance info	Field 70 — 4 × 35 chars, unstructured	`RmtInf` — structured or 140 chars unstructured
Debtor name	Field 50 — 35 chars	`Dbtr/Nm` — 140 chars
Charges	Field 71A — OUR/SHA/BEN	`ChrgBr` — DEBT/SHAR/CRED
Regulatory data	No standard field	`RgltryRptg` block
Tracking	No standard reference	UETR — UUID tracked via gpi
Character set	SWIFT charset (limited)	UTF-8
Translation rules — where product teams must make decisions
During the SWIFT coexistence period, your payment hub must translate between MT and MX for counterparties on different versions. These translation decisions require explicit product decisions — not just technical mapping:
Truncation policy — when MX data is longer than MT allows, define: truncate silently? reject? notify originator?
Unstructured address handling — MT address lines mapped to ISO 20022 structured address. Define fallback when structured data is unavailable (use `AdrLine` free text vs. reject).
Charges translation — MT `OUR`/`SHA`/`BEN` maps approximately but not exactly to ISO 20022 `DEBT`/`SHAR`/`CRED`. Define edge cases per product.
---
Payment rail mapping
ISO 20022 adoption across major rails
Rail	Region	Message family	Real-time?	Status
SWIFT CBPR+	Global	pacs.008/009, camt	No	Live, coexistence until Nov 2025
SEPA SCT Inst	Europe	pacs.008, pacs.002	Yes (≤10s)	Mandatory Jan 2025
SEPA SCT	Europe	pacs.008	No (D+1)	Live
SEPA SDD	Europe	pacs.003	No	Live
Fedwire	US	pacs.009	No (RTGS)	Live Mar 2025
CHIPS	US	pacs.009	No (netting)	Live
CHAPS	UK	pacs.009	No (RTGS)	Live Apr 2023
UPI	India	Proprietary (NPCI)	Yes (instant)	Live — ISO 20022 alignment planned
IMPS	India	Proprietary (NPCI)	Yes (24/7)	Live
Buna	Arab region	pacs.008	Yes	Live
UAEFTS	UAE	pacs	No (RTGS)	Live
UPI — India's real-time rail
UPI (Unified Payments Interface) is the NPCI-governed instant payment rail processing 10+ billion transactions/month. While UPI uses a proprietary message format (not yet ISO 20022), product teams building India-facing platforms must understand the interplay:
UPI uses VPA (Virtual Payment Address) — handle@bank — rather than IBAN/account number
Credit on UPI — extends UPI for credit disbursement (personal loans, BNPL)
UPI Lite — offline, low-value payments using on-device wallet
Cross-border UPI — expanding to Singapore (PayNow), UAE, UK. These corridors use ISO 20022 on the international leg with UPI on the domestic leg. Translation at the gateway is a product design challenge.
---
Compliance-by-design — embedding AML into ISO 20022
One of the most underused capabilities of ISO 20022 is its ability to carry compliance data natively in the message — making sanctions screening, AML monitoring, and Travel Rule compliance faster and more accurate.
FATF Travel Rule fields in pacs.008
```xml
<RgltryRptg>
  <Dtls>
    <Tp>CRED</Tp>
    <Inf>Beneficiary full name and address for Travel Rule compliance</Inf>
  </Dtls>
</RgltryRptg>
```
Sanctions screening on structured data
Legacy MT-based screening operates on free-text fields — prone to false positives from name formatting variations. ISO 20022 structured fields (`Dbtr/Nm`, `DbtrAgt/FinInstnId/BICFI`, `Cdtr/Nm`) enable:
Deterministic name matching — names are in dedicated fields, not buried in address line 2
BIC-based entity resolution — screen against the SWIFT BIC directory for counterparty entity matching
Lower false positive rates — structured data reduces noise; calibrate screening engines separately for MX vs. MT inputs
GenAI integration — SAR narrative automation
ISO 20022 structured remittance data (invoice references, purpose codes, counterparty identifiers) provides the clean, structured inputs that LLMs need to automate SAR narrative generation — replacing hours of analyst time with near-instant draft narratives for human review.
---
Product ownership — epics and user stories
Epic: ISO 20022 message processing
```
Epic: Implement ISO 20022 pacs.008 inbound processing
Goal: Process inbound SCT Inst pacs.008 messages within 10-second SLA

Features:
  F1: Schema validation and parsing
  F2: Sanctions screening on structured fields
  F3: Duplicate detection (UETR-based)
  F4: Account posting and notification
  F5: pacs.002 status response generation
  F6: Exception queue and repair workflow
```
Sample user story — end-to-end ID preservation
```
As a payment operations manager,
I want the End-to-End ID (EndToEndId) from the original pacs.008
to be preserved unchanged through every system in the payment chain,
So that I can reconcile customer queries to a specific transaction
without manual investigation.

Acceptance Criteria:
  GIVEN a pacs.008 message with EndToEndId = "CUST-REF-20240115-001"
  WHEN the payment is processed through the payment hub
  THEN the EndToEndId in the outbound pacs.008 equals "CUST-REF-20240115-001"
  AND the EndToEndId in the pacs.002 status response equals "CUST-REF-20240115-001"
  AND the EndToEndId is stored in the transaction record for audit

  GIVEN a downstream system attempts to modify the EndToEndId
  WHEN the modified message reaches the hub
  THEN the payment is rejected with reason code "AM05" (Duplicate)
  AND an alert is raised to the operations team
```
Sample user story — sanctions screening SLA
```
As a compliance officer,
I want sanctions screening to complete within 500ms for automated matches,
So that the 10-second SCT Inst SLA is met without degrading compliance controls.

Acceptance Criteria:
  GIVEN an inbound pacs.008 with no sanctions matches
  WHEN the sanctions screening service processes the message
  THEN a PASS result is returned within 500ms (P95)
  AND the payment proceeds to account posting

  GIVEN an inbound pacs.008 with a potential sanctions match
  WHEN the screening service identifies a hit
  THEN the payment is placed in a compliance hold queue within 500ms
  AND a pacs.002 with status "PDNG" is returned to the sending PSP
  AND a compliance analyst is notified within 60 seconds
```
---
Glossary
Term	Definition
CBPR+	Cross-Border Payments and Reporting Plus — SWIFT's ISO 20022 migration programme for cross-border payments
EPC	European Payments Council — governs SEPA payment schemes and rulebooks
UETR	Unique End-to-end Transaction Reference — UUID4 that tracks a payment across the entire correspondent banking chain
SCT Inst	SEPA Instant Credit Transfer — real-time payment settled in ≤10 seconds across SEPA
pacs	Payment Clearing and Settlement — ISO 20022 message namespace for interbank payments
pain	Payment Initiation — ISO 20022 namespace for customer-to-bank payment messages
camt	Cash Management — ISO 20022 namespace for account reporting and statement messages
STP	Straight-Through Processing — automated payment processing without manual intervention
HVPS+	High Value Payments Plus — SWIFT working group for high-value domestic system migration to ISO 20022
Travel Rule	FATF Recommendation 16 — requires originator and beneficiary information to travel with cross-border payments
VPA	Virtual Payment Address — UPI identifier in the format handle@bank (e.g. sanjay@hdfc)
gpi	SWIFT Global Payments Innovation — tracking and transparency layer built on UETR
BIC	Bank Identifier Code — 8 or 11 character code identifying a financial institution (also SWIFT code)
IBAN	International Bank Account Number — standardised account number format used across SEPA and many other regions
---
Resources
ISO 20022 Message Catalogue — official message definitions
SWIFT CBPR+ Migration Hub — SWIFT's migration resources and documentation
EPC SCT Inst Rulebook — EPC rulebook for SEPA Instant
Bank of England ISO 20022 Programme — CHAPS migration details
Federal Reserve ISO 20022 — Fedwire migration details
NPCI UPI Documentation — UPI product specifications
---
About the author
Sam — Sr. Product Owner with 20+ years in Payments, Core Banking, and Financial Crime across Tier-1 global banks, RegTech vendors, and FinTech platforms.
Delivered ISO 20022 and payment modernisation programmes at Standard Chartered (8+ markets), Capgemini (European bank SEPA SCT Inst migration), Wolters Kluwer (AML/KYC across Europe and APAC), and Maybank Innovation Lab (Open Banking, Southeast Asia).
💼 LinkedIn
📺 YouTube — Risk Finance Regulation 360
🗂️ Full GitHub portfolio
---
Content is based on practitioner experience and publicly available regulatory documentation. Always refer to the latest EPC rulebook, SWIFT standards, and your local regulatory authority for implementation decisions.
