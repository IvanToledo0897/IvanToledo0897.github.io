---
title: 'Movilidad urbana y productividad económica en ciudades de LATAM'
description: Empece con la limpieza y preparación de datos para su análisis, resultando en las métricas necesarias para entender si existe correlación entre altos retrasos por congestión de autos y el PIB de Países en Latinoamérica. El resultado fue un reporte realizado en Jupyter Notebook con la documentación completa del Proceso en Python. El reporte presento una respuesta contundente a la correlación de los parámetros analizados junto con la sugerencia de inversión en ciudades clave.

publishDate: '9 Marzo 2026'
seo:
  image:
    src: '../../assets/images/project-1.jpg'
    alt: Project preview
---

![Project preview](../../assets/images/project-1.jpg)

**Note:** This case study is entirely fictional and created for the purpose of showcasing [Dante Astro.js theme functionality](https://justgoodui.com/astro-themes/dante/).

**Project Overview:**
EcoBuddy is a revolutionary mobile application designed to make sustainable living accessible, engaging, and rewarding. With a focus on gamification and real-world impact, EcoBuddy encourages users to adopt eco-friendly habits, reduce their carbon footprint, and contribute to a healthier planet.

## Objectives

1. Develop a user-friendly mobile app that motivates individuals to adopt sustainable practices in their daily lives.
2. Utilize gamification elements to make sustainable living fun and interactive.
3. Provide educational resources and personalized challenges to empower users to make informed eco-conscious decisions.

## Features

1. **EcoScore and Challenges:**

- Users are assigned an EcoScore based on their sustainable activities and choices.
- Daily and weekly challenges encourage users to adopt new habits and compete with friends or the community to earn EcoPoints.

2. **Personalized Eco-Goals:**

- Users can set and track personalized eco-goals, such as reducing plastic usage, conserving water, or choosing eco-friendly transportation.
- The app provides tips and suggestions to help users achieve their goals.

3. **Green Rewards Marketplace:**

- EcoPoints earned through challenges and sustainable actions can be redeemed in a virtual Green Rewards Marketplace.
- The marketplace offers discounts on eco-friendly products, services, and even contributions to environmental causes.

4. **Community Hub:**

- A community feature allows users to connect, share their eco-friendly achievements, and inspire others.
- Users can join local eco-groups, organize clean-up events, and collaborate on sustainability projects.

5. **EcoEducator AI Assistant:**

- An AI-powered assistant, EcoEducator, provides personalized eco-tips, facts, and information based on users' preferences and habits.
- Users can chat with EcoEducator for instant advice on sustainable living.

## Technology Stack

- Frontend: React Native for cross-platform mobile app development.
- Backend: Firebase for real-time data synchronization and user authentication.
- Database: Firestore for scalable and flexible data storage.
- AI Integration: Dialogflow for natural language processing and conversation with EcoEducator.

## Outcome

EcoBuddy has successfully created a community of environmentally conscious individuals who actively participate in sustainable living practices. The app not only educates and motivates users but also provides tangible rewards for their commitment to a greener lifestyle, fostering a positive impact on the environment.

## Client Testimonial

> We couldn't be happier with the results delivered by Ethan Donovan. From the initial concept discussions to the final product, their responsiveness and collaborative approach were impressive. Our startup's website now stands out, thanks to their creative input and commitment to excellence.

**Note:** This case study is entirely fictional and created for the purpose of showcasing [Dante Astro.js theme functionality](https://justgoodui.com/astro-themes/dante/).

## Introducción

Como analista de datos, el objetivo es **evaluar cómo la movilidad urbana se relaciona con la productividad económica en las principales ciudades latinoamericanas**. 
Para ello trabajaré con datos reales de TomTom Traffic Index y OECD Cities; debo limpiar, combinar y analizar para identificar en qué ciudades conviene invertir en infraestructura de transporte.

## 🧩 Paso 1: Cargar y explorar
 
Antes de limpiar o combinar los datos, es necesario **revisar la estructura de ambos datasets**.
Validar que los archivos se carguen correctamente, conocer sus columnas y tipos de datos, y detectar posibles inconsistencias.
 
### 1.1 Carga de datos y vista rápida
 
**🎯Objetivo:**
Importar las librerías necesarias, cargar los archivos CSV en DataFrames y realizar una revisión preliminar para entender su contenido.

### Importar librerías
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

### Cargar archivos
```python
traffic = pd.read_csv('/datasets/tomtom_traffic.csv')
eco = pd.read_csv('/datasets/oecd_city_economy.csv')
```

### Mostrar las primeras 5 filas de traffic
```python
traffic.head()
```

### Mostrar las primeras 5 filas de eco
```python
eco.head()
```

---

## 🧩Paso 2: Explorar, limpiar y preparar los datos

Anotar las columnas que necesiten limpieza y luego estandarizar los nombres de columnas.
 
### 2.1 Explorar la estructura y tipos de datos
 
**🎯Objetivo:**
Identificar columnas con tipos incorrectos, distribución y nulos, anotar las columnas que requieren conversión.
 
### Examinar la estructura de traffic
```python
traffic.info()
```

### Comentarios personales
En la estructura del DF traffic, se observa que:
 - Las columnas `UpdateTimeUTC` y `UpdateTimeUTC` son de tipo objeto cuando deberian ser datetime. Lo mismo ocurre con `UpdateTimeUTCWeekAgo`.
 - Las columnas `Country` y `City` son object en vez de `STRING`.
 - JamsCount estaría mejor como `INT`.


### Examinar la estructura de eco
```python
eco.info()
```

### Comentarios personales
En la estructura del DF eco, se observa que:
 - Las columnas `City GDP/capita`, `Unemployment %`, `PM2.5(μg/m³)` y `Population (M)` son `object` en vez de `FLOAT`.
 - Las columnas `City`, `Country` son object en vez de `STRING`.

### 2.2 Renombrar columnas
 
**🎯Objetivo:**
Estandarizar los nombres de columnas para evitar errores y facilitar la unión de los datasets.
 
### Estandarizar los nombres de las columnas de traffic
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
### verificar cambios
```python
traffic.columns
```

### Estandarizar los nombres de las columnas de eco
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

### verificar cambios
```python
eco.columns
```

 
### 2.3 Corregir formatos numéricos y de fecha
 
**🎯Objetivo:**
Asegurar que las columnas de fechas y valores numéricos estén en formatos correctos para permitir análisis, cálculos y comparaciones precisas.
 
### Convertir las columnas de traffic a tipo fecha con `pd.to_datetime()`
```python
traffic['update_time_utc'] = pd.to_datetime(traffic['update_time_utc'], errors="coerce", utc=True)
traffic['update_time_utc_week_ago'] = pd.to_datetime(traffic['update_time_utc_week_ago'], errors="coerce", utc=True)
```

### verificar el cambio
```python
traffic.info()
```

### Limpia separadores y convierte columnas numéricas en eco
```python
eco['city_gdp_capita'] = eco['city_gdp_capita'].astype(str).str.replace('.', '').str.replace(',', '.').astype(float)
eco['unemployment_pct'] = eco['unemployment_pct'].astype(str).str.replace('%', '').str.replace(',', '.').astype(float)
eco['population_m'] = eco['population_m'].astype(str).str.replace(',', '.').astype(float)
```

### Calcula la población total en unidades absolutas (Multiplica * 1000000)
```python
eco['population'] = eco['population_m']*1000000
```

### verificar el cambio
```python
eco.info()
eco.head(3)
```

---

## 🧩Paso 3: Extraer año y filtrar
 
Extraer el año permite filtrar la información y trabajar solo con el período más reciente y relevante.
 
### 3.1 Extraer columna año y filtrar 2024
 
**🎯Objetivo**
Identificar el año de cada registro y mantener solo los registros del 2024.
 
### Extraer el año de las fechas en update_time_utc
```python
traffic['year'] = traffic['update_time_utc'].dt.year
```

### Verificar cambio
```python
traffic.head(3)
```

### Filtra los registros del año 2024
```python
traffic_2024 = traffic[traffic['year'] == 2024].copy()
eco_2024 = eco[eco['year'] == 2024].copy()
```

### Revisar dataframes nuevos
```python
display(traffic_2024.head())
display(eco_2024.head())
```

 
---
 
## 🧩Paso 4: Analizar y resumir datos de movilidad
 
Como el dataset de tráfico contiene **múltiples registros por ciudad**. En esta parte, calcularás los promedios anuales por ciudad para simplificar el análisis y obtener una visión más clara de las tendencias generales.
 
### 4.1 Calcular promedios de tráfico por ciudad
 
**🎯Objetivo:**
Obtener una vista consolidada del tráfico promedio por ciudad y año, para analizar patrones generales sin depender de datos diarios.

### Calcular los  promedios de trafico por ciudad, país y año
```python
traffic_city_year_2024 = traffic_2024\
    .groupby(['city', 'country', 'year'])[['jams_delay', 'traffic_index_live', 'jams_length_kms', 'jams_count', 'mins_delay', 'travel_time_live_per_10kms_mins', 'travel_time_hist_per_10kms_mins']]\
    .agg('mean')\
    .reset_index()
```

### Mostrar resultado
```python
traffic_city_year_2024.head()
```



### 🧠 **Momento de reflexión**
```python
traffic_city_year_2024.sort_values(["jams_delay"], ascending=False)
```

La ciudad con el mayor tiempo promedio de tráfico es la Ciudad de Mexico con 2833 min aprox de retraso (jams_delay).

---

## 🧩Paso 5: Unir movilidad y economía
 
Combinar datasets permite analizar cómo se relacionan los indicadores económicos con los de movilidad.
 
### 5.1 Unir tráfico (tabla principal) con indicadores económicos
 
**🎯Objetivo:**
Combinar la información de tráfico y economía en un solo DataFrame para analizar cómo las condiciones económicas se relacionan con la movilidad urbana.
 
### Seleccionar columnas clave de tráfico y economía
```python
left_cols = ['city','country','year','jams_delay','traffic_index_live',
             'jams_length_kms','jams_count','mins_delay',
             'travel_time_live_per_10kms_mins','travel_time_hist_per_10kms_mins']

right_cols = ['city','year','city_gdp_capita','unemployment_pct','pm25','population']
```

### Usar .copy() para crear los dos nuevos datasets reducidos
```python
traffic_2024_small = traffic_city_year_2024[left_cols].copy()
eco_2024_small = eco_2024[right_cols].copy()

traffic_2024_small.head()
eco_2024_small.head()
```

### Unir datasets
```python
merged = pd.merge(traffic_2024_small,eco_2024_small,on=['city','year'],how='inner')
```

### Mostrar las primeras 5 filas
```python
merged.head()
```

---
 
## 🧩Paso 6: Visualización y análisis de relaciones
 
Ya con un dataset limpio y unificado, es momento de **visualizar patrones**.

### 6.1 Visualizar relaciones entre economía y tráfico
 
**🎯Objetivo:**
Analizar visualmente la distribución y la relación entre indicadores de tráfico y economía en 2024, para identificar posibles patrones o tendencias generales entre ambas variables.

### Crear boxplot para observar el comportamiento de los minutos de congestion JamsDelay

```python
sns.boxplot(data=merged, x='jams_delay', showmeans=True)
```

### obtener promedio para mostrarlo en título
```python
mean_value = merged['jams_delay'].mean()
plt.title(f'Boxplot de JamsDelay (2024)\nPromedio: {mean_value:.2f}')
plt.xlabel("Retraso promediio en minutos")
plt.show()
```

### Crear histograma para ver la distribución de la economía (city_gdp_capita)
```python
merged['city_gdp_capita'].hist(bins=5,figsize=(10,5))
plt.title("Distribución de la economía")
plt.ylabel("Numero de ciudades")
plt.xlabel("Valor promedio PIB per capita")
```

### Gráfico de barras para comparar jams_delay y city_gdp_capita por ciudad
```python
merged.plot( kind='bar' , y=['jams_delay', 'city_gdp_capita'],x='city')
plt.xticks(rotation=90)
plt.xlabel('Ciudades')
plt.ylabel('PIB per capita y retraso (min)')
plt.title('Comparativa entre retraso de trafico en minutos y PIB per capita')
plt.show()
```


### 🧠 **Momento de reflexión**
Las ciudades con mayor congestión tienden a tener un PIB mayor, parece una tendencia más que una regla.

--- 

## 🧩Paso 7: Exportar y documentar resultados
 
En esta etapa final consolidarás todo tu trabajo: guardarás el dataset limpio y crearás un resumen que documente los resultados del proyecto.
 
### 7.1 Guardar dataset final
 
**🎯Objetivo:**
Generar un CSV limpio, reproducible y con columnas relevantes para análisis posterior.

### Exporta el dataset final como CSV
```python
merged.to_csv("ladb_mobility_economy_2024_clean.csv", index=False)
```


---

## ✅ Entregables

1. **Notebook `.ipynb`** con todas las celdas (código + comentarios).
2. **CSV final**: `ladb_mobility_economy_2024_clean.csv`.
3. **Resumen ejecutivo breve** en Markdown (3–5 párrafos).

---

## 🧾 Resumen ejecutivo (plantilla)
- Las variables claves fueron la ciudad, el PIB per capita y los jams_delays (retraso en minutos por congestión). Encontré una ligera tendencia tendencia entre un PIB y la congestión, pero no hay una relación directa entre la movibilidad y la productividad.
- Filtré solamente el año 2024, 15 ciudades de paises como México, Perú, Brazil, Colombia, Uruguay, Chile y Argentina
- Realicé analisis de 2 Datasets: tomtom_traffic.csv y oecd_city_economy.csv. Realicé correcciones de columnas y formatos. Filtre a entradas en el año 2024 y paises de America Latina. Obtuve el promedio de retrasos en tiempo por trafico y uní ambos Datasets por modalidad "inner" por ciudad y año. Posterior a eso realice grafica boxplot para encontrar anomalías, histograma para la observar la distribución de economía y una grafica de barras para comparar el PIB con el índice de trafico.
- Si bien si aparece una tendencia entre un mayor PIB y los mayores índices de trafico, no es una relación contundente. En ciudades como Brasilia, Buenos Aires y Rio de Janeiro hay valores altos de PIB sin índices altos de trafico. Las ciudades de Montevideo y Ciudad de Mexico requieren un analisis más profundo, ya que la primera tiene el mayor PIB pero índice más bajo y la Ciudad de México rebasa por mucho el promedio de retrasos por trafico.
- Las ciudades de Santiago, Lima, Sao Paulo y Bogota son las ciudades que tienen un menor PIB y un mayor índice de trafico. Lo que puede sugerir ser una ciudad prioritaria.
