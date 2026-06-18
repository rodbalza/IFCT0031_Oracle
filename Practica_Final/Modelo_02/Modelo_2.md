# Modelo 2

# **Caso práctico: Consultas de negocio objetos de base de datos y seguridad básica**

Apellidos:

Nombre:

Fecha:

Firma. 

**Herramienta de trabajo:** Oracle SQL Developer **24.3.1.347**

**Formato de entrega:** un único archivo SQL

**Nombre obligatorio del archivo:** `NombreApellido.sql`

**Puntuación total:** 10 puntos. Tendrás 20 preguntas disponibles para **responder 10**. Puntuación 1 punto c/u.

**Modelo de examen a escoger**: El de tu preferencia

Son dos modelos de examen: Modelo 1 y Modelo 2 de modo que el estudiante tiene la libertad de escoger el que quiera. 

---

# **1. Instrucciones generales**

1. Todo el examen debe realizarse exclusivamente con **Oracle SQL Developer 24.3.1.347**.
2. El estudiante entregará únicamente un archivo llamado:  **NombreApellido.sql. Ejemplo:** AnaLopez.sql
3. El archivo debe comenzar con este encabezado:
    
    ```sql
    -- EXAMEN PRÁCTICO DE ORACLE SQL - MODELO 2
    -- Nombre y apellidos:
    -- Fecha:
    -- Archivo: NombreApellido.sql
    ```
    
4. Cada respuesta debe estar separada e identificada mediante comentarios:
    
    ```sql
    -- =====================================================
    -- PREGUNTA 1
    -- =====================================================
    
    -- Código SQL del estudiante
    ```
    
5. No se deben incluir las soluciones de una pregunta dentro de otra.
6. Los `INNER JOIN` y `LEFT JOIN` solicitados deben relacionar únicamente **dos tablas**.
7. El `SELF JOIN` debe realizarse utilizando una sola tabla con dos alias diferentes.
8. Las restricciones deben tener nombres claros, por ejemplo:
    
    ```
    pk_socios
    fk_prestamos_socios
    uk_socios_email
    ck_libros_precio
    ```
    
9. Después de los `INSERT`, `UPDATE` y `DELETE` se debe utilizar `COMMIT` cuando se solicite.
10. Las preguntas de administración deben ejecutarse desde la conexión habilitada para ello
11. Las contraseñas utilizadas en el examen son únicamente para un entorno de prácticas.
12. No se entregarán capturas, archivos PDF ni documentos adicionales.

---

# **3. Parte 1 — Biblioteca LibroAbierto**

## **3.1. Escenario de negocio**

LibroAbierto es una biblioteca privada que presta libros a sus socios. La dirección necesita consultar información sobre libros, socios, préstamos y empleados. La base de datos contiene estas tablas:

- `categorias`;
- `libros`;
- `socios`;
- `empleados`;
- `prestamos`.

Debes responder a las preguntas 1 a 10 mediante consultas SQL.

---

## **3.2. Preparación del usuario de la Parte 1**

Crear desde `SYS` el usuario:

```sql
CREATE USER examen_biblioteca IDENTIFIED BY "Biblioteca_2026"
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp
    QUOTA 30M ON users;

GRANT CREATE SESSION, CREATE TABLE TO examen_biblioteca;
```

### **Conexión**

| **Campo** | **Valor** |
| --- | --- |
| Nombre de conexión | `EXAMEN_BIBLIOTECA` |
| Usuario | `EXAMEN_BIBLIOTECA` |
| Contraseña | `Biblioteca_2026` |
| Tipo de conexión | Básica |
| Host | localhost |
| Puerto | `1521` |
| Servicio | `FREEPDB1` |

---

## **3.3. Script de creación de tablas y registros**

El siguiente script debe ejecutarse conectado como `EXAMEN_BIBLIOTECA` antes de comenzar las preguntas.

