---
id: fslq6pq8ew
title: "Trading Review: Jun–Jul 2026"
sort: 1000
version: 1
---

A read of the first **21 XAUUSD trades** in [forex trading journal](page:p6h3o0i29k) (5 June to 2 July 2026, $1,000 starting account), with a chart for each finding. The trades are copied below with a few worked-out columns (risk, R multiple, how each trade ended), so the charts read from clean numbers.

<callout icon="📌">

**The verdict:** the account is up **+$644.51 (+64.5%)**, with a profit factor of **2.96**. But measured in risk (R), the same 21 trades come to **−0.83R**: slightly negative. The profit came from **position size**, mostly two big trades (T13, T14), not from an edge in the setups. Fix the sizing and let winners run before reading too much into the dollar result.

</callout>

<columns>

<column>

### 💰 Result

**+$644.51** net

**11 wins / 10 losses** (52.4%)

**Expectancy:** +$30.69 per trade

</column>

<column>

### ⚖️ Wins vs losses

**Average win:** +$88.49 (**0.77R**)

**Average loss:** −$32.89 (**0.93R**, stops respected)

**Profit factor:** 2.96

</column>

<column>

### ⚠️ Risk

**Largest risk:** **65.5%** of the account on one trade (T14)

**Median risk:** 5.6% per trade

**Max drawdown:** −$184.36

</column>

</columns>

---

## 1\. The equity curve: up, but in two jumps

<database-chart by="title" chart="area" db="iyxpol1fsp" measure="sum" show="running" value="n3ad69f6cw"/>

The account went **below the start** after the losing run on 7 June (−$106 by T06), recovered slowly, then jumped on **T13 (+$215)** and **T14 (+$211)**. Those two trades are **66% of the total profit**: without them the result is +$218. After the peak (+$536 at T15) came the deepest drawdown, **−$184** to T18.

<database-chart by="title" chart="bar" colors="color" db="iyxpol1fsp" measure="sum" value="n3ad69f6cw"/>

---

## 2\. In risk terms, the edge isn't there yet

Dollars depend on lot size; **R** (profit divided by what the trade risked) shows the quality of the trade itself. +1R means you made what you risked.

<database-chart by="title" chart="bar" db="iyxpol1fsp" measure="sum" value="qs25b202s9"/>

<columns>

<column>

<database-chart bins="6" chart="histogram" db="iyxpol1fsp" of="qs25b202s9"/>

</column>

<column>

**How R is spread:**

- **9 of the 10 losses hit the stop, at about −1R.** Stops are set and respected. That's real discipline.
- **Wins average only 0.77R**, and 7 of 11 wins are under 0.7R.
- With wins at 0.77R and losses at 0.93R, you need a **win rate above \~55%** to break even in R. You're at 52%.

</column>

</columns>

---

## 3\. Winners are cut short, often by plan

<database-chart by="title" chart="bar" db="iyxpol1fsp" measure="sum" values="3ppradbnri,qs25b202s9"/>

Planned R:R (first bar) against the R actually achieved (second):

- **7 trades were planned with a reward smaller than the risk** (planned R:R below 1): T02, T07, T08, T09, T10, T12, T15. T15 risked $274 to make $0.32: probably a take-profit set by mistake.
- **6 wins were closed by hand**, four of them at 0.3R or less (T07, T08, T14, T20), well short of their targets.

<database-chart by="yyml2a6wmh" chart="stacked" colors="color" db="iyxpol1fsp" series="t3krefpx0d"/>

---

## 4\. Position size is all over the place

<database-chart by="title" chart="bar" db="iyxpol1fsp" measure="sum" value="mw0iqmyx3q"/>

Risk per trade ranges from **0.7% to 65.5%** of the account. **T14 risked $868 on a $1,325 account** (0.5 lots, stop 17.4 points away). It won, but the same trade losing would have erased most of the account.

<database-chart chart="scatter" color="t3krefpx0d" db="iyxpol1fsp" x="mw0iqmyx3q" y="n3ad69f6cw"/>

