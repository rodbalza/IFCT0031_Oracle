# **Caso práctico (Opcional)**

## **Análisis de pedidos de una cafetería**

---

## **1. Contexto del caso**

La cafetería **Café Central** quiere empezar a analizar sus pedidos diarios usando **Oracle SQL Developer**.  Hasta ahora registraban la información en una hoja de cálculo, pero han decidido pasar los datos a una tabla en Oracle para poder responder preguntas como:

- qué productos se venden más;
- qué clientes compran con mayor frecuencia;
- qué pedidos tienen mayor importe;
- qué categorías generan más ingresos;
- qué pedidos pertenecen a determinados métodos de pago;
- qué ventas se realizaron en determinados rangos de fechas;
- qué clientes o productos cumplen ciertos patrones de búsqueda.

> Importante: todo el script, la creación de la tabla, la inserción de registros y las consultas deben realizarse únicamente en **Oracle SQL Developer**.
> 

---

## **2. Tabla de trabajo**

La empresa usará una única tabla llamada:

```sql
CAF_PEDIDOS
```

Esta tabla almacenará información de pedidos realizados por clientes de la cafetería.

---

## **3. Script de preparación en Oracle SQL Developer**

> Este script debe ejecutarse únicamente en una hoja de trabajo de **Oracle SQL Developer**.
> 

## **3.1. Eliminar la tabla si ya existe**

```sql
DROP TABLE CAF_PEDIDOS PURGE;
```

## **3.2. Crear la tabla**

```sql
CREATE TABLE CAF_PEDIDOS (
    id_pedido       NUMBER PRIMARY KEY,
    cliente         VARCHAR2(100) NOT NULL,
    email           VARCHAR2(120),
    ciudad          VARCHAR2(50),
    producto        VARCHAR2(100) NOT NULL,
    categoria       VARCHAR2(50) NOT NULL,
    cantidad        NUMBER CHECK (cantidad > 0),
    precio_unitario NUMBER(8,2) CHECK (precio_unitario >= 0),
    metodo_pago     VARCHAR2(30) CHECK (metodo_pago IN ('Tarjeta', 'Efectivo', 'Bizum', 'Transferencia')),
    estado          VARCHAR2(30) CHECK (estado IN ('Pendiente', 'Pagado', 'Cancelado', 'Entregado')),
    fecha_pedido    DATE NOT NULL
);
```

## **3.3. Insertar registros**

