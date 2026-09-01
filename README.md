#  AquaTerrace — Pond Catchment Analysis

A backend-based geospatial application for analyzing contour maps and identifying suitable pond locations based on terrain and catchment characteristics.

This project is developed as **Phase 2** of the Pond Construction / Catchment Analysis project. The system accepts a contour map in **KML/KMZ format**, processes the terrain information, estimates catchment areas, identifies potential pond locations, and returns the results through a REST API.

---

## Application

The application is currently running locally at:

**http://localhost:8000**

> **Note:** `localhost` is accessible only when the application is running on the user's machine.

---

##  Problem Statement

Selecting a suitable location for pond construction requires understanding the surrounding terrain and determining how much area naturally drains toward a potential pond location.

The objective of this phase is to develop a backend API that:

- Accepts a contour map in KML/KMZ format
- Extracts terrain elevation information
- Generates a terrain representation from contour data
- Analyzes terrain flow and depressions
- Identifies potential pond locations
- Estimates the corresponding catchment area
- Returns the analysis in a structured JSON format
- Can be integrated with a map-based frontend

The implementation derives the results from the uploaded contour map rather than relying on hard-coded coordinates or locations.

---

## Objectives

1. Accept KML/KMZ contour maps through an API.
2. Extract coordinates and elevation information from contour data.
3. Generate an interpolated Digital Elevation Model (DEM).
4. Analyze terrain flow using D8 flow routing.
5. Detect potential terrain depressions suitable for pond construction.
6. Delineate the contributing catchment area for candidate pond locations.
7. Calculate catchment area in square metres and hectares.
8. Estimate runoff and basic pond requirements.
9. Return the results in a structured JSON response.
10. Visualize the detected contours, catchments, and candidate pond sites on an interactive map.

---

## System Workflow

```text
                    Contour Map
                     KML / KMZ
                         │
                         ▼
               ┌──────────────────┐
               │  KML/KMZ Parser  │
               └────────┬─────────┘
                        │
                        ▼
              Contour Coordinates
               + Elevation Data
                        │
                        ▼
              ┌──────────────────┐
              │ DEM Generation   │
              │ & Interpolation  │
              └────────┬─────────┘
                       │
                       ▼
              Terrain / Elevation Grid
                       │
                       ▼
              ┌──────────────────┐
              │  D8 Flow Routing │
              └────────┬─────────┘
                       │
                       ▼
             Flow / Depression Analysis
                       │
                       ▼
              Candidate Pond Sites
                       │
                       ▼
              Catchment Delineation
                       │
                       ▼
             Catchment Area Estimation
                       │
                       ▼
              Runoff / Pond Estimation
                       │
                       ▼
                  JSON Response
                       │
                       ▼
              Interactive Map Interface


---

##  Methodology

### 1. Contour Map Input

The API accepts contour maps in:

* `.kml`
* `.kmz`

The contour map contains geographic coordinates and elevation information.

The uploaded file is processed dynamically, so pond locations and catchment areas are derived from the input terrain.

---

### 2. Contour Data Extraction

The backend parses the uploaded KML/KMZ file and extracts:

* Latitude
* Longitude
* Elevation
* Contour geometry

These points form the basis for constructing the terrain surface.

---

### 3. Digital Elevation Model (DEM)

Since contour maps provide elevation values along contour lines rather than at every location, the extracted elevation points are interpolated to create a regular elevation grid.

This provides a continuous representation of the terrain that can be used for flow analysis.

---

### 4. Terrain Flow Analysis

A **D8 flow routing approach** is used to determine the direction in which water would naturally flow from each terrain cell.

For each cell, the neighbouring cell with the greatest downward elevation gradient is selected as the primary flow direction.

```text
        NW   N   NE

        W    X    E

        SW   S   SE
