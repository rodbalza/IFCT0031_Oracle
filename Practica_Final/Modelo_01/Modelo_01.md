# Modelo 1

# **Examen práctico de Oracle SQL**

## **Caso práctico: Consultas de negocio y gestión básica de datos**

Apellidos:

Nombre:

Fecha:

Firma. 

**Herramienta de trabajo:** Oracle SQL Developer **24.3.1.347**

**Formato de entrega:** un único archivo SQL

**Nombre obligatorio del archivo:** `NombreApellido.sql`

**Puntuación total:** 10 puntos. Tendrás 20 preguntas disponibles para responder 10. Puntuación 1 punto c/u.

**Modelo de examen a escoger**: El de tu preferencia

Son dos modelos de examen: Modelo 1 y Modelo 2 de modo que el estudiante tiene la libertad de escoger el que quiera.  

---

# **1. Instrucciones generales**

1. Todo el examen debe realizarse exclusivamente con **Oracle SQL Developer 24.3.1.347**.
2. El estudiante entregará únicamente un archivo llamado: **NombreApellido.sql.** Ejemplo: **AnaLopez.sql**
3. El archivo debe comenzar con el siguiente encabezado:
    
    ```sql
    -- EXAMEN PRÁCTICO DE ORACLE SQL
    -- Nombre y apellidos:
    -- Fecha:
    -- Archivo: NombreApellido.sql
    ```
    
4. Cada respuesta debe estar identificada mediante comentarios:
    
    ```sql
    -- =====================================================
    -- PARTE 1 - PREGUNTA 1
    -- =====================================================
    
    -- Consulta SQL del estudiante
    ```
    
5. Cada pregunta debe resolverse por separado.
6. En las consultas con relaciones se utilizarán alias sencillos, por ejemplo `c` para clientes y `v` para ventas.
7. Los `INNER JOIN`, `LEFT JOIN` y `RIGHT JOIN` de este examen relacionan únicamente **dos tablas**.
8. No se deben modificar los datos de la Parte 1, salvo cuando una pregunta lo solicite expresamente.
9. En la Parte 2, las restricciones deben tener nombres claros, por ejemplo:
    
    ```
    pk_propietarios
    fk_mascotas_propietarios
    uk_propietarios_dni
    ck_mascotas_especie
    ```
    
10. Los cambios realizados con `INSERT`, `UPDATE` y `DELETE` deben confirmarse con `COMMIT` cuando se indique.
11. No se entregarán capturas, documentos PDF ni archivos adicionales.
12. Los scripts de preparación incluidos en este documento no forman parte de las respuestas del estudiante.

---

# **2. Parte 1 — Análisis comercial de NovaMarket**

## **2.1. Escenario de negocio**

NovaMarket es una empresa que vende productos en varias sucursales. La dirección quiere conocer mejor a sus clientes, revisar las ventas y comprobar cómo está organizado el equipo de empleados.

La base de datos contiene información sobre:

- sucursales;
- empleados y supervisores;
- clientes;
- ventas.

Debes responder a diez preguntas de negocio mediante consultas SQL.

---

## **2.2. Preparación del usuario**

**Desde el usuario** `SYSTEM` o `SYS`

```sql
CREATE USER examen_p1 IDENTIFIED BY "ExamenP1_2026"
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp
    QUOTA 30M ON users;

GRANT CREATE SESSION, CREATE TABLE TO examen_p1;
```

### **Datos orientativos de la conexión en SQL Developer**

| **Campo** | **Valor** |
| --- | --- |
| Nombre de conexión | `EXAMEN_PARTE_1` |
| Usuario | `EXAMEN_P1` |
| Contraseña | `ExamenP1_2026` |
| Tipo de conexión | Básica |
| Host | localhost |
| Puerto | `1521` |
| Tipo | Nombre de servicio |
| Servicio | `FREEPDB1`,  |

---

## **2.3. Script de creación de tablas y datos**

Después de crear la conexión `EXAMEN_PARTE_1`, este script debe ejecutarse conectado como `EXAMEN_P1`.