```sql
INSERT INTO CAF_PEDIDOS VALUES
(1, 'Ana López', 'ana.lopez@email.com', 'Madrid', 'Café latte', 'Bebida caliente', 2, 2.80, 'Tarjeta', 'Pagado', DATE '2026-01-10');

INSERT INTO CAF_PEDIDOS VALUES
(2, 'Luis García', 'luis.garcia@email.com', 'Valencia', 'Tostada integral', 'Comida', 1, 3.50, 'Efectivo', 'Pagado', DATE '2026-01-11');

INSERT INTO CAF_PEDIDOS VALUES
(3, 'María Torres', 'maria.torres@email.com', 'Madrid', 'Capuccino', 'Bebida caliente', 3, 2.60, 'Bizum', 'Entregado', DATE '2026-01-12');

INSERT INTO CAF_PEDIDOS VALUES
(4, 'Carlos Ruiz', 'carlos.ruiz@email.com', 'Bilbao', 'Zumo natural', 'Bebida fría', 2, 3.20, 'Tarjeta', 'Pagado', DATE '2026-01-13');

INSERT INTO CAF_PEDIDOS VALUES
(5, 'Laura Sánchez', 'laura.sanchez@email.com', 'Sevilla', 'Croissant', 'Bollería', 4, 1.80, 'Efectivo', 'Entregado', DATE '2026-01-14');

INSERT INTO CAF_PEDIDOS VALUES
(6, 'Pedro Martín', 'pedro.martin@email.com', 'Madrid', 'Café americano', 'Bebida caliente', 1, 2.20, 'Tarjeta', 'Pendiente', DATE '2026-01-15');

INSERT INTO CAF_PEDIDOS VALUES
(7, 'Lucía Gómez', 'lucia.gomez@email.com', 'Valencia', 'Ensalada', 'Comida', 2, 6.50, 'Bizum', 'Pagado', DATE '2026-01-16');

INSERT INTO CAF_PEDIDOS VALUES
(8, 'Javier Pérez', 'javier.perez@email.com', 'Bilbao', 'Té verde', 'Bebida caliente', 1, 2.00, 'Transferencia', 'Cancelado', DATE '2026-01-17');

INSERT INTO CAF_PEDIDOS VALUES
(9, 'Sofía Moreno', 'sofia.moreno@email.com', 'Madrid', 'Batido de fresa', 'Bebida fría', 2, 4.20, 'Tarjeta', 'Entregado', DATE '2026-01-18');

INSERT INTO CAF_PEDIDOS VALUES
(10, 'Diego Herrera', 'diego.herrera@email.com', 'Sevilla', 'Bocadillo vegetal', 'Comida', 1, 5.90, 'Efectivo', 'Pagado', DATE '2026-01-19');

INSERT INTO CAF_PEDIDOS VALUES
(11, 'Paula Jiménez', 'paula.jimenez@email.com', 'Madrid', 'Café latte', 'Bebida caliente', 2, 2.80, 'Bizum', 'Entregado', DATE '2026-01-20');

INSERT INTO CAF_PEDIDOS VALUES
(12, 'Raúl Navarro', 'raul.navarro@email.com', 'Valencia', 'Muffin chocolate', 'Bollería', 3, 2.10, 'Tarjeta', 'Pagado', DATE '2026-01-21');

INSERT INTO CAF_PEDIDOS VALUES
(13, 'Elena Castro', 'elena.castro@email.com', 'Bilbao', 'Zumo natural', 'Bebida fría', 1, 3.20, 'Efectivo', 'Pendiente', DATE '2026-01-22');

INSERT INTO CAF_PEDIDOS VALUES
(14, 'Andrés Molina', 'andres.molina@email.com', 'Madrid', 'Tostada integral', 'Comida', 2, 3.50, 'Transferencia', 'Pagado', DATE '2026-01-23');

INSERT INTO CAF_PEDIDOS VALUES
(15, 'Marta Romero', 'marta.romero@email.com', 'Sevilla', 'Capuccino', 'Bebida caliente', 2, 2.60, 'Tarjeta', 'Entregado', DATE '2026-01-24');

INSERT INTO CAF_PEDIDOS VALUES
(16, 'Hugo Ortega', 'hugo.ortega@email.com', 'Madrid', 'Croissant', 'Bollería', 5, 1.80, 'Bizum', 'Pagado', DATE '2026-01-25');

INSERT INTO CAF_PEDIDOS VALUES
(17, 'Nuria Vidal', 'nuria.vidal@email.com', 'Valencia', 'Café americano', 'Bebida caliente', 2, 2.20, 'Efectivo', 'Cancelado', DATE '2026-01-26');

INSERT INTO CAF_PEDIDOS VALUES
(18, 'Iván Ramos', 'ivan.ramos@email.com', 'Bilbao', 'Ensalada', 'Comida', 1, 6.50, 'Tarjeta', 'Entregado', DATE '2026-01-27');

INSERT INTO CAF_PEDIDOS VALUES
(19, 'Clara León', 'clara.leon@email.com', 'Madrid', 'Batido de mango', 'Bebida fría', 3, 4.50, 'Bizum', 'Pagado', DATE '2026-01-28');

INSERT INTO CAF_PEDIDOS VALUES
(20, 'Sergio Campos', 'sergio.campos@email.com', 'Sevilla', 'Té negro', 'Bebida caliente', 2, 2.10, 'Tarjeta', 'Pendiente', DATE '2026-01-29');

INSERT INTO CAF_PEDIDOS VALUES
(21, 'Beatriz Alonso', 'beatriz.alonso@email.com', 'Madrid', 'Brownie', 'Bollería', 2, 2.70, 'Efectivo', 'Pagado', DATE '2026-01-30');

INSERT INTO CAF_PEDIDOS VALUES
(22, 'Óscar Medina', 'oscar.medina@email.com', 'Valencia', 'Café solo', 'Bebida caliente', 1, 1.60, 'Tarjeta', 'Entregado', DATE '2026-01-31');

INSERT INTO CAF_PEDIDOS VALUES
(23, 'Irene Castillo', 'irene.castillo@email.com', 'Bilbao', 'Sándwich mixto', 'Comida', 2, 4.80, 'Bizum', 'Pagado', DATE '2026-02-01');

INSERT INTO CAF_PEDIDOS VALUES
(24, 'Adrián Vega', 'adrian.vega@email.com', 'Sevilla', 'Agua mineral', 'Bebida fría', 3, 1.30, 'Efectivo', 'Entregado', DATE '2026-02-02');

INSERT INTO CAF_PEDIDOS VALUES
(25, 'Natalia Reyes', 'natalia.reyes@email.com', 'Madrid', 'Café con leche', 'Bebida caliente', 2, 2.40, 'Transferencia', 'Pagado', DATE '2026-02-03');

INSERT INTO CAF_PEDIDOS VALUES
(26, 'Rubén Santos', 'ruben.santos@email.com', 'Valencia', 'Napolitana', 'Bollería', 4, 1.90, 'Tarjeta', 'Entregado', DATE '2026-02-04');

INSERT INTO CAF_PEDIDOS VALUES
(27, 'Patricia Gil', 'patricia.gil@email.com', 'Bilbao', 'Chocolate caliente', 'Bebida caliente', 1, 3.10, 'Bizum', 'Pendiente', DATE '2026-02-05');

INSERT INTO CAF_PEDIDOS VALUES
(28, 'Daniel Mora', 'daniel.mora@email.com', 'Madrid', 'Wrap de pollo', 'Comida', 2, 6.20, 'Tarjeta', 'Pagado', DATE '2026-02-06');

INSERT INTO CAF_PEDIDOS VALUES
(29, 'Carmen Prieto', 'carmen.prieto@email.com', 'Sevilla', 'Limonada', 'Bebida fría', 2, 3.00, 'Efectivo', 'Cancelado', DATE '2026-02-07');

INSERT INTO CAF_PEDIDOS VALUES
(30, 'Víctor Cano', 'victor.cano@email.com', 'Valencia', 'Café latte', 'Bebida caliente', 3, 2.80, 'Bizum', 'Entregado', DATE '2026-02-08');

INSERT INTO CAF_PEDIDOS VALUES
(31, 'Rocío Ibáñez', 'rocio.ibanez@email.com', 'Madrid', 'Magdalena', 'Bollería', 6, 1.20, 'Tarjeta', 'Pagado', DATE '2026-02-09');

INSERT INTO CAF_PEDIDOS VALUES
(32, 'Alberto Fuentes', 'alberto.fuentes@email.com', 'Bilbao', 'Bocadillo de tortilla', 'Comida', 1, 5.50, 'Transferencia', 'Pagado', DATE '2026-02-10');

INSERT INTO CAF_PEDIDOS VALUES
(33, 'Sara Núñez', 'sara.nunez@email.com', 'Sevilla', 'Smoothie tropical', 'Bebida fría', 2, 4.80, 'Bizum', 'Entregado', DATE '2026-02-11');

INSERT INTO CAF_PEDIDOS VALUES
(34, 'Miguel Domínguez', 'miguel.dominguez@email.com', 'Madrid', 'Té chai', 'Bebida caliente', 2, 2.70, 'Efectivo', 'Pagado', DATE '2026-02-12');

INSERT INTO CAF_PEDIDOS VALUES
(35, 'Teresa Lozano', 'teresa.lozano@email.com', 'Valencia', 'Empanada', 'Comida', 3, 3.90, 'Tarjeta', 'Entregado', DATE '2026-02-13');

INSERT INTO CAF_PEDIDOS VALUES
(36, 'Pablo Serrano', 'pablo.serrano@email.com', 'Bilbao', 'Café mocca', 'Bebida caliente', 1, 3.00, 'Bizum', 'Pendiente', DATE '2026-02-14');

INSERT INTO CAF_PEDIDOS VALUES
(37, 'Alicia Flores', 'alicia.flores@email.com', 'Madrid', 'Cookie avena', 'Bollería', 4, 1.60, 'Efectivo', 'Pagado', DATE '2026-02-15');

INSERT INTO CAF_PEDIDOS VALUES
(38, 'Mario Rivas', 'mario.rivas@email.com', 'Sevilla', 'Gazpacho', 'Comida', 2, 4.30, 'Tarjeta', 'Entregado', DATE '2026-02-16');

INSERT INTO CAF_PEDIDOS VALUES
(39, 'Cristina Aguilar', 'cristina.aguilar@email.com', 'Valencia', 'Refresco cola', 'Bebida fría', 3, 2.20, 'Bizum', 'Pagado', DATE '2026-02-17');

INSERT INTO CAF_PEDIDOS VALUES
(40, 'Fernando Peña', 'fernando.pena@email.com', 'Madrid', 'Café cortado', 'Bebida caliente', 2, 1.90, 'Efectivo', 'Entregado', DATE '2026-02-18');

INSERT INTO CAF_PEDIDOS VALUES
(41, 'Noelia Márquez', 'noelia.marquez@email.com', 'Bilbao', 'Tarta de queso', 'Bollería', 2, 3.80, 'Tarjeta', 'Pagado', DATE '2026-02-19');

INSERT INTO CAF_PEDIDOS VALUES
(42, 'Álvaro Cabrera', 'alvaro.cabrera@email.com', 'Sevilla', 'Café americano', 'Bebida caliente', 1, 2.20, 'Bizum', 'Cancelado', DATE '2026-02-20');

INSERT INTO CAF_PEDIDOS VALUES
(43, 'Silvia Montero', 'silvia.montero@email.com', 'Madrid', 'Hamburguesa vegetal', 'Comida', 2, 7.20, 'Tarjeta', 'Pagado', DATE '2026-02-21');

INSERT INTO CAF_PEDIDOS VALUES
(44, 'Manuel Bravo', 'manuel.bravo@email.com', 'Valencia', 'Zumo natural', 'Bebida fría', 4, 3.20, 'Efectivo', 'Entregado', DATE '2026-02-22');

INSERT INTO CAF_PEDIDOS VALUES
(45, 'Eva Pardo', 'eva.pardo@email.com', 'Bilbao', 'Café latte', 'Bebida caliente', 2, 2.80, 'Transferencia', 'Pagado', DATE '2026-02-23');

INSERT INTO CAF_PEDIDOS VALUES
(46, 'Jorge Esteban', 'jorge.esteban@email.com', 'Madrid', 'Palmera chocolate', 'Bollería', 3, 2.40, 'Bizum', 'Entregado', DATE '2026-02-24');

INSERT INTO CAF_PEDIDOS VALUES
(47, 'Lorena Arias', 'lorena.arias@email.com', 'Sevilla', 'Té verde', 'Bebida caliente', 2, 2.00, 'Tarjeta', 'Pagado', DATE '2026-02-25');

INSERT INTO CAF_PEDIDOS VALUES
(48, 'Guillermo Soler', 'guillermo.soler@email.com', 'Valencia', 'Sándwich vegetal', 'Comida', 1, 4.90, 'Efectivo', 'Pendiente', DATE '2026-02-26');

INSERT INTO CAF_PEDIDOS VALUES
(49, 'Isabel Carmona', 'isabel.carmona@email.com', 'Madrid', 'Batido de vainilla', 'Bebida fría', 2, 4.10, 'Tarjeta', 'Entregado', DATE '2026-02-27');

INSERT INTO CAF_PEDIDOS VALUES
(50, 'Tomás Rubio', 'tomas.rubio@email.com', 'Bilbao', 'Café solo', 'Bebida caliente', 3, 1.60, 'Bizum', 'Pagado', DATE '2026-02-28');

COMMIT;
```

