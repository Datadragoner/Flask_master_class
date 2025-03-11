# Flask Master Class

Repositorio para encaminar a cualquiera a ser un maestro de la herramienta Flask.

## 📌 Primera referencia para completos principiantes  
🔗 **[Guía en YouTube](https://www.youtube.com/watch?v=-1DmVCPB6H8&ab_channel=UskoKruM2010)**  

¡Empieza tu viaje con Flask hoy mismo! 🚀

---

## 📜 Proyecto: Aplicación Flask con SQLite

Este repositorio contiene una aplicación básica de Flask que permite registrar y visualizar calificaciones de alumnos utilizando SQLite como base de datos. Vamos a desglosar el código paso a paso.

### 🔹 1. Instalación de dependencias
Antes de comenzar, asegúrate de tener Flask instalado. Puedes hacerlo con:

```bash
pip install flask
```

También utilizaremos SQLite, que viene incluido en Python por defecto, por lo que no es necesario instalarlo.

### 🔹 2. Creando la aplicación Flask
El primer paso es importar las librerías necesarias y crear una instancia de Flask:

```python
from flask import Flask, render_template, request, redirect
import sqlite3

app = Flask(__name__)
```

Aquí:
- `Flask` es la clase principal para crear la aplicación.
- `render_template` nos permitiría usar archivos HTML (aunque en este ejemplo usamos cadenas HTML en las rutas).
- `request` y `redirect` permiten manejar formularios y redireccionar a otras páginas.
- `sqlite3` es la biblioteca para manejar nuestra base de datos.

### 🔹 3. Creando la base de datos
Ahora, definimos la base de datos y creamos la tabla si no existe:

```python
conn = sqlite3.connect("alumnos.db")
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS alumnos (
                idal INTEGER PRIMARY KEY,
                asignatura TEXT NOT NULL,
                nota REAL NOT NULL)''')
conn.commit()
conn.close()
```

Aquí:
- `sqlite3.connect("alumnos.db")` crea un archivo de base de datos SQLite.
- `CREATE TABLE IF NOT EXISTS alumnos` crea la tabla si aún no existe.
- `conn.commit()` guarda los cambios.
- `conn.close()` cierra la conexión a la base de datos.

### 🔹 4. Creando la ruta de inicio

Definimos una ruta principal (`/`) con enlaces a las demás secciones:

```python
@app.route('/')
def home():
    return '''<h1>Bienvenido</h1>
              <a href="/formulario">Ingresar Datos</a><br>
              <a href="/resultados">Ver Resultados</a>'''
```

Esta función genera una página simple con enlaces a:
- `/formulario`: para ingresar datos de los alumnos.
- `/resultados`: para ver las notas almacenadas.

### 🔹 5. Creando el formulario para ingresar datos

```python
@app.route('/formulario', methods=['GET', 'POST'])
def formulario():
    if request.method == 'POST':
        idal = request.form['idal']
        asignatura = request.form['asignatura']
        nota = request.form['nota']
        
        conn = sqlite3.connect("alumnos.db")
        c = conn.cursor()
        c.execute("INSERT INTO alumnos (idal, asignatura, nota) VALUES (?, ?, ?)", (idal, asignatura, nota))
        conn.commit()
        conn.close()
        
        return redirect('/resultados')
    
    return '''<form method="post">
                ID Alumno: <input type="number" name="idal" required><br>
                Asignatura: <input type="text" name="asignatura" required><br>
                Nota: <input type="number" step="0.1" name="nota" required><br>
                <input type="submit" value="Guardar">
              </form>'''
```

Aquí:
- Si el método es `POST`, se guardan los datos del formulario en la base de datos.
- Se usa `request.form[]` para obtener los valores ingresados.
- `redirect('/resultados')` redirige a la página de resultados.
- Si es `GET`, simplemente se muestra el formulario en HTML.

### 🔹 6. Mostrando los resultados

```python
@app.route('/resultados')
def resultados():
    conn = sqlite3.connect("alumnos.db")
    c = conn.cursor()
    c.execute("SELECT * FROM alumnos")
    alumnos = c.fetchall()
    conn.close()
    
    tabla = "<table border='1'><tr><th>ID Alumno</th><th>Asignatura</th><th>Nota</th></tr>"
    for alumno in alumnos:
        tabla += f"<tr><td>{alumno[0]}</td><td>{alumno[1]}</td><td>{alumno[2]}</td></tr>"
    tabla += "</table>"
    
    return f"<h1>Resultados</h1>{tabla}<br><a href='/'>Volver</a>"
```

Aquí:
- Se consulta la base de datos (`SELECT * FROM alumnos`).
- Se construye una tabla HTML con los datos.
- Se muestra un enlace para volver a la página principal.

### 🔹 7. Ejecutando la aplicación
Finalmente, ejecutamos la aplicación en modo debug:

```python
if __name__ == "__main__":
    app.run(debug=True)
```

Esto permite que los cambios en el código se reflejen automáticamente al recargar la página.

---

## 🚀 Ejecución del proyecto
Para correr la aplicación, simplemente usa el siguiente comando en la terminal:

```bash
python app.py
```

Luego, abre tu navegador y ve a `http://127.0.0.1:5000/` para interactuar con la aplicación.

---

¡Listo! Ahora tienes una aplicación funcional en Flask con una base de datos SQLite. 🎉



¡Perfecto! Vamos a estructurar el README de GitHub explicando cada parte del código y proporcionando las celdas para que los usuarios puedan copiarlo de manera fácil. A continuación te doy una sugerencia de cómo puedes estructurarlo.

---

# Aplicación Flask para Gestionar Notas

Este es un proyecto básico de una aplicación web utilizando Flask y SQLite para gestionar las notas de los estudiantes. Los usuarios pueden ingresar, corregir y ver las notas, todo ello protegido por un sistema de autenticación simple.

## Requisitos

- Python 3.x
- Flask
- SQLite

Puedes instalar Flask ejecutando:

```bash
pip install flask
```

## Estructura del Proyecto

El código está dividido en varias rutas y funciones que gestionan la autenticación de usuarios, la inserción y actualización de notas, y la visualización de datos. Vamos a ver cada parte del código con más detalle.

---

## 1. Configuración Inicial

Lo primero que hacemos es importar las librerías necesarias, como Flask, y configurar nuestra base de datos SQLite. 

```python
from flask import Flask, render_template, request, redirect, url_for, flash
import sqlite3

app = Flask(__name__)
app.secret_key = 'secreto'
```

- **Flask**: Es el framework web que utilizamos para crear la aplicación.
- **sqlite3**: Usamos esta librería para interactuar con la base de datos SQLite.
- **`app.secret_key`**: Es importante para gestionar las sesiones y mensajes flash.

---

## 2. Configuración de la Base de Datos

A continuación, definimos dos funciones para conectar a la base de datos y crear la tabla donde almacenaremos las notas.

### Conexión a la base de datos

```python
def conectar_db():
    conn = sqlite3.connect('notas.db')
    conn.row_factory = sqlite3.Row
    return conn
```

Esta función se encarga de crear una conexión con la base de datos `notas.db` y configurar el formato de las filas como diccionarios para facilitar el acceso a las columnas.

### Creación de la tabla

```python
def crear_tabla():
    with conectar_db() as conn:
        conn.execute('''CREATE TABLE IF NOT EXISTS notas (
                        idal INTEGER PRIMARY KEY,
                        nota_media REAL NOT NULL)''')
        conn.commit()

crear_tabla()
```

Aquí, creamos una tabla `notas` con dos columnas: `idal` (ID del alumno) y `nota_media` (la nota del alumno). Si la tabla ya existe, no la crea de nuevo.

---

## 3. Rutas y Funciones de la Aplicación

Ahora vamos a explorar las distintas rutas de la aplicación. Cada ruta corresponde a una página o funcionalidad específica.

### Ruta de login

```python
@app.route('/', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        usuario = request.form['usuario']
        contraseña = request.form['contraseña']
        if usuario == 'admin' and contraseña == 'admin':
            return redirect(url_for('menu'))
        else:
            flash('Usuario o contraseña incorrectos. Inténtelo de nuevo.')
    return render_template('login.html')
```

- **`@app.route('/')`**: Esta es la página de inicio, donde los usuarios deben iniciar sesión.
- Si las credenciales son correctas (`usuario = admin` y `contraseña = admin`), redirige al usuario al menú principal.

---

### Ruta de Menú

```python
@app.route('/menu')
def menu():
    return render_template('menu.html')
```

- **`/menu`**: Una vez que el usuario ha iniciado sesión correctamente, se muestra el menú con las opciones disponibles.

---

### Ruta para Introducir Notas

```python
@app.route('/introducir', methods=['GET', 'POST'])
def introducir():
    if request.method == 'POST':
        idal = request.form['idal']
        nota_media = request.form['nota_media']
        try:
            with conectar_db() as conn:
                conn.execute('INSERT INTO notas (idal, nota_media) VALUES (?, ?)', (idal, nota_media))
                conn.commit()
            imagen = 'aprobado.png' if float(nota_media) >= 5 else 'suspendido.png'
            return render_template('resultado.html', imagen=imagen)
        except sqlite3.IntegrityError:
            flash('Error: IDAL ya existe.')
    return render_template('introducir.html')
```

- **`/introducir`**: En esta ruta, el usuario puede ingresar las notas de los estudiantes.
- Si la nota es mayor o igual a 5, se muestra una imagen de "aprobado", de lo contrario, una de "suspendido".
- Si el `IDAL` ya existe en la base de datos, se muestra un mensaje de error.

---

### Ruta para Corregir Notas

```python
@app.route('/corregir', methods=['GET', 'POST'])
def corregir():
    if request.method == 'POST':
        idal = request.form['idal']
        nota_media = request.form['nota_media']
        with conectar_db() as conn:
            conn.execute('UPDATE notas SET nota_media = ? WHERE idal = ?', (nota_media, idal))
            conn.commit()
        flash(f'Nota media actualizada para el IDAL: {idal}')
    return render_template('corregir.html')
```

- **`/corregir`**: Esta ruta permite al usuario corregir las notas de los estudiantes mediante su `IDAL`.

---

### Ruta para Mostrar Notas

```python
@app.route('/mostrar')
def mostrar():
    with conectar_db() as conn:
        notas = conn.execute('SELECT * FROM notas').fetchall()
    return render_template('mostrar.html', notas=notas)
```

- **`/mostrar`**: Muestra todas las notas almacenadas en la base de datos.

---

## 4. Ejecutar la Aplicación

Finalmente, al final del archivo se incluye este bloque para ejecutar la aplicación en modo de depuración:

```python
if __name__ == '__main__':
    app.run(debug=True)
```

Este bloque asegura que la aplicación se ejecute si el archivo se ejecuta directamente desde la línea de comandos.

---

## Plantillas HTML

Este proyecto también requiere las plantillas HTML (`login.html`, `menu.html`, `introducir.html`, `resultado.html`, `corregir.html`, `mostrar.html`). Estas plantillas deben estar en un directorio llamado `templates`.

---

## Conclusión

Este proyecto es un ejemplo sencillo de cómo manejar bases de datos con Flask y SQLite, permitiendo la gestión de notas a través de una interfaz web. Puedes ampliar este proyecto añadiendo más funcionalidades como la autenticación avanzada, validación de entradas o incluso la creación de un sistema de usuarios más robusto.

---

¡Y eso es todo! Ahora los usuarios pueden seguir las instrucciones paso a paso y copiar cada fragmento de código según lo vayan necesitando.




Lo que tu profesor te dio (`{% block titulo %}{% endblock %}`) es parte de una característica poderosa de **Jinja2**, que es el motor de plantillas que Flask utiliza. La idea aquí es tener una **plantilla base** con una estructura común, y luego, en las páginas específicas, puedes reemplazar o extender partes de esa plantilla base para personalizar el contenido según sea necesario.

### Pasos para usar bloques y plantillas base en Flask:

#### 1. **Crear la plantilla base (base.html):**
   Este archivo contendrá el HTML común, donde se definen los bloques. En tu caso, parece que tienes un bloque llamado `titulo` que probablemente se usa para definir el título de la página.

   Ejemplo de `base.html`:

   ```html
   <!DOCTYPE html>
   <html lang="es">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>{% block titulo %}Mi Aplicación Flask{% endblock %}</title>
   </head>
   <body>
       <header>
           <h1>Bienvenido a Mi Aplicación Flask</h1>
           <nav>
               <ul>
                   <li><a href="/">Inicio</a></li>
                   <li><a href="/pista">Pista</a></li>
                   <li><a href="/profesor">Profesor</a></li>
               </ul>
           </nav>
       </header>

       <div class="content">
           {% block contenido %}{% endblock %}
       </div>

       <footer>
           <p>&copy; 2025 Mi Aplicación Flask</p>
       </footer>
   </body>
   </html>
   ```

   En este ejemplo:
   - **`{% block titulo %}`** y **`{% endblock %}`** definen un bloque para el título de la página. Si no se especifica un título, el valor por defecto es "Mi Aplicación Flask".
   - **`{% block contenido %}`** es otro bloque donde se insertará el contenido específico de cada página.

#### 2. **Crear una plantilla específica (por ejemplo, pista.html):**
   En esta página, utilizarás la plantilla base y reemplazarás los bloques que has definido en `base.html`.

   Ejemplo de `pista.html`:

   ```html
   {% extends "base.html" %}

   {% block titulo %}Pista - Mi Aplicación Flask{% endblock %}

   {% block contenido %}
   <h2>Información sobre la Pista</h2>
   <p>Este es el contenido relacionado con la pista.</p>
   {% endblock %}
   ```

   En este archivo:
   - **`{% extends "base.html" %}`** indica que esta página se basa en la plantilla `base.html`.
   - **`{% block titulo %}`** reemplaza el bloque `titulo` de `base.html` con "Pista - Mi Aplicación Flask".
   - **`{% block contenido %}`** reemplaza el bloque `contenido` de `base.html` con información específica de la pista.

#### 3. **Crear otra plantilla específica (por ejemplo, profesor.html):**
   Siguiendo el mismo concepto, si deseas hacer una página para el profesor, se haría de manera similar.

   Ejemplo de `profesor.html`:

   ```html
   {% extends "base.html" %}

   {% block titulo %}Profesor - Mi Aplicación Flask{% endblock %}

   {% block contenido %}
   <h2>Información sobre el Profesor</h2>
   <p>Este es el contenido relacionado con el profesor.</p>
   {% endblock %}
   ```

#### 4. **Modificar la vista en `app.py`:**
   Ahora, en tu archivo `app.py`, puedes crear rutas para estas páginas específicas y renderizar las plantillas correspondientes.

   ```python
   from flask import Flask, render_template

   app = Flask(__name__)

   @app.route('/')
   def home():
       return render_template('index.html')

   @app.route('/pista')
   def pista():
       return render_template('pista.html')

   @app.route('/profesor')
   def profesor():
       return render_template('profesor.html')

   if __name__ == '__main__':
       app.run(debug=True)
   ```

#### 5. **Estructura del proyecto final:**

Tu proyecto debería verse algo así:

```
/mi_aplicacion
├── app.py
└── templates/
    ├── base.html
    ├── index.html
    ├── pista.html
    └── profesor.html
```

### Explicación:

- **`{% extends "base.html" %}`** en `pista.html` y `profesor.html` indica que ambas páginas heredarán de la plantilla base `base.html`.
- **`{% block titulo %}` y `{% block contenido %}`** son lugares donde puedes insertar contenido específico para cada página. Cuando visitas la página `/pista`, el bloque `titulo` se reemplazará por "Pista - Mi Aplicación Flask", y el bloque `contenido` se reemplazará con el contenido de esa página específica.
  
De esta forma, puedes mantener una estructura común (encabezado, pie de página, navegación) y personalizar solo las partes necesarias para cada página específica, como el título o el contenido.

¡Y eso es todo! Con esto, puedes crear una aplicación Flask modular y bien estructurada utilizando plantillas base y bloques.
