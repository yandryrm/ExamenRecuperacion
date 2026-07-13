# ExamenRecuperacion{

from flask import Flask, jsonify, request
import psycopg2

app = Flask(__name__)

conexion = psycopg2.connect(
    host="localhost",
    database="bibliteca",   
    user="postgres",        
    password="12345",     
    port="5432"
)
print("Conexión exitosa a la base de datos 'biblioteca'")


@app.route('/', methods=['GET'])
def obtener_todo():  
    cursor = conexion.cursor()
    
    cursor.execute(
        """
        SELECT p.id_prestamo, p.nombre_estudiante, p.curso, p.fecha_prestamo, p.fecha_devolucion, l.titulo
        FROM prestamos p
        INNER JOIN libros l ON p.id_libro = l.id_libro
        """
    )
    datos_prestamos = cursor.fetchall()
    lista_prestamos = []
    for p in datos_prestamos:
        lista_prestamos.append({
            "id_prestamo": p[0],
            "nombre_estudiante": p[1],
            "curso": p[2],
            "fecha_prestamo": str(p[3]),  
            "fecha_devolucion": str(p[4]),
            "titulo_libro": p[5]          
        })

    # 2. Obtener libros
    cursor.execute(
        "SELECT id_libro, titulo, autor, editorial, anio_publicacion, cantidad FROM libros"
    )
    datos_libros = cursor.fetchall()
    lista_libros = []
    for l in datos_libros:
        lista_libros.append({
            "id_libro": l[0],
            "titulo": l[1],
            "autor": l[2],
            "editorial": l[3],
            "anio_publicacion": l[4],
            "cantidad": l[5]
        })

    cursor.close()

    return jsonify({
        "prestamos": lista_prestamos,
        "libros": lista_libros
    })



@app.route('/prestamos', methods=['GET'])
def obtener_prestamos():  
    cursor = conexion.cursor()
    cursor.execute(
        """
        SELECT p.id_prestamo, p.nombre_estudiante, p.curso, p.fecha_prestamo, p.fecha_devolucion, l.titulo
        FROM prestamos p
        INNER JOIN libros l ON p.id_libro = l.id_libro
        """
    )
    datos = cursor.fetchall()
    prestamos = []
    for prestamo in datos:
        prestamos.append({
            "id_prestamo": prestamo[0],
            "nombre_estudiante": prestamo[1],
            "curso": prestamo[2],
            "fecha_prestamo": str(prestamo[3]),  
            "fecha_devolucion": str(prestamo[4]),
            "titulo_libro": prestamo[5]          
        })
    cursor.close()
    return jsonify(prestamos)


@app.route('/prestamos/<int:id>', methods=['GET'])
def obtener_prestamo(id):  
    cursor = conexion.cursor()
    cursor.execute(
        """
        SELECT p.id_prestamo, p.nombre_estudiante, p.curso, p.fecha_prestamo, p.fecha_devolucion, l.titulo
        FROM prestamos p
        INNER JOIN libros l ON p.id_libro = l.id_libro
        WHERE p.id_prestamo=%s
        """,
        (id,)
    )
    prestamo = cursor.fetchone()
    cursor.close()
    if prestamo:
        return jsonify({
            "id_prestamo": prestamo[0],
            "nombre_estudiante": prestamo[1],
            "curso": prestamo[2],
            "fecha_prestamo": str(prestamo[3]),
            "fecha_devolucion": str(prestamo[4]),
            "titulo_libro": prestamo[5]
        })
    return jsonify({"error": "Prestamo no encontrado"}), 404


@app.route('/prestamos', methods=['POST'])
def crear_prestamo():  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        """
        INSERT INTO prestamos(id_libro, nombre_estudiante, curso, fecha_prestamo, fecha_devolucion) 
        VALUES(%s, %s, %s, %s, %s) RETURNING id_prestamo
        """,
        (datos['id_libro'], datos['nombre_estudiante'], datos['curso'], datos['fecha_prestamo'], datos['fecha_devolucion'])
    )
    nuevo_id = cursor.fetchone()[0]
    conexion.commit()
    cursor.close()
    return jsonify({
        'id_prestamo': nuevo_id,
        'id_libro': datos['id_libro'],
        'nombre_estudiante': datos['nombre_estudiante'],
        'curso': datos['curso'],
        'fecha_prestamo': datos['fecha_prestamo'],
        'fecha_devolucion': datos['fecha_devolucion']
    }), 201


