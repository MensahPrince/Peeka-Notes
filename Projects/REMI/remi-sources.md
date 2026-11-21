---
id: 841cvgtq57
title: "REMI Sources"
sort: 11000
version: 1
---

Every dataset, method, paper and report used in REMI so far: **what it is**, **where it's used**, and **why it was chosen**, to cite in your research. Tracked from [REMI Project Hub](page:imqxd436oq). *Started 2026-11-21; add to it as the project grows.*

<callout icon="📚">

**How to read this page.** Each section is a kind of source. **Used in** names the script in `~/gitRepo/REMI/` (code at `github.com/MensahPrince/REMI`) and the Peeka page where the results are written up. **Why** is the reason it was picked over the alternatives. Licences matter if you publish maps: they're noted where known.

</callout>

## 🛰️ Satellite and elevation data

| Source | What it gives | Used in | Why |
| --- | --- | --- | --- |
| **Copernicus Sentinel-2 Level-2A**, via the **Element 84 Earth Search** STAC catalogue (`earth-search.aws.element84.com/v1`, collection `sentinel-2-l2a`) | Surface reflectance at 10–20 m, every \~5 days. Images of 2024-03-31 and 2025-02-04 | `s2_earth_search.py` (download), `normalise.py` (per-tile reload for the seam fit). [REMI Test Imagery](page:epzdv5qm9v), [REMI M2 Preprocessing Report](page:8u0jq0qub6) | Free, the sharpest open imagery (10 m), and Earth Search serves it as cloud-ready files with no account. ⚠️ Its metadata wrongly says to subtract a 0.1 offset; these files already have it removed (see the M2 report) |
| **Landsat 8/9 Collection 2 Level-2**, via the **Microsoft Planetary Computer** (collection `landsat-c2-l2`) | Surface reflectance at 30 m, every \~8 days with both satellites, archive back to 2013 (and older Landsats to 1984) | `landsat_planetary.py`. [REMI Test Imagery](page:epzdv5qm9v) | Longer history than Sentinel-2, for older "before" dates. The Planetary Computer was used because USGS EarthExplorer was down on 2026-10-21 (503 errors) |
| **Copernicus DEM GLO-30**, via the Planetary Computer (collection `cop-dem-glo-30`) | Elevation at 30 m | `dem_streams.py` → `data/derived/streams_dem.gpkg`. [REMI Project Hub](page:imqxd436oq) (OSM river gap) | Free, global, and the best open 30 m elevation model. Water flows downhill, so it gives the small streams OSM is missing, even under forest |

## 🗺️ Map data

| Source | What it gives | Used in | Why |
| --- | --- | --- | --- |
| **OpenStreetMap** (© OpenStreetMap contributors, ODbL), via the `osmnx` Python library | Rivers, water bodies and roads for the box | `osm_download.py` → `data/raw/osm.gpkg`. Used by `water_mask.py` checks, `candidates.py` (`river_m`, `road_m`) and `reported_sites.py` (the Ankobra) | Free and editable. ⚠️ Incomplete: only 130 km of rivers in the box, almost all the main Ankobra; most small streams are missing |
| **OSM Nominatim and Overpass** (place search) | Coordinates of named towns and villages, searched 2026-11-21 | `reported_sites.py` (`PLACES`) | To place news reports that name villages but give no coordinates. ⚠️ OSM has only 68 named places in the box; Ehyeamu, Awodua, Prestea-Hemang, Tarkwa Bremang and Kadadwen weren't found |

## 🏷️ Label datasets

