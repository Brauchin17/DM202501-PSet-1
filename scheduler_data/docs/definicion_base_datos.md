# Esquema de Base de Datos (Raw)

## Tablas

### Tabla: `invoices`
| Campo              | Tipo de Dato | Descripción                                      |
|--------------------|---------------|--------------------------------------------------|
| `id`               | INTEGER       | Identificador único de la factura                |
| `payload`          | JSONB       | Datos extraidos del QB           |
| `ingested_at_utc`     | TIMESTAMP       | Metadato del tiempo al que fue ingestado                        |
| `extract_window_start_utc`       | TIMESTAMP     | Metadato del tiempo al que empezó la extracción                |
| `extract_window_end_utc`         | TIMESTAMP         | Metadato del tiempo al que terminó la extracción                 |
| `page_number`         | INTEGER         | Página de la cual fue extraida                 |
| `request_payload`         | STRING         | query con la cual fue extraido                 |

### Tabla `customers`

| Campo              | Tipo de Dato | Descripción                                      |
|--------------------|---------------|--------------------------------------------------|
| `id`               | INTEGER       | Identificador único del cliente                |
| `payload`          | JSONB       | Datos extraidos del QB           |
| `ingested_at_utc`     | TIMESTAMP       | Metadato del tiempo al que fue ingestado                        |
| `extract_window_start_utc`       | TIMESTAMP     | Metadato del tiempo al que empezó la extracción                |
| `extract_window_end_utc`         | TIMESTAMP         | Metadato del tiempo al que terminó la extracción                 |
| `page_number`         | INTEGER         | Página de la cual fue extraida                 |
| `request_payload`         | STRING         | query con la cual fue extraido                 |
  
### Tabla `items`

  | Campo              | Tipo de Dato | Descripción                                      |
|--------------------|---------------|--------------------------------------------------|
| `id`               | INTEGER       | Identificador único del item                |
| `payload`          | JSONB       | Datos extraidos del QB           |
| `ingested_at_utc`     | TIMESTAMP       | Metadato del tiempo al que fue ingestado                        |
| `extract_window_start_utc`       | TIMESTAMP     | Metadato del tiempo al que empezó la extracción                |
| `extract_window_end_utc`         | TIMESTAMP         | Metadato del tiempo al que terminó la extracción                 |
| `page_number`         | INTEGER         | Página de la cual fue extraida                 |
| `request_payload`         | STRING         | query con la cual fue extraido                 |
