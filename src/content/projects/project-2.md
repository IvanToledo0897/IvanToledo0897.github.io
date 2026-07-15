---
title: 'Diagnóstico estratégico integral para RappiPlus'
description: Proyecto para RappiPlus, preparando datos para corroborar la estrategia de Rappiplus en sus ganacias y retención de usuarios.

publishDate: '14 Julio 2026'
seo:
  image:
    src: '../../assets/images/project-1.jpg'
    alt: Project preview
---

![Project preview](../../assets/images/project-1.jpg)

# Proyecto Diagnóstico estratégico integral para RappiPlus

## 1. Introducción

El objetivo de este proyecto es evaluar el desempeño del servicio
**RappiPlus** para apoyar **decisiones de negocio basadas en datos**.

Trabajaré con múltiples datasets del negocio:

-   **rappiplus_orders_raw.csv** → información de pedidos, precios,
    descuentos y revenue\
-   **rappiplus_catalog.csv** → costos de productos, categorías y
    proveedores\
-   **rappiplus_marketing_spend.csv** → inversión en marketing por canal
    y país\
-   **events / users / user_activity (SQL)** → comportamiento del
    usuario dentro de la plataforma\
-   **experiment_checkout_ui.csv** → resultados de un experimento A/B en
    el checkout

El análisis consistirá en los siguientes pasos:

1.  Evaluar la fiabilidad en los datos (limpieza y preparación de datos)
2.  Analizar rentabilidad del negocio (revenue, costos y profit)
3.  Observar la curva de conversion (funnel de conversión)
4.  Evaluar si los usuarios reinciden en una compra (retención por cohortes)
5.  Validar si los cambios en el tratamiento generan impacto (test estadístico)
6.  Comunicar los resultados (dashboard en Tableau)

## Paso 1: Cargar e inspeccionar los datos

### 1.1 importar librerías
``` python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

### 1.2 cargar archivos
``` python
orders = pd.read_csv('https://practicum-content.s3.amazonaws.com/datasets/rappiplus_orders_raw.csv')
catalog = pd.read_csv('https://practicum-content.s3.amazonaws.com/datasets/rappiplus_catalog.csv')
marketing = pd.read_csv('https://practicum-content.s3.amazonaws.com/datasets/rappiplus_marketing_spend.csv')
```

``` python
# explorar datasets
orders.head()
```

``` python
#Conocer renglones y columnas
orders.shape
```

``` python
#informacion general, vemos valores nulos y mal formato
orders.info()
```

``` python
#Deep dive en los valores nulos
orders.isna().sum()
```

``` python
#Que porcion de datos ocupa
orders.isna().mean()
```

``` python
#Hay algo que nos ayude a llenar los valores nulos de los paises?
orders[orders['pais'].isna()]
```

``` python
#Hay algo que nos ayude a llenar los categorias nulas?
orders[orders['categoria_producto'].isna()]
```

``` python
#id_pedido es la unica que no puede tener duplicados, revisemos
orders['id_pedido'].duplicated().sum()
```

``` python
#Necesito ver el comportamiento de los ids duplicados
orders[orders['id_pedido'].duplicated(keep=False)]\
    .sort_values(by='id_pedido')
```

``` python
#Vista general de catalogo, no hay nulos, duplicados ni imposibles
catalog.head(7)
```

``` python
#Tamaño
catalog.shape
```


``` python
#Vista general Marketing
marketing.head()
```

``` python
#Tamaño
marketing.shape
```

``` python
#Vemos valores nulos
marketing.info()
```


``` python
#Directamente podemos ver que porcentaje ocupa
marketing.isna().sum()
```


``` python
#Hay algo que podemos hacer para rellenar los canales nulos?
marketing[marketing['canal'].isna()]
```


``` python
#En el caso de Marketing, no hay valores unicos forzosos asi que hasta aqui 
#llega la exploracion
```

### Revisión y calidad de datos

**🎯 Objetivo:** Detectar y corregir problemas en los datos que puedan
afectar el análisis de revenue, costos y rentabilidad.

Se revisan los 3 datasets

-   Validar y convertir fechas al formato correcto\
-   Revisar variables numéricas (sin negativos o ceros inválidos)\
-   Verificar consistencia de montos\
-   Eliminar duplicados\
-   Revisar variables categóricas

------------------------------------------------------------------------

``` python
# tu código aquí
```

``` python
#Creare una copia por cualquier cosa
clean_orders=orders.copy()
```

``` python
#En la vista de paises nulos encontré que puedo asumir que el usuario hizo 
#la compra desde el pais mas frecuente donde ha estado, así que puedo rellenar
#los paises faltantes bajo esa premisa
def moda(series):
    moda = series.mode()
    return moda.iloc[0] if not moda.empty else np.nan
