# Parte 1: Introducción a MongoDB

## Capítulo 1: Introducción a MongoDB

MongoDB es la base de datos de documentos NoSQL más popular del mundo. En lugar de utilizar tablas relacionales tradicionales, almacena datos en un formato flexible similar a JSON. Los desarrolladores la valoran por su facilidad de escalado, sólidas características de seguridad y rendimiento confiable. Con herramientas como la búsqueda mejorada por IA, el *sharding* y el cifrado, MongoDB ofrece una plataforma integral para gestionar diversas necesidades de datos.

MongoDB 8.0 es la versión más rápida y de mayor rendimiento de MongoDB hasta la fecha, con un rendimiento aproximadamente un 32% superior al de la versión anterior de MongoDB (Fuente: [https://www.mongodb.com/company/blog/mongodb-8-0-improving-performance-avoiding-regressions](https://www.mongodb.com/company/blog/mongodb-8-0-improving-performance-avoiding-regressions)). Además de las mejoras de rendimiento, MongoDB 8.0 ofrece diversas mejoras en agregación, seguridad, *sharding*, replicación y más, como las siguientes:

- Las mejoras en el *sharding* distribuyen los datos a través de los fragmentos (*shards*) hasta 50 veces más rápido y con un costo inicial hasta un 50% menor, reduciendo el costo de infraestructura inicial en hasta $5,000 (USD) al año.
- Soporte mejorado para una amplia gama de aplicaciones de búsqueda e IA a mayor escala y menor costo, utilizando vectores cuantizados (representaciones comprimidas de vectores de fidelidad completa) que requieren hasta un 96% menos de memoria y son más rápidos de recuperar mientras preservan la precisión.
- Soporte ampliado para el *Queryable Encryption* de MongoDB, una innovación revolucionaria desarrollada por el Grupo de Investigación en Criptografía de MongoDB, que ahora también admite consultas por rango.

Este libro proporciona una descripción general completa de lo que MongoDB, en particular la versión 8.0, puede ofrecer para sus necesidades de desarrollo. Si bien nos esforzamos por ser exhaustivos y actuales, MongoDB agrega constantemente nuevas funciones. Para obtener la información más detallada y actualizada, consulte la documentación de MongoDB en [https://mongodb.com/docs](https://mongodb.com/docs).

En este capítulo, vamos a cubrir los siguientes temas:

- ¿Por qué MongoDB?
- ¿Quién utiliza MongoDB?
- Arquitectura de MongoDB
- ¿Qué hay de nuevo en MongoDB 8.0?

---

### Sección 1: ¿Por qué MongoDB?

MongoDB es la plataforma de datos preferida por los desarrolladores por varias razones:

- **Flexibilidad:** MongoDB le permite trabajar con datos sin encerrarlo en esquemas rígidos. Esto facilita la adaptación de su aplicación a medida que cambian sus datos, o al manejar formatos que se benefician de un almacén de datos flexible, como el contenido no estructurado o semiestructurado.
- **Escalabilidad y rendimiento:** MongoDB es altamente escalable y eficiente, lo que le permite admitir tanto aplicaciones a gran escala como proyectos individuales más pequeños.
- **Seguridad:** MongoDB ofrece varios métodos de seguridad, desde autenticación y autorización de usuarios hasta cifrado de datos y red, incluido el soporte en la versión 8.0 para autenticación y autorización OIDC, y consultas por rango en *Queryable Encryption*.
- **Lenguaje de consulta:** MongoDB ofrece un potente lenguaje de consulta que puede utilizar para acceder a sus datos, simplificando operaciones comunes como `findOne` y `updateOne`. También ofrece capacidades de indexación para una mayor eficiencia en las consultas.
- **Formato de datos amigable para el desarrollador:** MongoDB almacena datos en un formato de documento que se asemeja a la estructura de objetos en muchos lenguajes de programación ampliamente utilizados, lo que ayuda a simplificar el manejo de datos y acelera el proceso de desarrollo.
- **Inicio rápido:** La simplicidad de MongoDB y su configuración intuitiva hacen que sea fácil comenzar a utilizarlo.

Sencillamente, MongoDB es simple de usar. Puede interactuar con su despliegue de varias maneras, como a través de controladores (*drivers*) de lenguajes de programación, métodos en el MongoDB Shell y comandos de base de datos. MongoDB proporciona una interfaz simple y optimizada para crear, actualizar e interactuar con datos. Por ejemplo, considere a un desarrollador de Python que intenta insertar un documento utilizando el controlador de Python:

```python
from pymongo import MongoClient
# Connect to MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['mydatabase'] # Specify the database name
collection = db['mycollection'] # Specify the collection name
# Create a document to be inserted
document = {
   'name': 'Chinazom',
   'email': 'chinazom@example.com'
}
# Insert the document into the collection
result = collection.insert_one(document)
# Check if the insertion was successful
if result.acknowledged:
    print('Document inserted successfully.')
    print('Inserted document ID:', result.inserted_id)
else:
    print('Failed to insert document.')
```

¡Fácil! No necesita crear un ID para el documento, porque MongoDB crea uno automáticamente por usted. En este caso, todo lo que el desarrollador necesita definir son los detalles de nombre, edad y correo electrónico del documento.

Ahora, suponga que el desarrollador desea recuperar este documento mediante una consulta. Puede consultar por igualdad (por ejemplo, buscar documentos en los que el nombre sea Chinazom). También puede consultar por desigualdad. En el siguiente ejemplo, estamos construyendo una consulta para buscar documentos cuya edad sea menor o igual a 25.

```python
from pymongo import MongoClient
# Connect to MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['mydatabase'] # Specify the database name
collection = db['mycollection'] # Specify the collection name
# Retrieve documents based on specific conditions
query = {
  'age': {'$lte': 25}, # Retrieve documents where age is less than or equal to 25
}
documents = collection.find(query)
# Iterate over the retrieved documents
for document in documents:
    print(document)
```

Este ejemplo muestra cómo puede utilizar un operador de consulta de MongoDB como `$lte` para filtrar una consulta. MongoDB devuelve un documento que se representa como un diccionario de Python, donde cada campo es un par clave-valor en el diccionario. Vea el siguiente ejemplo:

```json
{
  '_id': ObjectId('60f5c4c4543b5a2c7c4c73a2'),
  'name': 'Chinazom',
  'age': 24,
  'email': 'chinazom@example.com'
}
```

Como puede ver, el campo `_id` ha sido insertado por MongoDB y se representa como un tipo de datos `ObjectId`. El campo `_id` es un identificador único y rápido de generar para cada documento. Se utiliza como el identificador principal de un documento.

MongoDB cuenta con un conjunto de controladores (*drivers*) en varios lenguajes de programación que actúan como una capa de traducción entre el cliente y el servidor. Al utilizar estos controladores, puede interactuar con los datos con su lenguaje de programación nativo. También puede interactuar con sus datos utilizando el MongoDB Shell, comandos de base de datos y otras herramientas ofrecidas por MongoDB.

La misión de MongoDB es ser una base de datos potente para los desarrolladores, y sus características se desarrollan teniendo en cuenta las comunidades de lenguajes de programación y las integraciones de *frameworks*. Esto se hará más evidente en los capítulos posteriores, donde aprenderá sobre operaciones CRUD, *sharding*, replicación, MongoDB Atlas y más, todo a través de la perspectiva de un desarrollador.

---

### Sección 2: ¿Quién utiliza MongoDB?

MongoDB es utilizado por muchas industrias diferentes y sus casos de uso abarcan todo tipo de situaciones y tipos de datos. Los usuarios van desde pequeñas empresas y empresas emergentes (*start-ups*), e incluso proyectos de estudiantes, hasta algunos de los bancos, fabricantes de automóviles y agencias gubernamentales más grandes del mundo.

MongoDB ofrece tres entornos diferentes para que cree una base de datos y almacene sus datos, cada uno de los cuales satisface diferentes necesidades de los desarrolladores:

- **MongoDB Atlas:** Atlas es el servicio de base de datos en la nube de MongoDB. Simplifica el despliegue y la gestión de sus bases de datos, eliminando la responsabilidad del usuario de mantener el hardware y estar al día con los parches de software. Puede aprovisionar su base de datos a través de la interfaz de usuario de Atlas y escalar fácilmente según sea necesario. Cuando utiliza Atlas, tiene acceso a características adicionales como búsqueda de texto completo integrada a través de Atlas Search, búsqueda basada en vectores a través de Atlas Vector Search y Atlas Stream Processing, que le permite procesar flujos de datos complejos. Atlas es utilizado por aquellos que desean concentrarse en el desarrollo de modelos de datos en lugar de dedicar recursos al aprovisionamiento y la gestión de su base de datos.
- **MongoDB Enterprise Advanced (EA):** MongoDB EA es la edición comercial de MongoDB. Incluye capacidades adicionales como un motor de almacenamiento en memoria para un alto rendimiento y baja latencia, características avanzadas de seguridad como controles de acceso Kerberos y cifrado para datos en reposo. EA es ideal para organizaciones que requieren despliegues autogestionados en entornos locales (*on-premises*), nubes privadas o entornos híbridos. La suscripción a EA, a través de la cual obtiene MongoDB EA, incluye soporte 24/7/365 y herramientas como MongoDB Ops Manager para ayudar a simplificar el despliegue.
- **MongoDB Community:** MongoDB Community es la versión gratuita de MongoDB. Incluye soporte para operaciones básicas como consultas, indexación y agregación. Debido a que es gratuita y de código disponible (*source-available*), MongoDB Community suele ser utilizada por pequeñas empresas y *start-ups* que buscan una plataforma de datos para desarrolladores con una baja barrera de entrada.

Consulte los siguientes recursos para obtener más información:

- [https://www.mongodb.com/try](https://www.mongodb.com/try)
- [https://www.mongodb.com/docs/atlas/production-notes/](https://www.mongodb.com/docs/atlas/production-notes/)
- [https://www.mongodb.com/docs/manual/administration/production-notes/](https://www.mongodb.com/docs/manual/administration/production-notes/)
- [https://www.mongodb.com/blog/post/mongodb-8-0-raising-the-bar](https://www.mongodb.com/blog/post/mongodb-8-0-raising-the-bar)

---

### Sección 3: Arquitectura de MongoDB

Esta sección ofrece un resumen de algunos de los aspectos de un despliegue típico de MongoDB. Continúe leyendo para obtener más información sobre la arquitectura de clústeres, clústeres fragmentados y colecciones.

#### Clústeres

La configuración típica de MongoDB es un conjunto de réplicas (*replica set*), también conocido como clúster: un grupo de instancias de `mongod` que mantienen el mismo conjunto de datos. `mongod` es el demonio principal de MongoDB. Por defecto, un conjunto de réplicas es una configuración de tres nodos. Cada nodo contiene una copia completa de su base de datos, lo que permite redundancia y alta disponibilidad: si un nodo se cae, aún puede acceder a los otros dos.

Los nodos se clasifican generalmente como nodos primarios o secundarios. El nodo primario recibe todas las operaciones de escritura del cliente. Las operaciones de escritura se registran en el `oplog`, que es una colección que mantiene un registro de todas las operaciones realizadas en el nodo primario. Luego, los nodos secundarios replican este registro y realizan las operaciones en sus conjuntos de datos para garantizar que coincidan con los datos del nodo primario. Las operaciones de lectura se realizan en el nodo primario por defecto. Sin embargo, los usuarios pueden especificar que las operaciones se realicen en diferentes nodos, como el nodo más cercano o explícitamente en nodos secundarios.

Si un nodo primario deja de responder o no está disponible, un nodo secundario en buen estado es promovido a primario. El conjunto de réplicas lleva a cabo una elección para elegir cuál de los secundarios se convierte en el nuevo primario. Varios factores afectan las elecciones, como la capacidad de respuesta, la prioridad del nodo y la frescura de los datos.

> **Figura 1.1:** Arquitectura de clúster replicado

#### Clústeres fragmentados (*Sharded clusters*)

Los clústeres fragmentados son útiles cuando desea dividir aún más sus datos en particiones replicadas mientras mantiene la eficiencia. El *sharding* es más útil cuando su aplicación requiere grandes conjuntos de datos y operaciones de alto rendimiento. Cada clúster fragmentado consta de *shards*, o conjuntos de réplicas, sobre los cuales se distribuyen los datos a nivel de colección. También incluyen instancias de `mongos`, que actúan como una interfaz entre el cliente y el servidor, enrutando una consulta al fragmento o fragmentos apropiados, así como fusionando los datos resultantes. El aspecto final de un clúster fragmentado es un servidor de configuración (*config server*), que es un conjunto de réplicas que almacena metadatos de replicación y configuración para el clúster.

A partir de MongoDB 8.0, ahora puede desfragmentar colecciones (*unshard collections*), así como mover colecciones no fragmentadas entre fragmentos en clústeres fragmentados. Esto le ofrece más flexibilidad sobre cómo optimizar el rendimiento de su clúster.

> **Figura 1.2:** Arquitectura de clúster fragmentado

Para obtener más información sobre la replicación y el *sharding* y sus mejoras en MongoDB 8.0, consulte el [Capítulo 2](https://subscription.packtpub.com/book/data/9781837021970/2), Arquitectura de MongoDB. Consulte también la documentación de MongoDB para obtener la información más actualizada.

#### Colecciones

Una colección es una agrupación de documentos de MongoDB. Es el equivalente a una tabla en una base de datos relacional. En MongoDB, las colecciones no imponen un esquema para los datos que se insertan en ellas.

> **Figura 1.3:** Documentos en una colección

A las colecciones se les asigna un UUID, o identificador único universal. El UUID es especialmente útil en un clúster fragmentado al identificar qué fragmentos de datos pertenecen a una colección específica.

Al ejecutar una consulta, MongoDB debe escanear cada documento de una colección para devolver los resultados de la consulta. Sin embargo, puede crear índices en campos específicos a nivel de colección para recorrer los datos de manera más eficiente. Por ejemplo, si tiene una colección llena de datos de empleados y a menudo necesita buscar información de empleados según el ID de empleado, puede crear un índice a nivel de colección en el campo ID para mejorar el rendimiento de las consultas.

Los índices almacenan una pequeña porción de los datos de la colección en una estructura de datos de árbol B (*B-tree*), lo que permite búsquedas en tiempo logarítmico (O(log n)). El índice almacena el valor de un campo específico o conjunto de campos ordenados por el valor del campo, lo que permite un recorrido más fácil a través de los datos.

---

### Sección 4: ¿Qué hay de nuevo en MongoDB?

MongoDB 8.0 es la versión de mayor rendimiento de MongoDB hasta el momento, con un enfoque en las siguientes cuatro áreas:

- Optimización del rendimiento
- Mejoras de cifrado
- Reducción de costos y mejoras de escalado
- Mayor resiliencia

#### Rendimiento

MongoDB mejora las consultas y la transformación de datos, con hasta un 36% mejor rendimiento (*throughput*), escrituras masivas un 56% más rápidas, agregaciones complejas de datos de series temporales un 200% más rápidas y escrituras concurrentes un 20% más rápidas durante la replicación de datos. En general, MongoDB 8.0 funciona aproximadamente un 32% más rápido que la versión anterior de MongoDB. Esto significa que puede realizar operaciones más rápido con un menor uso de recursos y costos.

#### Cifrado

Con una combinación de protecciones de autenticación y autorización y cifrado en reposo, en tránsito y en uso, MongoDB protege los datos a lo largo de su ciclo de vida. *Queryable Encryption*, que ofrece un cifrado robusto para datos en uso, permite una mayor flexibilidad en la forma de interactuar con sus datos, garantizando al mismo tiempo que estén cifrados en todo momento.

#### Costo y escalabilidad

Con las mejoras de *sharding* y replicación en MongoDB, el escalado horizontal de su aplicación, o el aumento de la cantidad de recursos que utiliza, es más rápido y fácil a un menor costo. MongoDB 8.0 distribuye datos a través de *shards* hasta 50 veces más rápido y a un costo un 50% menor que la versión anterior.

#### Resiliencia

Para promover la resiliencia de su aplicación incluso bajo cargas de trabajo de alto estrés, MongoDB incluye mejoras como tiempos máximos predeterminados para consultas, actualizaciones de TCMalloc y mejoras en la configuración de consultas.

---

### Sección 5: Resumen

Con su flexibilidad y rendimiento, MongoDB maneja los detalles de bajo nivel para que usted pueda concentrarse en el desarrollo de su aplicación. Es sencillo de configurar y proporciona un entorno eficiente y amigable para el desarrollador.

El resto de este libro detalla cómo MongoDB puede respaldar sus necesidades y las diferentes herramientas y características que ofrece. Desde MongoDB Atlas hasta *Queryable Encryption* y MongoDB Shell, existen innumerables herramientas que le permiten utilizar MongoDB a su favor.

Además de proporcionar una infraestructura robusta y escalable para sus despliegues de producción y gestionar tareas de alto rendimiento, MongoDB también es una excelente opción para aprender y crear pruebas de concepto.

También cubriremos cómo las nuevas características de MongoDB 8.0 le permitirán crear aplicaciones más rápidas y resilientes. En el próximo capítulo, aprenderá cómo la replicación y el *sharding* pueden ayudarle a construir una base de datos confiable, disponible y eficiente para las necesidades de su aplicación.

¡Feliz desarrollo!
