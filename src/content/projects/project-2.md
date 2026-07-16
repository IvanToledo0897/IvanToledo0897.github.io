---
title: 'Diagnóstico estratégico integral para RappiPlus'
description: Proyecto para RappiPlus, preparando datos para corroborar la estrategia de Rappiplus en sus ganacias y retención de usuarios.

publishDate: '14 Julio 2026'
seo:
  image:
    src: '../../assets/images/project-2.png'
    alt: Project preview
---

![Project preview](../../assets/images/project-2.png)

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

------------------------------------------------------------------------

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

### 1.3 Explorar datasets
``` python
orders.head()
```

#### Conocer renglones y columnas
``` python
orders.shape
```

#### Informacion general: vemos valores nulos y mal formato
``` python
orders.info()
```

Deep dive en los valores nulos
``` python
orders.isna().sum()
```

¿Que porcion de datos ocupa?
``` python
orders.isna().mean()
```

¿Hay algo que nos ayude a llenar los valores nulos de los paises?
``` python
orders[orders['pais'].isna()]
```

¿Hay algo que nos ayude a llenar los categorias nulas?
``` python
orders[orders['categoria_producto'].isna()]
```

`id_pedido` es la unica que no puede tener duplicados, revisemos.
``` python
orders['id_pedido'].duplicated().sum()
```

Necesito ver el comportamiento de los ids duplicados.
``` python
orders[orders['id_pedido'].duplicated(keep=False)]\
    .sort_values(by='id_pedido')
```

Vista general de catalogo, no hay nulos, duplicados ni imposibles. Al ser tan pequeño puedo verlo todo.
``` python
catalog.head(7)
```

Tamaño... no hay mucho que ver entonces. Pasemos al siguiente csv.
``` python
catalog.shape
```

Vista general Marketing
``` python
marketing.head()
```

Tamaño
``` python
marketing.shape
```

Vemos valores nulos
``` python
marketing.info()
```

Directamente podemos ver que porcentaje ocupa
``` python
marketing.isna().sum()
```

¿Hay algo que podemos hacer para rellenar los canales nulos?
``` python
marketing[marketing['canal'].isna()]
```

En el caso de Marketing, no hay valores unicos forzosos asi que hasta aqui llega la exploracion


### 1.4 Revisión y calidad de datos

De los 3 datasets, debo cerciorarme:
-   Fechas al formato correcto\
-   Revisar valores imposibles en variables numéricas (sin negativos o ceros inválidos)\
-   Consistencia de montos\
-   Eliminar duplicados\
-   Variables categóricas

------------------------------------------------------------------------

Creare una copia por cualquier cosa
``` python
clean_orders=orders.copy()
```

En la vista de paises nulos encontré que puedo asumir que el usuario hizo la compra desde el pais mas frecuente donde ha estado, así que puedo rellenar los paises faltantes bajo esa premisa:
``` python
def moda(series):
    moda = series.mode()
    return moda.iloc[0] if not moda.empty else np.nan
```

Aplico la moda para que sea la que más se repite y reviso cuantos quedan
``` python
clean_orders['pais'] = clean_orders['pais']\
    .fillna(clean_orders\
        .groupby('id_usuario')['pais']\
        .transform(moda))
print(orders['pais'].isna().sum(),' vs ',clean_orders['pais'].isna().sum())
```

Descubri que puedo llenar las categorias nulas a partir del nombre del producto y viceversa, siempre que haya uno de los dos, asi que lo mapeare en un diccionario y reviso que concuerde con el csv
``` python
mapa_cat=clean_orders\
    .dropna(subset=['nombre_producto','categoria_producto'])\
    .set_index('nombre_producto')['categoria_producto']\
    .to_dict()
mapa_cat
```

Aplico el cambio y reviso si surtio efecto
``` python
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

Reviso si hay algo mas que pueda hacer
``` python
clean_orders[clean_orders['categoria_producto'].isna()].head(30)
```

No puedo hacer mas por las categorias, reviso si puedo deducir el precio unitario a partir del monto total y la cantidad
``` python
clean_orders[clean_orders['precio_unitario'].isna()].head(50)
```

No puedo resolver nada más de los valores nulos, asi que quiero ver si son tan pocos como para que pueda presindir de ellos
``` python
clean_orders.isna().sum().sum()/clean_orders.shape[0]
```

Son menos del 1%, así que si puedo deshacerme de ellos
``` python
clean_orders=clean_orders.dropna().reset_index(drop=True)
clean_orders.isna().sum()
```

Ya no hay nulos, veamos que tanto quedo del dataset
``` python
clean_orders.shape[0]/orders.shape[0]
```

Dropeo los duplicados ya que no aportan nada y reviso que este correcto
``` python
clean_orders=clean_orders\
    .drop_duplicates(subset='id_pedido',keep='first')\
    .reset_index(drop=True)
