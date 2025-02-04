# Taller Grupal 2: Persistencia de datos en archivos - Base de datos

## 1. Generar el archivo CSV

<img width="315" alt="image" src="https://github.com/user-attachments/assets/e86f3968-8a4b-48cb-9bba-43ef89fe1059" />

## 2. Genere la tabla en MYSQL para esta información.

<img width="271" alt="image" src="https://github.com/user-attachments/assets/771c6b84-00f8-46a2-b01e-29af7e89178d" />

## 3. Elabore un programa que inyecte los datos del archivo CSV a la base de datos. 

### 1. Lectura del archivo CSV:

En el objeto Main, se define la ruta del archivo CSV con:

```scala
val path2DataFile2 = "src/main/resources/data/estudiantes.csv"
```

Luego, el archivo CSV es leído usando la librería kantan.csv, que proporciona una forma sencilla de leer archivos CSV y mapearlos a objetos de Scala. La función readCsv de kantan.csv se usa para leer el archivo y mapear cada fila a un objeto Estudiantes:

```scala
val dataSource = new File(path2DataFile2)
  .readCsv[List, Estudiantes](rfc.withHeader.withCellSeparator(','))
```

 - rfc.withHeader: Indica que el archivo CSV tiene una fila de cabecera que debe ser ignorada al leer los datos.
 - .withCellSeparator(','): Define que el separador de celdas en el CSV es una coma.

Después de leer el archivo, se recoge el resultado que es una lista de objetos Estudiantes:

```scala

val estudiantes = dataSource.collect {
  case Right(estudiantes) => estudiantes
}
```

 - Right(estudiantes) representa la lectura exitosa de una fila en el CSV que se puede mapear correctamente al tipo Estudiantes.
 - En caso de error, Left(error) no se maneja explícitamente en el código, pero podrías tratarlo si lo deseas.

### 2. Inserción de datos en la base de datos:

En la función run, después de haber leído los datos del archivo CSV y almacenado en la lista estudiantes, el siguiente paso es insertar estos datos en la base de datos. Esto se logra mediante el método insertAll de EstudianteDAO, que acepta una lista de objetos Estudiantes:

```scala
inserted <- EstudianteDAO.insertAll(estudiantes)
```

La función insertAll de EstudianteDAO:

```scala
def insertAll(estudiantes: List[Estudiantes]): IO[List[Int]] = {
  Database.transactor.use { xa =>
    estudiantes.traverse(t => insert(t).transact(xa))
  }
}
```

 - traverse es una operación de cats que mapea los elementos de la lista a una acción que devuelve una lista de resultados. En este caso, mapea cada Estudiantes a la acción de insertar (insert(t)), que luego se ejecuta en la base de datos usando transact(xa).
 - transactor es el recurso de conexión a la base de datos proporcionado por Database.transactor, que maneja la transacción.
 - El método insert realiza la inserción de un solo objeto Estudiantes en la base de datos mediante una consulta SQL (INSERT INTO estudiantes...):

```scala
def insert(estudiantes: Estudiantes): ConnectionIO[Int] = {
  sql"""
    INSERT INTO estudiantes (nombre, edad, calificacion, genero)
    VALUES (
      ${estudiantes.nombre},
      ${estudiantes.edad},
      ${estudiantes.calificacion},
      ${estudiantes.genero}
    )
  """.update.run
}
```

.update.run ejecuta la consulta de inserción y devuelve un Int que indica el número de filas afectadas (en este caso, debería ser 1 por cada inserción exitosa).

### 3. Confirmación de la inserción:

Después de insertar los estudiantes en la base de datos, se imprime el número de registros insertados:

```scala
_ <- IO.println(s"Registros insertados: ${inserted.size}")
```

#### Capturas de pantalla:

<img width="224" alt="image" src="https://github.com/user-attachments/assets/06c58816-0ff6-43f9-b38b-9893504fb9d2" />

<img width="224" alt="image" src="https://github.com/user-attachments/assets/0c46b44c-10be-459f-86cd-37a09ceac9d1" />

## 4. En el mismo programa agregue la funcionalidad para obtener de la base de datos todos los registros de Estudiantes. 

Para verificar que los datos fueron insertados correctamente, el programa obtiene todos los registros de la base de datos con la función getAll de EstudianteDAO:

```scala
allEstudiantes <- EstudianteDAO.getAll
```

 - getAll ejecuta una consulta SQL (SELECT nombre, edad, calificacion, genero FROM estudiantes) y mapea los resultados a una lista de objetos Estudiantes.

Finalmente, los registros actuales en la base de datos se imprimen en consola:

```scala
_ <- IO.println("Registros actuales en la base de datos:")
_ <- IO.println(allEstudiantes.mkString("\n"))
```
#### Captura de pantalla:

<img width="334" alt="image" src="https://github.com/user-attachments/assets/cbe67ccf-1ea1-49de-80d0-52e4adb41d80" />
