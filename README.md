
# Entorno Python + PostgreSQL con Docker

Este repositorio contiene un ejemplo mínimo de cómo levantar un servidor PostgreSQL mediante Docker Compose, administrar la base de datos con Adminer y ejecutar un script en Python (`demo.py`) que se conecta a la base de datos para ejecutar una consulta. A continuación se describe cada componente y los pasos para poner todo en funcionamiento.

---

## 📂 Estructura de archivos

```
├── demo.py
├── docker-compose.yaml
└── README.md

````

- **`docker-compose.yaml`**  
  Define los servicios Docker necesarios:
  1. **`database`**: contenedor con PostgreSQL (imagen oficial).  
     - Crea un usuario `docker` (contraseña `docker`) y una base de datos `exampledb`.  
     - Expone el puerto `5432` para conexiones externas.  
  2. **`adminer`**: contenedor con Adminer (interfaz web para gestionar bases de datos).  
     - Depende de `database` y expone el puerto `8080` para el acceso desde el navegador.

- **`demo.py`**  
  Script en Python que:
  1. Se conecta a PostgreSQL usando la librería `psycopg2`.  
  2. Ejecuta `SELECT * FROM student` sobre la base de datos `exampledb`.  
  3. Imprime por pantalla cada fila de la tabla `student`.  
  4. Cierra cursor y conexión al finalizar.

- **`README.md`**  
  Este archivo, con toda la documentación necesaria para el usuario.

---

## 🔧 Requisitos previos

Antes de empezar, asegúrate de tener instalado en tu máquina:

1. **Docker** (versión ≥ 20.10)  
2. **Docker Compose** (versión ≥ 1.25)  
3. **Python 3.8+**  
4. Un entorno virtual de Python (recomendado)  
5. Librería **psycopg2** para Python:  
   ```bash
   pip install psycopg2-binary

---

## 🚀 Paso a paso para poner todo en marcha

### 1. Levantar los contenedores con Docker Compose

1. Abre una terminal en esta carpeta (la que contiene `docker-compose.yaml`).

2. Ejecuta:

   ```bash
   docker-compose up -d
   ```

   Esto hará lo siguiente:

   * Descargará las imágenes de PostgreSQL y Adminer (si no las tienes localmente).
   * Iniciará el contenedor `database`, que:

     * Levanta un servidor PostgreSQL en el puerto `5432`.
     * Crea automáticamente el usuario `docker` (contraseña: `docker`) y la base de datos `exampledb`.
   * Iniciará el contenedor `adminer`, que expone la interfaz web en `http://localhost:8080`.

3. Verifica que ambos contenedores estén corriendo:

   ```bash
   docker ps
   ```

   Deberías ver algo similar a:

   ```
   NOMBRE                IMAGEN        PUERTOS
   repo_database_1     postgres    0.0.0.0:5432->5432/tcp
   repo_adminer_1      adminer     0.0.0.0:8080->8080/tcp
   ```

### 2. Crear la tabla `student` y poblarla