```sql
-- =====================================================
-- TABLA: CATEGORIAS
-- =====================================================

CREATE TABLE categorias (
    id_categoria  NUMBER(3),
    nombre        VARCHAR2(60) NOT NULL,

    CONSTRAINT pk_categorias
        PRIMARY KEY (id_categoria),

    CONSTRAINT uk_categorias_nombre
        UNIQUE (nombre)
);

-- =====================================================
-- TABLA: LIBROS
-- =====================================================

CREATE TABLE libros (
    id_libro       NUMBER(4),
    titulo         VARCHAR2(120) NOT NULL,
    autor          VARCHAR2(100) NOT NULL,
    precio         NUMBER(8,2) NOT NULL,
    anio_publicacion NUMBER(4) NOT NULL,
    disponible     CHAR(1) DEFAULT 'S' NOT NULL,
    id_categoria   NUMBER(3) NOT NULL,

    CONSTRAINT pk_libros
        PRIMARY KEY (id_libro),

    CONSTRAINT ck_libros_precio
        CHECK (precio > 0),

    CONSTRAINT ck_libros_disponible
        CHECK (disponible IN ('S', 'N')),

    CONSTRAINT fk_libros_categorias
        FOREIGN KEY (id_categoria)
        REFERENCES categorias (id_categoria)
);

-- =====================================================
-- TABLA: SOCIOS
-- =====================================================

CREATE TABLE socios (
    id_socio       NUMBER(4),
    nombre         VARCHAR2(80) NOT NULL,
    email          VARCHAR2(120) NOT NULL,
    ciudad         VARCHAR2(60) NOT NULL,
    fecha_alta     DATE NOT NULL,
    tipo_socio     VARCHAR2(15) NOT NULL,

    CONSTRAINT pk_socios
        PRIMARY KEY (id_socio),

    CONSTRAINT uk_socios_email
        UNIQUE (email),

    CONSTRAINT ck_socios_tipo
        CHECK (tipo_socio IN ('BASICO', 'PREMIUM'))
);

-- =====================================================
-- TABLA: EMPLEADOS
-- =====================================================

CREATE TABLE empleados (
    id_empleado    NUMBER(4),
    nombre         VARCHAR2(80) NOT NULL,
    puesto         VARCHAR2(60) NOT NULL,
    salario        NUMBER(10,2) NOT NULL,
    id_supervisor  NUMBER(4),

    CONSTRAINT pk_empleados
        PRIMARY KEY (id_empleado),

    CONSTRAINT ck_empleados_salario
        CHECK (salario > 0),

    CONSTRAINT fk_empleados_supervisor
        FOREIGN KEY (id_supervisor)
        REFERENCES empleados (id_empleado)
);

-- =====================================================
-- TABLA: PRESTAMOS
-- =====================================================

CREATE TABLE prestamos (
    id_prestamo       NUMBER(5),
    id_socio          NUMBER(4) NOT NULL,
    id_libro          NUMBER(4) NOT NULL,
    id_empleado       NUMBER(4) NOT NULL,
    fecha_prestamo    DATE NOT NULL,
    fecha_devolucion  DATE,
    estado            VARCHAR2(15) NOT NULL,

    CONSTRAINT pk_prestamos
        PRIMARY KEY (id_prestamo),

    CONSTRAINT ck_prestamos_estado
        CHECK (estado IN ('PRESTADO', 'DEVUELTO', 'RETRASADO')),

    CONSTRAINT fk_prestamos_socios
        FOREIGN KEY (id_socio)
        REFERENCES socios (id_socio),

    CONSTRAINT fk_prestamos_libros
        FOREIGN KEY (id_libro)
        REFERENCES libros (id_libro),

    CONSTRAINT fk_prestamos_empleados
        FOREIGN KEY (id_empleado)
        REFERENCES empleados (id_empleado)
);

-- =====================================================
-- REGISTROS: CATEGORIAS
-- =====================================================

INSERT INTO categorias VALUES (1, 'Novela');
INSERT INTO categorias VALUES (2, 'Tecnologia');
INSERT INTO categorias VALUES (3, 'Historia');
INSERT INTO categorias VALUES (4, 'Ciencia');
INSERT INTO categorias VALUES (5, 'Arte');

-- =====================================================
-- REGISTROS: LIBROS
-- =====================================================

INSERT INTO libros VALUES (101, 'El jardin secreto', 'Frances Hodgson Burnett', 18.50, 1911, 'S', 1);
INSERT INTO libros VALUES (102, 'Cien años de soledad', 'Gabriel Garcia Marquez', 22.90, 1967, 'N', 1);
INSERT INTO libros VALUES (103, 'Introduccion a SQL', 'Maria Torres', 29.95, 2023, 'S', 2);
INSERT INTO libros VALUES (104, 'Fundamentos de Oracle', 'Carlos Ruiz', 35.50, 2024, 'N', 2);
INSERT INTO libros VALUES (105, 'Historia de Europa', 'Laura Martin', 26.40, 2018, 'S', 3);
INSERT INTO libros VALUES (106, 'El universo', 'Ana Vega', 31.20, 2022, 'S', 4);
INSERT INTO libros VALUES (107, 'Fisica cotidiana', 'Luis Gomez', 24.80, 2020, 'N', 4);
INSERT INTO libros VALUES (108, 'Pintura moderna', 'Elena Soler', 27.75, 2019, 'S', 5);
INSERT INTO libros VALUES (109, 'Redes para principiantes', 'Jorge Gil', 33.10, 2024, 'S', 2);
INSERT INTO libros VALUES (110, 'La ciudad invisible', 'Marta Leon', 19.60, 2021, 'S', 1);

-- =====================================================
-- REGISTROS: SOCIOS
-- =====================================================

INSERT INTO socios VALUES (1, 'Ana Lopez', 'ana.lopez@email.com', 'Madrid', DATE '2025-01-10', 'PREMIUM');
INSERT INTO socios VALUES (2, 'Luis Perez', 'luis.perez@email.com', 'Sevilla', DATE '2025-02-18', 'BASICO');
INSERT INTO socios VALUES (3, 'Marta Sanchez', 'marta.sanchez@email.com', 'Madrid', DATE '2025-03-05', 'PREMIUM');
INSERT INTO socios VALUES (4, 'Carlos Diaz', 'carlos.diaz@email.com', 'Valencia', DATE '2025-04-12', 'BASICO');
INSERT INTO socios VALUES (5, 'Elena Ruiz', 'elena.ruiz@email.com', 'Bilbao', DATE '2025-05-20', 'PREMIUM');
INSERT INTO socios VALUES (6, 'Jorge Martin', 'jorge.martin@email.com', 'Madrid', DATE '2025-06-08', 'BASICO');
INSERT INTO socios VALUES (7, 'Lucia Romero', 'lucia.romero@email.com', 'Sevilla', DATE '2025-07-15', 'PREMIUM');
INSERT INTO socios VALUES (8, 'Pablo Moreno', 'pablo.moreno@email.com', 'Zaragoza', DATE '2025-08-22', 'BASICO');

-- =====================================================
-- REGISTROS: EMPLEADOS
-- Los supervisores se insertan primero.
-- =====================================================

INSERT INTO empleados VALUES (1, 'Laura Gomez', 'DIRECTORA', 42000, NULL);
INSERT INTO empleados VALUES (2, 'Diego Martin', 'RESPONSABLE', 32000, 1);
INSERT INTO empleados VALUES (3, 'Sara Molina', 'BIBLIOTECARIA', 24500, 2);
INSERT INTO empleados VALUES (4, 'Raul Torres', 'BIBLIOTECARIO', 23800, 2);
INSERT INTO empleados VALUES (5, 'Nuria Castro', 'AUXILIAR', 21000, 3);

-- =====================================================
-- REGISTROS: PRESTAMOS
-- =====================================================

INSERT INTO prestamos VALUES (1001, 1, 102, 3, DATE '2026-05-02', DATE '2026-05-12', 'DEVUELTO');
INSERT INTO prestamos VALUES (1002, 2, 104, 4, DATE '2026-05-05', NULL, 'PRESTADO');
INSERT INTO prestamos VALUES (1003, 1, 107, 3, DATE '2026-05-08', NULL, 'RETRASADO');
INSERT INTO prestamos VALUES (1004, 3, 101, 4, DATE '2026-05-10', DATE '2026-05-18', 'DEVUELTO');
INSERT INTO prestamos VALUES (1005, 4, 105, 3, DATE '2026-05-12', DATE '2026-05-20', 'DEVUELTO');
INSERT INTO prestamos VALUES (1006, 5, 106, 4, DATE '2026-05-14', NULL, 'PRESTADO');
INSERT INTO prestamos VALUES (1007, 3, 103, 3, DATE '2026-05-16', DATE '2026-05-23', 'DEVUELTO');
INSERT INTO prestamos VALUES (1008, 6, 108, 4, DATE '2026-05-18', NULL, 'PRESTADO');
INSERT INTO prestamos VALUES (1009, 2, 109, 3, DATE '2026-05-20', DATE '2026-05-27', 'DEVUELTO');
INSERT INTO prestamos VALUES (1010, 7, 110, 4, DATE '2026-05-22', NULL, 'RETRASADO');

COMMIT;
```

