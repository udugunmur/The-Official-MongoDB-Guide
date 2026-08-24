# Parte 4: Modelado de Datos y Optimización de Índices

## Capítulo 4: Modelado de Datos y Optimización de Índices

Elegir cómo organizar y modelar sus datos es uno de los pasos más importantes para garantizar que su aplicación sea eficiente y de alto rendimiento. Una de las principales causas del bajo rendimiento en MongoDB es un diseño de esquema deficiente. Un mal diseño de esquema no solo afecta negativamente el rendimiento de la aplicación, sino que también dificulta el escalado y mantenimiento del modelo de datos. Al planificar y diseñar adecuadamente su esquema, se asegura de que su aplicación pueda acceder rápidamente a la información que necesita. El modelo de datos flexible de MongoDB le permite almacenar sus datos para que coincidan con los patrones de acceso a los datos, lo que significa que se puede acceder a todos los datos necesarios con una sola consulta. Diseñar su esquema en función de los patrones de acceso de su aplicación evita la necesidad de operaciones de unión (*join*) lentas entre múltiples tablas, comunes en las bases de datos relacionales. Este capítulo describe las mejores prácticas para planificar y mantener su esquema de MongoDB y cómo crear índices para optimizar el rendimiento de las consultas.

En este capítulo, vamos a cubrir los siguientes temas principales:

- El modelo de datos de documentos de MongoDB y en qué se diferencia de los modelos de datos tabulares y relacionales
- Cómo planificar su esquema en función de los patrones de acceso a los datos de su aplicación
- Diseños de esquemas que puede utilizar para manejar patrones de acceso comunes
- Cómo optimizar los índices para mejorar el rendimiento de las consultas, incluidas las nuevas configuraciones de consulta persistentes disponibles en MongoDB

---

### Sección 1: Modelo de datos documental

El modelo de datos documental es una forma de estructurar y almacenar datos donde la información se guarda en documentos flexibles y autónomos. El modelo de datos documental proporciona los siguientes beneficios:

- **Documentos autónomos:** Puede modelar su esquema para que cada documento almacene todos los datos relacionados juntos, lo que reduce la necesidad de uniones (*joins*) complejas.
- **Flexibilidad:** Los documentos pueden tener estructuras y tipos de datos variables.
- **Datos incrustados (*Embedded data*):** Los documentos pueden contener objetos y matrices incrustados, lo que le permite modelar fácilmente relaciones complejas entre entidades, como uno a muchos y muchos a muchos.

El siguiente documento de muestra ilustra la estructura de datos de documentos de MongoDB:

```json
{
  "_id": ObjectId("650d9b7fb6a6f4f8d45ce0b8"),
  "username": "johndoe",
  "email": "johndoe@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "birthdate": ISODate("1994-04-17T00:00:00Z")
  },
  "skills": ["JavaScript", "MongoDB"],
  "experience": [
    {
      "company": "Tech Corp",
      "role": "Software Engineer",
      "startDate": ISODate("2018-06-01T00:00:00Z"),
      "endDate": ISODate("2021-08-30T00:00:00Z")
    }
  ]
}
```

El modelo de documentos de MongoDB promueve una forma natural e intuitiva de representar datos, reflejando la forma en que los desarrolladores a menudo piensan y trabajan con la información en sus aplicaciones.

#### BSON y sus tipos de datos

MongoDB almacena datos como documentos BSON (JSON binario), que son colecciones de pares clave-valor. BSON es una extensión de JSON que comparte su estructura de clave-valor legible por humanos y la mejora con optimizaciones para el almacenamiento y la velocidad de escaneo.

BSON fue diseñado para tener las siguientes tres características:

- **Ligero:** BSON reduce la sobrecarga espacial, lo que resulta especialmente útil cuando se envían datos a través de una red.
- **Fácil de navegar:** Las estructuras BSON se pueden navegar de manera eficiente, lo cual es fundamental ya que es el formato central que utiliza MongoDB para representar los datos.
- **Eficiente:** En un documento BSON, los elementos grandes tienen como prefijo un campo de longitud. Estos prefijos permiten que las consultas escaneen y recuperen resultados más rápido que el JSON estándar, especialmente en aplicaciones a gran escala. Además, la codificación de datos a BSON y la decodificación desde BSON se pueden realizar muy rápidamente en la mayoría de los lenguajes debido al uso de tipos de datos de C.

BSON amplía las capacidades de JSON estándar al admitir una gama más amplia de tipos de datos. En particular, incluye tipos especializados como los siguientes:

- **`BinData`:** Para almacenar de manera eficiente datos binarios, incluidos contenidos de archivos y objetos serializados.
- **`MinKey` y `MaxKey`:** Para comparar valores con los elementos BSON mínimos o máximos absolutos, normalmente para ordenar o definir límites de rango de consulta.

Además, BSON proporciona representación decimal de alta precisión con el tipo `Decimal128`. Esto permite una representación numérica de alta precisión, que es especialmente importante en dominios como servicios financieros, computación científica o sistemas monetarios donde la representación decimal exacta y el comportamiento de redondeo son esenciales. A diferencia del tipo `Number` de JavaScript, que es un valor de formato binario IEEE 754 de 64 bits de doble precisión, `Decimal128` evita errores de redondeo en casos de uso de alta precisión.

