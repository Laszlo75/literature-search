# PubMed Search Strategy for Protocol Reviews

## General Approach

Run multiple focused searches rather than one broad query. Each search should target
a specific clinical question raised by the protocol.

## Search Construction

### Step 1: Identify the core clinical topic

From the protocol, extract the primary condition/procedure and the specific interventions.

Example for an ABOi transplant protocol:
- Core: `ABO incompatible kidney transplantation`
- Subtopics: rituximab dosing, plasmapheresis, immunoadsorption, titre targets,
  tacrolimus, mycophenolate, desensitisation outcomes

### Step 2: Build targeted queries

For each subtopic, construct a query using this pattern:

```
[condition/procedure] AND [intervention/topic] AND [study type filter]
```

**Study type filters** (append to narrow results):

| Filter | PubMed syntax |
|--------|--------------|
| Reviews only | `AND (Review[Publication Type] OR Systematic Review[Publication Type])` |
| Meta-analyses | `AND Meta-Analysis[Publication Type]` |
| RCTs | `AND Randomized Controlled Trial[Publication Type]` |
| Guidelines | `AND (Guideline[Publication Type] OR Practice Guideline[Publication Type])` |
| High quality mix | `AND (Review[pt] OR Meta-Analysis[pt] OR Randomized Controlled Trial[pt])` |

### Step 3: Date filtering

- **Primary window**: Last 5 years from today
- **Extended window**: Last 10 years for landmark studies
- Use `date_from` parameter, not query syntax, for date filtering

### Step 4: Iterative refinement

1. Start with the broadest relevant query, `max_results: 20`, sorted by `pub_date`
2. Review titles/abstracts via `get_article_metadata`
3. For the most relevant papers, use `find_related_articles` to discover similar work
4. If too few results, broaden terms or remove the study type filter
5. If too many results, add more specific terms or tighten the date range

## Example Searches (Transplant Protocol)

```
# Outcomes and overview
"ABO incompatible kidney transplantation" AND (outcomes OR survival OR graft)
  date_from: 5 years ago, sort: pub_date, max_results: 20

# Desensitisation regimen
"ABO incompatible" AND (desensitization OR desensitisation) AND kidney
  AND (Review[pt] OR Meta-Analysis[pt])
  date_from: 5 years ago

# Specific drug
rituximab AND "ABO incompatible" AND (kidney OR renal) AND transplant
  date_from: 5 years ago

# Antibody removal technique
(plasmapheresis OR immunoadsorption OR "plasma exchange") AND "ABO incompatible"
  AND kidney AND transplant
  date_from: 5 years ago

# Titre measurement
(isoagglutinin OR "anti-A" OR "anti-B") AND titer AND kidney AND transplant
  date_from: 5 years ago

# Complications
"antibody mediated rejection" AND "ABO incompatible" AND kidney
  date_from: 5 years ago
```

## Quality Assessment Heuristic

When selecting which papers to include in the review, prioritise:

1. **Cochrane reviews and well-conducted meta-analyses** — highest value
2. **Multi-centre RCTs** — strong evidence for intervention questions
3. **Large registry studies** (>500 patients) — good for outcomes data
4. **National/international society position statements** — reflect consensus
5. **Single-centre cohort studies** (>100 patients) — useful if no better evidence exists
6. **Case series** (<50 patients) — include only if it's the only evidence available

De-prioritise:
- Case reports (unless describing a novel safety signal)
- Conference abstracts without full publication
- Narrative reviews without systematic methodology
- Papers from predatory journals

**Exclude entirely:**
- Retracted papers — check for "[Retracted]" in PubMed titles. If a key paper
  has been retracted, note the retraction in the Evidence Gaps section

## Target Reference Count

Aim for **15-30 references** for a typical protocol review:
- 2-3 national/international guidelines
- 3-5 systematic reviews or meta-analyses
- 5-10 key primary studies (RCTs, large cohorts)
- 2-5 papers addressing specific protocol details (dosing, monitoring, etc.)

More is fine if the protocol covers many topics, but every reference should earn its place
by directly supporting a specific recommendation.
