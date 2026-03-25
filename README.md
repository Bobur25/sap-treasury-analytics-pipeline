# SAP ECC Treasury Data Pipeline

End-to-end analytics pipeline for a treasury department: raw SAP data → Python cleaning → AI analysis (with data anonymization) → Power BI dashboard.

**Tech Stack:** Python · Pandas · Anthropic Claude API · Power BI

```
SAP ECC Export (10K rows) → Python Cleaning → Encoding → Claude AI Analysis → Decoding → Power BI Dashboard
```

---

## What It Does

1. **Generates** a realistic 10K-row SAP ECC treasury dataset for a DACH-region IT distributor (hardware, software, services) — with intentional dirty data (duplicates, mixed date formats, missing values, logical errors)
2. **Cleans** the data with a 14-step Pandas pipeline — deduplication, date standardization, currency fixes, business rule validation
3. **Encodes** all corporate identifiers (company names, bank accounts, vendor IDs) into pseudonyms for data governance compliance
4. **Sends** anonymized data to Claude API for treasury-specific analysis — liquidity risk, FX exposure, payment efficiency, anomaly detection
5. **Decodes** the AI response back to real entity names
6. **Visualizes** everything in a 4-page Power BI dashboard

---

## Project Files

| File | Purpose |
|------|---------|
| `sap_treasury_data_generator.ipynb` | Phase 1: Generate realistic SAP dataset |
| `sap_treasury_data_cleaning.ipynb` | Phase 2: Clean and validate (14 transformations) |
| `sap_treasury_encoding_analysis.ipynb` | Phase 3: Encode → AI analysis → Decode |
| `sap_ecc_treasury_export_raw.csv` | Raw dataset with data quality issues |
| `sap_ecc_treasury_clean.csv` | Cleaned dataset |
| `ai_treasury_insights.csv` | Structured AI analysis output |
| `ai_analysis_decoded.txt` | Full decoded AI analysis report |
| `compliance_log.txt` | Data anonymization audit trail |
| `SAP_Treasury_Dashboard.pbix` | Power BI dashboard (interactive) |
| `SAP_Treasury_Dashboard.pdf` | Dashboard export (static view) |

---

## How to Run

```bash
pip install pandas numpy openpyxl anthropic
```

Run notebooks in order (1 → 2 → 3). Set `ANTHROPIC_API_KEY` environment variable before Phase 3. Open `.pbix` in Power BI Desktop for the dashboard.

---

## SAP Field Reference

<details>
<summary><b>Click to expand — SAP field names and what they mean</b></summary>

### Identifiers
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| MANDT | Mandant | SAP system/client ID |
| BUKRS | Buchungskreis | Company code (legal entity) |
| GJAHR | Geschäftsjahr | Fiscal year |
| BELNR | Belegnummer | Document number (transaction ID) |
| BUZEI | Belegzeile | Line item number within a document |

### Dates
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| BUDAT | Buchungsdatum | Posting date (when recorded in books) |
| BLDAT | Belegdatum | Document date (date on the invoice/contract) |
| VALUT | Valutadatum | Value date (when bank actually moves money) |
| CPUDT | CPU-Datum | Entry date (when entered into SAP) |
| AEDAT | Änderungsdatum | Last change date |

### Transaction Types
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| BLART | Belegart | Document type: KZ=Vendor payment, DZ=Customer payment, SA=G/L posting, ZP=Payment run, AB=Clearing |
| BSCHL | Buchungsschlüssel | Posting key (debit/credit indicator) |
| HKONT | Hauptbuchkonto | G/L account number |

### Amounts
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| WRBTR | Währungsbetrag | Amount in transaction currency |
| DMBTR | Betrag in Hauswährung | Amount converted to local currency (EUR) |
| WAERS | Währungsschlüssel | Currency code (USD, EUR, CHF, etc.) |
| KURSF | Kurs | Exchange rate used for conversion |

### Counterparties
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| LIFNR | Lieferantennummer | Vendor ID |
| KUNNR | Kundennummer | Customer ID |
| NAME1 | Name 1 | Counterparty name |

### Banking
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| HBKID | Hausbankkennung | House bank ID (which of the company's banks) |
| HKTID | Hauskontokennung | Bank account ID at that bank |
| BANKN | Bankkontonummer | External IBAN/account number |
| RZAWE | Zahlweg | Payment method: T=SEPA, W=SWIFT Wire, E=ACH, C=Check |

### Organizational
| Field | German Full Name | Meaning |
|-------|-----------------|---------|
| PRCTR | Profit Center | Business division (Hardware, Software, Services) |
| KOSTL | Kostenstelle | Cost center |
| SGTXT | Segmenttext | Free-text transaction description |
| ZUESSION | Zuordnung | Assignment (PO or invoice reference) |
| ERNAM | Erfasser Name | User who created the entry |

</details>

<details>
<summary><b>Click to expand — Treasury concepts explained</b></summary>

- **Cash Position:** Total money across all bank accounts at a given moment
- **FX Exposure:** Currency risk from buying in USD but earning in EUR
- **FX Spot:** Buying/selling currency immediately at today's rate
- **FX Forward:** Locking in an exchange rate for a future date to hedge risk
- **Hedging Ratio:** % of FX exposure covered by forward deals
- **SEPA vs SWIFT:** SEPA = cheap EUR transfers within Europe (~€0.20). SWIFT = expensive international wires (~€15-50)
- **Payment Run (F110):** SAP automated batch payment process
- **Intercompany Transfer:** Moving money between legal entities in the same group
- **Value Date vs Posting Date:** Posting = when booked. Value = when bank moves the cash
- **G/L Account:** Categorized ledger account (11xxxx = bank, 14xxxx = receivables, 16xxxx = payables, 48xxxx = FX gains/losses)

</details>

---

## SAP Tables Simulated

| Table | What It Stores |
|-------|---------------|
| BKPF | Transaction headers (who, when, what type) |
| BSEG | Transaction line items (amounts, accounts, currencies) |
| FEBEP | Bank statement lines |
| REGUH | Automatic payment run records |
| TCURR | Exchange rate table |
| T012K | Company bank account master data |
| VTBFHA | Treasury deal headers (FX, deposits) |
| VTBFHAPO | Treasury deal cash flows |

---

## Key Design Decisions

- **Lognormal amount distribution** — real financial data has many small and few large transactions, not uniform random
- **Weighted probabilities** — document types, payment methods, currencies all follow realistic frequency patterns
- **Deterministic encoding** — same entity always maps to same pseudonym, preserving analytical relationships
- **Amount scaling** — amounts multiplied by secret factor to hide real scale while preserving ratios and trends
