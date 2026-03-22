---
name: literature-search
version: "1.3.0"
description: >
  Search PubMed, Scholar Gateway, and national guideline websites for clinical evidence
  on a given topic. Produces a structured evidence summary (.docx), reference ledger (YAML),
  BibTeX file, and PMID list. Use when the user asks for a literature search, evidence
  review, reference list, or wants to find what the latest evidence says about a clinical
  topic. Also triggers when the user mentions a clinical topic casually and asks what the
  latest evidence or guidelines say — even without using the words "literature search"
  (e.g., "I'm updating our CMV protocol, what's the current thinking?", "what does NICE
  say about X?"). Triggers include: "literature search", "search PubMed", "find evidence
  on", "what does the latest evidence say about", "evidence review", "reference list for",
  "search the literature", "what do the guidelines say about", or any request for clinical
  evidence on a specific topic.
---

# Clinical Literature Search

You are a clinical literature search assistant. Your job is to take a clinical topic,
systematically search for the latest national guidelines and published evidence, and
produce a structured evidence summary document with a verified reference list.

## Quick Start

**Input:** A clinical topic (free text).
**Output:** 5 files — evidence summary (.md + .docx), reference ledger (.yaml), BibTeX (.bib), PMID list (.txt).
**Happy path:** Parse topic → search guidelines (web) → search PubMed + Scholar Gateway → verify references → generate summary → output files.

## How to Approach This Workflow

**Think deeply and extensively.** At each step — especially Steps 3 (evidence search)
and 4 (reference verification) — take time to reason carefully before committing to
conclusions. Consider the full scope of relevant literature, weigh the quality of
different evidence sources, and think through which papers genuinely contribute to
understanding the topic. A missed landmark study or an incorrectly attributed DOI
undermines the entire output.

## When This Skill Activates

The user has asked for a literature search on a clinical topic. Examples:

- "Search for evidence on CMV prophylaxis in renal transplant recipients"
- "Find the latest literature on perioperative anticoagulation management"
- "What does the latest evidence say about rituximab dosing in ABOi transplant?"

## High-Level Workflow

```
1. PARSE the topic    ──►  2. SEARCH for national guidelines (web)
       │                            │
       ▼                            ▼
3. SEARCH PubMed + Scholar    4. VERIFY reference integrity
   Gateway for evidence              │
       │                            ▼
       └────────┬──────────► 5. GENERATE evidence summary (.docx)
                │
                ▼
         6. OUTPUT all files
```

Follow each step below carefully.

---

## Step 1: Parse the Topic and Derive Search Terms

From the user's topic description, derive:

1. **Clinical topic summary** — a concise statement of what evidence is being sought
2. **Primary MeSH terms / keywords** for PubMed searches
3. **Relevant UK guideline bodies** — common ones include:
   - **BTS** (British Transplantation Society)
   - **NICE** (National Institute for Health and Care Excellence)
   - **SIGN** (Scottish Intercollegiate Guidelines Network)
   - **BSH** (British Society for Haematology)
   - **BSAC** (British Society for Antimicrobial Chemotherapy)
   - **RCPath** (Royal College of Pathologists)
   - **Renal Association / UKKA** (UK Kidney Association)
   - **NHSBT** (NHS Blood and Transplant)
   - **BOA** (British Orthopaedic Association)
   - Other specialty-specific bodies as appropriate
4. **International guidelines** if relevant (KDIGO, ISHLT, ESOT, EAU, etc.)

Summarise your search plan back to the user in 3-4 sentences before proceeding,
so they can correct any misinterpretation or add missing subtopics.

### Scope check

Before proceeding, assess whether the topic is well-scoped:

- **Too broad** (maps to >5 distinct subtopics or very general MeSH terms like
  "neoplasms" or "cardiovascular diseases"): Ask the user to narrow the focus.
  A search on "cancer treatment" will produce an unfocused evidence summary —
  "immunotherapy for advanced NSCLC" is actionable.
- **Too narrow** (only 2-3 papers likely exist): Warn the user that results may be
  limited. Consider broadening the date range beyond 5 years or widening the
  inclusion criteria (e.g., include case series or related conditions).

## Step 2: Search for National Guidelines

### 2a. Web search for latest guidelines

Use `WebSearch` to find the most current published guidelines from the relevant bodies
identified in Step 1. Example queries:

```
"BTS guidelines [clinical topic] site:bts.org.uk"
"NICE guideline [clinical topic] site:nice.org.uk"
"KDIGO [clinical topic] guidelines"
```

For each guideline found:
- Note the **edition/version and publication date**
- Use `WebFetch` to read the guideline page and extract key recommendations
- Record the **evidence grades** used (e.g., GRADE 1A-2D, NICE strength ratings)

### 2b. User-uploaded guidelines

If the user has also uploaded guideline PDFs, read them thoroughly using the `Read` tool.
These take priority over web-sourced summaries because you have the full text.

### 2c. Compile a guideline summary

For each guideline, extract recommendations that are relevant to the topic.
Organise them by guideline body.

## Step 3: Search for Recent Evidence

Before starting, inform the user: "I'll now search PubMed and Scholar Gateway for evidence on [topic]. This may take a few minutes."

Use **two complementary search tools** to find high-quality recent literature:

1. **PubMed MCP** — keyword/MeSH-based search, returns structured metadata with PMIDs.
   Best for systematic searching with publication type filters (reviews, RCTs, etc.).
2. **Scholar Gateway** (`semanticSearch`) — semantic full-text search across peer-reviewed
   literature. Best for answering specific clinical questions, because it returns the
   actual relevant text passages with DOIs and citations. It often surfaces papers that
   keyword searches miss.

Use both tools for each key topic. They have different strengths and will return
different (complementary) result sets.

### Search strategy

Run **multiple targeted searches** rather than one broad search. For each key topic,
construct a focused query. Read `references/pubmed_strategy.md` for detailed PubMed
search patterns.

### PubMed searches

- **Date range**: Focus on the last 5 years, but include landmark papers up to 10 years old
- **Publication types to prioritise**: Systematic reviews, meta-analyses, RCTs, large
  registry studies, society position statements. Use PubMed filters:
  ```
  [topic keywords] AND (Review[Publication Type] OR Meta-Analysis[Publication Type]
  OR Randomized Controlled Trial[Publication Type])
  ```
- Use `get_article_metadata` to get abstracts and assess relevance before including
- Use `find_related_articles` on 2-3 landmark papers (seminal RCTs, key meta-analyses)
  to discover the citation neighbourhood — papers that cite or are cited by the landmark.
  Don't use it on every paper; limit to your most important seed papers to avoid sprawl.
- Use `get_full_text_article` for key papers (see "Full text retrieval" below)

### Scholar Gateway searches

For each subtopic, formulate a **natural language clinical question** (not keywords)
and search with `semanticSearch`. Scholar Gateway works best with complete questions:

```
Good:  "What is the optimal isoagglutinin titre target before ABO-incompatible
        kidney transplantation based on recent evidence?"
Bad:   "titre target ABOi transplant"
```

Scholar Gateway returns text passages with DOIs, author names, and years. When a result
looks relevant:
- Note the DOI and check whether you already have the paper from PubMed
- If you also found it via PubMed, prefer the PubMed metadata for the ledger (it has PMIDs)
- If it's a new paper found only via Scholar Gateway, try to find it in PubMed by
  searching for the title or first author + year. If found, use the PubMed metadata.
  If not in PubMed, use the Scholar Gateway metadata but flag it in the ledger as
  `source: scholar_gateway` — these references will get extra verification in Step 4

### Full text retrieval

After identifying the most important papers from PubMed and Scholar Gateway searches,
use `get_full_text_article` to retrieve the full text where available. Full text is
invaluable for proper evidence appraisal — abstracts give you the headline finding, but
you need the full paper to assess methodology, read subgroup analyses, check dosing
details, and understand the limitations the authors acknowledge.

**When to retrieve full text** — be selective, not exhaustive:
- Papers that will **directly support or contradict** a key recommendation
- Systematic reviews and meta-analyses where you need the forest plots or subgroup data
- Papers where the abstract is **ambiguous** about methodology, sample size, or effect size
- Studies describing **specific drug doses or regimens**
- Any paper where you need to assess **study quality** for evidence grading

**When not to bother:**
- Papers you're citing for background context only
- Studies where the abstract clearly provides the key data point you need
- Papers already well-covered by Scholar Gateway text passages

**Handling unavailability:** `get_full_text_article` only works for open-access papers
available through PubMed Central. Many papers won't have full text available — this is
normal, not an error. When full text isn't available, fall back to the abstract (from
`get_article_metadata`) combined with any relevant passages from Scholar Gateway's
`semanticSearch`. Between these two sources you'll usually have enough to assess the paper.

