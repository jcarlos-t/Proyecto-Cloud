# Proyecto-Cloud

Web de Delibery, proyecto implementado para el curso de cloud computing.

## compose.yml

* Define **5 contenedores app** (ms1–ms5) con imágenes publicadas y **puertos externos**: 8001–8005.

  - **8001 → ms1-usuarios**
  - **8002 → ms2-productos**
  - **8003 → ms3-pedidos**
  - **8004 → ms4-orquestadorDelivery**
  - **8005 → ms5-data**

* Usa **red `backend` (bridge)** compartida entre servicios para *service discovery* por nombre.
* Variables de entorno parametrizadas via **`${DB_HOST}`** y **`${GLOBAL_CORS}`** (compatibles con `.env`).
* Dependencias externas de BD (MySQL:3307, PostgreSQL:5555, Mongo:27017) se consumen por hostname/IP privado del **host de BD**.
* **Ruteo interno** entre microservicios: p. ej. ms3 llama a `http://ms1-usuarios:8000` y `http://ms2-productos:8080`.
* Reinicio **`unless-stopped`** y mapeo 1:1 de puertos internos/externos por servicio.

---

## proyecto-cloud.yaml

Plantilla de **CloudFormation** que despliega la capa de cómputo y entrada para 5 microservicios (puertos **8001–8005**) con balanceo y seguridad básica:

* **Parámetros**: AMI, tipo/volumen de EC2, `KeyName`, `VpcId`, subredes (A/B) y `HealthCheckPath` para health checks HTTP.
* **Seguridad**:

  * `GSProdPC`: habilita **SSH (22)** y tráfico HTTP directo a **8001–8005** desde Internet.
  * `GSDatabase`: expone **3707 (MySQL)**, **5555 (PostgreSQL)** y **27017 (MongoDB)** **solo** desde `GSProdPC`; además 22 y 8080 abiertos.
* **Entrada (ALB)**: ALB público con **listeners 8001–8005**, cada uno enruta a su **Target Group** dedicado; health checks HTTP configurables (200–399).
* **Cómputo (EC2)**:

  * **Apps**: `EC2Prod1` y `EC2Prod2` (reciben tráfico del ALB en 8001–8005).
  * **BD**: `EC2Database` (acceso restringido desde Prod).
  * **Ingesta**: `EC2Ingesta` (procesos auxiliares).
* **Outputs**: **DNS del ALB**, IDs de instancias y **IP privada** de la instancia de BD (para configurar apps/compose).

En conjunto, define **separación por rol (apps/BD/ingesta)**, **balanceo por puerto por servicio**, y controles de **acceso L3/L4** mediante SGs, dejando la **escalabilidad horizontal** lista al añadir instancias a los Target Groups.


---

## Microservicio 1: Gestión de Usuarios

