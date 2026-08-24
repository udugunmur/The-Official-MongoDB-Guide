# Parte 5: Consultas

## Capítulo 5: Consultas

MongoDB ofrece un sistema de consultas potente y flexible diseñado para trabajar con datos jerárquicos y semiestructurados almacenados en formato BSON. Como base de datos NoSQL orientada a documentos, MongoDB permite consultas expresivas y dinámicas que van mucho más allá de las simples búsquedas de clave-valor. Su motor de consultas está optimizado para una amplia gama de cargas de trabajo, incluido el acceso a datos operativos, el procesamiento analítico y los casos de uso de aplicaciones en tiempo real.

En su base, MongoDB admite las operaciones estándar de creación, lectura, actualización y eliminación (CRUD), ejecutadas a través de una sintaxis intuitiva similar a JSON que facilita el trabajo con matrices y documentos anidados. Las consultas se pueden componer utilizando operadores a nivel de campo, expresiones lógicas, expresiones regulares y criterios geoespaciales, entre otros.

Este capítulo cubre los siguientes temas:

- Operaciones CRUD de MongoDB
- Operaciones CRUD con mongosh y PyMongo
- Operaciones por lotes (*Batch operations*) con mongosh
- Consultas avanzadas y agregaciones
- Formas de consulta (*Query shapes*)
- Comandos de base de datos introducidos en MongoDB

---

### Sección 1: Requisitos técnicos

Para seguir el código de este capítulo, debe conectarse a un despliegue local (*on-premises*) o a una base de datos de MongoDB Atlas.

Para conectarse a un despliegue local, debe instalar MongoDB localmente. Puede descargar MongoDB Community Edition para entornos locales desde www.mongodb.com, de forma gratuita. Como alternativa a MongoDB Community Edition, la base de datos como servicio (DBaaS) totalmente administrada, MongoDB Atlas, también cuenta con un nivel gratuito, así como con la oportunidad de actualizar sin problemas.

Los ejemplos de código de este capítulo utilizan `mongosh` y `PyMongo`:

