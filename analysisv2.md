# Draft


```python
import pandas as pd
df = pd.read_csv('./globalterrorismdb.csv', low_memory=False, engine='c')
```

## Overview

The **Global Terrorism Database** (GTD) is an extensive resource that tracks terrorist attacks across the world, offering data from over 200,000 incidents between 1970 and 2020. This dataset, maintained by the National Consortium for the Study of Terrorism and Responses to Terrorism (START) at the University of Maryland, serves as an invaluable tool for understanding the global patterns, causes, and consequences of terrorism. By analyzing the GTD, we gain insights into trends such as the methods, regions, and targets involved in terrorism, as well as the political, economic, and social contexts that drive these violent acts.

In this project, we leverage the GTD to explore the geographical hotspots of terrorism, uncover trends over time, and analyze the effectiveness and tactics of terrorist groups. With over four decades of data at our disposal, the goal is to identify patterns in terrorist activity, visualize global hotspots, and discuss key findings that contribute to understanding the global impact of terrorism.

The insights from this data can serve as a foundation for policymakers, security agencies, and researchers aiming to mitigate terrorism’s impact. By focusing on trends over time, geographical distribution, and methods of attack, we aim to provide a comprehensive view of the state of terrorism worldwide, offering a clear perspective on both historical and emerging trends.

## Preprocessing Phase
After selecting the necessary columns for analysis, the next steps in the data cleaning process involve addressing missing values, handling incorrect or inconsistent data, and preparing the data for analysis. Here’s a breakdown of the key steps I took:

One of the first things I had to tackle was dealing with missing or null values. This is common in large datasets, especially one like the Global Terrorism Database (GTD), where certain data points might be missing due to incomplete reports or the limitations of data collection in conflict zones. Missing values can lead to biased or inaccurate analysis, so it’s important to handle them carefully.

In the case of this project, where we are exploring terrorism hotspots, trends over time, and methods of attack, the missing data could significantly impact our insights if not addressed properly.


```python
# this gets the number of null values for each attribute(column)
df.isnull().sum()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>eventid</th>
      <td>0</td>
    </tr>
    <tr>
      <th>iyear</th>
      <td>0</td>
    </tr>
    <tr>
      <th>imonth</th>
      <td>0</td>
    </tr>
    <tr>
      <th>iday</th>
      <td>0</td>
    </tr>
    <tr>
      <th>approxdate</th>
      <td>197017</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>INT_LOG</th>
      <td>0</td>
    </tr>
    <tr>
      <th>INT_IDEO</th>
      <td>0</td>
    </tr>
    <tr>
      <th>INT_MISC</th>
      <td>0</td>
    </tr>
    <tr>
      <th>INT_ANY</th>
      <td>0</td>
    </tr>
    <tr>
      <th>related</th>
      <td>179102</td>
    </tr>
  </tbody>
</table>
<p>135 rows × 1 columns</p>
</div><br><label><b>dtype:</b> int64</label>



From the output above, we can see that some columns have null values while 11 columns no null values.

- Latitude and longitude have over 4600+ missing values. Since these columns are crucial for mapping geographical hotspots, I decided to drop rows with missing latitude and longitude data to ensure accurate location-based analysis.

- For columns like nkill and nwound (fatalities and wounded), which had over 12,000 and 19,000 missing values, I filled the missing values with 0, assuming no casualties or injuries were reported for these incidents. This approach preserved the data for further analysis without excluding important rows.

- For columns like targsubtype1_txt (target subtype) and natlty1_txt (target nationality), I filled missing values with "Unknown", as these fields weren't critical for the overall analysis but still added value for categorization and grouping.

 To clean up duplicates, I removed redundant rows using df.drop_duplicates(inplace=True). Lastly, I standardized columns like attacktype1_txt by converting all text to lowercase and stripping extra spaces to ensure consistency across categories. Here's the code I used for these steps:


```python
# Drop rows with missing latitude and longitude
df = df.dropna(subset=['latitude', 'longitude'])

# Fill missing values for fatalities and injuries with 0
df['nkill'] = df['nkill'].fillna(0)
df['nwound'] = df['nwound'].fillna(0)

# Fill missing categorical values with 'Unknown'
df['targsubtype1_txt'] = df['targsubtype1_txt'].fillna('Unknown')
df['natlty1_txt'] = df['natlty1_txt'].fillna('Unknown')

# Remove duplicate rows
df = df.drop_duplicates()

# Standardize text columns
df['attacktype1_txt'] = df['attacktype1_txt'].str.lower().str.strip()


