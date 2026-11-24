---
id: iwe3pjm84k
title: "Trading Dashboard"
sort: 2000
version: 1
---

The live view of the journal: how the account is doing, where the edge is (or isn't), and the rules every trade is checked against. For a written read of each period, see the reviews below.

## 📚 Reviews

| Period | Trades | Win rate | Net P&L | Profit factor | Max drawdown | Read |
| --- | --: | --: | --: | --: | --: | --- |
| Oct–Nov 2026 (latest) | 21 | 42\.9% | \+$833.79 (+83.4%) | 1\.96 | −$448.28 (−38.5%) | [Trading Review: Oct–Nov 2026](page:i75b72gcry) |
| Jun–Jul 2026 | 21 | 52\.4% | \+$644.51 (+64.5%) | 2\.96 | −$184.36 | [Trading Review: Jun–Jul 2026](page:fslq6pq8ew) |

<callout icon="📌">

**Where things stand (24 Nov 2026):** the edge now holds up **without lot size** (+78 points, winners 2.9× the size of losers), unlike Jun–Jul (−0.83R). The two things to fix next: **lot size is still too big** (losses of 5–14% of the balance), and **one trade (T21) is 54% of the profit**. Next review at 50 trades.

</callout>

---

## 🆕 Latest period: Oct–Nov 2026

From [XAUUSD\_Trading\_Journal\_Oct-Nov\_2026](page:2zt8nsi3ou), 19 Oct to 18 Nov, balance **$1,000 → $1,833.79**.

<columns>

<column width=0.35>

### 🎯 Win / loss

<database-chart by="u5y24f1ciz" chart="donut" colors="color" db="dpibq1upm0"/>

</column>

<column width=0.65>

### 📈 Equity curve

<database-chart by="title" chart="area" db="dpibq1upm0" measure="sum" show="running" value="4813xy313n"/>

</column>

</columns>

<columns>

<column>

### Points per trade (lot size taken out)

<database-chart by="title" chart="bar" db="dpibq1upm0" measure="sum" value="z70n8q8jwu"/>

</column>

<column>

### Result as % of balance

<database-chart by="title" chart="bar" db="dpibq1upm0" measure="sum" value="b3vb32rhb2"/>

</column>

</columns>

---

## 🗂️ Jun–Jul 2026 in detail

<callout background="gray" icon="ℹ️">

The charts and trade log from here down read the Jun–Jul trades (T01–T21) in [forex trading journal](page:p6h3o0i29k). They stay on that data because the Oct–Nov export has **no stop loss, planned R:R, setup or emotion columns**, so the risk and psychology charts would be empty. Add those columns to the next export and these blocks can be pointed at it.

</callout>

<columns>

<column width=0.442>

### 🎯 Win / loss

<database-chart by="r3kuahvfwz" chart="donut" colors="color" db="fihhki519f"/>

</column>

<column width=0.2584>

### 💰 P&L by outcome

<database-chart by="r3kuahvfwz" chart="bars-across" colors="color" db="fihhki519f" measure="sum" value="4nd7skqkkd"/>

</column>

<column width=0.2996>

### ⚖️ Avg P&L per trade

<database-chart by="r3kuahvfwz" chart="bars-across" colors="color" db="fihhki519f" measure="average" value="4nd7skqkkd"/>

</column>

</columns>

---

## 📈 Equity curve

<database-chart by="title" chart="area" db="fihhki519f" measure="sum" show="running" value="4nd7skqkkd"/>

Running total of P&L, trade by trade. A healthy curve climbs in steps, not in one or two jumps.

<database-chart by="title" chart="bar" db="fihhki519f" measure="sum" value="4nd7skqkkd"/>

---

## 🛡️ Risk and discipline

<columns>

<column>

### Risk per trade (% of account)

<database-chart by="title" chart="bar" db="fihhki519f" measure="sum" value="tn04ylbxig"/>

</column>

<column>

### Planned R:R

<database-chart by="title" chart="bar" db="fihhki519f" measure="sum" value="2w3isrsf8j"/>

</column>

</columns>

<columns>

<column>

### Lot size vs P&L

<database-chart chart="scatter" color="r3kuahvfwz" db="fihhki519f" x="lo1awo3ups" y="4nd7skqkkd"/>

</column>

<column>

**What to look for**

- Risk bars should be **flat**: same % every trade.
- Planned R:R should sit **at 1.5 or above**.
- In the scatter, big wins and big losses clustering at the largest lots means **size is driving the result**, not the setup.

</column>

</columns>

---

## 🕒 Sessions, direction and days

<columns>

<column>

### P&L by session

<database-chart by="4dob1o2isa" chart="bars-across" db="fihhki519f" measure="sum" value="4nd7skqkkd"/>

</column>

<column>

### Wins and losses by session

<database-chart by="4dob1o2isa" chart="stacked" colors="color" db="fihhki519f" series="r3kuahvfwz"/>

</column>

</columns>

<columns>

<column>

### Buy vs sell

<database-chart by="v8fs9hhtbg" chart="grouped" colors="color" db="fihhki519f" series="r3kuahvfwz"/>

</column>

<column>

### Daily P&L

<database-chart chart="heatmap" date="3hulenehev" db="fihhki519f" measure="sum" value="4nd7skqkkd"/>

</column>

</columns>

---

## 🧠 Setups and psychology

<columns>

<column>

### P&L by setup

<database-chart by="73xcnzdrc1" chart="bars-across" db="fihhki519f" measure="sum" value="4nd7skqkkd"/>

</column>

<column>

### Outcome by emotion

<database-chart by="zmyr37ik9b" chart="stacked" colors="color" db="fihhki519f" series="r3kuahvfwz"/>

</column>

</columns>

<callout icon="💡">

These two charts are only as good as the logging: **Setup / Pattern** and **Emotion** are mostly empty in the dummy data. Fill them on every trade and this section starts answering *which setups pay*.

</callout>

---

## 📒 Trade log

<database-view db="fihhki519f"/>

---

## ✅ Pre-trade checklist

Run through before every entry. If one box can't be ticked, no trade.

- [ ] Higher-timeframe trend agrees with the direction
- [ ] Clear setup, named in **Setup / Pattern**
- [ ] Stop placed at a level, not at a dollar amount
- [ ] Lot size worked out for **1–2% risk** from the stop distance
- [ ] Planned R:R **≥ 1.5**
- [ ] No high-impact news in the next 30 minutes
- [ ] Fewer than 2 losses today, and above the −3% daily limit
- [ ] Calm: not chasing the last trade

## 📏 Rules

<columns>

<column>

### Risk

- Max **2%** per trade, never above 5%
- Daily stop: **2 losses or −3%**
- Weekly stop: **−6%**

</column>

<column>

### Management

- Let winners reach target, or move stop to **break-even**; don't close early by hand
- No moving the stop further away
- No revenge trades after a loss

</column>

</columns>

---

## 🗓️ Weekly review

<toggle>

Week of TBD

- **Trades / wins / P&L:** TBD
- **Best trade and why:** TBD
- **Worst trade and why:** TBD
- **Rules broken:** TBD
- **One thing to change next week:** TBD

</toggle>

---

## 📥 Before the next import

- [ ] Delete empty rows at the bottom of the sheet (rows 22–50 in the dummy data)
- [ ] Add **Risk ($)** and **R multiple** columns, so results can be read in R as well as dollars
- [ ] Fix **Pips (+/−)**: it's about 10,000× too big in the current sheet
- [ ] Rename or drop the unnamed **Column 22–24**
- [ ] Fill **Setup / Pattern** and **Emotion** for every trade
- [ ] Import once only (the last import made three copies of the side sheets)
- [ ] Point this page's charts and trade log at the new database

**Learned from the Oct–Nov import:**

- [x] Oct–Nov journal imported once (Trade Log, Summary, Weekly) and reviewed
- [ ] Add **Stop loss**, **Take profit** and **Open time** columns to the MT5 export, so R, planned R:R and hold time can be worked out
- [ ] Keep explanation notes **outside** the data range (they came in as rows of Trade Log, Weekly and Summary)
- [ ] Fill **Setup / reason**, **Followed plan?** and **Notes** before importing
