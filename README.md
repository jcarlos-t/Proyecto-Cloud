# Proyecto-Cloud
Gestión de cursos, proyecto implementado para el curso de cloud computing

### **Microservicio 1: Gestión de Usuarios**

#### **Relaciones entre tablas**:

1. **Usuarios** → **Direcciones**: Relación 1:N. Un usuario puede tener varias direcciones asociadas.

#### **Tablas y relaciones**:

* **Usuarios** (1:N con Direcciones)

  * `id_usuario` (PK, INT, AUTO_INCREMENT)
  * `nombre` (VARCHAR(100))
  * `correo` (VARCHAR(100), UNIQUE)
  * `contraseña` (VARCHAR(255))
  * `telefono` (VARCHAR(15))

* **Direcciones**

  * `id_direccion` (PK, INT, AUTO_INCREMENT)
  * `id_usuario` (FK a Usuarios, INT)
  * `direccion` (VARCHAR(255))
  * `ciudad` (VARCHAR(100))
  * `codigo_postal` (VARCHAR(10))

#### **Endpoints**:

1. **Usuarios:**

   * `GET /usuarios/{id_usuario}`: Obtener detalles de un usuario.
   * `POST /usuarios`: Crear un nuevo usuario.
   * `PUT /usuarios/{id_usuario}`: Actualizar detalles de un usuario.
   * `DELETE /usuarios/{id_usuario}`: Eliminar un usuario.
2. **Direcciones:**

   * `GET /direcciones/{id_usuario}`: Obtener todas las direcciones de un usuario.
   * `POST /direcciones`: Agregar una nueva dirección.
   * `PUT /direcciones/{id_direccion}`: Actualizar una dirección existente.
   * `DELETE /direcciones/{id_direccion}`: Eliminar una dirección.

---

### **Microservicio 2: Gestión de Productos**

#### **Relaciones entre tablas**:

1. **Productos** → **Categorías**: Relación 1:N. Un producto pertenece a una categoría.

#### **Tablas y relaciones**:

* **Productos** (1:N con Categorías)

* **Productos** (1:N con Categorías)

  * `id_producto` (PK, INT, AUTO_INCREMENT)
  * `nombre` (VARCHAR(100))
  * `descripcion` (TEXT)
  * `precio` (DECIMAL(10, 2))
  * `categoria_id` (FK a Categorías, INT)

* **Categorías**

  * `id_categoria` (PK, INT, AUTO_INCREMENT)
  * `nombre_categoria` (VARCHAR(100))
  * `descripcion_categoria` (TEXT)


#### **Endpoints**:

1. **Productos:**

   * `GET /productos`: Obtener todos los productos.
   * `GET /productos/{id_producto}`: Obtener detalles de un producto.
   * `POST /productos`: Crear un nuevo producto.
   * `PUT /productos/{id_producto}`: Actualizar un producto.
   * `DELETE /productos/{id_producto}`: Eliminar un producto.
2. **Categorías:**

   * `GET /categorias`: Obtener todas las categorías.
   * `GET /categorias/{id_categoria}`: Obtener detalles de una categoría.
   * `POST /categorias`: Crear una nueva categoría.
   * `PUT /categorias/{id_categoria}`: Actualizar una categoría.
   * `DELETE /categorias/{id_categoria}`: Eliminar una categoría.

---

### **Microservicio 3: Gestión de Pedidos**

https://github.com/jcarlos-t/Pedidos-MS3.git


#### **Relaciones entre colecciones**:

1. **Pedidos** → **Usuarios**: Relación 1:N. Un pedido pertenece a un usuario.
2. **Pedidos** → **Productos**: Relación N:M. Un pedido puede incluir múltiples productos y un producto puede estar en muchos pedidos. Esto se maneja con un array de productos dentro de cada pedido en la base de datos no relacional.

#### **Colecciones**:
 
* **Pedidos**

  * `_id` (ID de pedido, ObjectId)
  * `id_usuario` (Referencia al usuario que hizo el pedido, INT)
  * `fecha_pedido` (ISODate)
  * `estado` (STRING, Ejemplo: "pendiente", "entregado", "cancelado")
  * `total` (DECIMAL(10, 2))
  * `productos` (Array de objetos, cada objeto con `id_producto` (INT), `cantidad` (INT), `precio_unitario` (DECIMAL(10, 2)))

* **Historial de Pedidos**

  * `_id` (ID de historial, ObjectId)
  * `id_pedido` (Referencia al pedido, ObjectId)
  * `fecha_entrega` (ISODate)
  * `estado` (STRING)
  * `comentarios` (TEXT)

#### **Endpoints**:

1. **Pedidos:**

   * `GET /pedidos/{id_usuario}`: Obtener todos los pedidos de un usuario.
   * `GET /pedidos/{id_pedido}`: Obtener detalles de un pedido específico.
   * `POST /pedidos`: Crear un nuevo pedido.
   * `PUT /pedidos/{id_pedido}`: Actualizar el estado o detalles de un pedido.
   * `DELETE /pedidos/{id_pedido}`: Cancelar o eliminar un pedido.

2. **Historial de Pedidos:**

   * `GET /historial/{id_usuario}`: Obtener historial de pedidos de un usuario.
   * `POST /historial`: Registrar un cambio de estado o entrega en el historial de un pedido.
