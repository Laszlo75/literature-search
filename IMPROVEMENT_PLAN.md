# Literature Search Skill — Improvement Plan

Created: 2026-03-22

## Critical Issues (implementing now)

### 1. Add bioRxiv/medRxiv and ClinicalTrials.gov integration
- **Problem**: The user has bioRxiv and Clinical Trials MCP tools available but the skill ignores them. medRxiv preprints provide very recent evidence on evolving clinical topics. ClinicalTrials.gov data transforms "no RCT data exist" into actionable intelligence about ongoing trials.
- **Fix**: Add optional Step 3b searching medRxiv for preprints (clearly labeled non-peer-reviewed) and ClinicalTrials.gov for ongoing/recently completed trials. Add preprint data to an "Emerging Evidence" subsection and trial data to the "Evidence Gaps" section.

### 2. Write reference ledger to file incrementally
- **Problem**: The skill tells the model to build the ledger "as you go" but doesn't specify where. For 25+ references across many tool calls, holding structured data in context memory is fragile — context is finite and lossy.
- **Fix**: Instruct the model to write the ledger to a working file (`_working_ledger.yaml`) after each batch of searches, appending as new references are found. The final `_References.yaml` is a cleaned-up copy.

### 3. Close Scholar Gateway DOI verification gap
- **Problem**: Line 173 claims Scholar Gateway DOIs are "reliable." Step 4 verification only spot-checks PubMed references. Scholar Gateway-only references get no verification.
- **Fix**: Remove the "reliable" claim. For Scholar Gateway-only papers, attempt to find the PMID via PubMed search and cross-verify the DOI. Add Scholar Gateway references to the Step 4 verification checklist.

### 4. Add retraction checking
- **Problem**: PubMed marks retracted papers but the skill never checks. Including a retracted paper in a clinical evidence summary would be a serious error.
- **Fix**: Add a retraction check to Step 4 verification — look for "[Retracted]" in PubMed titles or retraction notices.

### 5. Add explicit treatment of conflicting guidelines
- **Problem**: The template has "Evidence Gaps" but not "Conflicting Recommendations." BTS vs KDIGO disagreements are common and clinically important.
- **Fix**: Add a "Conflicting Recommendations" subsection to the template. Instruct the model to flag cases where guidelines disagree or where recent evidence contradicts guideline recommendations.

### 6. Add scope guardrails
- **Problem**: The skill assumes well-scoped clinical questions. No handling for "search for evidence on cancer treatment" (too broad) or extremely narrow topics with 2-3 papers.
- **Fix**: Add a scope check after Step 1. If too broad, ask user to narrow. If too narrow, warn and consider broadening search window.

## Medium-Priority Issues (implementing now)

### 7. Fix BibTeX citation key collisions
- **Problem**: Template uses `@article{AuthorYear,` which collides when two Smith2024 papers exist.
- **Fix**: Use `AuthorYeara`, `AuthorYearb` convention for collisions.

### 8. Fix version string fragility
- **Problem**: `skill_version` in YAML and `v[skill version]` in transparency disclaimer require the model to dynamically read and propagate the frontmatter version — models forget this.
- **Fix**: Hardcode version in the YAML schema example. Remove dynamic version from transparency disclaimer (the YAML metadata is sufficient).

### 9. Add `find_related_articles` guidance
- **Problem**: Mentioned once (line 155) with no guidance on when/how to use it effectively.
- **Fix**: Add specific guidance: use after finding a landmark paper (seminal RCT, key meta-analysis) to discover the citation neighbourhood. Limit to 2-3 seed papers to avoid exponential sprawl.

### 10. Move "Model Requirements" to description/README
- **Problem**: Lines 32-44 tell the executing model to use Opus 4.6 — but the model can't switch itself. This is user-facing advice.
- **Fix**: Keep the "think deeply" instruction (valid prompt guidance). Move the model recommendation to README.

## Lower-Priority (not implementing now)

- **Duplicate content across files**: DOI warning appears in 3 files. Monitor for drift.
- **Non-English literature**: Add a note that the skill searches English-language papers only.
- **Parallelization guidance**: Note that subtopic searches can run in parallel when subagents are available.
- **Evals lack assertions**: Add structured assertions for automated benchmarking.
- **pandoc path resolution**: Be more explicit about how the model finds `assets/reference.docx`.