The biggest dollar results sit where risk was highest. That is luck amplified by size, not skill being rewarded, and it works the same way in reverse.

---

## 5\. Sessions, days and direction

<columns>

<column>

### P&L by session

<database-chart by="9ni0r1jggz" chart="bars-across" db="iyxpol1fsp" measure="sum" sort="desc" value="n3ad69f6cw"/>

</column>

<column>

### Wins and losses by session

<database-chart by="9ni0r1jggz" chart="stacked" colors="color" db="iyxpol1fsp" series="t3krefpx0d"/>

</column>

</columns>

| Session | Trades | Wins | P&L | In R |
| --- | --: | --: | --: | --: |
| London | 14 | 6 (43%) | \+$384.93 | −3.32R |
| NY | 3 | 3 (100%) | \+$211.14 | \+2.60R |
| NY/London overlap | 4 | 2 (50%) | \+$48.43 | −0.11R |

- **London** is where you trade most and where most losses are; its dollar profit is T13 and T14. In R it's negative.
- **NY** is 3 for 3, but three trades is far too few to call it an edge. Worth watching.

<database-chart by="9ni0r1jggz" chart="box" db="iyxpol1fsp" of="n3ad69f6cw"/>

<columns>

<column>

### Buy vs sell

<database-chart by="h3rf402rh9" chart="grouped" colors="color" db="iyxpol1fsp" series="t3krefpx0d"/>

</column>

<column>

### Daily P&L

<database-chart chart="heatmap" date="d8ptm1q7to" db="iyxpol1fsp" measure="sum" value="n3ad69f6cw"/>

</column>

</columns>

- **No direction bias:** buys +$285 (5 of 9 won, +0.52R), sells +$359 (6 of 12, −1.35R).
- **7 June was the worst day:** six trades, the first four losers in a row (−$121) inside two and a half hours. A daily loss limit would have stopped the day after two or three.

---

## 6\. Problems in the journal itself

<callout background="gray" icon="🧾">

Fix these and the next review gets much sharper:

- **Pips (+/−) is wrong:** values like −104,490 are about 10,000× too big, and two trades have none. P&L is right: it matches entry, exit and lot size to the cent.
- **Win Rate shows 0** on the Statistics sheet (it's 52.4%), and so does Avg % Risk. These are formulas that didn't survive the import from the spreadsheet.
- **Rows 22–50 are empty**, which skews anything that counts rows.
- **Dates:** T03–T08 are dated 7 June 2026, a Sunday, when the gold market is closed during those hours. Check the date.
- **Setup / Pattern is empty on every trade**, and Emotion is filled for only 4. Without them, the Setup and Psychology tables can't say anything.
- The journal's sheets were imported **three times**: there are three copies of Statistics, Trading Sessions, Timeframe Guide, Pre-Trade Checklist and Instructions.

</callout>

## 7\. What to try next

- [ ] **Fixed risk per trade:** 1–2% of the account, with lot size worked out from the stop distance. No trade above 5%.
- [ ] **Only take trades planned at 1.5R or better**, and let them reach the target, or move the stop to break-even instead of closing early.
- [ ] **Daily stop:** after 2 losses, or −3% in a day, stop trading for the day.
- [ ] **Log Setup and Emotion on every trade**, so the next review can say which setups work.
- [ ] **Review again at 50 trades**, in R as well as dollars. 21 trades is a small sample for any conclusion.

<toggle>

The trades behind the charts

A copy of T01–T21 as they stood on 20 October 2026, with worked-out columns: Risk ($) is the stop distance × lot × 100 oz; Risk (% of account) is against the balance before the trade; R multiple is P&L ÷ Risk ($). New trades in the journal don't appear here until it's refreshed.

<database-view db="iyxpol1fsp" view="pkq0kuzes5"/>

</toggle>

<callout background="gray" icon="ℹ️">

This is a reading of your own numbers, not financial advice.

</callout>
