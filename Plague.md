A mysterious plague sweeps through the league each season. Players randomly fall victim, being sidelined for the entire year — changing the competitive landscape in unpredictable ways.

# How It Works
- At the start of each season, every player has a small chance of being infected
- Infected players are ruled out for 150 games, covering the entire regular season and all playoffs/play-ins
- Infected players remain on rosters but do not play at all that year
- The current version does not infect undrafted players (e.g., rookies from upcoming drafts)
- It's recommended to run the plague just before the draft begins, so:
  - Newly drafted players aren’t affected
  - You maintain a healthier player pool for the season
  - Knowing who is infected before making offseason decisions (trades, free agency, etc.). Doesn't really matter for AI though.
- Enable the "No starting injuries" setting under League Settings. Only works with Real Player Mode

```js
var season = bbgm.g.get("season") + 1;
var players = await bbgm.idb.cache.players.getAll();

for (const p of players) {
    if (p.tid >= -1) {
      if (Math.random() < 0.5) {
        p.injury = {"type": "Plague", "gamesRemaining": 150}
        p.injuries.push({season: season, games: 150, type: "Plague"})
        await bbgm.idb.cache.players.put(p);
      }
    }
}
```