- `mongosh` es un entorno REPL de JavaScript y Node.js para interactuar con despliegues de MongoDB. Puede descargar `mongosh` desde [https://www.mongodb.com/docs/mongodb-shell/](https://www.mongodb.com/docs/mongodb-shell/).
- `PyMongo` es el controlador oficial de MongoDB para Python. Puede descargar `PyMongo` u otro controlador de MongoDB de su elección en [https://www.mongodb.com/docs/drivers/](https://www.mongodb.com/docs/drivers/).

---

### Sección 2: Operaciones CRUD en MongoDB

Las operaciones CRUD son la base de la interacción con un despliegue de MongoDB. Estas operaciones requieren que se conecte al servidor de MongoDB antes de poder consultar los documentos relevantes, ajustar las propiedades especificadas y, posteriormente, transmitir los datos de regreso a la base de datos para las actualizaciones. Cada operación CRUD cumple un propósito distinto:

- La operación de **creación (*create*)** crea e inserta nuevos documentos en la base de datos.
- La operación de **lectura (*read*)** consulta un documento o documentos en la base de datos.
- La operación de **actualización (*update*)** modifica documentos existentes en la base de datos.
- La operación de **eliminación (*delete*)** elimina documentos de la base de datos.

#### CRUD básico con mongosh

`mongosh`, también conocido como MongoDB Shell, es un entorno para interactuar con despliegues de MongoDB. Es equivalente a la consola de administración que utilizan las bases de datos relacionales. Puede descargar `mongosh` en [https://www.mongodb.com/docs/mongodb-shell/install/](https://www.mongodb.com/docs/mongodb-shell/install/).

Esta sección explica cómo conectarse a su despliegue con `mongosh`, insertar un documento en una base de datos, consultar su base de datos para un documento específico, actualizar un documento y eliminar un documento. Para obtener información y ejemplos más detallados, también puede consultar la sección de operaciones CRUD del manual de la base de datos de MongoDB en [https://www.mongodb.com/docs/manual/crud/](https://www.mongodb.com/docs/manual/crud/).

#### Conectarse a MongoDB

Antes de poder realizar operaciones CRUD, debe conectarse a su despliegue de MongoDB. Para conectarse a `mongosh`, debe especificar la cadena de conexión de su despliegue así como cualquier parámetro especificado.

Por ejemplo, ingrese una variación del siguiente bloque de código en su terminal para conectarse a un despliegue de MongoDB con `mongosh`:

```bash
mongosh "mongodb+srv://mycluster.packt.mongodb.net/myDatabase"
--username myUsername --password myPassword
```

#### Crear documentos

Para crear un documento en `mongosh`, puede utilizar el comando `db.collection.insertOne()`. Este comando de `mongosh` crea un nuevo documento e inserta el documento creado en la colección especificada.

Por ejemplo, el siguiente comando `db.collection.insertOne()` crea e inserta un nuevo documento en la colección `library` de la base de datos. El nuevo documento tiene un campo `title` establecido en `Mastering MongoDB 8.0` y un campo `isbn` establecido en `101`:

```javascript
db.library.insertOne(
  { title: 'Mastering MongoDB 8.0', isbn: '101' }
)
```

Cuando crea e inserta con éxito un documento, MongoDB devuelve una salida de confirmación que contiene el valor `ObjectId` del nuevo documento:

```json
{
  acknowledged: true,
  insertedId: ObjectId("652024e7ab44f3bf77788a3d")
}
```

También puede utilizar los siguientes comandos de `mongosh` para insertar o actualizar documentos:

- `db.collection.insertMany()`
- `db.collection.updateOne()` con la opción `upsert: true`
- `db.collection.updateMany()` con la opción `upsert: true`
- `db.collection.findAndModify()` con la opción `upsert: true`
- `db.collection.findOneAndUpdate()` con la opción `upsert: true`
- `db.collection.findOneAndReplace()` con la opción `upsert: true`

#### Leer documentos

Las operaciones de lectura en MongoDB también se denominan consultas. Para realizar una operación de lectura básica, o consulta, para un solo documento, utilice el método `db.collection.findOne()` y especifique los criterios de selección del documento que desea leer.

Por ejemplo, la siguiente operación consulta un documento en la colección `library` que contiene el valor de campo `isbn` de `101`:

```javascript
db.library.find( { isbn: '101' } )
```

Si la colección `library` contiene un documento que coincide con sus criterios de selección especificados, MongoDB devuelve una matriz que contiene el documento coincidente:

```json
[ {  
  _id: ObjectId("652024e7ab44f3bf77788a3d"),
  title: 'Mastering MongoDB 8.0',
  isbn: '101'
} ]
```

También puede utilizar el siguiente comando de `mongosh` para leer documentos:

```javascript
db.collection.findOne()
```

#### Actualizar documentos

Para actualizar un documento, puede utilizar el comando `db.collection.updateOne()`. Este comando busca un documento que coincida con los criterios especificados y actualiza el documento encontrado.

Por ejemplo, el siguiente código busca el documento en la colección `library` que tiene un valor de campo `isbn` de `101` y actualiza ese documento para que contenga un campo `price` con un valor de `30`:

```javascript
db.library.updateOne( { isbn: '101' }, { $set: { price: 30 } } )
```

Cuando actualiza exitosamente un documento, MongoDB devuelve el siguiente resumen de confirmación:

```json
{
  acknowledged: true,
  insertedId: null,
  matchedCount: 1,
  modifiedCount: 1,
  upsertedCount: 0
}
```

También puede utilizar los siguientes comandos de `mongosh` para actualizar documentos:

- `db.collection.updateMany()`
- `db.collection.replaceOne()`
- `db.collection.findOneAndReplace()`
- `db.collection.findOneAndUpdate()`
- `db.collection.findAndModify()`

MongoDB 8.0 también introduce la capacidad de ordenar documentos dentro de una operación de actualización. Si especifica un parámetro `sort` en una operación que utiliza `db.collection.updateOne()`, `db.collection.replaceOne()` o `update`, MongoDB ordena los documentos antes de actualizarlos para poder seleccionar un documento específico para la operación cuando varios documentos coinciden con la consulta.

Por ejemplo, la siguiente operación de actualización ordena todos los libros por precio y actualiza el libro de menor precio para que esté en oferta con un 25% de descuento:

```javascript
db.library.updateOne(  
  { },
  [
    { $set: { price: { $multiply: ["$price", 0.75] } } }
  ],
  { sort: { price: 1 } }
)
```

#### Eliminar documentos

Para eliminar un documento, puede utilizar el comando `db.collection.deleteOne()`. Este comando consulta un documento que coincide con criterios especificados y elimina el documento encontrado.

Por ejemplo, la siguiente operación elimina un documento en la colección `library` que tiene un valor de campo `isbn` de `101`:

```javascript
db.library.deleteOne( { isbn: '101' } )
```

Cuando elimina exitosamente un documento, MongoDB devuelve la siguiente salida:

```json
{  
  acknowledged: true,
  deletedCount: 1
}
```

También puede utilizar los siguientes comandos de `mongosh` para eliminar documentos:

- `db.collection.deleteMany()`
- `db.collection.remove()`
- `db.collection.findOneAndDelete()`
- `db.collection.findAndModify()`

#### CRUD básico con el controlador de Python

Como alternativa a `mongosh`, MongoDB proporciona controladores oficiales para interactuar con su despliegue de MongoDB. Para obtener una lista completa de las bibliotecas de controladores oficiales de MongoDB, consulte [https://www.mongodb.com/docs/drivers/](https://www.mongodb.com/docs/drivers/).

Las siguientes secciones le guiarán a través de la realización de operaciones CRUD con el controlador de MongoDB para Python, PyMongo. PyMongo proporciona un puente fluido entre el dinámico mundo de la programación en Python y la eficiente base de datos NoSQL orientada a documentos de MongoDB.

#### Instalar y conectarse a PyMongo

Puede usar `pip` para instalar PyMongo. Debe tener Python instalado antes de instalar PyMongo.

Por ejemplo, la siguiente operación instala PyMongo a través de su terminal:

```bash
$ python -m pip install pymongo
```

Si su máquina utiliza Python 3, reemplace `python` en su comando de instalación por `python3`.

Para conectarse a su despliegue de MongoDB con PyMongo, debe utilizar su cadena de conexión. Por ejemplo, el siguiente fragmento de código en un archivo `.py` se conecta a un despliegue de MongoDB:

```python
from pymongo import MongoClient
uri = "<connection string>"
client = MongoClient(uri)
try:
    client.admin.command('ping')
    print("Pinged your deployment. You successfully connected to MongoDB!")
except Exception as e:
    print(e)
```

Si ejecuta el código anterior en un archivo `.py` y se conecta exitosamente a su despliegue, MongoDB devuelve la siguiente confirmación:

```text
"Pinged your deployment. You successfully connected to MongoDB!"
```

Puede probar su conexión a su despliegue de MongoDB en Atlas con el siguiente bloque de código en un archivo `.py`. Este bloque de código utiliza el marco asíncrono `asyncio`:

```python
import asyncio
from motor.motor_asyncio import AsyncIOMotorClient
from pymongo.server_api import ServerApi
async def ping_server():
    uri = "<connection string>"
    client = AsyncIOMotorClient(uri, server_api=ServerApi('1'))
    try:
        await client.admin.command('ping')
        print("Pinged your deployment. You successfully connected to MongoDB!")
    except Exception as e:
        print(e)
asyncio.run(ping_server())
```

Si ejecuta el código anterior en un archivo `.py` y se conecta exitosamente a su despliegue, MongoDB devuelve la siguiente confirmación:

```text
"Pinged your deployment. You successfully connected to MongoDB!".
```

#### Crear documentos

Para crear e insertar documentos con PyMongo, utilice el comando `insert_one`.

Por ejemplo, el bloque de código que sigue realiza las siguientes operaciones:

1. Especifica la colección `library` dentro de la base de datos `resources`.
2. Define un nuevo documento que representa un libro titulado *Python and MongoDB*.
3. Inserta el nuevo documento en la base de datos `library`.
4. Imprime el valor `ObjectId` del nuevo documento.

```python
library = client.resources.library
book = {
  'isbn': '301',
  'name': 'Python and MongoDB',
  'meta': {'version': 'MongoDB 8.0'},
  'price': 60  
}
insert_result = library.insert_one(book)
print(insert_result)
```

Si inserta un documento exitosamente con este script, MongoDB imprime el valor `ObjectId` del documento insertado.

#### Leer documentos

Puede leer documentos de su base de datos mediante consultas.

Para leer un documento, especifique la(s) condición(es) del documento que desea leer con un comando `find_one`.

Por ejemplo, el bloque de código que sigue realiza las siguientes acciones:

1. Especifica la colección `library` en la base de datos `resources`.
2. Busca documentos con el valor `name` de *Python and MongoDB*.
3. Imprime el documento encontrado.

```python
library = client.resources.library
result = library.find_one( {"name": "Python and MongoDB"} )
print (result)
```

#### Actualizar documentos

Para actualizar un documento con PyMongo, utilice el comando `update_one`. Este comando busca un documento según los criterios especificados y actualiza el documento encontrado.

Por ejemplo, el bloque de código que sigue realiza las siguientes operaciones:

1. Busca el documento donde el campo `name` es *Advanced MongoDB Techniques*.
2. Establece el campo `price` del documento encontrado en el valor de `75`.
3. Imprime el resultado de la operación `update_one`.
4. Imprime el documento actualizado.

```python
update_result = library.update_one(
  { "name": "Advanced MongoDB Techniques"},
  { "$set": { "price": 75 } }
)
print(update_result.raw_result)
updated_document = library.find_one(
  {"name": "Advanced MongoDB Techniques"}
)
print(updated_document)
```

#### Eliminar un documento

Para eliminar un documento con PyMongo, utilice el comando `delete_one`. Por ejemplo, el bloque de código que sigue realiza las siguientes operaciones:

1. Especifica el valor `isbn` del libro que se eliminará como `303`.
2. Utiliza el comando `delete_one` para eliminar un libro de la colección `library` con el valor de `isbn` deseado.
3. Imprime el resultado de la operación `delete_one`.

```python
isbn_to_delete = '303'
delete_result = library.delete_one({"isbn": isbn_to_delete})
print(delete_result.raw_result)
```

Si la operación eliminó exitosamente el documento deseado, MongoDB devuelve la siguiente confirmación:

```json
{'n': 1, 'ok': 1.0}
```

El valor `n` indica el número de documentos que MongoDB eliminó. En este caso, MongoDB eliminó un documento. El valor `ok` indica si la operación causó algún error. El valor `1.0` en la confirmación anterior significa que MongoDB no encontró ningún error con esta operación.

#### Operaciones masivas (*Bulk operations*)

Para realizar operaciones CRUD más complejas, MongoDB 8.0 introduce un nuevo comando de base de datos `bulkWrite` que puede crear, actualizar y eliminar operaciones en múltiples colecciones en una sola solicitud. `bulkWrite` amplía la funcionalidad del comando existente `db.collection.bulkWrite()`, que solo le permite modificar una colección a la vez.

#### Sintaxis

El comando `bulkWrite` toma los siguientes campos comúnmente utilizados:

| Campo | Tipo | Necesidad | Descripción |
| :--- | :--- | :--- | :--- |
| `insert` | Integer | Requerido | Índice de ID de espacio de nombres (*Namespace ID index*) para una operación de inserción, que debe coincidir con el índice de ID de espacio de nombres en el campo `ns` de la matriz `nsInfo`. Los índices comienzan en 0. |
| `document` | Document | Requerido | Documento a insertar en la colección. |
| `update` | Integer | Requerido | Índice de ID de espacio de nombres para una operación de actualización, que debe coincidir con un índice de ID de espacio de nombres en el campo `ns` de la matriz `nsInfo`. Los índices comienzan en 0. |
| `filter` | Document | Opcional | Selector de consulta para limitar los documentos para la operación de actualización o eliminación. |

> **Tabla 5.1:** Campos comunes en el comando bulkWrite

Consulte la página de documentación de MongoDB sobre `bulkWrite` para obtener una lista de todos los parámetros de `bulkWrite`, ejemplos detallados y consideraciones de comportamiento: [https://www.mongodb.com/docs/manual/reference/command/bulkWrite/](https://www.mongodb.com/docs/manual/reference/command/bulkWrite/).

Existen algunas consideraciones adicionales para el comando `bulkWrite`, como se analiza en las siguientes subsecciones.

#### El campo multi y escrituras reintentables (*Retryable writes*)

`bulkWrite` puede tomar un campo opcional `multi` que especifica si una operación de actualización o eliminación afecta a todos los documentos que coinciden con el filtro de documentos. Si establece `multi` en `true`, la operación actualiza o elimina todos los documentos que coinciden con el filtro. Si `multi` es `false`, MongoDB solo actualiza o elimina el primer documento que coincide con el filtro.

Puede habilitar escrituras reintentables (*retryable writes*) para reintentar automáticamente operaciones de escritura que puedan encontrar errores de red o no puedan encontrar un primario en buen estado al que dirigirse. Las escrituras reintentables son compatibles con algunas operaciones de `bulkWrite`:

- Puede habilitar escrituras reintentables con operaciones de inserción `bulkWrite` que establecen `multi` en `true`.
- Si `multi` es `true`, no puede habilitar escrituras reintentables con operaciones de actualización o eliminación de `bulkWrite`.
- Si `multi` es `false`, puede habilitar escrituras reintentables con operaciones de actualización o eliminación de `bulkWrite`.

#### Rendimiento

Las operaciones `bulkWrite` que insertan documentos pueden producir una única entrada de registro de operaciones (o entrada de *oplog*) para múltiples operaciones de inserción. En consecuencia, `bulkWrite` mejora el rendimiento de las inserciones de múltiples documentos y elimina posibles retrasos en la replicación que normalmente son el resultado de múltiples escrituras en el *oplog*. Sin embargo, el apagado de una base de datos inmediatamente después de una operación de inserción `bulkWrite` puede llevar más tiempo debido a los datos adicionales que se vacían en el disco.

En general, utilizar un comando `bulkWrite` para múltiples operaciones mejora el rendimiento de la base de datos y reduce la cantidad de viajes de ida y vuelta (*round trips*) al servidor de la base de datos. `bulkWrite` puede ser especialmente óptimo para aplicaciones de alto rendimiento (*high-throughput*).

---

### Sección 3: Agregaciones en MongoDB

MongoDB ofrece un amplio conjunto de funciones de consulta avanzadas que se extienden más allá de la recuperación y manipulación básica de datos. Estas características le permiten optimizar el rendimiento de las consultas, crear canalizaciones de agregación (*aggregation pipelines*) y realizar agregaciones complejas.

#### Introducción al marco de agregación

El marco de agregación (*aggregation framework*) de MongoDB es una herramienta de procesamiento de datos para realizar cálculos y transformaciones de datos complejos. Puede utilizar el marco para filtrar, transformar y analizar datos dentro de MongoDB.

El marco de agregación se centra en las canalizaciones de agregación, que son canalizaciones de procesamiento de datos que reciben datos y los transforman a medida que pasan de una etapa de agregación a la siguiente. Con diferentes etapas de agregación, puede filtrar, agrupar y remodelar documentos completos. Las etapas de agregación están representadas por operadores precedidos por `$`. Por ejemplo, una etapa de agregación `$group` agrupa documentos según parámetros especificados.

El siguiente diagrama ilustra cómo fluyen los datos a través de una canalización de agregación que contiene las etapas de agregación `$match`, `$sort`, `$group` y `$set`:

> **Figura 5.1:** Un diagrama que ilustra cómo se pasan los datos a través de una canalización de agregación

Con más de 150 operadores y expresiones, el ecosistema de MongoDB ofrece una amplia gama de posibilidades, lo que le permite realizar operaciones de datos increíblemente complejas y útiles mediante pasos relativamente simples. Todos los controladores específicos de lenguaje de MongoDB (JavaScript, Python, etc.) son totalmente compatibles con el marco de agregación, lo que hace que le resulte conveniente escribir agregaciones en cualquier lenguaje. Los ejemplos de esta sección muestran canalizaciones de agregación que utilizan `mongosh`.

#### Ejemplo de canalización de agregación

Una universidad podría utilizar una canalización de agregación para calcular la calificación final de cada estudiante. Considere una base de datos universitaria que contiene una colección `student_grades`. Cada documento de la colección tiene la siguiente estructura de datos:

```json
{
  "_id": ObjectId("5f4b7de8e8189a46aaf6e3ad"),
  "student_id": ObjectId("5f4b7de8e7139a46aaf5e4a5"),
  "course_id": ObjectId("5f4b7de8e7179a56aaf6e1b2"),
  "grade": 76,
  "semester": "Fall 2025"
}
```

Para calcular las calificaciones finales utilizando múltiples pasos, la universidad puede utilizar una canalización de agregación para realizar las siguientes operaciones:

1. **Filtrar calificaciones:** Si la universidad desea especificar que las calificaciones finales solo tomen en cuenta ciertos semestres, pueden usar una etapa de agregación `$match` para filtrar las calificaciones por el campo `semester`.
2. **Agrupar calificaciones:** Para agrupar documentos por `student_id`, pueden usar una etapa de agregación `$group`.
3. **Calcular la calificación promedio:** Para calcular la calificación promedio de las calificaciones filtradas por estudiante, pueden usar una etapa de agregación `$project`.

La siguiente canalización de agregación logra estas operaciones utilizando `mongosh`:

```javascript
db.student_grades.aggregate( [
  { $match: { semester: "Fall 2025" } },
  { $group: {
          _id: "$student_id",
          averageGrade: { $avg: "$grade" }
      } },
    { $project: {
            _id: 0,
            student_id: "$_id",
            averageGrade: 1
    } }
] )
```

El resultado de esta canalización de agregación muestra la calificación promedio de cada estudiante, donde cada estudiante se identifica por su ID de estudiante.

#### Beneficios del marco de agregación

El marco de agregación de MongoDB proporciona una forma sólida de procesar y analizar sus datos. Le permite realizar transformaciones complejas, cálculos y combinaciones de documentos dentro de sus colecciones. Esto facilita la extracción de conocimientos, el resumen de información y el respaldo de consultas analíticas directamente dentro de la base de datos.

La agregación en MongoDB proporciona una variedad de beneficios, tales como los siguientes:

- **Rendimiento:** Las operaciones nativas de base de datos dentro de MongoDB son a menudo más rápidas que extraer datos y procesarlos con una herramienta externa.
- **Flexibilidad:** El marco de agregación proporciona una amplia gama de herramientas y operadores para transformar datos de formas complejas, lo que a menudo reduce la necesidad de lógica de procesamiento de datos externa.
- **Integridad de los datos:** El procesamiento de datos a nivel de base de datos garantiza la coherencia y la integridad en comparación con los procesos externos, que pueden encontrar problemas transaccionales o de sincronización.

#### Etapas de agregación

MongoDB proporciona una amplia gama de etapas de agregación que procesan y generan datos. Una etapa de agregación representa un paso específico en la canalización donde se lleva a cabo el proceso de transformación de datos. Cada etapa procesa datos y genera documentos que pueden ser procesados posteriormente por las etapas subsiguientes.

Una lista completa y actualizada de todas las etapas de agregación de MongoDB está disponible en el sitio web de documentación de MongoDB: [https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/](https://www.mongodb.com/docs/manual/reference/operator/aggregation-pipeline/).

#### Compatibilidad

Puede utilizar etapas de canalización de agregación en los siguientes entornos:

- **MongoDB Atlas:** El servicio totalmente administrado para despliegues de MongoDB en la nube.
- **MongoDB Enterprise:** La versión autogestionada basada en suscripción de MongoDB.
- **MongoDB Community:** La versión autogestionada, gratuita y de código disponible de MongoDB.

#### Etapas de agregación comunes

La siguiente lista muestra las etapas de agregación comúnmente utilizadas:

- **`$match`:** Filtra documentos y envía el conjunto de documentos filtrados a la siguiente etapa de la canalización.
- **`$limit`:** Limita la cantidad de documentos enviados a la siguiente etapa.
- **`$sort`:** Ordena los documentos según un orden especificado.
- **`$project`:** Modifica la forma de los datos para generar solo los campos especificados.
- **`$set`:** Especifica un nuevo campo para agregar a los documentos para la nueva etapa.
- **`$unwind`:** Aplana campos de matriz y genera un nuevo documento para cada elemento de la matriz.
- **`$lookup`:** Realiza una unión externa izquierda (*left outer join*) con otra colección.

#### Mejoras en las etapas de agregación de MongoDB 8.0

MongoDB 8.0 realiza varias mejoras y optimizaciones en la oferta de etapas de agregación de MongoDB.

#### $convert

La etapa de agregación `$convert` convierte un valor a un tipo de datos especificado. El campo de entrada `input` de una etapa de agregación `$convert` puede ser cualquier expresión válida. El campo `to` especifica el tipo al que MongoDB convierte el valor de entrada. Para obtener una lista completa de los parámetros de `$convert`, consulte la documentación de MongoDB en [https://www.mongodb.com/docs/manual/reference/operator/aggregation/convert/](https://www.mongodb.com/docs/manual/reference/operator/aggregation/convert/).

Por ejemplo, la siguiente etapa de agregación convierte el campo `$price` de los documentos de una colección de inventario en una cadena de texto (*string*):

```javascript
db.inventory.aggregate( [
  {
    $set: {
      price: {
        $convert: {
          input: "$price",
          to: "string",  
        }
      }
    }
  }
] )
```

MongoDB 8.0 amplía las capacidades de `$convert` para realizar las siguientes conversiones, que anteriormente no estaban disponibles:

- Valores `string` a valores `binData`
- Valores `binData` a valores `string`

Además, MongoDB 8.0 introduce una nueva expresión auxiliar, `$toUUID`, que proporciona una sintaxis más simple para convertir cadenas a valores UUID.

Para obtener una lista completa de las capacidades de conversión, consulte la documentación de MongoDB.

#### $denseRank y $rank

Los operadores de canalización de agregación `$denseRank` y `$rank` devuelven la posición del documento (rango) en relación con otros documentos dentro de una ventana (rango de documentos) especificada por una operación `$setWindowFields`.

Por ejemplo, considere una colección `sales` que contiene documentos que representan ventas individuales y el estado en el que se realizó la venta. La siguiente etapa de agregación utiliza `$rank` dentro de la etapa de agregación `$setWindowFields` para clasificar la cantidad de ventas para cada estado:

```javascript
db.sales.aggregate( [
   {
      $setWindowFields: {
         partitionBy: "$state",
         sortBy: { quantity: -1 },
         output: {
            rankQuantityForState: {
               $rank: {}
            }
         }
      }
   }
] )
```

Si bien `$denseRank` y `$rank` tienen varias diferencias en la forma en que clasifican los documentos, a partir de MongoDB 8.0, `$denseRank` y `$rank` tratan los valores de campo nulos y faltantes de manera equivalente al calcular las clasificaciones.

#### $queryStats

MongoDB 7.1 introduce la etapa de agregación `$queryStats`, que devuelve estadísticas de tiempo de ejecución para las consultas registradas en despliegues de Atlas. `$queryStats` devuelve métricas para consultas `aggregate()`, `find()` y `distinct()` en la base de datos `admin` y debe ser la primera etapa en una canalización de agregación.

La etapa de agregación `$queryStats` no es compatible con todas las versiones de MongoDB y no se garantiza que sea estable en una versión futura del servidor.

`$queryStats` genera una matriz de entradas de estadísticas de consultas. Algunas propiedades de entrada de estadísticas de consulta contienen valores literales y MongoDB normaliza otras propiedades para agrupar consultas similares. MongoDB normaliza las propiedades de `$queryStats` por campos proporcionados por el usuario a sus tipos de datos. Por ejemplo, MongoDB normaliza un filtro `{ item: 'card' }` a `{ item: '?string' }`.

Cada entrada de estadísticas de consulta en la salida contiene los siguientes documentos de nivel superior:

| Campo del Documento | Descripción |
| :--- | :--- |
| `key` | La combinación única de atributos que definen una entrada, como la forma de la consulta, la información del cliente, el *read concern* o el tipo de colección. |
| `as0f` | La hora, en UTC, en que `$queryStats` leyó la entrada correspondiente. |
| `metrics` | Métricas de tiempo de ejecución agregadas de la entrada de estadísticas de consulta correspondiente. |

> **Tabla 5.2:** Campos de nivel superior en la salida de $queryStats

Para obtener descripciones más detalladas de los documentos anidados en la matriz de salida, consulte la documentación completa de MongoDB: [https://www.mongodb.com/docs/manual/reference/operator/aggregation/queryStats/](https://www.mongodb.com/docs/manual/reference/operator/aggregation/queryStats/).

Por ejemplo, la siguiente canalización de agregación genera estadísticas de consulta en la base de datos `admin`:

```javascript
db.getSiblingDB("admin").aggregate( [
   {
      $queryStats: { }
   }
] )
```

#### Canalizaciones de agregación avanzadas

Si bien los ejemplos anteriores de este capítulo utilizan canalizaciones de agregación de una sola etapa para demostrar las funcionalidades básicas de las etapas, puede utilizar múltiples etapas para realizar un procesamiento de datos complejo.

Por ejemplo, considere una aplicación que utiliza una colección de usuarios donde cada documento de usuario contiene la edad del usuario. La siguiente canalización de agregación de múltiples etapas utiliza `$sort` para ordenar primero a los usuarios de mayor a menor y luego utiliza `$limit` para generar los tres usuarios más antiguos:

Aquí está la entrada:

```json
[
  { "_id": 1, "name": "Justin", "age": 28 },
  { "_id": 2, "name": "Sydney", "age": 23 },
  { "_id": 3, "name": "Lauren", "age": 26 },
  { "_id": 4, "name": "Sophie", "age": 19 }
]
```

Esta es la canalización de agregación:

```javascript
db.users.aggregate([
  { $sort: { age: -1 } },
  { $limit: 3 }
])
```

Aquí está la salida:

```json
[
  { _id: 1, name: 'Justin', age: 28 },
  { _id: 3, name: 'Lauren', age: 26 },
  { _id: 2, name: 'Sydney', age: 23 }
]
```

Con la misma colección de usuarios, considere una aplicación que clasifica a los usuarios en grupos de edad.

La canalización de agregación que sigue realiza las siguientes operaciones:

1. Utiliza la etapa de agregación `$addFields` para agregar un campo `age_group`.
2. Utiliza la etapa de agregación `$group` para agrupar usuarios por grupo de edad.
3. Utiliza la etapa de agregación `$sort` para ordenar en orden ascendente.
4. Utiliza la etapa de agregación `$project` para generar el grupo de edad, la cantidad de usuarios en cada grupo de edad y los nombres de los usuarios en cada grupo de edad.

Aquí está la entrada:

```json
[
  { "_id": 1, "name": "Justin", "age": 28 },
  { "_id": 2, "name": "Sydney", "age": 23 },
  { "_id": 3, "name": "Lauren", "age": 26 },
  { "_id": 4, "name": "Sophie", "age": 19 },
]
```

Esta es la canalización de agregación:

```javascript
db.users.aggregate([
  {
    $addFields: {
      age_group: {
        $switch: {
          branches: [
            { case: { $lt: ["$age", 18] }, then: "18 and under" },
            { case: { $lt: ["$age", 25] }, then: "19 to 25" },
          ],
          default: "26 and Older"
        }
      }
    }
  },
  {
    $group: {
      _id: "$age_group",
      count: { $sum: 1 },
      names: { $push: "$name" }
    }
  },
  {
    $sort: { count: 1 }
  },
  {
    $project: {
      _id: 0,
      ageGroup: "$_id",
      count: 1,
      names: 1
    }
  }
])
```

Aquí está la salida:

```json
[
  { count: 2, names: [ 'Lauren', 'Justin' ], ageGroup: '26 and Older' },
  { count: 2, names: [ 'Sophie', 'Sydney' ], ageGroup: '19 to 25' }
]
```

#### Mejores prácticas

En esta sección, cubriremos algunos principios clave para escribir canalizaciones de agregación eficientes y robustas.

#### Modularidad del código

Incluso las canalizaciones de agregación más complejas deben estructurarse como una serie de etapas claras y enfocadas. Dividir su canalización en partes separadas lógicamente no solo mejora la legibilidad sino que también facilita la prueba y depuración de cada etapa individualmente. Por ejemplo, si utiliza MongoDB Compass para crear prototipos de canalizaciones, puede deshabilitar temporalmente etapas específicas sin eliminarlas, lo que ayuda con el desarrollo iterativo. Al trabajar en un editor de código, mantener las etapas claramente separadas (a menudo comentándolas según sea necesario) puede mejorar la capacidad de mantenimiento. Por ejemplo, la extensión de MongoDB para Visual Studio Code ofrece herramientas para ayudar a visualizar, depurar y refinar sus canalizaciones. En MongoDB Shell, donde JavaScript está disponible, definir cada etapa como una variable separada y luego ensamblarlas en una matriz puede hacer que la estructura de la canalización sea más fácil de administrar y modificar.

#### Etapas de transmisión (*streaming*) y bloqueo (*blocking*) de una canalización

Las etapas de agregación generalmente se dividen en dos categorías: transmisión (*streaming*) y bloqueo (*blocking*). Las etapas de transmisión procesan lotes de documentos a medida que llegan, lo que permite que los datos fluyan continuamente a través de la canalización. Por el contrario, las etapas de bloqueo deben recopilar todos los documentos entrantes antes de realizar su operación. Las etapas de bloqueo comunes incluyen `$group` y `$sort`, así como otras como `$bucket`, `$count`, `$sortByCount` y `$facet`, que también dependen de la acumulación total de entradas.

Aunque a veces son necesarias etapas de bloqueo, ubicarlas descuidadamente puede afectar negativamente el rendimiento al aumentar la latencia y limitar la concurrencia. Para mitigar estos problemas, considere las siguientes estrategias:

- **Filtrar antes de ordenar:** Siempre que sea posible, utilice etapas `$match` para reducir el conjunto de trabajo antes de introducir operaciones costosas como `$sort`. El filtrado temprano puede reducir significativamente el volumen de datos.
- **Ordenar temprano en campos indexados:** Si se requiere `$sort`, intente ordenar en campos que estén indexados y ubicados cerca del inicio de la canalización. Esto puede reducir sustancialmente la carga de procesamiento.
- **Combinar `$sort` con `$limit`:** Cuando solo se necesita un subconjunto de documentos ordenados, colocar `$limit` después de `$sort` le permite a MongoDB optimizar la operación internamente, reduciendo el uso de memoria al reducir el alcance de los documentos desde el principio.
- **Evitar el antipatrón `$unwind` + `$group`:** Si está modificando o resumiendo matrices, prefiera utilizar operadores de expresión de matriz en su lugar. A menudo proporcionan un mejor rendimiento y claridad.
- **Diferir las transformaciones de campos:** Al ordenar o agrupar, es mejor hacerlo en campos que aún no hayan sido alterados dentro de la canalización. Confiar en los campos indexados existentes antes de la mutación mejora la eficiencia.

---

### Sección 4: Formas de consulta (*Query shapes*)

Una forma de consulta (*query shape*) es un conjunto de especificaciones que agrupan consultas similares. Las consultas que coinciden con las especificaciones tienen la misma forma de consulta. Si ejecuta con frecuencia consultas que tienen la misma forma, puede aprovechar sus cualidades compartidas para optimizar el rendimiento.

Los siguientes términos también son útiles para esta sección:

- **Plan de consulta (*Query plan*):** Un plan de consulta es un método para ejecutar una consulta. Una sola consulta puede tener múltiples planes de consulta que producen los mismos resultados.
- **Optimizador/Planificador de consultas (*Query optimizer/planner*):** El optimizador de consultas de MongoDB, también llamado planificador de consultas, evalúa los planes de consulta y determina qué plan ejecutar. La mayoría de las veces, el plan de consulta que produce la mayor cantidad de resultados durante el plan de evaluación mientras realiza la menor cantidad de trabajo es el plan ganador que ejecuta MongoDB. El optimizador de consultas evalúa las consultas con la misma forma de consulta como equivalentes.

Anteriormente, MongoDB usaba el término forma de consulta (*query shape*) para describir específicamente consultas con cualidades compartidas que evalúa el planificador de consultas de MongoDB. MongoDB 8.0 cambia el nombre de las formas de consulta preexistentes a formas de consulta de caché de planes (*plan cache query shapes*). La forma de consulta de caché de plan (anteriormente, la forma de consulta) admite la planificación de consultas, filtros de índice obsoletos y un subconjunto de la canalización de agregación.

Con MongoDB 8.0, la nueva forma de consulta admite las siguientes características:

- Configuraciones de consulta (*Query settings*)
- Estadísticas de `$queryStats`
- La mayoría de los campos y operandos que están disponibles para los comandos de base de datos `find`, `distinct` y `aggregate`
- Toda la canalización de agregación

En particular, la nueva forma de consulta amplía las capacidades de agregación de la forma de consulta de caché de plan.

Las consultas también tienen un `queryShapeHash`, que identifica las formas de consulta. Las consultas con la misma forma de consulta tienen el mismo valor de `queryShapeHash`.

#### Configuraciones de consulta (*Query settings*)

MongoDB 8.0 introduce la capacidad de especificar configuraciones de consulta para formas de consulta. El optimizador de consultas de MongoDB toma en cuenta las configuraciones de consulta cuando determina cómo ejecutar una consulta. Las configuraciones de consulta son útiles para especificar índices o configuraciones de ejecución. Para agregar configuraciones de consulta, use el comando de base de datos `setQuerySettings`.

Por ejemplo, puede utilizar una configuración de consulta para especificar el índice que desea que utilice una consulta para optimizar el rendimiento de la consulta.

Considere una colección `users` que contiene los siguientes datos:

```json
[
  { "_id": 1, "name": "Justin", "age": 28 },
  { "_id": 2, "name": "Sydney", "age": 23 },
  { "_id": 3, "name": "Lauren", "age": 26 },
  { "_id": 4, "name": "Sophie", "age": 19 },
]
```

Esta colección `users` también contiene un índice ascendente en el campo `age`, llamado `age_1`. El siguiente comando especifica una configuración de consulta para una operación `find` que utiliza `filter` y `sort`:

```javascript
db.adminCommand({
  setQuerySettings: {
    find: "users",
    filter: {age: {$gt: 23 }},
    sort: {age: 1},
    $db: "test"  
  },
  settings: {
    indexHints: {
      ns: {db: "test", coll: "users"},
      allowedIndexes: ["age_1"]
  },
  queryFramework: "classic",
  comment: "Index hint for age_1 index to improve performance"
  }
})
```

Las siguientes consultas tienen la misma forma y, por lo tanto, la configuración de consulta especificada se aplica a ambas consultas:

```javascript
db.users.find( { age: { $gt: 23 } } ).sort( { age: 1 } )
db.users.find( { age: { $gt: 21 } } ).sort( { age: -1 } )
```

Cuando MongoDB ejecuta consultas con esta forma, el optimizador de consultas toma en cuenta la configuración de la consulta como entrada adicional.

#### Comandos de configuración de consultas

Puede utilizar los siguientes comandos de base de datos para administrar sus configuraciones de consulta:

- Para agregar configuraciones de consulta, use el comando `setQuerySettings`
- Para eliminar configuraciones de consulta, use el comando `removeQuerySettings`
- Para recuperar configuraciones de consulta, use una etapa `$querySettings` en una canalización de agregación

#### Filtros de rechazo de operaciones (*Operation rejection filters*)

MongoDB 8.0 también introduce filtros de rechazo de operaciones, que rechazan temporalmente operaciones que están asociadas con una forma de consulta específica. Un filtro de rechazo de operaciones también se conoce como una forma de consulta rechazada (*rejected query shape*).

El planificador de consultas toma en cuenta los filtros de rechazo de operaciones cuando determina cómo ejecutar una consulta. Los filtros de rechazo de operaciones pueden ser útiles si una aplicación tiene una consulta ineficiente que genera una carga de trabajo excesiva.

Por ejemplo, si una aplicación tiene una colección `users` y desea evitar operaciones `find` que usen `filter` y `sort`, una configuración de consulta con un filtro de rechazo de operaciones puede evitar esas operaciones.

Considere una colección `users` con los siguientes datos:

```json
[
  { "_id": 1, "name": "Justin", "age": 28 },
  { "_id": 2, "name": "Sydney", "age": 23 },
  { "_id": 3, "name": "Lauren", "age": 26 },
  { "_id": 4, "name": "Sophie", "age": 19 }
]
```

La siguiente operación `setQuerySettings` rechaza las operaciones `find` en la colección `users` que tienen `filter` y `sort`:

```javascript
db.adminCommand({
   setQuerySettings: {
      find: "users",
      filter: {
         age: { $gt: 24 }
      },
      sort: {
         age: 1
      },
      $db: "test"
   },
   settings: {
      reject: true
   }
})
```

La salida de este comando muestra el campo `queryShapeHash` de la forma de consulta que bloquea la configuración de consulta.

Si intenta ejecutar una consulta con la forma de consulta especificada, MongoDB devuelve el siguiente error:

```text
MongoServerError: Query rejected by admin query settings
```

Para ver sus filtros de rechazo de operaciones, puede utilizar las etapas de agregación `$querySettings` y `$match` en una canalización de agregación.

Por ejemplo, la siguiente canalización de agregación en la colección `users` genera el `queryShapeHash` de las consultas bloqueadas:

Esta es la canalización de agregación:

```javascript
db.aggregate([
  { $querySettings: { showDebugQueryShape: true } },
  { $match: { "settings.reject": true } }
])
```

Esta es la salida:

```json
[
  {
    queryShapeHash: 'AB8ECADEE8F0EB0F447A30744EB4813AE7E0BFEF523',
    settings: { reject: true },
    representativeQuery: {
      find: 'users',
      filter: { age: { '$gt': 24 } },
      sort: { age: 1 },
      '$db': 'test'
    },
    debugQueryShape: {
      cmdNs: { db: 'test', coll: 'users' },
      command: 'find',
      filter: { age: { '$gt': 24 } },
      sort: { totalNumber: 1 }
    }
  }
]
```

#### Estadísticas de $queryStats

Las operaciones `$queryStats` devuelven un campo `key` que contiene la forma de consulta y los atributos que agrupan consultas similares. El documento anidado `key.queryShape` contiene campos relacionados con la forma de la consulta. Los campos varían según el tipo de comando que genera la entrada de estadísticas de consulta. Por ejemplo, la siguiente tabla ilustra algunos de los campos que pueden aparecer en un documento anidado `key.queryShape` para un comando `find`:

| Campo | Tipo | Literal o Normalizado |
| :--- | :--- | :--- |
| `key.queryShape.filter` | Document | Normalizado |
| `key.queryShape.sort` | Document | Literal |
| `key.queryShape.projection` | Document | Normalizado |
| `key.queryShape.skip` | Integer | Normalizado |

> **Tabla 5.3:** Campos en key.queryShape para comandos Find en la salida de $queryStats

#### Campos, operandos y canalizaciones de agregación

La forma de consulta de MongoDB 8.0 admite todos los campos y operandos que admite la forma de consulta de caché de planes. Por ejemplo, la forma de consulta admite operaciones `filter`, `sort` y `project`. En otras palabras, puede especificar configuraciones de consulta en operaciones de filtro, ordenación y proyección.

Además, la forma de consulta admite la mayoría de los campos y operandos que admiten los comandos `find`, `distinct` y `aggregate`.

La forma de consulta de MongoDB 8.0 también admite toda la canalización de agregación. Antes de MongoDB 8.0, las formas de consulta no tomaban en cuenta canalizaciones de agregación completas. Por ejemplo, en MongoDB 7.0, una operación `find` y una operación `aggregate` podrían compartir la misma forma de consulta si las operaciones son semánticamente similares. Sin embargo, en MongoDB 8.0, las formas de consulta toman en cuenta todas las etapas de una canalización de agregación para diferenciar aún más las formas de consulta.

---

### Sección 5: Nuevos comandos de base de datos

MongoDB 8.0 también proporciona nuevos comandos de base de datos con soporte a nivel de controlador para ampliar las capacidades anteriores de la base de datos. Estos comandos son típicamente tareas administrativas ad-hoc.

#### Comandos de base de datos a nivel de colección

MongoDB 8.0 introduce los siguientes comandos de base de datos a nivel de colección:

- **`abortMoveCollection`:** Detiene una operación `moveCollection` en curso.
- **`unshardCollection`:** Desfragmenta una colección fragmentada existente.
- **`abortUnshardCollection`:** Detiene una operación `unshardCollection` en curso.
- **`moveCollection`:** Mueve una sola colección no fragmentada a un fragmento diferente.

#### Comandos de base de datos a nivel de servidor

MongoDB 8.0 introduce los siguientes comandos de base de datos a nivel de servidor:

- **`transitionFromDedicatedConfigServer`:** Realiza la transición de un servidor de configuración desde un servidor de configuración dedicado a un fragmento de configuración (*config shard*).
- **`transitionToDedicatedConfigServer`:** Realiza la transición de un servidor de configuración desde un fragmento de configuración a un servidor de configuración dedicado.

---

### Sección 6: Resumen

Este capítulo exploró cómo interactuar con sus datos de MongoDB mediante operaciones CRUD básicas, consultas avanzadas y canalizaciones de agregación. Aprendió formas de crear, leer, actualizar y eliminar documentos tanto con `mongosh` como con `PyMongo`. Para aprovechar estas interacciones básicas con la base de datos, analizamos las operaciones masivas (*bulk operations*) que pueden realizar múltiples operaciones en múltiples colecciones dentro de la misma solicitud. También aprendió sobre el marco de agregación de MongoDB y exploró varias canalizaciones de agregación básicas y avanzadas, utilizando etapas de agregación de uso común. Este capítulo también incluyó información sobre las mejores prácticas de agregación y formas de consulta (*query shapes*), las cuales se pueden utilizar para mejorar el rendimiento de sus datos.

En el próximo capítulo, cambiaremos nuestro enfoque de las consultas y la manipulación de datos al aspecto operativo del trabajo con MongoDB. Aprenderá a monitorear el rendimiento, configurar despliegues seguros, configurar copias de seguridad e implementar estrategias de auditoría. Estas herramientas y prácticas son esenciales para crear sistemas MongoDB confiables, escalables y listos para producción.
