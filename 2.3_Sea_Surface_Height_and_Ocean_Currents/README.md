# 2.3 Sea Surface Height and Ocean Currents

Sea surface height anomalies and total surface currents across the tropical Pacific, from the SWOT mission and the OSCAR project.

Warm water takes up more room than cold water, so a column of ocean with a thick warm layer on top stands taller. Measure the height of the sea surface precisely enough and you are measuring the heat stored beneath it, which is why this is the sharpest view of ENSO in the whole tutorial. Water does not sit still on a slope either, so the second half of the module reads the currents that height field drives, and the two halves are one measurement seen twice.

Notebook: `2.3.1_SSHA_and_Currents.ipynb`

## What we will do

**Part 1, sections 1 to 7.** Load pre-processed SWOT L2_LR_SSH data and map SSHA across the tropical Pacific (30°S–30°N, 90°E–60°W) for each time period, so ocean surface topography can be related to the phases of ENSO.

**Part 2, sections 8 to 13.** Map total surface current speed and direction over the same box and the same three periods.

**Sections 6, 12 and 13 are required**, and assignment questions 5, 6 and 8 ask you to paste their figures in. Section 5 is optional practice, not marked:

| | Experiment | The question behind it |
|---|---|---|
| **2.3 §5** *(optional)* | A day-by-day animation of one month of SWOT passes accumulating | How does a satellite measuring in narrow swaths build up a picture of a whole ocean basin? |
| **2.3 §6** | The equatorial SSHA profile, and the difference between periods | Can the state of an entire ocean basin be reduced to one number? |
| **2.3 §12** | Total surface currents mapped for all three periods, on one colour scale (you paste in the December 2023 one) | Does the flow you can see agree with the SSHA tilt from section 6, and with the ENSO phase of the period? |
| **2.3 §13** | The eastward speed on the equator, one number per period | Which period has eastward flow on the equator? The bands are named on the plot as context, not as something to learn. |

Section 5 animates the collection: one frame per day, showing passes accumulate until the basin fills. It writes a GIF to `output/figures/` and plays it inline. You can measure SWOT's 21-day repeat cycle off your own data rather than reading it off a spec sheet.