clean_orders['id_pedido'].duplicated().sum()
```

Vamos a revisar la distribucion de las columnas categoricas por si falta algo
``` python
categoric_cols=['pais','dispositivo','fuente_referencia','nombre_producto','categoria_producto']
for i in categoric_cols:
    print(i)
    print(clean_orders[i].value_counts())
    print()
```

Las categorias de paises estan incorrectas, corregimos y revisamos
``` python
clean_orders['pais']=clean_orders['pais']\
    .str.replace('mexico','Mexico')

clean_orders['pais']=clean_orders['pais']\
    .str.replace('colombia','Colombia')

clean_orders['pais']=clean_orders['pais']\
    .str.replace('argentina','Argentina')

clean_orders['pais'].value_counts()
```

Hacemos el mismo analisis para las columnas numericas
``` python
num_cols=['cantidad','precio_unitario','monto_descuento','monto_total']
for i in num_cols:
    print(i)
    print(clean_orders[i].describe())
    print()
```

Encontre valores imposibles, revisare su naturaleza en la columna 'cantidad'
``` python
clean_orders[clean_orders['cantidad']<=0]
```

y en la columna 'monto_total'
``` python
clean_orders[clean_orders['monto_total']<=0]
```

#Encontre que la aritmetrica esta correcta, parece que la cantidad estaba negativa y al multiplicarla el monto total quedo negativo por error, se puede corregir con `.abs()`
``` python
clean_orders['cantidad']=clean_orders['cantidad'].abs()
clean_orders['monto_total']=clean_orders['monto_total'].abs()
clean_orders['cantidad'].describe()
```

La cantidad de 20000 tambien me parece un error de tipo outlier, necesito revisar más a detalle
``` python
clean_orders[clean_orders['cantidad']>2]
```

``` python
clean_orders[clean_orders['cantidad']<=2]
```

Veamos la comparacion
``` python
print(clean_orders[clean_orders['cantidad']<=2]['cantidad'].count(),\
      " vs ", \
      clean_orders[clean_orders['cantidad']>2]['cantidad'].count())
```

Son 24871 entradas donde las cantidades varian de 1 a 2 y 10 donde solo salta de 10K a 20K no se ve un patron de error pero es un outlier de manual. 
Lo ideal sería ir con cliente para ver como tratarlo, sin embargo, por el comportamiento tan inusual, particular y la cantidad tan pequeña de entradas, tambien sería plausible asumir que cada 10000 es 1 y cada 20000 es 2
``` python
clean_orders.loc[clean_orders['cantidad'].isin([10000, 20000]), 'cantidad'] = clean_orders['cantidad'] / 10000
clean_orders[clean_orders['cantidad']>2]['cantidad'].count()
```

Este deberia ser el total
``` python
print(clean_orders[clean_orders['cantidad']<=2]['cantidad'].count(),' de ', clean_orders.shape[0])
```

Solo faltaría arreglar el monto total
``` python
clean_orders['monto_total']=(clean_orders['cantidad']*clean_orders['precio_unitario'])-clean_orders['monto_descuento']
```

Parece que ya quedaron bien, aunque son valores poco reales, no son imposibles
``` python
clean_orders['monto_total'].describe()
```

#En la exploracion notamos que la fecha esta como str y la cantidad como float, los cambiare a Datetime e int respectivamente.
El formato %Y-%m-%d' lo uso por los cambios de horarios entre paises y solo necesito la fecha, no es necesario el tiempo
``` python
clean_orders['fecha_hora_pedido']=\
    pd.to_datetime(clean_orders['fecha_hora_pedido'],format='%Y-%m-%d')
clean_orders['cantidad']=clean_orders['cantidad'].astype('int64')
```

Reviso que haya quedado correcto
``` python
clean_orders.dtypes
```

Y que no haya fechas imposibles
``` python
clean_orders['fecha_hora_pedido'].describe()
```

Creare una copia de marketing tambien
``` python
clean_marketing=marketing.copy()
```

En la exploracion de nulos encontre que se puede llenar los canales faltantes a través de id_campaña
``` python
clean_marketing['canal']=clean_marketing['canal']\
    .fillna(clean_marketing['id_campaña'].str.rsplit('_',n=1).str[0])
