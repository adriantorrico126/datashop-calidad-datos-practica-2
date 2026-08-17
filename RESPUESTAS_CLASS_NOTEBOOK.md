# Unidad 1 — Práctica 2

## Caso de estudio: DataShop

**Objetivo.** Evaluar la calidad del archivo histórico de clientes de DataShop, limpiarlo de manera trazable y realizar un análisis exploratorio previo a una futura segmentación de clientes.

## Actividad 1: Conociendo los datos

1. **¿Cuántos registros contiene el dataframe?**

   El dataframe original tiene **52 registros**. Se obtuvo este valor utilizando la instrucción `df_original.shape[0]`.

2. **¿Cuántas variables contiene?**

   El dataframe tiene **8 variables**. Se obtuvo este valor utilizando `df_original.shape[1]`.

3. **¿Cuáles son las variables numéricas?**

   Las variables numéricas son: `CustomerID`, `Age`, `MonthlyIncome`, `PurchaseFrequency`, `AveragePurchaseAmount` y `WebsiteVisits`.

4. **¿Cuáles son las variables categóricas?**

   Las variables categóricas son: `PreferredCategory` y `Subscribed`.

5. **¿Qué variable representa el identificador del cliente?**

   La variable identificadora es `CustomerID`.

6. **¿Observas algún valor que pueda ser sospechoso?**

   Sí. Se detectaron edades de **250** y **-5** años, un ingreso mensual de **-500** y un número de visitas de **-3**. También se observaron registros con la edad o el ingreso faltante, identificadores repetidos y distintas formas de escribir una misma categoría, por ejemplo `Electronics`, `electronics` y `ELECTRONICS`.

7. **¿Qué problemas de calidad de datos identificaste inicialmente?**

   Se identificaron: valores faltantes en `Age` y `MonthlyIncome`; valores fuera de dominio; identificadores repetidos (`CustomerID`); e inconsistencias de formato en `PreferredCategory` y `Subscribed`. Aunque no había filas completamente duplicadas antes de normalizar los textos, la normalización reveló una repetición lógica para el cliente 50.

## Actividad 2: Limpieza de datos

### 2.a Problemas identificados e instrucciones de diagnóstico

| Problema | Instrucción de Python empleada |
|---|---|
| Tipos y estructura | `df_original.info()` y `df_original.dtypes` |
| Valores faltantes | `df_original.isna().sum()` |
| Filas duplicadas | `df_original.duplicated().sum()` |
| Identificadores repetidos | `df_original['CustomerID'].duplicated().sum()` |
| Valores fuera de dominio | Consultas con condiciones, por ejemplo `df_original.loc[~df_original['Age'].between(18, 100)]` |
| Variantes categóricas | `df_original['PreferredCategory'].value_counts(dropna=False)` y `df_original['Subscribed'].value_counts(dropna=False)` |

### 2.b Instrucciones utilizadas para solucionar los problemas

Primero se creó una copia para no modificar los datos originales:

```python
df_clean = df_original.copy(deep=True)
```

Se normalizaron los textos:

```python
df_clean['PreferredCategory'] = df_clean['PreferredCategory'].str.strip().str.title()
df_clean['Subscribed'] = df_clean['Subscribed'].str.strip().str.title()
```

Se eliminaron registros repetidos por identificador, conservando la primera ocurrencia como regla reproducible:

```python
df_clean = df_clean.drop_duplicates(subset='CustomerID', keep='first').copy()
```

Los valores fuera de dominio se transformaron en faltantes:

```python
df_clean.loc[~df_clean['Age'].between(18, 100), 'Age'] = np.nan
df_clean.loc[df_clean['MonthlyIncome'] < 0, 'MonthlyIncome'] = np.nan
df_clean.loc[df_clean['WebsiteVisits'] < 0, 'WebsiteVisits'] = np.nan
```

Finalmente, los valores faltantes se imputaron con la mediana de cada variable:

```python
for column in ['Age', 'MonthlyIncome', 'WebsiteVisits']:
    df_clean[column] = df_clean[column].fillna(df_clean[column].median())
```

La mediana se eligió porque es resistente a los valores atípicos. La eliminación de los IDs repetidos se documenta como una decisión de calidad: sin una fuente maestra de clientes no es válido combinar datos contradictorios de un mismo identificador.

## Actividad 3: Comparación antes y después

| Indicador | Antes | Después |
|---|---:|---:|
| Registros | 52 | 50 |
| Variables | 8 | 8 |
| Valores faltantes en `Age` | 1 | 0 |
| Valores faltantes en `MonthlyIncome` | 1 | 0 |
| IDs repetidos | 2 | 0 |
| Categorías preferidas distintas | 10 | 4 |
| Valores distintos de suscripción | 6 | 2 |
| Edad fuera de dominio | 2 | 0 |
| Ingreso negativo | 1 | 0 |
| Visitas negativas | 1 | 0 |

En el notebook se muestran ambos dataframes (`df_original` y `df_clean`). Se mantienen 50 clientes únicos y las 8 variables originales; no se eliminó ninguna columna.

## Actividad 4: Visualización de los datos

### Visualización 1: Distribución de edades

El intervalo de edad más frecuente es **35–44 años** (18 clientes); en conjunto, la mayor concentración se sitúa entre **25 y 45 años**. La distribución se agrupa principalmente en clientes adultos jóvenes y de mediana edad; después de la limpieza ya no hay edades imposibles que distorsionen el gráfico.

### Visualización 2: Frecuencia de compra

La mayor parte de los clientes realiza entre **2 y 10 compras**. Existe un grupo menor con frecuencia alta, de **11 a 15 compras**, que podría representar clientes más comprometidos o de mayor valor.

### Visualización 3: Categoría preferida

Tras normalizar las etiquetas, **Electronics** es la categoría preferida con mayor número de clientes (17). **Books** y **Home** son las menos populares, con 10 clientes cada una. Esta conclusión se verifica directamente en el gráfico de barras y con `value_counts()`.

### Visualización 4: Ingreso y frecuencia de compra

Se observa una **tendencia positiva marcada** (correlación aproximada de **0,75**): en general, ingresos mensuales mayores se asocian con una frecuencia de compra mayor, aunque la relación no es perfecta. También hay clientes con ingreso alto y una frecuencia relativamente baja, por lo que el ingreso no debe usarse como único criterio de segmentación. Los valores negativos se trataron como errores de calidad; después de limpiar no se observan atípicos imposibles en este par de variables.

### Visualización 5: Visitas y frecuencia de compra

Se aprecia una **relación positiva muy clara** (correlación aproximada de **0,96**): quienes visitan más el sitio suelen realizar más compras. El patrón sugiere que la actividad digital (`WebsiteVisits`) puede ser una variable útil para una futura segmentación y para campañas de conversión.

## Conclusión

El archivo original no estaba listo para aplicar algoritmos de Data Mining por errores de dominio, valores faltantes, registros con IDs repetidos e inconsistencias en textos. Después de una limpieza controlada y reproducible, se obtuvo una base de 50 clientes únicos sin faltantes ni valores imposibles. Como siguiente etapa se recomienda escalar las variables numéricas, codificar las categorías y evaluar segmentación con K-Means, seleccionando el número de grupos mediante métodos como codo y silhouette.