```sql
-- =====================================================
-- TABLA: SUCURSALES
-- =====================================================

CREATE TABLE sucursales (
    id_sucursal  NUMBER(3),
    nombre       VARCHAR2(80) NOT NULL,
    ciudad       VARCHAR2(60) NOT NULL,
    activa       CHAR(1) DEFAULT 'S' NOT NULL,

    CONSTRAINT pk_sucursales
        PRIMARY KEY (id_sucursal),

    CONSTRAINT uk_sucursales_nombre
        UNIQUE (nombre),

    CONSTRAINT ck_sucursales_activa
        CHECK (activa IN ('S', 'N'))
);

-- =====================================================
-- TABLA: EMPLEADOS
-- =====================================================

CREATE TABLE empleados (
    id_empleado    NUMBER(4),
    nombre         VARCHAR2(80) NOT NULL,
    puesto         VARCHAR2(60) NOT NULL,
    salario        NUMBER(10,2) NOT NULL,
    id_sucursal    NUMBER(3),
    id_supervisor  NUMBER(4),

    CONSTRAINT pk_empleados
        PRIMARY KEY (id_empleado),

    CONSTRAINT ck_empleados_salario
        CHECK (salario >= 0),

    CONSTRAINT fk_empleados_sucursal
        FOREIGN KEY (id_sucursal)
        REFERENCES sucursales (id_sucursal),

    CONSTRAINT fk_empleados_supervisor
        FOREIGN KEY (id_supervisor)
        REFERENCES empleados (id_empleado)
);

-- =====================================================
-- TABLA: CLIENTES
-- =====================================================

CREATE TABLE clientes (
    id_cliente      NUMBER(5),
    nombre          VARCHAR2(80) NOT NULL,
    email           VARCHAR2(120) NOT NULL,
    ciudad          VARCHAR2(60) NOT NULL,
    fecha_registro  DATE NOT NULL,
    segmento        VARCHAR2(15) NOT NULL,

    CONSTRAINT pk_clientes
        PRIMARY KEY (id_cliente),

    CONSTRAINT uk_clientes_email
        UNIQUE (email),

    CONSTRAINT ck_clientes_segmento
        CHECK (segmento IN ('BASICO', 'PLUS', 'PREMIUM'))
);

-- =====================================================
-- TABLA: VENTAS
-- =====================================================

CREATE TABLE ventas (
    id_venta     NUMBER(6),
    fecha_venta  DATE NOT NULL,
    id_cliente   NUMBER(5) NOT NULL,
    id_empleado  NUMBER(4) NOT NULL,
    canal        VARCHAR2(15) NOT NULL,
    importe      NUMBER(10,2) NOT NULL,
    descuento    NUMBER(10,2) DEFAULT 0 NOT NULL,
    estado       VARCHAR2(15) NOT NULL,

    CONSTRAINT pk_ventas
        PRIMARY KEY (id_venta),

    CONSTRAINT ck_ventas_canal
        CHECK (canal IN ('TIENDA', 'WEB', 'TELEFONO')),

    CONSTRAINT ck_ventas_importe
        CHECK (importe > 0),

    CONSTRAINT ck_ventas_descuento
        CHECK (descuento >= 0 AND descuento <= importe),

    CONSTRAINT ck_ventas_estado
        CHECK (estado IN ('COMPLETADA', 'PENDIENTE', 'CANCELADA')),

    CONSTRAINT fk_ventas_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES clientes (id_cliente),

    CONSTRAINT fk_ventas_empleado
        FOREIGN KEY (id_empleado)
        REFERENCES empleados (id_empleado)
);

-- =====================================================
-- TABLA AUXILIAR PARA LA PREGUNTA 10
-- =====================================================

CREATE TABLE revision_cierre (
    id_revision  NUMBER(3) PRIMARY KEY,
    observacion  VARCHAR2(200) NOT NULL
);

-- =====================================================
-- REGISTROS: SUCURSALES
-- =====================================================

INSERT INTO sucursales VALUES (1, 'NovaMarket Centro', 'Madrid', 'S');
INSERT INTO sucursales VALUES (2, 'NovaMarket Norte', 'Bilbao', 'S');
INSERT INTO sucursales VALUES (3, 'NovaMarket Este', 'Valencia', 'S');
INSERT INTO sucursales VALUES (4, 'NovaMarket Sur', 'Sevilla', 'S');
INSERT INTO sucursales VALUES (5, 'NovaMarket Oeste', 'Salamanca', 'N');

-- =====================================================
-- REGISTROS: EMPLEADOS
-- Los supervisores se insertan antes que sus subordinados.
-- =====================================================

INSERT INTO empleados VALUES
(1, 'Laura Gómez', 'DIRECTORA COMERCIAL', 52000, 1, NULL);

INSERT INTO empleados VALUES
(2, 'Diego Martín', 'JEFE DE VENTAS', 38000, 1, 1);

INSERT INTO empleados VALUES
(3, 'Marta Ruiz', 'VENDEDORA', 26500, 1, 2);

INSERT INTO empleados VALUES
(4, 'Carlos Vega', 'JEFE DE VENTAS', 37000, 2, 1);

INSERT INTO empleados VALUES
(5, 'Ana Torres', 'VENDEDORA', 25800, 2, 4);

INSERT INTO empleados VALUES
(6, 'Pablo Gil', 'JEFE DE VENTAS', 36500, 3, 1);

INSERT INTO empleados VALUES
(7, 'Lucía Navarro', 'VENDEDORA', 25200, 3, 6);

INSERT INTO empleados VALUES
(8, 'Sergio León', 'JEFE DE VENTAS', 36000, 4, 1);

-- =====================================================
-- REGISTROS: CLIENTES
-- =====================================================

INSERT INTO clientes VALUES
(101, 'María López', 'maria.lopez@email.es', 'Madrid', DATE '2025-01-15', 'PREMIUM');

INSERT INTO clientes VALUES
(102, 'Miguel Santos', 'miguel.santos@email.es', 'Bilbao', DATE '2025-02-11', 'PLUS');

INSERT INTO clientes VALUES
(103, 'Mónica Herrera', 'monica.herrera@email.es', 'Valencia', DATE '2025-03-20', 'BASICO');

INSERT INTO clientes VALUES
(104, 'Javier Castro', 'javier.castro@email.es', 'Sevilla', DATE '2025-04-03', 'PLUS');

INSERT INTO clientes VALUES
(105, 'Marta Molina', 'marta.molina@email.es', 'Madrid', DATE '2025-05-18', 'PREMIUM');

INSERT INTO clientes VALUES
(106, 'Pedro Alonso', 'pedro.alonso@email.es', 'Salamanca', DATE '2025-06-09', 'BASICO');

INSERT INTO clientes VALUES
(107, 'Lucía Ramos', 'lucia.ramos@email.es', 'Valencia', DATE '2025-07-25', 'PLUS');

INSERT INTO clientes VALUES
(108, 'Manuel Ortega', 'manuel.ortega@email.es', 'Bilbao', DATE '2025-08-12', 'PREMIUM');

INSERT INTO clientes VALUES
(109, 'Sara Núñez', 'sara.nunez@email.es', 'Madrid', DATE '2025-09-07', 'BASICO');

INSERT INTO clientes VALUES
(110, 'Eva Campos', 'eva.campos@email.es', 'Toledo', DATE '2025-10-08', 'PLUS');

-- =====================================================
-- REGISTROS: VENTAS
-- El cliente 110 no tiene ventas para comprobar el LEFT JOIN.
-- =====================================================

INSERT INTO ventas VALUES
(1001, DATE '2026-01-08', 101, 3, 'TIENDA', 850.00, 50.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1002, DATE '2026-01-12', 102, 5, 'WEB', 420.00, 20.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1003, DATE '2026-01-18', 103, 7, 'TIENDA', 190.00, 0.00, 'PENDIENTE');

INSERT INTO ventas VALUES
(1004, DATE '2026-01-25', 104, 3, 'TELEFONO', 630.00, 30.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1005, DATE '2026-02-02', 105, 3, 'WEB', 1200.00, 100.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1006, DATE '2026-02-07', 106, 5, 'TIENDA', 275.00, 25.00, 'CANCELADA');

INSERT INTO ventas VALUES
(1007, DATE '2026-02-14', 107, 7, 'WEB', 510.00, 10.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1008, DATE '2026-02-20', 108, 5, 'TIENDA', 980.00, 80.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1009, DATE '2026-02-28', 109, 3, 'TELEFONO', 340.00, 0.00, 'PENDIENTE');

INSERT INTO ventas VALUES
(1010, DATE '2026-03-04', 101, 3, 'WEB', 760.00, 60.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1011, DATE '2026-03-09', 102, 5, 'TELEFONO', 310.00, 10.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1012, DATE '2026-03-13', 103, 7, 'WEB', 680.00, 30.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1013, DATE '2026-03-18', 104, 3, 'TIENDA', 215.00, 15.00, 'CANCELADA');

INSERT INTO ventas VALUES
(1014, DATE '2026-03-22', 105, 3, 'WEB', 1450.00, 150.00, 'COMPLETADA');

INSERT INTO ventas VALUES
(1015, DATE '2026-03-29', 107, 7, 'TIENDA', 395.00, 20.00, 'COMPLETADA');

-- =====================================================
-- REGISTROS: TABLA AUXILIAR
-- =====================================================

INSERT INTO revision_cierre VALUES
(1, 'Tabla auxiliar para practicar operaciones DDL');

COMMIT;
```