@app.route('/prestamos/<int:id>', methods=['PUT'])
def actualizar_prestamo(id):  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        """
        UPDATE prestamos 
        SET id_libro=%s, nombre_estudiante=%s, curso=%s, fecha_prestamo=%s, fecha_devolucion=%s 
        WHERE id_prestamo=%s
        """,
        (datos['id_libro'], datos['nombre_estudiante'], datos['curso'], datos['fecha_prestamo'], datos['fecha_devolucion'], id)
    )
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Prestamo actualizado correctamente'})


@app.route('/prestamos/<int:id>', methods=['DELETE'])
def eliminar_prestamo(id):  
    cursor = conexion.cursor()
    cursor.execute('DELETE FROM prestamos WHERE id_prestamo=%s', (id,))
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Prestamo eliminado correctamente'})



@app.route('/libros', methods=['GET'])
def obtener_libros():  
    cursor = conexion.cursor()
    cursor.execute("SELECT id_libro, titulo, autor, editorial, anio_publicacion, cantidad FROM libros")
    datos = cursor.fetchall()
    libros = []
    for libro in datos:
        libros.append({
            "id_libro": libro[0],
            "titulo": libro[1],
            "autor": libro[2],
            "editorial": libro[3],
            "anio_publicacion": libro[4],
            "cantidad": libro[5]
        })
    cursor.close()
    return jsonify(libros)


@app.route('/libros/<int:id>', methods=['GET'])
def obtener_libro(id):  
    cursor = conexion.cursor()
    cursor.execute("SELECT id_libro, titulo, autor, editorial, anio_publicacion, cantidad FROM libros WHERE id_libro=%s", (id,))
    libro = cursor.fetchone()
    cursor.close()
    if libro:
        return jsonify({
            "id_libro": libro[0],
            "titulo": libro[1],
            "autor": libro[2],
            "editorial": libro[3],
            "anio_publicacion": libro[4],
            "cantidad": libro[5]
        })
    return jsonify({"error": "Libro no encontrado"}), 404




@app.route('/libros', methods=['POST'])
def crear_libro():  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        'INSERT INTO libros(titulo, autor, editorial, anio_publicacion, cantidad) VALUES(%s, %s, %s, %s, %s) RETURNING id_libro',
        (datos['titulo'], datos['autor'], datos['editorial'], datos.get('anio_publicacion'), datos['cantidad'])
    )
    nuevo_id = cursor.fetchone()[0]
    conexion.commit()
    cursor.close()
    return jsonify({
        'id_libro': nuevo_id,
        'titulo': datos['titulo'],
        'autor': datos['autor'],
        'editorial': datos['editorial'],
        'anio_publicacion': datos.get('anio_publicacion'),
        'cantidad': datos['cantidad']
    }), 201


@app.route('/libros/<int:id>', methods=['PUT'])
def actualizar_libro(id):  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        'UPDATE libros SET titulo=%s, autor=%s, editorial=%s, anio_publicacion=%s, cantidad=%s WHERE id_libro=%s',
        (datos['titulo'], datos['autor'], datos['editorial'], datos['anio_publicacion'], datos['cantidad'], id)
    )
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Libro actualizado correctamente'})


@app.route('/libros/<int:id>', methods=['DELETE'])
def eliminar_libro(id):  
    cursor = conexion.cursor()
    cursor.execute('DELETE FROM libros WHERE id_libro=%s', (id,))
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Libro eliminado correctamente'})