```

``` python
#Aplico la moda para que sea la que más se repite y reviso cuantos quedan
clean_orders['pais'] = clean_orders['pais']\
    .fillna(clean_orders\
        .groupby('id_usuario')['pais']\
        .transform(moda))
print(orders['pais'].isna().sum(),' vs ',clean_orders['pais'].isna().sum())
```


``` python
#Descubri que puedo llenar las categorias nulas a partir del nombre del producto
#y viceversa, siempre que haya uno de los dos, asi que lo mapeare en un 
#diccionario y reviso que concuerde con el csv
mapa_cat=clean_orders\
    .dropna(subset=['nombre_producto','categoria_producto'])\
    .set_index('nombre_producto')['categoria_producto']\
    .to_dict()
mapa_cat
```

``` python
#Aplico el cambio y reviso si surtio efecto
clean_orders['categoria_producto']=clean_orders['categoria_producto']\
    .fillna(clean_orders['nombre_producto']\
        .map(mapa_cat))
print(orders['categoria_producto'].isna().sum(),\
      ' vs ',\
      clean_orders['categoria_producto'].isna().sum())
print(orders['nombre_producto'].isna().sum(),\
      ' vs ',\
      clean_orders['nombre_producto'].isna().sum())
```

``` python
#Reviso si hay algo mas que pueda hacer
clean_orders[clean_orders['categoria_producto'].isna()].head(30)
```

``` python
#No puedo hacer mas por las categorias, reviso si puedo deducir el precio unitario
#a partir del monto total y la cantidad
clean_orders[clean_orders['precio_unitario'].isna()].head(50)
```

``` python
#No puedo resolver nada más de los valores nulos, asi que quiero ver si son tan 
#pocos que pueda presindir de ellos
clean_orders.isna().sum().sum()/clean_orders.shape[0]
```

``` python
#Son menos del 1%, así que si puedo deshacerme de ellos
clean_orders=clean_orders.dropna().reset_index(drop=True)
clean_orders.isna().sum()
```

``` python
#Ya no hay nulos, veamos que tanto quedo del dataset
clean_orders.shape[0]/orders.shape[0]
```

``` python
#Dropeo los duplicados ya que no aportan nada y reviso que este correcto
clean_orders=clean_orders\
    .drop_duplicates(subset='id_pedido',keep='first')\
    .reset_index(drop=True)
clean_orders['id_pedido'].duplicated().sum()
```

``` python
#Vamos a revisar la distribucion de las columnas categoricas por si falta algo
categoric_cols=['pais','dispositivo','fuente_referencia','nombre_producto','categoria_producto']
for i in categoric_cols:
    print(i)
    print(clean_orders[i].value_counts())
    print()
```

``` python
#Las categorias de paises estan incorrectas, corregimos y revisamos
clean_orders['pais']=clean_orders['pais']\
    .str.replace('mexico','Mexico')

clean_orders['pais']=clean_orders['pais']\
    .str.replace('colombia','Colombia')

clean_orders['pais']=clean_orders['pais']\
    .str.replace('argentina','Argentina')

clean_orders['pais'].value_counts()
```

``` python
#Hacemos el mismo analisis para las columnas numericas
num_cols=['cantidad','precio_unitario','monto_descuento','monto_total']
for i in num_cols:
    print(i)
    print(clean_orders[i].describe())
    print()
```

``` python
#Encontre valores imposibles, revisare su naturaleza en la columna 'cantidad'
clean_orders[clean_orders['cantidad']<=0]
```


``` python
#y en la columna 'monto_total'
clean_orders[clean_orders['monto_total']<=0]
```

``` python
#Encontre que la aritmetrica esta correcta, parece que la cantidad estaba negativa
#Y al multiplicarla el monto total quedo negativo por error, se puede corregir con
#.abs()
clean_orders['cantidad']=clean_orders['cantidad'].abs()
clean_orders['monto_total']=clean_orders['monto_total'].abs()
clean_orders['cantidad'].describe()
```

``` python
#La cantidad de 20000 tambien me parece un error de tipo outlier, 
#necesito revisar más a detalle
clean_orders[clean_orders['cantidad']>2]
```

``` python
clean_orders[clean_orders['cantidad']<=2]
```

``` python
#Veamos la comparacion
print(clean_orders[clean_orders['cantidad']<=2]['cantidad'].count(),\
      " vs ", \
      clean_orders[clean_orders['cantidad']>2]['cantidad'].count())
