# Earth Vision Grid — FDPlan Input Generator

Single-page app that:
1) takes geographic bounds + projection params,
2) calls the **GEBCO** bathymetry API proxied by **ODB**,
3) projects to UTM, grids, smooths, and
4) downloads an FDPlan-style text file with header + rows.

> Pure client-side (no backend). One external request: the ODB GEBCO API.

---

## What this page does

- **Inputs**
  - **Coordinates**
    - `longitude.start` (west), `longitude.end` (east)
    - `latitude.start` (south), `latitude.end` (north)
  - **Projection**
    - `zone` (UTM zone: 1–60)
    - `South` (bool; Southern Hemisphere)
  - **Optional parameters (advanced)**
    - `sample` (string, default `"1"`) — sampling for API
    - `Grid spacing (m)` — `dx = dy` in meters (default `200`)
    - `Edge offset (steps)` — integer margin in grid steps to keep points inside bounds (default `3`)

- **Map schematic**
  - Shows a world frame with a highlighted rectangle representing your bounds.
  - **Clicking** the schematic opens Google Maps in a new tab with **four pins** (SW, NW, NE, SE). If four waypoints aren’t allowed, it falls back to two pins (SW → NE).

- **Run — Call API & Download**
  - Calls `https://api.odb.ntu.edu.tw/gebco` with `mode=Polygon`, `sample`, and the polygon you defined.
  - Displays a non-blocking **Working…** overlay (safe to keep the tab open).
  - When processing finishes, downloads `updated_bathymetry_data_LATLONG`.

- **Debug panels** (collapsed by default)
  - **API Response (raw JSON):** shows returned payload (either array form or `{longitude[], latitude[], z[]}`).
  - **Processed Preview (first 30 rows):** quick peek at the processed grid.

---

## Processing pipeline (client-side)

1. **Input → Polygon ring**  
   `[[lonS,latS],[lonS,latN],[lonE,latN],[lonE,latS],[lonS,latS]]`

2. **API call**  
   `GET /gebco?mode=Polygon&sample=<sample>&jsonsrc=<urlencoded GeoJSON Polygon>`

3. **Normalization**  
   Supports ODB format `{longitude[], latitude[], z[]}` (and falls back to common array/object row formats).

4. **Projection** (via `proj4`)  
   WGS84 → UTM `+proj=utm +zone=<zone> +south? +datum=WGS84 +units=m +no_defs`.

5. **Grid generation**
   - Inner axes with **edge offset** (in steps) to prevent grid points on/outside bounds.
   - Regular grid with `dx = dy`.

6. **Interpolation**  
   IDW (k=12, p=2) to fill `Z` on the grid.

7. **Smoothing**  
   Separable Gaussian (σ=5) for a clean surface.

8. **Output assembly**  
   - Header:
     ```
     # Grid_size: <W> x <H>
     # Grid_space: <xmin>,<xmax>,<ymin>,<ymax>
     ```
   - Body rows: `x y z column row`  
     (`x,y` rounded to 4 decimals; rows sorted by row then column)

---

## Output file

- **Filename:** `updated_bathymetry_data_LATLONG`
- **Format:** TEart Vision Grid


- **Bounds safety:** axes are contracted by `Edge offset (steps) * grid spacing` to keep all grid points strictly inside the bounds.

---

## How to use

1. **Open** https://github.com/pmachado333/Bathy_extractor
2. Fill **Coordinates** and **Projection** (defaults cover SE Brazil, UTM 23S).
3. (Optional) Open **Advanced** to adjust `sample`, `Grid spacing`, `Edge offset`.
4. Click **Run — Call API & Download** and keep the tab open.  
 The file download starts automatically.
5. (Optional) Click the **Map schematic** to verify the selection in Google Maps with boundary pins.

---

## Dev notes

- **Tech:** Vanilla HTML/CSS/JS + `proj4` (no frameworks).  
- **Performance:** Uses requestAnimationFrame yields and a small overlay to avoid “page unresponsive” prompts.  
- **Privacy:** Sends only the polygon and `sample` param to ODB. No API keys.  

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

- Bathymetry and API usage are subject to their respective licenses/terms above.
- This client performs geoprocessing **in-browser** and is provided “as is” without warranties.
