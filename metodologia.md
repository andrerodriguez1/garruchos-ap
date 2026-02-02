# Metodología de Filtrado de Mapas de Rendimiento

## Introducción

Los monitores de rendimiento instalados en cosechadoras registran datos georreferenciados de productividad durante la cosecha. Sin embargo, estos datos crudos contienen errores sistemáticos que deben ser corregidos antes de su uso en agricultura de precisión.

## Fuentes de error en mapas de rendimiento

### Errores de cabecera
- **Causa:** Al comenzar o terminar una pasada, el flujo de grano no se estabiliza inmediatamente
- **Efecto:** Rendimientos falsamente bajos al inicio y falsamente altos al final de cada pasada
- **Solución:** Filtrar por flujo mínimo y altura de cabezal

### Errores de velocidad
- **Causa:** Cuando la cosechadora reduce velocidad (por ejemplo, para descargar tolva), el flujo de grano continúa pero el área cubierta disminuye
- **Efecto:** Rendimientos falsamente altos en zonas de baja velocidad
- **Solución:** Filtrar puntos con velocidad fuera del rango normal de operación

### Errores de calibración
- **Causa:** Sensor de flujo o humedad mal calibrado
- **Efecto:** Sesgo sistemático en todo el mapa (sobre o subestimación)
- **Nota:** Un error del 1% en humedad modifica el rendimiento en ~88 kg/ha

### Superposición de pasadas
- **Causa:** GPS con deriva o pasadas superpuestas en cabeceras
- **Efecto:** Puntos duplicados en la misma ubicación
- **Solución:** Filtrar por distancia mínima entre puntos

## Pipeline de filtrado implementado

### Paso 1: Filtro por flujo de grano
```
Eliminar puntos donde: flujo < umbral_flujo (típicamente Q05 de la referencia)
```
Elimina datos de cabeceras, paradas y momentos de bajo flujo.

### Paso 2: Filtro por rango de rendimiento
```
Eliminar puntos donde: rendimiento < Q02 OR rendimiento > Q98
```
Elimina valores biológicamente imposibles o atípicos extremos.

### Paso 3: Filtro de outliers locales (KDTree)
```
Para cada punto:
  1. Encontrar los k vecinos más cercanos
  2. Calcular la media y desviación estándar del rendimiento de los vecinos
  3. Calcular z-score del punto: z = (rend_punto - media_vecinos) / std_vecinos
  4. Eliminar si |z| > umbral_z
```
Detecta puntos que difieren significativamente de su entorno espacial.

## Parámetros por defecto

| Parámetro | Valor típico | Descripción |
|-----------|--------------|-------------|
| Umbral flujo | Q05 de referencia | Percentil 5 del flujo en la referencia |
| Q_LOW | Q02 del rendimiento | Límite inferior de rendimiento |
| Q_HIGH | Q98 del rendimiento | Límite superior de rendimiento |
| k vecinos | 15-25 | Cantidad de vecinos para análisis local |
| Umbral z-score | 2.5-3.0 | Máxima desviación permitida |

## Validación contra referencia

El mapa filtrado se compara contra un mapa de referencia usando las siguientes métricas:

### Coincidencia espacial
Porcentaje de puntos del filtrado que también están en la referencia (usando búsqueda por radio con KDTree).

### Correlación de Pearson
Correlación entre valores de rendimiento en puntos coincidentes. Mide si la estructura espacial de variabilidad se conservó.

### RMSE (Root Mean Square Error)
Error cuadrático medio entre valores de rendimiento en puntos coincidentes. Representa la precisión de las estimaciones locales.

## Referencias bibliográficas

1. **Kharel, T. P., et al. (2018).** "Protocols for processing cereal grain yield monitor data." *Agronomy Journal*.

2. **Bragachini, M., Méndez, A., Scaramuzza, F.** "Agricultura de Precisión." *INTA Manfredi, Proyecto de Agricultura de Precisión*.

3. **Blackmore, S., & Moore, M. (1999).** "Remedial correction of yield map data." *Precision Agriculture, 1(1), 53-66*.

4. **Manual de Agricultura de Precisión.** *IICA/PROCISUR*.

## Historial de versiones

| Versión | Fecha | Cambios |
|---------|-------|---------|
| v5 | 2026-02 | Detección automática de cultivo, validación multi-lote |
| v4 | 2026-01 | Agregado filtro KDTree para outliers locales |
| v3 | 2025-12 | Integración con mapa de referencia |
