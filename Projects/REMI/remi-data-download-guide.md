---
id: sf7hcbolae
title: "REMI Data Download Guide"
sort: 4000
version: 1
---

How to get **test data for M1 (Data access)**: Sentinel-2 from Copernicus plus the other two open sources, Landsat and OpenStreetMap. Sources and their status are tracked in the Data Sources table on [REMI Project Hub](page:imqxd436oq); the design is in [🛰️R.E.M.I](page:ckbvimllzp).

<callout icon="📌">

**For testing, start small.** Pick one pilot area (the example below is the **Prestea / Pra river** galamsey belt, Western Region, Ghana), two dates about a year apart, and the **dry season (Dec–Feb)**, when the sky is clearest. Download only the area you need, not whole 100 km tiles.

</callout>

<columns>

<column>

### 📍 Example pilot area

**Bounding box (lon/lat, WGS84):**

`-2.20, 5.30, -1.90, 5.60`

About 33 × 33 km.

</column>

<column>

### 📅 Example dates

- **Before:** 2023-12-01 → 2024-02-28
- **After:** 2024-12-01 → 2025-02-28
- Cloud cover **≤ 20 %**

</column>

<column>

### 🧰 Tools

```bash
pip install pystac-client odc-stac \
  planetary-computer rioxarray \
  requests osmnx
```

</column>

</columns>

## Which route to take

| Source | Main route | Other route | Account needed? | Format |
| --- | --- | --- | --- | --- |
| Sentinel-2 L2A | Earth Search STAC (AWS) for testing | Copernicus Data Space (official) | Only for Copernicus downloads | COG bands / SAFE zip |
| Landsat 8–9 C2 L2 | **Microsoft Planetary Computer** | USGS EarthExplorer (optional backup) | **No** (USGS only if you use it) | COG bands / tar |
| OpenStreetMap | Overpass via `osmnx` | Geofabrik extract | No | GeoPackage / .pbf |

<callout icon="💡">

**STAC routes crop to your box.** They stream just the pixels inside the bounding box from cloud-optimised GeoTIFFs: one date of the pilot box, all six bands, was **127 MB in about 40 s** when tested. The official portals hand you whole 100 km tiles (\~1 GB each; the pilot box needs **four** of them per date), which is fine for a final dataset but slow for testing.

</callout>

---

## 🛰️ 1 · Copernicus Sentinel-2

Use **Level-2A** (surface reflectance, already atmosphere-corrected, with a scene classification layer `SCL` for cloud masking in Layer 2).

<callout background="yellow" icon="⚠️">

**The pilot box spans four tiles** (30NWL, 30NXL, 30NWM, 30NXM), and not every pass captures all four. A day with only three leaves part of the box empty (2024-12-01 was about 20 % no-data). Days with all four in the example window: **2024-12-21, 2025-01-05, 2025-02-04**.

The cloud filter also works **per tile**, not per box: "under 20 %" for the tile still meant about 9 % cloud or shadow inside the box. Layer 2 masks clouds with the `SCL` band either way.

</callout>

### Option A: Copernicus Data Space Ecosystem (official)

