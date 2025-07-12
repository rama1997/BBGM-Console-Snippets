Fountain of Youth: Championship team rewarded with being 1 year younger.

Tips:
- Run right after the playoffs and before the draft

```js
var season = bbgm.g.get("season");
var players = await bbgm.idb.cache.players.getAll();
console.log(season)

for (const p of players) {
    const playerAwards = p.awards;
    for (const a of playerAwards){
        if (a.season === season && a.type === "Won Championship"){
           p.born.year = p.born.year+1
           console.log(p.firstName + " " + p.lastName)
        }
    }
}
```