```

``` python
#Son 24871 entradas donde las cantidades varian de 1 a 2 
#y 10 donde solo salta de 10K a 20K
#No se ve un patron de error pero es un outlier de manual. 
#Lo ideal sería ir con cliente para ver como tratarlo, sin embargo,
#por el comportamiento tan inusual, particular y la cantidad tan pequeña de
#entradas, tambien sería plausible asumir que cada 10000 es 1 
#y cada 20000 es 2
clean_orders.loc[clean_orders['cantidad'].isin([10000, 20000]), 'cantidad'] = clean_orders['cantidad'] / 10000
clean_orders[clean_orders['cantidad']>2]['cantidad'].count()
```

``` python
#Este deberia ser el total
print(clean_orders[clean_orders['cantidad']<=2]['cantidad'].count(),' de ', clean_orders.shape[0])
```

``` python
#Solo faltaría arreglar el monto total
clean_orders['monto_total']=(clean_orders['cantidad']*clean_orders['precio_unitario'])-clean_orders['monto_descuento']
```

``` python
#Parece que ya quedaron bien, aunque son valores poco reales, no son imposibles
clean_orders['monto_total'].describe()
```

``` python
#En la exploracion notamos que la fecha esta como str y la cantidad como float,
#Los cambiare a Datetime e int respectivamente.
#El formato %Y-%m-%d' lo uso por los cambios de horarios entre paises
#y solo necesito la fecha, no es necesario el tiempo
clean_orders['fecha_hora_pedido']=\
    pd.to_datetime(clean_orders['fecha_hora_pedido'],format='%Y-%m-%d')
clean_orders['cantidad']=clean_orders['cantidad'].astype('int64')
```

``` python
#Reviso que haya quedado correcto
clean_orders.dtypes
```

``` python
#Y que no haya fechas imposibles
clean_orders['fecha_hora_pedido'].describe()
```

``` python
#Creare una copia de marketing tambien
clean_marketing=marketing.copy()
```

``` python
#En la exploracion de nulos encontre que se puede llenar los canales faltantes
#a través de id_campaña
clean_marketing['canal']=clean_marketing['canal']\
    .fillna(clean_marketing['id_campaña'].str.rsplit('_',n=1).str[0])
clean_marketing.isna().mean()
```

``` python
#Aqui se comprueba que la separacion fue correcta
print(clean_marketing.iloc[98:105])
```

``` python
#Vamos a revisar la distribucion de las columnas categoricas por si falta algo
categoric_cols=['pais','id_campaña','canal']
for i in categoric_cols:
    print(i)
    print(clean_marketing[i].value_counts())
    print()
```

``` python
#Revisamos si el gasto de marketing tiene numeros imposibles o Outliers
clean_marketing['gasto'].describe()
```

``` python
#Finalmente nos encargamos de formatear la fecha
clean_marketing['fecha']=\
    pd.to_datetime(clean_marketing['fecha'],format='%Y-%m-%d')
clean_marketing.dtypes
```

``` python
#Y reviso si hay alguna fecha imposible
clean_marketing['fecha'].describe()
```

``` python
#Con esto finalizaria la limpieza
print("Se perdio el ",\
      (1-(clean_orders.shape[0]/orders.shape[0]))*100.0,\
      "% de info durante la limpieza de las ordenes")
print("Se perdio el ",\
      (1-(clean_marketing.shape[0]/marketing.shape[0]))*100.0,\
      "% de info durante la limpieza de las campañas de Marketing")
