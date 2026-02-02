# Pipeline de Mapas de Rendimiento

Procesamiento y validación de mapas de rendimiento para agricultura de precisión.

## Uso

1. Colocar archivos en `datos/inputs/`:
   - Mapa original (.gpkg)
   - Mapa referencia (.gpkg)  
   - Contorno del lote (.kmz)

2. Ejecutar `notebooks/Procesamiento_Rinde.ipynb`

3. Ejecutar `notebooks/Generar_Reporte.ipynb`

4. El reporte HTML se genera en `datos/outputs/`

## Metodología

Ver [docs/metodologia.md](docs/metodologia.md)