1. Register for a free account at [dataspace.copernicus.eu](https://dataspace.copernicus.eu).
1. **By hand:** open the [Copernicus Browser](https://browser.dataspace.copernicus.eu), draw the pilot area, set the dates, choose *Sentinel-2 L2A*, set max cloud cover, and download the product.
1. **By script:** search the catalogue (no account needed), remove duplicates, then get a token and download (code below).

<toggle>

Python: search and download Sentinel-2 L2A from Copernicus Data Space

```python
import os, requests

os.makedirs("data/raw", exist_ok=True)

# 1. Search the catalogue (no login needed), following every page of results
aoi = "POLYGON((-2.20 5.30,-1.90 5.30,-1.90 5.60,-2.20 5.60,-2.20 5.30))"
flt = (
    "Collection/Name eq 'SENTINEL-2' "
    f"and OData.CSC.Intersects(area=geography'SRID=4326;{aoi}') "
    "and ContentDate/Start gt 2024-12-01T00:00:00.000Z "
    "and ContentDate/Start lt 2025-02-28T00:00:00.000Z "
    "and Attributes/OData.CSC.StringAttribute/any(a:a/Name eq 'productType' "
    "and a/OData.CSC.StringAttribute/Value eq 'S2MSI2A') "
    "and Attributes/OData.CSC.DoubleAttribute/any(a:a/Name eq 'cloudCover' "
    "and a/OData.CSC.DoubleAttribute/Value le 20)"
)
url = "https://catalogue.dataspace.copernicus.eu/odata/v1/Products"
params = {"$filter": flt, "$orderby": "ContentDate/Start", "$top": 100}
products = []
while url:
    page = requests.get(url, params=params).json()
    products += page["value"]
    url, params = page.get("@odata.nextLink"), None   # next page, if any

# 2. Keep one product per tile and day: the newest processing.
#    Names look like S2B_MSIL2A_20241201T102319_N0511_R065_T30NXL_20241201T125430.SAFE
#    and reprocessed copies of the same image share the date and tile.
latest = {}
for p in products:
    sat, _, sensed, baseline, _, tile, processed = p["Name"].removesuffix(".SAFE").split("_")
    key = (sensed[:8], tile)
    if key not in latest or (baseline, processed) > latest[key][0]:
        latest[key] = ((baseline, processed), p)
products = [p for _, p in sorted(latest.values(), key=lambda v: v[1]["Name"].split("_")[2])]

for p in products:
    print(p["Name"], f"{p['ContentLength'] / 1e9:.1f} GB")

# 3. Access token (expires after ~10 minutes; get a new one per batch)
token = requests.post(
    "https://identity.dataspace.copernicus.eu/auth/realms/CDSE/protocol/openid-connect/token",
    data={"client_id": "cdse-public", "grant_type": "password",
          "username": os.environ["CDSE_USER"], "password": os.environ["CDSE_PASSWORD"]},
).json()["access_token"]

# 4. Download the tiles of one day (each a ~1 GB SAFE zip)
day = "20250204"
for p in products:
    if p["Name"].split("_")[2][:8] != day:
        continue
    with requests.get(
        f"https://zipper.dataspace.copernicus.eu/odata/v1/Products({p['Id']})/$value",
        headers={"Authorization": f"Bearer {token}"}, stream=True,
    ) as r:
        r.raise_for_status()
        with open(f"data/raw/{p['Name']}.zip", "wb") as f:
            for chunk in r.iter_content(1 << 20):
                f.write(chunk)
```

Tested 2026-10-21: the search finds 22 products for the example window, and **20 remain after removing duplicates** (2024-12-16 had two copies of each tile). The download step is not yet tested.

</toggle>

### Option B: Earth Search STAC on AWS (fastest for testing)

No account. The same Sentinel-2 L2A data, as cloud-optimised GeoTIFFs, cropped to the box.

<toggle>

Python: load a cropped Sentinel-2 stack and save it

```python
import os
from pystac_client import Client
import odc.stac
import rioxarray  # adds .rio to xarray

os.makedirs("data/raw", exist_ok=True)

bbox = [-2.20, 5.30, -1.90, 5.60]
catalog = Client.open("https://earth-search.aws.element84.com/v1")

items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2024-12-01/2025-02-28",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()
print(len(items), "scenes")

ds = odc.stac.load(
    items,
    bands=["blue", "green", "red", "nir", "swir16", "scl"],
    bbox=bbox,
    crs="EPSG:32630",      # UTM 30N covers Ghana
    resolution=10,
    groupby="solar_day",   # merge tiles from the same pass
    chunks={},
)

# One GeoTIFF per date (~127 MB each)
for t in ds.time.values:
    day = str(t)[:10]
    ds.sel(time=t).to_array().rio.to_raster(f"data/raw/s2_{day}.tif")
```

Tested 2026-10-21: 22 scenes over 8 days; one day wrote a 6-band, 10 m, 3323 × 3329 GeoTIFF in UTM 30N.

</toggle>

---

## 🌎 2 · NASA / USGS Landsat 8 and 9

Use **Collection 2, Level-2** (surface reflectance). 30 m pixels, but a record back to 2013 (Landsat 8), which helps find older "before" images when Sentinel-2 dates are cloudy.

<callout background="green" icon="✅">

**No USGS account needed.** The Microsoft Planetary Computer serves the same Landsat Collection 2 Level-2 data as USGS (same scenes, bands and cloud flags) without an account, cropped to the box. USGS is only a backup, for the very latest scenes (the mirror can lag by a few days), or for citing the official source.

</callout>

<callout background="yellow" icon="⚠️">

**Only one Landsat grid square covers the box: path 194, row 056.** Its neighbour, row 057, only touches a corner, so its scenes come out **99 % empty**. Keep only **Tier 1** scenes (names ending `_T1`); Tier 2 has lower positional accuracy, which matters for change detection. The script below filters for both.

</callout>

### Option A: Microsoft Planetary Computer STAC (main route, no account)

<toggle>

Python: download a cropped Landsat GeoTIFF per date

```python
import os
from pystac_client import Client
import planetary_computer, odc.stac
import rioxarray  # adds .rio to xarray

os.makedirs("data/raw", exist_ok=True)

bbox = [-2.20, 5.30, -1.90, 5.60]
catalog = Client.open(
    "https://planetarycomputer.microsoft.com/api/stac/v1",
    modifier=planetary_computer.sign_inplace,   # adds read tokens to asset URLs
)

items = catalog.search(
    collections=["landsat-c2-l2"],
    bbox=bbox,
    datetime="2024-12-01/2025-02-28",
    query={"eo:cloud_cover": {"lt": 20},
           "platform": {"in": ["landsat-8", "landsat-9"]},
           "landsat:wrs_path": {"eq": "194"},
           "landsat:wrs_row": {"eq": "056"},               # the one grid square covering the box
           "landsat:collection_category": {"eq": "T1"}},   # Tier 1 only
).item_collection()
print(len(items), "scenes")

ds = odc.stac.load(
    items,
    bands=["blue", "green", "red", "nir08", "swir16", "qa_pixel"],
    bbox=bbox, crs="EPSG:32630", resolution=30,
    groupby="solar_day", chunks={},
)

# One GeoTIFF per date (~15 MB each)
for t in ds.time.values:
    day = str(t)[:10]
    ds.sel(time=t).to_array().rio.to_raster(f"data/raw/landsat_{day}.tif")
```

Landsat L2 values are scaled integers: reflectance = `value × 0.0000275 − 0.2`.

Tested 2026-10-21. Without the row and tier filters, the "after" window gave 8 scenes, four of them 99 % empty. With them:

| Window | Scenes (scene cloud %) |
| --- | --- |
| Before, Dec 2023 – Feb 2024 | 2023-12-19 (18.6), 2024-01-20 (18.0), **2024-01-28 (2.5)**, **2024-02-21 (2.7)** |
| After, Dec 2024 – Feb 2025 | **2024-12-21 (2.9)**, 2025-01-30 (15.3, but 74 % cloud inside the box) |

</toggle>

<callout icon="💡">

The Planetary Computer also has `sentinel-2-l2a` (band names there are `B02`, `B03`, `B04`, `B08`, `SCL`), so one catalogue can serve both satellites.

</callout>

### Option B: USGS EarthExplorer (optional backup)

<callout background="orange" icon="🚧">

**USGS was down on 2026-10-21**: EarthExplorer, the sign-up page and the M2M API all returned *503 Service Unavailable*. Use Option A until it's back.

</callout>

1. Register at [ers.cr.usgs.gov](https://ers.cr.usgs.gov/register).
1. In [EarthExplorer](https://earthexplorer.usgs.gov), set the area (*Search Criteria → Polygon*, or upload a shapefile) and dates.
1. *Data Sets → Landsat → Landsat Collection 2 Level-2 → Landsat 8-9 OLI/TIRS C2 L2*.
1. *Additional Criteria*: land cloud cover under 20 %, and WRS path 194 / row 056.
1. *Results*: download the product bundle (a `.tar` with one GeoTIFF per band plus `QA_PIXEL` for clouds).

Scripted downloads from USGS go through the **M2M API**, which needs a separate access request on your account.

---

## 🗺️ 3 · OpenStreetMap (rivers and roads)

Used in Layer 6: mining clusters near water, and roads show access.

<callout background="yellow" icon="⚠️">

**OSM's rivers are incomplete here.** Roads are well mapped, but the rivers layer is almost only the **Ankobra River** itself: most smaller streams, where riverside galamsey happens, are missing. Use OSM for roads, river names and the main channel, not as the full river network. How the gap is being closed is under Risks on [REMI Project Hub](page:imqxd436oq).

</callout>

### Option A: just the pilot area, with `osmnx`

<toggle>

Python: rivers, water bodies and roads in the bounding box

```python
import os
import osmnx as ox

os.makedirs("data/raw", exist_ok=True)

bbox = (-2.20, 5.30, -1.90, 5.60)   # west, south, east, north

layers = {
    "rivers": {"waterway": ["river", "stream", "canal", "drain"]},
    "water":  {"natural": "water"},            # ponds and lakes, including flooded pits
    "roads":  {"highway": True},
}

for name, tags in layers.items():
    gdf = ox.features_from_bbox(bbox, tags=tags)
    lines_only = name != "water"
    keep = ["LineString", "MultiLineString"] if lines_only else ["Polygon", "MultiPolygon"]
    gdf = gdf[gdf.geom_type.isin(keep)]                     # drop stray points (bus stops, springs...)
    cols = [c for c in ["name", *tags.keys(), "water"] if c in gdf.columns]
    gdf = gdf[["geometry", *dict.fromkeys(cols)]].reset_index()
    gdf.to_file("data/raw/osm.gpkg", layer=name)
    print(f"{name:7s} {len(gdf):5d} features")
```

Tested 2026-10-21 (about 4 minutes, one 1.2 MB file with three layers):

| Layer | Features | Total | What's in it |
| --- | --: | --: | --- |
| `roads` | 2,410 | 1,136 km | Unclassified 377 km, residential 347, secondary 119, service 113, track 62, primary 50, tertiary 49, path 15 |
| `rivers` | 33 | 130 km | River 104 km (the Ankobra), stream 26 km, drain 0.2 km |
| `water` | 8 | 27 ha | Ponds and lakes; most mining ponds are missing |

</toggle>

### Option B: all of Ghana, from Geofabrik

Download [ghana-latest.osm.pbf](https://download.geofabrik.de/africa/ghana-latest.osm.pbf) (updated daily), then open it in QGIS or with GDAL:

```bash
ogr2ogr -f GPKG data/raw/ghana_osm.gpkg ghana-latest.osm.pbf lines \
  -where "waterway IN ('river','stream') OR highway IS NOT NULL"
```

---

## 📁 Where to put it

```text
data/
  raw/          # untouched downloads (never edit these)
    s2_2024-03-31.tif
    s2_2025-02-04.tif
    landsat_2024-02-21.tif
    landsat_2024-12-21.tif
    osm.gpkg
  interim/      # Layer 2 output: masked, aligned, tiled
```

Keep `data/` out of git, and record what was downloaded (source, product IDs, dates) in a small `data/raw/MANIFEST.md` so a test can be reproduced.

## ✅ Checklist for M1

- [x] Create a Copernicus Data Space account
- [ ] *(Optional)* Create a USGS ERS account, as a backup for Landsat once USGS is back up
- [x] Pull Sentinel-2 L2A for the "before" and "after" windows (2024-03-31 and 2025-02-04; no clear "before" day with all four tiles until March)
- [x] Pull Landsat 8–9 C2 L2 for the same windows, from the Planetary Computer (2024-01-28, 2024-02-21 and 2024-12-21)
- [x] Pull OSM rivers and roads for the pilot area
- [x] Open everything in QGIS and check it lines up
- [ ] Update each row's **Status** in the Data Sources table on [REMI Project Hub](page:imqxd436oq)
- [ ] Request concession boundaries from the Minerals Commission (no download route; see the Risks on the hub)

<toggle>

Troubleshooting

- **No scenes found:** widen the date range or raise the cloud limit to 40 %; tropical cloud is the usual cause.
- **Fewer Copernicus results than expected:** the catalogue returns results a page at a time; follow `@odata.nextLink` as the script does.
- **Copernicus 401 on download:** the token expired (about 10 minutes). Request a new one.
- **Planetary Computer 403:** the asset URLs weren't signed; pass `modifier=planetary_computer.sign_inplace`.
- **A Landsat image is almost all empty:** it's from grid row 057, which only touches the box's corner; filter for `landsat:wrs_row` 056.
- **USGS returns 503:** the service is down; the Planetary Computer has the same Landsat data.
- **Earth Search "InvalidCredentials" error on an older Sentinel-2 scene:** that item is stored in a pay-to-access bucket (`s3://sentinel-s2-l2a`). Skip items whose asset link doesn't start with `https`.
- **Part of a Sentinel-2 image is empty:** that pass didn't capture every tile over the box. `groupby="solar_day"` merges the tiles that exist but can't fill a missing one; pick a day with all four tiles.
- **OSM download is slow:** the Overpass server can take several minutes for the road layer; that's normal.
- **Images don't line up:** load both satellites with the same `crs` and `bbox`, then resample Sentinel-2 to 30 m when comparing with Landsat.

</toggle>
