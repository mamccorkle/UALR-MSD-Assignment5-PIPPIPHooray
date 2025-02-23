#### **Interactive Rotating Globe with Population Density**

#### **Results**

**Purpose of the script**

> The purpose of the script is that it gives "Log Scaled Population Density" interactive map. It displays a glode that gives population densities for browsed areas.

**Steps followed**

> As per the git comments, here is a list of actions taken:
>
>    + README.MD
>       Added a readme that will contain the instructions and observations
>    + main.py
>       Added the main python file with code that will be worked on or refactored
>    + [LIB] installed  geopandas
>    + [LIB] installed requests
>    + [LIB] installed plotly
>    + [LIB] installed pipdeptree
>    * Exported dependency tree
>    * Exported ALL dependencies tree
>    - [LIB] uninstalled all dependencies (Produces an error upon running)
>    + [LIB] reinstalled pip
>    + [LIB] installed all dependencies from dependency_tree.txt
>    * Verified script works as expected
>    * Reexported dependency_tree_v2.txt for comparison to the original dependency_tree.txt

**Output of pipdeptree**

>    geopandas==1.0.1<br>
>      - numpy [required: >=1.22, installed: 2.2.3]<br>
>      - packaging [required: Any, installed: 24.2]<br>
>      - pandas [required: >=1.4.0, installed: 2.2.3]<br>
>        - numpy [required: >=1.26.0, installed: 2.2.3]<br>
>        - python-dateutil [required: >=2.8.2, installed: 2.9.0.post0]<br>
>          - six [required: >=1.5, installed: 1.17.0]<br>
>        - pytz [required: >=2020.1, installed: 2025.1]<br>
>        - tzdata [required: >=2022.7, installed: 2025.1]<br>
>      - pyogrio [required: >=0.7.2, installed: 0.10.0]<br>
>        - certifi [required: Any, installed: 2025.1.31]<br>
>        - numpy [required: Any, installed: 2.2.3]<br>
>        - packaging [required: Any, installed: 24.2]<br>
>      - pyproj [required: >=3.3.0, installed: 3.7.1]<br>
>        - certifi [required: Any, installed: 2025.1.31]<br>
>      - shapely [required: >=2.0.0, installed: 2.0.7]<br>
>        - numpy [required: >=1.14,<3, installed: 2.2.3]<br>
>    pipdeptree==2.25.0<br>
>      - packaging [required: >=24.1, installed: 24.2]<br>
>      - pip [required: >=24.2, installed: 24.3.1]<br>
>    plotly==6.0.0<br>
>      - narwhals [required: >=1.15.1, installed: 1.27.1]<br>
>      - packaging [required: Any, installed: 24.2]<br>
>    requests==2.32.3<br>
>      - certifi [required: >=2017.4.17, installed: 2025.1.31]<br>
>      - charset-normalizer [required: >=2,<4, installed: 3.4.1]<br>
>      - idna [required: >=2.5,<4, installed: 3.10]<br>
>      - urllib3 [required: >=1.21.1,<3, installed: 2.3.0]<br>

**Observations or issues**

> The only issue I faced (easily fixed), occurred on step 4. This step removed pip along with all other dependencies available. Therefore, I had to reinstall pip manually before proceeding


#### **Objective**

The goal is to learn **dependency management** by manually handling package installations, generating a clean requirements.txt file, and verifying reproducibility.

**globe.py**

```python
import os
import geopandas as gpd
import pandas as pd
import numpy as np
import requests
import zipfile
import plotly.graph_objects as go

# Define dataset URL and local paths

DATA_URL = "https://naciscdn.org/naturalearth/110m/cultural/ne_110m_admin_0_countries.zip"
DATA_DIR = "data"
ZIP_PATH = os.path.join(DATA_DIR, "ne_110m_admin_0_countries.zip")
SHP_PATH = os.path.join(DATA_DIR, "ne_110m_admin_0_countries.shp")

# Ensure the data directory exists

os.makedirs(DATA_DIR, exist_ok=True)

# Download dataset if not present

if not os.path.exists(SHP_PATH):
    print("Dataset not found. Downloading...")
    response = requests.get(DATA_URL, stream=True)

    if response.status_code == 200:
        with open(ZIP_PATH, "wb") as file:
            file.write(response.content)
        print("Download complete.")

        # Extract the ZIP file
        print("Extracting dataset...")
        with zipfile.ZipFile(ZIP_PATH, "r") as zip_ref:
            zip_ref.extractall(DATA_DIR)
        print("Extraction complete.")
    else:
        raise Exception(f"Failed to download dataset (HTTP {response.status_code})")

# Load world data

world = gpd.read_file(SHP_PATH)

# Convert to a projected coordinate system for correct area calculations

world = world.to_crs(epsg=3857)  # Web Mercator projection

# Calculate country areas in square km

world["area_km2"] = world.geometry.area / 1e6

# Ensure valid population density calculation (handle missing data)

world["pop_est"] = pd.to_numeric(world["POP_EST"], errors="coerce")
world["pop_density"] = world["pop_est"] / world["area_km2"]

# Apply log scale to population density (use log1p to avoid log(0) issues)

world["pop_density_log"] = np.log1p(world["pop_density"])

# Convert geometries to latitude/longitude for Plotly

world = world.to_crs(epsg=4326)

# Create 3D interactive globe

fig = go.Figure()

fig.add_trace(go.Choropleth(
    geojson=world.__geo_interface__,  # Convert to GeoJSON for Plotly
    locations=world.index,
    z=world["pop_density_log"].fillna(0),  # Fill missing values with 0
    colorscale="plasma",  # Adjusted for better contrast with log scale
    zmin=world["pop_density_log"].min(),
    zmax=world["pop_density_log"].max(),
    marker_line_color="black",
    marker_line_width=0.3,
    colorbar_title="Log(Pop Density)",
))

fig.update_geos(
    projection_type="orthographic",  # 3D Sphere Projection
    showcoastlines=True,
    showland=True,
    showcountries=True,
)

fig.update_layout(
    title="Interactive Rotating Globe - Log Scaled Population Density",
    height=400,
    width=400,
)

# Show the interactive globe

fig.show()
```

