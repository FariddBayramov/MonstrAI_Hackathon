# MostrAI Hackathon - Understanding Life Through Data and Personalization:

This project, developed within the scope of the Mapindata Hackathon, aimed to leverage data not just as numbers, but as a tool to understand people, cities, and life itself. Our goal was to determine a "wealth score" for individuals living in or passing through a given region, and to categorize these individuals into multiple "persona" groups that reflect their multifaceted behaviors.

By blending diverse datasets—from restaurant and hotel segments to mobile signal data, demographic information, and amenity areas—we didn't just focus on income levels. Instead, we revealed an individual's "life value" based on various dimensions such as access, consumption habits, mobility, and lifestyle preferences. This allowed us to capture multiple personality traits like "Pet Lover," "Office Explorer," or "Gourmet Follower" through data, ultimately creating a unique and dynamic profile for each person.

Our project aims to be a valuable decision-support system across various fields, from smart city planning to target audience analysis, thanks to the insights we've gained.

---

## Files

### Notebook files

Run the Jupyter Notebook files in the project in order to create the necessary CSV files:

1. `99_kisi_id.ipynb`
2. `restoran_preprocessing.ipynb`
3. `restoran_cluster.ipynb`
4. `person1-dateplace.ipynb`
5. `person2-favoriler-timecluster.ipynb`
6. `person3-rich_score.ipynb`
7. `person4-persona.ipynb`

### CSV files

&#x20;Files edited manually in Excel or Jupyter Notebook:

1. `Clustered_Veteriner.csv`

   Classification was made in Excel as A B C according to the neighborhood name.
2. `Kahve_New.csv`

   In Excel, the data types of the `latitude` and `longitude` columns have been changed to float. Unnecessary columns have been removed.

# Notebooks
Note: Before running Notebooks, the `Polygons` folder and Mobility Data paraquet data must be manually added to the `data` folder.
---

## 1. 99\_kisi\_id.ipynb

This notebook creates a new DataFrame by selecting the first 99 unique `device_aid` values ​​from the large mobility dataset (MobilityDataMay2024.parquet).

The reason why 99 IDs are selected is that the RAM memory fills up when the DataFrame is read.

1. **Installing Dask and Pandas**: `dask.dataframe` and `pandas` are imported.
2. **Reading and Filtering Data**:

   * Parquet file is read with Dask.
   * Unique IDs are taken from the `device_aid` column.
   * The first 99 IDs are selected and the motion data is filtered for these IDs.

---

## 2. restoran\_preprocessing.ipynb

This notebook reads and cleans the raw restaurant data to prepare it for analysis and clustering.

1. **Loading Libraries**: Import the `pandas` and `numpy` packages.
2. **Reading Data**: Load the main dataset (`Hackathon_MainData.xlsx`) in Excel format.
3. **Column Selection**: Create a new DataFrame (`df`) by selecting the columns required for analysis:

   * Latitude, Longitude, District, Type, Modern/Traditional/Hotel values
   * Average Spending Amount, Restaurant Type
   * Map Profile & Population Score, Mapin Segment.
     
4. **Correcting Incorrect Values

   * The capital letters and Turkish character missing spellings in the district column are corrected.
   * The spelling error (`TRADITIONAL`) in the sales channel names is corrected.
   * The restaurant type is updated as 'Balik' → 'Balık'.
5. **Clearing Coded Values**:

   * Modern (`D`), Traditional (`R`) ve Hotel (`H`) değerlerinden harfler çıkarılır.
6. **Formatting Scores and Coordinates**:

   * Latitude, Longitude, Map Profile/Population scores are converted from string to float format.
   * Five-digit numbers are divided by 1000 to bring them to a thousandths scale.
7. **Average Spend Conversion

   *The average of the range values ​​"10-20 TL" and values ​​such as "+30" are converted into numbers as the lower limit.
8. **Filling in Missing Values**:

   * `NaN` values ​​in Modern, Traditional, Hotel columns are filled with 0.
9. **Final Check and Record**:

   * The cleaned DataFrame is printed to the screen and saved to the `MainData_updated.csv` file.
10. **Column List**:

* The resulting columns are listed in the console.

---

## 3. restoran\_cluster.ipynb

This notebook applies KMeans clustering analysis on the data cleaned in step one and separates the restaurants into clusters on the map.

1. **Loading Libraries and Data**: Import `pandas`, `sklearn.cluster.KMeans`, `matplotlib` and `seaborn`; Read `MainData_updated.csv`.

2. **Filling in Missing Average Spending**:

   * Empty `Ortalama Harcama Tutarı` values ​​are filled with the average value of the column.
3. **NaN Row Removal**:

* Rows with missing `Map Profile Score` or `Map Population Score` are removed.
4. 4. **One-Hot Encoding**:

