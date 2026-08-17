# DataShop — Calidad de Datos y Análisis Exploratorio

Solución reproducible para la **Práctica 2, Unidad 1** de Minería de Datos. El proyecto evalúa, limpia y visualiza el conjunto `customers.csv` antes de cualquier técnica de segmentación.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/adriantorrico126/datashop-calidad-datos-practica-2/blob/main/analisis_datashop.ipynb)

## Entregables

- `analisis_datashop.ipynb`: desarrollo en Python, listo para Google Colab.
- `customers.csv`: fuente original proporcionada para la práctica.

## Ejecutar en Google Colab

1. Abra `analisis_datashop.ipynb` desde GitHub con el botón **Open in Colab** o pegue la URL del notebook en [Google Colab](https://colab.research.google.com/github).
2. Ejecute las celdas de arriba hacia abajo. El notebook detecta automáticamente si corre en Colab o de forma local.
3. Revise las figuras y las tablas de comparación generadas.

> Para que el notebook funcione en Colab, mantenga `customers.csv` en la raíz del repositorio.

## Criterio de limpieza y trazabilidad

No se sobrescribe el archivo fuente. El notebook conserva una copia (`df_original`), registra cada regla de calidad y produce el dataframe limpio (`df_clean`). Las reglas aplicadas son:

1. Normalizar textos: espacios, mayúsculas/minúsculas y valores de `Subscribed`.
2. Detectar identificadores repetidos y conservar el primer registro documentado por `CustomerID`, porque no existe una fuente externa para resolver conflictos.
3. Convertir valores imposibles (edad fuera de 18–100 años, ingresos negativos y visitas negativas) a faltantes.
4. Imputar faltantes numéricos con la mediana de la variable, una medida robusta frente a valores extremos.

## Estructura

```text
.
├── analisis_datashop.ipynb
├── customers.csv
├── requirements.txt
└── README.md
```

## Reproducibilidad

El análisis utiliza Python, pandas, NumPy, Matplotlib y Seaborn. Las versiones mínimas están declaradas en `requirements.txt`.