@app.route('/personas', methods=['GET'])
def obtener_personas():  
    nombre_buscar = request.args.get('nombre')
    cursor = conexion.cursor()
    
    if nombre_buscar:
        cursor.execute("SELECT persona, nombre, apellido, fecha_nacimiento, vigente, salario FROM personas WHERE nombre ILIKE %s", (f"%{nombre_buscar}%",))
    else:
        cursor.execute("SELECT persona, nombre, apellido, fecha_nacimiento, vigente, salario FROM personas")
        
    datos = cursor.fetchall()
    personas = []
    for p in datos:
        personas.append({
            "persona": p[0],
            "nombre": p[1],
            "apellido": p[2],
            "fecha_nacimiento": str(p[3]) if p[3] else None, 
            "vigente": p[4],
            "salario": p[5]
        })
    cursor.close()
    
    if not personas and nombre_buscar:
        return jsonify({"error": f"No se encontraron personas con el nombre '{nombre_buscar}'"}), 404
        
    return jsonify(personas)

@app.route('/personas/<int:id>', methods=['GET'])
def obtener_persona_por_id(id):  
    cursor = conexion.cursor()
    cursor.execute("SELECT persona, nombre, apellido, fecha_nacimiento, vigente, salario FROM personas WHERE persona=%s", (id,))
    p = cursor.fetchone()
    cursor.close()
    if p:
        return jsonify({
            "persona": p[0],
            "nombre": p[1],
            "apellido": p[2],
            "fecha_nacimiento": str(p[3]) if p[3] else None,
            "vigente": p[4],
            "salario": p[5]
        })
    return jsonify({"error": "Persona no encontrada por ID"}), 404


@app.route('/personas/buscar/<string:nombre>', methods=['GET'])
def obtener_personas_por_nombre(nombre):
    cursor = conexion.cursor()
    cursor.execute("SELECT persona, nombre, apellido, fecha_nacimiento, vigente, salario FROM personas WHERE nombre LIKE %s", (f"%{nombre}%",))
    datos = cursor.fetchall()
    cursor.close()
    
    if not datos:
        return jsonify({"error": f"No se encontraron personas con el nombre '{nombre}'"}), 404
        
    personas = []
    for p in datos:
        personas.append({
            "persona": p[0],
            "nombre": p[1],
            "apellido": p[2],
            "fecha_nacimiento": str(p[3]) if p[3] else None, 
            "vigente": p[4],
            "salario": p[5]
        })
        
    return jsonify(personas)


@app.route('/personas', methods=['POST'])
def crear_persona():  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        'INSERT INTO personas(nombre, apellido, fecha_nacimiento, vigente, salario) VALUES(%s, %s, %s, %s, %s) RETURNING persona',
        (datos['nombre'], datos['apellido'], datos.get('fecha_nacimiento'), datos.get('vigente', True), datos.get('salario'))
    )
    nuevo_id = cursor.fetchone()[0]
    conexion.commit()
    cursor.close()
    return jsonify({
        'persona': nuevo_id,
        'nombre': datos['nombre'],
        'apellido': datos['apellido'],
        'fecha_nacimiento': datos.get('fecha_nacimiento'),
        'vigente': datos.get('vigente', True),
        'salario': datos.get('salario')
    }), 201


@app.route('/personas/<int:id>', methods=['PUT'])
def actualizar_persona(id):  
    datos = request.get_json()
    cursor = conexion.cursor()
    cursor.execute(
        'UPDATE personas SET nombre=%s, apellido=%s, fecha_nacimiento=%s, vigente=%s, salario=%s WHERE persona=%s',
        (datos['nombre'], datos['apellido'], datos['fecha_nacimiento'], datos['vigente'], datos['salario'], id)
    )
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Persona actualizada correctamente'})


@app.route('/personas/<int:id>', methods=['DELETE'])
def eliminar_persona(id):  
    cursor = conexion.cursor()
    cursor.execute('UPDATE personas SET vigente=FALSE WHERE persona=%s', (id,))
    conexion.commit()
    cursor.close()
    return jsonify({'mensaje': 'Persona dada de baja (vigente = False) correctamente'})

if __name__ == '__main__':
    app.run(debug=True)
