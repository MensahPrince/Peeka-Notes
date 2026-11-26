---
id: fdwo4levxa
title: "Growth Journey"
sort: 2000
version: 1
---

<callout background="blue" icon="📌">

<b>One row a day in the </b>[**Growth Log**](page:o9mr418pfx)**.** Fill it in through the day. Automations tick habits, update the Status and tell you when the day is done. A perfect day is **8/8**.

</callout>

<columns>

<column>

### The 8 daily wins

- [ ] 🏋️ **Gym**: 30+ min (auto-ticks from *Gym min*)
- [ ] 📖 **Reading**: 10+ pages (auto from *Pages*)
- [ ] ✍️ **Writing**: 300+ words. Click the day's title to open its page and write there; *Words* counts them as you type and ticks Writing at 300.
- [ ] 💻 **Project Dev**: 1+ hour (auto from *Dev hours*)
- [ ] 🛡️ **SR**: held today
- [ ] 🍬 **Sugar ≤ 36 g** (counts only once you log *Sugar g*)
- [ ] 😴 **Sleep ≥ 7 h**
- [ ] 💧 **Water ≥ 2.5 L**

</column>

<column>

### What the automations do

- **Numbers tick the box**: log 45 *Gym min* and *Gym* ticks itself. The same goes for Pages, Words and Dev hours.
- **Day started**: the first win moves Status to *In progress*.
- **Day complete**: 8/8 sets Status to *Complete* and notifies you, with your *Tomorrow's focus*.
- **Sugar over limit**: more than 36 g raises an alert.
- **SR milestone**: a notification every 7 days of *SR streak*.
- **Low sleep**: under 6 h sends a warning.

</column>

</columns>

<callout background="yellow" icon="⚠️">

**New days add themselves:** when a day reaches **8/8**, the *Add tomorrow's row* automation adds the next day, dated and set to *Not started*. This needs a Peeka built on Blockwright `e1a1e7b` or later. The log is pre-filled to November 30, so the chain starts on its own from there. If you miss a day, add the next row by hand (the **+** under the table): give it the date as its title, like *December 3, 2026*, and set its Date.

</callout>

## Today

<database-view db="c5x394veib" view="ku4b3zll3f"/>

## Overall growth

### Daily score (%)

<database-chart by="x6s7mffoqy" chart="line" db="c5x394veib" measure="sum" smooth=true unit="day" value="nxm07knq49"/>

### Consistency heatmap

<database-chart chart="heatmap" date="x6s7mffoqy" db="c5x394veib" measure="sum" value="1x1bklc5mr"/>

### Habits hit per week

<database-chart by="x6s7mffoqy" chart="grouped" colors="color" db="c5x394veib" measure="sum" unit="week" values="xh5uk5hx87,o2c28427uq,x3nacw9d3u,yam9yzfeic,1llx71vinb"/>

<columns>

<column width=0.562>

### Days by status

<database-chart by="imo6gzbfp7" chart="donut" colors="color" db="c5x394veib"/>

</column>

<column width=0.438>

### Mood and energy

<database-chart by="x6s7mffoqy" chart="line" db="c5x394veib" measure="average" smooth=true unit="day" values="xkoekvo7cl,v48ky07uu9"/>

</column>

</columns>

## 🏋️ Body

<columns>

<column width=0.4883>

### Gym minutes per week

<database-chart by="x6s7mffoqy" chart="bar" db="c5x394veib" measure="sum" unit="week" value="hqigvhlmve"/>

</column>

<column width=0.5117>

### Training split

<database-chart by="fmjty1shpb" chart="donut" colors="color" db="c5x394veib"/>

</column>

</columns>

### Weight (kg, weekly average)

<database-chart by="x6s7mffoqy" chart="line" db="c5x394veib" measure="average" smooth=true unit="week" value="ykqa3yjfsy"/>

<columns>

<column>

### Sleep (h)

<database-chart by="x6s7mffoqy" chart="area" db="c5x394veib" measure="sum" unit="day" value="1yhp7nqms6"/>

</column>

<column>

### Sleep vs energy

<database-chart chart="scatter" db="c5x394veib" x="1yhp7nqms6" y="v48ky07uu9"/>

</column>

</columns>

### Water (L) and steps

<database-chart by="x6s7mffoqy" chart="line" db="c5x394veib" measure="sum" right="k0aqptvo0p" unit="day" values="j5i5ogwxmj,k0aqptvo0p"/>

## 🧠 Mind and craft

### Pages read, total so far

<database-chart by="x6s7mffoqy" chart="area" db="c5x394veib" measure="sum" show="running" unit="day" value="ufcfl98cgq"/>

### Words written, total so far

<database-chart by="x6s7mffoqy" chart="area" db="c5x394veib" measure="sum" show="running" unit="day" value="8rmvac4713"/>

<columns>

<column>

### Dev hours per week

<database-chart by="x6s7mffoqy" chart="bar" db="c5x394veib" measure="sum" unit="week" value="udo516tqjw"/>

</column>

<column>

### Hours by project

<database-chart by="mime9ngyoa" chart="bars-across" colors="color" db="c5x394veib" measure="sum" sort="desc" value="udo516tqjw"/>

</column>

</columns>

### Screen time (h)

<database-chart by="x6s7mffoqy" chart="bar" db="c5x394veib" measure="sum" unit="day" value="ij73exk00d"/>

## 🛡️ SR

### SR streak

<database-chart by="x6s7mffoqy" chart="line" db="c5x394veib" measure="max" unit="day" value="6o1ljrnk2i"/>

<callout icon="💡">

Each day, type the streak count into **SR streak** (yesterday + 1, or 0 after a reset). A formula can't read the row before, so you set this number by hand. Every 7th day you get a notification.

</callout>

## 🍬 Sugar

<callout background="pink" icon="🍬">

**Daily limit: 36 g added sugar** (the AHA maximum for men; 25 g is ideal). One can of soda is about 35–39 g, so a single can uses up the whole day's limit. Log grams from labels in **Sugar g**, and tick where it came from in **Sugar sources**.

</callout>

### Sugar per day (g)

<database-chart by="x6s7mffoqy" chart="bar" db="c5x394veib" measure="sum" unit="day" value="6bm82vq6xg"/>

<columns>

<column>

### Where it comes from

<database-chart by="fvc39yjrl5" chart="bars-across" colors="color" db="c5x394veib" sort="desc"/>

</column>

<column>

### Sugary drinks per week

<database-chart by="x6s7mffoqy" chart="bar" db="c5x394veib" measure="sum" unit="week" value="1r2bhplgtb"/>

</column>

</columns>

### Sugar vs energy

<database-chart chart="scatter" db="c5x394veib" x="6bm82vq6xg" y="v48ky07uu9"/>

## Weekly review

<toggle>

Every Sunday, answer these questions

1. What was my average score, and which habit dragged it down?
1. Did sugar go over 36 g on any day? What was the trigger?
1. What did I ship on my projects this week?
1. One thing to change next week:

</toggle>
