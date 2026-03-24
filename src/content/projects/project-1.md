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

#### Importar librerías
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

#### Cargar archivos
```python
traffic = pd.read_csv('/datasets/tomtom_traffic.csv')
eco = pd.read_csv('/datasets/oecd_city_economy.csv')
```

#### Mostrar las primeras 5 filas de traffic
```python
traffic.head()
```

#### Mostrar las primeras 5 filas de eco
```python
eco.head()
```

---

## 🧩Paso 2: Explorar, limpiar y preparar los datos

Anotar las columnas que necesiten limpieza y luego estandarizar los nombres de columnas.
 
### 2.1 Explorar la estructura y tipos de datos
 
**🎯Objetivo:**
Identificar columnas con tipos incorrectos, distribución y nulos, anotar las columnas que requieren conversión.
 
#### Examinar la estructura de traffic
```python
traffic.info()
```

#### Comentarios personales
En la estructura del DF traffic, se observa que:
 - Las columnas `UpdateTimeUTC` y `UpdateTimeUTC` son de tipo objeto cuando deberian ser datetime. Lo mismo ocurre con `UpdateTimeUTCWeekAgo`.
 - Las columnas `Country` y `City` son object en vez de `STRING`.
 - JamsCount estaría mejor como `INT`.


#### Examinar la estructura de eco
```python
eco.info()
```

#### Comentarios personales
En la estructura del DF eco, se observa que:
 - Las columnas `City GDP/capita`, `Unemployment %`, `PM2.5(μg/m³)` y `Population (M)` son `object` en vez de `FLOAT`.
 - Las columnas `City`, `Country` son object en vez de `STRING`.
 -Cambiar variables a <code>int</code>


### 2.2 Renombrar columnas
 
**🎯Objetivo:**
Estandarizar los nombres de columnas para evitar errores y facilitar la unión de los datasets.
 
#### Estandarizar los nombres de las columnas de traffic
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
# verificar cambios
traffic.columns

# **Comentario a revisor:**
# Intente usar la sintaxis recomendada en la leccion de limpieza y organización del dataset (traffic.columns=["nuevos","nombres","de","columnas"]), pero me salía error y tras varios intentos solo pude dejarlo con el metodo de rename tradicional. Sucede con ambos Datasets.


# Estandarizar los nombres de las columnas de eco
#tu código aquí

eco=eco.rename(columns={
    'Year':'year',
    'City':'city',
    'Country':'country',
    'City GDP/capita':'city_gdp_capita',
    'Unemployment %':'unemployment_pct',
    'PM2.5 (μg/m³)':'pm25',
    'Population (M)':'population_m'})

# verificar cambios
eco.columns

# 
# ### 2.3 Corregir formatos numéricos y de fecha
# 
# **🎯Objetivo:**
# Asegurar que las columnas de fechas y valores numéricos estén en formatos correctos para permitir análisis, cálculos y comparaciones precisas.
# 
# **Instrucciones:**
# 
# - Convierte las columnas de fecha de `traffic` a formato `datetime`. Haz el cambio a prueba de errores.
# - En el dataset `eco`, limpia los valores numéricos:
#     - En `city_gdp_capita`: elimina separadores de miles (`.`) y reemplaza las comas (`','`) por puntos (`'.'`) antes de convertir a tipo `float`.
#     - En `unemployment_pct`: elimina el símbolo de porcentaje (`%`) y reemplaza las comas (`','`) por puntos (`'.'`) antes de convertir a tipo `float`.
#     - En `population_m`: reemplaza las comas (`','`) por puntos (`'.'`) antes de convertir a tipo `float`.
# - Finalmente, crea una nueva columna llamada `population` multiplicando `population_m` por 1,000,000 para obtener la población total.
# 


# <details>
# <summary>Haz clic para ver la pista</summary>
# para eliminar símbolos, puedes reemplazarlos por un texto vacío.


# Convertir las columnas de traffic a tipo fecha con pd.to_datetime()
traffic['update_time_utc'] = pd.to_datetime(traffic['update_time_utc'], errors="coerce", utc=True)
traffic['update_time_utc_week_ago'] = pd.to_datetime(traffic['update_time_utc_week_ago'], errors="coerce", utc=True)

