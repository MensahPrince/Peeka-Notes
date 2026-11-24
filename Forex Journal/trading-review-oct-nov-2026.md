---
id: i75b72gcry
title: "Trading Review: Oct–Nov 2026"
sort: 3000
version: 1
---

A read of the **21 XAUUSD trades** in [XAUUSD\_Trading\_Journal\_Oct-Nov\_2026](page:2zt8nsi3ou) (19 October to 18 November 2026, $1,000 starting deposit), with a chart for each finding. The trades are copied below with a few worked-out columns (session, weekday, trade of the day, what came before), so the charts read from clean numbers. For the previous period, see [Trading Review: Jun–Jul 2026](page:fslq6pq8ew).

<callout icon="📌">

**The verdict:** the account is up **+$833.79 (+83.4%)**, with a profit factor of **1.96**. This time the result is **not just size**: measured in points (which ignores lot size), the 21 trades made **+78 points**, and winners ran **2.9× as far as losers** (16.1 vs 5.6 points). That's a real improvement on Jun–Jul. The two weak spots: **one trade (T21, +$450) is 54% of the profit**, and **lots of 0.15 on a $1k account** turned four losses on 23 October into a **−38.5% drawdown**.

</callout>

<columns>

<column>

### 💰 Result

**+$833.79** net, balance **$1,833.79**

**9 wins / 12 losses** (42.9%)

**Expectancy:** +$39.70 per trade

</column>

<column>

### ⚖️ Wins vs losses

**Average win:** +$189.08 (**16.1 points**)

**Average loss:** −$72.33 (**5.6 points**)

**Payoff ratio:** 2.61, so break-even needs only a **28% win rate**

</column>

<column>

### ⚠️ Risk

**Max drawdown:** **−$448.28 (−38.5%)**, all on 23 October

**Median loss:** 6.8% of the balance

**Largest loss:** 14.0% of the balance (T09)

</column>

</columns>

---

## 1\. The equity curve: a deep hole, then a climb

<database-chart by="title" chart="area" db="dpibq1upm0" measure="sum" show="running" value="4813xy313n"/>

The account dropped to **$715.82 by T09** (−28% from the deposit, −38.5% from the peak), recovered fully by T12 with three buys in a row, gave back $200 on 29–30 October, then made new highs on **T18–T19** and again on **T21**.

<database-chart by="title" chart="bar" colors="color" db="dpibq1upm0" measure="sum" value="4813xy313n"/>

<database-chart by="title" chart="line" db="dpibq1upm0" measure="sum" value="ea46iy2t4q"/>

Drawdown from the peak, trade by trade. The account spent **T06 to T17 under water**: 12 of 21 trades.

---

## 2\. The edge, without the lot size

Dollars depend on lot size. **Points** (the price move in your favour) show the trade itself, so they compare fairly across 0.10 and 0.15 lots.

<database-chart by="title" chart="bar" db="dpibq1upm0" measure="sum" value="z70n8q8jwu"/>

<columns>

<column>

<database-chart bins="8" chart="histogram" db="dpibq1upm0" of="z70n8q8jwu"/>

</column>

<column>

**How points are spread:**

- **Losses are tight and alike:** 10 of 12 lost between 3.5 and 9.7 points. Stops are being placed and respected.
- **Winners are let run:** five wins went past 10 points, and T21 made **45 points**.
- At a fixed **0.10 lots** every trade, these 21 trades would have made **+$780.80**: nearly the same result with far less risk.

</column>

</columns>

<callout background="yellow" icon="⚠️">

**Concentration:** T21 (+$450.45), T05 (+$275.73) and T18 (+$242.29) together made **$968**, more than the whole profit. Without those three, the other 18 trades are **−$134.68** (about −10 points). The edge depends on catching the occasional long run, so **never cutting those winners early matters more than anything else**.

</callout>

---

## 3\. Lot size: cutting to 0.10 was the right call

<columns>

<column>

### P&L by lot size

<database-chart by="9h1yluz8aw" chart="bars-across" db="dpibq1upm0" measure="sum" value="4813xy313n"/>