### **Comprobación inicial**

Comprobar que las tablas existen mediante:

```sql
SELECT table_name
FROM user_tables
ORDER BY table_name;
```

Esta consulta es solo de comprobación y no forma parte de las respuestas.

---

## **2.4. Preguntas de negocio — Parte 1**

### **Pregunta 1 — Ciudades de los clientes**

Marketing quiere saber en qué ciudades existen clientes registrados.

Muestra únicamente las ciudades diferentes, sin repetir, y ordénalas alfabéticamente.

**Conceptos que deben aparecer:** `SELECT`, `DISTINCT`, `ORDER BY`.

---

### **Pregunta 2 — Clientes para una campaña**

Marketing quiere localizar clientes cuyo nombre comience por la letra `M` y que pertenezcan al segmento `PLUS` o `PREMIUM`.

Muestra:

- identificador del cliente;
- nombre;
- ciudad;
- segmento.

Ordena el resultado por nombre.

**Conceptos que deben aparecer:** `WHERE`, `LIKE`, `AND`, `OR`, paréntesis y `ORDER BY`.

---

### **Pregunta 3 — Ventas de un periodo**

El departamento financiero necesita revisar las ventas realizadas entre el **1 de febrero de 2026** y el **31 de marzo de 2026**, ambas fechas incluidas.