El siguiente documento de ejemplo muestra los diversos tipos de datos disponibles en BSON, incluidos tipos de datos no disponibles en JSON estándar:

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439013"),
  "doubleField": 87.9,
  "stringField": "Greetings!",
  "objectField": {
    "name": "Sarah",
    "age": 28
  },
  "arrayField": [10, 20, 30, 40, 50],
  "binaryField": BinData(0, "different-binary-data"),
  "booleanField": false,
  "dateField": ISODate("2024-12-15T14:30:00Z"),
  "nullField": null,
  "regexField": /^xyz/,
  "javascriptField": Code("function () { print('hello world!'); }"),
  "int32Field": 128,
  "timestampField": Timestamp(1701410400, 2),
  "int64Field": NumberLong("1234567890"),
  "decimal128Field": NumberDecimal("9.876543210987654321"),
  "minKeyField": MinKey(),
  "maxKeyField": MaxKey()
}
```

En este documento, cada campo muestra un tipo de datos BSON único. Esto resalta la flexibilidad de MongoDB cuando se trata de representar datos. Los tipos de datos se resumen en la siguiente tabla:

| Tipo | Alias | Explicación |
| :--- | :--- | :--- |
| Double | `double` | Número decimal de punto flotante con una precisión aproximada de 16 dígitos decimales. |
| String | `string` | Cadena de caracteres que representan texto. |
| Object | `object` | Representa un documento. |
| Array | `array` | Lista de elementos. |
| Binary data | `binData` | Datos binarios sin formato arbitrarios. |
| ObjectId | `objectId` | Identificador único para un objeto. |
| Boolean | `bool` | Representa verdadero o falso. |
| Date | `date` | Representa un punto específico en el tiempo. |
| Null | `null` | Representa un valor nulo. |
| Regular Expression | `regex` | Representa reglas de coincidencia de patrones para cadenas. |
| JavaScript | `javascript` | Almacena código JavaScript ejecutable. |
| 32-bit integer | `int` | Número entero. |
| Timestamp | `timestamp` | Representa un punto en el tiempo relativo a la época (*epoch*). |
| 64-bit integer | `long` | Número entero con un rango de valores más amplio que `int`. |
| Decimal128 | `"decimal"` | Número decimal de alta precisión con hasta 34 dígitos decimales de precisión. |
| MinKey | `"minKey"` | Representa un valor menor que todos los demás valores BSON posibles. Normalmente se utiliza para representar un límite inferior. |
| MaxKey | `"maxKey"` | Representa un valor mayor que todos los demás valores BSON posibles. Normalmente se utiliza para representar un límite superior. |

> **Tabla 4.1:** Tipos de datos BSON

A medida que diseña y modela sus bases de datos, tener una comprensión sólida de los tipos BSON le ayudará a tomar decisiones más inteligentes e informadas sobre cómo se almacenan y recuperan sus datos.

#### Datos incrustados (*Embedded data*)

Una de las características clave del modelo de datos documental es la incrustación de datos relacionados en una estructura jerárquica, lo que le permite mantener datos relacionados en un solo documento. La incrustación de datos le permite modelar fácilmente relaciones entre diferentes entidades de datos. Al emplear estos modelos de datos desnormalizados, las aplicaciones pueden acceder y modificar datos correlacionados a través de una sola operación de base de datos.

El siguiente ejemplo muestra un documento de libro en MongoDB con detalles del autor incrustados:

```json
{
  "_id": ObjectId("65a1b2c3d4e5f6789abcdef0"),
  "title": "The Digital Revolution",
  "isbn": "978-0-123456-78-9",
  "publication_date": ISODate("2024-03-15T00:00:00Z"),
  "pages": 342,
  "genres": ["Technology", "History"],
  "price": NumberDecimal("29.99"),
  "author": {
    "name": "Sarah Johnson",
    "bio": "Dr. Sarah Johnson is a professor of Computer Science at Stanford University and expert in digital transformation.",
    "nationality": "American",
    "previous_works": [
      "AI and Society",
      "The Future of Computing"
    ]
  },
  "description": "An exploration of how digital technology has changed society over the past three decades.",
  "average_rating": 4.7,
  "in_stock": true
}
```

Si la aplicación siempre devuelve los detalles del libro y del autor juntos, incrustar esas entidades en un solo documento permite que la aplicación devuelva todos los datos requeridos con una sola operación de lectura. Por el contrario, la misma relación en una base de datos relacional requeriría que las entidades de libro y autor se almacenen en tablas separadas, las cuales tendrían que unirse y probablemente tendrían un peor rendimiento.

En el modelo de datos de documentos, el uso de referencias aún puede ser un patrón de diseño válido. Más adelante en este capítulo, aprenderá cómo determinar si incrustar datos o utilizar referencias es la mejor opción en función de los patrones de acceso a los datos de su aplicación.

---

### Sección 2: Diseño de esquemas

Diseñar un esquema eficaz es fundamental para optimizar el rendimiento y la escalabilidad de su aplicación. Para diseñar un esquema eficaz, primero debe comprender los datos que necesita su aplicación y con qué frecuencia necesita acceder a ellos. Esto le permite diseñar su esquema para que los datos a los que se accede juntos se almacenen juntos, lo que conduce a consultas eficientes y un rendimiento óptimo.

El proceso de diseñar un esquema que mejor satisfaga los requisitos de su aplicación consta de los siguientes pasos:

1. Identifique la carga de trabajo de su aplicación.
2. Mapee las relaciones entre sus entidades de datos.
3. Aplique el diseño de esquemas para mejorar el rendimiento de la aplicación y reducir la complejidad del esquema.

Al tomarse en serio el diseño del esquema, aprovecha las fortalezas de MongoDB al tiempo que mitiga los posibles inconvenientes que provienen de su estructura flexible y no relacional. Esto da como resultado una base de datos robusta que admite el funcionamiento y desarrollo eficiente de aplicaciones.

#### Identificar la carga de trabajo de la aplicación

Para comenzar a diseñar el esquema de su base de datos, primero determine las operaciones más frecuentes de su aplicación. Comprender estas consultas comunes es crucial para crear índices eficientes y reducir la cantidad de llamadas a la base de datos que debe realizar su aplicación.

Al analizar la carga de trabajo de su aplicación, considere tanto sus funcionalidades actuales como las posibles funciones futuras. Diseñe su esquema para que siga siendo eficaz en todas las fases del desarrollo de su aplicación.

Para crear una visión clara de los requisitos de su aplicación, puede crear una tabla de carga de trabajo con las consultas que su aplicación necesita ejecutar. Cada fila de la tabla debe contener una acción que la aplicación debe manejar, con qué frecuencia ocurre esa acción y cuán crítico es que la acción tenga éxito.

Por ejemplo, una tabla de carga de trabajo para una aplicación de blog podría verse de la siguiente manera:

| Acción | Tipo de Consulta | Información | Frecuencia | Prioridad |
| :--- | :--- | :--- | :--- | :--- |
| Enviar un nuevo artículo | Escritura | autor, texto | 10 por día | Alta |
| Enviar un comentario en un artículo | Escritura | usuario, texto | 1,000 por día (100 por artículo) | Media |
| Ver un artículo | Lectura | id del artículo, texto, comentarios | 1,000,000 por día | Alta |
| Ver analíticas de artículos | Lectura | id del artículo, comentarios, clics | 10 por hora | Baja |

> **Tabla 4.2:** Tabla de carga de trabajo para una aplicación de blog

Una vez que complete este paso del proceso de diseño del esquema, debería tener una sólida comprensión de la carga de trabajo de su aplicación y de los datos requeridos por ella. En el siguiente paso, aprenderá cómo mapear relaciones entre datos relacionados para devolver la información requerida en una sola consulta.

#### Mapear relaciones de esquema

La forma en que modela las relaciones entre diferentes entidades de datos afecta significativamente el rendimiento y la capacidad de escalado de su aplicación.

El modelo de datos de documentos de MongoDB le permite incrustar información relacionada como subdocumentos. La incrustación permite que su aplicación obtenga los datos necesarios con una sola operación de lectura, evitando la sobrecarga de rendimiento asociada con las operaciones `$lookup`. Sin embargo, en ciertas situaciones, particularmente cuando se requieren actualizaciones frecuentes, almacenar datos relacionados en colecciones separadas y usar referencias puede ser más adecuado.

Para decidir si incrustar datos o utilizar referencias, evalúe la importancia de estos factores para su aplicación:

- **Mejorar las consultas sobre datos relacionados:** Si su aplicación recupera frecuentemente datos de una entidad para acceder a información sobre una entidad relacionada, incrustar los datos relacionados puede optimizar el rendimiento de las consultas al eliminar la necesidad de operaciones `$lookup` frecuentes.
- **Mejorar los datos devueltos por diferentes entidades:** Si su aplicación normalmente devuelve datos de entidades relacionadas juntos, incrustar estos datos dentro de una sola colección puede agilizar la recuperación de datos.
- **Mejorar el rendimiento de las actualizaciones:** Si su aplicación realiza actualizaciones frecuentes de datos relacionados, almacenar estos datos en colecciones separadas y usar referencias puede resultar ventajoso. Este enfoque evita la duplicación de datos y reduce la carga de trabajo de escritura al requerir actualizaciones en una sola ubicación.

Para este paso del proceso de diseño del esquema, cree un mapa de esquema que muestre el tipo de relación entre esos campos (uno a uno, uno a muchos, muchos a muchos). Por ejemplo, considere el siguiente mapa de esquema para una aplicación de blog:

> **Figura 4.1:** Esquema de aplicación de blog que muestra diferentes relaciones

Dadas estas entidades de datos, ahora se le presenta la opción de incrustar datos relacionados en la misma colección o almacenar datos en colecciones separadas y usar referencias cuando necesite devolver datos de múltiples entidades. La mejor opción depende de los datos que los usuarios necesitan cuando utilizan su aplicación.

#### Optimización de consultas para artículos

En una aplicación de blog, tiene dos entidades de datos clave para conectar: publicaciones de blog y autores. Imagine que los usuarios de su aplicación buscan principalmente publicaciones de blog según palabras clave del título y rara vez buscan por autor. Su aplicación aún necesita devolver la información del autor y la información de la publicación del blog juntas. En este escenario, debe diseñar su esquema para incrustar la información del autor dentro del documento de la publicación del blog. De esta manera, la aplicación puede recuperar toda la información necesaria con una sola operación de lectura. Un documento de muestra con este diseño de esquema se vería de la siguiente manera:

```json
{
  title: "My Favorite Vacation",
  date: ISODate("2023-06-02"),
  text: "We spent seven days in Italy...",
  tags: [
    {
      name: "travel",
      url: "<blog-site>/tags/travel"
    },
    {
      name: "adventure",
      url: "<blog-site>/tags/adventure"
    }
  ],
  comments: [
    {
      name: "pedro123",
      text: "Great article!"
    }
  ],
  author: {
    name: "alice123",
    email: "alice@mycompany.com",
    avatar: "photo1.jpg"
  }
}
```

#### Optimización de consultas para artículos y autores

Imagine un patrón de acceso a datos diferente en el que los usuarios desean ver las publicaciones de blog y la información del autor por separado. Por ejemplo, si las publicaciones de blog no contienen información del autor y, en cambio, la información del autor existe en una página de perfil independiente. En este escenario, sería mejor almacenar los artículos y los autores en colecciones separadas. Este diseño de esquema reduce el trabajo necesario para devolver la información requerida y no incluye campos innecesarios.

El siguiente diseño de esquema utiliza dos colecciones. La colección `articles` contiene un campo `authorId`, que es una referencia a la colección `authors`:

```json
// articles collection
{
     title: "My Favorite Vacation",
     date: ISODate("2023-06-02"),
     text: "We spent seven days in Italy...",
     authorId: 987,
     tags: [
        {
           name: "travel",
           url: "<blog-site>/tags/travel"
        },
        {
           name: "adventure",
           url: "<blog-site>/tags/adventure"
        }
     ],
     comments: [
        {
           name: "pedro345",
           text: "Great article!"
        }
     ]
  }
