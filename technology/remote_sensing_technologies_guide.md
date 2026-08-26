# Remote Sensing Technologies: A Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — Earth Observation, Geospatial, Remote Sensing, Satellite/Aerial Sensors, Climate Risk
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Last Updated:** August 2026

---

## Table of Contents

1. [Remote Sensing Fundamentals](#1-remote-sensing-fundamentals)
2. [The Platforms](#2-the-platforms)
3. [The Sensor Types](#3-the-sensor-types)
4. [Data Processing](#4-data-processing)
5. [Data Sources and Platforms](#5-data-sources-and-platforms)
6. [The Applications](#6-the-applications)
7. [Worked Example — NDVI Rice Crop Monitoring](#7-worked-example--ndvi-rice-crop-monitoring)
8. [The Future (2026+)](#8-the-future-2026)
9. [Summary — Remote Sensing in One Page](#9-summary--remote-sensing-in-one-page)
10. [Glossary](#10-glossary)
11. [Claims Status, References and Further Reading](#11-claims-status-references-and-further-reading)

### How to Read This Guide

This is the dedicated deep-dive on **Remote Sensing Technologies** — the acquisition of information about the Earth's surface *without physical contact* — in the `technology/` general-tech series. It is the umbrella guide for the geospatial/Earth-observation domain: everything that measures the Earth from a distance, from the physics of light to the business of satellite data. Several sibling guides carry adjacent depth and are cross-referenced inline:

- **Maritime application** — [maritime_domain_awareness_guide.md](maritime_domain_awareness_guide.md) is the sibling use-case guide: maritime domain awareness (AIS, SAR/EO satellite surveillance, oil-spill detection) is one of remote sensing's flagship applications. Cross-reference its §2.3 (satellite sensors), §3.1 (satellite AIS), §6.3 (environmental — oil spills), and §9 (dark-fleet detection) as the worked maritime case of everything this guide covers generically.
- **Sensing fundamentals** — [ips_rtls_guide.md](ips_rtls_guide.md) covers *indoor* RF positioning; its core concepts (signal → measurement → position fix, error sources, sensor fusion) map onto remote sensing's measurement chain, but the physics (radio indoors vs. optical/SAR from orbit) and scales differ.
- **Sensor data streaming** — [event_stream_processing_guide.md](event_stream_processing_guide.md) and [complex_event_processing_guide.md](complex_event_processing_guide.md) cover the streaming/CEP plumbing that downstream EO pipelines (change-detection alerts, anomaly rules) sit on.
- **ML/AI** — classification, anomaly detection, and the deep-learning material in §4.7–4.8 build on the `ai_llm/` series ([deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md), [llm_evaluation_vs_validation_guide.md](ai_llm/llm_evaluation_vs_validation_guide.md)) and [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md).
- **Data engineering** — the geospatial data pipelines in §4 and §5 draw on [data/data_pipeline_guide.md](data/data_pipeline_guide.md) and [data/data_fabric_guide.md](data/data_fabric_guide.md).
- **Banking** — §6.9 (remote sensing in banking: climate physical risk, ESG, commodities) cross-references [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md). Note: the repo does not yet have dedicated ESG/climate-risk or GIS guides — the geospatial-analysis essentials are covered inline here, and a dedicated GIS guide remains a natural future companion (the maritime guide flagged the same gap).

Suggested reading paths: **solution architects** start with §1, §4, §5, §6, §7; **data/ML engineers** start with §3, §4, §8.3; **banking/risk readers** start with §1, §6.8–6.9, §7; **geospatial newcomers** start with §1–§3.

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against agency/vendor/standards-body or reputable industry sources during research), **Reported** (widely reported but not independently confirmed), or hedged/flagged in-line where sources diverge. The full claims-status table is in §11.1. Satellite programs and commercial constellations move fast — re-verify before procurement or policy decisions.

---

## 1. Remote Sensing Fundamentals

### 1.1 What Is Remote Sensing?

**Remote sensing** is the acquisition of information about an object, area, or phenomenon — most commonly the Earth's surface — **without making physical contact** with it. Instead of touching the target, a remote sensor measures the *electromagnetic radiation* (EMR) that the target reflects, emits, or scatters, and converts those measurements into data from which information is inferred.

> *"Remote sensing is the science of obtaining information about objects or areas from a distance, typically from aircraft or satellites."* — the working definition used across NASA/USGS educational material. **Verified** as the standard framing (the exact wording varies slightly by agency).

The field sits at the intersection of physics (electromagnetic radiation, atmospheric optics), engineering (sensors, platforms, orbits), and data science (image processing, statistics, machine learning). The modern discipline is inseparable from **Earth Observation (EO)** — the satellite-borne wing of remote sensing — but the techniques predate satellites by a century (balloon photography, 1858; aerial photography in WWI; aerial survey between the wars).

The essential chain is always the same:

```
   EM RADIATION SOURCE         ATMOSPHERE           TARGET          SENSOR          DATA          INFORMATION
 (sun, or own transmitter)  (scattering/absorption) (land/ocean)  (camera, radar)  (pixels/DNs)  (maps, indices,
        |                          |                   |              |              |           classifications,
        v                          v                   v              v              v           decisions)
```

### 1.2 Active vs Passive Remote Sensing

The first and most fundamental distinction is where the measured radiation comes from:

- **Passive sensors** measure radiation that originates elsewhere — almost always **reflected sunlight** (optical/multispectral/hyperspectral imaging) or **naturally emitted heat** (thermal infrared). They are the "eyes" of remote sensing: they see what the Earth reflects or radiates. Passive optical sensors **require daylight** and **cannot see through clouds**.
- **Active sensors** carry their own illumination — they transmit a signal (microwave pulses for radar, laser pulses for LiDAR) and measure the echo that returns. Active systems **work day and night** and, in the microwave region, **penetrate clouds**, rain, and (partially) vegetation and dry soil. This is why radar and LiDAR dominate all-weather and polar-region applications.

| Aspect | Passive (optical, thermal) | Active (SAR, LiDAR) |
|---|---|---|
| Energy source | Reflected sunlight / emitted heat | Own transmitter |
| Day/night | Optical: day only; thermal: day+night | Day and night |
| Clouds | Blocked (optical) / partially blocked (thermal) | Penetrated (microwave); LiDAR blocked |
| Typical data | Reflectance/emissivity images | Backscatter images, range/point clouds |
| Examples | Landsat, Sentinel-2, MODIS | Sentinel-1 (SAR), GEDI (LiDAR) |

### 1.3 The Electromagnetic Spectrum

Remote sensing works across a wide slice of the electromagnetic spectrum. Each region carries different information about the surface, because materials reflect, absorb, and emit differently at each wavelength. **Verified** region boundaries (sources vary by a few hundredths of a µm; the table gives the widely used convention):

| Region | Wavelength | What it sees | Notes |
|---|---|---|---|
| **Visible (VIS)** | 0.4–0.7 µm | What the human eye sees: true-colour imagery, chlorophyll green | Blue (0.45–0.51), green (0.53–0.59), red (0.64–0.67) bands |
| **Near-infrared (NIR)** | 0.7–1.3 µm | Vegetation health, water bodies | Plant cell structure reflects NIR strongly; water absorbs it |
| **Shortwave-infrared (SWIR)** | 1.3–3 µm | Soil moisture, minerals, snow/cloud discrimination, fire hotspots | Sensitive to water content in plants and soils |
| **Mid-wave infrared (MWIR)** | 3–5 µm | Very hot targets (fires, flares, lava) | Thermal window, less used than TIR for land |
| **Thermal infrared (TIR)** | 8–14 µm | Surface temperature, heat, fires, sea surface temperature | The main "thermal window"; measures *emitted* heat, not reflected light |
| **Microwave (radar)** | 1 mm – 1 m | Surface roughness, structure, moisture; penetrates clouds | Wavelength 1000× longer than visible — the basis of SAR |

**Radar bands.** Within the microwave region, specific frequency bands (IEEE standard letter designations) matter enormously because penetration and scattering behaviour scale with wavelength. **Verified**:

| Band | Frequency | Wavelength | Behaviour | Example missions |
|---|---|---|---|---|
| **X** | 8–12 GHz | ~2.5–3.75 cm | Short wavelength: high resolution, limited penetration; sensitive to fine roughness | TerraSAR-X, TanDEM-X, COSMO-SkyMed, Capella, ICEYE |
| **C** | 4–8 GHz | ~3.75–7.5 cm | The workhorse band: good all-round resolution/penetration balance | Sentinel-1 (5.405 GHz), RADARSAT-2, ENVISAT ASAR |
| **L** | 1–2 GHz | ~15–30 cm | Long wavelength: deep vegetation/soil penetration; less affected by ionosphere | ALOS-2/PALSAR-2, NISAR (2025), SAOCOM |
| **S** | 2–4 GHz | ~7.5–15 cm | Middle ground, less common for EO | HJ-1C, NovaSAR |
| **P** (future) | 0.3–1 GHz | ~30–100 cm | Deepest penetration — planned for biomass mapping | BIOMASS (ESA, planned) |

Rule of thumb: the longer the wavelength, the deeper the penetration into vegetation and dry soil, but the coarser the achievable resolution at a given antenna size — which is why SAR uses *synthetic aperture* processing (§3.4).

### 1.4 Spectral Signatures — the Fingerprint of Materials

The single most important concept in optical remote sensing is the **spectral signature**: the reflectance curve of a material — the fraction of incident light reflected at each wavelength. Different materials have characteristic, repeatable curves, so **materials can be identified from their reflectance spectrum alone**. **Verified** (classic, textbook-established behaviour; see claims table §11.1):

- **Healthy vegetation**: low reflectance in the visible (chlorophyll absorbs blue and especially red for photosynthesis), then a **sharp rise** in reflectance across ~680–750 nm — the **red edge** — into very high NIR reflectance (leaf mesophyll cell structure scatters NIR), peaking in the NIR, then declining through the SWIR as leaf water content absorbs. The red edge is the steepest part of the plant spectrum; its position shifts with chlorophyll content and stress, which is why "red-edge" bands (Sentinel-2 B5–B7, Pleiades Neo) are so valuable for agriculture.
- **Water**: low reflectance across the visible, and **very strong absorption in the NIR and SWIR** — open water appears nearly black in NIR imagery. This makes NIR the classic water-detection band (NDWI, §4.6).
- **Bare soil**: reflectance **increases with wavelength** across the VIS→SWIR range (roughly monotonic rise toward the SWIR, modulated by mineralogy and moisture — wet soil is darker everywhere).
- **Snow/ice**: very high reflectance in the visible, dropping sharply in the SWIR — the VIS/SWIR contrast distinguishes snow from clouds.
- **Urban materials**: concrete, asphalt, and roofing have mixed, angular signatures with low NIR relative to vegetation.

ASCII illustration of the canonical vegetation vs. water vs. soil curves:

```
Reflectance (%)
  70 |                        .·´  vegetation
     |                  .·´·´
  50 |            .·´·´
     |       .·´
  30 |  .·´                ·´ soil (rising into SWIR)
     |  |              ·´
  10 |  |_ water (flat,     ·´
     |    near-zero in NIR)
   0 +----|----|----|----|----|----|---->
      0.4  0.6  0.8  1.0  1.5  2.0  2.5 µm
           red edge: sharp rise ~680–750 nm
```

Because reflectance is a *ratio* (reflected / incident energy), it is roughly comparable across scenes and dates once sensor calibration and atmospheric correction are applied — which is what makes multi-temporal analysis and spectral indices possible (§4).

### 1.5 Resolution — the Four Dimensions of an Image

"Resolution" in remote sensing is not one thing but four, and every sensor is a compromise among them:

1. **Spatial resolution** — the ground area each pixel represents, the **Ground Sampling Distance (GSD)**: the pixel edge length on the ground. Landsat = 30 m (multispectral), Sentinel-2 = 10 m, PlanetScope = 3 m, WorldView Legion = 0.30 m (30 cm). A 30 m pixel covers 900 m²; a 30 cm pixel covers 0.09 m² — a 10,000× difference in area.
2. **Spectral resolution** — the number and width of bands. Panchromatic: 1 broad band; multispectral: 4–13 bands (Sentinel-2 has 13); hyperspectral: 200+ narrow (~10 nm) contiguous bands (EnMAP: 242).
3. **Temporal resolution** — the revisit time: how often the sensor images the same spot. Geostationary: continuous (every ~10–15 min for full-disk weather imagery); Planet: daily; Landsat/Sentinel-2: ~5–16 days; EnMAP: 27 days. (Note: revisit is often *effective* — with side-looking and constellation pairs, e.g., Sentinel-2A+2B give 5-day revisit at the equator, ~2–3 days at mid-latitudes.)
4. **Radiometric resolution** — the sensor's ability to distinguish levels of brightness/energy: the **bit depth** of the digitised signal. Legacy Landsat (1–7) recorded 8-bit (256 grey levels); **Landsat 8/9 OLI and Sentinel-2 MSI record 12-bit** (4,096 levels) and distribute data as 16-bit integers. **Verified.** Finer radiometric depth matters for dark targets (water, shadows), bright targets (snow), and subtle vegetation stress.

### 1.6 Resolution Tradeoffs

The four resolutions fight each other, and both physics and economics enforce the tradeoffs:

- **Spatial vs temporal**: a wide swath (more frequent revisit) requires smaller optics/arrays and coarser GSD — or a constellation of many small satellites (Planet's ~200+ Doves give 3 m + daily, at the cost of per-pixel sophistication). One big satellite can be fine (30 cm) or frequent (daily), but not both at low cost.
- **Spatial vs radiometric/spectral**: squeezing 200+ narrow bands onto a detector array leaves less light per band (noisier, coarser GSD). Hyperspectral systems trade swath and GSD for spectral depth.
- **Cost**: sub-metre commercial imagery is priced per km² (tens of US$ per km² for 30 cm class) while Landsat/Sentinel are free (§5.7). Constellations amortise launch cost over many small satellites but add processing complexity.
- **Physics**: diffraction limits how sharp an aperture can be; revisit is limited by orbit mechanics (sun-synchronous = fixed local time, ~daily–16-day revisit; geostationary = continuous but only ~1–4 km GSD at 36,000 km).

### 1.7 History — from Spy Satellites to Open Data

The modern era of remote sensing is a story of declassification and democratisation. **Verified** milestones:

| Year | Milestone | Significance |
|---|---|---|
| 1858 | First aerial photograph (balloon, Paris — Gaspard-Félix Tournachon "Nadar") | Birth of remote sensing as a practice |
| 1914–18 | Aerial reconnaissance in WWI | Military adoption; aerial survey becomes a profession |
| 1960–1972 | **CORONA** US spy-satellite program (first successful film-return mission Aug 1960; declassified 1995) | First operational satellite imaging; ~2 m best resolution on KH-4B; images now freely downloadable |
| 23 Jul 1972 | **Landsat-1** (then ERTS-1) launched | The first civilian Earth-observation satellite; MSS sensor at **80 m**; continuous Landsat archive ever since |
| 1982 | Landsat-4 TM — **30 m** | The 30 m era; still the workhorse scale |
| 1986 | SPOT-1 (France) — **10 m** | First commercial very-high-resolution system |
| 1999 | IKONOS — **1 m** | First commercial sub-metre-class satellite |
| 1999/2002 | Terra and Aqua (MODIS) | Daily global coverage for climate-scale monitoring |
| 2007–08 | WorldView-1 (0.5 m), GeoEye-1 (0.41 m) | Commercial half-metre |
| 2013/2014/2015 | Landsat 8 · Sentinel-1 · Sentinel-2 | Open-data era: free Landsat, then free-and-open Copernicus |
| 2014 | WorldView-3 — **0.31 m (31 cm)** | Commercial 30 cm class (WorldView Legion continues it in 2024+) |
| 2015–17 | Planet Mission 1 completes | Daily 3 m coverage of all land from ~150+ small satellites |

The resolution progression the industry quotes — **80 m → 30 m → 10 m → 1 m → 0.5 m → 0.3 m** — tracks Landsat MSS (1972) → Landsat TM (1982) → SPOT (1986) → IKONOS (1999) → WorldView-1 (2007) → WorldView-3 (2014). **Verified** as the standard telling. The second great shift is *open data*: USGS made the Landsat archive free in 2008, and ESA's Copernicus made Sentinel data free and open since 2014 — together they created the modern analytics industry (§5, §8).

### 1.8 Why Remote Sensing Matters

Remote sensing is the only way to observe the whole Earth systematically, repeatedly, and cheaply per square kilometre. It underpins weather forecasting, climate monitoring, disaster response, agriculture, defence, and increasingly **financial services** (physical climate risk, crop insurance, commodity monitoring — §6.8–6.9). For a bank's perspective: every physical asset a bank finances — a factory, a port, a farm, a residential tower — sits on the Earth's surface, and remote sensing is the technology that lets an institution observe that surface at scale, historically, and going forward.

---

## 2. The Platforms

A remote sensor needs something to sit on. The **platform** determines altitude, coverage, revisit, resolution, and cost — often more than the sensor does.

### 2.1 Satellite Platforms

**Sun-synchronous (polar) orbits** — the workhorse of optical EO. The orbit plane rotates with the Earth's orbit around the sun so the satellite always crosses the equator at the **same local solar time** (Landsat ~10:00–10:30 descending; Sentinel-2 ~10:30), giving **consistent illumination** across passes — essential for comparing images across dates. **Verified.** Altitude ~600–900 km; swath tens to hundreds of km; revisit days to weeks per satellite.

- Landsat 8/9: 705 km, 16-day revisit (each), 185 km swath, 30 m.
- Sentinel-2A/B: 786 km, 10 m (VIS/NIR), 290 km swath, 5-day combined revisit.
- MODIS (Terra 705 km, Aqua 705 km): 2,330 km swath — daily global coverage at 250 m–1 km.

**Geostationary (GEO) orbits** — ~35,786 km over the equator; the satellite matches Earth's rotation and appears fixed, so it views the **same hemisphere continuously**. **Verified.** This is the weather-satellite home: GOES (US), Himawari (Japan), Meteosat (EUMETSAT), Fengyun-4 (China). Full-disk imagery every 10–15 min, but GSD is ~0.5–2 km — too coarse for most land applications. Used for weather, fire detection, lightning, and now some crop/vegetation monitoring (e.g., Himawari/AHI NDVI).

**Constellations** — the defining trend of the 2010s–20s: many small satellites in coordinated orbits to collapse revisit time. Planet operates **~200+ PlanetScope "Dove" cubesats** in sun-synchronous orbits, achieving **daily** 3 m multispectral coverage of all land. **Verified** (daily repeat cycle; fleet size fluctuates as flocks are launched and deorbited). Others: Spire (GNSS-RO), ICEYE/Capella (SAR), SkySat (~15 sats, 50 cm, sub-daily tasking + video). The physics tradeoff: many small sats = daily revisit but per-satellite capability is modest.

### 2.2 Aerial Platforms (Aircraft and Drones)

- **Manned aircraft** — the classic aerial survey platform: large-format cameras and LiDAR flown on demand, at ~1–8 km altitude, producing centimetre-to-decimetre orthophotos and point clouds. Still the standard for high-precision engineering mapping, cadastre, and corridor surveys. Flexible timing (fly when you need it), but expensive per km² and weather-limited.
- **Drones / UAVs (Unmanned Aerial Vehicles)** — "drone remote sensing" (consumer-to-commercial, e.g., DJI) is the fastest-growing tier. **Verified** as a mature practice: small aircraft fly 50–400 m AGL, carrying RGB/multispectral/thermal cameras or compact LiDAR, yielding **centimetre GSD**, on-demand revisit (fly daily), and very low cost per mission. Limits: small area per flight, legal restrictions (visual line of sight, airspace, beyond-visual-line-of-sight rules), battery endurance, and no cloud advantage (they fly *under* the clouds — which is often the point). Typical in precision agriculture: a multispectral drone maps a field at ~5 cm, generates NDVI, and drives variable-rate fertiliser prescription.

| Platform | Typical altitude | GSD | Revisit | Typical swath/area |
|---|---|---|---|---|
| Drone | 50–400 m | 1–10 cm | On demand | < 1–5 km²/flight |
| Manned aircraft | 1–8 km | 5–50 cm | On demand | 100–10,000 km²/day |
| GEO satellite | 36,000 km | 0.5–2 km | 10–15 min | Hemisphere |
| LEO satellite | 600–900 km | 0.3–30 m | 1–16 days | 10–300 km swath |

### 2.3 Ground-Based ("Near-Surface") Sensing

The ground tier of remote sensing is often forgotten but anchors everything above it:

- **Terrestrial LiDAR** — tripod/vehicle-mounted laser scanners producing millimetre-resolution point clouds of a scene (buildings, slopes, forests). Used for change monitoring (landslides, construction), forestry inventory, and calibration/validation of satellite products.
- **Flux towers** (eddy-covariance towers) — measure actual CO₂/H₂O/energy exchange between land and atmosphere; they are the *ground truth* against which satellite vegetation and carbon products (NDVI, GPP) are calibrated.
- **Phenocams, radiometers, weather stations** — automated cameras and spectrometers observing a fixed plot; continuous *in situ* data to validate the daily satellite pass.
- **GNSS ground stations** — used to validate SAR InSAR displacements (§3.5) and to correct satellite orbits.

This "near-surface" tier is the calibration and validation (cal/val) backbone: **satellite products are only as good as the ground truth they were tuned against**.

### 2.4 Platform Comparison Table

| Platform | Altitude | Coverage | Typical resolution | Revisit | Relative cost | Best for |
|---|---|---|---|---|---|---|
| **Geostationary satellite** | 35,786 km | Hemisphere/continent | 0.5–2 km | Continuous (10–15 min) | High (govt programs) | Weather, fire, convection, diurnal vegetation |
| **Sun-sync satellite (govt open data)** | 600–800 km | Global, systematic | 10–30 m (250 m–1 km for MODIS) | 5–16 days (daily for MODIS) | Free data (program cost borne by agencies) | Land cover, agriculture, climate, disaster |
| **Constellation (small sats)** | 400–600 km | Global, daily | 3–5 m (0.3–1 m for SAR/video tiers) | Daily–sub-daily | Commercial subscription | Rapid change, agriculture, security, insurance |
| **Very-high-res satellite** | 400–700 km | Global, taskable | 0.3–1.5 m | On-demand (1–3 days tasking) | High per km² | Defence, engineering, urban, disputes |
| **Manned aircraft** | 1–8 km | Regional, on demand | 5–50 cm | On demand | High per km² | Cadastre, engineering, orthophoto programs |
| **Drone/UAV** | 50–400 m | Local | 1–10 cm | On demand (daily if needed) | Low per mission | Precision ag, inspection, small-area mapping |
| **Ground-based (LiDAR, towers)** | 0–100 m | Point/site | mm–cm | Continuous | Low–medium | Cal/val, forestry plots, slope monitoring |

---

## 3. The Sensor Types

Sensors are the heart of remote sensing: they convert electromagnetic radiation into digital measurements. Five families matter in practice.

### 3.1 Optical Multispectral

The workhorse. A **multispectral** imager records a handful (4–13) of relatively broad bands across the visible–SWIR range, typically including **RGB + NIR** (the classic "false colour" combo: vegetation appears red). **Verified** examples:

- **Landsat 8/9 OLI**: 9 bands — coastal aerosol, blue, green, red, NIR, two SWIR, panchromatic (15 m), cirrus — at **30 m** (multispectral), 16-day revisit each, free.
- **Sentinel-2 MSI**: **13 bands** at 10 m (B2 blue, B3 green, B4 red, B8 NIR), 20 m (red-edge B5/B6/B7, B8A, SWIR B11/B12), and 60 m (atmospheric bands); 290 km swath; 5-day combined revisit; free. Its **red-edge bands** (B5–B7 at ~705/740/783 nm) make it the default agricultural sensor.

Multispectral is the right tool for indices (NDVI/NDWI/EVI), land-cover classification, and change detection at landscape scale — cheap, systematic, well-understood.

### 3.2 Hyperspectral

A **hyperspectral** (imaging spectroscopy) sensor records **hundreds of narrow, contiguous bands** (~10 nm wide), producing a full reflectance spectrum per pixel — enabling **material identification** rather than just discrimination (mineral mapping, gas detection, plant chemistry). **Verified**:

- **AVIRIS** (NASA airborne, since 1987) — the benchmark airborne imaging spectrometer; 224 bands, ~0.4–2.5 µm.
- **EnMAP** (German, DLR/GFZ) — launched **1 April 2022**; 242 bands (420–2450 nm, ~6.5–10 nm sampling); **30 m** GSD, 30 km swath; ~27-day revisit (4 days off-nadir); data open via DLR. **Verified.**
- **PRISMA** (Italian Space Agency, 2019) — ~237 bands, 30 m, commercial+science.
- GHGSat's methane payloads are narrow-band spectrometers rather than full imaging spectrometers, but they ride the same physics (§8.2).

Hyperspectral costs more, covers less, and needs more processing (dimensionality, atmospheric correction), so it is used where spectra matter: geology/mining, environmental contamination, defence, and as the calibration reference for multispectral algorithms.

### 3.3 Thermal Infrared (TIR)

Thermal sensors measure **emitted heat** (8–14 µm window; 3–5 µm MWIR for very hot targets), not reflected light — so they work at night and detect anything warmer or cooler than its surroundings. **Verified** uses:

- **Fire detection**: MODIS (1 km thermal bands, MOD14/MYD14 active-fire products, ~daily global) and VIIRS (375 m, VNP14IMG) are the global fire-watch standard; geostationary (GOES, Himawari) add 10–15 min hotspot updates.
- **Sea surface temperature (SST)**: MODIS, Sentinel-3 SLSTR, and AVHRR series produce operational global SST.
- **Landsat TIRS**: 100 m thermal bands (8/9: bands 10–11) for local-scale heat — urban heat islands, irrigation scheduling, volcano monitoring, building heat loss.
- **Urban climate**: surface urban heat island (SUHI) studies use TIR + land-cover data.

Thermal caveats: clouds block TIR; emissivity must be known (water ≈ 0.98–0.99, metals/roofs vary wildly); atmospheric water vapour absorbs in the window (corrected with split-window techniques).

### 3.4 SAR — Synthetic Aperture Radar

**SAR** is the active microwave workhorse: the radar illuminates the ground sideways with microwave pulses (C/X/L band), and the sensor synthesises a large virtual antenna from the along-track motion of the platform, achieving **metre-class resolution from orbit**. **Verified** characteristics:

- **All-weather, day/night**: microwaves penetrate clouds, rain, smoke, and darkness — the only optical-class capability that works in tropical cloud belts (and at night).
- **Sensitive to structure and moisture**: backscatter responds to surface roughness, geometry, and dielectric (water) content — different information from reflectance.
- **Speckle**: inherent noise-like pattern; must be averaged (multi-looking/filtering), costing resolution.
- **Geometry**: side-looking — layover, shadow, and foreshortening in mountains; polarimetry (HH/HV/VH/VV) and repeat-pass interferometry add dimensions.

**Verified** missions: **Sentinel-1** (ESA, C-band, launched **April 2014**, 20 m resolution, 12-day repeat with A+B, free); **RADARSAT-2** (Canada, C-band, 3 m spotlight); **ALOS-2/PALSAR-2** (Japan, L-band, deep vegetation penetration); **TerraSAR-X/TanDEM-X** (Germany, X-band, 1 m); commercial **Capella, ICEYE, Umbra** (X-band, 0.3–1 m, constellations). Flood mapping (Sentinel-1 is the global standard — water is a dark, smooth radar target), ship detection (the maritime guide's dark-fleet work, §2.3), deforestation (L-band penetration), ice monitoring, and ground-movement measurement (next).

### 3.5 InSAR — Interferometric SAR

**InSAR** compares the *phase* of two SAR images of the same area (repeat passes, or two antennas on one platform) to measure **millimetre-to-centimetre ground movement** along the line of sight. **Verified**:

- **Subsidence/uplift mapping**: Sentinel-1 time-series InSAR (PS-InSAR/SBAS methods) maps urban subsidence, groundwater extraction, mining, and infrastructure deformation over years, at mm/yr precision (e.g., Jakarta, Mexico City, Venice subsidence studies).
- **Earthquakes**: coseismic displacement maps (e.g., the 2023 Türkiye–Syria earthquakes were mapped with Sentinel-1 within days).
- **Volcanoes and landslides**: inflation/deformation monitoring.
- **DEM generation**: TanDEM-X produced a global 12 m DEM from bistatic InSAR.

Limitations: coherence loss (vegetated areas decorrelate; L-band helps), atmospheric phase delays, and the fact that only the line-of-sight component of motion is measured.

### 3.6 LiDAR — Light Detection and Ranging

**LiDAR** is the active *optical* cousin of radar: it fires rapid laser pulses (typically 1064 nm) and times the returns, building a **point cloud** of 3-D coordinates. **Verified**:

- **Airborne/terrestrial LiDAR**: centimetre-accurate elevation, building models, powerline corridors, and **canopy structure** — multiple returns per pulse see the top of the canopy, branches, and the ground, so forest height and biomass can be measured directly.
- **Spaceborne**: ICESat-2 (photon-counting, 2018) and **GEDI** (ISS, 2018–2023) sample global vegetation height/biomass; GEDI's samples calibrate the radar/SAR biomass missions.
- **Bathymetric LiDAR** (green 532 nm) maps shallow coastal water.

LiDAR gives *structure* (elevation, height) where optical gives *colour*; the two are complementary, and fused point-cloud + imagery products are standard in modern mapping.

### 3.7 Sensor Comparison Table

| Sensor | Spectrum | Active/Passive | Typical resolution | Strengths | Limitations | Use cases |
|---|---|---|---|---|---|---|
| **Multispectral** (Landsat, Sentinel-2) | VIS–SWIR, 4–13 bands | Passive | 10–30 m | Systematic, free, indices, land cover | Clouds, daylight only | Agriculture, land cover, change detection |
| **Hyperspectral** (EnMAP, AVIRIS, PRISMA) | VIS–SWIR, 200+ bands | Passive | 30 m (airborne: cm–m) | Material identification | Narrow swath, cost, processing, clouds | Minerals, contamination, defence, plant chemistry |
| **Thermal TIR** (MODIS, Landsat TIRS, SLSTR) | 3–5 / 8–14 µm | Passive (emitted heat) | 100 m–1 km | Heat detection, night, fires, SST | Clouds, emissivity, coarse GSD | Fires, SST, urban heat, drought |
| **SAR** (Sentinel-1, RADARSAT, ALOS-2) | Microwave (X/C/L) | Active | 1–30 m | All-weather day/night, moisture, structure | Speckle, geometry distortions, cost | Floods, ships, deforestation, ice, agriculture |
| **InSAR** (Sentinel-1 time series) | Microwave | Active | 5–30 m (mm displacement) | mm-cm ground motion | Coherence, atmospheric phase, line-of-sight only | Subsidence, earthquakes, volcanoes, dams |
| **LiDAR** (airborne, GEDI, ICESat-2) | Optical laser | Active | cm–m (point clouds) | Direct 3-D structure, elevation | Cost per km², no canopy penetration to ground in dense forest (partial), clouds for airborne | DEM, forestry, powerlines, flood modelling |

---

## 4. Data Processing

Raw satellite data is a stream of digital numbers (DNs) with instrument artefacts, distorted geometry, and atmosphere in the way. The processing chain turns it into information. The canonical pipeline:

```
 RAW DATA (L0/L1)                 CORRECTED IMAGE                DERIVED PRODUCTS               ANALYSIS
 +----------------+   +-----------------------------+   +--------------------------+   +---------------------+
 | radiometric DN |-->| radiometric correction      |-->| spectral indices (NDVI)  |-->| time series / change |
 | + metadata     |   | geometric correction        |   | classification (LULC)    |   | anomaly detection   |
 | + orbit/att.   |   | orthorectification (DEM)    |   | biophysical params (LAI) |   | statistics, maps,   |
 +----------------+   | atmospheric correction      |   | (per-date products)     |   | reports, decisions  |
                      | cloud masking               |   +--------------------------+   +---------------------+
                      +-----------------------------+
```

### 4.1 Radiometric Correction

Converts raw DNs to physically meaningful units — top-of-atmosphere **reflectance** or **radiance** — using sensor calibration coefficients (gains/offsets measured on the ground and updated in flight), and normalises for **solar illumination** (sun angle, Earth–sun distance). Without it, two images taken on different dates look different even if the surface didn't change, because the sun was lower in the sky. (For SAR: radiometric calibration converts backscatter power to sigma-nought σ⁰.)

### 4.2 Geometric Correction and Orthorectification

- **Geometric correction / georeferencing** — assigns map coordinates (e.g., UTM, Web Mercator) to pixels using the platform ephemeris and attitude, correcting basic distortions. Result: the image overlays a map approximately.
- **Orthorectification** — the harder, important step: removes **terrain-induced displacement** (in mountains, off-nadir pixels are displaced by many pixels) by reprojecting each pixel through a **Digital Elevation Model (DEM)**, producing a true, scale-correct, "looking straight down" image (an *orthoimage*). **Verified** as standard practice: any pixel-accurate analysis (change detection between dates, overlay with GIS layers, sub-pixel work) requires orthorectified data. Landsat/Sentinel L1C+ products are already orthorectified; drone/aircraft photogrammetry needs it done in the processing software.

### 4.3 Atmospheric Correction

The atmosphere scatters and absorbs light between the surface and the sensor (aerosols, water vapour, haze): a dark forest can look hazy grey, and NIR values inflate. **Atmospheric correction** estimates and removes these effects to recover **surface reflectance**. **Verified** methods: physically based radiative-transfer models (**6S, MODTRAN**) and their operational wrappers (**Sen2Cor** for Sentinel-2, LaSRC for Landsat); simpler empirical alternatives (Dark Object Subtraction — DOS — assuming the darkest pixels are near-zero reflectance); and the growing use of aerosol/water-vapour bands measured by the sensor itself (Sentinel-2's 60 m bands 1, 9, 10). For thermal data, atmospheric correction removes water-vapour absorption using split-window techniques. For SAR, the atmospheric correction problem is inverted — the *ionosphere/troposphere delay* is the error source that InSAR must model away (§3.5).

### 4.4 Cloud Masking

Optical sensors cannot see through clouds, and clouds are the #1 operational nuisance (tropical regions are cloud-covered most of the time). **Cloud masking** detects cloudy pixels (brightness, whiteness, temperature — clouds are cold; plus cirrus-band detection at 1.38 µm) and removes or flags them before analysis. **Verified** tools: **FMask** (the standard for Landsat/Sentinel-2, returns cloud/cloud-shadow/water masks), **s2cloudless** (Sentinel-2, deep-learning based), and the scene-classification layers shipped with Sentinel-2 L2A. Time-series methods then composite the *clear* pixels across dates (e.g., median composites in Google Earth Engine) — the practical answer to the cloud problem.

### 4.5 Spectral Indices and Band Math

Indices are arithmetic combinations of bands that amplify a target property and cancel noise (illumination, atmosphere, soil background). The canonical family:

- **NDVI — Normalized Difference Vegetation Index**. The most-used index in remote sensing:
  ```
  NDVI = (NIR − Red) / (NIR + Red)
  ```
  **Verified** (standard formula; Rouse et al. 1973). Healthy vegetation: high NIR, low red → NDVI ≈ 0.6–0.9; sparse/stressed vegetation: lower; bare soil ≈ 0.1–0.2; water ≈ negative. NDVI saturates in dense canopy — hence **EVI** (Enhanced Vegetation Index), which adds the blue band to correct aerosol and decouple the soil background:
  ```
  EVI = 2.5 × (NIR − Red) / (NIR + 6×Red − 7.5×Blue + 1)
  ```
  **Verified** (standard formulation; Huete et al. 2002).
- **NDWI — Normalized Difference Water Index** (McFeeters 1996):
  ```
  NDWI = (Green − NIR) / (Green + NIR)
  ```
  Water reflects green and absorbs NIR → strongly positive for open water; used for flood mapping and water-body change. (A related "NDWI" variant uses SWIR for vegetation water content — the literature is inconsistent; state which one you mean.)
- **Others**: NDMI (moisture), NBR (burn severity, fire), BSI (bare soil), MNDWI (modified water index using SWIR), SAVI (soil-adjusted vegetation index).
- **Band math** generally: any per-pixel combination of bands — the basis of custom indices, ratios, and feature engineering for ML.

### 4.6 Classification — Turning Pixels into Categories

Classifying imagery into land-cover/land-use categories (crop, forest, urban, water, bare) is the core "understanding" step. Four families:

- **Unsupervised**: the algorithm discovers clusters in spectral space with no training labels — **k-means** and **ISODATA** (the classic; iteratively splits/merges clusters). Fast, label-free, useful for exploration and for generating training data; cluster *labels* must be assigned by an analyst afterwards.
- **Supervised**: the analyst provides **training data** (labelled polygons/pixels, from field surveys, photo-interpretation, or existing maps); a classifier learns the spectral (and textural) signatures. Classic statistical: maximum likelihood. Modern ML workhorses: **Random Forest** and **SVM (Support Vector Machines)** — **Verified** as the standard off-the-shelf classifiers in remote sensing (RF especially: robust, handles many features, out-of-bag error estimate). Libraries: scikit-learn, and EO-specific (Google Earth Engine's classifiers, SNAP, Orfeo ToolBox).
- **Object-based (OBIA — Object-Based Image Analysis)**: first *segments* the image into homogeneous objects (e.g., eCognition's multiresolution segmentation, or simple linear iterative clustering — SLIC), then classifies the objects using spectral *plus* shape/context/texture features. **Verified** practice: OBIA typically beats per-pixel classifiers on very-high-resolution imagery (where a single tree crowns many pixels) and reduces the salt-and-pepper noise of per-pixel classification.
- **Deep learning**: **CNNs** and transformer models do **semantic segmentation** — per-pixel classification with learned spatial context — with U-Net-style architectures as the remote-sensing standard. **Verified** as the state of the art on benchmark land-cover tasks and object detection (buildings, ships, cars; e.g., xView, SpaceNet datasets). Data hungry and compute hungry, but the 2020s default for high-resolution work; see the `ai_llm/` series for the model engineering ([deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md)) and evaluation discipline ([llm_evaluation_vs_validation_guide.md](ai_llm/llm_evaluation_vs_validation_guide.md)).

Accuracy assessment is non-negotiable: a confusion matrix against *held-out* reference samples, with overall accuracy, user's/producer's accuracy, and Cohen's kappa, is the standard reporting (and the same train/validate/test discipline as ML elsewhere).

### 4.7 Time Series and Change Detection

Remote sensing's unique asset is **history**: the archive lets you analyse *change*. Common methods:

- **NDVI time series** — the vegetation trend backbone: monthly/16-day composites (MODIS MOD13, Landsat-derived) fitted with smoothing (Savitzky–Golay, harmonic/seasonal-trend models) to reveal greening/browning trends, phenology (start/peak/end of season), and **anomalies** (this year's NDVI vs. the long-term mean, z-score style).
- **Change detection** — bitemporal differencing (NDVI difference, image differencing, change vector analysis), post-classification comparison (map 2015 vs map 2025), and sophisticated breakpoint algorithms (**BFAST**, **LandTrendr**) that detect abrupt changes (deforestation, fire, urbanisation) inside the time series.
- **Radar time series** — Sentinel-1 backscatter time series for flood extent, rice paddies (the unique double-bounce/flood signature), and InSAR displacement series (§3.5).

### 4.8 Processing Tools

| Tool | What it is | Notes |
|---|---|---|
| **GDAL** | The open-source geospatial data library (raster/vector I/O, reprojection, warping) | The foundation under nearly everything else; `gdalwarp`, `gdal_translate`, `gdal_calc.py` |
| **QGIS** | Free desktop GIS | Full analysis/visualisation; the default open desktop |
| **SNAP** (ESA) | Sentinel Application Platform | The official Sentinel toolbox (SAR processing, InSAR, ocean) |
| **Google Earth Engine (GEE)** | **Cloud** planetary-scale analysis platform: petabyte archive (Landsat, Sentinel, MODIS, etc.) + server-side computation, JS/Python API | **Verified**: free for research/education, commercial licensing for production; the default for time-series analysis at scale; can process the whole Landsat archive in minutes |
| **Sentinel Hub** (Sinergise) | **Cloud** API for on-demand Sentinel/Landsat tiles and composites (WMS/WMTS/WCS) | **Verified**: fast pixel-level access; free tier, commercial plans; powers many EO apps |
| **Copernicus Data Space / AWS Open Data** | Direct archive access (S3 buckets, STAC metadata) | The data-first path for engineering teams (§5.6) |
| **Orfeo ToolBox / eCognition / ArcGIS** | Open and commercial processing/OBIA suites | OBIA and production workflows |
| **Python stack** | rasterio, xarray, rioxarray, geopandas, scikit-learn, torchgeo | The modern data-science way to do everything above |

For an engineering team the pattern is: **STAC catalogue → object storage (AWS S3) → GDAL/xarray processing → PostGIS/geoparquet → ML (torchgeo/scikit-learn) → visualisation (QGIS/MapLibre)**. See [data/data_pipeline_guide.md](data/data_pipeline_guide.md) for the pipeline engineering.

---

## 5. Data Sources and Platforms

### 5.1 NASA / USGS — Landsat and MODIS

- **Landsat** (USGS/NASA) — the longest continuous civilian EO record (since 1972). **Landsat 8** (2013) and **Landsat 9** (Sep 2021) carry OLI (30 m, 9 bands) + TIRS (100 m thermal). 16-day revisit each, 8-day combined. **Free and open** since 2008 — the entire 50-year archive is downloadable. **Verified.**
- **MODIS** (NASA) — on **Terra** (1999) and **Aqua** (2002); 36 bands at **250 m** (red/NIR), 500 m, and **1 km**; 2,330 km swath → **daily global coverage**; the source of global NDVI/EVI, land cover (MCD12Q1), active fire (MOD14/MYD14), SST, and snow products. **Verified.**
- Also: **VIIRS** (Suomi NPP/NOAA-20/21, 375 m, day/night band — night lights), **Harmonized Landsat Sentinel (HLS)** — NASA's fused Landsat+Sentinel-2 product at 30 m with 2–3 day revisit, the foundation for NASA's Prithvi foundation model (§8.3).

### 5.2 ESA Copernicus — the Sentinel Family

The **Copernicus** program (ESA/EU) is the free-and-open benchmark. **Verified**:

- **Sentinel-1** — C-band SAR, **launched April 2014** (1A; 1B 2016–2022); 20 m (IW mode), 12-day repeat (A+B), all-weather day/night; the global flood/subsidence/ship-detection workhorse. **Verified.**
- **Sentinel-2** — high-resolution optical, **launched June 2015** (2A; 2B 2017); 13 bands, **10 m** (VIS/NIR), 5-day combined revisit; the agricultural/land-cover default. **Verified.**
- **Sentinel-3** — ocean and land: ocean colour, **sea surface temperature** (SLSTR), radar altimetry (SRAL); launched Feb 2016; ~300 m–1 km; the marine/ocean-climate workhorse (cross-ref the maritime guide's "blue data", §3.5). **Verified.**
- **Sentinel-5P** (2017) — atmospheric trace gases incl. **methane** (TROPOMI, 7 km) for wide-area methane screening.
- **Copernicus data policy**: free, full, and open access for all users — the 2014 regulation that seeded the EO analytics industry. **Verified.**

### 5.3 Commercial Very-High-Resolution (VHR)

- **Maxar** (US) — the sub-metre leader: **WorldView-1/2/3** (0.31–0.5 m), GeoEye-1; **WorldView Legion** (2024+, six-sat constellation, 30 cm); 8-band multispectral; taskable, ~daily revisit at high latitudes; priced per km². **Verified** (30 cm class; exact pricing is commercial).
- **Planet** (US) — the daily-revisit pioneer: **PlanetScope** (~200+ Doves, 3 m, daily, 4-band); **SkySat** (~15 sats, **50 cm**, sub-daily tasking, **video** capability — up to ~2-minute panchromatic video clips, useful for moving-target monitoring). **Verified.**
- **Airbus** (EU) — **SPOT 6/7** (1.5 m), **Pléiades 1A/1B** (50 cm, 2011/12), and **Pléiades Neo** (four sats, 2021–22, **30 cm**, 6-band incl. red-edge). **Verified.**

### 5.4 Chinese EO

- **Gaofen (高分, "high resolution")** — the state **CHEOS** program (civilian): **GF-2** (2014) 0.8 m pan / 3.2 m MS; **GF-7** (2019) 0.65 m stereo for 3-D mapping; GF-3 C-band SAR; GF-4 geostationary optical; GF-6 agricultural. **Verified.** So the popular "0.5 m" figure applies to the *commercial* tier rather than Gaofen proper.
- **Commercial Chinese**: **Jilin-1** (Chang Guang — ~0.5–0.75 m, large constellation), **SuperView-1** (0.5 m), **Beijing-2** — sub-metre commercial options, increasingly used across Asia/Africa.
- **Note on BeiDou**: BeiDou is a **GNSS** (navigation/timing) constellation — the Chinese counterpart of GPS — *not* an Earth-imaging system. It gets conflated with Chinese EO in casual writing; it matters to remote sensing only as a positioning/timing source (and for GNSS-RO atmospheric sensing). The same clarification applies to GPS/Galileo/GLONASS.

### 5.5 Sub-Metre Economics

Sub-metre imagery is a **tasking** business: you order an area, a satellite slews to point at it, and you pay per km². **Flagged — pricing is commercial and negotiable**: indicative ranges reported in the industry are ~US$10–30+/km² for 30–50 cm class archived imagery (more for fresh tasking, less for large-area deals and older archives), vs effectively **US$0** for Landsat/Sentinel/MODIS. The economic logic: open data made the *medium-resolution* layer a commodity; commercial VHR monetises the *last metre* (defence, insurance, disputes, engineering). Analyst estimates for the total EO market cluster around **US$3.7–5.1B in 2024 depending on definition** (one widely cited figure: ~US$2.2B commercial EO data revenue 2024, growing ~7%/yr) — **Flagged**, definitions differ too widely (data vs services vs analytics) to cite a single number responsibly (§11.1).

### 5.6 Data Access — the Engineering View

| Path | What you get | Cost model |
|---|---|---|
| **AWS Open Data** (usgs-landsat, sentinel-s2-l1c/l2a, sentinel-s1, MODIS via NASA) | The full archives as S3 buckets, STAC metadata | Free data; you pay AWS egress/compute — `aws s3 ls s3://sentinel-s2-l2a/...` |
| **Copernicus Data Space Ecosystem** | Sentinel + other EO via API/STAC | Free (registration) |
| **Google Earth Engine** | Analysis-ready data + compute (JS/Python) | Free research/education; commercial via GEE for Earth |
| **NASA Earthdata / LP DAAC / AppEEARS** | NASA/USGS products (MODIS, HLS, VIIRS) | Free |
| **Sentinel Hub / Planet / Maxar APIs** | On-demand tiles, subscriptions, tasking | Subscription / per-km² |
| **STAC + object storage** (self-hosted) | Your own analysis-ready data lake | Infra cost only |

The modern pattern for enterprises: **analysis-ready data (ARD)** — already orthorectified, atmospherically corrected, cloud-masked — consumed via STAC APIs into a data lake, then processed with the §4.8 stack.

### 5.7 Data Economics — Open vs Commercial vs "Data as a Service"

- **Open (Landsat, Sentinel, MODIS)**: free at the point of use; the taxpayer pays. Enables research, startups, and the analytics layer — the entire EO analytics industry sits on Copernicus/Landsat data.
- **Commercial (VHR, daily constellations, tasking)**: priced per km² or by subscription; margins are in *freshness, resolution, and reliability* (contractual revisit guarantees).
- **Data as a Service / analytics platforms**: the fastest-growing layer — you buy *answers*, not pixels: crop-health dashboards (e.g., Descartes Labs, Planet's ag offerings, S&P Global's ag analytics), flood products (Cloud to Street), methane alerts (GHGSat's public "Methane Map", Kayrros), deforestation alerts (Global Forest Watch). The bank/insurer buys a decision-ready product; the remote sensing is invisible behind it. **Verified** as the dominant commercial model of the 2020s.

### 5.8 Data Source Comparison Table

| Source | Sensor/Products | Resolution | Revisit | Cost | Best for |
|---|---|---|---|---|---|
| **Landsat 8/9** (USGS) | 30 m MS + 100 m TIR | 30 m | 8–16 days | Free | Long-term change, thermal, global archive since 1972 |
| **MODIS** (NASA) | 36-band, 250 m–1 km | 250 m–1 km | Daily | Free | Global climate/vegetation/fire products |
| **Sentinel-1** (ESA) | C-band SAR | 20 m | 6–12 days | Free | Floods, subsidence (InSAR), ships, ice |
| **Sentinel-2** (ESA) | 13-band MS | 10/20/60 m | 5 days | Free | Agriculture, land cover, red-edge indices |
| **Sentinel-3** (ESA) | Ocean colour, SST, altimetry | 300 m–1 km | 1–2 days | Free | Ocean, SST, climate |
| **Maxar WorldView/Legion** | VHR optical | 0.3–0.5 m | Taskable | $$$ per km² | Defence, engineering, disputes |
| **PlanetScope** | Small-sat MS | 3 m | Daily | $$ subscription | Rapid change, agriculture, insurance |
| **SkySat** | VHR + video | 0.5 m | Sub-daily | $$$ | Moving targets, events |
| **Pléiades Neo / SPOT** (Airbus) | VHR MS | 0.3 / 1.5 m | Taskable | $$$ | Urban, engineering, red-edge VHR |
| **Gaofen / Jilin-1** (China) | VHR MS | 0.5–0.8 m | Taskable | $$ | Regional programs, Asia/Africa coverage |
| **GHGSat** | Methane spectrometers | ~25 m (plume) | Daily–weekly tasking | Service | Methane point-source detection (§8.5) |
| **ICESat-2 / GEDI** | Space LiDAR | ~10–30 m footprints | Sampling | Free | Vegetation height, elevation, biomass |

---

## 6. The Applications

### 6.1 Agriculture — the Flagship Market

Agriculture is the largest civilian remote-sensing market, and **NDVI crop monitoring** (§7) is its emblem. Verified, standard practice:

- **Crop health**: NDVI/EVI/red-edge time series track greenness; anomalies flag stress (water, nutrients, pests) weeks before the eye can see it.
- **Yield prediction**: empirical models (NDVI/LAI at critical growth stages vs historical yields) and process models (assimilation of LAI/biomass into crop models) produce regional yield forecasts — the basis of the USDA/MARS-type bulletins and commodity analytics (§6.10).
- **Precision agriculture**: field-level (Sentinel-2) and sub-field-level (drone, 5 cm) maps drive **variable-rate fertiliser/irrigation** — the classic ROI case (inputs down 5–20%, yield up, per industry case studies — **Reported**, varies by crop/region).
- **Rice monitoring** (Southeast Asia): Sentinel-1 **SAR** sees rice paddies through clouds and detects the flooded-phase double-bounce signature — the practical backbone of regional rice systems (Vietnam's Mekong Delta, India's kharif monitoring).

### 6.2 Forestry

- **Deforestation detection**: Landsat time series + breakpoint algorithms (LandTrendr) power **Global Forest Watch**'s annual tree-cover loss alerts — the de facto global standard. **Verified** (GFW is a WRI-led open platform).
- **REDD+ / carbon**: forest carbon stock estimation from LiDAR (GEDI) + SAR (L-band biomass) + optical; UN-REDD MRV (monitoring, reporting, verification) is a remote-sensing institutional framework.
- **Fire**: burn scars via NBR (dNBR = burn severity), active fires via MODIS/VIIRS/geostationary (§3.3).
- **Illegal logging and concessions**: SAR penetration + VHR change detection.

### 6.3 Urban / Land Use

- **Land-use/land-cover (LULC) classification**: the global land-cover products (ESA WorldCover 10 m, Esri Sentinel-2 10 m, MODIS MCD12Q1) are all remote-sensing outputs.
- **Urban growth**: multi-decadal Landsat time series + night lights (VIIRS DNB) map sprawl — used for planning and, increasingly, for *physical-risk exposure* (where did the city grow? into floodplains?).
- **Utilities and infrastructure**: building footprints from VHR (xView/SpaceNet-style deep learning), powerline/pipeline corridor monitoring, roof-area estimation for solar potential and insurance exposure.

### 6.4 Disasters

- **Floods**: **Sentinel-1 SAR flood mapping** is the global standard — water is dark and smooth in SAR backscatter, clouds don't matter, and emergency agencies task it within hours of an event. **Verified** (the Copernicus Emergency Management Service — CEMS — Rapid Mapping uses Sentinel-1/2 systematically). SAR flood extent + DEM → flood depth → exposure analytics for insurers and banks.
- **Fires**: MODIS/VIIRS active-fire detections and FRP (fire radiative power); geostationary rapid refresh; burned-area products (MODIS MCD64, Sentinel-2). Operational worldwide (FIRMS — Fire Information for Resource Management System). **Verified.**
- **Earthquakes**: **InSAR** displacement maps (e.g., 2023 Türkiye–Syria: Sentinel-1 coseismic interferograms within days), damage assessment from VHR before/after, night-light drop detection. **Verified.**
- **Volcanoes**: thermal anomalies (hotspots), InSAR inflation/deformation, ash-plume tracking (geostationary + TROPOMI SO₂).
- **Cyclones/typhoons**: storm track and intensity from geostationary and microwave imagers; pre-event exposure mapping for parametric triggers (§6.8).

### 6.5 Ocean and Coastal

- **Sea surface temperature**: MODIS/Sentinel-3/AVHRR operational SST — fisheries, climate, coral bleaching warnings.
- **Oil spills**: SAR slick detection (slicks damp the radar backscatter → dark patches) via **EMSA CleanSeaNet** (Copernicus-based, operational since 2007) and commercial services — **cross-reference [maritime_domain_awareness_guide.md](maritime_domain_awareness_guide.md) §6.3 and §2.3** for the full maritime treatment (ship detection, dark-fleet, AIS fusion).
- **Ocean colour**: chlorophyll-a from Sentinel-3/OLCI — phytoplankton blooms, harmful algal blooms, water quality.
- **Sea level**: radar altimetry (Sentinel-6, Jason-3) — global mean sea level at ~3–4 mm/yr and rising (IPCC; **Verified** trend direction, exact rate varies by record).
- **Coastal change**: shoreline mapping, bathymetry (green LiDAR), mangrove monitoring.

### 6.6 Climate

- **Ice**: sea-ice extent (passive microwave SSMIS, daily), glacier mass balance (altimetry, DEM differencing), ice-sheet velocity (InSAR).
- **Glaciers/permafrost**: decadal Landsat glacier inventories; InSAR permafrost subsidence.
- **Carbon**: GEDI/ICESat-2 biomass, FLUXNET-calibrated GPP products (MODIS), and the coming **CO2M** constellation (ESA, ~2026) for CO₂ column monitoring.
- **The climate role of EO**: essentially every IPCC physical-climate line of evidence — temperature, ice, sea level, vegetation — is a remote-sensing measurement. **Verified** (IPCC AR6 relies on EO for the cryosphere, land cover, and ocean chapters).

### 6.7 Defence and Intelligence

The original remote-sensing customer. Military EO uses: VHR tasking for target characterisation, SAR for all-weather surveillance and GMTI-style moving-target tracking, change detection for activity monitoring, hyperspectral for material/target identification, night lights for economic activity. Commercial imagery now rivals early national systems — hence the 2014+ rise of commercial defence contracts (NRO's "Electro-Optical Commercial Layer" buys 30 cm commercial imagery). **Verified** (public procurement records; capability details classified).

### 6.8 Insurance — Crop and Parametric

- **Crop insurance**: satellite NDVI/yield estimates validate claims and price multi-peril crop insurance in regions without field adjusters; index-based products pay on NDVI/yield-index thresholds. **Verified** as a mature market (India's PMFBY, African Risk Capacity — ARC, ~35+ African countries use satellite-based drought index insurance).
- **Parametric (index-based) insurance**: payouts triggered by *measured indices* — flood extent/depth (SAR), cyclone wind (model+satellite), rainfall (satellite rainfall products like CHIRPS/GPM) — rather than assessed loss. Fast payouts (days), no loss adjustment, basis risk as the tradeoff. **Verified** practice; growth driven by climate-risk finance in EM/Asia.
- **Property**: building footprints + flood models + subsidence (InSAR) feed exposure and pricing; wildfire exposure from fuel/land-cover + history.

### 6.9 Banking — Climate Risk, ESG, Commodities

Remote sensing is quietly becoming a bank-grade data source (see [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) for the risk-governance context):

- **Physical climate risk**: flood, fire, heat, and drought exposure of **collateral** (mortgages, CRE, project finance) computed from EO-derived hazard layers + asset location. Regulators are pushing this: ECB climate stress tests (2022), HKMA/MAS climate disclosures, and IFRS S2 (2024+) all require physical-risk assessment — EO is the cheapest systematic input. **Verified** (regulatory direction); methodologies vary.
- **ESG**: deforestation-free supply-chain screening (soy, palm, cattle — Global Forest Watch-style alerts), water stress, biodiversity indicators (habitat proxies), methane super-emitter screening of financed oil and gas assets (§8.5). Banks' financed-emissions (PCAF) work increasingly uses EO for land-sector emissions.
- **Commodities**: crop supply monitoring (USDA/MARS-style yield forecasts from NDVI) informs ag-commodity trading and trade-finance credit decisions; the maritime guide's vessel-level tracking (§6.6 of that guide) covers the logistics leg. EO is now an input to sell-side and trading desks' fundamental models. **Verified** as industry practice (e.g., commodity analysts subscribe to satellite crop analytics).
- **Basis risk and governance caveat**: EO-derived indicators are *estimates* — banks must treat them as model inputs subject to the model-risk discipline of the banking guides, not as ground truth.

### 6.10 Applications Table

| Application | Primary sensors | Data used | Typical analysis | Stakeholders |
|---|---|---|---|---|
| Crop monitoring / yield | Sentinel-2, MODIS, SAR (rice) | NDVI/EVI/LAI time series, SAR backscatter | Greenness anomalies, phenology, yield models | Farmers, agribusiness, insurers, commodity desks, governments |
| Precision agriculture | Drones, Sentinel-2 | cm–10 m multispectral | Variable-rate maps, NDVI prescription | Farmers, ag-tech |
| Deforestation | Landsat, Sentinel-1/2, LiDAR | Time series, biomass | Breakpoint detection, carbon accounting | Governments, NGOs, banks (ESG), FMCG supply chains |
| Urban / land use | Landsat, Sentinel-2, VHR | LULC, night lights | Classification, growth analysis | Planners, insurers, property |
| Flood mapping | Sentinel-1 (SAR), Sentinel-2 | Backscatter/NDWI, DEM | Extent, depth, exposure | Emergency services, insurers, banks, reinsurers |
| Fire detection | MODIS, VIIRS, geostationary | Thermal hotspots, burned area | Active-fire alerts, severity (dNBR) | Fire services, insurers, climate |
| Earthquakes / subsidence | Sentinel-1 (InSAR) | Phase time series | Displacement maps, damage proxy | Geotech, insurers, governments |
| Ocean (SST, spills, colour) | Sentinel-3, MODIS, SAR | SST, chlorophyll, slicks | Marine monitoring (see maritime guide) | Maritime agencies, fisheries, shipping, insurers |
| Climate (ice, sea level, carbon) | Altimetry, SAR, LiDAR, MODIS | Ice extent, elevation, biomass | Trend analysis, IPCC evidence | Climate science, policy, finance |
| Defence / intelligence | VHR, SAR, hyperspectral | Tasked imagery | Change detection, target ID | Militaries, intelligence |
| Crop and parametric insurance | Sentinel-1/2, rainfall sats | Indices (NDVI, flood, rain) | Index triggers, payouts | Insurers, reinsurers, governments |
| Banking (climate risk, ESG, commodities) | Sentinel, MODIS, VHR | Hazard + exposure + supply chain | Physical-risk scoring, screening, yield forecasts | Banks, asset managers, regulators |

---

## 7. Worked Example — NDVI Rice Crop Monitoring

### 7.1 Scenario

You are a solution architect at a bank with an agri-lending portfolio in **the Mekong Delta (Vietnam)** — rice country, monsoon climate, clouds most of the growing season. The credit team needs **seasonal crop-condition monitoring** to (a) detect a bad season early (drought, flood, pest) before it hits loan performance, and (b) feed a satellite-indexed credit enhancement pilot. Deliverable: a quarterly crop-condition dashboard over the bank's lending districts.

### 7.2 The Data

| Choice | Why |
|---|---|
| **Sentinel-2** (L2A, 10 m, free) | 10 m resolution resolves individual paddy parcels; red-edge bands (B5–B7) track rice phenology; 5-day revisit; free and open |
| **Sentinel-1** (C-band SAR) as complement | The monsoon clouds will obliterate S2 for weeks at a time — S1's all-weather backscatter sees through; rice has a distinctive flooded-phase signature |
| **MODIS** (250 m, 16-day NDVI composite) as the long-history baseline | The 20+ year archive defines the "normal" phenology curve for anomaly scoring |
| **Administrative boundaries + field polygons** | The bank's loan parcels / districts as the analysis units (AOIs) |

### 7.3 The Processing

1. **Ingest**: pull S2 L2A tiles (already orthorectified + atmospherically corrected via Sen2Cor) from the Copernicus Data Space / AWS Open Data; join with the AOI polygons.
2. **Cloud mask**: apply the S2 scene-classification layer / s2cloudless; keep only clear pixels; per-date cloud-free coverage maps.
3. **NDVI calculation** (per clear date, per pixel):
   ```
   NDVI = (B8 − B4) / (B8 + B4)      # S2: B8 = NIR (10 m), B4 = Red (10 m)
   ```
   Also compute **EVI** for dense-canopy robustness and, for the flooded-rice detection, the S1 backscatter threshold (σ⁰_VH below ~−18 dB during transplant = flooded field).
4. **Composite**: monthly median composites of clear pixels (clouds are the #1 data-quality fight — §7.7).
5. **Time series**: per AOI, extract the NDVI curve and align it to the crop calendar (sowing → tillering → panicle → maturity → harvest). A healthy Mekong rice cycle shows NDVI rising from ~0.2 (bare/flooded, post-sowing) to a plateau ~0.7–0.9 at panicle stage, then dropping at harvest.

### 7.4 The Analysis

- **Anomaly score**: for each AOI and date, compare this season's NDVI against the MODIS-based 20-year mean ± standard deviation:
  ```
  z = (NDVI_now − NDVI_mean) / NDVI_std   # by dekad/week of the season
  ```
  z < −1.5 at a critical stage → "stressed" flag; z < −2.5 → "crop failure watch".
- **Phenology deviation**: delay in the green-up (transplant date shift) detected from the NDVI trajectory → late sowing, likely flood or water shortage.
- **Flood events**: SAR-based water masks during the season; flooded rice at transplant stage is normal (paddy), but flooding *after* panicle stage is crop damage — distinguish by stage-aware rules (a simple decision table; a CEP-style rule engine would do this — see [complex_event_processing_guide.md](complex_event_processing_guide.md) §6–7).
- **Yield estimate**: season-peak NDVI / greenness-duration vs historical district yields → district yield forecast (a linear/random-forest model, cross-validated on 10+ years of official yields). **Reported** accuracy: ±5–10% at district level in good-data years — treat as indicative, and validate against the bank's actual repayment data.

### 7.5 The Application

- **Dashboard per district**: NDVI curve vs normal band, anomaly z-scores, flood flags, forecast yield vs last year, per-loan-parcel condition class (Good / Watch / Stress / Failure).
- **Credit workflow**: Stress/Failure flags trigger credit-risk review and, for the parametric pilot, automatic index-payout eligibility (a *satellite-triggered parametric product* — the §6.8 model).
- **Early warning**: a Mekong-wide flood in the panicle stage of the main crop is detectable within 3–5 days of the event (SAR) — days to weeks before field-level damage assessment.

### 7.6 The Worked Flow

```
 Sentinel-2 L2A  ──►  cloud mask (SCL/s2cloudless)
 Sentinel-1 SAR  ──►  flood mask (σ⁰_VH threshold)      ──►  monthly clear composites
 MODIS archive   ──►  long-term NDVI mean/std (by stage)
                                   │
                                   ▼
                        NDVI = (B8−B4)/(B8+B4)
                                   │
              ┌────────────────────┼─────────────────────┐
              ▼                    ▼                     ▼
        per-parcel curve     z-anomaly score        phenology shift
              │                    │                     │
              └─────────┬──────────┴──────────┬──────────┘
                        ▼                     ▼
              district yield forecast   condition flags
                        │                     │
                        ▼                     ▼
              credit-review trigger ──►  parametric payout index
```

### 7.7 Challenges — the Honest List

- **Clouds**: S2 sees a Mekong field ~10–25% of the time during monsoon (a **Reported** order-of-magnitude figure; it varies by year and district). Mitigation: monthly composites, SAR fusion, and accepting coarser temporal detail in wet seasons.
- **Revisit vs growth stage**: a 5-day S2 revisit + clouds can still miss a 10-day critical window; the parametric product must define its trigger on data that actually exists.
- **NDVI saturation** in dense canopies (use EVI); **soil/water background** in young rice (use SAVI/NDWI hybrids); **atmosphere** if you use L1C instead of L2A.
- **Yield model drift**: models trained on old varieties/agronomy degrade — re-fit annually, and always report the model's own uncertainty, not just the point forecast.
- **The "last mile"**: the hard part is not the NDVI, it's joining parcels to loans and turning a z-score into a credit decision — the classic data-to-action gap (see [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md)).

---

## 8. The Future (2026+)

### 8.1 SAR Constellations — All-Weather Goes Commercial

The biggest capability shift of the 2020s: SAR is leaving the government domain. **ICEYE** (Finland) and **Capella Space** (US) operate X-band SAR constellations with **~0.5 m class resolution** (Capella spotlight to ~0.3–0.5 m; ICEYE Gen4 platforms cited at up to ~16 cm in fine modes) and **sub-daily–daily revisit**, at small-satellite cost. **Verified.** Umbra (US) adds X-band VHR SAR; China (GF-3 series, commercial) and Japan (ALOS-4, 2024) keep the government tier current. Implication: **flood, ship, infrastructure, and defence monitoring become daily and weather-proof**, and InSAR subsidence monitoring moves from studies to continuous commercial products.

### 8.2 Hyperspectral Goes Commercial

**EnMAP** (2022, open data) and **PRISMA** (2019) made spaceborne imaging spectroscopy routine; the next step is commercial constellations (e.g., Pixxel, and the ESA **CHIME** mission planned for ~2029). **GHGSat** already runs the commercial methane model: **~25 m resolution point-source methane detection** (13 satellites, GHGSat-D 2016 + C1–C13), with public alert layers — the reference case for "EO as an environmental regulator's evidence base" and for oil and gas financed-emissions screening. **Verified.**

### 8.3 AI — Deep Learning and EO Foundation Models

- **Deep learning** is now the default for classification, object detection (buildings, ships, cars), and super-resolution at VHR scales (§4.6).
- **EO foundation models** — self-supervised models pre-trained on massive unlabelled satellite archives, then fine-tuned cheaply per task — are the 2023+ frontier. **Verified** examples: **Prithvi** (NASA/IBM/Jülich, open source, trained on NASA HLS Landsat+Sentinel-2 data), **Clay** (Development Seed, open), and commercial offerings. They cut the labelled-data cost of land-cover/flood/damage models dramatically — the same pattern as LLMs, one domain over (see [autonomous_agents_guide.md](ai_llm/autonomous_agents_guide.md) for the agentic layer on top).
- **AI + SAR + InSAR**: automated flood extent, subsidence, and damage products are becoming API-callable analytics, not research.

### 8.4 EO as a Service

The industry is consolidating around **platforms, not pixels**: GEE-style cloud processing, STAC-based data lakes, analytics APIs (flood, fire, crop, methane), and single-vendor "space data marketplaces" (e.g., AWS/Azure/Google space data initiatives, Up42-style marketplaces). For enterprises, the pattern is: **buy decisions, not imagery**; build in-house only where the analytics are a competitive moat. **Verified** as the market's stated direction (vendor roadmaps, procurement patterns); exact revenue splits are **Flagged** (see §11.1 market-size note).

### 8.5 Climate and Carbon Monitoring

- **Methane**: GHGSat (~25 m point sources) + Sentinel-5P TROPOMI (wide-area) + **MethaneSAT** (EDF, 2024) is building a global, attributable methane ledger — directly relevant to financed-emissions accounting (PCAF/OGMP frameworks) and to the EU's methane regulation.
- **CO₂**: the ESA **CO2M** constellation (planned ~2026) will deliver the first operational anthropogenic CO₂ column measurements — the "carbon Copernicus."
- **Biomass/carbon**: NASA-ISRO **NISAR** (L+S-band SAR, launched 2025) and ESA **BIOMASS** (P-band, planned) target global biomass; combined with GEDI-class LiDAR they close the terrestrial carbon cycle loop.
- **Climate-risk finance**: expect EO-based physical-risk data to become a *mandatory* input class in banking regulation (IFRS S2, ECB/HKMA/MAS climate expectations already imply it — **Verified** direction, **Reported** exact timing).

### 8.6 Trends Summary

| Trend | What changes | 2026 status |
|---|---|---|
| SAR constellations (ICEYE, Capella, Umbra) | Daily all-weather VHR SAR + InSAR products | Operational |
| Commercial hyperspectral (EnMAP/PRISMA/CHIME, Pixxel) | Material/gas identification at scale | EnMAP/PRISMA operational; CHIME ~2029 |
| EO foundation models (Prithvi, Clay) | Cheap fine-tuning for classification/detection | Operational, rapidly improving |
| EO as a service (GEE, STAC marketplaces, analytics APIs) | Decisions bought, not pixels | Operational, consolidating |
| Methane/CO₂ monitoring (GHGSat, MethaneSAT, CO2M) | Attributable emissions accounting | GHGSat/MethaneSAT operational; CO2M ~2026 |
| NISAR + BIOMASS | Global biomass/ground-motion at scale | NISAR launched 2025; BIOMASS planned |
| EO in regulated finance | Physical-risk disclosure (IFRS S2, ECB, MAS) | Early adoption; standard-setting ongoing |

---

## 9. Summary — Remote Sensing in One Page

**Remote sensing** = measuring the Earth's surface from a distance via electromagnetic radiation. Passive sensors see reflected sunlight and emitted heat (optical, thermal); active sensors illuminate the target themselves (SAR, LiDAR) and so work day/night, and through clouds in the microwave. The **electromagnetic spectrum** is the palette: visible (0.4–0.7 µm), NIR (0.7–1.3 µm — vegetation), SWIR (1.3–3 µm — moisture/minerals), TIR (8–14 µm — heat), microwave (1 mm–1 m — radar; X/C/L bands). **Spectral signatures** — characteristic reflectance curves — let us identify materials; vegetation's red edge (~680–750 nm) is the canonical example, and indices like **NDVI = (NIR−Red)/(NIR+Red)** encode them into numbers.

Resolution is four-dimensional — **spatial** (GSD: 30 cm WorldView to 30 m Landsat to 1 km MODIS), **spectral** (bands), **temporal** (revisit: continuous GEO to daily Planet to 16-day Landsat), **radiometric** (8-bit legacy, 12-bit modern) — and the four trade off against physics and cost. Platforms span **satellites** (sun-synchronous workhorses, geostationary weather eyes, small-sat constellations), **aircraft and drones** (cm-scale, on demand), and **ground systems** (LiDAR, flux towers — the calibration backbone). Sensors divide into **multispectral** (Landsat, Sentinel-2), **hyperspectral** (EnMAP), **thermal** (MODIS, TIRS), **SAR/InSAR** (Sentinel-1, all-weather; mm-cm ground motion), and **LiDAR** (structure and elevation).

Data processing is a chain: **radiometric → geometric/orthorectification → atmospheric correction → cloud masking → indices → classification → time-series analysis**, with QGIS/GDAL/GEE/Sentinel Hub as the tools. Data is overwhelmingly **open** (Landsat free since 2008; Copernicus free-and-open since 2014; MODIS daily) with a commercial layer on top (Maxar 30 cm, Planet daily 3 m, SkySat 50 cm video, Airbus 30 cm, China's Gaofen/Jilin-1 sub-metre) and a fast-growing **analytics-as-a-service** layer. Applications span **agriculture** (NDVI crop monitoring, yield, precision farming), **forestry**, **urban land use**, **disasters** (Sentinel-1 floods, MODIS fires, InSAR earthquakes), **ocean** (SST, oil spills — see the maritime guide), **climate**, **defence**, **insurance** (crop, parametric), and **banking** (physical climate risk, ESG screening, commodity monitoring).

The worked example — Mekong Delta rice monitoring with Sentinel-2 NDVI + Sentinel-1 SAR + MODIS baselines — shows the full chain from pixels to a credit decision. The future is **more radar, more spectra, more AI** (SAR constellations, commercial hyperspectral, EO foundation models like Prithvi) and **EO as a service** — bought by the decision, increasingly mandated by climate finance.

---

## 10. Glossary

- **Active sensor** — a sensor that illuminates its target (radar, LiDAR) and measures the return; works day/night; microwaves penetrate clouds.
- **Atmospheric correction** — removing scattering/absorption effects to recover surface reflectance (6S, Sen2Cor, LaSRC, DOS).
- **AVIRIS** — NASA's airborne imaging spectrometer (224 bands), the hyperspectral benchmark.
- **Band math** — arithmetic combinations of bands; the basis of indices.
- **C-band** — radar band 4–8 GHz (~3.75–7.5 cm); Sentinel-1, RADARSAT.
- **Change detection** — comparing multi-date imagery to identify surface changes.
- **Classification** — assigning pixels/objects to categories (supervised/unsupervised/OBIA/deep learning).
- **Cloud masking** — detecting and removing cloudy pixels (FMask, s2cloudless).
- **CNN** — convolutional neural network; the deep-learning workhorse for image classification/segmentation.
- **Copernicus** — the EU/ESA Earth-observation program; Sentinel satellites; free-and-open data.
- **Crop monitoring** — tracking crop condition through the season with vegetation indices.
- **EO (Earth Observation)** — satellite-borne remote sensing.
- **EVI** — Enhanced Vegetation Index; aerosol-corrected NDVI variant: 2.5×(NIR−Red)/(NIR+6·Red−7.5·Blue+1).
- **EM spectrum** — electromagnetic spectrum: the range of radiation wavelengths; remote sensing uses visible → microwave.
- **EnMAP** — German hyperspectral satellite (2022; 242 bands, 30 m).
- **ESG** — Environmental, Social, Governance; EO feeds the environmental dimension (deforestation, emissions, water).
- **Fire detection** — thermal hotspot detection (MODIS/VIIRS/geostationary).
- **Flood mapping** — typically SAR-based water-extent mapping (Sentinel-1).
- **Foundation model** — a large pre-trained model fine-tuned per task (Prithvi, Clay for EO).
- **Gaofen (GF)** — China's CHEOS high-resolution satellite series (GF-2 0.8 m, GF-7 0.65 m stereo).
- **GDAL** — open-source geospatial data library (raster/vector I/O).
- **Geometric correction** — assigning map coordinates to imagery.
- **Geostationary orbit** — 35,786 km equatorial orbit; satellite appears fixed; continuous hemisphere views (weather).
- **GHGSat** — Canadian methane-monitoring constellation (~25 m point sources).
- **GSD — Ground Sampling Distance** — the ground size of one pixel; the spatial resolution.
- **Google Earth Engine (GEE)** — cloud planetary-scale EO analysis platform.
- **Hyperspectral** — 100s of narrow contiguous bands; material identification.
- **InSAR** — interferometric SAR; phase comparison across passes to measure mm–cm ground motion.
- **ISODATA** — classic unsupervised clustering algorithm.
- **k-means** — the standard unsupervised clustering algorithm.
- **Landsat** — NASA/USGS civilian EO program since 1972; Landsat 8/9 at 30 m, free.
- **L-band** — radar band 1–2 GHz (15–30 cm); deep vegetation penetration (ALOS-2, NISAR).
- **LiDAR** — Light Detection and Ranging; laser pulses → 3-D point clouds; elevation/canopy structure.
- **Maxar** — US VHR satellite operator (WorldView, Legion, 30 cm).
- **Methane** — potent greenhouse gas; now monitored at point-source scale (GHGSat, MethaneSAT, TROPOMI).
- **Microwave** — 1 mm–1 m wavelengths; radar region; cloud-penetrating.
- **MODIS** — NASA's daily global imager (Terra/Aqua; 250 m–1 km; 36 bands).
- **Multispectral** — several broad bands (RGB+NIR typical; Sentinel-2 13 bands).
- **NDVI** — Normalized Difference Vegetation Index: (NIR−Red)/(NIR+Red).
- **NDWI** — Normalized Difference Water Index: (Green−NIR)/(Green+NIR).
- **NIR** — near-infrared, 0.7–1.3 µm; vegetation and water detection.
- **OBIA** — Object-Based Image Analysis; segment-then-classify.
- **Oil spill** — SAR-detected dark slicks (CleanSeaNet; see the maritime guide).
- **Orthorectification** — terrain-corrected georeferencing using a DEM.
- **Parametric insurance** — index-triggered payouts (satellite indices); no loss adjustment.
- **Passive sensor** — measures reflected sunlight/emitted heat; daylight/cloud-limited (optical).
- **Planet / PlanetScope** — daily 3 m small-satellite constellation (~200+ Doves).
- **Pleiades** — Airbus VHR satellites (1A/1B 50 cm; Neo 30 cm).
- **Point cloud** — 3-D coordinate set from LiDAR/radar/photogrammetry.
- **Precision agriculture** — field/sub-field management driven by imagery and sensors.
- **Radiometric correction** — DN → reflectance/radiance calibration.
- **Radiometric resolution** — bit depth of the signal (8-bit legacy; 12-bit Landsat 8/9, Sentinel-2).
- **Red edge** — the sharp vegetation reflectance rise ~680–750 nm; chlorophyll/stress indicator.
- **SAR** — Synthetic Aperture Radar; all-weather, day/night imaging radar.
- **Sea surface temperature (SST)** — thermal-band ocean temperature (MODIS, Sentinel-3).
- **Semantic segmentation** — per-pixel classification by deep learning (U-Net).
- **Sentinel** — ESA Copernicus satellites (S1 SAR 2014, S2 optical 2015, S3 ocean 2016, S5P atmosphere 2017).
- **Sentinel Hub** — cloud API for EO data (Sinergise).
- **SkySat** — Planet's 50 cm VHR satellites with video capability.
- **Spatial resolution** — pixel GSD.
- **Spectral resolution** — number/width of bands.
- **Spectral signature** — a material's characteristic reflectance curve.
- **SPOT** — French/Airbus medium-VHR satellite series (SPOT 6/7 at 1.5 m).
- **Supervised classification** — learning from labelled training data (random forest, SVM).
- **SWIR** — shortwave-infrared, 1.3–3 µm; moisture, minerals, fire.
- **Temporal resolution** — revisit time.
- **Thermal (TIR)** — 8–14 µm emitted-heat sensing (also MWIR 3–5 µm for hot targets).
- **Time series** — multi-date analysis (NDVI trends, change detection, breakpoints).
- **UAV/drone** — low-altitude platform; cm-scale imagery.
- **Unsupervised classification** — data-driven clustering (k-means, ISODATA).
- **Visible** — 0.4–0.7 µm; true colour.
- **WorldView** — Maxar's VHR satellite family (0.3–0.5 m).
- **X-band** — radar band 8–12 GHz (~2.5–3.75 cm); VHR SAR (TerraSAR-X, Capella, ICEYE).

---

## 11. Claims Status, References and Further Reading

### 11.1 Claims Status

| # | Claim | Status | Notes |
|---|---|---|---|
| 1 | Remote-sensing definition (information from a distance, no contact) | **Verified** | Standard framing across NASA/USGS educational material |
| 2 | Spectrum windows: VIS 0.4–0.7, NIR 0.7–1.3, SWIR 1.3–3, TIR 8–14 µm, microwave 1 mm–1 m | **Verified** | Boundaries vary ±0.05 µm by source; table states the common convention |
| 3 | Radar bands: X 8–12 GHz, C 4–8 GHz, L 1–2 GHz; Sentinel-1 C (5.405 GHz) | **Verified** | IEEE band designations; mission frequencies per ESA/vendor |
| 4 | Vegetation red edge ~680–750 nm; high NIR; water absorbs NIR; soil reflectance rises with wavelength | **Verified** | Textbook-established behaviour (multiple sources, e.g., red-edge 680–750 nm per WUR/AGU literature) |
| 5 | Radiometric: legacy Landsat 8-bit; Landsat 8/9 OLI and Sentinel-2 12-bit, distributed 16-bit | **Verified** | Landsat 8 OLI 12-bit per USGS/literature; Sentinel-2 12-bit per ESA |
| 6 | History: CORONA 1960s spy satellites (first success Aug 1960, declassified 1995); Landsat-1 23 Jul 1972 (80 m MSS); resolution progression 80→30→10→1→0.5→0.3 m | **Verified** | NASA/USGS mission pages; progression is the standard industry telling |
| 7 | Sun-synchronous orbits give consistent local-time illumination; geostationary continuous | **Verified** | Orbital mechanics; standard reference |
| 8 | Planet PlanetScope ~3 m daily revisit (~200+ Doves); SkySat ~50 cm + video | **Verified** | Planet/ESA documentation; fleet sizes fluctuate |
| 9 | Sentinel-1 launched Apr 2014 (C-SAR); Sentinel-2 Jun 2015 (10 m); Sentinel-3 Feb 2016 (ocean) | **Verified** | ESA mission pages |
| 10 | Landsat 8/9 free 30 m; MODIS 250 m–1 km daily; Copernicus free-and-open policy | **Verified** | USGS open-data policy (2008); EU Copernicus Regulation (2014) |
| 11 | EnMAP: launched 1 Apr 2022; 242 bands 420–2450 nm; 30 m; ~27-day revisit | **Verified** | DLR/eoPortal; band count cited 230–246 across sources (242 nominal) |
| 12 | Commercial VHR: Maxar WorldView/Legion 30 cm; Pléiades 50 cm / Pléiades Neo 30 cm / SPOT 6-7 1.5 m | **Verified** | Vendor documentation |
| 13 | Gaofen: GF-2 0.8 m (2014), GF-7 0.65 m stereo; commercial Chinese ~0.5 m (Jilin-1, SuperView) | **Verified** | eoPortal/CNSA sources; the "0.5 m Gaofen" popular claim refers to the commercial tier |
| 14 | ICEYE/Capella X-band SAR ~0.3–0.5 m; ICEYE Gen4 fine modes up to ~16 cm | **Verified** | eoPortal; vendor specs (fine modes are single-look, limited duty) |
| 15 | GHGSat ~25 m methane point-source detection, 13 sats | **Verified** | GHGSat/ESA; fleet count as of 2025 |
| 16 | EO foundation models: Prithvi (NASA/IBM/Jülich, open), Clay (Development Seed) | **Verified** | NASA science blog, Hugging Face, project docs |
| 17 | InSAR measures mm–cm ground motion; used for subsidence, earthquakes (2023 Türkiye–Syria) | **Verified** | Standard InSAR literature; event mapping well documented |
| 18 | Sentinel-1 SAR flood mapping = global emergency standard (Copernicus EMS) | **Verified** | CEMS/ESA operational documentation |
| 19 | MODIS/VIIRS active-fire products operational (FIRMS); NBR/dNBR burn severity | **Verified** | NASA FIRMS/USGS documentation |
| 20 | Parametric/crop insurance with satellite indices (ARC ~35+ African countries; PMFBY India) | **Verified** (mechanism); **Reported** (country counts) | Program scope varies by year |
| 21 | Banks' physical-risk/climate reporting (IFRS S2, ECB stress tests, MAS/HKMA) implies EO inputs | **Verified** (regulatory direction); **Reported** (adoption pace) | Methodology still evolving |
| 22 | EO market size | **Flagged — not cited as a single number** | Estimates for 2024 range ~US$2.2B (commercial data, SpaceNews) to ~US$3.7–5.1B (broader definitions, GM Insights/Grand View); definitions vary too widely to cite one figure |
| 23 | Commercial per-km² prices (US$10–30+ for 30–50 cm) | **Flagged** | Indicative industry ranges; pricing is negotiated, per-order, and not published |
| 24 | NDVI/EVI/NDWI formulas | **Verified** | Rouse et al. 1973; Huete et al. 2002; McFeeters 1996 (standard references) |
| 25 | Drone remote sensing mature (cm GSD, on-demand) | **Verified** | Mature commercial practice (DJI-class platforms); regulatory constraints vary by jurisdiction |

### 11.2 Key Sources Consulted

- NASA/USGS — Landsat mission pages and data policy; MODIS/VIIRS product documentation; HLS (Harmonized Landsat Sentinel); NASA Earth Observatory educational material.
- ESA — Sentinel mission pages (S1/S2/S3/S5P), Copernicus data policy, EnMAP entry on Earth Online, eoPortal mission articles (EnMAP, Planet, Capella, ICEYE, Gaofen-2).
- DLR — EnMAP launch and mission material.
- GHGSat — constellation and technology pages.
- Vendor/public documentation — Maxar (WorldView/Legion), Planet (PlanetScope, SkySat), Airbus (Pléiades Neo, SPOT), ICEYE, Capella Space.
- Industry/analyst reporting — SpaceNews (EO market), GM Insights / Grand View Research (market size — used only as flagged ranges).
- Scientific literature — Rouse et al. 1973 (NDVI); Huete et al. 2002 (EVI); McFeeters 1996 (NDWI); red-edge literature (680–750 nm).
- Academic/agency material on InSAR, OBIA, and semantic segmentation (standard textbooks and review papers).

### 11.3 Further Reading (Adjacent Repo Guides)

- [maritime_domain_awareness_guide.md](maritime_domain_awareness_guide.md) — the maritime application of remote sensing: SAR/EO vessel detection, satellite AIS, oil-spill monitoring (CleanSeaNet), dark-fleet analytics. **The sibling use-case guide to this one.**
- [ips_rtls_guide.md](ips_rtls_guide.md) — indoor RF sensing; the "sensing fundamentals" twin at short range.
- [event_stream_processing_guide.md](event_stream_processing_guide.md) / [complex_event_processing_guide.md](complex_event_processing_guide.md) — streaming and pattern detection for sensor-data pipelines (flood flags, anomaly rules).
- [ai_llm/deep_learning_frameworks_comparison_guide.md](ai_llm/deep_learning_frameworks_comparison_guide.md) — the model engineering behind CNN/segmentation work; [ai_llm/llm_evaluation_vs_validation_guide.md](ai_llm/llm_evaluation_vs_validation_guide.md) — evaluation discipline for ML-derived indicators.
- [advanced_analytics_solutions_guide.md](advanced_analytics_solutions_guide.md) — the analytics layer behind dashboards and risk scoring.
- [data/data_pipeline_guide.md](data/data_pipeline_guide.md) / [data/data_fabric_guide.md](data/data_fabric_guide.md) — geospatial data pipeline engineering (STAC → object storage → lakehouse).
- [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — model-risk and climate-risk governance for EO-derived bank indicators.
- The repo has **no dedicated GIS guide yet** — QGIS/GDAL/GEE are covered in §4.8 here; a standalone GIS/geospatial-analysis guide is a suggested future companion (the maritime guide flagged the same gap).
