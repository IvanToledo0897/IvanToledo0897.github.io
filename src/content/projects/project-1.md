---
title: 'Movilidad urbana y productividad económica en ciudades de LATAM'
description: Proyecto para Latin American Development Bank, preparando datos para probar la correlación entre índices altos de tráfico y el PIB en ciudades de LatinoAmerica

publishDate: '9 Marzo 2026'
seo:
  image:
    src: '../../assets/images/project-1.jpg'
    alt: Project preview
---

![Project preview](../../assets/images/project-1.jpg)

## Introducción

El objetivo es **evaluar cómo la movilidad urbana se relaciona con la productividad económica en las principales ciudades latinoamericanas**. 
Para ello trabajaré con datos reales de TomTom Traffic Index y OECD Cities; debo limpiar, combinar y analizar para identificar en qué ciudades conviene invertir en infraestructura de transporte.

## Paso 1: Conocer los datos.
 
Antes de limpiar o combinar los datos, es necesario **revisar la estructura de ambos datasets**.
Validar que los archivos se carguen correctamente, conocer sus columnas y tipos de datos, y detectar posibles inconsistencias.

### 1.1 Importar librerías
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

### 1.2 Cargar archivos
```python
traffic = pd.read_csv('/datasets/tomtom_traffic.csv')
eco = pd.read_csv('/datasets/oecd_city_economy.csv')
```

#### Primeras 5 filas de traffic
```python
traffic.head()
```

#### Primeras 5 filas de eco
```python
eco.head()
```

---

## Paso 2: Limpieza y preparacion
 
### 2.1 Estructura y tipos de datos
 
#### Estructura de traffic
```python
traffic.info()
```

#### Comentarios
En la estructura del DF traffic:
 - Las columnas `UpdateTimeUTC` y `UpdateTimeUTC` son de tipo objeto cuando deberian ser datetime. Lo mismo ocurre con `UpdateTimeUTCWeekAgo`.
 - JamsCount estaría mejor como `INT`.


#### Estructura de eco
```python
eco.info()
```

#### Comentarios personales
En la estructura del DF eco, se observa que:
 - Las columnas `City GDP/capita`, `Unemployment %`, `PM2.5(μg/m³)` y `Population (M)` son `object` en vez de `FLOAT`.

### 2.2 Renombrar columnas
 
#### 2.2.1 Estandarizar los nombres de las columnas de traffic
```python
traffic=traffic.rename(columns={
    'Country':'country',
    'City':'city',
    'UpdateTimeUTC':'update_time_utc',
    'JamsDelay':'jams_delay',
    'TrafficIndexLive':'traffic_index_live',
    'JamsLengthInKms':'jams_length_kms',
    'JamsCount':'jams_count',
    'TrafficIndexWeekAgo':'traffic_index_week_ago',
    'UpdateTimeUTCWeekAgo':'update_time_utc_week_ago',
    'TravelTimeLivePer10KmsMins':'travel_time_live_per_10kms_mins',
    'TravelTimeHistoricPer10KmsMins':'travel_time_hist_per_10kms_mins',
    'MinsDelay':'mins_delay'})
```
#### Verifico cambios
```python
traffic.columns
```

#### 2.2.2 Estandarizar los nombres de las columnas de eco
```python
eco=eco.rename(columns={
    'Year':'year',
    'City':'city',
    'Country':'country',
    'City GDP/capita':'city_gdp_capita',
    'Unemployment %':'unemployment_pct',
    'PM2.5 (μg/m³)':'pm25',
    'Population (M)':'population_m'})
```

#### Verifico cambios
```python
eco.columns
```

 
### 2.3 Corregir formatos numéricos y de fecha
 
#### Convertir las columnas de traffic a tipo fecha
```python
traffic['update_time_utc'] = pd.to_datetime(traffic['update_time_utc'], errors="coerce", utc=True)
traffic['update_time_utc_week_ago'] = pd.to_datetime(traffic['update_time_utc_week_ago'], errors="coerce", utc=True)
```

#### Verifico el cambio
```python
traffic.info()
```

#### Limpiar separadores y conversion a columnas numéricas en eco
```python
eco['city_gdp_capita'] = eco['city_gdp_capita'].astype(str).str.replace('.', '').str.replace(',', '.').astype(float)
eco['unemployment_pct'] = eco['unemployment_pct'].astype(str).str.replace('%', '').str.replace(',', '.').astype(float)
eco['population_m'] = eco['population_m'].astype(str).str.replace(',', '.').astype(float)
```

#### Calcula la población total en unidades absolutas (Multiplica * 1000000)
```python
eco['population'] = eco['population_m']*1000000
```

#### Verifico el cambio
```python
eco.info()
eco.head(3)
```

---

## Paso 3: Extraer año y filtrar
 
Me han pedido que trabaje solo con el año 2024.

### 3.1 Extraer el año de las fechas en update_time_utc
```python
traffic['year'] = traffic['update_time_utc'].dt.year
```

#### Verifico cambio
```python
traffic.head(3)
```

### 3.2 Filtra año 2024
```python
traffic_2024 = traffic[traffic['year'] == 2024].copy()
eco_2024 = eco[eco['year'] == 2024].copy()
```

#### Revisar dataframes nuevos
```python
display(traffic_2024.head())
display(eco_2024.head())
```

