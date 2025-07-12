# How It Works
- At the start of every season, one player from each team is randomly selected and instantly killed (retired with cause)

- Run the assassination after the first game of the season, to ensure all AI-controlled teams have fully formed rosters
  - Prevents bugs where teams may not have enough players
  - Ensures maximum randomness and fairness
 
```js
var players = await bbgm.idb.cache.players.getAll();
var season = bbgm.g.get("season");
var teams = {}

// Get teams of all players
for (const p of players) {
    if (p.tid >= 0) {
      if (p.tid in teams){
        teams[p.tid].push(p)
      }
      else {
        teams[p.tid] = [p]
      }
    }
}

async function customTragedy(pid, reason) {
    const p = await bbgm.idb.getCopy.players({ pid });
    const tid = p.tid;
    await bbgm.player.retire(p, {});    
    p.diedYear = season;
    await bbgm.idb.cache.players.put(p);
    bbgm.logEvent(
        {
            type: "tragedy",
            text: `<a href="${bbgm.helpers.leagueUrl(["player", p.pid])}">${p.firstName} ${
                p.lastName
            }</a> ${reason}.`,
            showNotification: false,
            pids: [p.pid],
            tids: [tid],
            persistent: true,
            score: 20,
        },
    );
}

// Pick a random player on each team to assassinate
for (const t in teams){
  function getRandomItem(arr) {
    const randomIndex = Math.floor(Math.random() * arr.length);
    const item = arr[randomIndex];
    return item;
  }

  var playerToKill = getRandomItem(teams[t])
  customTragedy(playerToKill.pid, "Assassinated");
  console.log(playerToKill.firstName + " " + playerToKill.lastName + " - " + playerToKill.ratings[playerToKill.ratings.length - 1].ovr)
}
    
```