---

# **4. Preguntas de la Parte 1**

## **Pregunta 1 — Filtro y ordenación**

Mostrar el título, el autor y el precio de los libros cuyo precio esté entre **20 y 32 euros**. Ordenar el resultado desde el libro más caro hasta el más barato.

Debe utilizarse:

- `SELECT`;
- `WHERE`;
- `BETWEEN`;
- `ORDER BY`.

---

## **Pregunta 2 — Búsqueda con `LIKE`**

Mostrar el identificador, nombre y correo electrónico de los socios cuyo nombre comience por la letra **M**.

Debe utilizarse `LIKE`.

---

## **Pregunta 3 — Valores diferentes**

Mostrar las ciudades diferentes en las que viven los socios, sin repetir ninguna ciudad y ordenadas alfabéticamente.

Debe utilizarse `DISTINCT`.

---

## **Pregunta 4 — Funciones de agregación**

Calcular en una sola consulta:

- número total de libros;
- precio medio redondeado a dos decimales;
- precio mínimo;
- precio máximo.

Debe utilizarse `COUNT`, `AVG`, `ROUND`, `MIN` y `MAX`.

---

## **Pregunta 5 — Agrupación**

Mostrar cada categoría de libro y cuántos libros pertenecen a ella.

Debe utilizarse un `INNER JOIN` sencillo entre las tablas `categorias` y `libros`, además de `GROUP BY`.