# verificar el cambio
traffic.info()

# Limpia separadores y convierte columnas numéricas en eco
eco['city_gdp_capita'] = eco['city_gdp_capita'].astype(str).str.replace('.', '').str.replace(',', '.').astype(float)
eco['unemployment_pct'] = eco['unemployment_pct'].astype(str).str.replace('%', '').str.replace(',', '.').astype(float)
eco['population_m'] = eco['population_m'].astype(str).str.replace(',', '.').astype(float)

# Calcula la población total en unidades absolutas (Multiplica * 1000000)
eco['population'] = eco['population_m']*1000000

# verificar el cambio
eco.info()
eco.head(3)

# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# Bien hecho!<br/>
# 
# Los datos fueron revisados y modificados apropiadamente, ahora se puede empezar a trabajar con ellos comodamente
# </div>
# 


# 
# ---
# 
# ## 🧩Paso 3: Extraer año y filtrar
# 
# Extraer el año permite filtrar la información y trabajar solo con el período más reciente y relevante.
# 
# ### 3.1 Extraer columna año y filtrar 2024
# 
# **🎯Objetivo**
# Identificar el año de cada registro y mantener solo los registros del 2024.
# 
# **Intrucciones**
# 
# - Como el DataFrame `traffic` no tiene una columna de año, utiliza el atributo `.dt.year` sobre su columna de fecha para crear una nueva columna llamada `year`.
# - Filtra las filas donde el año sea **2024**.
# - Utiliza `.copy()` para crear dos nuevos DataFrames (`traffic_2024` y `eco_2024`) para evitar modificar el dataset original.


# Extraer el año de las fechas en update_time_utc
traffic['year'] = traffic['update_time_utc'].dt.year

# Verificar cambio
traffic.head(3)

# Filtra los registros del año 2024
traffic_2024 = traffic[traffic['year'] == 2024].copy()
eco_2024 = eco[eco['year'] == 2024].copy()

# Revisar dataframes nuevos
display(traffic_2024.head())
display(eco_2024.head())


# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# Buena manera de extraer el año y de filtrar los datos, ahora podemos trabajar con los datos requeridos
# </div>
# 


# 
# ---
# 
# ## 🧩Paso 4: Analizar y resumir datos de movilidad
# 
# Como el dataset de tráfico contiene **múltiples registros por ciudad**. En esta parte, calcularás los promedios anuales por ciudad para simplificar el análisis y obtener una visión más clara de las tendencias generales.
# 
# ### 4.1 Calcular promedios de tráfico por ciudad
# 
# **🎯Objetivo:**
# Obtener una vista consolidada del tráfico promedio por ciudad y año, para analizar patrones generales sin depender de datos diarios.
# 
# **Instrucciones**
# 
# - Agrupa los datos por `city`, `country` y `year`.
# - Calcula el promedio **solo de las métricas de tráfico más relevantes**: como `jams_delay`, `traffic_index_live`, `jams_length_kms`, `jams_count`, `mins_delay`, y tiempos de viaje (`travel_time_live_per_10kms_mins` y `travel_time_hist_per_10kms_mins`).
# - Guarda el resultado como `traffic_city_year_2024`, mantén las columnas como variables (no índices).
# 


# <details>
# <summary>Haz clic para ver la pista</summary>
# Usa ".agg()" para aplicar funciones de promedio. Al final, reinicia el índice para mantener las columnas de la agrupación como variables (no índices).


# Calcular los  promedios de trafico por ciudad, país y año
traffic_city_year_2024 = traffic_2024\
    .groupby(['city', 'country', 'year'])[['jams_delay', 'traffic_index_live', 'jams_length_kms', 'jams_count', 'mins_delay', 'travel_time_live_per_10kms_mins', 'travel_time_hist_per_10kms_mins']]\
    .agg('mean')\
    .reset_index()

# Mostrar resultado
traffic_city_year_2024.head()

# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# Excelente!<br/>
# 
# Has tomado las columnas correctas para hacer la agrupación del conjunto de datos, podemos ver ahora en las columnas los promedios de cada una de las ciudades
# </div>
# 


# ### 🧠 **Momento de reflexión**
# 
# ¡Excelente trabajo hasta aquí!
# 
# Ahora que ya tienes los promedios anuales por ciudad, es momento de **observarlos** con atención.
# 
# Piensa:
# 
# - ¿Cuál crees que tiene el mayor tiempo promedio de tráfico?
# - ¿Será una ciudad de **Europa**, de **Latinoamérica** o de **otra región** del mundo?
# 
# Para descubrirlo, ejecuta esta línea de código:
# 
# `traffic_city_year_2024.sort_values(["jams_delay"], ascending=False)`
# 
# 
# 🔍 Observa qué ciudad aparece en los primeros lugares.
# 
# ¿Te sorprenden los resultados? , ¿Coinciden con lo que imaginabas?


traffic_city_year_2024.sort_values(["jams_delay"], ascending=False)

# La ciudad con el mayor tiempo promedio de tráfico es la Ciudad de Mexico con 2833 min aprox de retraso (jams_delay).


# 
# ---
# 
# ## 🧩Paso 5: Unir movilidad y economía
# 
# Combinar datasets te permite analizar cómo se relacionan los indicadores económicos con los de movilidad.
# 
# ### 5.1 Unir tráfico (tabla principal) con indicadores económicos
# 
# **🎯Objetivo:**
# Combinar la información de tráfico y economía en un solo DataFrame para analizar cómo las condiciones económicas se relacionan con la movilidad urbana.
# 
# **Instrucciones**
# - Selecciona solo las **columnas relevantes** de cada dataset (por ejemplo, variables clave de tráfico y de economía).
# - Usa `.copy()` al crear subconjuntos para evitar modificar el dataset original.
# - Une ambos DataFrames y define como **claves de unión** a `city` y `year`.
# - Mantén solo las ciudades y años presentes en ambos datasets.
# - Guarda el resultado en una nueva variable llamada `merged` y muestra las primeras 5 filas.
# 


# <details>
# <summary>Haz clic para ver la pista</summary>
# Aplica una unión de tipo "inner" para mantener las ciudades y años presentes en ambos datasets.


# Seleccionar columnas clave de tráfico y economía
left_cols = ['city','country','year','jams_delay','traffic_index_live',
             'jams_length_kms','jams_count','mins_delay',
             'travel_time_live_per_10kms_mins','travel_time_hist_per_10kms_mins']

right_cols = ['city','year','city_gdp_capita','unemployment_pct','pm25','population']

# Usar .copy() para crear los dos nuevos datasets reducidos
traffic_2024_small = traffic_city_year_2024[left_cols].copy()
eco_2024_small = eco_2024[right_cols].copy()

#traffic_2024_small.head()
#eco_2024_small.head()
# Unir datasets
merged = pd.merge(traffic_2024_small,eco_2024_small,on=['city','year'],how='inner')

# Mostrar las primeras 5 filas
merged.head()

# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# Correcto!<br/>
# 
# La union de los datos se hizo de una buena manera al tomar solo las columnas necesarias de cada uno de los conjuntos de datos
# </div>
# 


# 
# ---
# 
# ## 🧩Paso 6: Visualización y análisis de relaciones
# 
# Ahora que tienes un dataset limpio y unificado, es momento de **visualizar patrones**.
# Los gráficos te ayudarán a entender cómo se relacionan las variables económicas con las de movilidad urbana.
# 
# ### 6.1 Visualizar relaciones entre economía y tráfico
# 
# **🎯Objetivo:**
# Analizar visualmente la distribución y la relación entre indicadores de tráfico y economía en 2024, para identificar posibles patrones o tendencias generales entre ambas variables.
# 
# **Instrucciones**
# - Usa las librerías `seaborn` y `matplotlib.pyplot` para generar los gráficos.
# - Visualiza la distribución del **tráfico** (`jams_delay`) mediante:
#     - **Boxplot** → para observar la media, mediana y detectar valores atípicos.
# - Visualiza la distribución de la **economía** (`city_gdp_capita`) mediante:
#     - **Histograma** → para analizar la forma de la distribución y el valor promedio del PIB per cápita.
# - Finalmente, **compara ambas variables**, para observar si existe alguna relación entre ellas, haciendo un solo gráfico de barras donde aparezcan ambos indicadores.
# - Recuerda agregar título y etiquetas a los ejes de tus gráficos.
# - Observa y comenta los patrones, valores extremos o posibles relaciones que identifiques.


