# 2.1 Ocean Colour and Sea Surface Temperature

Phytoplankton change the colour of seawater. This module uses that fact to map **chlorophyll-a** and **particulate organic carbon** from NASA's PACE mission, then adds **sea surface temperature** from JAXA's Himawari-9 so the three can be read together.

**Assignment questions 1, 2 and 3 all come from this notebook.**

Notebook: `2.1.1_PACE_OCI_Chlorophyll_a.ipynb`

## What we will do

Download six months of Level-3 mapped ocean colour from the Ocean Colour Instrument, look inside the file to see what is in it, and map chlorophyll-a, globally, over a region you choose, and with all the months side by side on one shared colour scale. Then download four months of Himawari sea surface temperature and map those the same way. Every figure is saved as a PNG for your write-up.

**Sections 8 and 9 are required.** Section 8 answers question 1, section 9 answers questions 2 and 3. The other blocks of settings are optional practice, not marked:

| | Experiment | The question behind it |
|---|---|---|
| **2.1 §5** *(optional)* | A region explorer with named boxes, and any of the four variables in the file | Where is the ocean green, and what supplies the nutrients there? |
| **2.1 §7** *(optional)* | The seasonal cycle as a regional time series | *When* does each region bloom, and what is the wind doing at the time? |
| **2.1 §8** | Four months of Himawari SST on one colour scale, with regional medians | How does the temperature of the Asia-Pacific move through the year? |
| **2.1 §9** | Chlorophyll-a, POC and SST mapped side by side, for two months and two regions | Does warmer water mean less phytoplankton, and is the answer the same everywhere? |

An optional cell maps the change between two months as a ratio rather than a difference, the right way to compare a quantity that spans orders of magnitude.

## What you need first

- A [NASA Earthdata](https://urs.earthdata.nasa.gov/users/new) account, free and instant, for the ocean colour. See [0.2](../0.2_Data_Access_Accounts/).
- A [JAXA P-Tree](https://www.eorc.jaxa.jp/ptree/) account for the sea surface temperature in section 8. **Approval takes a few working days**, so register early. Sections 1 to 7 run without it.
- [0.1 Setup](../0.1_Setup_and_Environment/) completed.

## The data

PACE OCI Level-3 Global Mapped Ocean Biogeochemical Properties (BGC), version 3.2, monthly, 0.1°.

Chlorophyll-a is the `chlor_a` variable inside it, and **POC is the `poc` variable in the same file**, so there is no second ocean-colour download. The file also carries `carbon_phyto` and `pic`.

Sections 8 and 9 also need **sea surface temperature**, which is not in that file. Section 8 downloads JAXA's Himawari-9 monthly Level-3 SST, 0.02°, for four months of 2025, into this module's own `data/SST/` folder.

Browse it at <https://search.earthdata.nasa.gov/search?portal=obdaac>. The notebook fetches it with `earthaccess` (`short_name="PACE_OCI_L3M_BGC"`).

Files land in `data/Chlorophyll-a/` and figures in `output/figures/`. Both are gitignored; the data does not belong in the repository.

## The science behind it

The reasoning sits in the notebook, beside the maps it explains: **§1** for why phytoplankton change the colour of the water, **§5** for where the ocean is green and what supplies the nutrients there, and **§7** for the monsoon and the seasonal cycle. None of it is examinable on its own.

One thing the notebook does not cover is what **"Level-3 mapped"** means. Two processing steps have already been applied at the data centre: the individual satellite passes have been averaged over time, and the result resampled onto a regular grid of latitudes and longitudes. That is why this data drops straight into a map, with no reprojection needed. In notebook 2.3 you meet SWOT data that has *not* had this done to it, and the difference is immediately obvious.

## Background reading

- <https://pace.oceansciences.org/mission.htm>
- <https://pace.oceansciences.org/oci.htm>

## What you end up with

**PNG figures in `output/figures/`**, the global map, the region and monthly-panel maps, and one figure per experiment. Each is named after the settings that made it (`chl_poc_sst_south_china_sea_202501_202504.png`), so your figure is labelled before you paste it in.