## Paso 4: Analizar y resumir datos de movilidad
 
Como el dataset de tráfico contiene múltiples registros por ciudad. Haré los promedios anuales por ciudad para simplificar el análisis y obtener una visión más clara de las tendencias generales.

```python
traffic_city_year_2024 = traffic_2024\
    .groupby(['city', 'country', 'year'])[['jams_delay', 'traffic_index_live', 'jams_length_kms', 'jams_count', 'mins_delay', 'travel_time_live_per_10kms_mins', 'travel_time_hist_per_10kms_mins']]\
    .agg('mean')\
    .reset_index()
```

#### Resultado
```python
traffic_city_year_2024.head()
```



#### Momento chisme
```python
traffic_city_year_2024.sort_values(["jams_delay"], ascending=False)
```

La ciudad con el mayor tiempo promedio de tráfico es la Ciudad de Mexico con 2833 min aprox de retraso (jams_delay).

---

## Paso 5: Unir movilidad y economía
 
Combinar datasets permite analizar cómo se relacionan los indicadores económicos con los de movilidad.
 
#### Seleccionar columnas clave de tráfico y economía
```python
left_cols = ['city','country','year','jams_delay','traffic_index_live',
             'jams_length_kms','jams_count','mins_delay',
             'travel_time_live_per_10kms_mins','travel_time_hist_per_10kms_mins']

right_cols = ['city','year','city_gdp_capita','unemployment_pct','pm25','population']
```

#### Usar `.copy()` para crear los dos nuevos datasets reducidos
```python
traffic_2024_small = traffic_city_year_2024[left_cols].copy()
eco_2024_small = eco_2024[right_cols].copy()

traffic_2024_small.head()
eco_2024_small.head()
```

#### Unir datasets
```python
merged = pd.merge(traffic_2024_small,eco_2024_small,on=['city','year'],how='inner')
```

#### Mostrar las primeras 5 filas
```python
merged.head()
```
 
## Paso 6: Visualización y análisis de relaciones
 
Ya con un dataset limpio y unificado, es momento de **visualizar patrones**.

### 6.1 Boxplot para observar el comportamiento de los minutos de congestion JamsDelay

```python
sns.boxplot(data=merged, x='jams_delay', showmeans=True)
```

### 6.2 Promedio para mostrarlo en título
```python
mean_value = merged['jams_delay'].mean()
plt.title(f'Boxplot de JamsDelay (2024)\nPromedio: {mean_value:.2f}')
plt.xlabel("Retraso promediio en minutos")
plt.show()
```

### 6.3 Histograma para ver la distribución de la economía (city_gdp_capita)
```python
merged['city_gdp_capita'].hist(bins=5,figsize=(10,5))
plt.title("Distribución de la economía")
plt.ylabel("Numero de ciudades")
plt.xlabel("Valor promedio PIB per capita")
```

### 6.4 Gráfico de barras para comparar jams_delay y city_gdp_capita por ciudad
```python
merged.plot( kind='bar' , y=['jams_delay', 'city_gdp_capita'],x='city')
plt.xticks(rotation=90)
plt.xlabel('Ciudades')
plt.ylabel('PIB per capita y retraso (min)')
plt.title('Comparativa entre retraso de trafico en minutos y PIB per capita')
plt.show()
```


#### Momento chisme
Las ciudades con mayor congestión tienden a tener un PIB mayor, parece una tendencia más que una regla.


## Paso 7: Exportar y documentar resultados
 
```python
merged.to_csv("ladb_mobility_economy_2024_clean.csv", index=False)
```


---

## Entregables

1. **Notebook `.ipynb`** con todas las celdas (código + comentarios).
2. **CSV final**: `ladb_mobility_economy_2024_clean.csv`.

### 🧾 Resumen ejecutivo (plantilla)
- Las variables claves fueron la ciudad, el PIB per capita y los jams_delays (retraso en minutos por congestión). Encontré una ligera tendencia tendencia entre un PIB y la congestión, pero no hay una relación directa entre la movibilidad y la productividad.
- Filtré solamente el año 2024, 15 ciudades de paises como México, Perú, Brazil, Colombia, Uruguay, Chile y Argentina
- Realicé analisis de 2 Datasets: tomtom_traffic.csv y oecd_city_economy.csv. Realicé correcciones de columnas y formatos. Filtre a entradas en el año 2024 y paises de America Latina. Obtuve el promedio de retrasos en tiempo por trafico y uní ambos Datasets por modalidad "inner" por ciudad y año. Posterior a eso realice grafica boxplot para encontrar anomalías, histograma para la observar la distribución de economía y una grafica de barras para comparar el PIB con el índice de trafico.
- Si bien si aparece una tendencia entre un mayor PIB y los mayores índices de trafico, no es una relación contundente. En ciudades como Brasilia, Buenos Aires y Rio de Janeiro hay valores altos de PIB sin índices altos de trafico. Las ciudades de Montevideo y Ciudad de Mexico requieren un analisis más profundo, ya que la primera tiene el mayor PIB pero índice más bajo y la Ciudad de México rebasa por mucho el promedio de retrasos por trafico.
- Las ciudades de Santiago, Lima, Sao Paulo y Bogota son las ciudades que tienen un menor PIB y un mayor índice de trafico. Lo que puede sugerir ser una ciudad prioritaria.