**For question 8**, section 1 also downloads a pre-processed extract for the El Niño currently under way. Instructors: see [Rebuilding the current-month extract](#rebuilding-the-current-month-extract) below.

## The sea surface height data

Two pre-processed files hold SSHA for the tropical Pacific, plus the August 2026 extract for question 8. Section 1 downloads all three from the course bucket over HTTPS, the two oldest as MATLAB `.mat`, the newest as NetCDF. The notebook reads either container, so it makes no difference if you hand students the files directly instead.

| Period | Available |
|---|---|
| December 2022 | **No**, a Sentinel-6/Jason-3 composite is provided instead, in section 7 of the notebook |
| December 2023 | Yes |
| July 2024 | Yes |
| August 2026 | Yes, the extract question 8 uses |

SWOT launched on 16 December 2022 and was in commissioning for months afterwards, the earliest Version D granule is 27 March 2023, so for that month the measurement was never made. See *[Which SWOT collection](#which-swot-collection)* below. `Figs/ssha_composite_dec2022.png` covers it with conventional nadir altimetry, note it is in **centimetres**, while the SWOT files are in metres.

Files go in `data/SWOT/`, which is gitignored.

## The one dataset that is not a grid

PACE and Himawari hand students a regular grid: a mesh of latitudes and longitudes with one value per cell. SWOT does not. It measures along the two halves of its swath as it flies, so each extract is a **list of samples**, one row per measurement, carrying its own longitude, latitude and anomaly:

| Column | |
|---|---|
| 1 | Longitude (90°E to 60°W) |
| 2 | Latitude (30°S to 30°N) |
| 3 | SSHA, in metres |

In a MATLAB `.mat` that array is called `master_data`. Two things follow, and both are why this notebook looks different from the others:

- There is no `ssha[lat, lon]` array, so it cannot be sliced by degrees the way the PACE and Himawari notebooks are.
- It is drawn with a **scatter plot**, one dot per sample, rather than `pcolormesh`, there is no mesh to fill. The gaps between swaths and the nadir gap are visible in the result, which is exactly the sampling story mission question 1 asks about.

Section 6 bins the samples onto a grid with `scipy.stats.binned_statistic_2d` before it does anything else, because the periods cannot be averaged along the equator or subtracted from one another until there is one value per cell. Its `BIN_DEG` setting is the cell size, and changing it shows that the resolution of a Level-3 grid is a processing choice with consequences, not a property of the ocean.

## The ocean current data

OSCAR Near Real Time (NRT) ocean surface currents, v2.0. **Three `.nc` files**, one per time period, hosted for the course at `.../tutorial_2/processed_data/OSCAR_ocean_currents/`, section 9 downloads them over HTTPS into `data/OSCAR/`, which is gitignored.

These are **daily** NRT fields, one date standing in for each period, not monthly means like the PACE and Himawari products. A single day can catch a passing eddy that a monthly average would smooth away, so read a single arrow with that in mind.

| Period | File |
|---|---|
| December 2022 | `oscar_currents_nrt_20221210.nc` (the 10th) |
| December 2023 | `oscar_currents_nrt_20231201.nc` |
| July 2024 | `oscar_currents_nrt_20240701.nc` |

Students can also fetch them with `earthaccess` (`short_name="OSCAR_L4_OC_NRT_V2.0"`), the notebook shows how.

Each file contains two kinds of current, each with two components:

| Variable | Meaning |
|---|---|
| `u`, `v` | **Total** surface current, eastward, northward |
| `ug`, `vg` | **Geostrophic** surface current, eastward, northward |

The assignment asks for the **total** current, so the ENSO maps use `u` and `v`. The file also carries `ug` and `vg`.

> **Dimension names and array order.** The dimensions in these files are named `longitude`/`latitude` while the coordinate variables are `lon`/`lat`, and the arrays arrive as (longitude, latitude). `load_currents()` swaps the dimensions so `.sel(lat=...)` works in degrees, and transposes to (lat, lon), which is the order the rest of the tutorial assumes. Section 10 explains why.

## The most important thing about the current data

**OSCAR does not measure currents. Nothing in orbit does.** It *calculates* them from things satellites do measure, sea surface height, wind, and SST, using two pieces of physics: geostrophic flow from the slope of the sea surface, and Ekman flow from the wind.

That makes it the one dataset in the tutorial that is a model output constrained by observations rather than an observation. Section 8 of the notebook makes the point explicitly, because knowing which of your datasets are measured and which are derived tells you how far to trust the fine detail in each.

## The science behind it

Optional reading, the notebook links here.

### Geostrophy, and the size of the current

Water starts to run **down** a slope in the sea surface. The Coriolis effect acts on moving water, so as soon as it is moving it is deflected, right in the northern hemisphere, left in the southern, and it keeps turning until the deflection exactly opposes the downslope push. The two forces balance and the water ends up running **along** the slope rather than down it, circling a high instead of draining off it.

That balance, between the pressure-gradient force and the Coriolis force, gives a speed of

> slope × *g* / *f*,  where  *f* = 2Ω sin φ

is the Coriolis parameter, Ω the Earth's rotation rate and φ the latitude.

To get a feel for the size of it: a 10 cm rise over 100 km at 10°N is a slope of 10⁻⁶, and *g*/*f* there is about 3.9 × 10⁵ m s⁻¹ per unit slope, so roughly **0.4 m s⁻¹**. That is a typical tropical surface current, from a sea-surface bump you would never notice by eye. Note that *f* shrinks towards the equator, so the *same* slope drives a faster current the closer to it you look.

The useful consequence for students: an SSHA map is a current map. North of the equator the geostrophic flow keeps the **high on its right**, south of it on its left, which is how the maps from 2.3 and the maps from this notebook can be read against each other.

### Why it breaks down on the equator

Because *f* = 2Ω sin φ and sin 0° = 0, the Coriolis parameter vanishes at the equator and the geostrophic speed, which goes as 1/*f*, would be unbounded. The balance simply does not hold there.

OSCAR handles the equatorial band with the **beta-plane approximation**, deriving the geostrophic current from the *curvature* of the sea surface rather than its slope, β being the rate at which *f* changes with latitude, which is at its largest exactly where *f* itself is smallest. This is why the geostrophic estimate does not fall to zero on the equator.

### Ekman transport

Wind drags the surface water along, Coriolis deflects that layer, the layer drags the one beneath it, which is deflected a little further again, the Ekman spiral. Integrated over the whole wind-driven layer, the **net transport ends up at 90° to the wind**, to the right in the northern hemisphere, to the left in the southern.

That 90° is the mechanism behind upwelling, and it turns up twice in this tutorial. Along a coast, wind blowing parallel to the shore drives surface water offshore and cold, nutrient-rich water rises to replace it, which is why the Peru and Benguela coasts are among the greenest water in the chlorophyll maps of notebook 2.1. On the equator, the westward trades push water northward just north of it and southward just south of it, so the surface diverges and deeper water rises along the whole line: the cold tongue in the 2.2 SST maps and the equatorial green band in 2.1 are the same mechanism seen by two more instruments.

## Background reading

**Sea surface height**

- How satellite altimetry works: <https://geodesy.science/ggos/item/satellite-altimetry/>
- SWOT at PO.DAAC: <https://podaac.jpl.nasa.gov/SWOT>
- The L2_LR_SSH product, **Version D**, the live one: <https://podaac.jpl.nasa.gov/dataset/SWOT_L2_LR_SSH_BASIC_D>
- Mission site: <https://swot.jpl.nasa.gov/>
- SWOT User Handbook (D-109532): <https://express.adobe.com/page/pX485lhUb4Pml/>

**Ocean surface currents**

- Dataset page: <https://podaac.jpl.nasa.gov/dataset/OSCAR_L4_OC_NRT_V2.0>
- User guide (PDF): <https://archive.podaac.earthdata.nasa.gov/podaac-ops-cumulus-docs/oscar/open/L4/oscar_v2.0/docs/oscarv2guide.pdf>

**ENSO**

- ENSO monitoring: <https://www.ncei.noaa.gov/access/monitoring/enso/>

## After this notebook

Go back to the [Tutorial 2 overview](../2.0_Tutorial_2_Overview_and_Assignment/README.md) for the assignment, which brings chlorophyll-a, POC, SST, SSHA and surface currents together, and then points them at the El Niño currently under way.

---

# For instructors

Everything below this line is about producing and distributing the data. Students do not need any of it.

## Which SWOT collection

PO.DAAC publishes each L2 low-rate SSH pass in four variants and two processing baselines. `scripts/make_swot_extract.py` uses **`SWOT_L2_LR_SSH_BASIC_D`**.

| Variant | One pass | What it is |
|---|---|---|
| **Basic** | ~9 MB | Limited variable set for the general user, `latitude`, `longitude`, `ssha_karin`, `ssha_karin_qual`. What we use. |
| Expert | ~31 MB | Every geophysical correction as a separate variable, plus uncertainties |
| WindWave | ~9 MB | Wind and wave parameters |
| Unsmoothed | ~835 MB | The 250 m native grid, minimally smoothed |

> **Do not use `SWOT_L2_LR_SSH_BASIC_2.0`.** It is Version C: *SUPERSEDED*, granules stop **3 May 2025**. Version D is the reprocessed replacement, 27 March 2023 to present.
>
> Both still have a live page, a working DOI, and "Start/Stop Date: 2022-12-16 to Present". That is a declared extent, not an inventory. Version C returns zero granules for a recent month, with no error.

**The crossover correction is not applied for you.** `ssha_karin` and `ssha_karin_2` both leave out the crossover calibration; the file's own metadata says to add `height_cor_xover` yourself. Skip it and spacecraft roll leaves a ~2 m tilt across the swath, one side of every pass reads high, the other low. The script uses `ssha_karin_2 + height_cor_xover`, keeping only samples where `ssha_karin_2_qual`, `height_cor_xover_qual` and `ancillary_surface_classification_flag` are all 0, plus a `|ssha| < 1 m` sanity screen for the handful of coastal samples that pass every flag and still come back at 2–3 m. Measured on one tropical Pacific pass: −2.59 to +2.09 m before, −0.34 to +0.31 m after.

Two things to know before picking a period:

- **No version has December 2022.** SWOT launched 16 December 2022; the earliest Version D granule is 27 March 2023. Hence the Sentinel-6/Jason-3 composite in section 7.
- **March–July 2023 was the 1-day calibration orbit**, not the 21-day science orbit. A period in that window would break the repeat-cycle observation in section 5.

## Rebuilding the current-month extract

**Instructors only — students never run this.** They download the finished file from the course bucket in section 1.

Question 8 needs a SWOT extract for whatever month the El Niño is in when the course runs. Build it with:

```bash
python scripts/make_swot_extract.py --latest
```

It resolves the most recent complete month with data, fetches every pass crossing 30°S–30°N, 90°E–60°W (about 500 of them, ~4.5 GB), reduces each to the samples inside the box, and writes a single compressed NetCDF of about 100 MB. It downloads 25 passes at a time and deletes each granule once read, so peak disk is ~225 MB, and it checkpoints after every batch, an interrupted run resumes instead of restarting.

Give it a month directly with `python scripts/make_swot_extract.py 202611`, cap it with `--max-passes 20` for a test run, or make a smaller file with `--thin-along 8`.

When it finishes it prints the `aws s3 cp` line to upload it, and the edits to make afterwards.

**The month is written out in full everywhere it appears**, deliberately, there is no one variable to change. Each of the two 2.3 notebooks names it three times: `SSHA_FILES` in section 1, `ANIM_FILE` in section 5, and `DIFF_NOW` in section 6. It is in the prose as well, question 8 in the [tutorial handout](../2.0_Tutorial_2_Overview_and_Assignment/README.md#question-8--the-el-niño-happening-now-seen-from-sea-surface-height), and the prose around section 6 of this notebook, both say *August 2026* and `202608` outright. Grep the repository for `202608` and for `August 2026` and change every hit, or the handout will ask for a month the notebooks no longer download.

The output is the same point layout as the `.mat` extracts, `lon`, `lat` and `ssha` along one `obs` dimension, so `load_ssha()` and every experiment read it unchanged. Longitudes are 0–360; the plotting and gridding code normalises with `% 360` either way.

It adds one variable the older extracts do not have: **`day`**, the day of the month each sample was measured on, as `uint8`. Section 5 animates on it. It comes from the granule's `time`, which is one timestamp per along-track line, broadcast across the swath. It costs about a byte per sample and compresses to near nothing; a full timestamp would have doubled the file.

A hand-converted `.mat` extract will have no `day` variable, and section 5 will refuse to run on it. That is intended, a `.mat` extract has no time information to recover.

## Distributing the data

The notebook reads the `.mat` extracts **directly**, `scipy.io.loadmat`, falling back to `h5py` for MATLAB v7.3 files, which are really HDF5. There is no conversion step to run and nothing for students to install beyond the course environment.

If you would rather hand out NetCDF, convert it while **keeping the point layout**. A single `obs` dimension, with longitude, latitude and SSHA as three variables along it:

```python
import numpy as np
import scipy.io
import xarray as xr

raw = scipy.io.loadmat("SWOT_SSHA_202312.mat", squeeze_me=True)
data = np.asarray(raw["master_data"], dtype="float64")   # N x 3

xr.Dataset(
    {
        "lon": ("obs", data[:, 0]),
        "lat": ("obs", data[:, 1]),
        "ssha": ("obs", data[:, 2].astype("float32"),
                 {"units": "m", "long_name": "Sea surface height anomaly"}),
    },
    attrs={"source": "SWOT L2_LR_SSH Basic 2.0, pre-processed extract"},
).to_netcdf("SWOT_SSHA_202312.nc")
```

Do **not** reshape it onto a lat/lon grid on the way out. Regridding scattered swath samples fills the gaps with interpolated values, and the gaps are part of what the tutorial is teaching.

Either way the files are too large to commit to the repository, so they are hosted alongside the other processed data at `.../tutorial_2/processed_data/SWOT_SSHA/` and the notebook pulls them from there. If you move them, update `COURSE_DATA` and `SSHA_FILES` in section 1 of both notebooks.
