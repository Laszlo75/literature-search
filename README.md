# Clinical Literature Search

A Claude skill that searches PubMed, Scholar Gateway, and national guideline websites for clinical evidence on a given topic, producing a structured evidence summary document and verified reference list.

## What It Does

Provide a clinical topic and the skill will:

1. **Parse and plan** — derive MeSH terms, identify relevant guideline bodies
2. **Search national guidelines** (BTS, NICE, KDIGO, BSH, SIGN, etc.) via web search
3. **Search published evidence** using both PubMed (keyword/MeSH) and Scholar Gateway (semantic search)
4. **Verify references** — DOI spot-check, title-PMID consistency, no fabricated identifiers
5. **Generate an evidence summary** (.docx) with guidelines, recent evidence, and evidence gaps
6. **Export references** as YAML ledger, BibTeX (.bib), and PMID list (.txt) for Zotero import

## Output Files

Each search produces five files:

| File | Purpose |
|------|---------|
| `*_Evidence_Summary_*.md` | Markdown source for the evidence summary |
| `*_Evidence_Summary_*.docx` | Formatted Word document (via pandoc) |
| `*_References.yaml` | Structured reference ledger (for protocol-reviewer) |
| `*_References.bib` | BibTeX file for Zotero/reference manager import |
| `*_PMIDs.txt` | PMID list for Zotero bulk import |

## Requirements

- [Claude Desktop](https://claude.ai/download) or another Claude client with MCP connector support
- **Recommended model**: Claude Opus 4.6 — the literature search, evidence appraisal, and reference integrity steps benefit from stronger reasoning capability
- The following MCP connectors enabled:
  - **PubMed** — literature search and article metadata
  - **Scholar Gateway** — semantic search across peer-reviewed literature
- **pandoc** — for markdown to .docx conversion
- The skill is designed for **UK NHS context** (references MHRA, NICE TAs, UK registries)

### Optional MCP connectors (enhance coverage)

- **bioRxiv** — searches medRxiv/bioRxiv for preprints on rapidly evolving topics
- **Clinical Trials** — searches ClinicalTrials.gov for ongoing/completed trials to enrich the Evidence Gaps section

## Installation

### Option 1: Install from repository

1. Clone this repository:
   ```bash
   git clone https://github.com/Laszlo75/literature-search.git
   ```
2. In Claude Desktop, open **Settings → Skills** and add the path to the `literature-search` folder

### After installation

1. **Enable the required MCP connectors** in Claude Desktop (Settings → Connectors):
   - **PubMed** — for literature search, article metadata, and full text retrieval
   - **Scholar Gateway** — for semantic search across peer-reviewed literature
2. **Install pandoc**: `brew install pandoc` (macOS) or `sudo apt install pandoc` (Linux)

## Example Prompts

- *"Search for evidence on CMV prophylaxis in solid organ transplant recipients"*
- *"What does the latest literature say about perioperative anticoagulation in DOAC patients?"*
- *"Literature search on rituximab dosing for ABO-incompatible kidney transplantation"*

## Integration with protocol-reviewer

The [protocol-reviewer](https://github.com/Laszlo75/protocol-reviewer) skill can consume this skill's YAML reference ledger for protocol reviews. You can either:

- **Run literature-search first**, then provide the YAML to protocol-reviewer
- **Let protocol-reviewer auto-trigger** a literature search — just upload a protocol and ask for a review

## Related Skills

- [protocol-reviewer](https://github.com/Laszlo75/protocol-reviewer) — reviews clinical protocols against evidence (consumes this skill's YAML ledger)

## AI Use & Governance (ISO 42001)

This tool uses AI-assisted evidence synthesis. AI outputs are advisory only and must be critically appraised by a clinician. The AI system is Claude (Anthropic), accessed via Claude Desktop with PubMed and Scholar Gateway integrations.

Every evidence summary is generated as a draft with a "DRAFT — NOT FOR CLINICAL USE" callout and transparency disclaimer. References are retrieved programmatically from PubMed and verified against source metadata.

See [`CLAUDE.md`](CLAUDE.md) for the full AI use policy.

## Trigger Phrases

The skill activates on phrases like: "literature search", "search PubMed", "find evidence on", "what does the latest evidence say about", "evidence review", "reference list for", or any request for clinical evidence on a specific topic.
