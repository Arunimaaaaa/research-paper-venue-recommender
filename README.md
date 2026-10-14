# Research Paper Venue Recommender

Upload a research paper (PDF) and get a ranked list of suitable journals to submit it to, chosen from **48,151 Scopus-indexed venues**, with a reason for each recommendation. The app also gives a writing-quality score and suggestions to improve the paper before submission.

*Team project (4 members), Vidyalankar Institute of Technology. Built for our Probabilistic Graphical Models and Recommendation Systems coursework.*

## Why

Choosing where to submit is hard for students: there are tens of thousands of journals, publisher "journal finder" tools only show their own journals, and a first-time author has no history to learn from. We treat venue selection as a **recommendation problem** that works without any user history (cold start).

## Features

- **PDF analysis:** text extraction, abstract detection or extractive summary
- **Venue recommendations:** top 10 journals with a match score (0–100) and an explanation
- **Author preferences:** open access only, active venues only, Medline-indexed, venue type, publisher, minimum coverage year
- **Similar venues:** TF-IDF and cosine similarity between the paper and venue titles
- **Writing feedback:** Random Forest quality score (0–10) plus suggestions on length, sentence structure, citations and conclusion

## How the recommender works

1. **Data preparation:** the Scopus source list (Jan 2026) was profiled column by column for missing values; sparse columns were dropped and meaningful blanks filled (e.g. no Medline flag means not Medline-indexed). 31,254 venues are active. See `docs/PREPROCESSING_PLAN.md`.
2. **Paper profile:** the research domain is detected from keywords and mapped to Scopus ASJC subject areas.
3. **Utility score:** every venue gets a weighted score:

| Criterion | Weight | What it checks |
|---|---|---|
| Subject match | 35% | Venue covers the paper's subject area |
| Venue quality | 25% | Active status, coverage history, Medline, newly added |
| Paper–venue fit | 25% | Language, open access, coverage recency |
| Publisher profile | 15% | Publisher reputation, source type |

4. **Preferences** are applied as soft constraints: a venue that misses one is multiplied by a penalty (e.g. ×0.6 if open access is required but the venue is not).
5. The top 10 venues are returned with a generated reason.

## Evaluation

We tested the recommender on **100 recent arXiv abstracts from 10 fields**, counting a recommended venue as relevant if it covers the expected subject area.

| Metric | Our recommender | TF-IDF title baseline |
|---|---|---|
| Hit rate@10 (≥ 1 relevant venue) | 82% | 25% |
| Precision@10 | 46.7% | 7% |

**Known limitations** found by this evaluation: keyword domain detection is weak outside computer science (the keyword "ai" matches inside words like "obtain"); the Medline check expects "yes" while the data says "Medline"; and broad multidisciplinary journals are favoured. Fixing these is the next step.

## Tech stack

React · Node.js / Express · Python Flask · pandas · scikit-learn · PyMuPDF

```
React frontend (3000)  ──►  Flask ML API (5000): /predict, /recommend, /health
                       └─►  Express gateway (3001): /analyze, /api/recommend
```

## Run it locally

```
# Python API
pip install -r requirements.txt
python app.py

# Frontend (new terminal)
cd frontend
npm install
npm start
```

Open http://localhost:3000.

## Data

Scopus Source Title List (Elsevier, January 2026): `datasets/ext_venues_enhanced.csv` (48,151 venues, 52 columns), `ext_venues_active.csv` (31,254 active venues).