---

## **4. Comprobación inicial en Oracle SQL Developer**

Antes de resolver las preguntas, comprueba que la tabla tiene datos.

```sql
SELECT *
FROM CAF_PEDIDOS;
```

```sql
SELECT COUNT(*) AS total_pedidos
FROM CAF_PEDIDOS;
```

Resultado esperado:

```
TOTAL_PEDIDOS
-------------
50
```

---

## **5. Ejercicios propuestos**

## **Bloque A. Operadores lógicos y filtros**

1. Mostrar todos los pedidos realizados en la ciudad de `Madrid`.
2. Mostrar los pedidos de `Madrid` cuyo estado sea `Pagado`.
3. Mostrar los pedidos realizados en `Madrid` o `Valencia`.
4. Mostrar los pedidos que no sean de la ciudad de `Madrid`.
5. Mostrar los pedidos cuyo método de pago sea `Tarjeta` o `Bizum`.
6. Mostrar los pedidos cuyo estado no sea `Cancelado`.
7. Mostrar los pedidos de la categoría `Bebida caliente` que estén pagados o entregados.
8. Mostrar los pedidos de `Madrid` cuya categoría sea `Bebida caliente` o `Bebida fría`.
    
    La consulta debe usar paréntesis.
    
9. Mostrar los pedidos cuyo estado sea `Pagado` y cuyo método de pago sea `Tarjeta` o `Bizum`.
    
    La consulta debe usar paréntesis.
    
