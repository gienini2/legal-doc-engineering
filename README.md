# legal-doc-engineering

> Prompt engineering system for generating legally-structured official police reports in Catalan, with an AI-assisted normative knowledge base built from official traffic regulations.

---

## What this is

This project has two independent components that work together:

**1. Normative knowledge base** — A Python script that parses the official Catalan traffic regulation documents (published by the *Servei Català de Trànsit*) from DOCX format into a structured JSON master file of 550+ infractions, each with its legal code, article reference, severity level, fine amount, and licence point deductions.

**2. Prompt engineering system** — A designed prompt architecture that feeds the structured normative context to an LLM (Claude) to generate official police reports: traffic accidents, road obstruction incidents, public space inspection reports, illegal waste dumping, and parked vehicle control records.

The output documents are written in **technical Catalan police register**, include **automatic legal citation** (article codes, infraction IDs, fine amounts), and are suitable for direct submission to municipal authorities.

---

## The pipeline

```
Official SCT regulation DOCX files
        ↓
Python parser (python-docx)
        ↓
infractions_master.json  ←── 550 infractions, 4 regulations
        ↓
Structured context fed to LLM
        ↓
Natural language incident description  (officer's notes, photos)
        ↓
Claude API + domain-specific prompt architecture
        ↓
Official police report (.pdf)
  · Legal sections and structure
  · Coded infraction table (e.g. RGC-0368, OGC Art. 56.2.d)
  · Chronological action log
  · Photographic evidence annex
  · Proposed sanctions with exact fine amounts
  · Digitally signed by TIP number
```

---

## Normative knowledge base

### Source documents parsed
| Code | Regulation | Decree | Infractions |
|------|-----------|--------|-------------|
| RGC | Reglament General de Circulació | RD 1428/2003 | 461 |
| RGV | Reglament General de Vehicles | RD 2822/1998 | 45 |
| RGCOND | Reglament General de Conductors | RD 818/2009 | 17 |
| AOV | Assegurança Obligatòria de Vehicles | RDL 8/2004 | 8 |

**Total: 531 infractions** structured with:
- Unique ID (`RGC-0368`, `AOV-0003`...)
- Category and subcategory
- Full infraction text + clarifying notes
- Article reference
- Severity level: `L` (Lleu) / `G` (Greu) / `MG` (Molt Greu)
- Fine amount (€) and 50% early payment discount
- Licence points lost (null if none)
- Fast-lookup indexes by article, severity, regulation

### Parser technical notes
The DOCX files use merged cells and inconsistent column structures across regulations (AOV has 4 columns, others have 5-6). The parser handles this by reading the XML internal structure via `python-docx` rather than relying on visual column count, detecting section headers (single-column rows in uppercase), and splitting infraction text from clarifying notes using whitespace patterns.

---

## Prompt architecture

The system prompt encodes:
- Official Catalan municipal police report structure (sections, numbering, administrative register)
- Legal citation rules (which regulation takes precedence, how to reference articles)
- Catalan technical police register (not colloquial Catalan — formal administrative language)
- Infraction codification logic (severity escalation, cumulative infractions, subsidiary recovery of public costs)
- Evidence documentation standards (photographic annex structure, GPS coordinates, timestamps)

The user prompt provides:
- Officer's raw field notes
- Incident data (vehicles, persons, times, locations)
- Relevant infraction JSON context (pre-filtered from master)
- Local municipal ordinances as PDF context when applicable

---

## Sample outputs (anonymised)

Each report used a different pipeline depending on volume and complexity:

| Report | Type | Pipeline | Complexity |
|--------|------|---------|-----------|
| [S-03 — Traffic accident](demos/S-03_accident_anonymised.pdf) | Collision, 2 vehicles, 1 minor injury | Photos + field notes → Claude | Legal citation, accident diagram, services log |
| [S-04 — Parked vehicle control](demos/S-04_vehicle_control_anonymised.pdf) | 22 vehicles inspected, 49 photos | ChatGPT Vision (damage per photo) → Word → Claude (professional register) | Multi-model pipeline, systematic damage inventory |
| [S-05 — Illegal waste dumping](demos/S-05_illegal_waste_anonymised.pdf) | Public space inspection | Photos + inspection notes → Claude | Itemised inventory, public health risk assessment |
| [S-11 — Road obstruction](demos/S-11_road_obstruction_anonymised.pdf) | 8h20m road closure, large vehicle breakdown | Field notes + SCT infraction JSON + municipal ordinances PDF → Claude | 7 coded infractions across 3 normative sources, subsidiary cost recovery |

### S-04 preview

![S-04 Parked Vehicle Control Report](demos/S-04_preview.png)

The S-04 pipeline is the only one using multi-model architecture: GPT-4 Vision handled the visual damage assessment across 49 photos (where cost and volume made it the practical choice), the results were compiled into Word, then Claude refined the administrative register to official quality. Each model used for what it does best.

---

## What this demonstrates

- **Domain-driven prompt engineering** — the prompt architecture encodes 18 years of operational police knowledge, not generic legal formatting
- **Structured normative context** — LLMs perform significantly better with pre-structured JSON regulatory data than with raw PDF text
- **Legal reasoning via prompt** — infraction severity hierarchy, cumulative charging, subsidiary liability — encoded as prompt logic
- **Multilingual professional register** — output is formal Catalan administrative language, not a translation
- **AI-assisted implementation** — Python parser co-developed with AI assistance; system design, domain validation and prompt architecture by the author

---

## Stack

`Python` · `python-docx` · `Claude API (Anthropic)` · `JSON` · `Prompt engineering` · `Catalan legal domain`

---

## Author note

These reports were generated without using any of the other systems in this portfolio (Bitácola, Sherlock). They represent a standalone exploration of what's possible when domain expertise is encoded directly into a prompt architecture — before building any automation layer around it.

The S-11 report (road obstruction) includes 7 infractions correctly cross-referenced across three different normative sources (RGC, local traffic ordinance, civic ordinance), with exact fine amounts, severity escalation logic, and a legally-founded subsidiary cost recovery claim — generated from a field officer's notes and photos in a single session.