```

To streamline the dataset and focus on the project's goals, I narrowed it down to only the columns essential for analyzing trends, hotspots, and attack characteristics. These included fields for date, location, attack type, targets, and impact, such as `iyear`, `country_txt`, `attacktype1_txt`, `nkill`, and `gname`. By selecting just these columns, I ensured the dataset remained manageable and directly relevant to uncovering insights about terrorism.


```python
needed_columns = [
    'iyear', 'imonth', 'iday', 'country_txt', 'region_txt', 'latitude', 'longitude',
    'attacktype1_txt', 'weaptype1_txt', 'targtype1_txt', 'targsubtype1_txt', 'target1', 'natlty1_txt',
    'gname', 'nkill', 'nwound', 'suicide', 'success'
]
df = df[needed_columns]
df.head()
```





  <div id="df-af65064a-d3e9-4e16-8f02-5bf3cc00875a" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>iyear</th>
      <th>imonth</th>
      <th>iday</th>
      <th>country_txt</th>
      <th>region_txt</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>attacktype1_txt</th>
      <th>weaptype1_txt</th>
      <th>targtype1_txt</th>
      <th>targsubtype1_txt</th>
      <th>target1</th>
      <th>natlty1_txt</th>
      <th>gname</th>
      <th>nkill</th>
      <th>nwound</th>
      <th>suicide</th>
      <th>success</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1970</td>
      <td>7</td>
      <td>2</td>
      <td>Dominican Republic</td>
      <td>Central America &amp; Caribbean</td>
      <td>18.456792</td>
      <td>-69.951164</td>
      <td>assassination</td>
      <td>Unknown</td>
      <td>Private Citizens &amp; Property</td>
      <td>Named Civilian</td>
      <td>Julio Guzman</td>
      <td>Dominican Republic</td>
      <td>MANO-D</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1970</td>
      <td>0</td>
      <td>0</td>
      <td>Mexico</td>
      <td>North America</td>
      <td>19.371887</td>
      <td>-99.086624</td>
      <td>hostage taking (kidnapping)</td>
      <td>Unknown</td>
      <td>Government (Diplomatic)</td>
      <td>Diplomatic Personnel (outside of embassy, cons...</td>
      <td>Nadine Chaval, daughter</td>
      <td>Belgium</td>
      <td>23rd of September Communist League</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1970</td>
      <td>1</td>
      <td>0</td>
      <td>Philippines</td>
      <td>Southeast Asia</td>
      <td>15.478598</td>
      <td>120.599741</td>
      <td>assassination</td>
      <td>Unknown</td>
      <td>Journalists &amp; Media</td>
      <td>Radio Journalist/Staff/Facility</td>
      <td>Employee</td>
      <td>United States</td>
      <td>Unknown</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1970</td>
      <td>1</td>
      <td>0</td>
      <td>Greece</td>
      <td>Western Europe</td>
      <td>37.997490</td>
      <td>23.762728</td>
      <td>bombing/explosion</td>
      <td>Explosives</td>
      <td>Government (Diplomatic)</td>
      <td>Embassy/Consulate</td>
      <td>U.S. Embassy</td>
      <td>United States</td>
      <td>Unknown</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1970</td>
      <td>1</td>
      <td>0</td>
      <td>Japan</td>
      <td>East Asia</td>
      <td>33.580412</td>
      <td>130.396361</td>
      <td>facility/infrastructure attack</td>
      <td>Incendiary</td>
      <td>Government (Diplomatic)</td>
      <td>Embassy/Consulate</td>
      <td>U.S. Consulate</td>
      <td>United States</td>
      <td>Unknown</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-af65064a-d3e9-4e16-8f02-5bf3cc00875a')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-af65064a-d3e9-4e16-8f02-5bf3cc00875a button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-af65064a-d3e9-4e16-8f02-5bf3cc00875a');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


<div id="df-c87fcdd0-aa4f-4cb0-a86b-5775f0a35822">
  <button class="colab-df-quickchart" onclick="quickchart('df-c87fcdd0-aa4f-4cb0-a86b-5775f0a35822')"
            title="Suggest charts"
            style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
  </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

  <script>
    async function quickchart(key) {
      const quickchartButtonEl =
        document.querySelector('#' + key + ' button');
      quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
      quickchartButtonEl.classList.add('colab-df-spinner');
      try {
        const charts = await google.colab.kernel.invokeFunction(
            'suggestCharts', [key], {});
      } catch (error) {
        console.error('Error during call to suggestCharts:', error);
      }
      quickchartButtonEl.classList.remove('colab-df-spinner');
      quickchartButtonEl.classList.add('colab-df-quickchart-complete');
    }
    (() => {
      let quickchartButtonEl =
        document.querySelector('#df-c87fcdd0-aa4f-4cb0-a86b-5775f0a35822 button');
      quickchartButtonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';
    })();
  </script>
</div>

    </div>
  </div>




After preprocessing the structure of our database looks like this:


```python
row, column = df.shape
print(f"Number of rows: {row}")
print(f"Number of columns: {column}")
```

    Number of rows: 205014
    Number of columns: 18


With that, we have `3498894` cells.

## Analysis and Insight

#### Trend over the years


```python
import seaborn as sns
import matplotlib.pyplot as plt

