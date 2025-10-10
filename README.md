# Earth Vision Grid — FDPlan Input Generator

Single-page app that:
1) takes geographic bounds + projection params,  
2) calls the **GEBCO** bathymetry API (via **ODB**),  
3) projects to UTM, grids, smooths (client-side), and  
4) exports an Earth Vision Grid text file **or** uploads it to **FDPlan**.

> Pure client-side (no backend). External requests: ODB GEBCO API and optional FDPlan APIs.

---

## What this page does

### Inputs
- **Coordinates**
  - `longitude.start` (west), `longitude.end` (east)
  - `latitude.start` (south), `latitude.end` (north)
- **Projection**
  - `zone` (UTM zone: 1–60)
  - `South` (bool; Southern Hemisphere)
- **Optional parameters (advanced)** *(collapsed)*  
  - `sample` (string, default `"1"`) — sampling for API  
  - `Grid spacing (m)` — `dx = dy` (default `200`)  
  - `Edge offset (steps)` — integer margin in grid steps to keep points **inside** bounds (default `3`)
- **FDPlan Upload parameters** *(collapsed, compact)*  
  - `Name` — dataset name (default `Bathymetry External`)  
  - `App Key` — required  
  - `Data Partition` — required 
  - `Authorization (key only)` — required; **enter only the token**, the app prefixes `Bearer `  
  - `StudyId` — required 

### Map schematic
A world frame with a highlighted rectangle for your bounds.  
**Click** to open Google Maps with **four pins** (SW, NW, NE, SE). 

### Actions
- **Download Grid File**  
  Calls ODB GEBCO, processes client-side, then downloads `updated_bathymetry_data_LATLONG.txt`.
- **Run & Upload to FDPlan**  
  Executes the full FDPlan pipeline to upload a data source to a container. 
  On success: status shows **“FDPlan upload completed.”**

### Logs
A single collapsed **Logs** section at the bottom (one line header).  
It includes: GEBCO URL/response, processed preview, `jobId`, `containerID`, and all FDPlan responses.  
Sensitive headers are **masked** in logs.

---

## FDPlan upload flow (what “Run & Upload to FDPlan” does)

1. **Create Job**     
2. **Poll for Container**     
3. **Upload File (multipart/form-data)**     
4. **Set Job State**     

> Upload won’t start unless all FDPlan parameters are filled. The app validates and opens the section if anything is missing.

---

## Processing pipeline (client-side)

1. **Input → Polygon ring**  
   `[[lonS,latS],[lonS,latN],[lonE,latN],[lonE,latS],[lonS,latS]]`
2. **API call**  
   `GET /gebco?mode=Polygon&sample=<sample>&jsonsrc=<urlencoded GeoJSON Polygon>`
3. **Normalization**  
   Supports ODB `{longitude[], latitude[], z[]}` (also common array/object row formats).
4. **Projection** (via `proj4`)  
   WGS84 → UTM `+proj=utm +zone=<zone> +south? +datum=WGS84 +units=m +no_defs`
5. **Grid generation**
   - Inner axes with **edge offset** to avoid points on/outside the bounds
   - Regular grid with `dx = dy`
6. **Interpolation**  
   IDW (k=12, p=2)
7. **Smoothing**  
   Separable Gaussian (σ=5)
8. **Output assembly**  
   - Header:
     ```
     # Grid_size: <W> x <H>
     # Grid_space: <xmin>,<xmax>,<ymin>,<ymax>
     ```
   - Rows: `x y z column row` (x,y rounded to 4 decimals; sorted by row then column)

---

## Output file

- **Filename:** `updated_bathymetry_data_LATLONG.txt`  
- **Format:** Earth Vision Grid (scattered data + header)  
- **Bounds safety:** axes are contracted by `Edge offset (steps) * grid spacing` so every grid point lies **inside** the original bounds.

---

## Endpoints used

- **ODB GEBCO**  
  `GET https://api.odb.ntu.edu.tw/gebco?mode=Polygon&sample=<s>&jsonsrc=<geojson>`

**Headers (FDPlan):**


---

## How to use

1. Open the page (GitHub Pages `index.html`).  
2. Fill **Coordinates** and **Projection** (defaults target SE Brazil, UTM 23S).  
3. (Optional) Open **Optional parameters (advanced)** to tune `sample`, grid spacing, edge offset.  
4. **Download only:** click **Download Grid File**.  
5. **Upload to FDPlan:** open **FDPlan Upload parameters**, fill **App Key**, **Data Partition**, **Authorization (key only)**, **StudyId**, (optional **Name**), then click **Run & Upload to FDPlan**.  
6. Keep the tab open until completion. Check **Logs** (collapsed) for details.

---

## Security & privacy

- The page is static; processing happens in your browser.  
- FDPlan credentials (App Key, Data Partition, Token) are used **only** for the live requests and are **not** persisted.  
- Logs mask sensitive header values.  
- No third-party analytics.

---

## Troubleshooting

- **“Cannot run: … required.”**  
  Fill all fields in **FDPlan Upload parameters**; the section opens automatically when validation fails.
- **Upload 400/404**  
  Container path may not be ready yet; the app automatically polls and retries.
- **CORS / network errors**  
  Ensure your network allows direct calls to `br1.api.delfi.slb.com` and `api.odb.ntu.edu.tw`.

---

## Attribution

**Data source**  
GEBCO Compilation Group (2023) **GEBCO 2023 Grid** (doi:10.5285/f98b053b-0cbc-6c23-e053-6c86abc0af7b)  
https://www.gebco.net/data_and_products/gridded_bathymetry_data/

**API provider**  
This API is compiled by Ocean Data Bank (ODB):  
Ocean Data Bank, National Science and Technology Council, Taiwan. https://doi.org/10.5281/zenodo.7512112. Retrieved 2025-10-10 from `api.odb.ntu.edu.tw/gebco`. v1.0.

**Google Maps**  
This page **does not embed** Google Maps or use Google Maps Platform APIs; it only **opens** a Google Maps URL in a new tab for convenience. For embedded maps or API usage, follow Google’s branding and terms (Google Maps Platform Terms of Service and attribution guidelines).

---

## License & disclaimer

- Bathymetry and APIs are subject to their respective licenses/terms above.  
- This client performs geoprocessing **in-browser** and is provided “as is” without warranties.
