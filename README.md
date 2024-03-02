# GeoGeussr Duels Stats Fetcher

This repo contains a collection of scripts to fetch all of your duels and their stats, and output them into a spreadsheet-safe format.

I have included both the minified version to paste into the console and also the pre-compiled version if you don't trust the code (who would?). You can find the minified script [here](main_condensed.js) and the full version [here](main.js)


*Disclaimer:* I do not condone the spam nor abuse of GeoGuessr's API. These scripts are intended for data analysis purposes only, and are not for misuse or exploitation of GeoGuessr. The use of the API is non-official, and may be revoked or blocked at any time. Not affiliated with GeoGuessr or any related entities. By using this script, you acknowledge and agree that the author and publisher shall not be liable for any damages or injuries whatsoever resulting from your use or inability to use the script, even if the author or publisher has been advised of the possibility of such damages.

# Table of Contents

[Changelog](#changelog)

[Phase 1 - Fetching Duel IDs](#phase-1-fetching-duel-ids)

[Phase 2 - Fetching Data from IDs](#phase-2-fetching-duel-data)

[Phase 3 - Data Visualization](#phase-3-data-visualization)

[Phase 4 - Updating Stats](#phase-4-updating-your-stats)


# Changelog

**v1.0** - Initial Release

**v1.1** - Added random-order input for updating data

**v1.2** - Added date filter to spreadsheet

**v1.3** - Added country data & stats

**v1.4** - Added median damage diff (dealt - taken) on all rounds per country

~ changes by sp4ghet

**v1.5** - Added more info about opponents and an option to filter duels by game mode

# Instructions

Using the scripts to retrieve your data is very simple.

**Note:** All of the following steps were performed and screenshotted in Chrome. Other browsers may not work as intended, but feel free to try them out. Please make sure you are logged in or GeoGuessr's API won't let you access the data.

*Note 2:* The # of duels might not match the number on your profile. I'm not sure why but frankly it's close enough that I'm not going to bother finding out.

## Phase 1: Get the data

1. Open [your GeoGuessr profile](https://www.geoguessr.com/me/profile)

2. Scroll to the bottom of the page and find your profile link:

![Profile link](images/profile_link.png)

3. Keep note of the ID part. It should be a bunch of numbers/letters, something like `5ed8463f5422eaa2e8e51dfd`

4. While on the same page, press `F12` on your keyboard or `Right Click` and click `Inspect` at the bottom of the context menu.

5. Navigate to the `Console` tab of the new window that just opened:

![Console](images/console.png)

6. Paste the following code snippet into the console. Don't press enter or submit yet. Replace `selfId` with the ID from step 3.
   If this is your first time fetching data for v1.5, leave lastId as is. (If you want to update data, then see phase 3).

```js
const selfId = "--->YOUR ID HERE<---"
const lastId = "" // set this value if you only want to fetch from specific duel (see Phase 3)
```

7. Enter the following snippet
```javascript
async function main(e, t) { function n(e, t) { return new Promise(n => { setTimeout(() => { n(e()) }, t) }) } function o(e) { return [...e.matchAll(/\\"gameId\\":\\"([\w\d\-]*)\\",\\"gameMode\\":\\"Duels\\"/g)].map(e => e[1]) } function a(t) { return btoa(`{"HashKey":{"S":"${e + "_activity"}"},"Created":{"S":"${t}"}}`) } async function r(e = 1, t = "", r = "") { let p = r, s = []; for (let l = 0; l < e; l++) { console.log("Fetching page", l + 1); let i = "https://www.geoguessr.com/api/v4/feed/private"; "" !== p && (i += "?paginationToken=" + p); let u = await (await fetch(i)).json(); if (0 === u.entries.length) { console.log("All data fetched."); break } if (s.push(...o(JSON.stringify(u))), s.includes(t)) break; p = a(u.entries[u.entries.length - 1].time.substring(0, 23) + "Z"), await n(() => console.log("Done"), 500) } let d = s.filter((e, t, n) => n.indexOf(e) === t); return d.includes(t) ? d.slice(0, d.indexOf(t)) : d } let p = await r(1e3, t), s = {}; async function l(t) { let n = {}; n.id = t.gameId, n.rounds = t.currentRoundNumber, n.startDate = new Date(t.rounds[0].startTime).toLocaleString("en-US"), n.endDate = new Date(t.rounds[n.rounds - 1].endTime).toLocaleString("en-US"), n.mode = t.options.competitiveGameMode; for (let o = 0; o < 2; o++) { let a = t.teams[o]; if (a.players[0].playerId === e) { if (n.selfHp = a.health, null === a.players[0].progressChange) n.befElo = a.players[0].rating, n.aftElo = n.befElo; else { let r = a.players[0].progressChange.competitiveProgress; null === r ? (n.befElo = a.players[0].rating, n.aftElo = n.befElo) : (n.befElo = r.ratingBefore, n.aftElo = r.ratingAfter) } [n.selfDist, n.selfTtg, n.selfCountries] = i(a.players[0].guesses, t.rounds, a.roundResults, t.teams[1 - o].roundResults) } else { if (n.oppId = a.players[0].playerId, n.oppHp = a.health, n.oppElo = a.players[0].rating, [n.oppDist, n.oppTtg, _nil] = i(a.players[0].guesses, t.rounds), !s[n.oppId]) { let p = `https://www.geoguessr.com/api/v3/users/${n.oppId}`; try { let l = await fetch(p, { credentials: "include" }), u = await l.json(); s[n.oppId] = u } catch (d) { if (d.response.status >= 400) { n.oppName = "Deleted User", n.oppCountry = "", n.oppBanned = !0, n.oppBlueCheck = !1, n.oppCreator = !1; continue } } } let c = s[n.oppId]; n.oppName = c.nick, n.oppCountry = c.countryCode, n.oppBanned = c.isBanned, n.oppBluecheck = (2 & c.flair) != 0, n.oppCreator = c.isCreator } } return n } function i(e, t, n = null, o = null) { let a = 0, r = 0, p = 0, s = {}; for (let l of e) { let i = l.roundNumber - 1, u = (new Date(l.created) - new Date(t[i].startTime)) / 1e3; if (p++, a += l.distance, r += u, !n) continue; let d = t[i].panorama?.countryCode; "" !== d && (d in s || (s[d] = [0, 0, 0, 0, 0, 0]), s[d][0]++, s[d][1] += l.distance, s[d][2] += u, n[i].healthAfter >= n[i].healthBefore ? s[d][3]++ : s[d][4] += o[i].score - n[i].score, s[d][5] += n[i].score - o[i].score) } return 0 === p ? ["", ""] : [a / p, r / p, Object.entries(s).map(e => e[0] + "," + e[1].join(",")).join(";")] } async function u(e) { let t = [], o = 0; for (let a of e) { console.log(`Fetching duel #${o++} / ${e.length}`); let r = await fetch(`https://game-server.geoguessr.com/api/duels/${a}`, { credentials: "include" }); r = await r.json(); let p = await l(r); t.push(p), await n(() => null, 150) } return t } let d = { id: "ID", rounds: "# Rounds", startDate: "Start Date", endDate: "End Date", selfHp: "My Health", befElo: "Start ELO", aftElo: "End ELO", selfDist: "Avg Distance", selfTtg: "Avg TTG", oppId: "Opp ID", oppHp: "Opp Health", oppElo: "Opp ELO", oppDist: "Opp Distance", oppTtg: "Opp TTG", selfCountries: "Self Countries", mode: "Game Mode", oppName: "Opp Name", oppCountry: "Opp Country", oppBanned: "Opp Banned", oppBluecheck: "Opp Bluecheck", oppCreator: "Opp Creator" }, c = await u(p), f = function e(t, n = "	", o = d) { let a = ""; for (let r of t = [...t]) { for (let p in o) a += r[p] + n; a += "\n" } return a }(c); return f }
await main(selfId, lastId);
```

8. Press `enter` to submit the code. Wait for it to retrieve the data. Make sure to keep the page open so it runs faster. It might take a few minutes to run depending on how many games you have. Once it is done running, you should see a big block of text:

![Data output](images/data_output.png)

9. `Right click` on the block of text and select "Copy string contents":

![Copy data](images/copy_string.png)

10. You can now paste this data into any spreadsheet!

## Phase 2: Data Visualization

To visualize the data you just copied at the end of phase 2, you can make your own spreadsheet or use the one I have created. By default, the spreadsheet calculates stats up to 5000 duels. If you want to include more, you might need to edit the formulas and extend the calculations sheet. To use my spreadsheet template:

1. Go to [the Google Sheets document](https://docs.google.com/spreadsheets/d/1TNY27d5CZUdjiE7bWMnCSq1uu630liZE94xA0q9UgDE/edit?usp=sharing)

2. Under `File`, click "Make a copy".

3. In your copy of the spreadsheet, paste the data as you copied it into the cell `A1` on the sheet named `PASTE HERE`. If needed, you can navigate to the sheet from the tabs at the bottom-left.

![A1 Cell](images/A1.png)

4. Navigate to the `Data` sheet using the tabs at the bottom to view your processed data and graphs.

***Note:*** the spreadsheet may take a few seconds to calculate the stats if you add/remove/change a large amount of data. Please wait until the blue bar at the top-right of the page finishes loading. If you don't see this, then you should be good to go!

![Loading](images/loading.png)


## Phase 3: Updating your stats
If you want to update your stats after playing more duels, you can do so very easily.

**Note:** If your spreadsheet version is before `v1.5`, you'll need to re-copy the spreadsheet and paste your data into cell `A1` on the sheet named `PASTE HERE`.

1. From your spreadsheet, copy the ID of your most recent duels match. If you're using my spreadsheet, it should be cell `A2` of the `CALC` sheet.
2. Follow Phase 1 and 2 from above like before. However, in step 1 add your most recent duels ID to `const lastId = "-->HERE<--";`

![Updated Script](images/update.png)

3. Complete the rest of Phase 1 like before.
4. Once done, you can paste your raw stats string into column 1 of any row of the spreadsheet (although it might be best for you to paste it right after your old stats).

![Paste New Stats](images/updatepaste.png)

5. Your stats should automatically update. Enjoy!
