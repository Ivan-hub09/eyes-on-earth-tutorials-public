# 2.2 Sea Surface Temperature and ENSO

Monthly sea surface temperature from the Himawari-8/9 geostationary satellites, over the tropical Pacific, where the rest of the tutorial happens.

> **Himawari is introduced in [notebook 2.1](../2.1_Ocean_Colour_with_PACE_OCI/), section 8**, which downloads four months of 2025 over FTP and maps the Asia-Pacific for assignment question 1. This module takes the same instrument to the tropical Pacific for the ENSO questions, and its three files need no account.

Notebook: `2.2.1_Himawari_SST.ipynb`

## What we will do

Download the three ENSO months over HTTPS, convert Kelvin to Celsius, crop the grid to a longitude range that map projections can actually handle, and map it, the full disc first and then the tropical Pacific. Each figure is saved as a PNG for your write-up.

Two blocks of settings you can change. **Only 2.2 §8 is required**, assignment question 7 asks you to paste it in. The other is optional practice, not marked:

| | Experiment | The question behind it |
|---|---|---|
| **2.2 §6** *(optional)* | The latitude of the thermal equator, from a north–south profile | Where is the warmest water, why is it not at 0°, and where does it go in July? |
| **2.2 §8** | Warm-pool area above a threshold, plus the equatorial east–west gradient | How do you reduce the state of an entire ocean basin to two numbers? |

Sections 7–9 are the tropical Pacific half of the notebook: SST for the three ENSO periods, the warm-pool experiment, and the difference maps between periods. **Only section 8 gives you an answer you hand in**, question 7. Question 4 comes from a published ENSO index rather than from this notebook, and questions 1 to 3 are answered in notebook 2.1.

> **Section 9 will not run until you fill it in.** It asks you to assign each of the three periods to its ENSO phase first, which is question 4 of the [tutorial assignment](../2.0_Tutorial_2_Overview_and_Assignment/README.md), and that phase has to come from a published index rather than from the look of the maps. Leave the three names blank and the cell stops with a `ValueError` naming the periods it has loaded. That is deliberate: reading the phase off a map and then using the phase to interpret the same map is circular, so the notebook will not help you do it. Nothing from section 9 is handed in — it is there so you can check your question 5 answer against the data.

## What you need first

- [0.1 Setup](../0.1_Setup_and_Environment/) completed.
- Nothing else. The three files this notebook uses are hosted for the course and section 2 fetches them over HTTPS. The P-Tree account is needed by [notebook 2.1](../2.1_Ocean_Colour_with_PACE_OCI/) section 8, not here.

## The data

Himawari-8/9 AHI monthly Level-3 SST, on a 0.02° full-disc grid.

- The three ENSO months are hosted for the course at `.../tutorial_2/processed_data/Himawari_SST/`, reachable over HTTPS from section 2
- The originals are on `ftp.ptree.jaxa.jp`, path `/pub/himawari/L3/SST/v201_nc4_normal_std_monthly/`. Notebook 2.1 section 8 uses that route. **The archive has gaps** — October and November 2025 were never published, so check the directory listing before putting a month into `SST_PERIODS`
- Filenames: `H09_YYYYMMDD_hhmm_1MSST201_FLDK.06001_06001.nc`
- Satellite overview: <https://www.jma.go.jp/jma/jma-eng/satellite/himawari89.html>

Files land in `data/SSTraw/` and figures in `output/figures/`. Both are gitignored.

Section 2 downloads exactly the three months questions 4–7 need, `202212`, `202312` and `202407`, and there is nothing to change. The four 2025 months that questions 1 to 3 use are downloaded by notebook 2.1 section 8, into that module's own `data/SST/` folder.

## Two things to handle before mapping

Section 4 of the notebook deals with both, and explains them as it goes: the SST arrives in **Kelvin** (subtract 273.15), and the grid runs to **200°E**, past the −180…180 range map projections expect. Sections 4 to 6 crop to 80–180°E; sections 7–9 need the part beyond 180°, because the tropical Pacific crosses the antimeridian, and move the seam instead of dropping data.

## The science behind it

Optional reading, the notebook links here. Background to what the notebook does, not examinable on its own.

### Moving the seam

A plate carrée projection is defined from −180° to +180°, and those two edges are the same meridian: the one line where the globe has to be cut open to be laid out flat. The tropical Pacific crosses it.

There are only two ways round that, throw away the data on one side of the seam, or move the seam. Section 4 of the notebook does the first, dropping everything past 180°, because Southeast Asia does not need it. Section 7 does the second: it centres the projection on 180° and renumbers the longitudes to match, so 180°E becomes 0, 90°E becomes −90 and 200°E becomes +20. The array stays in one contiguous piece.

The code reads `(da.lon % 360) - 180`. The `% 360` comes first because the coordinate in these files runs 80 → 179.98 and then jumps to −180 → −160; the modulo puts it back into an ascending 80 → 200, and the `- 180` shifts it into the projection's frame.

Relabelling the far side as negative instead would leave the longitudes running 80 … 180, −180 … −160, not in ascending order, and with a 240°-wide hole through the middle of the map.

### Why the region stops at 30°

Near the equator the Coriolis parameter is small, and that lets waves travel *along* the equator: an eastward Kelvin wave crosses the Pacific in two to three months, and westward Rossby waves return more slowly. Those crossings are what make ENSO an oscillation with a period of years rather than a one-way drift.

Beyond about 30° the seasonal cycle is larger than anything ENSO does, which is why section 9 of the notebook narrows further still, to 15°S–15°N, before asking you to read a difference map.

## Credentials

This notebook needs none. Its three files come over HTTPS from the course bucket.

The P-Tree FTP credentials are used by [notebook 2.1](../2.1_Ocean_Colour_with_PACE_OCI/) section 8, which reads them from `~/.netrc`, written once by [0.2.1](../0.2_Data_Access_Accounts/0.2.1_Account_Check.ipynb). Remember that P-Tree's **FTP** credentials are not the same as the P-Tree website login. That is the single most common reason the section 8 download fails.
