
Nairobi Community Health Accessibility Analysis
Project Overview
A GIS-based multi-criteria spatial analysis assessing geographic access to health facilities across Nairobi County, Kenya. The analysis integrates population distribution, road networks, health facility locations, administrative boundaries, and terrain (elevation/slope) to identify communities with poor healthcare accessibility and prioritize areas for humanitarian intervention.
________________________________________
Key Questions
1.	Where are health facilities concentrated across Nairobi County?
2.	Where are populations concentrated relative to terrain?
3.	Which populated areas are relatively close to health facilities and roads?
4.	Which populated areas appear underserved, particularly where terrain may make physical access more difficult?
________________________________________
Tools & Technologies
Tool	Version	Purpose
QGIS	3.44.11-Solothurn	Primary GIS platform
GDAL	3.13.0	Raster processing engine
PROJ	9.8.1	Coordinate reference system transformations
GEOS	3.14.1	Vector geometry operations
________________________________________
Data Sources
Dataset	Source	Original CRS	Resolution
Population	WorldPop (Kenya 2026, R2025A v1)	EPSG:4326	100m (resampled to 30m)
Roads	OpenStreetMap (via Geofabrik/QuickOSM)	EPSG:4326	Vector (lines)
Health Facilities	Kenya Open Data Portal / HDX	EPSG:4326	Vector (points)
Administrative Boundaries	Kenya Open Data Portal	EPSG:4326	Vector (polygons)
Elevation (DEM)	NASA SRTM (30m)	EPSG:4326	30m
Projected CRS used throughout analysis: EPSG:32737 – WGS 84 / UTM Zone 37S (meters)
________________________________________
Methodology
Phase 1: Data Preparation & CRS Standardization
All datasets were inspected for geometry, attribute completeness, and units. Layers were reprojected from EPSG:4326 (degrees) to EPSG:32737 (meters) to ensure accurate distance and proximity calculations. The study area was defined by clipping all layers to the Nairobi County boundary.
Phase 2: Terrain Analysis
The 30m DEM was used to derive a Slope raster (in degrees) using GDAL's raster terrain analysis. Slope was classified into terrain difficulty categories for accessibility scoring.
Phase 3: Population Analysis
The 100m WorldPop population raster was resampled to 30m using bilinear interpolation to match the resolution of the DEM and road/facility distance rasters. Resampling was validated by comparing pre- and post-resampling statistics (mean pixel value changed by <0.6%, confirming conservation of population mass). Zonal Statistics was used to calculate total population per ward, and a Population-to-Facility Ratio was derived via the Field Calculator.
Phase 4: Facility & Road Accessibility
Health facility points and road networks were rasterized (30m resolution, burn value = 1, NoData = -9999) and processed through the Proximity (Raster Distance) tool to generate:
•	Distance-to-Facility raster (meters)
•	Distance-to-Road raster (meters)
Phase 5: Reclassification
All three accessibility rasters were reclassified onto a common 1–4 scale (4 = best accessibility, 1 = worst):
Raster	Class 4	Class 3	Class 2	Class 1
Facility Distance	0–2 km	2–5 km	5–10 km	>10 km
Road Distance	0–500 m	500 m–1 km	1–2 km	>2 km
Slope	0–5°	5–10°	10–15°	>15°
Phase 6: Accessibility Index
An equal-weight Accessibility Index was computed using the Raster Calculator:
text
Accessibility Index = (Facility_Access + Road_Access + Terrain_Access) / 3
The resulting continuous index was reclassified into four categories:
Score Range	Accessibility Level
1.0 – 1.75	Very Low
1.75 – 2.5	Low
2.5 – 3.25	Moderate
3.25 – 4.0	High
Phase 7: Priority Area Identification
The population raster was reclassified into three classes (Low, Medium, High). Priority Areas were identified using the Raster Calculator:
text
Priority = (Population_Class = 3) AND (Accessibility_Index_Class <= 2)
This identifies locations where high population density overlaps with low healthcare accessibility—the areas of greatest humanitarian concern.
________________________________________
Results
Map 1: Health Facility Distribution
Where are healthcare services located?
https://mega.nz/file/F7YWgCDC#oSYCgY2Kt0DjHqz74N-UXTowVBXaUZ_CSvoxxebh0e8
________________________________________
Map 2: Population Distribution & Terrain
Where are populations located relative to terrain?
https://mega.nz/file/F7YWgCDC#oSYCgY2Kt0DjHqz74N-UXTowVBXaUZ_CSvoxxebh0e8
________________________________________
Map 3: Health Accessibility Index
Which areas have better or worse geographic accessibility?
https://mega.nz/file/F7YWgCDC#oSYCgY2Kt0DjHqz74N-UXTowVBXaUZ_CSvoxxebh0e8
________________________________________
Map 4: Humanitarian Priority Areas
Where should further investigation or intervention be prioritized?
https://mega.nz/file/F7YWgCDC#oSYCgY2Kt0DjHqz74N-UXTowVBXaUZ_CSvoxxebh0e8
________________________________________
Key Findings
Finding	Description
Facility Distribution	Health facilities are concentrated in central Nairobi, with notable gaps in peripheral wards.
Population Pressure	Some wards have population-to-facility ratios exceeding 10,000 people per facility.
Accessibility Gradient	The Accessibility Index ranges from ~1.0 (Very Low) to 4.0 (High), with a mean of approximately 2.4–2.8.
Priority Areas	Dense informal settlements (e.g., Kibera, Eastleigh, Kawangware) show high population density combined with low accessibility, identifying them as priority zones for intervention.
________________________________________
Limitations
This analysis uses straight-line (Euclidean) distance to measure proximity to roads and health facilities. In reality, travel accessibility is influenced by factors not captured in this model:
•	Actual road network travel time (not straight-line distance)
•	Road quality and seasonality (unpaved roads may be impassable during rains)
•	Transport availability and cost
•	Rivers, mountains, and missing bridges that force detours
•	Facility capacity (a hospital and a dispensary do not provide equivalent services)
Additionally, the equal-weight model assumes that facility distance, road distance, and terrain difficulty contribute equally to accessibility. This assumption may not reflect actual community experience and should be refined in future analyses.
The priority areas identified are analytical priority areas based on the project's accessibility model—they are not official humanitarian priority zones. They are intended to guide further investigation, not to serve as the sole basis for intervention decisions.
________________________________________
How to Reproduce
1.	Clone this repository:
bash
git clone https://github.com/YOUR_USERNAME/Nairobi-Health-Accessibility.git
2.	Download raw data using the links in data/sources.md.
3.	Open the QGIS project in QGIS 3.44 or later:
text
qgis/project.qgz
4.	Verify CRS — all layers should be in EPSG:32737 (WGS 84 / UTM Zone 37S).
5.	Run the workflow in the order described in the Methodology section above.
________________________________________
Project Structure
text
Nairobi_Community_Health_Accessibility/
│
├── README.md                    # This file
├── LICENSE                      # MIT License
│
├── data/
│   ├── sources.md               # URLs for all raw data
│   ├── raw/                     # Original downloads (not tracked in git)
│   ├── processed/               # Clipped/reprojected data
│   └── derived/                 # Reclassified rasters, Accessibility Index
│
├── qgis/
│   └── project.qgz              # QGIS project file (relative paths)
│
├── maps/                        # Final map exports (PNG/PDF, 300 DPI)
│   ├── Map1_Facility_Distribution.png
│   ├── Map2_Population_Terrain.png
│   ├── Map3_Accessibility_Index.png
│   └── Map4_Priority_Areas.png
│
├── outputs/                     # CSV tables and statistics
│
└── documentation/
    ├── methodology.md           # Detailed step-by-step narrative
    ├── data_sources.md          # Full citations
    └── limitations.md           # Extended limitations discussion
________________________________________
Data Citations
•	Population: WorldPop (2025). Kenya Population 2026 (R2025A v1). WorldPop, University of Southampton. https://www.worldpop.org
•	Roads: OpenStreetMap contributors (2026). Nairobi Road Network. https://www.openstreetmap.org
•	Administrative Boundaries: Kenya Open Data Portal (2026). Nairobi County & Ward Boundaries. https://opendata.go.ke
•	Health Facilities: Humanitarian Data Exchange (HDX) (2026). Kenya Health Facilities. https://data.humdata.org
•	Elevation: NASA SRTM (2025). Shuttle Radar Topography Mission (SRTM) 30m DEM. https://www.earthdata.nasa.gov
________________________________________
Author
Earnest Maina
•	📞 +254 701 075 200
•	📍 Nairobi, Kenya
________________________________________
License
This project is licensed under the MIT License — see the LICENSE file for details.
________________________________________
Acknowledgements
This project was completed as a self-directed GIS training exercise focused on mastering the integration of vector and raster analysis, spatial joins, proximity analysis, reclassification, raster calculator modelling, and cartographic production.
________________________________________
Last updated: September 2026