// authors collection
{
     _id: 987,
     name: "alice123",
     email: "alice@mycompany.com",
     avatar: "photo1.jpg"
}
```

Con este esquema, en caso de que la aplicación necesite devolver la información del artículo y del autor juntas, puede ejecutar una operación `$lookup` para unir los datos de las diferentes colecciones. Este diseño tendrá un rendimiento menor que una operación de lectura en una sola colección, pero está optimizado para el patrón de acceso típico de leer los datos de cada entidad por separado.

#### Modelado de relaciones

Los tipos de relaciones entre entidades que contiene su esquema pueden afectar significativamente la forma en que lo diseña. Las relaciones entre entidades pueden ser de uno a uno, de uno a muchos o de muchos a muchos.

#### Relaciones uno a uno

En una relación uno a uno, un único elemento de una entidad solo puede vincularse a un único elemento de otra entidad. Por ejemplo, país a ciudad capital es una relación uno a uno porque cada país solo tiene una ciudad capital y esa capital pertenece a un solo país. Para las relaciones uno a uno, la incrustación es la forma preferida de modelar datos, especialmente si los datos de las dos entidades se devuelven juntos.

#### Relaciones uno a muchos

En una relación de uno a muchos, un solo elemento de una entidad puede vincularse a múltiples elementos de otra entidad. Por ejemplo, el propietario de un automóvil con respecto a los automóviles es una relación de uno a muchos porque el propietario puede poseer varios automóviles y cada automóvil solo tiene un único propietario. Para modelar relaciones de uno a muchos, tanto la incrustación como las referencias pueden ser opciones de diseño de esquema válidas según sus patrones de acceso.

#### Relaciones muchos a muchos

En una relación de muchos a muchos, múltiples elementos de una entidad pueden vincularse a múltiples elementos de otra entidad. Por ejemplo, los libros respecto a los autores es una relación de muchos a muchos porque un libro puede estar escrito por varios autores y un autor puede haber escrito varios libros. Generalmente, las referencias son útiles para representar relaciones de muchos a muchos porque se adaptan bien al modelado de estructuras de datos jerárquicas complejas.

#### Incrustación frente a referencias (*Embedding versus references*)

Una decisión clave para el diseño de su esquema es si modelar sus datos con incrustación o con referencias. Incrustar datos significa almacenar datos relacionados en un solo documento donde se pueden recuperar con una sola operación de lectura. Por ejemplo, el siguiente documento incrusta datos de contacto y dirección en un solo documento:

```json
// users collection
{
  _id: <ObjectId1>,
  username: "user1",
  contact: {
    phone: "123-456-7890",
    email: "abc@example.com"
  },
  access: {
    level: 5,
    group: "dev"
  }
}
```

Los modelos de datos incrustados pueden resultar útiles en los siguientes escenarios:

- Tiene relaciones de tipo "contiene" entre entidades. Por ejemplo, un documento de contactos que contiene una dirección.
- Tiene relaciones de uno a muchos entre entidades. En estas relaciones, los documentos muchos o secundarios se ven en el contexto de los documentos uno o principales. Por ejemplo, un solo pedido que contiene muchos artículos.

Las referencias, por otro lado, almacenan datos en colecciones separadas y utilizan ciertos campos como enlaces entre entidades. Por ejemplo, en el siguiente esquema, los documentos de contactos y de acceso contienen referencias al documento de usuarios:

```json
// users collection
{
  _id: <ObjectId1>,
  username: "user1"
}
 
// contacts collection
{
  _id: <ObjectId2>,
  user_id: <ObjectId1>,
  phone: "123-456-7890",
  email: "abc@example.com"
}
 
// access collection
{
  _id: <ObjectId3>,
  user_id: <ObjectId1>,
  level: 5,
  group: "dev"
}
```

Las referencias dan como resultado modelos de datos normalizados porque los datos se dividen en múltiples colecciones y no se duplican. Las referencias pueden ser útiles en los siguientes escenarios:

- La incrustación provocaría una duplicación de datos pero no proporcionaría suficientes ventajas de rendimiento de lectura como para compensar las implicaciones de la duplicación. Por ejemplo, cuando los datos incrustados cambian con frecuencia.
- Necesita representar relaciones complejas de muchos a muchos o grandes conjuntos de datos jerárquicos.
- La entidad relacionada se consulta frecuentemente por sí sola. Por ejemplo, si tiene datos de empleados y departamentos, podría considerar incrustar la información del departamento en los documentos de los empleados. Sin embargo, si consulta a menudo una lista de departamentos, su aplicación funcionará mejor con una colección de departamentos independiente vinculada a la colección de empleados mediante una referencia.

#### Aplicación de patrones de diseño de esquemas

Los patrones de diseño de esquemas son formas de optimizar su modelo de datos para los patrones de acceso de su aplicación. Mejoran el rendimiento de la aplicación y reducen la complejidad del esquema. Antes de implementar patrones de diseño de esquemas, considere el problema que está intentando resolver. Cada patrón de diseño de esquema tiene diferentes casos de uso y compensaciones en cuanto a consistencia de datos, rendimiento y complejidad. Por ejemplo, algunos patrones de diseño de esquemas mejoran el rendimiento de escritura, mientras que otros mejoran el rendimiento de lectura. Implementar un patrón sin comprender su aplicación y los datos que necesita puede degradar el rendimiento de la aplicación y causar complicaciones innecesarias en el diseño de su esquema. Esta sección proporciona una descripción general de varios patrones de diseño de MongoDB.

#### El patrón cubo (*The bucket pattern*)

El patrón cubo separa series largas de datos en objetos distintos. Este patrón es más eficaz para gestionar datos de transmisión (*streaming*), como series temporales, análisis en tiempo real o aplicaciones de Internet de las cosas (IoT). Reduce la cantidad total de documentos en una colección, simplifica el acceso a los datos y mejora el rendimiento del índice.

Considere una aplicación de IoT donde un sensor envía lecturas de temperatura todos los días. En lugar de crear un documento nuevo para cada lectura, los datos se pueden agrupar en intervalos mensuales. En el siguiente ejemplo, el mes se indica mediante el ID del documento:

```json
{  
  "_id": "sensor_001_2023_10", // Unique ID (sensor_id + month year bucket)
  "sensor_id": "sensor_001",           // Sensor identifier  
  "location": {  
    "latitude": 37.7749,  
    "longitude": -122.4194  
  },  
  "month": "October",  
  "year": 2023,  
  "readings": [  
    {  
      "timestamp": ISODate("2023-10-01T10:15:00Z"),  
      "temperature": 72.5  
    },  
    {  
      "timestamp": ISODate("2023-10-01T12:30:00Z"),  
      "temperature": 74.1  
    },  
    {  
      "timestamp": ISODate("2023-10-02T08:45:00Z"),  
      "temperature": 68.9  
    }  
    // Additional readings for the bucket go here...  
  ]  
}
```

#### El patrón atributo (*The attribute pattern*)

El patrón atributo agrupa documentos en la misma colección que tienen diferentes campos pero comparten características comunes para ordenar y consultar. Este patrón es especialmente eficaz cuando estos campos diferentes aparecen en un número limitado de documentos.

Considere una plataforma de comercio electrónico con un catálogo de productos. Cada producto tiene un conjunto de atributos que pueden variar según el tipo de producto:

```json
{  
    "_id": ObjectId("652f89b8ac11b816a4cd1234"),
    "name": "Wireless Headphones",  
    "type": "Electronics",  
    "price": 99.99,  
    "attributes": [  
        {  
            "key": "battery_life",  
            "value": "30 hours"  
        },  
        {  
            "key": "connectivity",  
            "value": "Bluetooth 5.0"  
        },  
        {  
            "key": "warranty",  
            "value": "1 year"  
        },  
        {  
            "key": "noise_cancellation",  
            "value": true  
        }  
    ]  
}
```

#### El patrón polimórfico (*The polymorphic pattern*)

El patrón polimórfico utiliza el modelo de datos flexible de MongoDB para permitirle acceder a documentos que tienen diferentes campos o tipos de datos juntos en la misma consulta.

Considere una colección que almacena información financiera y mantiene datos de cuentas y clientes en la misma colección. Cada documento tiene un campo `docType` que indica si el documento es una cuenta o un cliente. Si bien no es necesario crear un campo para indicar el tipo de documento en una colección polimórfica, hacerlo puede ayudar con la organización y la validación del esquema.

```json
// Customer document
{
     "customerId": "CUST-123456789",
     "docType": "customer",
     "name": {
        "title": "Mr",
        "first": "Andrew",
        "middle": "James",
        "last": "Morgan"
     },
     "address": {
        "street1": "240 Blackfriars Rd",
        "city": "London",
        "postCode": "SE1 8NW",
        "country": "UK"
     },
     "customerSince": ISODate("2005-05-20")
  }