Solo deben aparecer las ventas cuyo importe esté entre **300 y 1.000 euros**.

Muestra:

- identificador de venta;
- fecha;
- canal;
- importe;
- estado.

Ordena desde la venta más reciente hasta la más antigua.

**Conceptos que deben aparecer:** `WHERE`, `BETWEEN`, operadores lógicos y `ORDER BY`.

---

### **Pregunta 4 — Ventas y clientes**

La dirección comercial quiere saber qué cliente realizó cada venta completada.

Relaciona únicamente las tablas `ventas` y `clientes`.

Muestra:

- identificador de venta;
- fecha de venta;
- nombre del cliente;
- importe;
- descuento;
- importe neto calculado como `importe - descuento`, redondeado a dos decimales.

Ordena el resultado por fecha de venta.

**Obligatorio:** utilizar un `INNER JOIN` sencillo entre **dos tablas**.

---

### **Pregunta 5 — Clientes con y sin ventas**

Atención al cliente necesita ver todos los clientes, incluidos aquellos que todavía no han realizado ninguna compra.

Relaciona únicamente las tablas `clientes` y `ventas`.

Muestra:

- identificador del cliente;
- nombre del cliente;
- cantidad de ventas realizadas.

Los clientes sin ventas deben aparecer con cantidad `0`.