1. Abre tu navegador y visita [http://localhost:8080](http://localhost:8080).

2. En la pantalla de login de Adminer, rellena los campos:

   * **Sistema gestor**: PostgreSQL
   * **Servidor**: `database`

     > ⚠️ Cuando Adminer y PostgreSQL corren en contenedores Docker de la misma red, el hostname “database” (nombre del servicio en el `docker-compose.yaml`) apunta al contenedor.
   * **Usuario**: `docker`
   * **Contraseña**: `docker`
   * **Base de datos**: `exampledb`

3. Haz clic en “Login”. Una vez dentro:

   1. En el panel izquierdo, haz clic en “SQL Command”.
   2. Copia y pega la siguiente instrucción para crear la tabla `student`:

      ```sql
      CREATE TABLE student (
        id   SERIAL PRIMARY KEY,
        name VARCHAR(50),
        age  INTEGER
      );
      ```
   3. Haz clic en “Execute”.
   4. Después, inserta algunos datos de ejemplo:

      ```sql
      INSERT INTO student (name, age) VALUES
        ('Ana', 22),
        ('Luis', 19),
        ('María', 21);
      ```
   5. Haz clic en “Execute” nuevamente.
   6. Para verificar que los datos existen, en la columna izquierda haz clic en “student” y luego en “Select data” → “star(\*)” → “Execute”. Deberías ver las filas recién insertadas.

### 3. Clonar/descargar este repositorio

Si aún no lo has hecho, clona este repositorio en tu máquina local:

```bash
git clone <URL_DE_TU_REPOSITORIO>
cd <nombre_de_la_carpeta>
```

### 4. Configurar el entorno Python (opcional pero recomendado)

1. Crea y activa un entorno virtual:

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```
2. Instala la librería `psycopg2-binary`:

   ```bash
   pip install psycopg2-binary
   ```

### 5. Ejecutar el script `demo.py`

Con los contenedores en funcionamiento y la tabla `student` creada y poblada, ejecuta:

```bash
python demo.py
```

Deberías ver por pantalla algo similar a:

```
(1, 'Ana', 22)
(2, 'Luis', 19)
(3, 'María', 21)
```

Cada tupla representa un registro de la tabla `student` (campos `id`, `name`, `age`).

---

## 🔍 Descripción detallada de cada archivo

### 1. `docker-compose.yaml`

```yaml
services:
  database:
    image: postgres
    ports:
      - "5432:5432"
    restart: always
    environment:
      POSTGRES_USER: docker
      POSTGRES_PASSWORD: docker
      POSTGRES_DB: exampledb

  adminer:
    image: adminer
    restart: always
    depends_on:
      - database
    ports:
      - "8080:8080"
```

* **`database`**

  * `image: postgres`: usa la imagen oficial de PostgreSQL.
  * `ports: "5432:5432"`: mapea el puerto 5432 del contenedor al puerto 5432 del anfitrión.
  * `environment`:

    * `POSTGRES_USER`: usuario que se creará (“docker”).
    * `POSTGRES_PASSWORD`: contraseña del usuario (“docker”).
    * `POSTGRES_DB`: nombre de la base de datos que se creará (“exampledb”).

* **`adminer`**

  * `image: adminer`: usa la imagen oficial de Adminer.
  * `depends_on: - database`: espera a que el contenedor `database` arranque antes de iniciar Adminer.
  * `ports: "8080:8080"`: mapea el puerto 8080 del contenedor al puerto 8080 del anfitrión.

### 2. `demo.py`

```python
import psycopg2

# Conexión a la base de datos
conn = psycopg2.connect(
    database="exampledb",
    user="docker",
    password="docker",
    host="0.0.0.0"
)

# Crear cursor para ejecutar consultas
cur = conn.cursor()

# Ejecutar consulta SELECT
cur.execute("SELECT * FROM student")
rows = cur.fetchall()

# Mostrar cada fila por pantalla
for row in rows:
    print(row)

# Cerrar cursor y conexión
cur.close()
conn.close()
```

* **Línea por línea**:

  1. `import psycopg2`: importa el adaptador de Postgres.
  2. `psycopg2.connect(...)`: abre la conexión a `exampledb` con usuario y contraseña “docker” en el host `0.0.0.0` (mapeado al contenedor).
  3. `cur = conn.cursor()`: crea el cursor para enviar sentencias SQL.
  4. `cur.execute("SELECT * FROM student")`: ejecuta la consulta.
  5. `rows = cur.fetchall()`: obtiene todos los resultados.
  6. Bucle `for row in rows`: imprime cada tupla recuperada.
  7. `cur.close()` y `conn.close()`: cierra recursos.

---

## ⚙️ Personalización y ampliaciones

* **Cambiar credenciales o nombre de base de datos**
  Para modificar usuario/contraseña o nombre de base de datos, solo ajusta las variables de entorno en `docker-compose.yaml` (líneas `POSTGRES_USER`, `POSTGRES_PASSWORD` y `POSTGRES_DB`) y, en `demo.py`, mantén los mismos valores en `psycopg2.connect(...)`.

* **Añadir más tablas o scripts**

  * Puedes crear nuevas tablas usando Adminer o ejecutando scripts SQL en un directorio `initdb/` (consultar la [documentación oficial de Postgres Docker](https://hub.docker.com/_/postgres) para montar volúmenes de inicialización).
  * Amplía `demo.py` (o crea nuevos scripts) para insertar, actualizar o eliminar datos, ejecutar joins, etc.

* **Dockerfile adicional (opcional)**
  Si prefieres incluir tu propio `Dockerfile` para empaquetar y distribuir el script Python junto con dependencias, podrías crear una imagen separada. Por ejemplo:

  ```dockerfile
  FROM python:3.10-slim
  WORKDIR /app
  COPY demo.py .
  RUN pip install psycopg2-binary
  CMD ["python", "demo.py"]
  ```

  Y luego ajustar un servicio extra en `docker-compose.yaml` para ejecutar `demo.py` dentro de un contenedor.

---

## 🤝 Contribuciones

Si quieres mejorar este repositorio, puedes:

1. Abrir un **Issue** describiendo la sugerencia o el error.
2. Crear una **branch** con tu mejora y enviar un **Pull Request**.

Por ejemplo:

* Añadir validaciones en `demo.py`.
* Incorporar más ejemplos de consultas.
* Documentar cómo agregar volúmenes persistentes para PostgreSQL.
* Crear un contenedor extra que ejecute de forma automática `demo.py` cada cierto intervalo.

---

## 📝 Licencia

Este proyecto no incluye un archivo de licencia específico. Si planeas usarlo en un entorno compartido o público, considera añadir un `LICENSE` (por ejemplo, MIT, Apache 2.0, GPLv3, etc.) para dejar claras las condiciones de uso y distribución.