```

------------------------------------------------------------------------

**📦 Exportación**: Una vez finalizada la limpieza, se exportan los
datasets para utilizarlos en la última etapa del proyecto.

``` python
# exportar datasets
clean_orders.to_csv('orders_clean.csv', index=False)
catalog.to_csv('catalog_clean.csv', index=False)
clean_marketing.to_csv('marketing_clean.csv', index=False)
```

## 🔹 Paso 2: Analizar si el negocio es rentable {#-paso-2-analizar-si-el-negocio-es-rentable}

### 2.1 Cálculo de KPIs principales {#21-cálculo-de-kpis-principales}

**🎯 Objetivo:** Calcular los indicadores clave del negocio para evaluar
ingresos, costos y rentabilidad.

Se usan los 3 datasets (`orders`, `catalog`, `marketing_spend`):

**📊 Parte 1: Rentabilidad del negocio**

-   ¿Cuál es el ingreso total (revenue)?
-   ¿Cuál es el costo total?
-   ¿Cuánto se ha invertido en marketing?
-   ¿El negocio es rentable? (calcular profit)

------------------------------------------------------------------------

**📈 Parte 2: Comportamiento de ventas**

-   ¿Cuál es el ticket promedio por orden?
-   ¿Cuál es la cantidad promedio de productos por orden?
-   ¿Cuál es el producto más vendido?
-   ¿Cuánto se ha gastado en marketing por canal?

``` python
# tu código aquí
```

``` python
#Empezamos a sacar los KPIs con el revenue
revenue=clean_orders['monto_total'].sum()
revenue
```

``` python
#Para sacar el costo de los productos hay que hace left join con el catalogo
#y reviso que este correcto
catalog['categoria_producto']=catalog['categoria_producto']\
    .str.replace('Electrónica','Electronica')
merged_orders=pd.merge(clean_orders,\
                       catalog,\
                       on=['nombre_producto','categoria_producto'],\
                       how='left')
merged_orders.head()
```

``` python
#Revisaré que no haya nada raro
print(merged_orders.isna().sum(),'\n')
print(merged_orders.shape[0], ' vs ',clean_orders.shape[0],'\n')
print(merged_orders.duplicated().sum(),'\n')
categoric_cols=['pais','dispositivo','fuente_referencia','nombre_producto','categoria_producto']
for i in categoric_cols:
    print(i)
    print(merged_orders[i].value_counts())
    print()
```

``` python
#Con el merged listo puedo sacar el costo multiplicando la cantidad por el costo unitario
merged_orders['costo_total']=\
    merged_orders['cantidad'] * merged_orders['costo_unitario']
merged_orders.head()
```

``` python
#Obtenemos el costo total por los productos
total_cost=merged_orders['costo_total'].sum()
total_cost
```

``` python
#Obtenemos el costo total de Marketing
total_marketing=clean_marketing['gasto'].sum()
total_marketing
```

``` python
#Teoricamente, un negocio es rentable si hay ganancia considerando el costo 
#de los productos y el costo de marketing
total_profit=revenue-total_cost-total_marketing
total_profit
```

``` python
#Parte 2
#Para el ticket promedio lo haremos con el monto total promedio
mean_ticket=merged_orders['monto_total'].mean()
mean_ticket
```

``` python
#La cantidad promedio por orden
mean_qty=merged_orders['cantidad'].mean()
mean_qty
```

``` python
#Para el producto mas vendido
merged_orders\
    .groupby('nombre_producto')['monto_total']\
    .sum().reset_index()\
    .sort_values(by='monto_total',ascending=False)
```

``` python
#El gasto por canal
clean_marketing\
    .groupby('canal')['gasto']\
    .sum().reset_index()\
    .sort_values(by='gasto',ascending=False)
```

## 🔹 Paso 3: Entender dónde se pierden los usuarios (funnel de conversión) {#-paso-3-entender-dónde-se-pierden-los-usuarios-funnel-de-conversión}

**🎯 Objetivo:** Analizar el comportamiento de los usuarios para
identificar en qué etapa del proceso se pierden.

⚙️**Conexión a la base de datos**:\
Se ejecuta la línea de configuración para conectar con la base de datos
y aplicar consultas SQL en la tabla **events**.

------------------------------------------------------------------------

**📊 Parte 1: Construcción del funnel**

-   ¿Cuántos usuarios llegan a cada etapa del funnel?\
-   Se calcula el número de usuarios únicos por `nombre_evento`\
-   Se ordenan los eventos según el flujo del usuario

------------------------------------------------------------------------

**📉 Parte 2: Análisis de conversión**

-   Se calcula la tasa de conversión entre cada paso del funnel\

-   Se identifica en qué etapa se pierde la mayor cantidad de usuarios\

-   ## ¿Cuál es la tasa de conversión final?

``` python
import pandas as pd
from sqlalchemy import create_engine

# ======================
# Conexión (NO modificar)
# ======================
db_config = {
    'user': 'practicum_student',
    'pwd': 'QnmDH8Sc2TQLvy2G3Vvh7',
    'host': 'yp-trainers-practicum.cluster-czs0gxyx2d8w.us-east-1.rds.amazonaws.com',
    'port': 5432,
    'db': 'data-analyst-production-db-en'
}

connection_string = 'postgresql://{}:{}@{}:{}/{}'.format(
    db_config['user'],
    db_config['pwd'],
    db_config['host'],
    db_config['port'],
    db_config['db']
)

