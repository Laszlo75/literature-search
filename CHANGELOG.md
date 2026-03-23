# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.4.0] - 2026-03-23

### Changed

- **Step 3 record-keeping: immediate write-to-file** — metadata from `get_article_metadata` must now be written to `_working_ledger.yaml` immediately after each call, in the same turn, not batched. This prevents context drift from corrupting DOIs and author names.
- **Step 4 verification: verify ALL references, not spot-check** — previously said "spot-check at least 5"; now requires re-fetching metadata for every PubMed reference and comparing DOI, title, and first author character-by-character. PubMed is authoritative; any mismatch overwrites the ledger.
- **Step 5: re-read ledger from disk** — before writing the reference list, the model must re-read `_working_ledger.yaml` from file, not from context memory.
- **YAML schema: expanded author fields** — `authors` replaced with `first_author` (surname + initials) and `authors_full` (complete author list), both copied verbatim from PubMed.
- Added real failure case as concrete warning in record-keeping section (DOI differed by 3 characters, wrong first author — caught only by re-verification against PubMed).

## [1.3.0] - 2026-03-22

### Added

- **bioRxiv/medRxiv integration** (Step 3b) — searches for preprints on medRxiv/bioRxiv when the bioRxiv MCP is available; results go in a separate "Emerging Evidence" section clearly labeled as non-peer-reviewed
- **ClinicalTrials.gov integration** (Step 3b) — searches for ongoing/completed trials when the Clinical Trials MCP is available; results appear in the Evidence Gaps section with NCT numbers, phase, and expected completion dates
- **Incremental ledger writing** — reference ledger is now written to `_working_ledger.yaml` after each batch of searches to protect against context loss during long searches
- **Retraction checking** — Step 4 verification now checks for retracted papers in PubMed and excludes them
- **Scholar Gateway DOI verification** — Scholar Gateway-only references now get cross-checked against PubMed in Step 4; removed the claim that Scholar Gateway DOIs are "reliable"
- **Scope guardrails** — Step 1 now includes a scope check; topics that are too broad or too narrow get flagged before searching begins
- **Conflicting Recommendations section** — new template section (Section 4) for cases where guidelines disagree or recent evidence contradicts guidelines
- **BibTeX key collision handling** — uses `AuthorYeara`, `AuthorYearb` convention when multiple papers share the same key
- New eval: CAR-T therapy (tests bioRxiv and ClinicalTrials.gov integration)
- New eval: overly broad topic (tests scope guardrail)

### Changed

- `find_related_articles` guidance: now recommends using on 2-3 landmark papers only, not every paper
- YAML schema: added `preprints` and `ongoing_trials` sections (optional)
- Evidence summary template: renumbered sections (4=Conflicting Recommendations, 5=Emerging Evidence, 6=Evidence Gaps, 7=Transparency Disclaimer, 8=References)
- Version string no longer needs dynamic propagation — hardcoded in YAML schema, removed from transparency disclaimer
- Model requirements section renamed to "How to Approach This Workflow" — model recommendation moved to README

### Fixed

- Scholar Gateway-only references previously had no DOI verification (Step 4 was PubMed-centric)
- Version string fragility — model no longer needs to read frontmatter version and propagate to two locations

## [1.2.0] - 2026-03-22

### Changed

- Removed protocol-review mode — skill now has a single output mode (standalone evidence summary)
- Protocol-reviewer skill now auto-triggers literature-search when no YAML is provided, removing the need for a separate mode
- Removed `protocol_section` field from YAML ledger schema (protocol-reviewer maps references to sections itself)
- Removed protocol-review mode template from `evidence_summary_template.md`

## [1.1.0] - 2026-03-22

### Changed

- Protocol-review mode now detected from natural language (mentions of "protocol", "protocol review") instead of `--protocol` flag
- Removed duplicated document structure from SKILL.md — now references `evidence_summary_template.md` as single source of truth
- Added Quick Start section to SKILL.md
- Added error handling guidance for missing tools (pandoc, PubMed MCP, Scholar Gateway)
- Added `.gitignore` for generated output files
- Added `.claude/settings.local.json` with pandoc permissions

## [1.0.0] - 2026-03-22

### Added

- 6-step clinical literature search workflow (SKILL.md)
- PubMed MCP and Scholar Gateway dual search strategy
- Evidence summary document generation (.md → .docx via pandoc) with two modes:
  - Standalone mode: full document with title page for general use
  - Protocol-review mode (`--protocol`): reformatted for protocol-reviewer skill
- YAML reference ledger output (`_References.yaml`) as contract for protocol-reviewer
- BibTeX (.bib) and PMID list (.txt) export for Zotero import
- PubMed search strategy guide (references/pubmed_strategy.md)
- Evidence summary template (references/evidence_summary_template.md)
- Reference integrity verification step (DOI spot-check, title-PMID consistency)
- Draft callout and transparency disclaimer in every evidence summary
- Evaluation scenarios for CMV prophylaxis, perioperative anticoagulation, and ABOi transplant

### Notes

- Split from [protocol-reviewer](https://github.com/Laszlo75/protocol-reviewer) v1.4.0
- The literature search steps (formerly Steps 2-4, 7 of protocol-reviewer) are now a standalone skill
- Evidence summary document is a new output not present in the original protocol-reviewer