### Disclaimers

> - This will use a lot of data.  Run somewhere will it will be fast and cheap for you.
> - The downloaded .zip and ultimately .shp files should not be hosted on your git.  Make sure to ignore them.
> - "pip uninstall -y -r all_requirements.txt" should only be run in a virtual environment.  To be sure your terminal prompt should look like "(.venv) PS C:\Users\nicho\PycharmProjects\PythonProject>" indicating that the virtual environment is active.

### **Project Setup**

You will:

> 1. **Set up a virtual environment** for dependency isolation.
> 2. **Manually install necessary libraries**.
> 3. **Generate a full requirements file** (all_requirements.txt) and refine it into requirements.txt.
> 4. **Remove all installed packages** to simulate a clean environment.
> 5. **Reinstall dependencies using requirements.txt**.
> 6. **Verify the script works** after reinstalling.
> 7. **Analyze dependencies using pipdeptree**.
> 8. **Document everything in a README.md**.

### **Instructions**

### ** :white_check_mark: 1. Set Up Your Repository**

> - Create a **new directory** in your Git repository for this assignment.
> - Initialize a **new Python project**.
> - **Create and activate a virtual environment**:
> - If using **PyCharm**, enable the option to **automatically create a virtual environment** when setting up the project.
> - Otherwise, manually create and activate the environment:
> 
> python -m venv venv<br>
> source venv/bin/activate   # macOS/Linux<br>
> venv\Scripts\activate      # Windows

### ** :white_check_mark: 2. Manually Install Required Packages**

Using pip, install the necessary libraries until you get the program to correctly run:

> pip install geopandas pandas ...
> 
> {plotly is used for the interactive rotating globe, geopandas for country geometries, and requests for downloading the dataset if missing.}

### ** :white_check_mark: 3. Generate a Full Requirements File (all_requirements.txt)**

After installing the required packages, export **all dependencies**:

> pip freeze > all_requirements.txt
> 
> {To create a cleaner requirements file, use pipdeptree (or pipreqs) to determine the minimal required dependencies. This ensures we avoid unnecessary packages.}
> 
> pip install pipdeptree<br>
> pipdeptree > dependency_tree.txt<br>
> pipreqs . --force  # Generates a minimal requirements.txt<br>

### ** :white_check_mark: 4. Remove All Installed Packages**

To simulate a **clean environment**, remove all installed packages **and verify** that pip list only shows pip:

> pip uninstall -y -r all_requirements.txt
> pip list
> 
> {Ensure that running python globe.py at this point results in an import error, proving that dependencies are missing.}

### ** :white_check_mark: 5. Reinstall from requirements.txt**

> Reinstall only the **minimal required** dependencies:
> 
> pip install -r requirements.txt
> 
> {This should install only the necessary libraries while avoiding unnecessary dependencies.}

### ** :white_check_mark: 6. Verify That the Script Works**

> Run the Python script to confirm that it functions correctly.
> 
> {The interactive globe should open in a **standalone window**, allowing rotation and zoom. If errors occur, check that requirements.txt includes all necessary dependencies.}

### ** :white_check_mark: 7. Analyze Dependencies with pipdeptree** 

> Generate and save the dependency tree:
> 
> pipdeptree > dependency_tree.txt
> 
> {This file should contain a hierarchical list of dependencies, allowing you to understand which packages are required by others.}

### ** :white_check_mark: 8. Document Your Work**

Create a README.md file in your project directory. It should include:

> - 📌 **Purpose of the script**: What it does and why.
> - 📝 **Steps followed**: A concise list of what was done.
> - 📊 **Output of pipdeptree** (copy from dependency_tree.txt).
> - 🔍 **Observations or issues**: Any insights, errors, or optimizations found during the process.