| Source | What it gives | Used in | Why |
| --- | --- | --- | --- |
| **ESA WorldCover 2021 v200** (10 m), via the Planetary Computer (collection `esa-worldcover`). Zanaga et al. (2022), Zenodo, doi:10.5281/zenodo.7254221. CC BY 4.0 | Global land cover in 11 classes at 10 m | `labels.py` → `data/labels/<grid>_landclass.tif`. [REMI M3 Labelled Data Report](page:spbr0bdedd) | The only free 10 m land-cover map matching Sentinel-2's resolution. Gives REMI's land classes for M4 without hand labelling. ⚠️ It calls 40 % of today's bare ground "forest", so use it for green land only |
| **Tang, L. & Werner, T. T. (2023).** *Global mining footprint mapped from high-resolution satellite imagery.* Communications Earth & Environment 4, 134. doi:10.1038/s43247-023-00805-6. Data: Zenodo record 7894216 (doi:10.5281/zenodo.7894216; earlier version 10.5281/zenodo.6806817). CC BY 4.0 | 74,548 hand-drawn mining outlines worldwide (pits, tailings, ponds, waste dumps, plant); 170 touch the box | `labels.py` → `data/labels/mining_tang2023.gpkg`, `<grid>_mining.tif`; `candidates.py` (`mining_tw2023`); `tiles.py` (mining band, stratified split). [REMI M3 Labelled Data Report](page:spbr0bdedd) | Hand-drawn on sharp imagery, so the outlines are more precise than any automatic map, and they include artisanal sites, not just industrial mines. ⚠️ They are loose envelopes (37 % of the land inside is green), and miss half of today's bare ground |
| **Maus, V. et al. (2022).** *An update on global mining land use.* Scientific Data 9, 433. doi:10.1038/s41597-022-01547-4. Data: *Global-scale mining polygons (Version 2)*, PANGAEA, doi:10.1594/PANGAEA.942325. CC BY-SA 4.0 | 44,929 mining outlines drawn on the 2019 Sentinel-2 cloudless mosaic, within 10 km of known mines. Validation: overall accuracy 88.3 % | **Not used yet.** PANGAEA's file store was down on 2026-11-21 | A second, independent set of outlines, to cross-check Tang & Werner. Retry the download later |

## 🧮 Methods and software

| Source | What it does | Used in | Why |
| --- | --- | --- | --- |
| **OmniCloudMask** (Python package; github.com/DPIRD-DMA/OmniCloudMask) | AI cloud and cloud-shadow detector for 10–50 m imagery | `cloud_mask.py`. [REMI M2 Preprocessing Report](page:8u0jq0qub6) step 1 | Sentinel-2's built-in cloud layer (SCL) missed most small clouds and called the bright Tarkwa tailings "cloud", which would hide fresh mining. OmniCloudMask caught the clouds and left the tailings clear |
| **Canty, M. J. & Nielsen, A. A. (2008).** *Automatic radiometric normalization of multitemporal satellite imagery with the iteratively re-weighted MAD transformation.* Remote Sensing of Environment 112(3), 1025–1036 | **IR-MAD**: finds pixels that didn't change between two dates | `normalise.py` (`irmad`). M2 step 3 | Matches the two dates' brightness using only ground that really didn't change, so real change (mining) isn't averaged away |
| **Xu, H. (2006).** *Modification of normalised difference water index (NDWI) to enhance open water features in remotely sensed imagery.* International Journal of Remote Sensing 27(14), 3025–3033 | **MNDWI** water index, (green − SWIR) / (green + SWIR) | `water_mask.py`. M2 step 2 | Separates water from built-up and bare ground better than the original NDWI, which matters around towns and mine tailings |
| **Tucker, C. J. (1979).** *Red and photographic infrared linear combinations for monitoring vegetation.* Remote Sensing of Environment 8(2), 127–150 | **NDVI** greenness index, (NIR − red) / (NIR + red) | `normalise.py` check, `candidates.py` (bare = NDVI \< 0.3) | The standard greenness measure; bare ground left by mining has low NDVI |
| **pysheds** (Python package; github.com/mdbartos/pysheds) | Flow direction, flow accumulation and stream extraction from a DEM | `dem_streams.py` | A simple, free way to compute a stream network in Python |
| Python stack: `rasterio`, `rioxarray`, `geopandas`, `shapely`, `odc-stac`, `pystac-client`, `planetary-computer`, `scipy`, `numpy`, `matplotlib` | Reading, writing and processing rasters and vectors | All scripts. [REMI Software Stack](page:uvhchbwrgv) | Free, standard tools for satellite data in Python |

## ⚖️ Concessions and governance

| Source | What it says | Used in | Why it matters |
| --- | --- | --- | --- |
| **Minerals Commission, Mining Cadastre** (mincom.gov.gh/mining-cadestre/) | Ghana's register of mineral rights, run on the Mining Cadastre Administration System since 2015. The Commission also sells a 1:1,000,000 concession map (US$150) | [REMI Concession Data Request](page:jnd1xbtqpj) | The official source of licence boundaries, needed for Layer 6's inside / outside / unknown licence check |
| Citi FM (2016), *Minerals Commission launches online portal for mining cadastre* (citifmonline.com/2016/08/…); Geospatial World, *Mining cadastre administration system launches in Ghana* | A public portal launched in 2016, reported at `ghana.revenuesystems.org`, showing concessions and application status | [REMI Concession Data Request](page:jnd1xbtqpj) (fallbacks) | A possible free route to licence data. Not confirmed to still work |
| **data.gov.gh**, *Shapefiles of Mineral Resource in Ghana 2010* | Open GIS data on mineral resources (2010) | Not used | Probably mineral occurrences rather than licences, and 15 years old |
| **Clark University (2016)**, *Extraction in Ghana* (wordpress.clarku.edu/…/ExtractioninGhana.pdf) | Mapped small-scale mining against large-scale concessions from Landsat: **52 %** of small-scale mining lay inside large-scale concessions | [REMI Concession Data Request](page:jnd1xbtqpj) | Shows why "inside a concession" doesn't mean "legal": the licence check must handle overlap |
| **Bright Simons, Joy Online**, *Bogus mining data in Ghana poses danger* | The Commission's small-scale licence figures differ by \~20 % between releases | [REMI Concession Data Request](page:jnd1xbtqpj) | Even official licence data needs checking |