yearly_trends = df.groupby('iyear').size().reset_index(name='incidents')

# Custom style settings
custom = {
    "axes.edgecolor": "white",
    "grid.linestyle": "solid",
}
sns.set_style('whitegrid', rc=custom)

# Increase the figure width to make room for more year intervals
plt.figure(figsize=(18, 10))  # Increased the width of the figure to fit more years

# Create the line plot
sns.lineplot(data=yearly_trends, x='iyear', y='incidents')

# Adjust x-axis tick labels to show intervals of 5 years
plt.xticks(ticks=range(yearly_trends['iyear'].min(), yearly_trends['iyear'].max() + 1, 5))

# Title and labels
plt.title('Trend of Terrorist Incidents Over the Years', fontsize=20)
plt.xlabel('Year', fontsize=16)
plt.ylabel('No. Incidents', fontsize=16)

# Add grid lines for clarity
plt.grid(axis='x')
```


    
![png](analysisv2_files/analysisv2_14_0.png)
    


#### Regional Comparison


```python
regional_trends = df.groupby('region_txt').size().reset_index(name='incidents')
sns.set_color_codes("muted")
plt.figure(figsize=(14, 10))
ax = sns.barplot(data=regional_trends, x='incidents', y='region_txt')
ax.bar_label(ax.containers[0])
plt.title('Regional Distribution of Terrorist Incidents', fontsize=20)
plt.xlabel('No. Incidents', fontsize=16)
plt.ylabel('Region', fontsize=16)
plt.show()
```


    
![png](analysisv2_files/analysisv2_16_0.png)
    


#### Common Attack Methods


```python
attack_type_data = df.groupby(['region_txt', 'attacktype1_txt']).size().unstack()
attack_type_data.plot(kind='bar', stacked=True, figsize=(12, 8), colormap='coolwarm')
plt.title('Regional Breakdown of Common Attack Methods', loc='left')
plt.xlabel('Region', fontsize=13)
plt.ylabel('No. Incidents', fontsize=13)
plt.grid(axis='x')
plt.show()

```


    
![png](analysisv2_files/analysisv2_18_0.png)
    



```python
import geopandas as gpd
# Load a world map shapefile (comes with GeoPandas)
world = gpd.read_file("./ne_110m_admin_0_countries.shp")

# Convert your dataset into a GeoDataFrame
gdf = gpd.GeoDataFrame(
    df,
    geometry=gpd.points_from_xy(df['longitude'], df['latitude']),
    crs="EPSG:4326"  # WGS84 Coordinate Reference System
)

# Plot the world map
fig, ax = plt.subplots(figsize=(15, 10))
world.plot(ax=ax, color='lightgrey', edgecolor='black')  # Base map

# Overlay terrorism data
gdf.plot(ax=ax, markersize=5, color='red', alpha=0.5, label='Terrorist Incidents')
ax.grid(False)
# Add a legend and title
plt.legend()
plt.title('Global Distribution of Terrorist Incidents')
plt.show()
```


    
![png](analysisv2_files/analysisv2_19_0.png)
    


#### Correlation between incidents and casualties



```python
# Scatter plot for fatalities vs. injuries with a larger figure size
plt.figure(figsize=(14, 10))  # Increase figure size (width, height)
sns.scatterplot(data=df, x='nkill', y='nwound', hue='region_txt', palette='viridis')

# Add titles and labels with larger font sizes
plt.title('Correlation Between Fatalities and Injuries', fontsize=20)
plt.xlabel('No. Fatalities', fontsize=16)
plt.ylabel('No. Injuries', fontsize=16)

# Show grid with custom properties for better readability
plt.grid(True, which='both', linestyle='--', linewidth=0.5)

# Show the plot
plt.show()
```


    
![png](analysisv2_files/analysisv2_21_0.png)
    



```python
correlation = df['nkill'].corr(df['nwound'])
print(f"Correlation: {correlation}")
```

    Correlation: 0.4408038022325402