clean_marketing.isna().mean()
```

Aqui se comprueba que la separacion fue correcta
``` python
print(clean_marketing.iloc[98:105])
```

Vamos a revisar la distribucion de las columnas categoricas por si falta algo
``` python
categoric_cols=['pais','id_campaña','canal']
for i in categoric_cols:
    print(i)
    print(clean_marketing[i].value_counts())
    print()
```

Revisamos si el gasto de marketing tiene numeros imposibles o Outliers
``` python
clean_marketing['gasto'].describe()
```

Finalmente nos encargamos de formatear la fecha
``` python
clean_marketing['fecha']=\
    pd.to_datetime(clean_marketing['fecha'],format='%Y-%m-%d')
clean_marketing.dtypes
```

Y reviso si hay alguna fecha imposible
``` python
Y reviso si hay alguna fecha imposible
clean_marketing['fecha'].describe()
```

Con esto finalizaria la limpieza
``` python
print("Se perdio el ",\
      (1-(clean_orders.shape[0]/orders.shape[0]))*100.0,\
      "% de info durante la limpieza de las ordenes")
print("Se perdio el ",\
      (1-(clean_marketing.shape[0]/marketing.shape[0]))*100.0,\
      "% de info durante la limpieza de las campañas de Marketing")
```

------------------------------------------------------------------------

**Exportación**: Una vez finalizada la limpieza, se exportan los
datasets para utilizarlos en la última etapa del proyecto.

``` python
clean_orders.to_csv('orders_clean.csv', index=False)
catalog.to_csv('catalog_clean.csv', index=False)
clean_marketing.to_csv('marketing_clean.csv', index=False)
```

## Paso 2: Analizar si el negocio es rentable

Usaré los 3 datasets (`orders`, `catalog`, `marketing_spend`):

**Parte 1: Rentabilidad del negocio**

-   ¿Cuál es el ingreso total (revenue)?
-   ¿Cuál es el costo total?
-   ¿Cuánto se ha invertido en marketing?
-   ¿El negocio es rentable? (calcular profit)

------------------------------------------------------------------------

**Parte 2: Comportamiento de ventas**

-   ¿Cuál es el ticket promedio por orden?
-   ¿Cuál es la cantidad promedio de productos por orden?
-   ¿Cuál es el producto más vendido?
-   ¿Cuánto se ha gastado en marketing por canal?

### 2.1 Parte 1
Empezamos a sacar los KPIs con el revenue
``` python
revenue=clean_orders['monto_total'].sum()
revenue
```

Para sacar el costo de los productos hay que hace left join con el catalogo y reviso que este correcto
``` python
catalog['categoria_producto']=catalog['categoria_producto']\
    .str.replace('Electrónica','Electronica')
merged_orders=pd.merge(clean_orders,\
                       catalog,\
                       on=['nombre_producto','categoria_producto'],\
                       how='left')
merged_orders.head()
```

Revisaré que no haya nada raro
``` python
print(merged_orders.isna().sum(),'\n')
print(merged_orders.shape[0], ' vs ',clean_orders.shape[0],'\n')
print(merged_orders.duplicated().sum(),'\n')
categoric_cols=['pais','dispositivo','fuente_referencia','nombre_producto','categoria_producto']
for i in categoric_cols:
    print(i)
    print(merged_orders[i].value_counts())
    print()
```

Con el merged listo puedo sacar el costo multiplicando la cantidad por el costo unitario
``` python
merged_orders['costo_total']=\
    merged_orders['cantidad'] * merged_orders['costo_unitario']
merged_orders.head()
```

Obtenemos el costo total por los productos
``` python
total_cost=merged_orders['costo_total'].sum()
total_cost
```

Obtenemos el costo total de Marketing
``` python
total_marketing=clean_marketing['gasto'].sum()
total_marketing
```

Teoricamente, un negocio es rentable si hay ganancia considerando el costo de los productos y el costo de marketing
``` python
total_profit=revenue-total_cost-total_marketing
total_profit
```

### 2.2 Parte 2
Para el ticket promedio lo haremos con el monto total promedio
``` python
mean_ticket=merged_orders['monto_total'].mean()
mean_ticket
```

La cantidad promedio por orden
``` python
mean_qty=merged_orders['cantidad'].mean()
mean_qty
```

Para el producto mas vendido
``` python
merged_orders\
    .groupby('nombre_producto')['monto_total']\
    .sum().reset_index()\
    .sort_values(by='monto_total',ascending=False)