// Account document
{
     "accountNumber": "ACC1000000890",
     "docType": "account",
     "accountType": "savings",
     "customerId": [
        "CUST-123456789"
     ],
     "dateOpened": ISODate("2003-12-15"),
     "balance": NumberDecimal("10341.89")
  }
```

#### El patrón de referencia extendida (*The extended reference pattern*)

Si su aplicación a menudo necesita realizar operaciones `$lookup` repetitivas en múltiples colecciones, el patrón de referencia extendida puede ayudar a reducir las operaciones `$lookup` al mover campos a los que se accede con frecuencia al documento principal. Como resultado, su aplicación puede lograr lecturas más rápidas al necesitar acceder únicamente a una sola colección.

Considere una plataforma de comercio electrónico con productos y pedidos. Una referencia simple entre estas colecciones estaría en el campo `_id`. Con el patrón de referencia extendida, el esquema incluye el nombre y el precio del producto además del campo `_id`. Con este diseño de esquema, cuando la aplicación muestra un pedido, puede mostrar el nombre y el precio del producto sin obtener todos los detalles del producto:

```json
// products collection
{
  "_id": ObjectId("652f89b8ac22b816a4cd0011"),
  "name": "Wireless Headphones",
  "price": 99.99,
  "category": "Electronics",
  "in_stock": 250
},
{
  "_id": ObjectId("652f89b8ac22b816a4cd0022"),
  "name": "Gaming Mouse",
  "price": 49.99,
  "category": "Accessories",
  "in_stock": 500
}
 
// orders collection:
{
  "_id": ObjectId("652f89d9ac33b816a4cd1234"),
  "order_date": ISODate("2023-10-01T14:32:00Z"),
  "customer_id": ObjectId("652f89b8ac22b816a4cd5678"),
  "products": [
    {
      "product_id": ObjectId("652f89b8ac22b816a4cd0011"),
      "name": "Wireless Headphones",  // Extended reference field
      "price": 99.99,                 // Extended reference field
      "quantity": 2
    },
    {
      "product_id": ObjectId("652f89b8ac22b816a4cd0022"),
      "name": "Gaming Mouse",  // Extended reference field
      "price": 49.99,         // Extended reference field
      "quantity": 1
    }
  ],
  "total": 249.97
}
```

#### El patrón de aproximación (*The approximation pattern*)

Utilice el patrón de aproximación cuando tenga valores que cambien con frecuencia pero los usuarios no necesiten conocer valores precisos. En lugar de actualizar los valores cada vez que cambian los datos, el patrón de aproximación actualiza los datos basándose en una granularidad mayor, lo que genera menos actualizaciones y una menor carga de trabajo de la aplicación.

Considere un sitio web que rastrea la cantidad de vistas y visitantes en sus artículos. Debido al alto tráfico, actualizar las métricas exactas en tiempo real para cada vista puede resultar muy demandante. En su lugar, puede realizar un seguimiento de un agregado diario de sus métricas clave para mantener una vista aproximada de los valores:

```json
{  
    "_id": ObjectId("652f89d9ac33b816a4cd1234"),
    "date": ISODate("2023-10-01"),
    "page": "/home",
    "aggregates": {  
        "page_views": 15432,
        "unique_visitors": 5432,
        "avg_time_spent": 215 
    }  
}
```

#### El patrón calculado (*The computed pattern*)

El patrón calculado implica precalcular los resultados de ciertos cálculos y almacenar esos resultados en el documento, en lugar de calcular valores como parte de la operación de lectura. Si un valor calculado se solicita con frecuencia, puede ser más eficiente guardar ese valor en la base de datos con anticipación. Cuando la aplicación solicita datos, solo se requiere una operación de lectura.

Considere datos de inventario donde la aplicación almacena el valor total de los artículos mantenidos en stock. El cliente podría realizar este cálculo multiplicando el precio de cada artículo por la cantidad en stock, pero si estos datos se solicitan con frecuencia, puede ser útil calcular este valor en el servidor y almacenarlo como parte del documento mismo:

```json
{  
    "_id": ObjectId("652f89b8ac11b816a4cd1234"),  
    "name": "Wireless Headphones",  
    "price": 99.99,  
    "in_stock": 250,  
    "stock_value": 24997.5  // Computed field
}
```

#### El patrón valor atípico (*The outlier pattern*)

Si su colección almacena documentos generalmente del mismo tamaño y forma, un documento drásticamente diferente (un valor atípico) puede causar problemas de rendimiento en consultas comunes. Utilice el patrón de valor atípico para aislar documentos que no coinciden con la forma esperada del resto de su colección.

Considere una aplicación que rastrea las ventas de libros. La mayoría de los libros tienen alrededor de 50 ventas, pero un valor atípico tiene 2,000 ventas. Dependiendo del diseño del esquema, esto podría crear un documento grande que afectaría negativamente el rendimiento de las consultas regulares. En su lugar, puede separar los documentos de ventas en colecciones independientes para evitar matrices gigantescas:

```json
// sales collection
{
     "_id": 2,
     "title": "The Wooden Amulet",
     "year": 2023,
     "author": "Lesley Moreno",
     "customers_purchased": [ "user00", "user01", "user02", ... "user49" ],
     "has_extras": true
}
// sales_extra collection
{
     "book_id": 2,
     "customers_purchased_extra": [ "user50", "user51", "user52", ... "user999" ]
}
```

#### El patrón subconjunto (*The subset pattern*)

El patrón de subconjunto mejora el acceso a los datos para aplicaciones que devuelven un número predeterminado de elementos, como un número determinado de reseñas de productos. Este patrón es particularmente útil cuando se manejan documentos grandes que tienen muchos elementos incrustados.

Considere un sistema de biblioteca donde cada libro tiene cientos de reseñas. Sin embargo, al mostrar una lista de libros, la aplicación solo muestra las últimas cinco reseñas para una navegación rápida. Esas cinco reseñas se pueden incrustar en el documento del libro, y otras reseñas se pueden almacenar en una colección separada y recuperarse según sea necesario mediante una operación `$lookup`.

```json
{
  "_id": ObjectId("549f1faabcf88cd799433037"),
  "title": "The Great Gatsby",
  "author": "F. Scott Fitzgerald",
  "subset_reviews": [
    {
      "user": "coolreader123",
      "rating": 5,
      "comment": "Great!",
      "date": ISODate("2023-10-01T10:00:00Z")
    },
    {
      "user": "cozyreading",
      "rating": 4,
      "comment": "Good read.",
      "date": ISODate("2023-09-29T08:30:00Z")
    }
    // ... 3 more recent reviews
  ]
}
```

#### El patrón de control de versiones de documentos (*The document versioning pattern*)

El patrón de control de versiones de documentos proporciona una forma de mantener un historial de versiones anteriores de los datos. Por ejemplo, una compañía de seguros podría utilizar el patrón de versiones de documentos para mantener un registro de las modificaciones que un cliente realiza en su póliza. En este ejemplo, la versión más actual de la póliza se almacena en su propia colección y el historial de la póliza se almacena en una colección separada:

```json
// currentPolicies collection
{
      _id: ObjectId("6626c9832a98aba8ddec31d3"),
      customerName: 'Michelle',
      dateSet: ISODate("2024-04-22T20:32:43.232Z"),
      itemsInsured: [ 'car', 'watch' ],
      policyId: 1,
      revision: 3
 }