10. Mostrar los pedidos que sean de `Madrid` o `Bilbao`, pero que no estén cancelados.

---

## **Bloque B. Filtros con `LIKE`, `IN`, `NOT IN` y `BETWEEN`**

1. Mostrar los clientes cuyo nombre empiece por la letra `A`.
2. Mostrar los clientes cuyo email contenga la palabra `garcia`.
3. Mostrar los productos que contengan la palabra `Café`.
4. Mostrar los pedidos cuya ciudad esté dentro de esta lista: `Madrid`, `Valencia`, `Bilbao`.
5. Mostrar los pedidos cuya ciudad no esté dentro de esta lista: `Madrid`, `Valencia`.
6. Mostrar los pedidos cuya cantidad esté entre 2 y 4 unidades.
7. Mostrar los pedidos cuyo precio unitario esté entre 2 y 4 euros.
8. Mostrar los pedidos realizados entre el `15/01/2026` y el `25/01/2026`.
9. Mostrar los pedidos de productos que terminen con la letra `l`.
10. Mostrar los pedidos cuyo producto contenga la letra `a`, sin importar si está en mayúscula o minúscula.

---

## **Bloque C. Funciones de texto**

1. Mostrar el nombre del cliente en mayúsculas usando `UPPER`.
2. Mostrar el nombre del cliente en minúsculas usando `LOWER`.
3. Mostrar el nombre del cliente con formato de iniciales en mayúscula usando `INITCAP`.
4. Mostrar el nombre del cliente y la cantidad de caracteres que tiene.
5. Mostrar el email de cada cliente y la posición donde aparece el símbolo `@`.
6. Mostrar los primeros 5 caracteres del nombre de cada producto.
7. Mostrar el dominio del correo electrónico de cada cliente usando `SUBSTR` e `INSTR`.
8. Buscar todos los clientes cuyo nombre contenga `ana`, sin importar mayúsculas o minúsculas.
9. Mostrar el producto, la categoría y la longitud del nombre del producto.
10. Mostrar los emails en minúsculas y comprobar visualmente si todos tienen el mismo dominio.

