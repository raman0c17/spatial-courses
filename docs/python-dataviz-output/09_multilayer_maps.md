# Creating Multi-layer Interactive Maps

[Folium](https://python-visualization.github.io/folium/) supports creating maps with multiple layers. Recent versions of GeoPandas have built-in support to create interactive folium maps from a GeoDataFrame. 

In this section, we will create a multi-layer interactive map using 2 vector datasets.


```python
import os
import folium
import geopandas as gpd
```


```python
data_pkg_path = 'data'
filename = 'karnataka.gpkg'
path = os.path.join(data_pkg_path, 'roads', filename)
roads_gdf = gpd.read_file(path, layer='karnataka_major_roads')
districts_gdf = gpd.read_file(path, layer='karnataka_districts')
state_gdf = gpd.read_file(path, layer='karnataka')

```

We can use the [explore()](https://geopandas.org/en/stable/docs/reference/api/geopandas.GeoDataFrame.explore.html) method to create an interactive folium map from the GeoDataFrame.


```python
districts_gdf.explore()
```


```python
districts_gdf.explore(
    color='black', 
    style_kwds={'fillOpacity': 0.3, 'weight': 0.5},
    tiles='Stamen Terrain')
```


```python
roads_gdf
```


```python
def get_category(row):
    ref = str(row['ref'])
    if 'NH' in ref:
        return 'NH'
    elif 'SH' in ref:
        return 'SH'
    else:
        return 'NA'
    
roads_gdf['category'] = roads_gdf.apply(get_category, axis=1)
roads_gdf
```


```python
filtered = roads_gdf[roads_gdf['category'].isin(['NH', 'SH'])]
filtered
```


```python
filtered.explore(
    column='category', 
    categories=['NH', 'SH'], 
    cmap=['#1f78b4', '#e31a1c'],
    categorical=True
)
```

You can have fine-grained control over how each feature is styled on the map. The `explore()` function takes a `style_function` parameter where you can specify a function that returns style properties using any of the supported parameters. The function is passed on each feature as a GeoJSON object and you can use any columns values to create your style. 

The function below will stlye all features of 'NH' category with thicker lines.


```python
def style(feature):
    if feature['properties']['category'] == 'NH':
        return {'weight': 2}
    else:
        return {'weight': 0.5}

filtered.explore(
    column='category', 
    categories=['NH', 'SH'], 
    cmap=['#1f78b4', '#e31a1c'],
    categorical=True,
    style_kwds={'style_function':style}
)

```

When you call `explore()` a folium object is created. You can save that object and add more layers to the same object.


```python
m = districts_gdf.explore(
    color='black', 
    style_kwds={'fillOpacity': 0.3, 'weight':0.5},
    name='districts',
    tooltip=False,
    tiles='Stamen Terrain')


def style(feature):
    if feature['properties']['category'] == 'NH':
        return {'weight': 2}
    else:
        return {'weight': 0.5}

filtered.explore(
    m=m,
    column='category', 
    categories=['NH', 'SH'], 
    cmap=['#1f78b4', '#e31a1c'],
    categorical=True,
    style_kwds={'style_function':style}
)

m
```

To make our map easier to explore, we also add a *Layer Control* that displays the list of layers on the top-right corner and also allows the users to turn them on or off. We need to add the `name` parameter to the `explore()` function with the name that will be displayed in the layer control.


```python
m = districts_gdf.explore(
    color='black', 
    style_kwds={'fillOpacity': 0.3, 'weight':0.5},
    tooltip=False,
    tiles='Stamen Terrain',
    name='Districts')


def style(feature):
    if feature['properties']['category'] == 'NH':
        return {'weight': 2}
    else:
        return {'weight': 0.5}

filtered.explore(
    m=m,
    column='category', 
    categories=['NH', 'SH'], 
    cmap=['#1f78b4', '#e31a1c'],
    categorical=True,
    style_kwds={'style_function':style},
    name='Highways')

folium.LayerControl().add_to(m)

m
```

## Exercise

Add the `state_gdf` layer to the folium map below with a thick blue border and no fill. Save the resulting map as a HTML file on your computer.

Hint: Use the `style_kwds` with *'fill'* and *'weight'* options.