// policyRevisions collection
{
  _id: ObjectId("6626c8f02a98aba8ddec31d1"),
  policyId: 1,
  customerName: 'Michelle',
  revision: 1,
  itemsInsured: [ 'golf clubs', 'car' ],
  dateSet: ISODate("2024-04-22T20:30:40.809Z")
},
{
  _id: ObjectId("6626c92b2a98aba8ddec31d2"),
  customerName: 'Michelle',
  dateSet: ISODate("2024-04-22T20:31:03.000Z"),
  itemsInsured: [ 'golf clubs', 'car', 'watch' ],
  policyId: 1,
  revision: 2
},
{
  _id: ObjectId("6626c9832a98aba8ddec31d3"),
  customerName: 'Michelle',
  dateSet: ISODate("2024-04-22T20:32:43.232Z"),
  itemsInsured: [ 'car', 'watch' ],
  policyId: 1,
  revision: 3
}
```

#### El patrón de control de versiones de esquemas (*The schema versioning pattern*)

El patrón de versiones de esquema le permite dar cuenta de los cambios en los requisitos de la aplicación almacenando múltiples versiones de un esquema en la misma colección. Cada documento tiene un campo que indica su esquema, el cual se puede incorporar a la lógica de la consulta para consultar los campos correctos.

```json
{ 
   _id: 1, 
   schemaVersion: 1, 
   name: 'Taylor', 
   home: '209-555-7788', 
   work: '503-555-0110' 
}, 
{ 
   _id: 2, 
   schemaVersion: 2, 
   name: "Cameron", 
   contactInfo: { 
      cell: "903-555-1122", 
      work: "670-555-7878", 
      instagram: "@camcam9090", 
      linkedIn: "CameronSmith123" 
   } 
}
```

#### El patrón árbol (*The tree pattern*)

El patrón de árbol utiliza matrices para almacenar la ruta completa desde un nodo hasta la parte superior de la jerarquía. Este patrón es útil para representar estructuras jerárquicas como un organigrama de empleados o una base de datos de productos con categorías anidadas.

```json
{
  _id: <ObjectId>,
  name: "Samsung EVO",
  price: {
    value: NumberDecimal("169.99"),
    currency: "USD"
  },
  parent_category: "Solid state drives",
  ancestor_categories: [
    "Solid state drives",
    "Hard drives",
    "Storage",
    "Computers",
    "Electronics"
  ]
}
```

#### Antipatrones de diseño de esquemas

Los antipatrones de diseño de esquemas son formas ineficientes de estructurar el esquema de su base de datos. Pueden generar complejidad innecesaria y causar problemas de rendimiento. Reconocer y evitar los antipatrones de diseño de esquemas puede ayudar a crear aplicaciones con mejor rendimiento. Esta sección describe los antipatrones de esquemas comunes y cómo evitarlos.

- **Matrices sin límites (*Unbounded arrays*):** Las matrices sin límites ocurren cuando el diseño de su esquema hace que un campo de matriz crezca indefinidamente. Un patrón común que puede conducir a matrices ilimitadas es si incrusta reseñas de un producto en el documento del producto. Sin una lógica de esquema adicional, a medida que se escriben más reseñas para el producto, la matriz crece indefinidamente, lo que genera documentos grandes que empeoran el rendimiento. Una forma común de evitar matrices ilimitadas es utilizar el patrón de subconjunto descrito anteriormente en este capítulo.
- **Índices innecesarios:** Si crea un índice para cada consulta posible que ejecuta su aplicación, podría terminar con índices innecesarios, lo que puede degradar el rendimiento de la base de datos. Para evaluar el uso de índices, utilice la etapa de agregación `$indexStats`. También puede ocultar índices para evaluar el impacto en la base de datos antes de eliminarlos por completo.
- **Demasiadas colecciones:** Cada colección que crea en MongoDB viene con un índice `_id` predeterminado, que ocupa almacenamiento adicional. Si bien el índice `_id` es pequeño, tener colecciones excesivas puede causar una sobrecarga innecesaria en el almacenamiento de su sistema. Para reducir la cantidad de colecciones, considere implementar incrustación o patrones de acceso a datos como el patrón cubo, según sea adecuado para su aplicación.
- **Documentos inflados (*Bloated documents*):** Almacenar campos de datos que están relacionados entre sí pero a los que no se accede juntos puede crear documentos inflados que conducen a un uso excesivo de RAM y ancho de banda. Para evitar documentos inflados, reestructure su esquema en múltiples colecciones y use referencias de documentos para separar campos que no se devuelven juntos. Este enfoque reduce el tamaño del conjunto de trabajo (*working set*) y mejora el rendimiento.
- **Operaciones $lookup excesivas:** Si bien la operación `$lookup` es útil cuando se usa con poca frecuencia, puede ser lenta y consumir muchos recursos en comparación con las operaciones que solo consultan una sola colección. Si utiliza a menudo operaciones `$lookup`, considere reestructurar su esquema para almacenar datos relacionados en una sola colección.

---

### Sección 3: Índices

Cada sistema de base de datos se basa en la indexación para mejorar el rendimiento de las consultas. La optimización de índices es el proceso de crear y refinar índices para satisfacer mejor las necesidades de su aplicación. Optimizar los índices es crucial para utilizar MongoDB a su máxima velocidad y potencial. MongoDB presenta un sistema de indexación granular y potente que puede mejorar el rendimiento general de la base de datos.

Un índice es una estructura de datos especial que permite un acceso más rápido a los datos de una colección, de forma muy parecida al índice de una enciclopedia. Es una lista ordenada de referencias a los contenidos reales (los documentos), lo que permite a MongoDB consultar mucho más rápido. Los índices almacenan valores ordenados de un campo específico o de un conjunto de campos. MongoDB puede devolver resultados ordenados utilizando el propio orden del índice, lo que significa que los índices no solo pueden mejorar el rendimiento al leer datos, sino también al ordenarlos.

Los índices de MongoDB utilizan una estructura de datos conocida como árbol B (*B-tree*). Los árboles B se autoequilibran, lo que permite operaciones eficientes como búsqueda, inserción, eliminación y acceso secuencial, todo en tiempo logarítmico. Puede pensar en un índice como un mapa ordenado de pares clave-valor, donde cada clave corresponde al valor de un campo indexado y el valor apunta al documento asociado. Al igual que el índice de un libro, las claves se almacenan en orden y se asocian con uno o más campos.

Esta estructura basada en árboles ayuda a minimizar la cantidad de comparaciones necesarias durante las búsquedas. Sin un índice, cada nuevo documento agregado a una colección aumenta el trabajo requerido para buscar resultados coincidentes. Pero con un árbol B, las nuevas entradas se colocan de una manera que preserva el orden y el equilibrio, por lo que las consultas siguen siendo eficientes sin agregar una sobrecarga innecesaria.

#### Determinar si una consulta utilizó un índice

Antes de crear un índice, rellene una base de datos con algunos datos de prueba. Utilizará estos datos para determinar si sus consultas utilizan un índice. Ejecute el siguiente script en el Shell de MongoDB para rellenar la colección `companies` dentro de la base de datos `test` con documentos de muestra:

```javascript
use test
 
for (i = 0; i < 1000; i++) {
  db.companies.insertOne({
    "i": i,
    "name": "Company_" + i,
    "ticker": "CMPY_" + i,
    "revenue": Math.floor(Math.random() * 1000000),
    "created": new Date(),
    "meta": {
      "employees": Math.floor(Math.random() * 100),
      "acquisitions": Math.floor(Math.random() * 5)
    },
    "info": "Lorem ipsum dolor sit amet"
  })
}
```

El script inserta 1,000 compañías con algunos datos simulados: nombre, ingresos y algunos campos `meta` incrustados para la cantidad de empleados y adquisiciones.

Puede ejecutar una consulta con el comando `explain` para ver cómo realiza MongoDB una consulta sin ningún índice. En el Shell de MongoDB, ejecute el siguiente comando:

```javascript
db.companies
  .find({name: 'Company_456'})
  .explain('executionStats')
