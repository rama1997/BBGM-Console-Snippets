## Stone's Power
- Power Stone — Raw Physical Dominance
  - +100 to Strength, Dunk, Endurance, Rebounding, Height, and Blocking
- Time Stone — Perpetual Prime
  - Age is locked at 25 (prime age)
  - Ratings set to the player’s career-best across all seasons
  - +100 to Speed, symbolizing the illusion of moving faster by stopping time
- Space stone — Teleportation and Positioning
  - +100 to Jumping – leaping through space
  - +100 to Passing – for pinpoint, space-folding passes
  - +100 to Stealing – intercepting as if teleporting
  - Holder is automatically traded to the reigning champion at the start of the season
- Reality
  - Alter Reality to always "make the shot"
  - +100 to Field Goal, Free Throw, Three Point, Inside Scoring, and Dunk
- Soul — Sacrifice
  - The holder is soul-bound to their most played team; returns to that team each season
  - Each season, a random teammate is sacrificed, and the holder absorbs a portion of their stats
- Mind
  - +100 to Offensive IQ and Defensive IQ
  - Teammates gain +2 IQ across the board — connected by a hive mind
  - Opponents lose -2 IQ — mind control
  
## Notes
- General
	- Stone Transfer: Each season, there's a 6% chance a stone leaves its current holder and is randomly reassigned to another player
	- Stone Activation: Mode is designed for stone activation/running code at the start of the season
- Bugs
	- Space/Soul Stone Bug: Players may be assigned to a team that no longer exists (e.g., after contraction or relocation). This won't crash the game, but the sim may stop progressing. You’ll need to manually reassign the player to an active team in this case.
- Tips
	- Sim one full season before activating the mode to ensure your league setup is stable
	- If a stone holder is traded mid-season, there’s no fix yet. You can use lore to justify it: "The Infinity Stones overwhelm mortal bodies and cannot be used too frequently..."

