# Proyecto de Integración con QuickBooks

Este proyecto se encarga de la integración de datos entre QuickBooks y nuestro sistema. El propósito es extraer y procesar datos de varias entidades de QuickBooks como **Invoice**, **Customer**, **Item**, entre otras, para almacenarlos y analizarlos en nuestro sistema.

## Diagrama de Arquitectura

La arquitectura del sistema está basada en contenedores Docker, que encapsulan diferentes servicios que interactúan entre sí. A continuación se describe cada componente y su función en el sistema:
### Base de datos (PostgreSQL)
- Servicio: warehouse
- Imagen: postgres:13
- Función: Almacena los datos extraídos de QuickBooks, como los detalles de las Invoice, Customer, e Item.
- Volumenes: Se monta un volumen en el contenedor para garantizar la persistencia de los datos.
- Puertos: El puerto 5432 se expone para que los servicios puedan acceder a la base de datos.

### Interfaz de Usuario para Base de Datos (pgAdmin):
- Servicio: warehouseui
- Imagen: dpage/pgadmin4
- Función: Proporciona una interfaz gráfica para gestionar la base de datos PostgreSQL y visualizar los datos almacenados.
- Volumenes: Se monta un volumen para la persistencia de los datos de pgAdmin.
- Puertos: El puerto 8080 se expone para acceder a la interfaz de pgAdmin.

### Orquestador (MageAI)
- Servicio: scheduler
- Imagen: mageai/mageai
- Función: MageAI es una plataforma que se utiliza para orquestar las tareas de integración de datos. En este caso, se encarga de ejecutar los pipelines que extraen datos de QuickBooks y los cargan en la base de datos.
- Volumenes: Se monta un volumen para la persistencia de los datos y configuración del orquestador.
- Puertos: El puerto 6789 se expone para acceder a la interfaz de MageAI y controlar el estado de las tareas programadas.
- Comando de inicio: El contenedor se inicializa con el comando /app/run_app.sh mage start scheduler, que lanza el scheduler de MageAI.

### Interacción entre los servicios
- QuickBooks: El sistema extrae datos de diferentes entidades desde QuickBooks a través de la API de QuickBooks.
- MageAI: MageAI se encarga de orquestar el proceso de extracción y transformación de los datos desde QuickBooks. Los pipelines definidos en MageAI son responsables de la segmentación de datos, gestión de errores y la persistencia de los datos procesados.
- Base de Datos PostgreSQL: Una vez que MageAI ha procesado los datos, estos se almacenan en una base de datos PostgreSQL. Esta base de datos es utilizada para almacenar todos los datos de QuickBooks.
- pgAdmin: pgAdmin proporciona una interfaz gráfica para que se pueda visualizar, consultar y gestionar los datos almacenados en la base de datos PostgreSQL.

### Flujo de Datos
1. Extracción de Datos: Los pipelines de MageAI se ejecutan y extraen datos desde la API de QuickBooks.
2. Transformación: Los datos extraídos son procesados, se les agrega las columnas de Metadata
3. Carga: Los datos transformados son cargados en la base de datos PostgreSQL, donde se almacenan de manera estructurada.
4. Visualización: Se utilizar pgAdmin para acceder a los datos almacenados y realizar consultas.

<img width="649" height="693" alt="image" src="https://github.com/user-attachments/assets/2ea0d4c6-eb80-461d-909c-6026beda5aab" />


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
# Checklist
- [x]Mage y Postgres se comunican por nombre de servicio.
- [x]Todos los secretos (QBO y Postgres) están en Mage Secrets; no hay secretos en el repo/entorno expuesto.
- [x]Pipelines qb_<entidad>_backfill acepta fecha_inicio y fecha_fin (UTC) y segmenta el rango.
- [x]Trigger one-time configurado, ejecutado y luego deshabilitado/marcado como completado.
- [x]Esquema raw con tablas por entidad, payload completo y metadatos obligatorios.
- [x]Idempotencia verificada: reejecución de un tramo no genera duplicados.
- [x]Paginación y rate limits manejados y documentados.
- [x]Volumetría y validaciones mínimas registradas y archivadas como evidencia.
- [x]Runbook de reanudación y reintentos disponible y seguido.
