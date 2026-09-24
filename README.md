# Cassandra Logistics App

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Apache Cassandra](https://img.shields.io/badge/Apache%20Cassandra-4.1-1287B1?logo=apachecassandra&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)

Aplicación de consola en Python para consultar **órdenes, productos y envíos** de un sistema de logística, con un modelo de datos en **Apache Cassandra** diseñado a partir de las consultas que necesita la aplicación.

Es un proyecto académico de bases de datos NoSQL. Partí de una plantilla base (menú, conexión y la primera consulta) y me tocó completar el modelo de datos, las consultas y la carga de datos.

## Funcionalidades

Desde un menú interactivo se puede:

| Opción | Consulta |
|---|---|
| 0 | Generar datos de ejemplo (100 órdenes, 300 productos y 1,000 envíos) |
| 1 | Órdenes de un cliente (Q1) |
| 2 | Productos de una orden, con subtotales y total (Q2) |
| 3 | Todos los envíos de una orden (Q3.1) |
| 4 | Envíos de una orden en un rango de fechas (Q3.2) |
| 5 | Envíos por orden y status, con rango de fechas (Q3.3) |
| 6 | Envíos por orden y tipo de envío, con rango de fechas (Q3.4) |
| 7 | Envíos por orden, tipo y status, con rango de fechas (Q3.5) |
| 8 | Cambiar el cliente con el que se trabaja |

Si no se escribe un rango de fechas, las consultas usan los últimos 30 días.

## Modelo de datos

En Cassandra no se hacen JOINs, así que se crea **una tabla por consulta**. La partition key es el dato con el que se busca y las clustering keys sirven para filtrar y ordenar dentro de la partición.

| Tabla | Partition key | Clustering keys | Consulta |
|---|---|---|---|
| `orders_by_customers` | `email` | `order_date DESC` | Q1 |
| `products_by_order` | `order_number` | `product_name` | Q2 |
| `shipments_by_o_sd` | `order_number` | `shipment_date DESC` | Q3.1, Q3.2 |
| `shipments_by_o_ssd` | `order_number` | `status`, `shipment_date DESC` | Q3.3 |
| `shipments_by_o_tsd` | `order_number` | `shipment_type`, `shipment_date DESC` | Q3.4 |
| `shipments_by_o_tssd` | `order_number` | `shipment_type`, `status`, `shipment_date DESC` | Q3.5 |

Algunas decisiones del diseño:

- **Datos duplicados a propósito:** los envíos se guardan en cuatro tablas, cada una organizada para una consulta distinta. Así cada consulta lee una sola partición.
- **Orden de las clustering keys:** los filtros de igualdad (`status`, `shipment_type`) van primero y la fecha al final, porque Cassandra solo permite filtrar por rango en la última columna que se usa.
- **Fechas como `TIMEUUID`:** dos envíos del mismo día no se sobrescriben, y el rango de fechas se consulta con `minTimeUuid()` / `maxTimeUuid()`.
- **Más recientes primero:** todas las tablas de envíos usan `CLUSTERING ORDER BY shipment_date DESC`.

## Tecnologías

- Python 3.10
- Apache Cassandra 4.1 con [cassandra-driver](https://github.com/datastax/python-driver)
- Docker y Docker Compose

## Cómo ejecutarlo

1. Levanta Cassandra (la primera vez tarda alrededor de un minuto en estar listo):

   ```bash
   docker compose up -d
   ```

2. Crea un entorno virtual e instala las dependencias:

   ```bash
   python -m venv venv
   source venv/bin/activate        # Windows: .\venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   ```

3. Corre la aplicación y usa primero la opción `0` para cargar datos de ejemplo:

   ```bash
   python app.py
   ```

Variables de entorno opcionales:

| Variable | Default |
|---|---|
| `CASSANDRA_CLUSTER_IPS` | `127.0.0.1` |
| `CASSANDRA_KEYSPACE` | `logistics` |
| `CASSANDRA_REPLICATION_FACTOR` | `1` |

## Ejemplo

Envíos con status `Delivered` de una orden, sin escribir fechas (últimos 30 días):

```
=== Delivered shipments for order: ORD-BA50B9D6 (2026-08-24 to 2026-09-23) ===
Shipment: TRK-569A95FFB5
  - Date: 2026-08-28
  - Customer: Sofía Hernández
  - Status: Delivered
  - Type: Express
  - Amount: $11,850.00
```

## Estructura

```
├── app.py               # Menú interactivo y entrada del usuario
├── model.py             # Tablas, consultas CQL, carga de datos y funciones de consulta
├── docker-compose.yml   # Cassandra local
└── requirements.txt
```

## Lo que aprendí

- Diseñar tablas en Cassandra empezando por las consultas, no por las entidades.
- Elegir partition y clustering keys según cómo se filtra y ordena cada consulta.
- Trabajar con `TIMEUUID` y rangos de fechas en CQL.
- Cargar datos en lotes con statements preparados.

## Autor

**Andres Olvera** · [GitHub](https://github.com/AndressOlvera)