</column>

<column>

### Wins and losses by lot size

<database-chart by="9h1yluz8aw" chart="stacked" colors="color" db="dpibq1upm0" series="u5y24f1ciz"/>

</column>

</columns>

| Lot size | Trades | Wins | P&L | Points |
| --- | --: | --: | --: | --: |
| 0\.15 lots (T02–T12) | 11 | 5 | \+$158.98 | \+10.6 |
| 0\.10 lots (T01, T13–T21) | 10 | 4 | \+$674.81 | \+67.5 |

At **0.15 lots, every point is $15**, so a normal 7-point stop costs about **$105: 10–14% of a $1k account**. From T13 on you traded 0.10 lots, and results improved. At today's $1,834 balance, a 6-point loss at 0.10 lots is about **3.3%**: better, but still above the 2% rule on the [Trading Dashboard](page:iwe3pjm84k).

<database-chart by="title" chart="bar" db="dpibq1upm0" measure="sum" value="b3vb32rhb2"/>

Each trade's result as a % of the balance before it. Losses should be **short, flat bars of the same height**; at 0.15 lots they were 9–14%.

---

## 4\. 23 October: the day that made the drawdown

<callout background="gray" icon="🧾">

**T06 to T09, four sells in two hours, all losses: −$448.28.**

- T06 lost at 14:43. **T07 and T08 were two positions opened together** and closed in the same minute at 16:08: a double-sized bet straight after a loss.
- **T09 followed 38 minutes later**, same direction, same result.
- A **daily stop of 2 losses** would have ended the day after T07 at about −$216, saving $232.

</callout>

<columns>

<column>

### Outcome by what came before

<database-chart by="5717pq1qq4" chart="stacked" colors="color" db="dpibq1upm0" series="u5y24f1ciz"/>

</column>

<column>

### Daily P&L

<database-chart chart="heatmap" date="5agqlttkkb" db="dpibq1upm0" measure="sum" value="4813xy313n"/>

</column>

</columns>

- Trading right after a loss isn't bad in itself here (5 of 12 won, +$481), **but stacking size or flipping direction after a loss is**.
- **Flip trades:** T13 (sell) then T14 (buy) two minutes later: both lost, −$148. T16 (buy) then T17 (sell) five minutes later: this one won. Flipping on the minute is a sign of reacting, not planning.
- **Busy days are mixed:** 23 October (4 trades, −$448) and 30 October (5 trades, +$435). More trades in a day doesn't help on its own.

---

## 5\. Direction, sessions and days

<columns>

<column>

### Buy vs sell

<database-chart by="p9bhxkimim" chart="grouped" colors="color" db="dpibq1upm0" series="u5y24f1ciz"/>

</column>

<column>

### P&L by direction

<database-chart by="p9bhxkimim" chart="bars-across" db="dpibq1upm0" measure="sum" value="4813xy313n"/>

</column>

</columns>

- **Sells:** 14 trades, 6 won, **+$560.23** (+65 points). **Buys:** 7 trades, 3 won, **+$273.56** (+13 points).
- Gold fell from about **4,175 to 3,820** over the period, so selling was trading with the trend. The three buys that won (T10–T12) caught the bounce on 27–28 October.

<columns>

<column>

### P&L by session

<database-chart by="fqkyvzt2x3" chart="bars-across" db="dpibq1upm0" measure="sum" sort="desc" value="4813xy313n"/>

</column>

<column>

### Wins and losses by session

<database-chart by="fqkyvzt2x3" chart="stacked" colors="color" db="dpibq1upm0" series="u5y24f1ciz"/>

</column>

</columns>

| Session (est.) | Trades | Wins | P&L | Points |
| --- | --: | --: | --: | --: |
| London | 12 | 4 (33%) | \+$136.16 | \+17.0 |
| Asia | 4 | 3 (75%) | \+$628.62 | \+56.5 |
| London/NY overlap | 5 | 2 (40%) | \+$69.01 | \+4.6 |

- **London is still where you trade most and win least.** Its profit is all T18 and T19.
- **Asia** looks strong, but $450 of it is T21. Four trades is far too few to call it an edge.