```

Para analizar el uso del índice, los campos más importantes a examinar en la salida son los siguientes:

- `totalKeysExamined`, que es el número de claves de índice utilizadas para cumplir la consulta.
- `totalDocsExamined`, que es el número de documentos examinados durante la ejecución de la consulta.
- El plan ganador (*winning plan*) de la consulta, que contiene `IXSCAN` si se utilizó un índice.

Sin un índice, `totalKeysExamined` es 0, `totalDocsExamined` es la cantidad de documentos en la colección y el plan ganador es una etapa `COLLSCAN`. Estos atributos indican que la consulta realizó un escaneo de colección, lo cual no es muy eficiente.

Generalmente, querrá crear índices para que se alineen con la forma de consulta esperada (*query shape*). Más precisamente, un índice respalda una consulta cuando contiene todos los campos escaneados por la consulta. En este caso, puede crear su primer índice en el campo del nombre de la compañía:

```javascript
db.companies.createIndex( { "name": 1 } )
```

Este comando crea un índice de un solo campo en el campo `name`, donde el número 1 indica el orden de clasificación. Este índice de ejemplo está en orden ascendente. Si desea que su aplicación acceda a los datos en orden descendente, puede establecer el orden de clasificación en -1.

Después de crear el índice, si vuelve a ejecutar la consulta anterior, debería ver los siguientes valores en `executionStats`:

- `totalKeysExamined: 1`
- `totalDocsExamined: 1`
- `winningPlan.inputStage.stage: IXSCAN`

Ahora, la consulta utiliza el índice y solo necesita examinar un único documento para devolver los resultados. La consulta devuelve resultados más rápido y el rendimiento mejora aún más a medida que aumenta la cantidad de documentos en la colección.

En una base de datos con muchos documentos complejos, los índices son necesarios. Sin embargo, los índices empeoran el rendimiento de escritura, así que tenga cuidado al crear índices en campos que se actualizan con frecuencia. Comprender la proporción de lectura:escritura de su aplicación es fundamental para crear índices eficaces y de alto rendimiento. La elección adecuada de los índices es una parte importante para dominar MongoDB. Más adelante en este capítulo, recibirá pautas sobre cómo y dónde crear índices.

#### Tipos de índices

La siguiente sección describe los diferentes tipos de índices disponibles en MongoDB y explica los diferentes tipos de consultas que admiten.

#### Índices de campo único (*Single-field indexes*)

Los índices de campo único almacenan información de un solo campo en una colección. Por defecto, todas las colecciones tienen un índice único en el campo `_id`. Este índice evita que se inserten en la colección documentos con valores `_id` duplicados.

En la sección anterior, aprendió a crear un índice de un solo campo en un campo de documento de nivel superior. También puede crear un índice de un solo campo en un campo de documento incrustado. Cuando lo haga, debe poner el nombre del campo entre comillas. Por ejemplo, el siguiente comando crea un índice en el campo `meta.employees`:

```javascript
db.companies.createIndex( { "meta.employees": 1 } )
```

También puede crear un índice de un solo campo en documentos incrustados como un todo, como se muestra en el siguiente ejemplo:

```javascript
db.companies.createIndex( { "meta": 1 } )
```

Sin embargo, solo las consultas que especifican el documento incrustado completo utilizan el índice. Las consultas en un campo específico dentro del documento no utilizan el índice. Esto puede provocar comportamientos inesperados si su modelo de esquema cambia y agrega o elimina campos de su documento indexado. Antes de crear un índice en un documento incrustado, considere si en su lugar debería indexar campos específicos en ese documento.

#### Índices compuestos (*Compound indexes*)

Los índices compuestos recopilan y ordenan datos de dos o más campos en cada documento de una colección. Los datos se agrupan según el orden en que aparecen los campos en el índice.

El índice compuesto suele ser la mejor solución para los casos en los que espera que se utilicen varios campos en sus consultas. Por ejemplo, basándose en nuestro ejemplo de empresas de esta sección, puede crear un índice compuesto por los campos `name` y `ticker`:

```javascript
db.companies.createIndex( { "name": 1, "ticker": 1 } )
```

Con los índices compuestos, el orden del índice determina si el índice puede admitir una operación de ordenación. Los índices compuestos admiten operaciones de ordenación que coinciden con el orden de ordenación del índice o con el orden de ordenación inverso del índice.

El índice compuesto anterior admite ordenaciones en las siguientes combinaciones de campos:

```javascript
db.companies.find().sort({ "name": 1,"ticker": 1 })
db.companies.find().sort({ "name": -1,"ticker": -1 })
```

El índice no se utiliza para las siguientes ordenaciones, porque el orden de ordenación no coincide con el orden del índice:

```javascript
db.companies.find().sort({ "name": 1,"ticker": -1 })
db.companies.find().sort({ "name": -1,"ticker": 1 })
```

#### Prefijos de índice (*Index prefixes*)

Los prefijos de índice son los subconjuntos iniciales de campos indexados. Los índices compuestos admiten consultas en todos los campos incluidos en el prefijo del índice. Por ejemplo, el índice compuesto del ejemplo anterior puede admitir una consulta en los campos de autor e ISBN, o solo en el campo de autor porque es parte del prefijo del índice. Consultar solo por el código ISBN no utilizará el índice y, en su lugar, realizará un escaneo de la colección.

Para ayudar a eliminar índices redundantes, busque cualquier índice que esté contenido en el prefijo de otro índice. Por ejemplo, considere los siguientes índices:

```javascript
{ "name": -1, "ticker": 1 }
{ "name": -1 }
```

Los índices anteriores tienen un prefijo común de `{ "name": -1 }`. Si ninguno de los índices tiene una restricción de unicidad o dispersión (*sparse*), puede eliminar el índice en el prefijo `{ "name": -1 }`. MongoDB utiliza el índice compuesto en todas las situaciones en las que habría utilizado el índice de prefijo.

#### Directriz ESR (Igualdad, Ordenación, Rango)

Al elegir el orden en el que colocar los campos en su índice compuesto, seguir la directriz ESR (*Equality, Sort, Range*) ayuda a crear un índice compuesto eficiente.

- **Equality (Igualdad):** Se refiere a una coincidencia exacta en un solo valor. Las búsquedas de índices hacen un uso eficiente de las coincidencias exactas para limitar el número de documentos que deben examinarse para completar una consulta. Coloque primero en su índice los campos que requieren coincidencias exactas.
- **Sort (Ordenación):** Determina el orden de los resultados. La ordenación sigue a las coincidencias de igualdad porque las coincidencias de igualdad reducen la cantidad de documentos que deben ordenarse. Ordenar después de las coincidencias de igualdad también le permite a MongoDB realizar una ordenación sin bloqueos.
- **Range (Rango):** Los filtros de rango escanean campos. El escaneo no requiere una coincidencia exacta, lo que significa que los filtros de rango están vinculados de forma flexible a las claves de índice. Para mejorar la eficiencia de las consultas, limite los límites del rango y utilice coincidencias de igualdad para reducir la cantidad de documentos a escanear. Coloque los campos que se utilizan para coincidencias de rango al final de su índice.

Usando el ejemplo de la empresa de principios de esta sección, imagine que la aplicación necesitara realizar una consulta con la siguiente forma:

```javascript
db.companies.find( {
  "meta.employees": { $gte: 20 },
  "name": "Company_45"
} ).sort( { "revenue": -1 } )
```

Si seguimos la directriz ESR, el siguiente índice sería óptimo para respaldar esta consulta:

```javascript
db.companies.createIndex( {
  "name": 1,
  "revenue": -1,
  "meta.employees": 1
  
} )
```

- La consulta realiza una coincidencia de igualdad en el campo `name`, lo que significa que debe ser el primer campo en el índice.
- La consulta realiza una ordenación en el campo `revenue`, lo que significa que debe ser el siguiente campo en el índice.
- La consulta contiene un filtro de rango en `meta.employees`, lo que significa que debe ser el último campo en el índice.

#### Índices multiclave (*Multikey indexes*)

La capacidad de MongoDB para almacenar matrices de valores se extiende también a la indexación. Para mejorar el rendimiento de las consultas sobre valores de matriz, puede crear un índice multiclave.

La base de datos de ejemplo no tiene matrices; sin embargo, puede insertar el siguiente documento:

```javascript
db.companies.insertOne({
  "name": "Company with tags",
  "ticker": "TAGS",
  "revenue": 23000,
  "tags": ["Finance", "Insurance", "Banking"]
})
```

Para crear un índice multiclave, utilice el mismo comando que para un índice de un solo campo y especifique un campo de matriz en la clave del índice:

```javascript
db.companies.createIndex( { "tags": 1 } )
```

También puede crear índices multiclave compuestos, con la limitación de que solo un campo del índice puede ser un campo de matriz. He aquí un ejemplo:

```javascript
db.companies.createIndex( { "name": 1, "tags": 1 } )
```

#### Búsqueda de texto (*Text search*)

MongoDB admite la búsqueda de texto completo a través de índices de texto, que son índices especiales en campos de valores de cadena. Todos los despliegues de MongoDB admiten índices de texto estándar; sin embargo, con MongoDB Atlas, puede crear índices de Atlas Search, que son una solución de consulta de texto completo mejorada.

#### Índices de texto

Los índices de texto admiten consultas de búsqueda de texto en campos que contienen contenido de cadena. Los índices de texto mejoran el rendimiento cuando busca palabras o cadenas específicas dentro del contenido de una cadena. Una colección solo puede tener un índice de texto, pero ese índice puede incluir varios campos.

Para crear un índice de texto, establezca el valor de la clave del índice en `text`. He aquí un ejemplo:

```javascript
db.companies.createIndex( { "info": "text" } )
```

Los índices de texto aplicarán lematización de palabras (*word stemming*, eliminando sufijos comunes y palabras vacías o *stop words*, como *a*, *an*, *the*). Cuando crea un índice de texto en varios campos, puede especificar la importancia de cada campo de texto. Por ejemplo, puede aplicar más peso a las coincidencias en el título o subtítulo de un documento en comparación con las mismas coincidencias en el párrafo del cuerpo de un artículo.

Dado que MongoDB realiza una búsqueda de índice completa para cada término de búsqueda individual, el rendimiento del índice de texto es directamente proporcional al número de términos de búsqueda. Para mejorar el rendimiento de una consulta de texto, limite la cantidad de documentos utilizando un índice compuesto. Por ejemplo, puede incluir el autor del documento o un rango de fechas en el índice, particionando así los documentos y acelerando la búsqueda del índice de texto al reducir el espacio de búsqueda.

Considere los siguientes factores adicionales al crear índices de texto:

- Los índices de texto pueden utilizar grandes cantidades de RAM porque crean una entrada de índice independiente para cada palabra lematizada única en cada campo indexado de cada documento insertado.
- Crear un índice de texto lleva más tiempo que crear un índice ordenado (escalar) simple sobre los mismos datos.
- Los índices de texto dividen las cadenas de texto en palabras individuales, sin almacenar frases ni información sobre qué tan cerca están las palabras entre sí en el documento original. Por lo tanto, las consultas con múltiples términos de búsqueda se ejecutan de manera más eficiente cuando toda la colección se puede mantener en la memoria.

#### Índices de Atlas Search

MongoDB Atlas tiene un índice de búsqueda de texto completo basado en Apache Lucene, que tiene capacidades adicionales en comparación con la indexación de texto de MongoDB. Los índices de Atlas Search crean asignaciones entre términos y documentos que contienen esos términos, lo que permite una recuperación rápida de documentos utilizando identificadores.

#### Índices TTL (Time-To-Live)

Los índices de tiempo de vida (*Time-To-Live* o TTL) son índices especiales que eliminan documentos automáticamente después de un período de tiempo específico. Estos índices son útiles para eliminar documentos que representan datos como sesiones de usuario o registros (*logs*), donde normalmente solo los datos más recientes son útiles.

Utilizando la colección de empresas de muestra, podría crear un índice en el campo `created` y especificar `60 * 60 * 24 = 86,400` segundos, lo que equivale a un día:

```javascript
db.companies.createIndex(
  { "created": 1 },
  { expireAfterSeconds: 86400 }
)
```

El índice anterior elimina todas las empresas con un valor de fecha y hora `created` que sea anterior al período `expireAfterSeconds` de 86,400 segundos o un día. La comprobación de eliminación se ejecuta como un trabajo en segundo plano cada 60 segundos. Tenga en cuenta que si espera eliminar una gran cantidad de documentos con un índice TTL, es posible que desee programar la eliminación en lotes.

Los índices TTL y las colecciones limitadas (*capped collections*) tienen propiedades similares en el sentido de que ambos eliminan datos antiguos para dejar espacio para datos nuevos. Generalmente, los índices TTL ofrecen mejor rendimiento y mayor flexibilidad que las colecciones limitadas y deben preferirse sobre las colecciones limitadas cuando sea posible.

#### Índices parciales (*Partial indexes*)

Los índices parciales solo indexan los documentos de una colección que cumplen con una expresión de filtro especificada. Al indexar un subconjunto de documentos en una colección, los índices parciales tienen menores requisitos de almacenamiento y reducen los costos de rendimiento para la creación y el mantenimiento de índices.

Por ejemplo, puede crear un índice en los campos `employees` y `name` de sus documentos, pero solo para empresas con más de 50 empleados:

```javascript
db.companies.createIndex(
  {
    "meta.employees": 1,
    "name": 1
  },
  {
    partialFilterExpression: {
      "meta.employees": { $gt: 50 }
    }
  }
)
```

Para consultas que podrían usar solo un subconjunto de los datos, como el estado de red social más reciente o las últimas noticias, un índice parcial puede ser una forma eficiente de devolver resultados.

MongoDB también proporciona índices dispersos (*sparse indexes*), que solo contienen entradas para documentos que tienen el campo indexado. Los índices parciales proporcionan un superconjunto de la funcionalidad de los índices dispersos y generalmente deberían preferirse a los índices dispersos.

#### Índices geoespaciales (*Geospatial indexes*)

Los índices geoespaciales admiten consultas sobre datos de ubicación, ya sea GeoJSON o pares de coordenadas heredados de MongoDB. Utilice estos índices para crear aplicaciones conscientes de la ubicación geográfica (*geo-aware*).

- **Índices 2d:** admiten consultas sobre datos almacenados como puntos en un plano bidimensional. Los posibles casos de uso de los índices 2d incluyen cálculos en un gráfico 2d o el análisis de similitudes visuales entre dos obras de arte.
- **Índices 2dsphere:** admiten consultas geoespaciales en una esfera similar a la Tierra. Pueden determinar puntos dentro de un área específica, calcular la proximidad a un punto específico y muchos otros cálculos útiles.

MongoDB proporciona varios operadores de consulta para usar con índices geoespaciales para realizar consultas geoespaciales. He aquí algunos ejemplos:

- **`$near`:** Devuelve puntos o formas que están cerca de una geolocalización especificada.
- **`$nearSphere`:** Devuelve objetos geoespaciales que están cerca de un punto en una esfera.
- **`$geoWithin`:** Devuelve todos los puntos dentro de un área especificada.
- **`$geoIntersects`:** Determina si un área especificada se cruza con otras áreas y objetos geoespaciales.

#### Índices comodín (*Wildcard indexes*)

Los índices comodín le permiten crear un índice en campos desconocidos o arbitrarios. Son útiles cuando no conoce los campos exactos que estarán presentes en el documento, o si espera que los campos del documento cambien con el tiempo.

Para crear un índice comodín en todos los campos del documento, puede utilizar el siguiente comando:

```javascript
db.companies.createIndex( { "$**": 1 } )
```

Sin embargo, los índices comodín no funcionan tan bien como los índices dirigidos en campos específicos. El índice comodín anterior puede funcionar en un conjunto de datos pequeño, pero en un conjunto de datos grande con muchos campos de documentos, el rendimiento podría no ser óptimo.

En cambio, un caso de uso común para los índices comodín es indexar un objeto incrustado específico donde los subcampos pueden variar entre documentos. Por ejemplo, podemos imaginar que el campo `meta` en los datos de muestra podría contener diferentes subcampos para diferentes compañías. Tendría sentido crear un índice comodín en el campo `meta` para admitir consultas en todos los subcampos `meta` posibles:

```javascript
db.companies.createIndex( { "meta.$**": 1 } )
```

Los índices comodín son alternativas útiles en escenarios donde su aplicación requeriría una gran cantidad de índices individuales para cubrir todas las consultas posibles. Sin embargo, los índices comodín no reemplazan de manera efectiva la planificación de índices basada en la carga de trabajo. Si su colección contiene nombres de campos arbitrarios que impiden índices dirigidos, considere remodelar su esquema para tener nombres de campos consistentes.

#### Índices ocultos (*Hidden indexes*)

Los índices ocultos no son visibles para el planificador de consultas y no se pueden utilizar para respaldar una consulta. A medida que itera el diseño y los índices de su esquema, ocultar índices es útil para evaluar el impacto de eliminar un índice antes de comprometerse por completo a eliminarlo.

Para ocultar un índice existente, use el comando `hideIndex`. He aquí un ejemplo:

```javascript
db.companies.hideIndex("name_1_tags_1")
```

El argumento para el comando `hideIndex` puede ser el nombre del índice o el documento de especificación de la clave del índice. Los índices se mantienen por completo mientras están ocultos, por lo que los índices ocultos todavía afectan negativamente el rendimiento de escritura.

#### Índices hash (*Hashed indexes*)

Los índices hash se utilizan en despliegues fragmentados para distribuir datos entre fragmentos en función de los hashes de los valores de las claves de fragmentación. El campo que elija para su clave de fragmentación hash debe tener una alta cardinalidad, lo que significa una gran cantidad de valores diferentes. La indexación hash es ideal para claves de fragmentación con campos que cambian monótonamente, como valores `ObjectId` o marcas de tiempo.

Para crear un índice hash en el campo `_id`, ejecute el siguiente comando:

```javascript
db.orders.createIndex( { _id: "hashed" } )
```

#### Índices agrupados (*Clustered indexes*)

Los índices agrupados se utilizan para crear colecciones agrupadas, que almacenan documentos indexados en el mismo archivo de WiredTiger que la especificación del índice. Este patrón de almacenamiento proporciona beneficios de rendimiento en comparación con los índices regulares porque las inserciones, actualizaciones y eliminaciones solo requieren una única operación de escritura. La única clave de índice agrupado permitida es `{ _id: 1 }`.

El siguiente ejemplo crea una colección agrupada llamada `events`:

```javascript
db.createCollection("events", {
  clusteredIndex: {
    key: { _id: 1 },
    unique: true
  }
})
```

#### Identificar consultas comunes

Hay múltiples herramientas disponibles para analizar la carga de trabajo de su clúster e identificar consultas comunes. Identificar consultas comunes le ayuda a decidir qué consultas estarían mejor respaldadas por índices.

- El **generador de perfiles de la base de datos (*Database profiler*)** recopila información detallada sobre los comandos de la base de datos que se ejecutan en su clúster, incluidos los comandos CRUD, de configuración y de administración. Habilitar el generador de perfiles y analizar su salida puede proporcionar una descripción general de las operaciones de lectura comunes que se beneficiarían de un índice.
- MongoDB 8.0 introduce la etapa de agregación `$queryStats`, que devuelve estadísticas de tiempo de ejecución para las consultas registradas. Las consultas se agrupan por forma de consulta (*query shape*), lo que le ayuda a identificar los campos comunes utilizados para consultar y ordenar.

El siguiente ejemplo muestra cómo utilizar la etapa `$queryStats`. Tenga en cuenta que el comando debe ejecutarse en la base de datos `admin`:

```javascript
db.getSiblingDB("admin").aggregate( [
   {
      $queryStats: { }
   }
] )
```

#### Configuraciones de consultas persistentes para índices (*Persistent query settings for indexes*)

MongoDB 8.0 introduce una nueva característica que le permite imponer un control más estricto sobre cómo se ejecutan sus consultas. Específicamente, puede forzar a sus consultas a usar un índice específico para todas las consultas que coincidan con una forma de consulta particular. Una forma de consulta (*query shape*) es un conjunto de especificaciones que agrupan consultas similares, como filtros, ordenaciones y proyecciones.

La idea de forzar una consulta a utilizar un índice particular se denomina "sugerencia de índice" (*index hinting*). Una sugerencia de índice anula el proceso de selección de índice predeterminado de MongoDB y hace que la consulta utilice un índice específico, asumiendo que ese índice es elegible para respaldar la consulta.

En la mayoría de los escenarios, las consultas no requieren sugerencias de índice porque la selección de índice predeterminada da como resultado la consulta más eficiente. Sin embargo, si la consulta puede elegir entre múltiples índices posibles, una sugerencia de índice puede ser útil para garantizar un rendimiento óptimo.

Considere el siguiente índice y consulta:

```javascript
db.companies.createIndex( { i: 1, revenue: 1 } )
db.companies.find( {
  i: { $gte: 30 }
} ).sort( { revenue: 1 } )
```

El siguiente ejemplo muestra cómo configurar los ajustes de consulta de su clúster para que las consultas que coincidan con la forma anterior siempre utilicen el mismo índice:

```javascript
db.adminCommand( {
  setQuerySettings: {
    find: "companies",
    filter: {
      i: { $gte: 30 }
    },
    sort: {
      revenue: 1
      },
      $db: "test"
   },
   settings: {
      indexHints: {
       ns: { db: "test", coll: "companies" },
       allowedIndexes: [ "i_1_revenue_1" ]
      },
      queryFramework: "classic",
      comment: "Index hint for i_1_revenue_1 index to improve query performance"
   }
} )
```

#### Consultas cubiertas (*Covered queries*)

Una consulta cubierta es una consulta que se puede satisfacer completamente utilizando un índice y no tiene que examinar ningún documento. Un índice cubre una consulta cuando se cumplen todas las condiciones siguientes:

- Todos los campos de la consulta son parte de un índice.
- Todos los campos devueltos en los resultados están en el mismo índice.
- Ningún campo en la consulta es igual a nulo.

Por ejemplo, considere el siguiente índice:

```javascript
db.companies.createIndex( { i: 1, name: 1 } )
```

El índice cubre la siguiente consulta:

```javascript
db.companies.find(
  {
    i: 30,
    name: "Company_30"
  },
  { name: 1, _id: 0 }
).explain("executionStats")
```

Este índice cubre la consulta porque solo consulta sobre los campos indexados, `i` y `name`. Además, la consulta solo devuelve el campo `name`. Para que la consulta esté cubierta, la proyección de la consulta debe omitir explícitamente el campo `_id` porque `_id` no está incluido en el índice.

Para confirmar que la consulta fue cubierta, examine el `executionStats` de la consulta y observe que `totalKeysExamined` es 1 y `totalDocsExamined` es 0.

Recuerde que las consultas siempre devuelven `_id` de forma predeterminada. Para lograr una consulta cubierta, debe excluir explícitamente el campo `_id` de los resultados de la consulta o agregarlo al índice.

#### Estrategias de indexación

Para ayudar a crear índices efectivos, revise las siguientes estrategias y mejores prácticas:

- Asegúrese de que sus índices respalden sus consultas más comunes. Utilice el plan `explain` con regularidad para analizar sus consultas y determinar si están utilizando índices y cómo lo hacen.
- Al crear índices compuestos, siga las pautas ESR descritas anteriormente en este capítulo para determinar mejor el orden de los campos del índice.
- Cuando sea posible, permita que su aplicación ejecute consultas cubiertas.
- Los índices comodín no reemplazan la planificación de índices basada en la carga de trabajo. Para cargas de trabajo con muchos patrones de consultas ad hoc o que manejan estructuras de documentos altamente polimórficas, los índices comodín brindan flexibilidad adicional. Sin embargo, conllevan una mayor sobrecarga de rendimiento y almacenamiento. Si los patrones de consulta de su aplicación se conocen de antemano, entonces debe utilizar índices más selectivos en los campos específicos a los que acceden las consultas.

---

### Sección 4: Resumen

En este capítulo, aprendió cómo optimizar su modelo de datos y utilizar índices para garantizar aplicaciones eficientes y de alto rendimiento. El diseño eficaz de esquemas y la optimización de índices son fundamentales para el éxito de un despliegue de MongoDB. Uno de los beneficios más importantes del modelo de datos flexible de MongoDB es cómo le permite iterar en el diseño de su esquema. A medida que su aplicación crece con el tiempo y los requisitos comerciales cambian, tiene opciones para rediseñar su esquema sin problemas y sin tiempo de inactividad para garantizar que su modelo de esquema coincida mejor con sus patrones de acceso a los datos.

A través de un cuidadoso modelado de datos y optimización de índices, puede aprovechar las capacidades de MongoDB para crear aplicaciones escalables y de alto rendimiento. El capítulo proporcionó una guía detallada sobre cómo alinear los requisitos de la aplicación con la arquitectura de la base de datos para lograr la máxima eficiencia.

En el próximo capítulo, aprenderá sobre las consultas en MongoDB, incluidas las nuevas funciones de consulta disponibles en MongoDB 8.0. Ahora que ha aprendido a crear índices eficaces, puede asegurarse de que las consultas sobre las que aprenderá sean eficientes y de alto rendimiento.