# **Tip:** Dentro de los parentesis del boxplot, agrega `showmeans=True` para ver la media en el gráfico.


# Crear boxplot para observar el comportamiento de los minutos de congestion JamsDelay
# crea tu gráfico

sns.boxplot(data=merged, x='jams_delay', showmeans=True)

# obtener promedio para mostrarlo en título
mean_value = merged['jams_delay'].mean()
plt.title(f'Boxplot de JamsDelay (2024)\nPromedio: {mean_value:.2f}')
plt.xlabel("Retraso promediio en minutos")
plt.show()


merged

# Crear histograma para ver la distribución de la economía (city_gdp_capita)
merged['city_gdp_capita'].hist(bins=5,figsize=(10,5))
plt.title("Distribución de la economía")
plt.ylabel("Numero de ciudades")
plt.xlabel("Valor promedio PIB per capita")


# Gráfico de barras para comparar jams_delay y city_gdp_capita por ciudad
merged.plot( kind='bar' , y=['jams_delay', 'city_gdp_capita'],x='city')
plt.xticks(rotation=90)
plt.xlabel('Ciudades')
plt.ylabel('PIB per capita y retraso (min)')
plt.title('Comparativa entre retraso de trafico en minutos y PIB per capita')
plt.show()

# **Tip:** Antes del `plt.show()` agrega el código `plt.xticks(rotation=90)` para rotar las etiquetas del eje X en 90 grados.


# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# 
# La visualización de los datos me parece correcta, podemos ver los distintos patrones en cada una de las ciudades lo cual es bastante útil para hacer hallazgos interesantes
# </div>
# 


# ### 🧠 **Reflexiona**
# Excelente trabajo llegando a esta etapa del análisis. Antes de avanzar, revisa tus gráficos, tómate un momento para pensar:
# 
# * ¿Las ciudades con mayor PIB per cápita también presentan más congestión?
# 
# * ¿O sucede lo contrario, o no existe una relación clara?


# Escribe tus comentarios:
# Las ciudades con mayor congestión tienden a tener un PIB mayor, parece una tendencia más que una regla.


# 
# ---
# 
# ## 🧩Paso 7: Exportar y documentar resultados
# 
# En esta etapa final consolidarás todo tu trabajo: guardarás el dataset limpio y crearás un resumen que documente los resultados del proyecto.
# 
# ### 7.1 Guardar dataset final
# 
# **🎯Objetivo:**
# Generar un CSV limpio, reproducible y con columnas relevantes para análisis posterior.
# 
# **Instrucciones**
# 
# - Exporta el DataFrame `merged` con el nombre: `ladb_mobility_economy_2024_clean.csv`
# - Usa `index=False` para no incluir el índice.
# 


# Exporta el dataset final como CSV
merged.to_csv("ladb_mobility_economy_2024_clean.csv", index=False)

# Para poder ver o descargar el archivo generado:   
# En el menú lateral que esta a la izquierda, ve hasta la parte de abajo, a la sección de **Exportar dataset** para más información. 


# 
# ---
# 
# ## ✅ Entregables
# 
# 1. **Notebook `.ipynb`** con todas las celdas (código + comentarios).
# 2. **CSV final**: `ladb_mobility_economy_2024_clean.csv`.
# 3. **Resumen ejecutivo breve** en Markdown (3–5 párrafos).
# 


