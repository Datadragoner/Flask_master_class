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