Ordena primero por cantidad de ventas de mayor a menor y después por nombre.

**Obligatorio:** utilizar un `LEFT JOIN` sencillo entre **dos tablas**, `COUNT` y `GROUP BY`.

---

### **Pregunta 6 — Empleados y supervisores**

Recursos Humanos necesita un listado de todos los empleados con el nombre de su supervisor.

También debe aparecer Laura Gómez, aunque no tenga supervisor.

Muestra:

- identificador del empleado;
- nombre del empleado;
- puesto;
- nombre del supervisor.

Ordena el resultado por nombre del empleado.

**Obligatorio:** realizar un `SELF JOIN` sobre la tabla `empleados` utilizando dos alias diferentes.

---

### **Pregunta 7 — Sucursales con y sin empleados**

La dirección quiere comprobar cuántos empleados tiene cada sucursal, incluyendo las sucursales sin empleados.

Relaciona únicamente las tablas `empleados` y `sucursales`.

Muestra:

- identificador de la sucursal;
- nombre de la sucursal;
- ciudad;
- cantidad de empleados.

Ordena por identificador de sucursal.

**Obligatorio:** utilizar un `RIGHT JOIN` sencillo entre **dos tablas**, `COUNT` y `GROUP BY`.

---

### **Pregunta 8 — Resumen de ventas por canal**

La dirección quiere comparar los canales `TIENDA`, `WEB` y `TELEFONO`.

Utilizando únicamente la tabla `ventas`, muestra para cada canal:

- cantidad de ventas;
- suma de los importes;
- promedio de los importes;
- importe mínimo;
- importe máximo.

Redondea el promedio a dos decimales.

Solo deben aparecer los canales cuya suma de importes sea superior a **1.500 euros**.

Ordena el resultado desde la mayor suma hasta la menor.

**Conceptos que deben aparecer:** `GROUP BY`, `HAVING`, `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `ROUND` y `ORDER BY`.

---

### **Pregunta 9 — Ventas con descuento**

La gerencia quiere revisar las ventas que tienen un descuento mayor que cero.

Muestra:

- identificador de venta;
- fecha;
- importe;
- descuento;
- importe neto calculado como `importe - descuento` y redondeado a dos decimales.

Ordena el resultado desde el descuento más alto hasta el más bajo.

**Conceptos que deben aparecer:** `WHERE`, operación aritmética, `ROUND` y `ORDER BY`.

---

### **Pregunta 10 — Operaciones DDL básicas**

Realiza las siguientes operaciones en el orden indicado:

1. Renombra la tabla `revision_cierre` como `revision_comercial`.
2. Añade un comentario a la tabla indicando que se utilizó para practicar operaciones DDL.
3. Añade un comentario a la columna `observacion` indicando que almacena una nota temporal.
4. Elimina definitivamente la tabla `revision_comercial`.

**Conceptos que deben aparecer:** `RENAME`, `COMMENT` y `DROP`.

---

# **3. Parte 2 — Diseño y mantenimiento de una clínica veterinaria**

## **3.1. Escenario de negocio**

La clínica veterinaria **HuellaCare** necesita registrar propietarios, mascotas y consultas. En esta parte, el estudiante debe crear tres tablas relacionadas, insertar datos y realizar operaciones sencillas de actualización y eliminación.

---

## **3.2. Preparación del usuario de la Parte 2**

Desde el usuario `SYSTEM` o `SyS` :

```sql
CREATE USER examen_p2 IDENTIFIED BY "ExamenP2_2026"
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp
    QUOTA 30M ON users;

