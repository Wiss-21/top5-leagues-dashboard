# Europe's Top 5 Leagues — Interactive Tableau Dashboard

A cross-league comparison of Europe's top five football leagues — **Premier League, La Liga, Bundesliga, Serie A, Ligue 1** — for the **2023/24** season. The data is pulled directly from a REST API in Python, reshaped with pandas, and visualised in a three-page Tableau Public dashboard.

**🔗 Live dashboard:** [Tableau Public](https://public.tableau.com/views/EuropesTop5Leagues202324/Rankings)
**🔗 Portfolio:** [wissam-ezzedine.netlify.app](https://wissam-ezzedine.netlify.app)

---

## The question

Football stats are usually shown one league at a time. Put the top five side by side and different questions open up: *Is home advantage really universal? Which league is the most competitive? Which teams won more than their goal difference deserved?* This project answers those with a clean, comparable dataset built from scratch.

## Data source & extraction

Data comes from the free tier of the [Football-Data.org](https://www.football-data.org) REST API (standings, matches, and scorers per competition). The notebook:

- authenticates with an API token, looping over the five league codes (`PL`, `PD`, `BL1`, `SA`, `FL1`);
- respects the free-tier rate limit (10 calls/min);
- tags every row with its league and writes tidy CSVs.

The token is read from a local `config.py` that is **git-ignored** — it never enters the repo. To run this yourself, create `config.py` with:

```python
TOKEN = "your_football_data_api_token"
```

## Data modelling

Home/away information lives one row per match, so the notebook reshapes it into a **team-match table** (one row per team per match, with venue, goals for/against, goal difference, and points). In Tableau these tidy tables are joined with **relationships** on `team` + `league` — a proper model rather than one flat file.

## Dashboards

1. **Leagues** — home advantage, goals per game, and competitiveness by league.
2. **Teams** — a points-vs-goal-difference efficiency scatter (over/underperformers) and an attack-vs-defense profile.
3. **Rankings** — top teams by points and top scorers, coloured by league.

## Key findings (2023/24)

- **Home advantage is not universal.** Home win rate ranges from **46% in the Premier League** down to **39% in Ligue 1** — a meaningful gap for anyone modelling match outcomes.
- **Highest-scoring leagues:** the Premier League and Bundesliga; La Liga and Serie A are the tightest defensively.
- **Champions:** Real Madrid (95 pts), Inter (94), Manchester City (91), Bayer Leverkusen (90, unbeaten), PSG (76).
- **Top scorer:** Harry Kane, 36 goals (Bundesliga).
- Several teams sit **above the points-vs-goal-difference trend line**, meaning they banked more points than their goal difference justified — the first regression candidates for next season.

## Tech stack

`Python` · `requests` · `pandas` · `Jupyter` · `Tableau Public` · `Football-Data.org REST API`

## Repository structure

```
top5-leagues-dashboard/
├── notebooks/
│   └── extract_football_data.ipynb   # API extraction + reshape
├── data/
│   ├── standings.csv
│   ├── matches.csv
│   ├── team_matches.csv              # reshaped (one row per team per match)
│   └── scorers.csv
├── Europe's Top 5 Leagues — 2023_24.twbx   # Tableau workbook
├── requirements.txt
├── config.py                         # your API token (git-ignored)
└── README.md
```

## Run it yourself

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# add your token to config.py, then run the notebook:
jupyter lab
```

## Notes & limitations

The free API tier provides **goals only** (no xG, shots, or assists), so the analysis is built around results, goals, and goal difference rather than expected-goals metrics. A natural next step would be layering in event-level data (e.g. StatsBomb's free open data) for xG-based over/underperformance.
