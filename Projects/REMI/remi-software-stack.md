---
id: uvhchbwrgv
title: "REMI Software Stack"
cover: "_assets/030577d9f2f7eea091a45d8b5c434135.jpg"
cover_position: 50
sort: 6000
version: 1
---

The software needed to process REMI's images, layer by layer, and how it is set up on this laptop. Data comes from the [REMI Data Download Guide](page:sf7hcbolae); the design is in [🛰️R.E.M.I](page:ckbvimllzp).

<callout background="green" icon="✅">

**Python environment ready** (set up 2026-10-21): Python 3.12 in `~/Downloads/REMI-test-data/.venv`, 1.6 GB, every library below installed and tested. A U-Net ran on a 6-band test tile, and the Sentinel-2 GeoTIFF opened correctly.

</callout>

<columns>

<column>

### ▶️ Use it

```bash
cd ~/Downloads/REMI-test-data
source .venv/bin/activate
python make_previews.py
jupyter lab
```

</column>

<column>

### 🔁 Rebuild it

```bash
pip install --user uv
uv venv --python 3.12 .venv
uv pip install --python .venv \
  -r requirements.txt \
  --index-strategy unsafe-best-match
```

About a minute. The library list is in `requirements.txt`.

</column>

</columns>

## 🖥️ Desktop tools

| Tool | What it's for | Status |
| --- | --- | --- |
| **QGIS** | View GeoTIFFs at full resolution, overlay rivers and roads, and **draw outlines of mining scars for training labels** (M3) | ✅ Installed (3.44) |
| **GDAL command-line tools** | Convert, crop and reproject rasters from the terminal (`gdalinfo`, `gdalwarp`, `ogr2ogr`) | ✅ Installed (3.11). Python doesn't need it; `rasterio` ships its own copy |
| **uv** | Creates the Python environment and installs libraries | ✅ Installed (`~/.local/bin/uv`) |

## 🐍 Python libraries, by layer

| Layer | Libraries | What they do |
| --- | --- | --- |
| 1 · Data Acquisition | `pystac-client`, `odc-stac`, `planetary-computer`, `requests` | Search satellite catalogues and download just the pilot box |
| 2 · Preprocessing | `rasterio`, `rioxarray`, `xarray`, `dask` | Read, crop, reproject and align rasters, including ones too big for memory |
| 2 · Cloud masking | `omnicloudmask` | A stronger cloud and shadow mask than Sentinel-2's own `SCL` band, which missed small clouds in [REMI Test Imagery](page:epzdv5qm9v) |
| 3–5 · Models | `torch`, `torchvision`, `timm` | PyTorch, plus ResNet and EfficientNet for land classification |
| 3–5 · Satellite AI | `torchgeo` | Satellite datasets, and **models pre-trained on Sentinel-2**, which matters because labelled galamsey data is scarce |
| 5 · Segmentation | `segmentation-models-pytorch` | U-Net and DeepLabV3, to outline damaged areas |
| 6 · Decision | `geopandas`, `shapely`, `osmnx` | Rivers, roads, concession boundaries and alert outlines as shapes; "is this near a river / inside a licence?" |
| 6 · Stream network | `pysheds` | Computes streams from the Copernicus elevation map (`dem_streams.py`), filling the gaps in OSM's rivers. The script includes a one-line patch, because `pysheds` still calls `numpy.in1d`, which NumPy 2.4 removed |
| Exploring | `jupyterlab`, `matplotlib`, `pillow` | Notebooks, plots and image previews |
| 7 · API and map (later) | `fastapi`, Leaflet or MapLibre | Not installed yet; added when M9 starts |

## 💻 Hardware

<callout background="yellow" icon="⚠️">

**No NVIDIA GPU on this laptop** (it has Intel HD Graphics 620), so PyTorch is the **CPU-only** build. That is fine for M1–M3 (downloading, cleaning, labelling) and for testing models on a few tiles. **Training** the models (M4–M6) on CPU would take far too long.

</callout>

- **For training:** use a free GPU notebook on [Google Colab](https://colab.research.google.com) or [Kaggle](https://www.kaggle.com/code). Install the same libraries from `requirements.txt` there (without the CPU-only line) and upload the tiles.
- **Disk:** about 31 GB was free on `/home` at setup. Each Sentinel-2 date of the pilot box is about 127 MB, so the pilot fits comfortably; full Copernicus tiles (about 1 GB each) would fill it fast.

## ✅ To do

- [x] Python 3.12 environment with every library installed and tested
- [x] Install QGIS
- [x] Install the GDAL command-line tools
- [ ] Set up a Colab or Kaggle account for training on a GPU
- [ ] Move `~/Downloads/REMI-test-data` to a permanent project folder and put the code in git (keeping `data/` and `.venv/` out of it)
