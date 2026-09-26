Markdown

````
# Device Systems API — Versión 4.0

API REST desarrollada con **FastAPI**, **SQLAlchemy** y **Alembic** para la gestión relacional de usuarios, dispositivos y préstamos con persistencia en base de datos SQLite, correspondiente a la evidencia **GA1-220501096-01-AA1-EV10 – FastAPI con Alembic, Relaciones y Consultas Avanzadas**.

---

## 1. Descripción general

`device_systems` implementa un modelo de datos relacional robusto compuesto por tres entidades principales:
* **Users**: Usuarios del sistema categorizados por roles (`admin`, `support`, `user`).
* **Devices**: Inventario de equipos tecnológicos con control de estado y disponibilidad operativa.
* **Loans**: Registro transaccional que gestiona el préstamo de un equipo a un usuario, validando disponibilidad de inventario y fechas de devolución.

El ciclo de vida del esquema de base de datos se administra mediante migraciones versionadas con **Alembic**, garantizando trazabilidad y reproducibilidad de cambios estructurales.

---

## 2. Tecnologías utilizadas

* **Python 3.14**
* **FastAPI 0.115+**
* **SQLAlchemy 2.0** (ORM y mapeo relacional)
* **Alembic 1.20** (Control de versiones y migraciones de base de datos)
* **Pydantic v2** (Validación estricta y esquemas anidados)
* **SQLite** (Motor relacional)
* **Uvicorn** (Servidor ASGI)
* **uv** (Gestor de dependencias y entornos virtuales)

---

## 3. Estructura modular del proyecto

```text
device_systems/
├── app/
│   ├── main.py
│   ├── database/
│   │   ├── __init__.py
│   │   └── connection.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user_model.py
│   │   ├── device_model.py
│   │   └── loan_model.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user_schema.py
│   │   ├── device_schema.py
│   │   └── loan_schema.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── user_routes.py
│   │   ├── device_routes.py
│   │   └── loan_routes.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── device_service.py
│   │   └── loan_service.py
│   └── dependencies/
│       ├── __init__.py
│       └── database_dependency.py
├── alembic/
│   ├── versions/
│   ├── env.py
│   └── script.py.mako
├── Imagenes/
├── alembic.ini
├── pyproject.toml
├── requirements.txt
└── README.md
````

## 4. Instalación y ejecución

1. **Instalar dependencias y sincronizar entorno:**
    
    Bash
    
    ```
    uv sync
    ```
    
2. **Aplicar migraciones de base de datos:**
    
    Bash
    
    ```
    uv run alembic upgrade head
    ```
    
3. **Iniciar el servidor de desarrollo:**
    
    Bash
    
    ```
    uv run uvicorn app.main:app --reload
    ```
    

Acceso a la documentación interactiva: `http://127.0.0.1:8000/docs`

## 5. Modelo Relacional y Reglas de Negocio

- **Relación User ↔ Loan (1 a N):** Un usuario puede tener múltiples préstamos registrados en el histórico (`User.loans`).
    
- **Relación Device ↔ Loan (1 a N):** Un dispositivo conserva el historial completo de sus préstamos (`Device.loans`).
    
- **Integridad Referencial:** Claves foráneas obligatorias en la tabla `loans` hacia `users.id` y `devices.id`.
    
- **Regla de Conflicto (HTTP 409):** No es posible crear un préstamo si el equipo tiene `is_available = False`.
    
- **Devolución Transaccional:** La devolución actualiza la fecha `return_date`, cambia el estado a `returned` y restaura automáticamente `is_available = True` en el equipo.
    

## 6. Endpoints de la API

|**Módulo**|**Método**|**Ruta**|**Descripción**|**Códigos HTTP**|
|---|---|---|---|---|
|**Users**|GET|`/users`|Listar usuarios (filtros por rol y estado)|200|
||GET|`/users/{id}`|Consultar usuario por ID|200, 404|
||POST|`/users`|Registrar usuario (email único)|201, 400, 422|
||PUT|`/users/{id}`|Reemplazo total de usuario|200, 400, 404|
||PATCH|`/users/{id}`|Actualización parcial|200, 400, 404|
||DELETE|`/users/{id}`|Eliminar usuario|204, 404|
||GET|`/users/{id}/loans`|Listar préstamos asociados al usuario|200, 404|
|**Devices**|GET|`/devices`|Listar dispositivos (filtros: tipo, estado, marca, búsqueda)|200|
||GET|`/devices/{id}`|Consultar dispositivo por ID|200, 404|
||POST|`/devices`|Registrar dispositivo (serial único)|201, 400, 422|
||PUT|`/devices/{id}`|Reemplazo total de dispositivo|200, 400, 404|
||PATCH|`/devices/{id}`|Actualización parcial|200, 400, 404|
||DELETE|`/devices/{id}`|Eliminar dispositivo|204, 404|
||GET|`/devices/{id}/loans`|Historial de préstamos del dispositivo|200, 404|
|**Loans**|GET|`/loans`|Listar préstamos básicos|200|
||GET|`/loans/details`|Listar préstamos con detalle anidado (**JOIN**)|200|
||GET|`/loans/{id}`|Consultar préstamo por ID|200, 404|
||POST|`/loans`|Crear préstamo (valida disponibilidad)|201, 404, 409|
||PATCH|`/loans/{id}/return`|Devolver equipo y restaurar disponibilidad|200, 404, 409|

## 7. Evidencias de desarrollo

### Control de versiones con Alembic

- **Inicialización del entorno de migraciones:**
    
- **Script de migración generado:**
    
- **Aplicación de la migración al estado HEAD:**
    

### Inspección de la base de datos

- **Estructura de tablas en SQLite:**
    

### Interfaz Swagger UI

- **Vista general de módulos v4.0.0:**
    

### Pruebas funcionales de negocio

- **Creación exitosa de préstamo (HTTP 201):**
    
- **Control de concurrencia y disponibilidad (HTTP 409):**
    
- **Consulta relacional con Joins (`/loans/details`):**
    
- **Filtrado relacional por estado:**
    
- **Devolución de equipo y liberación de disponibilidad:**
    

## 8. Reflexión técnica

La evolución desde modelos aislados hacia un esquema relacional con Alembic y SQLAlchemy aporta mejoras arquitectónicas determinantes:

1. **Trazabilidad y reproducibilidad con Alembic:** Centralizar los cambios de base de datos en archivos versionados elimina la manipulación manual de esquemas y previene inconsistencias entre entornos de desarrollo y producción. Cada migración actúa como un commit de infraestructura.
    
2. **Integridad referencial en el motor de base de datos:** Definir restricciones `ForeignKey` a nivel de DDL previene la inserción de registros huérfanos y garantiza que las asociaciones entre usuarios, dispositivos y préstamos se cumplan de manera determinista.
    
3. **Optimización mediante Joins y esquemas anidados:** Resolver consultas multicapa en una única sentencia SQL mediante `join()` evita el antipatrón de consultas redundantes (_N+1 problem_), reduciendo la latencia de red y entregando cargas útiles enriquecidas mediante modelos Pydantic estructurados.