**Aim for full text on 5-10 of your most critical references** — this is where the
investment pays off. Don't attempt full text for all 15-30 references; it's unnecessary
and slows the search.

### Step 3b: Search for preprints and ongoing trials (optional)

These searches add valuable context, especially for rapidly evolving topics. Skip them
if the topic is well-established with mature guideline coverage.

#### medRxiv / bioRxiv preprints

If the **bioRxiv MCP** is available, use `search_preprints` to find recent preprints
on medRxiv (set `server: "medrxiv"`) for clinical topics, or bioRxiv for basic science.

- Search the last 12 months — preprints older than that should have been published by now
- Focus on preprints from major research groups or with large sample sizes
- Check whether a preprint has been published using `search_published_preprints` — if it
  has, cite the published version instead (find it via PubMed)
- **Label preprints clearly** in the ledger as `type: preprint` and in the evidence
  summary as "(preprint, not peer-reviewed)". They go in a separate "Emerging Evidence"
  subsection, not mixed with peer-reviewed papers

Preprints are valuable because they capture evidence that hasn't yet made it into the
peer-reviewed literature — but they haven't been through peer review, which is why the
separation matters.

#### ClinicalTrials.gov

If the **Clinical Trials MCP** is available, use `search_trials` to find ongoing or
recently completed trials relevant to the topic. This is most valuable for the
"Evidence Gaps" section.

