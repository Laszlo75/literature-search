# Evidence Summary Template — Markdown-First Approach

## Strategy

Write the evidence summary as a **Markdown file** first, then convert to .docx using
pandoc with a styled reference template. This keeps the agent focused on clinical content
quality rather than formatting.

## Why Markdown First?

- The agent writes natural structured text instead of JavaScript
- Clickable hyperlinks work natively: `[link text](https://...)`
- In-text citations are just `[1]` in plain text
- Tables work in standard markdown
- Pandoc produces clean .docx that opens reliably in Microsoft Word
- The reference template handles fonts, colours, heading styles, and page layout
- Content is easy to review and debug before conversion

## Step-by-Step Process

### 1. Write the evidence summary as Markdown

Create a file called `[Topic_Name]_Evidence_Summary_[Year].md` with this structure:

```markdown
---
title: "Evidence Summary: [Topic]"
subtitle: "Systematic Literature Search"
author: "Clinical Literature Search — AI-Assisted Analysis"
date: "[Month Year]"
---

\newpage

> **DRAFT — NOT FOR CLINICAL USE**
>
> This is an AI-generated draft document. It has not been verified by a clinician
> and must not be used for clinical decision-making until it has been reviewed,
> cross-checked, and approved by a qualified professional.
>
> **To approve this document:** (1) review all evidence summaries and references,
> (2) delete this callout, (3) complete the "Reviewed and approved by" field in
> the Transparency Disclaimer (section 5).

# 1. Search Strategy

**Databases searched:** PubMed, Scholar Gateway
**Date range:** [start] to [end]
**MeSH terms:** [list]
**Publication types prioritised:** Systematic reviews, meta-analyses, RCTs, large registry studies
**Guideline bodies consulted:** [list with editions/dates]
**Total references included:** [N]

# 2. National Guidelines

## 2.1 [Guideline Body — e.g., BTS Guidelines for X, Nth Edition (Year)]

[Key recommendations with evidence grades. ALWAYS include the grade — e.g.,
"BTS recommends target titre <1:8 at transplantation (Grade 1C)"]

## 2.2 [Next Guideline Body]

[...]

# 3. Recent Evidence

## 3.1 [Subtopic — e.g., Drug Dosing]

[Papers grouped by subtopic. Summarise key findings with in-text citations:
"A recent meta-analysis [3] found no difference in outcomes. A UK registry
study confirmed this [7]."]

## 3.2 [Next Subtopic]

[...]

# 4. Conflicting Recommendations

[Include this section only if guidelines disagree with each other or if recent
evidence contradicts a current guideline. Present both positions with evidence
grades. Example:

"BTS recommends X (Grade 1C) while KDIGO suggests Y (Grade 2D). The discrepancy
reflects differing interpretations of [study]. A recent RCT [N] found Z, which
supports the BTS position but was published after the current KDIGO guideline."

Omit this section entirely if there are no conflicts to report.]

# 5. Emerging Evidence (Preprints)

[Include this section only if medRxiv/bioRxiv preprints were found in Step 3b.
Label each preprint clearly as "(preprint, not peer-reviewed)". Example:

"A recent medRxiv preprint (Smith et al., 2026; preprint, not peer-reviewed)
reports outcomes from a single-centre series of 45 patients..."

Omit this section entirely if no preprints were found or the bioRxiv MCP
was not available.]

# 6. Evidence Gaps

[Areas where evidence is limited or absent. Identify:
- Questions with no RCT data
- Topics covered only by single-centre studies
- Emerging areas with only preliminary data
- If ongoing trials were found (Step 3b), list them here with NCT numbers,
  phase, sample size, and expected completion dates]

# 7. Transparency Disclaimer

*This evidence summary was produced using AI-assisted literature search (Claude, Anthropic)
with PubMed and Scholar Gateway searches. The AI system was used for literature retrieval
and structured analysis; clinical interpretation remains the responsibility of the
reviewing clinician. References have been verified against PubMed metadata.*

**Reviewed and approved by:** ______________________ *(name, title, and institution)*

*AI system metadata: literature-search · [model identifier] · PubMed MCP · Scholar Gateway · Search date [YYYY-MM-DD] · [github.com/Laszlo75/literature-search](https://github.com/Laszlo75/literature-search)*

# 8. References

1. Author AB, Author CD, et al. Title of the article. *Journal Name*.
   Year;Vol(Issue):Pages. PMID: 12345678.
   [DOI](https://doi.org/10.1234/example)

2. British Transplantation Society. Guidelines for Antibody Incompatible
   Transplantation, Third Edition. February 2016.
   [BTS Guidelines](https://bts.org.uk/guidelines-standards/)

3. [Continue numbering sequentially...]
```