engine = create_engine(connection_string, connect_args={'sslmode':'require'})
```

``` python
# Explorar tabla events
# =========================
query_events = '''
SELECT *
FROM events;
'''
events = pd.read_sql(query_events, con=engine)
events.head()
```

``` python
# PARTE 0: Revision de datos
# ======================
#Reviso que si hay duplicados
query_totals = '''
SELECT 
    id_sesion,
    nombre_evento,
    timestamp_evento,
    COUNT(*) AS numero_registros
FROM events
GROUP BY id_sesion,
    nombre_evento,
    timestamp_evento
ORDER BY numero_registros DESC
'''

events_duplicates = pd.read_sql(query_totals, con=engine)
events_duplicates
```

``` python
#Reviso si hay faltantes en first_visit
query_totals = '''
SELECT  COUNT(DISTINCT id_usuario)
FROM events
WHERE nombre_evento = 'select_item'
AND id_usuario NOT IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'first_visit')
'''

event_missing_first_to_select = pd.read_sql(query_totals, con=engine)
event_missing_first_to_select
```


``` python
#Reviso si hay faltantes en select_item
query_totals = '''
SELECT  COUNT(DISTINCT id_usuario)
FROM events
WHERE nombre_evento = 'add_to_cart'
AND id_usuario NOT IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'select_item')
'''

event_missing_select_to_cart = pd.read_sql(query_totals, con=engine)
event_missing_select_to_cart
```

``` python
#Reviso si hay faltantes en add_to_cart
query_totals = '''
SELECT  COUNT(DISTINCT id_usuario)
FROM events
WHERE nombre_evento = 'begin_checkout'
AND id_usuario NOT IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'add_to_cart')
'''

event_missing_cart_to_checkout = pd.read_sql(query_totals, con=engine)
event_missing_cart_to_checkout
```

``` python
#Reviso si hay faltantes en begin_checkout
query_totals = '''
SELECT  COUNT(DISTINCT id_usuario)
FROM events
WHERE nombre_evento = 'add_payment_info'
AND id_usuario NOT IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'begin_checkout')
'''

event_missing_checkout_to_addpayinfo = pd.read_sql(query_totals, con=engine)
event_missing_checkout_to_addpayinfo
```

``` python
#Reviso si hay faltantes en add_payment_info
query_totals = '''
SELECT  COUNT(DISTINCT id_usuario)
FROM events
WHERE nombre_evento = 'purchase'
AND id_usuario NOT IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'add_payment_info')
'''

event_missing_checkout_to_addpayinfo = pd.read_sql(query_totals, con=engine)
event_missing_checkout_to_addpayinfo
```

``` python
# PARTE 1: Totales del funnel
# ======================
#Encontre faltantes en todas las fases, por lo que la idea es considerar 
#solo las que se va dando seguimiento. Que en este caso son las que aparecen 
#en first_visit

query_totals = '''
SELECT
    nombre_evento,
    COUNT(DISTINCT(id_usuario))
FROM events
WHERE id_usuario IN (
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento = 'first_visit')
GROUP BY nombre_evento
ORDER BY
    CASE nombre_evento 
        WHEN 'first_visit' THEN 1 
        WHEN 'select_item' THEN 2 
        WHEN 'add_to_cart' THEN 3 
        WHEN 'begin_checkout' THEN 4
        WHEN 'add_payment_info' THEN 5
        ELSE 6
    END;
'''

totals = pd.read_sql(query_totals, con=engine)
totals
```
``` python
#Podemos ver que la curva es muy ligera, parece que hay un error de 
#medicion entre select_item y add_to_chart. Pero creo puede justificarse
#ya que hay paginas que directamente pueden agregar al carrito.
#Aun así valdría la pena sugerir que la herramienta registre cada add_to_chart
#directo como un select_item.
```

``` python
# PARTE 2: Conversiones
# ======================