```js
var currentSeason = bbgm.g.get("season");
var lastSeason = currentSeason - 1;
var teams = await bbgm.idb.cache.teams.getAll();
var allPlayers = await bbgm.idb.cache.players.getAll();
allPlayers = allPlayers.filter((p) => p.tid >= -1);

var stones = ["Power", "Reality", "Time", "Space", "Mind", "Soul"];

function getRandomPlayer() {
	const randomIndex = Math.floor(Math.random() * allPlayers.length);
	const item = allPlayers[randomIndex];
	return item;
}

async function addStone(player, stoneName) {
	player.awards.push({
		season: currentSeason,
		type: `${stoneName} Stone`,
	});
	await bbgm.idb.cache.players.put(player);
}

async function removeStone(holder, stoneName) {
	holder.awards = holder.awards.filter((award) => {
		if (award.season === currentSeason && award.type === `${stoneName} Stone`) {
			return false;
		}

		return true;
	});
	await bbgm.idb.cache.players.put(holder);
}

function assignNewStoneHolder(stoneName) {
	var randomPlayer = getRandomPlayer();
	addStone(randomPlayer, stoneName);
	return randomPlayer;
}

function getCurrentStoneHolder(stoneName) {
	for (const p of allPlayers) {
		const playerAwards = p.awards;
		for (const a of playerAwards) {
			if (a.season === lastSeason && a.type === `${stoneName} Stone`) {
				addStone(p, stoneName);
				return p;
			}
		}
	}

	return null;
}

function getStoneHolder(stoneName) {
	let stoneHolder = getCurrentStoneHolder(stoneName);

	if (stoneHolder) {
		if (Math.random() < 0.06) {
			console.log(`${stoneHolder.firstName} ${stoneHolder.lastName} lost the ${stoneName} Stone!`);
			removeStone(stoneHolder, stoneName);
			stoneHolder = assignNewStoneHolder(stoneName);
		}
	} else {
		stoneHolder = assignNewStoneHolder(stoneName);
		console.log(`New ${stoneName} Stone Holder: ${stoneHolder.firstName} ${stoneHolder.lastName}`);
	}

	return stoneHolder;
}

async function activatePowerStone(holder) {
	const ratings = holder.ratings.at(-1);
	ratings.stre = bbgm.player.limitRating(100);
	ratings.dnk = bbgm.player.limitRating(100);
	ratings.endu = bbgm.player.limitRating(100);
	ratings.reb = bbgm.player.limitRating(100);
	ratings.hgt = bbgm.player.limitRating(100);
	ratings.blk = bbgm.player.limitRating(100);

	await bbgm.player.develop(holder, 0);
	await bbgm.player.updateValues(holder);
	await bbgm.idb.cache.players.put(holder);
}

async function activateTimeStone(holder) {
	const maxRating = {};

	// Check each year's stats and obtain the max value for each stats throughout career
	const ratingYearlyHistory = holder.ratings;
	for (const year of ratingYearlyHistory) {
		for (const stat in year) {
			if (stat in maxRating) {
				if (year[stat] > maxRating[stat]) {
					maxRating[stat] = year[stat];
				}
			} else {
				maxRating[stat] = year[stat];
			}
		}
	}

	// Make each current stats equal to the max values found in career
	const currRatings = holder.ratings.at(-1);
	for (const stat in maxRating) {
		currRatings[stat] = maxRating[stat];
	}

	// Set speed to 100 and age to 25
	currRatings.spd = bbgm.player.limitRating(100);
	holder.born.year = currentSeason - 25;

	await bbgm.player.develop(holder, 0);
	await bbgm.player.updateValues(holder);
	await bbgm.idb.cache.players.put(holder);
}

async function activateSpaceStone(holder) {
	async function getPreviousChampionTeamId() {
		for (const p of allPlayers) {
			const playerAwards = p.awards;
			for (const a of playerAwards) {
				if (a.season === lastSeason && a.type === "Won Championship") {
					let playerHistory = p.stats;
					for (const year of playerHistory) {
						if (year.season === lastSeason) {
							return year.tid;
						}
					}
				}
			}
		}
		return -1;
	}

	const previousChampionTeamId = await getPreviousChampionTeamId();
	if (previousChampionTeamId >= 0) {
		holder.tid = previousChampionTeamId;
	}

	const ratings = holder.ratings.at(-1);
	ratings.jmp = bbgm.player.limitRating(100);
	ratings.pss = bbgm.player.limitRating(100);
	ratings.stl = bbgm.player.limitRating(100);

	await bbgm.player.develop(holder, 0);
	await bbgm.player.updateValues(holder);
	await bbgm.idb.cache.players.put(holder);
}

async function activateSoulStone(holder) {
	async function sacrificePlayer(pid) {
		const p = await bbgm.idb.getCopy.players({ pid });
		const tid = p.tid;
		await bbgm.player.retire(p, undefined, {
			logRetiredEvent: true,
		});
		p.diedYear = currentSeason;
		await bbgm.idb.cache.players.put(p);
		bbgm.logEvent({
			type: "tragedy",
			text: `<a href="${bbgm.helpers.leagueUrl(["player", p.pid])}">${p.firstName} ${p.lastName}</a> Sacrificed to Soul Stone.`,
			showNotification: true,
			pids: [p.pid],
			tids: [tid],
			persistent: true,
			score: 20,
		});
	}

	// Soul bound team that holder has played the longest for
	const history = holder.stats;
	let teamCount = {};
	for (const year of history) {
		if (year.tid >= 0) {				
			teamCount[year.tid] = (teamCount[year.tid] || 0) + 1;
		}
	}
	if (teamCount) {
		let soulBoundTeamId = Object.keys(teamCount).reduce((a, b) => (teamCount[a] > teamCount[b] ? a : b));
		holder.tid = Number(soulBoundTeamId);
	}


	if (holder.tid >= 0) {
		const teamPlayers = [];
		for (const p of allPlayers) {
			if (p.tid === holder.tid && p.pid !== holder.pid) {
				teamPlayers.push(p);
			}
		}

		// Sacrifice one random teammate
		if (teamPlayers.length !== 0) {
			const randomIndex = Math.floor(Math.random() * teamPlayers.length);
			const playerToSacrifice = teamPlayers[randomIndex];
			await sacrificePlayer(playerToSacrifice.pid);

			// Obtain sacrificed teammate's stats
			const holderStats = holder.ratings.at(-1);
			const sacrificedPlayerStats = playerToSacrifice.ratings.at(-1);
			const statsToIgnore = ["fuzz", "ovr", "pos", "pot", "season", "skills", "injuryIndex"];
			for (const s in sacrificedPlayerStats) {
				if (!statsToIgnore.includes(s)) {
					holderStats[s] = bbgm.player.limitRating(holderStats[s] + Math.floor(sacrificedPlayerStats[s] * 0.05));
				}
			}

			await bbgm.player.develop(holder, 0);
			await bbgm.player.updateValues(holder);
		}
	}

	await bbgm.idb.cache.players.put(holder);
}

async function activateMindStone(holder) {
	if (holder.tid >= 0) {
		for (const p of allPlayers) {
			if (p.tid === holder.tid && p.pid !== holder.pid) {
				const ratings = p.ratings.at(-1);
				ratings.oiq = bbgm.player.limitRating(ratings.oiq + 2);
				ratings.diq = bbgm.player.limitRating(ratings.diq + 2);
			} else if (p.tid >= -1 && p.pid !== holder.pid) {
				const ratings = p.ratings.at(-1);
				ratings.oiq = bbgm.player.limitRating(ratings.oiq - 1);
				ratings.diq = bbgm.player.limitRating(ratings.diq - 1);
			}
			await bbgm.player.develop(p, 0);
			await bbgm.player.updateValues(p);
			await bbgm.idb.cache.players.put(p);
		}
	}

	const ratings = holder.ratings.at(-1);
	ratings.oiq = bbgm.player.limitRating(100);
	ratings.diq = bbgm.player.limitRating(100);

	await bbgm.player.develop(holder, 0);
	await bbgm.player.updateValues(holder);
	await bbgm.idb.cache.players.put(holder);
}

async function activateRealityStone(holder) {
	const ratings = holder.ratings.at(-1);
	ratings.fg = bbgm.player.limitRating(100);
	ratings.ft = bbgm.player.limitRating(100);
	ratings.ins = bbgm.player.limitRating(100);
	ratings.tp = bbgm.player.limitRating(100);
	ratings.dnk = bbgm.player.limitRating(100);

	await bbgm.player.develop(holder, 0);
	await bbgm.player.updateValues(holder);
	await bbgm.idb.cache.players.put(holder);
}

async function activateStonePower(player, stoneName) {
	if (stoneName === "Power") {
		await activatePowerStone(player);
	} else if (stoneName === "Time") {
		await activateTimeStone(player);
	} else if (stoneName === "Space") {
		await activateSpaceStone(player);
	} else if (stoneName === "Soul") {
		await activateSoulStone(player);
	} else if (stoneName === "Mind") {
		await activateMindStone(player);
	} else if (stoneName === "Reality") {
		await activateRealityStone(player);
	}
}

for (const s of stones) {
	var stoneHolder = getStoneHolder(s);
	await activateStonePower(stoneHolder, s);
	console.log(`${s} Stone: ` + stoneHolder.firstName + " " + stoneHolder.lastName);
}

```