### 2. Convert to .docx using pandoc

The skill bundles a `reference.docx` template (in the `assets/` directory) that sets
Arial font, A4 page size, navy headings, headers/footers, and page numbers.

```bash
pandoc "[Topic_Name]_Evidence_Summary_[Year].md" \
  -o "[Topic_Name]_Evidence_Summary_[Year].docx" \
  --reference-doc="[skill-path]/assets/reference.docx" \
  --from=markdown+yaml_metadata_block \
  --to=docx
```

The YAML frontmatter (`title`, `subtitle`, `author`, `date`) is rendered as a title
block on the first page by pandoc. The `\newpage` after the frontmatter forces the
search strategy section onto page 2, creating a clean title page.

### 3. Also produce the .bib and PMIDs.txt files

After the markdown and docx are done, generate:

**`[Topic_Name]_References.bib`** — BibTeX file:
```bibtex
@article{AuthorYear,
  author  = {Last, First and Last2, First2},
  title   = {Article title},
  journal = {Journal Name},
  year    = {2024},
  volume  = {36},
  pages   = {100--110},
  pmid    = {12345678},
  doi     = {10.1234/example}
}
```

If two references share the same `AuthorYear` key (e.g., two Smith2024 papers),
append a lowercase letter: `Smith2024a`, `Smith2024b`.

**`[Topic_Name]_PMIDs.txt`** — one PMID per line:
```
12345678
23456789
34567890
```

## Key Content Rules

### Evidence grades — do not skip

Every recommendation backed by a national guideline MUST include the evidence grade
inline. This is what clinicians look for first. Examples:

- "BTS recommends target titre <1:8 at transplantation **(Grade 1C)**"
- "NICE recommends DOACs over warfarin for non-valvular AF **(Strength: Strong)**"
- "KDIGO suggests monitoring DSA quarterly **(Grade 2C)**"

### In-text citations — every claim needs a source

Use numbered references in square brackets: `[1]`, `[2, 3]`, `[4-6]`.
Every reference in the list must be cited at least once in the body.
Every factual claim in the summary must cite its source.

### Clickable links in references — essential

In the markdown, write each reference with its DOI or URL as a clickable link:

```markdown
1. Smith AB, et al. Title. *Journal*. 2024;36:100. PMID: 12345678.
   [DOI](https://doi.org/10.1234/example)
```

Pandoc converts these to clickable hyperlinks in the .docx automatically.

### Reference accuracy — copy DOIs from PubMed, never from memory

Every DOI in the reference list MUST come directly from the `get_article_metadata`
output for that PMID. DOIs are opaque strings — you cannot reconstruct them from the
paper title or journal. If you write a DOI from memory, it will almost certainly point
to the wrong paper or be a dead link.

When writing the reference list, work from the reference ledger you built during the
search phase. Copy DOIs, titles, and author lists exactly as PubMed returned them.

### Guidelines in the reference list

National guidelines (BTS, NICE, KDIGO, etc.) must appear as numbered entries
in the reference list, just like PubMed papers. Use the guideline body's URL:

```markdown
15. British Transplantation Society. Guidelines for Antibody Incompatible
    Transplantation, 3rd Edition. 2016.
    [BTS Guidelines](https://bts.org.uk/guidelines-standards/)
```

## Output Files

The skill should produce five files in the user's workspace folder:

1. **`[Topic_Name]_Evidence_Summary_[Year].md`** — the markdown source
2. **`[Topic_Name]_Evidence_Summary_[Year].docx`** — the converted Word document
3. **`[Topic_Name]_References.yaml`** — structured reference ledger (for protocol-reviewer)
4. **`[Topic_Name]_References.bib`** — BibTeX for Zotero import
5. **`[Topic_Name]_PMIDs.txt`** — PMID list for bulk Zotero import
