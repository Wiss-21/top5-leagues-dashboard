# Europe's Top 5 Leagues: Interactive Tableau Dashboard

A cross-league comparison of Europe's top five football leagues (**Premier League, La Liga, Bundesliga, Serie A, Ligue 1**) for the **2025/26** season. The data is pulled directly from a REST API in Python, reshaped with pandas, and visualised in a five-page Tableau Public dashboard.

**🔗 Live dashboard:** [Tableau Public](https://public.tableau.com/app/profile/wissam.ezzedine2939/viz/EuropesTop5Leagues-2025_26/SeasonSnapshot)
**🔗 Portfolio:** [wissam-ezzedine.netlify.app](https://wissam-ezzedine.netlify.app)

---

## The question

Football stats are usually shown one league at a time. Put the top five side by side and different questions open up: *Is home advantage the same in every league? Which league scores the most? Which teams won more points than their goal difference alone would suggest? Who controlled their league from week to week?* This project explores those with a clean, comparable dataset built from scratch. It covers a single season, so the results describe 2025/26 and are not forecasts.

## Data source & extraction

Data comes from the free tier of the [Football-Data.org](https://www.football-data.org) REST API (standings, matches, and scorers per competition, `season=2025`). The notebook:

- authenticates with an API token, looping over the five league codes (`PL`, `PD`, `BL1`, `SA`, `FL1`);
- respects the free-tier rate limit (10 calls/min);
- tags every row with its league, keeps finished matches only, and writes tidy CSVs;
- pulls the top 20 scorers per league.

The token is read from a local `config.py` that is **git-ignored**, so it never enters the repo. To run this yourself, create `config.py` with:

```python
TOKEN = "your_football_data_api_token"
```

## Data modelling

Home/away information lives one row per match, so the notebook reshapes it into a **team-match table** (one row per team per match, with venue, goals for/against, goal difference, and points). In Tableau the tidy tables (`team_matches`, `standings`, `scorers`) are related on `team` + `league`, a proper model rather than one flat file. `matches.csv` is the match-level source and is not used directly by the workbook.

The workbook's parameter (`League Selector`) and calculated fields (for example Home Win %, Points per Game, Goals per Game, Competitiveness Index, and the champion and final-matchday labels) are built directly in Tableau.

The CSVs also carry two sets of derived columns that the workbook reads as plain fields:

- **Matchday-by-matchday league position** (`cumulative_points`, `cumulative_gd`, `cumulative_gf`, `league_rank` in `team_matches.csv`), used for the Season Story chart.
- **Over/underperformance** (`predicted_points`, `residual`, `is_top_overperformer`, `is_top_underperformer` in `standings.csv`), used on the Scouting Signals page. `predicted_points` reproduces exactly from an ordinary least-squares straight line of season points on goal difference, fitted separately for each league (18 or 20 teams per league; R-squared between 0.94 and 0.97). `residual` is actual points minus `predicted_points`. The team with the largest positive residual in each league is flagged as the top overperformer and the largest negative one as the top underperformer. This is a descriptive fit on one season, not a forecast, and a straight line is a rough fit at the extremes (for example, a very large goal difference).

## Dashboards

1. **Season Snapshot**: each league's champion with final points and points per game, season-wide totals (goals scored, matches played, goals per match), and the biggest result of the season.
2. **Competitive Landscape**: home win rate, goals per game, and a "competitiveness index" by league. The index is the standard deviation of per-match points (0, 1 or 3) for each league, not the spread of final league points; it is almost identical across leagues (about 1.30 to 1.32), so it does not separate them.
3. **Season Story**: a line chart of every team's league position across all 34 or 38 matchdays, with a league selector.
4. **Scouting Signals**: points vs. goal difference, faceted by league, labelling the one team in each league with the largest positive and negative residual from the fit described above.
5. **Top Scorers & Full Table**: the top 10 goalscorers across all five leagues, plus the complete standings table for the selected league.

Every page shares a single league-selector parameter that filters all five dashboards at once.

## Key findings (2025/26)

Figures below are recomputed from the CSVs in `data/`.

- **Champions:** FC Barcelona (94 pts, 2.47 per game), FC Bayern München (89 pts, 2.62 per game), FC Internazionale Milano (87 pts, 2.29), Arsenal FC (85 pts, 2.24), Paris Saint-Germain FC (76 pts, 2.24). Bundesliga and Ligue 1 seasons are 34 matches and the other three leagues 38, so raw points are not comparable across leagues; per game, Bayern's 2.62 is the highest of any team.
- **Top scorers:** Harry Kane (36, Bundesliga), Erling Haaland (27, Premier League), Kylian Mbappé (25, La Liga), Esteban Lepaul (21, Ligue 1), Lautaro Martínez (17, Serie A) lead their leagues. Across all five leagues the next names are Vedat Muriqi (23) and Thiago (22). Four players tied on 17 goals (Budimir, Semenyo, Lautaro Martínez, Guirassy), so a strict top 10 cuts through a tie.
- **Home win rate (home wins over all finished matches):** La Liga 48.9% (186 of 380), Ligue 1 46.2% (141 of 305), Bundesliga 43.8% (134 of 306), Premier League 42.6% (162 of 380), Serie A 38.9% (148 of 380); 44.0% overall (771 of 1,751). A chi-square test across the five leagues gives p = 0.07, so the differences are not significant at the 5% level. Only La Liga vs Serie A is significant on its own (two-proportion z-test, p = 0.006), and that is one of ten pairwise comparisons, so I treat it as suggestive.
- **Goals per match:** Bundesliga 3.24, Ligue 1 2.83, Premier League 2.75, La Liga 2.69, Serie A 2.43. The Bundesliga is higher than each other league (Welch t-test on match goals, p < 0.01 for every pair).
- **Biggest result of the season:** Bayern Munich 8-1 Wolfsburg (Bundesliga, matchday 16), a 7-goal margin.
- **4,844 goals** scored across **1,751 finished matches** (2.77 per match).
- **Largest points-vs-goal-difference residual in each league (points above/below the fitted line):**
  - Over: Napoli +8.5 (Serie A), Aston Villa +8.0 (Premier League), RB Leipzig +7.0 (Bundesliga), Stade Rennais +5.7 (Ligue 1), Athletic Club +3.6 (La Liga).
  - Under: FC Bayern München -8.6 (Bundesliga), Nantes -6.9 (Ligue 1), Como -6.3 (Serie A), Osasuna -6.0 (La Liga), Nottingham Forest -5.6 (Premier League).
  - Bayern's -8.6 is the largest shortfall in the Bundesliga even though its +86 goal difference was the best of the five leagues. With one season of data these gaps could be luck, finishing, or fit error, so I read them as teams worth a closer look, not as predictions.

## Tech stack

`Python` · `requests` · `pandas` · `Jupyter` · `Tableau Public` · `Football-Data.org REST API`

## Repository structure

```
top5-leagues-dashboard/
├── notebooks/
│   └── extract_football_data.ipynb        # API extraction (standings, matches, scorers) + team-match reshape
├── data/
│   ├── standings.csv                       # final tables, plus predicted_points, residual and over/underperformer flags
│   ├── matches.csv                         # finished matches, one row per match
│   ├── team_matches.csv                    # one row per team per match, plus cumulative points/GD/GF and matchday league rank
│   └── scorers.csv                         # top 20 scorers per league
├── Europe's Top 5 Leagues - 2025_26.twbx   # Tableau workbook (current season)
├── requirements.txt
├── config.py                               # your API token (git-ignored)
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

- The free API tier provides **goals only** (no xG, shots, or assists), so the analysis is built around results, goals, and goal difference rather than expected-goals metrics. A natural next step would be layering in event-level data (for example StatsBomb's free open data) for xG-based over/underperformance.
- It is **one season** (2025/26), so none of the comparisons say whether a gap persists from year to year.
- The extraction notebook writes the base columns only; the matchday-rank and over/underperformance columns are added outside the notebook, so the repo does not yet show how they were produced.
- One known data gap: the Ligue 1 fixture between Nantes and Toulouse is missing from the finished matches the API returned (305 of 306 Ligue 1 matches), and the API's own standings show both teams on 33 games played instead of 34. The API does not say why, so I treat it as not played; confirm against the official final table before relying on Ligue 1 details. Because both teams have one fewer match, their Season Story lines end a matchday early and the final-matchday rank for several Ligue 1 teams does not match the final table position.
- The "most competitive league" question is not settled here: the dashboard's competitiveness index is nearly identical across leagues (see Dashboards above). For reference, the standard deviation of final league points is 15.8 in the Premier League, 16.0 in Ligue 1, 16.2 in La Liga, 17.6 in the Bundesliga and 18.3 in Serie A, but the leagues have 18 or 20 teams and 34 or 38 matches, so I would not rank them on that alone.
