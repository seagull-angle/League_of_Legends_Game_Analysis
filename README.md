# League_of_Legends_Game_Analysis
Data science analysis of League of Legends match data for DSC 80 at UCSD

## Step 1: Introduction

### Dataset Overview

This project analyzes professional League of Legends esports matches from the 2022 season, using data compiled by Oracle's Elixir. The dataset contains **150,588 rows** and **164 columns**, with each match generating 12 rows (10 players + 2 team summaries). We focus on **team-level data** (25,098 team records) with performance statistics including gold, kills, deaths, creep score, and experience at 10, 15, 20, and 25-minute intervals.

Key columns include: `teamname`, `result` (win/loss), `gamelength`, `teamkills`, `teamdeaths`, `totalgold`, and time-stamped metrics like `goldat20`, `killsat20`, `deathsat20`, etc.

### Research Question

**Do certain professional League of Legends teams perform significantly differently in longer games compared to their overall performance, and what in-game factors drive success in extended matches?**

### Why This Matters

**Strategic Value:** Teams excel at different game paces—some dominate through early-game aggression (favoring short games), others through late-game scaling (favoring long games). Understanding these patterns helps teams optimize champion selection, pacing strategies, and opponent preparation.

**Predictive Insights:** If teams show statistically different performance in long games, we can build better prediction models that account for team-specific patterns rather than treating all teams identically. This reveals whether game length is merely a consequence of team strength or an independent factor affecting outcomes.

**Professional Meta Understanding:** Analyzing what drives victory in long games versus short games reveals whether the competitive meta rewards early aggression or late-game execution, and which statistics (gold, kills, deaths) become disproportionately important as matches extend.