```

El gasto por canal
``` python
clean_marketing\
    .groupby('canal')['gasto']\
    .sum().reset_index()\
    .sort_values(by='gasto',ascending=False)
```

## Paso 3: Funnel de conversion

⚙️**Conexión a la base de datos**:\
Se ejecuta la línea de configuración para conectar con la base de datos
y aplicar consultas SQL en la tabla **events**.

------------------------------------------------------------------------

**Parte 1: Construcción del funnel**

-   ¿Cuántos usuarios llegan a cada etapa del funnel?\
-   Calcular el número de usuarios únicos por `nombre_evento`\
-   Ordenar los eventos según el flujo del usuario

------------------------------------------------------------------------

**Parte 2: Análisis de conversión**

-   Calcular la tasa de conversión entre cada paso del funnel\
-   Identificar en qué etapa se pierde la mayor cantidad de usuarios\
-   Conversion Final

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

Explorar tabla events
``` python
query_events = '''
SELECT *
FROM events;
'''
events = pd.read_sql(query_events, con=engine)
events.head()
```

Reviso que si hay duplicados
``` python
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

Reviso si hay faltantes en first_visit
``` python
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

Reviso si hay faltantes en select_item
``` python
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

Reviso si hay faltantes en add_to_cart
``` python
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

Reviso si hay faltantes en begin_checkout
``` python
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

Reviso si hay faltantes en add_payment_info
``` python
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

### 3.1 Totales del funnel

Encontre faltantes en todas las fases, por lo que la idea es considerar solo las que se va dando seguimiento. 
Que en este caso son las que aparecen en first_visit

``` python

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

Podemos ver que la curva es muy ligera, parece que hay un error de medicion entre select_item y add_to_chart. Pero creo puede justificarse ya que hay paginas que directamente pueden agregar al carrito. Aun así valdría la pena sugerir que la herramienta registre cada add_to_chart directo como un select_item.

### 3.2 Conversiones

``` python
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
Por lo mismo que comente anteriormente, se muestra evidente que vale la pena dar esa sugerencia al cliente


## Paso 4: Retención por cohortes

**Tablas**

-   `users`
-   `user_activity`

------------------------------------------------------------------------

1.  Identificar la cohorte de cada usuario según el **mes de
    registro**.

2.  Calcular la retención semanal: cuántos usuarios **se mantienen
    activos** en cada semana desde su registro.

    -   `retenido_w1`: usuarios activos en la semana 1\
    -   `retenido_w2`: usuarios activos en la semana 2\
    -   `retenido_w3`: usuarios activos en la semana 3

3.  Calcular el porcentaje de retención para cada semana, dividiendo
    los usuarios retenidos entre los clientes iniciales de la cohorte:

    -   `semana_1`: porcentaje de usuarios retenidos en la semana 1\
    -   `semana_2`: porcentaje de usuarios retenidos en la semana 2\
    -   `semana_3`: porcentaje de usuarios retenidos en la semana 3

*Necesito revisar que la columna de fecha esté en formato correcto (`DATE`).

Explorando la tabla users
``` python
query_users = '''
SELECT *
FROM users;
'''
users = pd.read_sql(query_users, con=engine)
users.head(3)
```

Revisamos el tipo de dato de fecha de users
``` python
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

Reviso que si hay duplicados, solo id_usuario debe ser unico
``` python
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

Explorar tabla user_activity
``` python
query_user_activity = '''
SELECT *
FROM user_activity;
'''
user_activity = pd.read_sql(query_user_activity, con=engine)
user_activity.head(3)
```

Revisamos el tipo de dato de fecha de user_activity
``` python
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