# 
# ---
# 
# # 🧾 Resumen ejecutivo (plantilla)
# 
# > Completa este resumen al finalizar el análisis. Mantén 3–5 párrafos cortos, claros y accionables.
# 
# **Contexto & objetivo:**  
# - Responde la pregunta central del análisis: ¿qué relación existe entre la movilidad urbana (congestión, tiempos de viaje) y la productividad económica (PIB per cápita)?
# - Explica brevemente las variables clave utilizadas y su relevancia para la toma de decisiones.
# 
# **R**. Las variables claves fueron la ciudad, el PIB per capita y los jams_delays (retraso en minutos por congestión). Encontré una ligera tendencia tendencia entre un PIB y la congestión, pero no hay una relación directa entre la movibilidad y la productividad.
# 
# **Cobertura de datos:**  
# - Especifica los años analizados, número de ciudades y países incluidos.
# 
# **R**. Filtré solamente el año 2024, 15 ciudades de paises como México, Perú, Brazil, Colombia, Uruguay, Chile y Argentina
# 
# **Metodología (alto nivel):**  
# - Describe los procesos principales: limpieza de datos (formatos, estandarización de columnas).
# - Explica la agregación por ciudad–año y el uso de una unión INNER para integrar tráfico y economía.
# - Menciona las validaciones visuales empleadas (distribuciones, outliers, tendencias generales).
# 
# **R**. Realicé analisis de 2 Datasets: tomtom_traffic.csv y oecd_city_economy.csv. Realicé correcciones de columnas y formatos. Filtre a entradas en el año 2024 y paises de America Latina. Obtuve el promedio de retrasos en tiempo por trafico y uní ambos Datasets por modalidad "inner" por ciudad y año. Posterior a eso realice grafica boxplot para encontrar anomalías, histograma para la observar la distribución de economía y una grafica de barras para comparar el PIB con el índice de trafico.
# 
# **Hallazgos iniciales:**  
# - Resume los patrones más importantes entre índices de tráfico y PIB per cápita.
# - Destaca anomalías u outliers que podrían requerir revisión adicional o un análisis más profundo.
# 
# **R**. Si bien si aparece una tendencia entre un mayor PIB y los mayores índices de trafico, no es una relación contundente. En ciudades como Brasilia, Buenos Aires y Rio de Janeiro hay valores altos de PIB sin índices altos de trafico. Las ciudades de Montevideo y Ciudad de Mexico requieren un analisis más profundo, ya que la primera tiene el mayor PIB pero índice más bajo y la Ciudad de México rebasa por mucho el promedio de retrasos por trafico.
# 
# **Recomendaciones**  
# Aterriza los hallazgos en acciones: ciudades prioritarias, necesidad de validar fuentes, requerimiento de análisis adicionales, o propuestas de inversión.
# 
# - ¿Qué ciudad : Bogotá, Lima o Buenos Aires o alguna otra en particular, muestra la mayor correlación significativa entre altos niveles de congestión vehicular y bajos indicadores de productividad económica, sugiriendo ser una ciudad prioritaria para inversión en infraestructura de transporte?
# 
# **R**. Las ciudades de Santiago, Lima, Sao Paulo y Bogota son las ciudades que tienen un menor PIB y un mayor índice de trafico. Lo que puede sugerir ser una ciudad prioritaria.
# 


# ## Comentario general del revisor
# <div class="alert alert-block alert-success">
# <b>Comentario del revisor</b> <a class="tocSkip"></a><br />
# Has realizado un muy buen trabajo al desarrollar este proyecto, las observaciones que has hecho a lo largo del mismo han servido para tomar acción en los pasos posteriores, terminando en resultados positivos.
# 
# Los datos utilizados en este proyecto son bastante útiles e interesantes para hacer hallazgos interesantes, ya que son ciudades que conocemos y podemos asimilar de una mejor manera el contexto en el que situa el proyecto.    
# 
# Continúa con el buen trabajo y mucho éxito en el siguiente Sprint!    
# </div>
# 


# <div class="alert alert-block alert-success">
# <b>Aspectos positivos del proyecto</b> <a class="tocSkip"></a><br />
# 
# - Las observaciones intermedias, así como las conclusiones finales me parecen buenas
# - Las graficas utilizadas me parecen del tipo correcto
# - El proyecto esta ordenado
#     
# </div>
