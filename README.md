# Proyecto-Cloud

Web de Delibery, proyecto implementado para el curso de cloud computing.

## Compose

Con el *compose* los microservicios se ejecutan en los puertos:

* **8001 → ms1-usuarios**
* **8002 → ms2-productos**
* **8003 → ms3-pedidos**
* **8004 → ms4-orquestadorDelivery**
* **8005 → ms5-data**

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

* `GET /api-docs` (UI)
* `GET /health` (UI)

---

## Microservicio 4: Delivery

**Repositorio:** [https://github.com/J-D-Rosales/Microservicios_orquestador.git](https://github.com/J-D-Rosales/Microservicios_orquestador.git)

### Endpoints

* **POST /orq/cart/price-quote** — Calcula una cotización del carrito (subtotal, impuestos y total) validando usuario, dirección y precios en MS2.
* **POST /orq/orders** — Crea un pedido en MS3 con estado *pendiente*, registra historial (si existe) e incluye totales.
* **PUT /orq/orders/{order_id}/cancel** — Cancela un pedido del usuario (verifica propiedad) y anota el historial si está disponible.
* **GET /orq/_debug/addresses/{id_usuario}** — Muestra las direcciones crudas de MS1 y su versión normalizada para depuración.

---

## Microservicio 5: Data Science

**Repositorio:** [https://github.com/EV081/ms5.git](https://github.com/EV081/ms5.git)

### Endpoints (método + ruta)

* `GET /health`
* `GET /estado_historial/{id_usuario}`
* `GET /total_gastado/{id_usuario}?fecha_inicio=YYYY-MM-DD&fecha_fin=YYYY-MM-DD`
* `GET /ranking_categorias`