query_conversion = '''
WITH con_first_visit AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='first_visit'
    ),
    con_select_item AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='select_item'
        AND id_usuario IN (
            SELECT DISTINCT id_usuario
            FROM events
            WHERE nombre_evento = 'first_visit')
    ),
    con_add_to_cart AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='add_to_cart'
        AND id_usuario IN (
            SELECT DISTINCT id_usuario
            FROM events
            WHERE nombre_evento = 'first_visit')
    ),
    con_begin_checkout AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='begin_checkout'
        AND id_usuario IN (
            SELECT DISTINCT id_usuario
            FROM events
            WHERE nombre_evento = 'first_visit')
    ),
    con_add_payment_info AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='add_payment_info'
        AND id_usuario IN (
            SELECT DISTINCT id_usuario
            FROM events
            WHERE nombre_evento = 'first_visit')
    ),
    con_purchase AS(
    SELECT DISTINCT id_usuario
    FROM events
    WHERE nombre_evento='purchase'
        AND id_usuario IN (
            SELECT DISTINCT id_usuario
            FROM events
            WHERE nombre_evento = 'first_visit')
    )


SELECT
    (SELECT COUNT(*) FROM con_first_visit) AS first_visit,
    (SELECT COUNT(*) FROM con_select_item) AS select_item,
    (SELECT COUNT(*) FROM con_add_to_cart) AS add_to_cart,
    (SELECT COUNT(*) FROM con_begin_checkout) AS begin_checkout,
    (SELECT COUNT(*) FROM con_add_payment_info) AS add_payment_info,
    (SELECT COUNT(*) FROM con_purchase) AS purchase,

    ((SELECT COUNT(*) FROM con_first_visit) - (SELECT COUNT(*) FROM con_select_item))*100.0/NULLIF((SELECT COUNT(*) FROM con_first_visit),0) AS dropoff_after_first_visit_pct,
    ((SELECT COUNT(*) FROM con_select_item) - (SELECT COUNT(*) FROM con_add_to_cart))*100.0/NULLIF((SELECT COUNT(*) FROM con_select_item),0) AS dropoff_after_select_item_pct,
    ((SELECT COUNT(*) FROM con_add_to_cart) - (SELECT COUNT(*) FROM con_begin_checkout))*100.0/NULLIF((SELECT COUNT(*) FROM con_add_to_cart),0) AS dropoff_after_add_to_cart_pct,
    ((SELECT COUNT(*) FROM con_begin_checkout) - (SELECT COUNT(*) FROM con_add_payment_info))*100.0/NULLIF((SELECT COUNT(*) FROM con_begin_checkout),0) AS dropoff_after_begin_checkout_pct,
    ((SELECT COUNT(*) FROM con_add_payment_info) - (SELECT COUNT(*) FROM con_purchase))*100.0/NULLIF((SELECT COUNT(*) FROM con_add_payment_info),0) AS dropoff_after_payment_info_pct
    
'''

conversion = pd.read_sql(query_conversion, con=engine)
conversion
```

``` python
#Por lo mismo que comente anteriormente, se muestra evidente que vale la pena dar
#esa sugerencia al cliente
```

## 🔹 Paso 4: Evaluar si los usuarios regresan (retención por cohortes) {#-paso-4-evaluar-si-los-usuarios-regresan-retención-por-cohortes}

**🎯 Objetivo:** Analizar la retención de usuarios para entender si
regresan después de registrarse.

**Tablas**

-   `users`
-   `user_activity`

------------------------------------------------------------------------

1.  Se identifica la cohorte de cada usuario según el **mes de
    registro**.

2.  Se calcula la retención semanal: cuántos usuarios **se mantienen
    activos** en cada semana desde su registro.

    -   `retenido_w1`: usuarios activos en la semana 1\
    -   `retenido_w2`: usuarios activos en la semana 2\
    -   `retenido_w3`: usuarios activos en la semana 3

3.  Se calcula el porcentaje de retención para cada semana, dividiendo
    los usuarios retenidos entre los clientes iniciales de la cohorte:

    -   `semana_1`: porcentaje de usuarios retenidos en la semana 1\
    -   `semana_2`: porcentaje de usuarios retenidos en la semana 2\
    -   `semana_3`: porcentaje de usuarios retenidos en la semana 3

Se revisa que la columna de fecha esté en formato correcto (`DATE`).\
Se realiza la conversión usando: `CAST(fecha_registro AS DATE)`

``` python
# Explorar tabla users
# =========================
query_users = '''
SELECT *
FROM users;
'''
users = pd.read_sql(query_users, con=engine)
users.head(3)
```

``` python
# Revisamos el tipo de dato de fecha de users
query_users = '''
SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'users'
AND COLUMN_NAME = 'fecha_registro';
'''
users_datetype = pd.read_sql(query_users, con=engine)
users_datetype.head(3)
   
```

``` python
#Reviso que si hay duplicados, solo id_usuario debe ser unico
query_totals = '''
SELECT 
    id_usuario,
    COUNT(*) AS numero_registros
FROM users
GROUP BY id_usuario
ORDER BY numero_registros DESC
'''