<columns>

<column>

### P&L by weekday

<database-chart by="do8uwmzxwn" chart="bar" db="dpibq1upm0" measure="sum" value="4813xy313n"/>

</column>

<column>

### Trades by weekday

<database-chart by="do8uwmzxwn" chart="stacked" colors="color" db="dpibq1upm0" series="u5y24f1ciz"/>

</column>

</columns>

**Fridays:** 9 of 21 trades, 3 won, **−$12.98** net. Most of the activity, none of the profit. Tuesday and Wednesday made $831 from 5 trades.

<callout background="gray" icon="ℹ️">

Sessions are **estimated from the close time** (the export has no open time), assuming broker time is GMT+3 until 25 October and GMT+2 after, the usual MT5 setting. A trade opened in London and closed in New York counts as New York here.

</callout>

---

## 6\. Compared with Jun–Jul

|  | Jun–Jul 2026 | Oct–Nov 2026 |
| --- | --: | --: |
| Trades | 21 | 21 |
| Win rate | 52\.4% | 42\.9% |
| Net P&L | \+$644.51 (+64.5%) | \+$833.79 (+83.4%) |
| Profit factor | 2\.96 | 1\.96 |
| Average win | \+$88.49 | \+$189.08 |
| Average loss | −$32.89 | −$72.33 |
| Payoff (avg win ÷ avg loss) | 2\.69 | 2\.61 |
| Max drawdown | −$184.36 | −$448.28 (−38.5%) |
| Profit from top 2 trades | 66% | 87% |
| Lot sizes | 0\.02 to 0.5 | 0\.10 or 0.15 |
| Edge without lot size | −0.83R (negative) | \+78 points (positive) |

- **Better:** lot sizes are consistent, winners run further than losers, and the result holds up with lot size taken out.
- **Worse:** the drawdown is more than twice as deep, and the profit leans even harder on one or two trades.

---

## 7\. Problems in the journal itself

<callout background="gray" icon="🧾">

Fix these and the next review gets much sharper:

- **Setup / reason, Followed plan? and Notes are empty on every trade.** Without them, nothing here can say *which setups* work.
- **No stop loss, take profit or open time** in the export, so risk in R, planned R:R and hold time can't be worked out. Add **SL** and **Open time** columns.
- **Note rows inside the tables:** the Trade Log and Weekly sheets each have explanation text as their last rows, and the Summary sheet's first row became its column names (*Starting balance / 1000*). Keep notes outside the data range before importing.
- **Drawdown %** is stored as a fraction (−0.385), not a percent.
- The **25 October correction** line in the MT5 history is left out (no amount); the total still matches MT5's profit of $833.79.

</callout>

## 8\. What to try next

- [ ] **Lot size from the stop, not by habit:** lots = (1–2% of the balance) ÷ (stop in points × $100). At $1,834 and a 6-point stop, that's **0.03–0.06 lots**.
- [ ] **Never open two positions together after a loss** (T07 + T08).
- [ ] **Daily stop:** after 2 losses, or −3% in a day, done for the day.
- [ ] **No flipping direction within 15 minutes of a loss** (T13 → T14).
- [ ] **Keep letting winners run.** The whole profit comes from the 3 longest runs; one early close there would have wiped out the month.
- [ ] **Go easy on Fridays:** fewer, better trades.
- [ ] **Fill Setup / reason, Followed plan? and Notes on every trade**, and log the stop.
- [ ] **Review again at 50 trades** across both periods: 21 trades is still a small sample.

<toggle>

The trades behind the charts

A copy of T01–T21 as they stood on 24 November 2026, with worked-out columns: **% of balance** is P/L against the balance before the trade; **Drawdown %** is from the running peak; **Trade of the day** counts trades closed that day; **Came after** is the previous trade's result; **Session (est.)** is from the close time in UTC. New trades in the journal don't appear here until it's refreshed.

<database-view db="dpibq1upm0"/>

</toggle>

<callout background="gray" icon="ℹ️">

This is a reading of your own numbers, not financial advice.

</callout>