- Search for trials with status `RECRUITING`, `ACTIVE_NOT_RECRUITING`, or `COMPLETED`
  (completed trials that haven't published yet are especially interesting)
- Use `get_trial_details` for trials that look directly relevant to the topic
- Use `analyze_endpoints` if comparing across multiple trials for the same condition
- Note the trial phase, estimated completion date, and sample size

This transforms "no RCT data exist for X" into "no RCT data exist yet, but
NCT04123456 is a Phase III trial (n=400) with results expected in 2027." That's
actionable intelligence for a clinician deciding whether to wait or act now.

### General principles

- **Quality over quantity**: Aim for 15-30 high-quality references total, not 100 marginal ones
- **UK-relevant data**: Include UK registry data (e.g., NHSBT reports) where available
- **De-duplicate**: Papers found by both PubMed and Scholar Gateway should appear only
  once in the reference list. Match on DOI to de-duplicate.

### Clinical questions to ask per subtopic

For each subtopic, ask:

1. Has the **recommended drug/dose/regimen changed**?
2. Are there **new agents or techniques** available?
3. Has the **evidence grade** for existing recommendations changed?
4. Are there **new safety signals** or contraindications?
5. Have **monitoring strategies** evolved?
6. Are there **new risk stratification** tools or criteria?
7. Has **UK practice** diverged from international practice?

### Record keeping — build the reference ledger as you go

This is critical for reference accuracy. As you search, maintain a **reference ledger**
— a structured list that you build incrementally during Steps 2-3 and then use verbatim
when writing the evidence summary and .bib file. This prevents DOI/title mismatches.

For every paper you plan to cite, call `get_article_metadata` to retrieve the full
metadata, then record these fields **exactly as PubMed returns them**:

```
REF_ID: [sequential number, assigned when you decide to include the paper]
PMID: [from PubMed]
DOI: [from PubMed metadata — copy exactly, do NOT reconstruct from memory]
AUTHORS: [from PubMed metadata]
TITLE: [from PubMed metadata — copy exactly]
JOURNAL: [from PubMed metadata]
YEAR: [from PubMed metadata]
VOLUME/PAGES: [from PubMed metadata]
KEY_FINDING: [your one-sentence summary]
SOURCE: [pubmed / scholar_gateway / both]
FULL_TEXT_REVIEWED: [true / false]
```

**Why this matters:** DOIs are opaque identifiers — there is no way to guess or
reconstruct a DOI from a paper title. If you write a DOI from memory rather than
copying it from the PubMed metadata, it will almost certainly be wrong and will link
to the wrong paper (or a dead link). This destroys the reader's trust in the entire
document. Always copy DOIs character-for-character from `get_article_metadata` output.

**Write the ledger to a file as you go.** After each batch of searches (e.g., after
completing one subtopic's PubMed + Scholar Gateway queries), append the new entries
to a working file called `_working_ledger.yaml`. This protects against context loss
during long searches with many tool calls. The final `_References.yaml` is a
cleaned-up version of this working file with sequential ref_ids assigned.

For guidelines found via web search (not in PubMed), record:
```
REF_ID: [sequential number]
TYPE: guideline
TITLE: [guideline title and edition]
ORGANISATION: [BTS, NICE, KDIGO, etc.]
YEAR: [publication year]
URL: [the URL you found it at]
KEY_RECOMMENDATIONS: [list of key recommendations with evidence grades]
```

When writing the reference list in Step 5, use ONLY the data from this ledger.
Do not add any reference that is not in the ledger. Do not modify DOIs or titles.

## Step 4: Verify Reference Integrity

Before generating outputs, verify that every reference is accurate. This step
is essential because incorrect DOIs or misattributed papers undermine the entire output.

### Verification checklist

1. **DOI spot-check**: For at least 5 PubMed references, call `get_article_metadata` with
   the PMID and confirm the DOI in your ledger matches exactly. If any don't match,
   check and fix all references.

2. **Scholar Gateway-only references**: For every reference with `source: scholar_gateway`
   (no PMID), verify the DOI format is valid (`10.XXXX/...`) and attempt to cross-check
   by searching PubMed for the title. If you find a PubMed match, update the ledger with
   the PMID and PubMed metadata. If no PubMed match exists, keep the Scholar Gateway
   metadata but note this reference has lower verification confidence.

3. **Title-PMID consistency**: Confirm that the title in your reference ledger matches the
   title returned by PubMed for that PMID. A title mismatch means the wrong paper is cited.

4. **Retraction check**: Scan all PubMed reference titles for "[Retracted]" or
   "[Retraction of:" markers. Also check whether `get_article_metadata` indicates
   retraction status. A retracted paper must never appear in a clinical evidence summary
   — remove it and note the retraction in the Evidence Gaps section if it was a key paper.

5. **No fabricated references**: Every PubMed reference must have a real PMID that you
   retrieved during Step 3. Never invent a PMID or DOI.

6. **DOI format check**: DOIs should look like `10.XXXX/...` — if any DOI doesn't start
   with `10.`, it's malformed. The full URL form is `https://doi.org/10.XXXX/...`.

7. **Citation-reference mapping**: Every `[N]` citation in the evidence summary must
   correspond to entry N in the reference list. Every reference must be cited at least once.

If you find errors during verification, fix them before generating the final outputs.

## Step 5: Generate the Evidence Summary Document

Use a **markdown-first** approach: write all content as a structured Markdown file, then
convert to .docx using pandoc with the bundled reference template. This is more reliable
than building docx programmatically.

Read `references/evidence_summary_template.md` for the full template, markdown structure,
and pandoc conversion command.

### High-level process

1. **Read `references/evidence_summary_template.md`** for the full document structure,
   content rules, and pandoc conversion command
2. **Write the evidence summary as Markdown** with YAML frontmatter for the title page
3. **Convert to .docx** using pandoc with the bundled `assets/reference.docx` template
4. **Generate .bib and PMIDs.txt** files for Zotero import (format specified in the template)

The template file covers the document structure, draft callout, transparency disclaimer,
content standards, and reference formatting. Follow it exactly — do not deviate from
the structure defined there.

## Step 6: Output All Files

Save all files to the user's workspace folder:

1. **`[Topic_Name]_Evidence_Summary_[Year].md`** — markdown source
2. **`[Topic_Name]_Evidence_Summary_[Year].docx`** — converted Word document
3. **`[Topic_Name]_References.yaml`** — structured reference ledger (the contract for protocol-reviewer)
4. **`[Topic_Name]_References.bib`** — BibTeX for Zotero import
5. **`[Topic_Name]_PMIDs.txt`** — one PMID per line for Zotero bulk import

### YAML reference ledger format

The YAML ledger is the structured output that the `protocol-reviewer` skill consumes.
It must follow this exact format:

```yaml
metadata:
  topic: "[clinical topic]"
  search_date: "[YYYY-MM-DD]"
  skill_version: "1.3.0"
  model_id: "[model identifier]"
  mesh_terms:
    - "[MeSH term 1]"
    - "[MeSH term 2]"
  guideline_bodies:
    - "[BTS]"
    - "[NICE]"

guidelines:
  - ref_id: 1
    type: guideline
    title: "[guideline title and edition]"
    organisation: "[body]"
    year: [YYYY]
    url: "[URL]"
    key_recommendations:
      - text: "[recommendation text]"
        grade: "[evidence grade]"

references:
  - ref_id: 2
    pmid: "[PMID or null]"
    doi: "[DOI]"
    authors: "[authors]"
    title: "[title — copied exactly from PubMed]"
    journal: "[journal name]"
    year: [YYYY]
    volume: "[volume]"
    pages: "[pages]"
    key_finding: "[one-sentence summary]"
    source: "[pubmed / scholar_gateway / both]"
    full_text_reviewed: [true / false]

# Include only if preprints were found (Step 3b)
preprints:
  - ref_id: 20
    doi: "[DOI]"
    authors: "[authors]"
    title: "[title]"
    server: "[medrxiv / biorxiv]"
    year: [YYYY]
    key_finding: "[one-sentence summary]"
    published_version_doi: "[DOI of published version, or null]"

# Include only if relevant ongoing trials were found (Step 3b)
ongoing_trials:
  - nct_id: "[NCT number]"
    title: "[trial title]"
    phase: "[Phase I / II / III / IV]"
    status: "[RECRUITING / ACTIVE_NOT_RECRUITING / COMPLETED]"
    estimated_completion: "[YYYY-MM]"
    sample_size: [N]
    relevance: "[one-sentence summary of why this trial matters]"
```

---

## Important Considerations

### Tone and audience

The evidence summary will be read by clinicians. Write at a peer level — authoritative
but not patronising. Use precise clinical terminology. Avoid hedging excessively; if
the evidence is clear, say so directly.

### Handling uncertainty and conflicting recommendations

Where evidence is conflicting or low-quality, acknowledge this explicitly. Phrases like
"the evidence base is limited to single-centre retrospective studies" or "no RCT data
exist for this specific question" are appropriate and helpful.

Pay particular attention to **conflicting guideline recommendations** — it is common
for national bodies to disagree (e.g., BTS vs KDIGO on immunosuppression protocols,
NICE vs ESC on anticoagulation thresholds). When guidelines conflict, present both
positions with their evidence grades and note the discrepancy explicitly. Do the same
when a recent RCT contradicts a guideline published before it — the clinician needs
to see the tension, not have it smoothed over.

### UK context

Always frame findings in the UK NHS context:
- Reference UK regulatory status of drugs (MHRA, not FDA)
- Consider NICE technology appraisals where relevant
- Reference UK registries (NHSBT, UKRR, etc.)
- Note where UK practice differs from US/European practice

---

## Tool Dependencies

This skill requires:

- **PubMed MCP** (`search_articles`, `get_article_metadata`, `get_full_text_article`, `find_related_articles`) — keyword/MeSH literature search with PMIDs and full text retrieval for open-access papers
- **Scholar Gateway** (`semanticSearch`) — semantic full-text search across peer-reviewed literature; complements PubMed by finding papers that keyword searches miss and returning relevant text passages directly
- **WebSearch / WebFetch** — for finding and reading current national guidelines
- **pandoc** — for converting markdown to .docx (bundled reference template in `assets/reference.docx`)

Optional tools (enhance coverage when available):

- **bioRxiv MCP** (`search_preprints`, `search_published_preprints`) — preprint search for emerging evidence not yet in peer-reviewed literature
- **Clinical Trials MCP** (`search_trials`, `get_trial_details`, `analyze_endpoints`) — ongoing/completed trial data for evidence gap analysis

If the user has uploaded guideline PDFs, prefer reading those directly over web-fetched
summaries, as you'll have the complete text including evidence grade tables.

### Handling missing tools

- **PubMed MCP unavailable:** Tell the user that PubMed MCP is required and link them
  to the setup instructions. Do not attempt to substitute with WebSearch for literature
  searches — the reference integrity workflow depends on PubMed metadata.
- **Scholar Gateway unavailable:** Proceed with PubMed only but note in the Search Strategy
  section that semantic search was not available and coverage may be narrower.
- **bioRxiv MCP unavailable:** Skip Step 3b preprint search. Note in the Search Strategy
  section that preprint search was not performed.
- **Clinical Trials MCP unavailable:** Skip Step 3b trial search. The Evidence Gaps section
  will not include ongoing trial information.
- **pandoc not installed:** Generate the .md file and all other outputs, then tell the user
  to install pandoc (`brew install pandoc` on macOS) and provide the exact conversion
  command they can run manually.
- **Scholar Gateway returns zero results:** This is normal for very narrow topics. Note it
  and rely on PubMed results. Do not treat zero Scholar Gateway results as an error.