## ⛏️ Reported galamsey sites

Reports of illegal mining in or near the pilot box, used to check REMI's labels. Points are in `data/labels/reported_sites.gpkg`; see [REMI M3 Labelled Data Report](page:spbr0bdedd) step 4.

| Report | Date | Place | What it reports | In the box? |
| --- | --- | --- | --- | --- |
| **Ministry of Lands and Natural Resources**, *Galamsey fight: Prestea Blue Water Guards destroy 8 chanfang machines, seize 8 pumping machines near the Ankobra River* (mlnr.gov.gh) | 2025-07-10 | "Amponsah" site, Himan, on the Ankobra | 8 changfan destroyed, 8 pumps seized, 3 more changfan sank | ✅ Located (Himan) |
| **MyJoyOnline / JoyNews** (Erastus Asare Donkor), *Illegal miners block River Ankobra, turn it into a stagnant pool* | 2025-03-02 | Prestea-Hemang (near the bridge), Awodua (near Prestea), Ehyeamu | Excavators mining in the river; the Ankobra blocked by embankments; "over a year" of mining | Probably, but places not found in OSM |
| **MyJoyOnline** (David Apinga), *Bogoso Blue Water Guards destroy 5 galamsey equipment on River Ankobra* | 2025-11-21 | River Ankobra at Tarkwa Bremang, Prestea-Huni Valley | 5 changfan machines burned | Probably, but place not found in OSM |
| **Ghanaian Times** (Kingsley Asare), *12 excavators seized, 3 arrested in fresh anti-galamsey fight* | 2025-10-20 | Western Region: Wassa Agona, **Bogoso**, Bayerigya, Ehyireso Mpohor, along the Bonsa and Mansi | 22 pumps and 18 changfan destroyed in the Western Region (not split by site) | Bogoso only (town level) |
| **GhanaWeb**, *Navy arrests 56 'galamseyers' at Tarkwa-Nsuaem* | Not read (the site blocked the download) | Kadadwen, Tarkwa-Nsuaem | 56 arrests | Unknown |
| **Owusu-Nimo, F., Mantey, J., Nyarko, K. B., Appiah-Effah, E. & Aubynn, A. (2018).** *Spatial distribution patterns of illegal artisanal small scale gold mining (Galamsey) operations in Ghana: A focus on the Western Region.* Heliyon 4(2), e00534. doi:10.1016/j.heliyon.2018.e00534 | Field survey Dec 2014 – Apr 2016 | Western Region; communities include Prestea, Bogoso, Aboso, Bompieso, Huni Valley | GPS field survey of 911 sightings and 7,470 operations. **Tarkwa-Nsuaem: 294 sightings, 3,648 sites; Prestea-Huni Valley: 156 sightings, 1,130 sites.** Alluvial mining concentrated along the Bonsa, Ankobra and Huni rivers | Districts yes. Per-site coordinates **not published** |

<toggle>

Background reports (context, not mapped)

- **Graphic Online**, *Five galamseyers jailed 125 years for mining in River Ankobra*: arrests from August 2021.
- **Pan African Visions** (August 2025), *Reclaiming Ghana's forests*: the Lands Ministry puts forest reserve losses to illegal mining at **5,252 ha**; photo of the damaged **Neung Forest Reserve**, Nsuaem, Tarkwa (south of the box). Mentions L.I. 2462 (2022), which permitted mining in forest reserves.
- **3News**, *Lands Minister launches 4 mining cooperatives in Tarkwa* (and Bogoso), as a legal alternative to galamsey (November 2026).
- **GBC Ghana**, *NAIMOS arrests 11 suspected illegal miners* (2026): Western and Eastern Regions; not in the box.
- **Graphic Online / Citi Newsroom** (May 2025): nationwide police update, 208 arrests and 99 excavators retrieved.
- **GhanaWeb** (2017), *How 17 miners died in Nsuta*: pit collapse at Nsuta, just south of the box.
- **Mantey, J. et al. (2016)**, *Costed reclamation and decommissioning strategy for galamsey operations* (International Growth Centre final report): background on galamsey types and reclamation.
- **Prestea-Huni Valley Municipal Assembly**, District Environmental Sanitation Strategy (DESSAP 2017–2021): lists Dumasi, Damang, Kyekyewere, Bogoso, Bondaye, Awudua and Prestea among the towns most affected by mining (incl. cyanide spills).
- Not readable on 2026-11-21: **Starr FM**, *River Ankobra: NAIMOS, Blue Water Guards raid galamsey site* (certificate error) and **ModernGhana**, *20 Blue Water Guards deployed to Prestea Huni-Valley* (403). Read them in a browser.

