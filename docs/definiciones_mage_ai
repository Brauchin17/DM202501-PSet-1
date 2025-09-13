
## 2. Definiciones del proyecto y pipelines en MageAI

### Proyecto en MageAI
El proyecto de **MageAI** está diseñado para la ingestión y exportación de datos provenientes de QuickBooks hacia el destino configurado.  
Su estructura se basa en **pipelines de backfill** que permiten cargar información histórica de distintas entidades clave: *invoices*, *items* y *customers*.  

Cada pipeline implementa dos componentes principales:
- **Data Loader**: encargado de extraer los datos de QuickBooks, incluyendo metadatos cuando corresponde.  
- **Data Exporter**: responsable de cargar la información procesada hacia postgres.  

### Pipelines `qb_<entidad>_backfill`
Existen tres pipelines con un patrón de nombre común `qb_<entidad>_backfill`, donde `<entidad>` corresponde a la tabla o recurso de QuickBooks a sincronizar:

1. **qb_invoices_backfill**  
   Pipeline que carga y exporta datos históricos de facturas (invoices).

2. **qb_items_backfill**  
   Pipeline que carga y exporta datos históricos de productos o servicios (items).

3. **qb_customers_backfill**  
   Pipeline que carga y exporta datos históricos de clientes (customers).

Cada uno de estos pipelines tiene la misma estructura mínima:
- **Data Loader** con definición de metadatos y extracción de datos.  
- **Data Exporter** con la lógica de carga hacia el destino.  

### Trigger one-time
Todas las pipelines cuentan con un **trigger de tipo `one-time`**, el cual permite ejecutar la carga completa bajo demanda.  
Este trigger ya ha sido ejecutado en cada una de las pipelines (`qb_invoices_backfill`, `qb_items_backfill` y `qb_customers_backfill`) para realizar un *backfill* inicial de datos históricos en un rango de 4 meses.  