---

## **Bloque D. Funciones numéricas**

1. Mostrar el producto, la cantidad, el precio unitario y el importe total del pedido.El importe total se calcula como:

```
cantidad * precio_unitario
```

1. Mostrar el importe total de cada pedido redondeado a 2 decimales con `ROUND`.
2. Mostrar el importe total de cada pedido truncado a 1 decimal con `TRUNC`.
3. Calcular el precio con IVA del 21% para cada producto.
4. Mostrar el precio con IVA redondeado a 2 decimales.
5. Mostrar los pedidos cuyo `id_pedido` sea par usando `MOD`.
6. Mostrar los pedidos cuyo `id_pedido` sea impar usando `MOD`.
7. Mostrar el producto, precio unitario y diferencia entre el precio con IVA y el precio sin IVA.

---

## **Bloque E. Funciones de fecha**

1. Mostrar la fecha actual del sistema usando `SYSDATE`.
2. Mostrar cada pedido con una nueva columna llamada `fecha_entrega_estimada`, sumando 3 días a `fecha_pedido`.
3. Mostrar cada pedido con una fecha de revisión calculada un mes después de `fecha_pedido` usando `ADD_MONTHS`.
4. Mostrar cuántos meses han pasado desde la fecha del pedido hasta la fecha actual usando `MONTHS_BETWEEN`.
5. Mostrar el año de cada pedido usando `EXTRACT`.
6. Mostrar el mes de cada pedido usando `EXTRACT`.
7. Mostrar los pedidos realizados en enero de 2026.
8. Mostrar los pedidos cuya fecha sea posterior al `20/01/2026`.