GRANT CREATE SESSION, CREATE TABLE TO examen_p2;
```

### **Datos orientativos de la conexión en SQL Developer**

| **Campo** | **Valor** |
| --- | --- |
| Nombre de conexión | `EXAMEN_PARTE_2` |
| Usuario | `EXAMEN_P2` |
| Contraseña | `ExamenP2_2026` |
| Tipo de conexión | Básica |
| Host | localhost |
| Puerto | `1521` |
| Tipo | Nombre de servicio |
| Servicio | `FREEPDB1` |

---

## **3.3. Reglas obligatorias del modelo**

### **Tabla `propietarios`**

| **Columna** | **Tipo de dato** | **Reglas** |
| --- | --- | --- |
| `id_propietario` | `NUMBER(4)` | Clave primaria |
| `dni` | `VARCHAR2(12)` | Obligatorio y único |
| `nombre` | `VARCHAR2(80)` | Obligatorio |
| `telefono` | `VARCHAR2(15)` | Obligatorio |
| `email` | `VARCHAR2(120)` | Obligatorio y único |
| `ciudad` | `VARCHAR2(60)` | Obligatorio |

### **Tabla `mascotas`**

| **Columna** | **Tipo de dato** | **Reglas** |
| --- | --- | --- |
| `id_mascota` | `NUMBER(4)` | Clave primaria |
| `nombre` | `VARCHAR2(60)` | Obligatorio |
| `especie` | `VARCHAR2(20)` | Obligatorio. Solo `PERRO`, `GATO`, `AVE` o `CONEJO` |
| `microchip` | `VARCHAR2(30)` | Único. Puede ser `NULL` |
| `estado` | `CHAR(1)` | Obligatorio. Solo `A` o `I` |
| `id_propietario` | `NUMBER(4)` | Obligatorio y clave foránea hacia `propietarios` |

### **Tabla `consultas`**

| **Columna** | **Tipo de dato** | **Reglas** |
| --- | --- | --- |
| `id_consulta` | `NUMBER(5)` | Clave primaria |
| `id_mascota` | `NUMBER(4)` | Obligatorio y clave foránea hacia `mascotas` |
| `fecha_consulta` | `DATE` | Obligatorio |
| `motivo` | `VARCHAR2(150)` | Obligatorio |
| `importe` | `NUMBER(8,2)` | Obligatorio y mayor o igual que cero |
| `estado` | `VARCHAR2(15)` | Obligatorio. Solo `PROGRAMADA`, `ATENDIDA` o `CANCELADA` |

---

## **3.4. Datos que deben insertarse**

### **Propietarios**

| **ID** | **DNI** | **Nombre** | **Teléfono** | **Email** | **Ciudad** |
| --- | --- | --- | --- | --- | --- |
| 1 | `12345678A` | Carmen Díaz | `600111222` | `carmen.diaz@email.es` | Madrid |
| 2 | `23456789B` | Luis Romero | `600222333` | `luis.romero@email.es` | Toledo |
| 3 | `34567890C` | Paula Martín | `600333444` | `paula.martin@email.es` | Madrid |
| 4 | `45678901D` | Andrés Soto | `600444555` | `andres.soto@email.es` | Segovia |
| 5 | `56789012E` | Elena Cruz | `600555666` | `elena.cruz@email.es` | Guadalajara |

### **Mascotas**

| **ID** | **Nombre** | **Especie** | **Microchip** | **Estado** | **Propietario** |
| --- | --- | --- | --- | --- | --- |
| 101 | Luna | PERRO | `MC-10001` | A | 1 |
| 102 | Milo | GATO | `MC-10002` | A | 1 |
| 103 | Coco | AVE | Sin microchip | A | 2 |
| 104 | Nala | GATO | `MC-10004` | A | 3 |
| 105 | Thor | PERRO | `MC-10005` | A | 4 |
| 106 | Kira | CONEJO | Sin microchip | A | 5 |

> Cuando la tabla indique **Sin microchip**, debe insertarse `NULL`.
> 

### **Consultas**

| **ID** | **Mascota** | **Fecha** | **Motivo** | **Importe** | **Estado** |
| --- | --- | --- | --- | --- | --- |
| 7001 | 101 | 05/04/2026 | Vacunación anual | 48.00 | ATENDIDA |
| 7002 | 102 | 08/04/2026 | Revisión general | 35.00 | ATENDIDA |
| 7003 | 103 | 14/04/2026 | Problema de plumaje | 42.50 | ATENDIDA |
| 7004 | 104 | 22/04/2026 | Limpieza dental | 75.00 | ATENDIDA |
| 7005 | 105 | 30/04/2026 | Cojera | 68.00 | ATENDIDA |
| 7006 | 106 | 06/05/2026 | Falta de apetito | 30.00 | ATENDIDA |
| 7007 | 101 | 18/06/2026 | Revisión dermatológica | 55.00 | PROGRAMADA |
| 7008 | 104 | 02/07/2026 | Control de peso | 38.00 | CANCELADA |

---

## **3.5. Preguntas — Parte 2**

### **Pregunta 11 — Crear la tabla de propietarios**

Crea la tabla `propietarios` con las columnas y reglas indicadas.

Debe incluir:

- clave primaria;
- dos restricciones `UNIQUE`;
- columnas obligatorias con `NOT NULL`;
- nombres explícitos para las restricciones.

---

### **Pregunta 12 — Crear la tabla de mascotas**

Crea la tabla `mascotas` con las columnas y reglas indicadas.

Debe incluir:

- clave primaria;
- clave foránea hacia `propietarios`;
- restricción `UNIQUE` para `microchip`;
- restricciones `NOT NULL`;
- restricciones `CHECK` para `especie` y `estado`;
- nombres explícitos para las restricciones.

---

### **Pregunta 13 — Crear la tabla de consultas**

Crea la tabla `consultas` con las columnas y reglas indicadas.

Debe incluir:

- clave primaria;
- clave foránea hacia `mascotas`;
- restricciones `NOT NULL`;
- una restricción `CHECK` para impedir importes negativos;
- una restricción `CHECK` para limitar los estados permitidos;
- nombres explícitos para las restricciones.

---

### **Pregunta 14 — Insertar los propietarios**

Inserta los cinco propietarios indicados.

Requisitos:

- escribir los nombres de las columnas en cada `INSERT`;
- no usar secuencias ni columnas `IDENTITY`;
- confirmar el bloque con `COMMIT`.

---

### **Pregunta 15 — Insertar las mascotas**

Inserta las seis mascotas indicadas.

Requisitos:

- respetar las relaciones con sus propietarios;
- insertar `NULL` cuando no exista microchip;
- confirmar el bloque con `COMMIT`.

---

### **Pregunta 16 — Insertar las consultas**

Inserta las ocho consultas indicadas.

Requisitos:

- respetar las claves foráneas;
- utilizar fechas compatibles con Oracle, por ejemplo `DATE '2026-04-05'`;
- confirmar el bloque con `COMMIT`.

---

### **Pregunta 17 — Actualizar los datos de una propietaria**

Paula Martín ha cambiado de ciudad y de teléfono.

Actualiza el registro con `id_propietario = 3` para que tenga:

```
Ciudad: Sevilla
Teléfono: 611333444
```

Procedimiento:

1. Consulta primero el registro.
2. Ejecuta el `UPDATE` con una condición `WHERE`.
3. Vuelve a consultar el registro.
4. Confirma el cambio con `COMMIT`.

---

### **Pregunta 18 — Actualizar una consulta programada**

La consulta `7007` ya ha sido atendida.

Actualízala con estos valores:

```
Estado: ATENDIDA
Importe: 60.00
```

Procedimiento:

1. Consulta primero el registro.
2. Ejecuta el `UPDATE` con una condición `WHERE`.
3. Vuelve a consultar el registro.
4. Confirma el cambio con `COMMIT`.

---

### **Pregunta 19 — Eliminar una consulta cancelada**

Elimina la consulta cancelada con `id_consulta = 7008`.

Procedimiento:

1. Consulta primero el registro.
2. Ejecuta el `DELETE` utilizando `WHERE`.
3. Comprueba que el registro ya no existe.
4. Confirma el cambio con `COMMIT`.

---

### **Pregunta 20 — Comprobar tablas y restricciones**

Añade al final del archivo:

1. Una consulta que muestre las tres tablas creadas desde `USER_TABLES`.
2. Una consulta que muestre las restricciones desde `USER_CONSTRAINTS`.

---