**Tecnología:** Python y MySQL
**Repositorio:** [https://github.com/PauloMiraBarr/ms1-usuarios](https://github.com/PauloMiraBarr/ms1-usuarios)

### Estructuras de tablas

**Usuarios**

* `id_usuario` INT PK AUTO_INCREMENT
* `nombre` VARCHAR(100)
* `correo` VARCHAR(100) UNIQUE
* `contraseña` VARCHAR(255)
* `telefono` VARCHAR(15)

**Direcciones**

* `id_direccion` INT PK AUTO_INCREMENT
* `id_usuario` INT FK → Usuarios(`id_usuario`) ON DELETE CASCADE
* `direccion` VARCHAR(255)
* `ciudad` VARCHAR(100)
* `codigo_postal` VARCHAR(10)

### Relaciones

* `Usuarios (1) ── (N) Direcciones` por `Direcciones.id_usuario` (FK, ON DELETE CASCADE).

### Endpoints (método + ruta)

* `GET /docs`
* `GET /health`
* `POST /login`
* `GET /all`
* `GET /usuarios/{id_usuario}`
* `POST /usuarios`
* `PUT /usuarios/{id_usuario}`
* `DELETE /usuarios/{id_usuario}`
* `GET /direcciones/{id_usuario}`
* `POST /direcciones`
* `PUT /direcciones/{id_direccion}`
* `DELETE /direcciones/{id_direccion}`

---

## Microservicio 2: Gestión de Productos

**Tecnología:** Java y PostgreSQL
**Repositorio:** [https://github.com/EV081/ms2_products.git](https://github.com/EV081/ms2_products.git)

### Estructuras de tablas

**Categorías**

* `id_categoria` INT PK AUTO_INCREMENT
* `nombre_categoria` VARCHAR(100)
* `descripcion_categoria` TEXT

**Productos**

* `id_producto` INT PK AUTO_INCREMENT
* `nombre` VARCHAR(100)
* `descripcion` TEXT
* `precio` DECIMAL(10,2)
* `categoria_id` INT FK → Categorías(`id_categoria`)

### Relaciones

* `Productos (N) ── (1) Categorías` vía `Productos.categoria_id`.

### Endpoints (método + ruta)

**Health y Docs**

* `GET /health`
* `GET /swagger-ui/index.html`

**Productos**

* `GET /productos`
* `GET /productos/{id_producto}`
* `POST /productos`
* `PUT /productos/{id_producto}`
* `DELETE /productos/{id_producto}`

**Categorías**

* `GET /categorias`
* `GET /categorias/{id_categoria}`
* `POST /categorias`
* `PUT /categorias/{id_categoria}`
* `DELETE /categorias/{id_categoria}`

**Productos por categoría**

* `GET /categorias/{id_categoria}/productos`

---

## Microservicio 3: Gestión de Pedidos

**Tecnología:** TypeScript y MongoDB
**Repositorio:** [https://github.com/jcarlos-t/Pedidos-MS3.git](https://github.com/jcarlos-t/Pedidos-MS3.git)

### Estructuras (colecciones MongoDB)

**Pedido**

* `_id` ObjectId
* `id_usuario` Number
* `fecha_pedido` Date (default: now)
* `estado` String ∈ {`pendiente`, `entregado`, `cancelado`}
* `total` Number
* `productos` [ { `id_producto` Number, `cantidad` Number, `precio_unitario` Number } ]

**HistorialPedido**

* `_id` ObjectId
* `id_pedido` ObjectId (ref: **Pedido**)
* `id_usuario` Number
* `fecha_evento` Date (default: now)
* `estado` String ∈ {`pendiente`, `entregado`, `cancelado`}
* `comentarios` String (opcional)

### Relaciones

* **HistorialPedido (N) ── (1) Pedido** vía `id_pedido`.
* **Pedido (N) ── (1) Usuario** (lógica por `id_usuario`, referencia externa a MS1).
* **Pedido.productos[*].id_producto** referencia externa a **Producto** (MS2).

### Endpoints

**Pedidos**

* `GET /pedidos/user/:id_usuario`  (query opcional `?estado=`)
* `GET /pedidos/:id_pedido`
* `POST /pedidos`
* `PUT /pedidos/:id_pedido/estado`
* `PUT /pedidos/:id_pedido`
* `DELETE /pedidos/:id_pedido`

**Historial**

* `GET /historial/:id_usuario`  (query opcional `?estado=`)
* `POST /pedidos/:id_pedido/historial`

**Swagger y Health**

* `GET /docs` (UI)
* `GET /health` (UI)

---

## Microservicio 4: Delivery

**Repositorio:** [https://github.com/J-D-Rosales/Microservicios_orquestador](https://github.com/J-D-Rosales/Microservicios_orquestador)

### Endpoints

* **POST /orq/cart/price-quote** — Calcula una cotización del carrito (subtotal, impuestos y total) validando usuario, dirección y precios en MS2.
* **GET /orq/orders/{order_id}/details?id_usuario=...** Trae el pedido (MS3) y verifica que pertenezca a id_usuario.
En cada línea, trae el producto actual (MS2) y marca si cambió el precio desde que se creó el pedido.
Mapea categoría (MS2) y agrega un resumen del usuario (MS1) incluyendo cantidad de direcciones.

---

## Microservicio 5: Data Science

**Repositorio:** [https://github.com/EV081/ms5.git](https://github.com/EV081/ms5.git)

### Endpoints (método + ruta)

* `GET /health`
* `GET /estado_historial/{id_usuario}`
* `GET /total_gastado/{id_usuario}?fecha_inicio=YYYY-MM-DD&fecha_fin=YYYY-MM-DD`
* `GET /ranking_categorias`


## Base de datos

**Repositorio:** [https://github.com/jcarlos-t/PCloud-BD](https://github.com/jcarlos-t/PCloud-BD)

Contiene el Docker Compose para desplegar las bases de datos del proyecto y los scripts necesarios para cargar datos de prueba (fake data) con fines prácticos.


## Frontend

**Repositorio:** [https://github.com/JosephAnderson234/cloud-frontend](https://github.com/JosephAnderson234/cloud-frontend)

Contiene el frontend necesario para interactuar con el backend de la solución informática propuesta. Se desarrolla con *React*.

**Formato de consumo del backend desde el frontend**

Se usa el formato `/api/ms{n}/endpoint`

Por ejemplo, los `/health` de cada ***ms***
```
  - /api/ms1/health
  - /api/ms2/health
  - /api/ms3/health
  - /api/ms4/health
  - /api/ms5/health
```

## Ingesta
**Repositorio:** [https://github.com/EV081/ingesta_mongo.git](https://github.com/EV081/ingesta_mongo.git)
**Repositorio:** [https://github.com/EV081/ingesta_mysql.git](https://github.com/EV081/ingesta_mysql.git)
**Repositorio:** [https://github.com/EV081/ingesta_postgres.git](https://github.com/EV081/ingesta_postgres.git)
