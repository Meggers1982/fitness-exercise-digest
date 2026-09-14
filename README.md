# Fitness & Exercise Research Digest

A GitHub Actions workflow that searches curated exercise science, sports medicine, physical rehabilitation, sports nutrition, and exercise behavior journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Dashboard publish** - Pushes the merged results to the shared research-digest-dashboard repo

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as Runner's World, Outside, Men's Health, Women's Health Magazine, Shape, Well+Good, Self, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Fitness & Exercise Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Sports Medicine | 49 | 2 (chunks 1-2) |
| Physical and Rehabilitation Medicine | 62 | 2 (chunks 1-2) |
| Nutritional Sciences | 62 | 2 (chunks 1-2) |
| Behavioral Sciences | 89 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The CSVs in `data/` are now hand-maintained. `scripts/extract_journals.py` built them from `~/PubMed_Journals_Categorized.xlsx`, which no longer exists, so re-running it would wipe any rows added by hand.

## Journal list audit (2026-09-14)

Method: pulled OpenAlex's top sources for the digest's subject areas over the prior year, diffed them against the four CSVs by ISSN and title, and kept only titles NCBI lists with PubMed articles in the last 12 months. Of 126 candidates, 102 were never usable (not in NCBI, or zero PubMed articles), and 24 were reviewed by hand.

Added to `Sports Medicine.csv`:

- **Journal of Strength and Conditioning Research** - 1533-4287, ~460 PubMed articles/yr, MEDLINE
- **BMC Sports Science, Medicine and Rehabilitation** - 2052-1847, ~600/yr, PMC
- **Journal of Aging and Physical Activity** - 1543-267X, ~140/yr, MEDLINE
- **Biology of Sport** - 2083-1862, ~130/yr, PMC
- **Journal of Human Kinetics** - 1899-7562, ~105/yr, PMC

Left out:

- **Not in PubMed** - most of the 102 dropped titles are Indonesian, Ukrainian and Russian physical-education journals (the largest: Scientific Journal of National Pedagogical Dragomanov University Series 15, International Journal of Physical Education Sports and Health, Gelanggang Olahraga JPJO, Uchenye Zapiski Universiteta imeni P.F. Lesgafta, COMPETITOR). English-language titles that also can't be searched: Sport, Education and Society; Journal of Teaching in Physical Education; Physical Education and Sport Pedagogy; European Physical Education Review; International Journal of Performance Analysis in Sport; International Journal of Strength and Conditioning; Quest; Human Movement.
- **Too few PubMed articles** - International Journal of Sports Science & Coaching (216 on-topic articles in OpenAlex, 2 in PubMed), Strength and Conditioning Journal, Sports Engineering, Biomechanics (MDPI).
- **MDPI, PMC-only** - Sports (~500/yr) and Journal of Functional Morphology and Kinesiology (~460/yr); at that volume they would crowd the 30 candidate slots without adding much signal.
- **Orthopedic surgery, not exercise** - Journal of Foot & Ankle Surgery, Foot and Ankle Surgery, Foot & Ankle International, Foot & Ankle Orthopaedics, Foot & Ankle Specialist, Foot and Ankle Clinics, The Foot.
- **Off-beat** - Osteoporosis International, Archives of Osteoporosis, Current Osteoporosis Reports and Journal of Clinical Densitometry (bone metabolism; already covered by the aging and women's health digests), Advances in Wound Care, Cereal Chemistry.

## Manual Trigger

Go to **Actions -> Fitness & Exercise Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Sports Medicine`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (enables personalization from dashboard save/delete feedback) |
| `SUPABASE_KEY` | Supabase API key (read-only use; skips personalization if not set) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to `Meggers1982/research-digest-dashboard` |

## Repo Structure

```text
.github/
  workflows/
    fitness-exercise-digest.yml
scripts/
  fitness_exercise_digest.py
  merge_results.py
  extract_journals.py
data/
  Sports Medicine.csv
  Physical and Rehabilitation Medicine.csv
  Nutritional Sciences.csv
  Behavioral Sciences.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for fitness, performance, injury prevention, and active living journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