* İlçe column is converted to binary columns with `pd.get_dummies`.
5. **Remove Unnecessary Columns**:

   * The columns `Mapin Segment` and `Hotel Value` are deleted.
6. **Restaurant Type Assembly and Cleaning**:

   * Categories with few samples are merged or deleted..
7. **Sales Channel Filter**:

   * `Hotel` values ​​are removed from the `Tür` column
8. **Removal of 0 Scored Records**:

   * Rows with `Map Profile Score == 0` are extracted.
9. **Feature Selection and Normalization**:

   * `Ortalama Harcama Tutarı` and `Map Profile Score` are selected and scaled with `StandardScaler`.
10. **KMeans**:

    * Clusters are calculated with `n_clusters=3`.
11. **Visualization**:

    * Clusters are coloured with latitude-longitude scatter plot.
12. **Elbow Method Analysis**:

    * Inertia is calculated for k values between 1-10.**Elbow Method Analysis**:
    * Inertia is calculated for k values between 1-10.
13. **Cluster Labelling and Registration**:

    * The clusters are labelled as "Zengin Restoran", "Orta Halli Restoran", "Ucuz Restoran" and `Clustered_Restaurants.csv` is saved.

---

## 4. person1-dateplace.ipynb

This notebook matches the movement data of 99 selected devices with different POI types.

1. **Libraries**: `dask.dataframe`, `pandas`, `numpy`, `sklearn.neighbors.BallTree`.
2. **Filter by 99 IDs**:

   * Load movement data using `99_kisi_data.csv`.
3. **Timing and Location Processing**:

   * Convert `timestamp` column to datetime, add `zaman_bolumu`.
   * The filter `horizontal_accuracy` ≤ 200 is applied.
4. **Primary Records**:n

   * The first record is selected for each `device_aid`, `grid_id`, `date`, `zaman_bolumu` combination.
5. **Restaurant/Veterinarian/Coffee Shop Pairing**:

   * With `Clustered_Restaurants.csv`, `Clustered_Veteriner.csv`, `Kahve_New.csv` the closest POI is matched using BallTree/Haversine.
6. **POI Type Determination**:

   * The `matched_*` columns are processed and the `Place` column is created by selecting the closest category.
7. **Result**:

   * The file `id-date-zaman_bolumu-place.csv` is saved.

---

## 5. person2-favoriler-timecluster.ipynb

For each device, this notebook classifies the categories of venues visited, identifies favourite venues and performs time-set analysis.

1. **Data Upload**: Read `id-date-zaman_bolumu-place.csv`.
2. **Category Glossary**:

   * Groups such as Hospital, Park, Market, Restaurant etc. are defined according to keywords.
3. **categorize Function**:

   * Category assignment is made over the venue name.
4. **Pivot Tables**:

   * By time zones (`zaman_pivot`) and by space categories (`mekan_pivot`) the number of visits is calculated.
5. **Time Clustering**:

   * The optimal k is found with the Elbow method, and `time_cluster` is labelled with KMeans.
6. **Favourite Places**:

   * Frequently visited categories according to the visit rate are stored in the `favouriler` list.
7. **CSV**:

   * `id-favoriler-time_cluster.csv` is saved.

---

## 6. person3-rich\_score.ipynb

This notebook calculates a "zenginlik skoru" (`rich_score`) based on the neighbourhood, number of visits and demographics of the devices.

1. **Data and Demographics**:

   * `id-favoriler-time_cluster.csv` and `Ilce_Demografi.xlsx` are uploaded.
2. **Night Neighbourhood Analysis**:

   * "The neighbourhood most frequently visited during the "night" period is selected as `oturdugu_mahalle`.
3. **Visit Attributes**:

   * Restaurant, luxury villa and site visit numbers are calculated with pivot.
4. **Neighbourhood Level Matching**:

   * Neighbourhood levels (`A`, `B`, `C`) read from Excel are added to each device.
5. **Attribute Scaling**:

   * With `StandardScaler` and `MinMaxScaler` the numerical attributes are normalised.
6. **Rich Score Calculation**:

   * Scores are summed with weighted contribution factors (e.g. villa ×0.35, restaurant ×0.25, neighbourhood ×0.25).
7. **CSV**:

   * `id-mahalle-rich_score.csv` is created.

---

## 7. person4-persona.ipynb

This notebook combines favourites, time sets and wealth score to create the final persona dataset.

1. **Data Upload**: Read `id-favoriler-time_cluster.csv` and `id-mahalle-rich_score.csv`.
2. **Consolidation**: `Axes are concatenated with `pd.concat`.
3. **Final CSV**: `final.csv` file is saved and the process is completed.

![final_data](https://github.com/user-attachments/assets/f31043ee-6d7b-4c9d-8c83-f16ec3d833cd)