</toggle>

## 🔗 Links

<toggle>

All URLs

- Earth Search: https://earth-search.aws.element84.com/v1
- Planetary Computer STAC: https://planetarycomputer.microsoft.com/api/stac/v1
- Tang & Werner paper: https://www.nature.com/articles/s43247-023-00805-6
- Tang & Werner data: https://zenodo.org/records/7894216
- Maus et al. data: https://doi.pangaea.de/10.1594/PANGAEA.942325
- OmniCloudMask: https://github.com/DPIRD-DMA/OmniCloudMask
- pysheds: https://github.com/mdbartos/pysheds
- Minerals Commission cadastre: https://www.mincom.gov.gh/mining-cadestre/
- Minerals Commission home: https://www.mincom.gov.gh/
- Mineral rights: https://www.mincom.gov.gh/mineral-rights/
- Citi FM, cadastre portal: https://citifmonline.com/2016/08/minerals-commission-launches-online-portal-for-mining-cadastre
- Geospatial World: https://geospatialworld.net/news/mining-cadastre-administration-system-launches-ghana/
- data.gov.gh: https://data.gov.gh/dataset/shapefiles-mineral-resource-ghana-2010
- Clark University: https://wordpress.clarku.edu/wp-content/uploads/sites/393/2016/06/ExtractioninGhana.pdf
- Joy Online (Bright Simons): https://www.myjoyonline.com/bright-simons-bogus-mining-data-in-ghana-poses-danger
- MLNR Himan raid: https://mlnr.gov.gh/galamsey-fight-prestea-blue-water-guards-destroy-8-chanfang-machines-seize-8-pumping-machines-near-the-ankobra-river/
- JoyNews Ankobra blocked: https://www.myjoyonline.com/illegal-miners-block-river-ankobra-turn-it-into-a-stagnant-pool/
- MyJoyOnline Tarkwa Bremang: https://www.myjoyonline.com/bogoso-blue-water-guards-destroy-5-galamsey-equipment-on-river-ankobra/
- Ghanaian Times: https://ghanaiantimes.com.gh/12-excavators-seized-3-arrested-in-fresh-anti-galamsey-fight/
- GhanaWeb navy arrests: https://www.ghanaweb.com/GhanaHomePage/crime/Navy-arrests-56-galamseyers-at-Tarkwa-Nsuaem-1975464
- Owusu-Nimo et al. 2018: https://pmc.ncbi.nlm.nih.gov/articles/PMC5835009/
- Graphic, five jailed: https://www.graphic.com.gh/news/general-news/five-galamseyers-jailed-125-years-for-mining-in-river-ankobra.html
- Pan African Visions: https://panafricanvisions.com/2025/08/reclaiming-ghanas-forests-the-devastating-impact-of-illegal-gold-mining/
- 3News cooperatives: https://3news.com/news/lands-minister-launches-four-mining-cooperatives-tarkwa-bogoso
- GBC NAIMOS: https://www.gbcghanaonline.com/general/naimos-raids-arrests/2026/
- Citi Newsroom police update: https://citinewsroom.com/2025/05/galamsey-208-arrested-99-excavators-retrieved-nationwide-police/
- GhanaWeb Nsuta: https://www.ghanaweb.com/GhanaHomePage/NewsArchive/How-17-miners-died-in-Nsuta-The-inside-story-by-a-survivor-555494
- IGC (Mantey et al. 2016): https://www.theigc.org/sites/default/files/2017/01/Mantey-et-al-2016-Final-report.pdf
- PHMA DESSAP: https://phma.gov.gh/wp-content/uploads/2022/07/DESSAP-UPDATE-FROM-2017-2021.pdf
- Starr FM: https://starrfm.com.gh/river-ankobra-naimos-blue-water-guards-destroy-raid-galamsey-site-destroy-equipment/
- ModernGhana: https://www.modernghana.com/news/1400995/galamsey-fight-20-blue-water-guards-deployed-to.html

</toggle>