users_duplicates = pd.read_sql(query_totals, con=engine)
users_duplicates
```

``` python
# Explorar tabla user_activity
# =========================
query_user_activity = '''
SELECT *
FROM user_activity;
'''
user_activity = pd.read_sql(query_user_activity, con=engine)
user_activity.head(3)
```

``` python
# Revisamos el tipo de dato de fecha de user_activity
query_users = '''
SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'user_activity'
AND COLUMN_NAME = 'fecha_actividad';
'''
user_activity_datetype = pd.read_sql(query_users, con=engine)
user_activity_datetype.head(3)
```

``` python
# Retención por cohortes
# ======================

query_cohort_retention_final = '''

WITH cohortes AS (
    SELECT
        activity.id_usuario,   
        DATE_TRUNC('week',MIN(CAST(usr.fecha_registro AS DATE))) AS cohorte_w,
        activity.dias_despues_registro,
        activity.activo
    FROM user_activity AS activity
    LEFT JOIN users AS usr
        ON activity.id_usuario=usr.id_usuario
    GROUP BY activity.id_usuario, 
        activity.dias_despues_registro, 
        activity.activo),
retencion AS(
    SELECT
        cohorte_w,
        COUNT(*) AS clientes_iniciales,
        COUNT(CASE WHEN dias_despues_registro::numeric >= 7 AND activo::numeric = 1 THEN 1 END) AS retained_w1,
        COUNT(CASE WHEN dias_despues_registro::numeric >= 14 AND activo::numeric = 1 THEN 1 END) AS retained_w2,
        COUNT(CASE WHEN dias_despues_registro::numeric >= 21 AND activo::numeric = 1 THEN 1 END) AS retained_w3
    
    FROM cohortes
    GROUP BY cohorte_w)

SELECT
    TO_CHAR(cohorte_w, '"W"IW (DD-Mon)') as cohorte,
    clientes_iniciales,
    ROUND(retained_w1::numeric / clientes_iniciales, 2) AS semana_1,
    ROUND(retained_w2::numeric / clientes_iniciales, 2) AS semana_2,
    ROUND(retained_w3::numeric / clientes_iniciales, 2) AS semana_3

FROM retencion
ORDER BY cohorte_w

'''

# Ejecutar la consulta
cohorte_final = pd.read_sql(query_cohort_retention_final, con=engine)
cohorte_final
```

## 🔹 Paso 5: Validar si los cambios generan impacto (test estadístico) {#-paso-5-validar-si-los-cambios-generan-impacto-test-estadístico}

🎯 **Objetivo:** Evaluar si la modificación en la UI del checkout
impacta la **tasa de conversión de compra**.

------------------------------------------------------------------------

1.  **Analizar el dataset** `experiment_checkout_ui.csv` para
    identificar la métrica principal **conversion**.
    -   La métrica **conversion** es 1 si el usuario completó la compra,
        0 si no.\
2.  **Plantear la hipótesis estadística**\
3.  **Aplicar el test estadístico adecuado**
4.  **Interpretar el resultado**


------------------------------------------------------------------------

Hipótesis estadística

-   **H₀ (Hipótesis nula):** El tratamiento no afecta la tasa de
    conversion
-   **H₁ (Hipótesis alternativa):** El tratamiento influye en la tasa de
    conversion

**Test estadístico:** Prueba Z, resultado de z: 0.8132782986429474\
**Nivel de significancia alpha:** 0.41605851639119995

``` python
# tu código aquí
```

``` python
#Extraemos el csv y damos una revisada general
experiment = \
    pd.read_csv('https://practicum-content.s3.amazonaws.com/datasets/experiment_checkout_ui.csv')
experiment.head()
```

``` python
#Vemos el tamaño de la muestra
experiment.shape
```

``` python
#Revisamos si no hay valores nulos
experiment.info()
```

``` python
#Buscamos duplicados, en este caso solo id_usuario debe ser unico
experiment['id_usuario'].duplicated().sum()
```

``` python
#Solo necesitamos formatear la fecha
experiment['timestamp']=\
    pd.to_datetime(experiment['timestamp'],format='%Y-%m-%d')
experiment.dtypes
```

``` python
#Revisamos las categorias
categoric_cols=['variante','dispositivo','pais']
for i in categoric_cols:
    print(i)
    print(experiment[i].value_counts())
    print()
```

``` python
#Revisamos valores numericos
num_cols=['convirtio','duracion_sesion']
for i in num_cols:
    print(i)
    print(experiment[i].describe())
    print()
