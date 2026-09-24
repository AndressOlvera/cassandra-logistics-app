# Cassandra Logistics App

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Apache Cassandra](https://img.shields.io/badge/Apache%20Cassandra-4.1-1287B1?logo=apachecassandra&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

Aplicación de línea de comandos en Python para consultar **órdenes, productos y envíos** de un sistema de logística. Utiliza un modelo de datos en **Apache Cassandra** diseñado a partir de los patrones de consulta de la aplicación (*query-driven data modeling*).

## Contexto y alcance

Proyecto académico enfocado en bases de datos NoSQL. A partir de una plantilla base con el menú, la conexión al clúster y la consulta de órdenes por cliente, desarrollé:

- El diseño e implementación de las tablas de productos y envíos.
- Las consultas CQL para productos y envíos, incluyendo filtros por status, tipo de envío y rango de fechas.
- Las funciones de acceso a datos y las opciones del menú interactivo.
- La captura de rangos de fechas con valores por defecto y manejo de entradas inválidas.
- La carga masiva de datos de ejemplo en todas las tablas.

La implementación superó los 10 casos de prueba del validador automatizado del curso.

## Funcionalidades

| Opción | Consulta |
|---|---|
| 0 | Carga de datos de ejemplo (100 órdenes, 300 productos y 1,000 envíos) |
| 1 | Órdenes de un cliente |
| 2 | Productos de una orden, con subtotales y total |
| 3 | Historial completo de envíos de una orden |
| 4 | Envíos de una orden en un rango de fechas |
| 5 | Envíos por orden y status, en un rango de fechas |
| 6 | Envíos por orden y tipo de envío, en un rango de fechas |
| 7 | Envíos por orden, tipo y status, en un rango de fechas |
| 8 | Cambio del cliente activo |

Cuando no se especifica un rango de fechas, las consultas usan los últimos 30 días.

## Modelo de datos

Cassandra no admite JOINs, por lo que el esquema define **una tabla por consulta**. La partition key corresponde al criterio de búsqueda y las clustering keys determinan el filtrado y el orden dentro de cada partición.

| Tabla | Partition key | Clustering keys | Consulta que resuelve |
|---|---|---|---|
| `orders_by_customers` | `email` | `order_date DESC` | Órdenes por cliente |
| `products_by_order` | `order_number` | `product_name` | Productos por orden |
| `shipments_by_o_sd` | `order_number` | `shipment_date DESC` | Envíos por orden y fecha |
| `shipments_by_o_ssd` | `order_number` | `status`, `shipment_date DESC` | Envíos por status y fecha |
| `shipments_by_o_tsd` | `order_number` | `shipment_type`, `shipment_date DESC` | Envíos por tipo y fecha |
| `shipments_by_o_tssd` | `order_number` | `shipment_type`, `status`, `shipment_date DESC` | Envíos por tipo, status y fecha |

### Decisiones de diseño

- **Desnormalización intencional:** los envíos se almacenan en cuatro tablas, cada una organizada para una consulta. Así cada consulta se resuelve leyendo una sola partición.
- **Orden de las clustering keys:** los filtros de igualdad (`status`, `shipment_type`) preceden a la fecha, ya que Cassandra solo permite restricciones de rango en la última clustering key utilizada.
- **Fechas como `TIMEUUID`:** permite distinguir envíos registrados en la misma fecha y consultar rangos con `minTimeUuid()` y `maxTimeUuid()`.
- **Orden cronológico descendente:** las tablas de envíos devuelven primero los registros más recientes (`CLUSTERING ORDER BY shipment_date DESC`).

## Competencias técnicas aplicadas

- **Modelado de datos NoSQL:** diseño orientado a consultas y desnormalización en Apache Cassandra.
- **Diseño de claves:** selección de partition keys y clustering keys según los filtros y el orden de cada consulta.
- **CQL:** consultas por rango temporal sobre columnas `TIMEUUID`.
- **Python:** acceso a datos con `cassandra-driver`, *prepared statements* y escritura por lotes (*batches*).
- **Contenedores:** entorno de desarrollo reproducible con Docker Compose.

## Tecnologías

- Python 3.10
- Apache Cassandra 4.1
- [cassandra-driver](https://github.com/datastax/python-driver) (DataStax)
- Docker y Docker Compose

## Instalación y ejecución

1. Iniciar Cassandra (la primera vez puede tardar alrededor de un minuto en estar disponible):

   ```bash
   docker compose up -d
   ```

2. Crear un entorno virtual e instalar las dependencias:

   ```bash
   python -m venv venv
   source venv/bin/activate        # Windows: .\venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

3. Ejecutar la aplicación y seleccionar la opción `0` para cargar los datos de ejemplo:

   ```bash
   python app.py
   ```

### Configuración

| Variable de entorno | Valor por defecto |
|---|---|
| `CASSANDRA_CLUSTER_IPS` | `127.0.0.1` |
| `CASSANDRA_KEYSPACE` | `logistics` |
| `CASSANDRA_REPLICATION_FACTOR` | `1` |

## Ejemplo de uso

Envíos con status `Delivered` de una orden, usando el rango por defecto (últimos 30 días):

```
=== Delivered shipments for order: ORD-BA50B9D6 (2026-08-24 to 2026-09-23) ===
Shipment: TRK-569A95FFB5
  - Date: 2026-08-28
  - Customer: Sofía Hernández
  - Status: Delivered
  - Type: Express
  - Amount: $11,850.00
```

## Estructura del proyecto

```
├── app.py               # Menú interactivo y captura de datos del usuario
├── model.py             # Esquema, consultas CQL, carga de datos y acceso a datos
├── docker-compose.yml   # Servicio local de Cassandra
└── requirements.txt     # Dependencias de Python
```

## Autor

**Andres Olvera** · [GitHub](https://github.com/AndressOlvera)