---

## **Pregunta 6 — Filtrado de grupos**

Mostrar únicamente las categorías que tengan **dos o más libros**.

Debe utilizarse:

- `INNER JOIN` entre `categorias` y `libros`;
- `GROUP BY`;
- `HAVING`;
- `COUNT`.

---

## **Pregunta 7 — Socios con y sin préstamos**

Mostrar todos los socios y, cuando exista, el identificador de su préstamo y su estado.

Debe utilizarse un `LEFT JOIN` sencillo entre las tablas `socios` y `prestamos`.

El resultado debe incluir también a los socios que todavía no han realizado ningún préstamo.

---

## **Pregunta 8 — Empleados y supervisores**

Mostrar el nombre de cada empleado y el nombre de su supervisor.

Debe utilizarse un `SELF JOIN` sobre la tabla `empleados`.

El resultado debe incluir también a la directora, aunque no tenga supervisor.

---

## **Pregunta 9 — Operadores lógicos**

Mostrar los libros que:

- estén disponibles;
- pertenezcan a la categoría `Tecnologia` o `Ciencia`;
- tengan un precio inferior a 35 euros.

Debe utilizarse `AND`, `OR` y paréntesis.

---

## **Pregunta 10 — Total de préstamos por estado**

Mostrar cada estado de préstamo y la cantidad de préstamos que tiene ese estado. Ordenar el resultado desde el estado con más préstamos hasta el que tenga menos.

Debe utilizarse:

- `COUNT`;
- `GROUP BY`;
- `ORDER BY`.

---

# **5. Parte 2 — Centro de formación AulaDigital**

## **5.1. Escenario práctico**

AulaDigital necesita una pequeña base de datos para gestionar cursos, alumnos y matrículas. El estudiante debe construir las tablas, insertar datos y crear objetos que faciliten el trabajo diario.