```

``` python
#Al ser taza se conversion y para probar efectividad del 
#tratamiento usamos Prueba Z
from statsmodels.stats.proportion import proportions_ztest
conversion=experiment.groupby('variante')['convirtio'].sum()
conversion
```

``` python
#Ya con las conversiones, obtenemos los totales
total=experiment.groupby('variante')['convirtio'].count()
total
```

``` python
#Transformamos en lista para la Prueba Z
conteo=[conversion['tratamiento'],conversion['control']]
observaciones=[total['tratamiento'],total['control']]
```

``` python
#Aplicamos Prueba Z y revisamos los valores
z,p_value=proportions_ztest(conteo , observaciones)
print("Z: ",z)
print("P Value: ",p_value)
```


``` python
#Definimos alpha y dejamos que el mismo codigo responda 
#si se rechaza o no la hipotesis nula
alpha = 0.05  # umbral de significancia
if p_value < alpha:
    print("Rechazamos la hipótesis nula: hay evidencia de una diferencia.")
else:
    print("No rechazamos la hipótesis nula: no hay evidencia suficiente de una diferencia.")
```

``` python
#Finalmente obtenemos las tasas de conversion y comparamos
tasa_tratamiento = conteo[0]/observaciones[0]
tasa_control = conteo[1]/observaciones[1]

print("\nTasa de conversion en tratamiento: ",tasa_tratamiento)
print("Tasa de conversion en control: ",tasa_control)
print("Diferencia: ",tasa_tratamiento-tasa_control)
```

## 🔹 Paso 6: Comunicar los resultados (Dashboard en BI) {#-paso-6-comunicar-los-resultados-dashboard-en-bi}

🎯 **Objetivo**:\
Crear un dashboard que muestre de manera clara y visual los resultados
del análisis de ventas, costos, marketing y conversión.

Se usarán los CSVs limpios del Paso 1:

-   `orders_clean.csv`\
-   `catalog_clean.csv`\
-   `marketing_clean.csv`

------------------------------------------------------------------------

1️⃣ Preparación de los datos

1.  Cargar los CSVs en Power BI o Tableau.
2.  Revisar relaciones:
    -   `orders.nombre_producto` → `catalog.nombre_producto`
    -   `orders.fecha_pedido` → tabla de fechas (crear calendario para
        análisis temporal)
    -   `orders.fecha_pedido` → `dim_fecha.date`
3.  Crear columnas calculadas necesarias
4.  Crear tabla de fechas para poder calcular comparaciones YTD, YoY o
    períodos anteriores (`Previous Year`, `Previous Month`).

------------------------------------------------------------------------

2️⃣ Dashboard 1: Overview Ejecutivo **KPIs principales a mostrar:**

-   Revenue total
-   Profit total
-   Gasto total en marketing
-   Ticket promedio
-   Cantidad promedio de productos por orden

**Visualizaciones sugeridas:**

-   Tarjetas KPI para revenue, profit y gasto marketing
-   Gráfico de líneas: evolución mensual de revenue o profit
-   Gráfico de líneas YTD
-   Gráfico de barras: revenue y profit por producto o categoría

------------------------------------------------------------------------

3️⃣ Dashboard 2: Detalle / Drill-through\
**Objetivo:** Permitir explorar los datos desde el KPI general hasta
cada orden o producto.

**Visualizaciones sugeridas:**

-   Tabla detallada de órdenes con:
    -   producto, cantidad, revenue, cost, profit
    -   color condicional (profit negativo en rojo, positivo en verde)
-   Gráfico de barras por producto con medida `cantidad vendida`
-   Drill-through: seleccionar un producto y ver todos los pedidos
    relacionados
-   Filtros por fecha, categoría de producto, etc

## 🚀 Entrega Final {#-entrega-final}

Comparte el acceso a tu Dashboard para revisión.\
Puedes entregar el Dashboard utilizando **Power BI o Tableau**.

Incluye **uno de los siguientes**:

-   🔗 Link público del dashboard publicado en **Power BI Service o
    Tableau Public / Tableau Cloud**
-   🔗 Link de **Google Drive o OneDrive** con el archivo del proyecto
    (`.pbix`) y los 3 csvs limpios.

### 📎 Enlace del Dashboard {#-enlace-del-dashboard}

``` python
# (Pega aquí tu link)
#https://public.tableau.com/views/DA_FinalProject_17823503737380/OverviewEjecutivo?:language=en-GB&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link
```
