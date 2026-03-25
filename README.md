# Colecta, integración y procesamiento de datos biofísicos para mapeo con imágenes satelitales

## 1. Colecta de datos en campo
- **Variables medidas:** FAPAR y LAI
- **Instrumentos utilizados:**
  - LAIPEN
  - Accupar LP-80
- **Esquemas de muestreo:**
  - Puntual
  - Conglomerado (centro + puntos cardinales separados por 3 m)
- **Datos registrados:**
  - Coordenadas GPS
  - Instrumento utilizado
  - Tres lecturas de PAR
  - Promedio de PAR
  - Hora y fecha
  - Sitio
  - Valor de LAI

## 2. Imágenes satelitales
- **Fuente:** Planet (resolución espacial de 3 m)
- **Índices espectrales generados:**
  - Cuatro índices derivados para captar variabilidad del dosel (NDVI, GNDVI, NDRE, CIRE)
- **Objetivo:** dirigir trabajo de campo en primeras salidas (temporada seca 2025)

## 3. Integración de datos
- Extracción de valores de bandas espectrales de imágenes Planet cercanas a la fecha de muestreo
- Obtención de variables explicativas para el modelo (RED, GREEN, NIR, RE, NDVI, GNDVI, NDRE, CIRE)

## 4. Base de datos tabular
- Variables explicativas (bandas e índices espectrales)
- Valores de LAI y FAPAR
- Datos listos para entrenamiento de modelo

## 5. Modelado
- **Modelo:** Redes neuronales
- **Librerías:** `tensorflow` y `keras`
- **Lenguaje:** Python 3.9
- **Interfaz:** RStudio
- **Complemento:** librería `terra` para análisis espaciales

## 6. Resultados
- Entrenamiento del modelo con datos de campo + variables espectrales
- Reconstrucción de mapas espaciales de LAI y FAPAR
- Integración de análisis reproducibles en RStudio con Python