---

## **Bloque F. Funciones de agregación**

1. Contar cuántos pedidos hay en total.
2. Contar cuántos pedidos tienen estado `Pagado`.
3. Calcular la cantidad total de productos vendidos.
4. Calcular el importe total vendido.
    
    Recuerda que el importe de cada pedido se calcula como:
    

```
cantidad * precio_unitario
```

1. Calcular el precio unitario mínimo registrado.
2. Calcular el precio unitario máximo registrado.
3. Calcular el precio unitario medio de todos los productos.
4. Calcular el importe medio por pedido.
5. Calcular cuántos pedidos hay por cada ciudad.
6. Calcular cuántos pedidos hay por cada categoría.

---

## **Bloque G. Agrupaciones con `GROUP BY`**

1. Mostrar el total vendido por cada ciudad.
2. Mostrar el total vendido por cada categoría.
3. Mostrar el número de pedidos por cada método de pago.
4. Mostrar la cantidad total de productos vendidos por cada producto.
5. Mostrar el precio medio por categoría.
6. Mostrar la cantidad mínima, máxima y media vendida por categoría.
7. Mostrar el total vendido por estado del pedido.
8. Mostrar el número de pedidos por ciudad y estado.
9. Mostrar el total vendido por método de pago.
10. Mostrar el total de unidades vendidas por ciudad.

---

## **Bloque H. Filtrado de grupos con `HAVING`**

1. Mostrar solo las ciudades que tengan más de 3 pedidos.
2. Mostrar solo las categorías cuyo total vendido sea mayor que 20 euros.
3. Mostrar solo los métodos de pago que se hayan usado más de 4 veces.
4. Mostrar solo los productos cuya cantidad total vendida sea mayor o igual que 4.
5. Mostrar solo las categorías cuyo precio medio sea superior a 3 euros.
6. Mostrar solo los estados de pedido cuyo importe total sea mayor que 15 euros.

---

## **6. Preguntas para informe**

La dirección de **Café Central** quiere un pequeño informe SQL para conocer el comportamiento de ventas.

Debes crear consultas en **Oracle SQL Developer** que respondan a estas preguntas:

1. ¿Qué ciudad ha generado más ventas?
2. ¿Qué categoría tiene mayor importe total vendido?
3. ¿Qué método de pago se usa con más frecuencia?
4. ¿Qué productos han vendido más de 3 unidades en total?
5. ¿Qué pedidos pagados o entregados superan los 6 euros?
6. ¿Qué clientes de Madrid o Valencia compraron productos de bebida caliente?
7. ¿Qué pedidos no cancelados se realizaron entre el 15 y el 25 de enero de 2026?
8. ¿Qué categorías tienen un precio medio superior a 3 euros?
9. ¿Qué productos contienen la palabra `Café` o `Té`?
10. ¿Qué clientes tienen un email cuyo nombre de usuario contiene un punto?

---

# **7. Entregables**

Debes entregar:

1. Un archivo `.sql` con todas las consultas realizadas.