---
title: "Building an interactive Broadband Data Explorer application using Python, PostgreSQL/PostGIS, and Folium."
date: 2025-08-10
excerpt: <br/><img width="1201" src="https://github.com/user-attachments/assets/67601fe4-b7d2-40c4-9b1d-f49939c817d5" />"
collection: portfolio
---
<img width="1201" height="830" alt="2025-08-10 15_28_39" src="https://github.com/user-attachments/assets/67601fe4-b7d2-40c4-9b1d-f49939c817d5" />

## Introduction  

To help analyze broadband availability in a given state using FCC's publicly available dataset, I developed an interactive **Broadband Data Explorer** tool using Python, PostgreSQL/PostGIS, and Folium. This post details the development process, tools used, programming methodologies, and potential improvements.

---

## **1. Project Overview**  

The **Broadband Data Explorer** is a desktop application that:  
- Imports FCC broadband deployment data  
- Stores it in a PostgreSQL database with PostGIS for geospatial analysis  
- Provides an interactive map and filterable table view  
- Allows users to visualize coverage gaps and compare technologies
- Allows users to click symbology on a map and view detailed information in a popup  

### **Key Features**  
- **Data Import & Processing** – Handles FCC CSV/Shapefiles  
- **Interactive Map** – Folium/Leaflet-based visualization  
- **Filterable Table** – Pandas/QtTableWidget integration  
- **Cross-Platform** – Works on Windows, macOS, and Linux  

---

## **2. Tools & Technologies Used**  

### **Core Stack**  
| Technology       | Purpose                          |  
|------------------|----------------------------------|  
| **Python 3.10+** | Backend logic & GUI              |  
| **PostgreSQL**   | Relational database storage      |  
| **PostGIS**      | Geospatial queries               |  
| **PyQt5**        | Desktop GUI framework            |  
| **Folium**       | Interactive Leaflet maps in Qt   |  

### **Key Libraries**  
- **Pandas** – Data cleaning/transformation  
- **GeoPandas** – Shapefile processing  
- **Psycopg2** – PostgreSQL adapter  
- **PyInstaller** – Packaging to executable  

---

## **3. Development Process**  

### **3.1 Data Pipeline Architecture**  

The tool follows an **ETL (Extract, Transform, Load)** workflow:  

1. **Extract**  
   - FCC data (ZIP/CSV) → Pandas DataFrames  
   - Census block geometries (Shapefiles) → GeoPandas  

2. **Transform**  
   ```python  
   # Example: Clean technology codes  
   tech_mapping = {  
       10: "Asymmetric xDSL",  
       50: "Fiber to Premises"  
   }  
   df['tech_name'] = df['technology'].map(tech_mapping)  
   ```  

3. **Load**  
   - Bulk inserts via `psycopg2.extras.execute_values()`  
   - Spatial indexing for performance:  
     ```sql  
     CREATE INDEX idx_blocks_geometry ON census_blocks USING GIST(geometry);  
     ```  

---

### **3.2 Database Design**  

**Optimized Schema:**  
```sql  
CREATE TABLE providers (  
    provider_id BIGINT PRIMARY KEY,  
    brand_name VARCHAR(255)  
);  

CREATE TABLE census_blocks (  
    geoid BIGINT PRIMARY KEY,  
    geometry GEOMETRY(MULTIPOLYGON, 4326)  
);  

CREATE TABLE broadband_data (  
    technology INTEGER,  -- FCC tech codes (10, 40, etc.)  
    max_download_speed INTEGER,  
    block_geoid BIGINT REFERENCES census_blocks(geoid)  
);  
```  

**Key Decisions:**  
- **BIGINT for GEOIDs** – Ensures compatibility with FCC/Census data  
- **PostGIS Geometry** – Enables spatial joins (e.g., "show coverage in this area")  
- **Composite Indexes** – Speeds up provider+technology queries  

---

### **3.3 Interactive Map Implementation**  

**Folium + PyQt5 Integration:**  
1. Generate Leaflet maps in Python:  
   ```python  
   m = folium.Map(location=[41.5, -71.5], zoom_start=10)  
   FastMarkerCluster(data=locations).add_to(m)  
   ```  
2. Embed in Qt using `QWebEngineView`:  
   ```python  
   self.map_view.setHtml(m._repr_html_())  
   ```  

**Popup Design:**  
```python  
popup = folium.Popup(  
    f"""<b>{provider}</b><br>Speed: {speed} Mbps""",  
    max_width=300  
)  
```  

---

### **3.4 Filtering System**  

**Qt Combo Box Logic:**  
```python  
self.tech_combo.addItem("Fiber (50)", 50)  # Display text vs. stored value  

# Apply filters via SQL WHERE clauses:  
filters = {  
    'technology': 50,  
    'min_download': 100  
}  
```  

**Live SQL Query Updates:**  
```sql  
SELECT * FROM broadband_data  
WHERE technology = 50 AND max_download_speed >= 100;  
```  

---

## **4. Codebase Structure**  
```
broadband-data-explorer/  
│   README.md
│   requirements.txt
│
├───data
│   ├───census_blocks
│   │       tl_2024_44_tabblock20.zip
│   │
│   └───fcc_data
│           bdc_44_Cable_fixed_broadband.zip
│           bdc_44_Copper_fixed_broadband.zip
│           bdc_44_FibertothePremises_fixed_broadband.zip
│           bdc_44_GSOSatellite_fixed_broadband.zip
│           bdc_44_LicensedFixedWireless_fixed_broadband.zip
│           bdc_44_NGSOSatellite_fixed_broadband.zip
│           bdc_44_UnlicensedFixedWireless_fixed_broadband.zip
│
└───src
    ├── broadband_app.py    # Qt main window
    ├── config.py           # Configuration params
    ├── database.py         # Schema setup  
    ├── data_loader.py      # ETL pipelines 
    ├── map_utils.py        # Folium integration
    ├── reset_db.py
    ├── utils.py
    └── __init__.py
```

---

## **5. Potential Improvements**  

### **5.1 Enhanced Features**  
- **Speed Test Integration** – Compare advertised vs. actual speeds, such as through crowdsourced datasets  
- **Coverage Gap Analysis** – Identify unserved/underserved blocks  

### **5.2 Technical Upgrades**  
- **Async Loading** – Prevent GUI freezing during large imports  
- **Vector Tiles** – Improve performance for national-scale maps  

### **5.3 Deployment**  
- **Web Version** – Dash/Plotly port  
- **Docker Support** – Simplified PostGIS setup through containerization
