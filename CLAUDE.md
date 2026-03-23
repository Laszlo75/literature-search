# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code skill (custom prompt) that performs systematic clinical literature searches using PubMed and Scholar Gateway. Given a clinical topic, it searches for current national guidelines and published evidence, then produces a structured evidence summary document (.docx), a YAML reference ledger, and Zotero-compatible reference exports.

This skill is designed to be used standalone (for journal clubs, grant applications, teaching, clinical questions) or as the first step in a two-skill workflow with the [protocol-reviewer](https://github.com/Laszlo75/protocol-reviewer) skill.

## Repository Structure

- **SKILL.md** — The skill definition (prompt). Defines the 6-step workflow: parse topic → search guidelines → search evidence → verify references → generate summary → output files.
- **assets/reference.docx** — Pandoc reference template for .docx output (Arial, A4, navy headings, headers/footers).
- **references/evidence_summary_template.md** — Markdown template and pandoc conversion instructions for the evidence summary document.
- **references/pubmed_strategy.md** — PubMed search construction patterns and quality assessment heuristics.
- **evals/evals.json** — Test scenarios for the skill.

## Key Design Decisions

- **Markdown-first approach**: The evidence summary is written as Markdown with YAML frontmatter, then converted to .docx via pandoc.
- **Reference ledger**: During search, references are recorded in a structured ledger (PMID, DOI, authors, title copied verbatim from PubMed). The .bib and reference list are generated solely from this ledger to prevent DOI/title fabrication.
- **Dual search strategy**: Both PubMed MCP (keyword/MeSH) and Scholar Gateway (semantic search) are used for each topic — they return complementary results.
- **YAML contract**: The `_References.yaml` output file serves as the interface for the protocol-reviewer skill. It contains all metadata needed for cross-referencing.
- **Single output mode**: Produces a full evidence summary with title page. The YAML reference ledger can be consumed by the [protocol-reviewer](https://github.com/Laszlo75/protocol-reviewer) skill, which can also auto-trigger this skill when no YAML is provided.

## Build / Conversion Command

```bash
pandoc "[Topic_Name]_Evidence_Summary_[Year].md" \
  -o "[Topic_Name]_Evidence_Summary_[Year].docx" \
  --reference-doc=assets/reference.docx \
  --from=markdown+yaml_metadata_block \
  --to=docx
```

## Output Files (per search)

1. `[Topic_Name]_Evidence_Summary_[Year].md` — Markdown source
2. `[Topic_Name]_Evidence_Summary_[Year].docx` — Word document
3. `[Topic_Name]_References.yaml` — Structured reference ledger
4. `[Topic_Name]_References.bib` — BibTeX for Zotero
5. `[Topic_Name]_PMIDs.txt` — PMID list for Zotero bulk import

## Tool Dependencies

Required:
- **PubMed MCP** (`search_articles`, `get_article_metadata`, `get_full_text_article`, `find_related_articles`)
- **Scholar Gateway** (`semanticSearch`)
- **WebSearch / WebFetch** — for national guidelines
- **pandoc** — markdown to .docx conversion

Optional (enhance coverage when available):
- **bioRxiv MCP** (`search_preprints`, `search_published_preprints`) — preprint search for emerging evidence
- **Clinical Trials MCP** (`search_trials`, `get_trial_details`, `analyze_endpoints`) — ongoing trial data

## AI Use Policy (ISO 42001)

**System identity:** Claude Opus 4.6 (Anthropic), accessed via Claude Desktop with PubMed and Scholar Gateway integrations.

**Intended use:** AI-assisted literature search and evidence synthesis. The system retrieves, structures, and summarises evidence; it does not make clinical decisions.

**Human oversight:** All AI-generated outputs are advisory only. Every evidence summary must be critically appraised by a clinician before informing clinical decisions.

**Transparency:** Each evidence summary is generated as an explicit draft with a "DRAFT — NOT FOR CLINICAL USE" callout and a transparency disclaimer disclosing AI involvement.

**Reference integrity:** DOIs and article metadata are retrieved programmatically from PubMed and verified against source records. The system never fabricates identifiers.

## Clinical Content Rules

- Always include evidence grades inline (e.g., "BTS Grade 1C", "NICE Strength: Strong")
- Frame findings in UK NHS context (MHRA, NICE TAs, UK registries)
- Numbered in-text citations in square brackets: `[1]`, `[2, 3]`
- DOIs must be copied verbatim from `get_article_metadata` directly to `_working_ledger.yaml` — never from memory
- Step 4 verification re-fetches metadata for ALL references and compares character-by-character — not a spot-check
- Target 15-30 high-quality references per search