Retención por cohortes
``` python
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

## Paso 5: Test estadístico

1.  **Plantear la hipótesis estadística**\
2.  **Analizar el dataset** `experiment_checkout_ui.csv` para
    identificar la métrica principal **conversion**.
    -   La métrica **conversion** es 1 si el usuario completó la compra,
        0 si no.\
3.  **Aplicar el test estadístico**
4.  **Interpretar el resultado**


------------------------------------------------------------------------

### 5.1 Hipótesis estadística

-   **H₀ (Hipótesis nula):** El tratamiento no afecta la tasa de
    conversion
-   **H₁ (Hipótesis alternativa):** El tratamiento influye en la tasa de
    conversion

### 5.2 Analisis de la informacion
Extraemos el csv y damos una revisada general
``` python
experiment = \
    pd.read_csv('https://practicum-content.s3.amazonaws.com/datasets/experiment_checkout_ui.csv')
experiment.head()
```

Vemos el tamaño de la muestra
``` python
experiment.shape
```

Revisamos si no hay valores nulos
``` python
experiment.info()
```

Buscamos duplicados, en este caso solo id_usuario debe ser unico
``` python
experiment['id_usuario'].duplicated().sum()
```

Solo necesitamos formatear la fecha
``` python
experiment['timestamp']=\
    pd.to_datetime(experiment['timestamp'],format='%Y-%m-%d')
experiment.dtypes
```

Revisamos las categorias
``` python
categoric_cols=['variante','dispositivo','pais']
for i in categoric_cols:
    print(i)
    print(experiment[i].value_counts())
    print()
```

Revisamos valores numericos
``` python
num_cols=['convirtio','duracion_sesion']
for i in num_cols:
    print(i)
    print(experiment[i].describe())
    print()
```
### 5.3 Aplicar el Test Estadístico
Al ser taza se conversion y para probar efectividad del tratamiento usamos Prueba Z
``` python
from statsmodels.stats.proportion import proportions_ztest
conversion=experiment.groupby('variante')['convirtio'].sum()
conversion
```

Ya con las conversiones, obtenemos los totales
``` python
total=experiment.groupby('variante')['convirtio'].count()
total
```

Transformamos en lista para la Prueba Z
``` python
conteo=[conversion['tratamiento'],conversion['control']]
observaciones=[total['tratamiento'],total['control']]
```

Aplicamos Prueba Z y revisamos los valores
``` python
z,p_value=proportions_ztest(conteo , observaciones)
print("Z: ",z)
print("P Value: ",p_value)
```

### 5.4 Interpretación
Definimos alpha y dejamos que el mismo codigo responda si se rechaza o no la hipotesis nula
``` python
alpha = 0.05  # umbral de significancia
if p_value < alpha:
    print("Rechazamos la hipótesis nula: hay evidencia de una diferencia.")
else:
    print("No rechazamos la hipótesis nula: no hay evidencia suficiente de una diferencia.")
```

Finalmente obtenemos las tasas de conversion y comparamos
``` python
tasa_tratamiento = conteo[0]/observaciones[0]
tasa_control = conteo[1]/observaciones[1]

print("\nTasa de conversion en tratamiento: ",tasa_tratamiento)
print("Tasa de conversion en control: ",tasa_control)
print("Diferencia: ",tasa_tratamiento-tasa_control)
```


**Test estadístico:** Prueba Z, resultado de z: 0.8132782986429474\
**Nivel de significancia alpha:** 0.41605851639119995

## Paso 6: Comunicar los resultados (Dashboard en Tableau)

Se usarán los CSVs limpios del Paso 1:

-   `orders_clean.csv`\
-   `catalog_clean.csv`\
-   `marketing_clean.csv`

------------------------------------------------------------------------

Dashboard 1: Overview Ejecutivo 

**KPIs principales a mostrar:**
-   Revenue total
-   Profit total
-   Gasto total en marketing
-   Ticket promedio
-   Cantidad promedio de productos por orden

**Visualizaciones:**
-   Tarjetas KPIs
-   Gráfico de líneas: evolución mensual de revenue o profit
-   Gráfico de líneas WeekToDate
-   Gráfico de barras: revenue y profit por producto o categoría

------------------------------------------------------------------------

3️⃣ Dashboard 2: Detalle / Drill-through\

**Visualizaciones:**
-   Tabla detallada de órdenes con:
    -   producto, cantidad, revenue, cost, profit
    -   color condicional
-   Gráfico de barras por producto con medida `cantidad vendida`
-   Drill-through: seleccionar un producto y ver todos los pedidos
    relacionados
-   Filtros

## Entrega Final 

Link público del dashboard publicado en **Tableau Public**:

https://public.tableau.com/views/DA_FinalProject_17823503737380/OverviewEjecutivo?:language=en-GB&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link