```

This allows the system to estimate how water moves across the terrain.

---

### 5. Potential Pond Location Detection

Terrain depressions/sinks are identified as candidate locations where water can potentially accumulate.

Candidate locations are filtered using terrain characteristics such as elevation and depression depth.

---

### 6. Catchment Delineation

For every candidate pond location, upstream cells that drain toward that location are identified.

The total contributing area is estimated from the number and physical area of the contributing terrain cells.

The catchment area is reported in:

* Square metres
* Hectares

---

### 7. Runoff Estimation

The system uses a runoff coefficient and rainfall value to estimate runoff contribution from the catchment.

A simplified rainfall-runoff relationship is used:

```text
Runoff Volume = Runoff Coefficient × Rainfall × Catchment Area
```

The runoff coefficient can be selected according to the assumed soil/land characteristics.

---

### 8. Pond Recommendation

Multiple candidate pond locations can be generated from the terrain analysis.

The candidate locations are ranked according to their calculated catchment characteristics, and the best candidate is returned as the recommended pond location.

---

##  API

### Analyze Contour Map

**Primary Endpoint:**

```http
POST /api/analyze-contour
```

Additional compatible routes are also available:

```http
POST /analyzeContour
POST /findCatchment
```

### Request

The contour map is sent as a `multipart/form-data` file upload.

Example:

```bash
curl -X POST "http://localhost:8000/api/analyze-contour" \
  -F "file=@contours_1m.kml"
```

---

## Input

Supported file formats:

```text
.kml
.kmz
```

Example:

```text
contours_1m.kml
```

The system processes the uploaded contour map dynamically.

---

##  API Response

A successful analysis returns structured JSON containing information such as:

```json
{
  "status": "success",
  "processing_time_sec": 2.5,
  "pond_recommendation": {
    "rank": 1,
    "latitude": 21.251781,
    "longitude": 81.296501,
    "elevation_m": 273.29,
    "catchment_area_sqm": 124722.43,
    "catchment_area_hectares": 12.47
  },
  "all_pond_sites": [],
  "contour_summary": {},
  "contours_geojson": {}
}
```

The exact values depend on the uploaded contour map and analysis parameters.

---

## Interactive Map

The project includes an interactive map interface for visualizing the analysis results.

The interface displays:

* Contour lines
* Elevation variation
* Candidate pond locations
* Recommended pond location
* Catchment basin
* Catchment boundaries
* Terrain information
* Candidate site information

### Open the Application

**[http://localhost:8000](http://localhost:8000)**

---

## Example Analysis

The provided sample contour map is:

```text
contours_1m.kml
```

When the sample map is uploaded, the system analyzes the terrain and generates multiple candidate pond locations.

The output includes:

* Recommended pond location
* Latitude and longitude
* Elevation
* Catchment area
* Catchment area in hectares
* Estimated runoff
* Pond-related estimates
* Catchment geometry

The results are generated from the uploaded contour data rather than being hard-coded for the sample map.

---

## Technology Stack

### Backend

* Python
* FastAPI
* Uvicorn

### Geospatial / Terrain Processing

* KML/KMZ parsing
* NumPy
* SciPy
* GeoJSON
* DEM interpolation
* D8 flow routing

### Frontend / Visualization

* HTML
* CSS
* JavaScript
* Leaflet.js
* OpenStreetMap

---

##  Project Structure

```text
.
├── app.py
├── analyzer.py
├── requirements.txt
├── README.md
├── contours_1m.kml
├── templates/
│   └── index.html
├── static/
│   ├── css/
│   └── js/
└── ...
```

> The exact structure may vary depending on the current project setup.

---


The application will be available at:

**[http://localhost:8000](http://localhost:8000)**

---

##  Testing the API

### Using the Web Interface

Open:

**[http://localhost:8000](http://localhost:8000)**

Then:

1. Upload a `.kml` or `.kmz` contour map.
2. Select the required runoff coefficient.
3. Optionally provide rainfall information.
4. Click **Analyze Catchment**.
5. The system processes the terrain.
6. Candidate pond sites are displayed.
7. The recommended pond site and catchment information are shown on the map.

---

### Using cURL

```bash
curl -X POST "http://localhost:8000/api/analyze-contour" \
  -F "file=@contours_1m.kml"
```

---

### Using Postman

Set the request as:

```text
Method: POST
URL: http://localhost:8000/api/analyze-contour
```

Under:

```text
Body → form-data
```

add:

```text
Key: file
Type: File
Value: contours_1m.kml
```

Then click **Send**.

The API returns the catchment analysis as JSON.

---

## No Hard-Coded Sample Results

An important requirement of this project is that the system must not hard-code the location or result of the provided sample contour map.

The implementation therefore does **not** directly store the sample pond coordinates or catchment area as fixed values.

Instead, the analysis follows:

```text
Uploaded KML/KMZ
        ↓
Extract terrain data
        ↓
Generate DEM
        ↓
Analyze terrain
        ↓
Detect candidate locations
        ↓
Calculate catchment
        ↓
Return result
```

This allows the same API to be used with different contour maps.

---