Las tablas serán:

- `ad_cursos`;
- `ad_alumnos`;
- `ad_matriculas`.

---

## **5.2. Preparación del usuario de trabajo**

```sql
CREATE USER examen_aula IDENTIFIED BY "Aula_2026"
    DEFAULT TABLESPACE users
    TEMPORARY TABLESPACE temp
    QUOTA 50M ON users;

GRANT CREATE SESSION,
      CREATE TABLE,
      CREATE VIEW,
      CREATE MATERIALIZED VIEW
TO examen_aula;
```

---

# **6. Preguntas de la Parte 2**

## **Pregunta 11 — Crear tres tablas relacionadas**

Crear las tablas `ad_cursos`, `ad_alumnos` y `ad_matriculas` con las columnas y restricciones indicadas.

### **Tabla `ad_cursos`**

| **Columna** | **Tipo** | **Reglas** |
| --- | --- | --- |
| `id_curso` | `NUMBER(4)` | Clave primaria |
| `nombre` | `VARCHAR2(100)` | Obligatorio y único |
| `precio` | `NUMBER(8,2)` | Obligatorio y mayor que 0 |
| `modalidad` | `VARCHAR2(15)` | Solo `PRESENCIAL` u `ONLINE` |
| `activo` | `CHAR(1)` | Solo `S` o `N`, valor predeterminado `S` |

### **Tabla `ad_alumnos`**

| **Columna** | **Tipo** | **Reglas** |
| --- | --- | --- |
| `id_alumno` | `NUMBER(4)` | Clave primaria |
| `nombre` | `VARCHAR2(80)` | Obligatorio |
| `email` | `VARCHAR2(120)` | Obligatorio y único |
| `ciudad` | `VARCHAR2(60)` | Obligatorio |
| `fecha_alta` | `DATE` | Obligatorio |

### **Tabla `ad_matriculas`**

| **Columna** | **Tipo** | **Reglas** |
| --- | --- | --- |
| `id_matricula` | `NUMBER(5)` | Clave primaria |
| `id_alumno` | `NUMBER(4)` | Obligatorio y clave foránea |
| `id_curso` | `NUMBER(4)` | Obligatorio y clave foránea |
| `fecha_matricula` | `DATE` | Obligatorio |
| `estado` | `VARCHAR2(15)` | Solo `ACTIVA`, `FINALIZADA` o `CANCELADA` |
| `nota_final` | `NUMBER(4,2)` | Entre 0 y 10, puede ser nula |

Todas las restricciones deben tener un nombre descriptivo.

---

## **Pregunta 12 — Insertar registros**

Insertar como mínimo:

- 4 cursos;
- 5 alumnos;
- 7 matrículas.

Requisitos:

- debe existir al menos un curso `ONLINE`;
- debe existir al menos un curso `PRESENCIAL`;
- debe existir al menos una matrícula de cada estado;
- al menos una matrícula debe tener `nota_final` nula.

Finalizar con `COMMIT`.

---

## **Pregunta 13 — Actualizar registros**

Aumentar en un **10 %** el precio de todos los cursos cuya modalidad sea `ONLINE`. Después, mostrar los cursos modificados y confirmar el cambio con `COMMIT`.

---

## **Pregunta 14 — Eliminar registros**

Eliminar únicamente las matrículas cuyo estado sea `CANCELADA`. Antes del `DELETE`, realizar una consulta que permita comprobar qué filas se van a eliminar. Después del borrado, confirmar el cambio con `COMMIT`.

---

## **Pregunta 15 — Crear una vista sencilla**

Crear una vista llamada `vw_cursos_activos` que muestre:

- `id_curso`;
- `nombre`;
- `precio`;
- `modalidad`.

La vista debe incluir únicamente los cursos cuyo campo `activo` sea igual a `S`. Después de crearla, consultar todos sus registros.

---

## **Pregunta 16 — Crear una vista con dos tablas**

Crear una vista llamada `vw_alumnos_matriculas` que relacione únicamente las tablas `ad_alumnos` y `ad_matriculas`.

La vista debe mostrar:

- identificador del alumno;
- nombre del alumno;
- correo electrónico;
- identificador de matrícula;
- fecha de matrícula;
- estado.

Debe utilizarse un `INNER JOIN` entre dos tablas. Después de crear la vista, realizar una consulta sobre ella.

---

## **Pregunta 17 — Crear una vista materializada**

Crear una vista materializada llamada `mv_resumen_cursos` que muestre, para cada curso:

- identificador del curso;
- nombre del curso;
- número de matrículas.

Requisitos:

- utilizar las tablas `ad_cursos` y `ad_matriculas`;
- incluir también los cursos que no tengan matrículas;
- utilizar actualización manual con `REFRESH COMPLETE ON DEMAND`;
- crearla inicialmente con datos mediante `BUILD IMMEDIATE`.

Después de crearla:

1. consultar su contenido;
2. insertar una nueva matrícula válida en `ad_matriculas`;
3. confirmar el `INSERT`;
4. actualizar manualmente la vista materializada mediante `DBMS_MVIEW.REFRESH`;
5. consultar de nuevo la vista materializada.

---

## **Pregunta 18 — Crear una tabla temporal**

Crear una tabla temporal global llamada `tmp_informe_alumnos` con estas columnas:

| **Columna** | **Tipo** |
| --- | --- |
| `id_alumno` | `NUMBER(4)` |
| `nombre` | `VARCHAR2(80)` |
| `total_matriculas` | `NUMBER(4)` |

La tabla debe conservar sus filas hasta finalizar la transacción mediante:

```sql
ON COMMIT DELETE ROWS
```

Después:

1. insertar en la tabla temporal un resumen con el total de matrículas de cada alumno;
2. consultar su contenido antes del `COMMIT`;
3. ejecutar `COMMIT`;
4. volver a consultar la tabla para comprobar su comportamiento.

---

# **7. Parte 3 — Gestión básica de usuarios, roles y privilegios**

## **7.1. Conexión administrativa**

Las preguntas 19 y 20 deben ejecutarse desde una conexión administrativa . No deben ejecutarse desde el usuario `EXAMEN_AULA`.

# **8. Preguntas de la Parte 3**

## **Pregunta 19 — Crear y gestionar usuarios**

Desde la conexión administrativa:

1. crear el usuario `ad_consultor` con una contraseña de prácticas;
2. asignarle el tablespace `USERS` como predeterminado y `TEMP` como temporal;
3. concederle el privilegio `CREATE SESSION`;
4. crear el usuario `ad_gestor` con una contraseña de prácticas;
5. concederle `CREATE SESSION` y `CREATE VIEW`;
6. bloquear temporalmente la cuenta `ad_consultor`;
7. consultar el diccionario de datos para comprobar el estado de ambos usuarios;
8. desbloquear de nuevo la cuenta `ad_consultor`.

Todas las operaciones deben quedar escritas en el archivo SQL entregado.

---

## **Pregunta 20 — Crear un rol y administrar privilegios**

Desde la conexión administrativa:

1. crear un rol llamado `rol_consulta_aula`;
2. conceder al rol permiso `SELECT` sobre las tablas:
    - `EXAMEN_AULA.AD_CURSOS`;
    - `EXAMEN_AULA.AD_ALUMNOS`;
3. conceder al rol permiso `SELECT` sobre la vista `EXAMEN_AULA.VW_CURSOS_ACTIVOS`;
4. asignar el rol al usuario `ad_consultor`;
5. conceder directamente al usuario `ad_gestor` los privilegios `SELECT`, `INSERT` y `UPDATE` sobre `EXAMEN_AULA.AD_MATRICULAS`;
6. consultar las vistas del diccionario de datos necesarias para comprobar:
    - el rol asignado a `ad_consultor`;
    - los privilegios del rol;
    - los privilegios directos de `ad_gestor`;
7. revocar a `ad_gestor` únicamente el privilegio `UPDATE` sobre `EXAMEN_AULA.AD_MATRICULAS`;
8. realizar otra consulta al diccionario para comprobar que conserva `SELECT` e `INSERT`, pero ya no `UPDATE`.

---