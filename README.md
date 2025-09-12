# Proyecto de Integración con QuickBooks

Este proyecto se encarga de la integración de datos entre QuickBooks y nuestro sistema. El propósito es extraer y procesar datos de varias entidades de QuickBooks como **Invoice**, **Customer**, **Item**, entre otras, para almacenarlos y analizarlos en nuestro sistema.

# Proyecto de Integración con QuickBooks

Este proyecto se encarga de la integración de datos entre QuickBooks y nuestro sistema. El propósito es extraer y procesar datos de varias entidades de QuickBooks como **Invoice**, **Customer**, **Item**, entre otras, para almacenarlos y analizarlos en nuestro sistema.

## Diagrama de Arquitectura

![Diagrama de Arquitectura](./path_to_architecture_diagram.png)

La arquitectura está basada en contenedores que gestionan la extracción de datos de QuickBooks, su transformación y almacenamiento en nuestro sistema. Los datos extraídos son procesados, validados y almacenados para su posterior análisis.

## Pasos para levantar contenedores y configurar el proyecto

### Requisitos previos
1. **Docker**: Asegúrate de tener Docker instalado. Si no lo tienes, sigue las instrucciones [aquí](https://docs.docker.com/get-docker/).
2. **Docker Compose**: Este proyecto utiliza Docker Compose para orquestar los contenedores. Si no lo tienes instalado, sigue las instrucciones [aquí](https://docs.docker.com/compose/install/).

### Levantar contenedores

1. Clona el repositorio en tu máquina local:

   ```bash
   git clone https://github.com/TU-USUARIO/mi-proyecto.git
   cd mi-proyecto
2. Construir los contenedores
   docker-compose build
3. Levantar los contenedores
   docker-compose up
4. Accede a los servicios. Los contenedores se levantarán según lo definido en el archivo docker-compose.yaml.
## Gestión de secretos
### Nombres
- qb_client_id: ID del cliente para la integración con QuickBooks.
- qb_client_secret: Secreto del cliente para la integración con QuickBooks.
- qb_refresh_token: Token de refresco utilizado para obtener el access_token de la API de QuickBooks.
- qb_realm_id: Identificador único del entorno de QuickBooks.
### Propósito
Los secretos se utilizan para la autenticación con la API de QuickBooks y la configuración del entorno. Son vitales para realizar las conexiones seguras con el servicio de QuickBooks y obtener el access token.
### Rotación
El token de acceso se actualiza automaticamente mediante el refresh token, se espera que la rotación de secretos se realice cada 3 meses, ya que es la duración del refresh token.

## Pipelines
Cada pipeline se encarga de extraer datos de QuickBooks de distintas entidades (Invoice, Customer, Item) y almacenarlos en el sistema.
### Parámetros
- Fecha de inicio: Fecha desde la cual se comenzarán a extraer los datos.
- Fecha de fin: Fecha hasta la cual se extraerán los datos.
Las fechas y horas serán tomadas en formato ISO 8601, en horario UTC.
### Segmentación
Los datos son segmentados en intervalos de tiempo, en este caso diarios, dependiendo de las necesidades del proyecto. La segmentación permite manejar grandes volúmenes de datos de manera más eficiente.
### Límites
- Límite por página: 100 registros por página.
- Límite de reintentos: 5 reintentos en caso de error.
### Reintentos
En caso de que la extracción de datos falle debido a problemas de red, tiempo de espera, falla del token de acceso, el pipeline intentará reintentar la solicitud hasta 5 veces antes de detenerse.
### Runbook
- Reintentar: En caso de fallo, revisa los logs de error, que se imprimen cada ejecución
- Reintentos fallidos: Si los 5 reintentos fallan, el pipeline se detendrá.
- Acción manual: Verifica la conectividad de la API de QuickBooks y los parámetros de la consulta, por ejemplo con ayuda de Postman
## Trigger one-time
Los triggers one-time se ejecutan a una fecha y hora específica. El trigger tomará la hora en UTC y realizará la ejecución a esa hora exacta.
- Fecha y hora ejecutados (UTC): 2025-09-12 03:45:54
- Fecha y hora Guayaquil: 2025-09-12 22:45:54
Una vez que el trigger se ejecute, se desactivará automáticamente para evitar que se ejecute nuevamente. Esto asegura que las ejecuciones sean únicas y no se repitan.
## Esquema raw
Cada entidad extraída de QuickBooks se almacena en su propia tabla en el esquema raw. Las entidades incluyen:
- Invoice
- Customer
- Item
### Claves
Cada tabla tendrá una columna id que actúa como la clave primaria para identificar de manera única cada registro.
### Metadatos obligatorios
Las tablas contienen una serie de metadatos los cuales son:
- ingested_at_utc (timestamp de carga)
- extract_window_start_utc
- extract_window_end_utc
- page_number/page_size
- request_payload
### Idempotencia
Los pipelines son idempotentes, lo que significa que puedes ejecutar el mismo pipeline varias veces sin afectar el resultado final. Si un registro ya existe, no se duplicará.
## Validaciones/volumetría
Revisar los prints durante la ingesta de datos, estos contienen:
- Paginas ingestadas
- Fecha la cual se esta ingestando y cuantos datos contiene dicha fecha
- Si se encontraron o no datos en dicha fecha
- Duración de la ingesta de los datos
## Troubleshooting
### Autenticación
Si se experimentan problemas de autenticación con la API de QuickBooks:
- Verificar que los valores del client_id, client_secret, y refresh_token estén correctos.
- Asegurarse de que el realm_id sea válido.
- Asegurarse que el refresh_token no haya caducado.
### Paginación
Si la paginación no funciona como se espera:
- Revisa que el parámetro startposition esté correctamente configurado.
- Verifica que el tamaño de la página no exceda el límite de 100 registros, puede probarse con tamaños de página mas pequeños
### Límites
Asegurarse que no estas intentando acceder a más páginas de las posibles, ya que pueden ser limitadas por la ram.
### Timezones
Si los registros no se alinean con las fechas esperadas, asegúrate de que las fechas estén en UTC antes de hacer cualquier comparación.
### Almacenamiento y permisos
Si encuentras problemas con el almacenamiento o permisos:
- Verifica que las credenciales de la base de datos sean correctas.
- Asegúrate de que el contenedor tenga acceso a los volúmenes donde se almacenan los datos.
