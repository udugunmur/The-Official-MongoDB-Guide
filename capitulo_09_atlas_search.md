# Parte 9: Atlas Search

## Capítulo 9: Atlas Search

Este capítulo profundiza en algunas de las características más destacadas de MongoDB Atlas: Atlas Search y Vector Search. Una de las grandes ventajas de estas características es su integración perfecta en Atlas, lo que le permite comenzar a utilizar sus datos de inmediato sin la necesidad de software adicional en su pila tecnológica.

Si bien tanto Atlas Search como Vector Search están diseñados para ayudarle a realizar búsquedas, operan de maneras distintas. Atlas Search es una función de búsqueda de texto completo. Le permite crear índices de búsqueda en sus colecciones, lo que permite realizar búsquedas léxicas eficientes y potentes en sus datos.

Por otro lado, Vector Search aprovecha los modelos de incrustación (*embedding models*) para realizar búsquedas semánticas. Esta capacidad pone el poder de la IA al alcance de su mano, permitiéndole realizar búsquedas más matizadas basadas en el significado y no solo en palabras clave. Cuando se combinan, estas funciones permiten búsquedas híbridas, potenciando los resultados de búsqueda tanto con precisión de palabras clave como con comprensión contextual.

En este capítulo, exploraremos estas funciones en detalle y le brindaremos abundantes oportunidades prácticas para interactuar con ellas.

Este capítulo cubrirá los siguientes temas:

- Primeros pasos con Atlas Search
- El proceso mongot
- Nodos de búsqueda (*Search Nodes*)
- Índices de Atlas Search
- Consultas de Atlas Search
- Atlas Vector Search
- Generación de incrustaciones vectoriales con Voyage AI
- Índices de Vector Search
- Realización de búsquedas vectoriales con `$vectorSearch`
- Dar vida a sus datos con búsqueda híbrida, RAG y agentes de IA

Al final de este capítulo, comprenderá cómo implementar búsquedas tanto basadas en palabras clave como semánticas en MongoDB Atlas. Estas capacidades de búsqueda pueden transformar la forma en que interactúa con sus datos, yendo más allá de consultas simples para encontrar información basada en el contexto y el significado.

Ya sea que esté creando un sistema de recomendación, mejorando las experiencias de búsqueda de los usuarios o creando aplicaciones impulsadas por IA, la combinación de Atlas Search y Vector Search le proporciona las herramientas que necesita.

Nos entusiasma guiarle a través de estas técnicas paso a paso. ¡Prepárese para aplicar estos conceptos en escenarios prácticos que le ayudarán a crear aplicaciones más inteligentes y receptivas con MongoDB Atlas!

---

### Sección 1: Requisitos técnicos

Para seguir este capítulo, necesitará tener lo siguiente:

- Cuenta de MongoDB Atlas
- Python 3.7+
- Shell de MongoDB (`mongosh`)

Puede encontrar los ejemplos de código utilizados en GitHub en: [https://github.com/PacktPublishing/The-Official-MongoDB-Guide](https://github.com/PacktPublishing/The-Official-MongoDB-Guide).

---

### Sección 2: Primeros pasos con Atlas Search

Comencemos con Atlas Search porque comprender sus capacidades sentará una base sólida cuando pasemos a Vector Search. Antes de profundizar, aclaremos un error común: inicialmente podría pensar que Vector Search es superior debido a su capacidad para comprender el significado. Sin embargo, este no es siempre el caso. La búsqueda de texto completo a menudo puede ser más que suficiente para ciertos casos de uso y requiere significativamente menos recursos para ejecutarse.

A medida que exploremos cada función, considere los diversos escenarios donde destacan y cómo podría aprovecharlas en sus propias aplicaciones.

#### Herramientas para comenzar con Atlas Search

Comenzar a utilizar Atlas Search es ahora más fácil que nunca gracias al lanzamiento de nuevas herramientas y recursos de incorporación.

Una de estas herramientas es **Atlas Search Playground**, un entorno aislado (*sandbox*) diseñado para ayudarle a explorar las capacidades de Atlas Search sin la necesidad de una configuración completa de Atlas. Esta herramienta ofrece una forma sencilla de experimentar con la creación de índices de búsqueda y consultas sobre sus datos, todo dentro de una interfaz sencilla y fácil de usar. Adecuada tanto para principiantes como para usuarios experimentados, brinda la oportunidad de comprender y trabajar con las características de Atlas Search sin esfuerzo.

> **Figura 9.1:** Consulta de Atlas Search a través de Code Sandbox en Atlas Search Playground

Las características clave incluyen las siguientes:

- **Acceso instantáneo:** No es necesario registrarse ni iniciar sesión. Puede ir directamente a la página del entorno de Playground y comenzar a experimentar con las funciones de búsqueda de inmediato.
- **Espacio de trabajo Playground:** En este espacio de trabajo, puede agregar y modificar datos fácilmente, crear y probar índices de búsqueda y ejecutar consultas en tiempo real. Es un entorno práctico para probar nuevas ideas y comprender cómo funcionan las búsquedas.
- **Plantillas preconfiguradas:** Utilice una variedad de plantillas de muestra para simular escenarios del mundo real y probar sus habilidades de búsqueda en varios casos de uso de una manera sencilla.
- **Instantáneas compartibles:** Puede generar URL únicas para cada sesión para compartir sus experimentos y hallazgos con colegas o colaboradores, facilitando una fácil comunicación y colaboración.

Una de las formas de utilizar Atlas Search Playground es copiar y pegar una pequeña muestra de documentos de su colección que no se están devolviendo y solucionar problemas de su consulta teniéndolo todo justo delante de usted. Es más fácil lidiar con un par de cientos de documentos que con miles o incluso millones.

Los Playgrounds facilitan el proceso al permitir a los usuarios existentes de Atlas Search alinear sus configuraciones con Atlas Search Playground copiando y pegando documentos, definiciones de índices de búsqueda y consultas en los paneles de edición respectivos.

Otra opción que tenemos es el recientemente lanzado **Search Demo Builder**, que es una herramienta dinámica diseñada para simplificar su experiencia con MongoDB Atlas Search. Ubicada dentro de Atlas Search Playground, proporciona una interfaz intuitiva para explorar y configurar características de búsqueda clave como campos que admiten búsqueda, autocompletado y facetas, sin la necesidad de una amplia experiencia técnica o escritura manual de consultas. Search Demo Builder le permite experimentar y visualizar estas funcionalidades directamente, mejorando su comprensión y capacidades.

> **Figura 9.2:** Consulta de Atlas Search utilizando Search Demo Builder en Atlas Search Playground

Las características clave incluyen las siguientes:

- **Campos con capacidad de búsqueda:** De forma predeterminada, la herramienta aprovecha campos dinámicos, pero también tiene la flexibilidad de especificar campos particulares para sus búsquedas.
- **Autocompletado (*Autocomplete*):** Configure el autocompletado en campos de cadena para habilitar una experiencia de búsqueda mientras escribe, completa con una definición de índice y una consulta de autocompletado.
- **Filtros y facetas:** Estos elementos interactivos se pueden configurar utilizando matrices de cadenas y números, proporcionando una sólida experiencia de filtrado.
- **Vista previa de la experiencia:** Observe cómo sus configuraciones afectan los resultados de búsqueda en tiempo real a través de la pantalla de vista previa interactiva.
- **Definiciones generadas automáticamente:** La herramienta genera definiciones de índices y consultas basadas en las características de búsqueda configuradas, agilizando el proceso de configuración.

Esto tiene muchos beneficios, tales como:

- **Configuración instantánea:** Comience a explorar Atlas Search de inmediato utilizando conjuntos de datos precargados o cargando su propia colección, sin necesidad de configuraciones complejas ni registros.
- **Exploración guiada:** Benefíciese de recorridos paso a paso e información sobre herramientas que hacen que la herramienta sea accesible para usuarios de todos los niveles de habilidad, garantizando una curva de aprendizaje suave.
- **Espacio de trabajo interactivo:** Experimente con funciones como autocompletado y facetas en un entorno visual y práctico que fomenta la creatividad y la comprensión inmediata de sus configuraciones.
- **Resultados compartibles:** Copie y utilice fácilmente definiciones de consultas e índices generados automáticamente fuera de la herramienta, lo que respalda un uso y una colaboración más amplios en los proyectos.

Para comenzar a utilizar cualquiera de estas herramientas, visite el siguiente enlace y comience a explorar Atlas Search por su cuenta: [https://search-playground.mongodb.com/tools/code-sandbox/snapshots/new](https://search-playground.mongodb.com/tools/code-sandbox/snapshots/new).

Entonces, ¿qué utilizaremos en esta sección? Utilizaremos el Shell de MongoDB (`mongosh`) y también necesitaremos un clúster M0+ que contenga los datos de muestra de `sample_mflix`. Con esto, debería poder seguir el tutorial sin problemas.

---

### Sección 3: El mongot

Antes de comenzar a utilizar Atlas Search, familiaricémonos con la tecnología subyacente para que podamos entender qué sucede cuando realizamos una búsqueda. Esto proporcionará información sobre el proceso de Atlas Search conocido como `mongot`, explicando cómo sirve como la columna vertebral para crear y administrar índices de búsqueda y consultas dentro de MongoDB Atlas.

A estas alturas, probablemente esté familiarizado con cómo funcionan los índices y las consultas en MongoDB. Sin embargo, Atlas Search introduce un enfoque único, utilizando su propia tecnología de indexación basada en **Apache Lucene**. Al igual que el índice al final de un libro, un índice de búsqueda es una correspondencia entre términos y los documentos que contienen esos términos. Los índices de búsqueda también contienen otros metadatos relevantes, como las posiciones de los términos en los documentos.

Aquí entra `mongot`, un proceso especializado dentro de Atlas Search que funciona como una capa de abstracción, integrando el motor Apache Lucene a la perfección con MongoDB. A través de esta integración, `mongot` le permite administrar índices de búsqueda de manera eficiente, atendiendo a las reglas especificadas en sus definiciones de índice.

> **Figura 9.3:** Arquitectura de Atlas Search mostrando mongot y Lucene

Es importante saber que `mongot` es independiente del proceso `mongod`, que realiza las funciones centrales de la base de datos. En una arquitectura acoplada, estos dos procesos comparten y compiten por los mismos recursos. Podemos mejorar esto con una nueva característica llamada **Search Nodes**.

---

### Sección 4: Nodos de búsqueda (*Search Nodes*)

Entonces, ¿qué son los Search Nodes? Los Search Nodes proporcionan una infraestructura dedicada optimizada específicamente para las cargas de trabajo de Atlas Search y Vector Search. En lugar de ejecutar el proceso `mongot` en los mismos nodos que los procesos de su base de datos (`mongod`), los Search Nodes le permiten implementar `mongot` en una infraestructura independiente y especialmente diseñada.

Piense en esto como darle a su motor de búsqueda su propio departamento en lugar de hacer que comparta espacio con las operaciones de su base de datos; cada uno tiene espacio para respirar, crecer y operar sin perturbar al otro.

#### ¿Por qué migrar a Search Nodes dedicados?

Cuando los procesos `mongot` y `mongod` comparten la misma infraestructura (como en la arquitectura acoplada que discutimos anteriormente), surgen varios desafíos:

- **Contención de recursos:** Ambos procesos compiten por los mismos recursos de CPU, memoria y disco.
- **Limitaciones de rendimiento:** Las cargas de trabajo de búsqueda pesadas pueden afectar el rendimiento de la base de datos, y viceversa.
- **Restricciones de escalado:** No puede escalar de forma independiente su infraestructura de búsqueda en función de las necesidades específicas de la carga de trabajo de búsqueda.
- **Riesgos de disponibilidad:** El agotamiento de recursos de un proceso podría afectar al otro.

Veamos cómo los Search Nodes abordan estos desafíos.

#### Beneficios de los Search Nodes dedicados

- **Aislamiento de la carga de trabajo:** Con Search Nodes, sus operaciones de búsqueda se ejecutan independientemente de las operaciones de su base de datos. Esta separación garantiza lo siguiente:
  - La indexación de búsqueda pesada no ralentizará las operaciones centrales de su base de datos.
  - Los picos de carga de la base de datos no afectarán el rendimiento de las consultas de búsqueda.
  - Cada carga de trabajo obtiene recursos dedicados optimizados para sus necesidades específicas.
  *(En muchos casos, hemos observado disminuciones del 40 al 60% en el tiempo de consulta para operaciones de búsqueda complejas después de migrar a Search Nodes dedicados).*
- **Escalado independiente:** Una de las ventajas más poderosas de los Search Nodes es la capacidad de escalar su infraestructura de búsqueda independientemente del nivel de su base de datos:
  - Agregue más Search Nodes para manejar mayores volúmenes de consultas.
  - Actualice a niveles de Search Nodes optimizados para memoria para índices más grandes.
  - Configure recursos de búsqueda específicamente para cargas de trabajo de búsqueda vectorial.
  - Reduzca los recursos de búsqueda durante períodos de bajo uso sin afectar el rendimiento de la base de datos.
- **Disponibilidad mejorada:** Los Search Nodes mejoran la confiabilidad general del sistema de varias maneras:
  - Un mínimo de dos Search Nodes garantiza redundancia para sus cargas de trabajo de búsqueda.
  - Las operaciones de mantenimiento en su clúster de base de datos no afectarán la disponibilidad de la búsqueda.
  - Las operaciones de escalado son menos disruptivas para la funcionalidad de búsqueda.
- **Rendimiento optimizado:** Los Search Nodes están diseñados específicamente para cargas de trabajo de búsqueda, ofreciendo:
  - Configuraciones de hardware optimizadas para operaciones de búsqueda.
  - Soporte para búsqueda concurrente de segmentos (paralelizando la ejecución de consultas a través de segmentos de índice).
  - Opciones optimizadas para memoria diseñadas específicamente para Vector Search.

#### ¿Cuándo debería utilizar Search Nodes?

Si bien la arquitectura acoplada (donde `mongot` se ejecuta en los mismos nodos que `mongod`) funciona bien para desarrollo y pruebas, los Search Nodes se recomiendan encarecidamente para entornos de producción, especialmente en los siguientes casos:

- Su carga de trabajo incluye más de 2 millones de documentos para indexar.
- Sus datos indexados superan los 10 GB.
- Realiza más de 10,000 consultas en un período de 7 días.
- El rendimiento de la búsqueda es fundamental para su aplicación.
- Está implementando búsqueda vectorial a escala.

Consideremos un ejemplo práctico. Imagine que está creando una aplicación de comercio electrónico con lo siguiente:

- 5 millones de documentos de productos
- Búsqueda de texto completo en descripciones y características de productos
- Búsqueda vectorial para similitud visual y funciones de recomendación

Con la arquitectura acoplada, es posible que deba aprovisionar instancias de base de datos mucho más grandes de lo necesario solo para acomodar la carga de trabajo de búsqueda. Con Search Nodes, puede hacer lo siguiente:

- Configurar el nivel de su base de datos (M30, por ejemplo) basándose puramente en los requisitos de su base de datos.
- Implementar Search Nodes S30 optimizados específicamente para sus cargas de trabajo de búsqueda y vectoriales.
- Escalar cada uno de forma independiente a medida que crece su aplicación.

#### Primeros pasos con Search Nodes

¿Listo para migrar a Search Nodes? Aquí le mostramos cómo comenzar:

1. Asegúrese de que su clúster se esté ejecutando en un nivel dedicado (M10 o superior).
2. Verifique que su despliegue esté en una región que admita Search Nodes.
3. Habilite Search Nodes a través de la interfaz de usuario o la API de Atlas.
4. Configure el nivel de búsqueda adecuado y el recuento de nodos para su carga de trabajo.

Cuando habilita Search Nodes, Atlas crea índices sin problemas en sus nuevos Search Nodes mientras continúa atendiendo consultas desde su configuración existente, lo que garantiza cero tiempo de inactividad durante la transición.

Ahora que tenemos una comprensión básica de cómo Atlas facilita la búsqueda, cambiemos nuestra atención a verla en acción mediante la creación de un índice de búsqueda. Una vez que hayamos creado un índice de búsqueda, exploraremos cómo utilizarlo para realizar una búsqueda.

---

### Sección 5: Índices de Atlas Search

Antes de comenzar, tomemos un momento para revisar uno de los documentos de películas en la base de datos `sample_mflix`. Esto le dará una idea de con qué estamos trabajando:

```json
{
  "_id": ObjectId('573a13b2f29313caabd3a6e5'),
  "title": "V for Vendetta",
  "plot": "In a future British tyranny ...",
  "directors": ["James McTeigue"],
  "cast": ["Natalie Portman", "Hugo Weaving"],
  "genres": ["Action", "Drama", "Thriller"],
  "imdb": { rating: 8.2, votes: 709179, id: 434409 },
  "rated": "R",
  "metacritic": 62,
  "released": ISODate("2006-03-17T00:00:00.000Z"),  
  "languages": ["English"],  
  "fullplot": "Tells the story of Evey Hammond and her ...",
  "year": 2005,
  "writers": [],
  "type": "movie",
  // ...  
}
```

Utilizaremos los campos `plot`, `directors` y `released` para la mayoría de nuestros ejemplos en esta sección.

Comencemos con un ejemplo simple y luego profundicemos en los componentes de un índice de búsqueda para entender qué está sucediendo.

Supongamos que queremos indexar el campo `plot`, lo que permite una fácil búsqueda de descripciones de tramas de películas. Así es como puede lograr esto utilizando el método `createSearchIndex` en el Shell de MongoDB:

```javascript
db.movies.createSearchIndex(
  "plotIndex",
  {
    "mappings": {
      "fields": {
        "plot": {
          "type": "string"
        }
      }
    }
  }
)
```

Esto es lo que hace cada parte del comando:

- **Nombrar el índice:** Comience nombrando su índice. En nuestro caso, lo llamaremos `plotIndex`.
- **Definir campos:** Puede elegir indexar campos específicos o el documento completo. En este ejemplo, nos centramos en el campo `plot` para búsquedas de texto.
- **Especificar tipo de datos:** Especifique el tipo de datos esperado. Para el campo `plot`, es una cadena (`string`). Exploraremos más los tipos de datos en un momento.

De manera similar a otros índices, los índices de búsqueda ocupan espacio en el disco y se actualizan continuamente cuando se insertan, actualizan o eliminan documentos. El proceso `mongot` utiliza flujos de cambios (*change streams*) de MongoDB para mantener actualizados los índices de búsqueda.

Ahora es el momento de repasar cada uno de estos componentes juntos.

La creación de un índice de búsqueda comienza con un proceso clave conocido como **tokenización**. Esto implica transformar sus datos en piezas que admitan búsquedas, conocidas como tokens o términos. Un analizador (*analyzer*) facilita este proceso al gestionar la tokenización, normalización y derivación (*stemming*), haciendo que los datos estén representados de forma coherente y sean más fáciles de buscar.

> **Figura 9.4:** Analizador dividiendo la entrada en tokens en Atlas Search

Si bien los analizadores son una pieza importante del rompecabezas de la indexación, están más allá del alcance de este capítulo. Vale la pena señalar que juegan un papel vital al dictar cómo se analizan e indexan los datos. Una mayor exploración de los analizadores puede mejorar su comprensión de las capacidades de Atlas Search y se puede explorar con mayor profundidad en la documentación de MongoDB.

#### Mapeo de campos: dinámico frente a estático

La indexación eficaz requiere un mapeo de campos reflexivo, que afecta tanto el tamaño como el rendimiento de sus índices de búsqueda. A continuación se explica cómo se pueden aprovechar los diferentes enfoques:

- **Mapeo dinámico (*Dynamic mapping*):** Es el más adecuado para datos dinámicos donde las estructuras de campos son impredecibles o cambian con frecuencia. Se indexan todos los campos de un tipo de datos admitido dentro de un documento. Por ejemplo, en una aplicación de catálogo de películas, el mapeo dinámico indexaría todos los campos (título, trama, género, etc.) sin detalles predefinidos. Esta flexibilidad, sin embargo, genera tamaños de índice más grandes y una posible sobrecarga de rendimiento. Podemos crear un índice de búsqueda con mapeo dinámico ejecutando el siguiente comando:

```javascript
db.movies.createSearchIndex(
  {
    "mappings": {
      "dynamic": true
    }
  }
)
```

Para indexar todos los campos en la colección `movies`, usamos el campo `mappings` en la definición del índice y establecemos la opción `dynamic` en `true`. Como no proporcionamos un nombre para este índice, se guardará como `default`. Ahora, cada campo en el documento de la colección `movies` estará indexado. Esto significa que podemos crear una consulta de búsqueda para cualquiera de los campos.

- **Mapeo estático (*Static mapping*):** Es ideal cuando la estructura de su documento es estática y está bien definida. Solo se indexan los campos especificados, como `plot` y `released` en la colección `movies`, lo que permite un uso más eficiente de los recursos. Esta indexación enfocada reduce la carga computacional y mantiene el índice liviano. Podemos ver cómo se ve esto a continuación:

```javascript
db.movies.createSearchIndex(
  "plotReleasedIndex",
  {
    "mappings": {
      "dynamic": false,
      "fields": {
        "plot": {
          "type": "string"
        },
        "released": {
          "type": "date"
        }
      }
    }
  }
)
```

Al utilizar el mapeo estático, debemos especificar el tipo de datos para cada campo, como en el ejemplo anterior, donde el campo `plot` se define como `string` y el campo `released` se define como `date`.

Tenga en cuenta que MongoDB admite datos polimórficos. Si se intenta insertar un documento con un tipo de datos de valor de campo que difiere del especificado en el índice de búsqueda, ese documento en particular no se incluirá en el índice.

En escenarios donde se necesita un enfoque mixto, a los campos se les pueden asignar selectivamente propiedades dinámicas o estáticas, dirigiéndose únicamente a áreas específicas del documento. Considere situaciones como indexar el campo `plot` estáticamente mientras permite que el campo `released`, que podría contener subdocumentos, permanezca dinámico para acomodar variaciones. Podemos ver cómo se ve esto a continuación:

```javascript
db.movies.createSearchIndex(
  "plotReleasedIndex",
  {
    "mappings": {
      "dynamic": false,
      "fields": {
        "plot": {
          "type": "string"
        },
        "released": {
          "type": "embeddedDocument",
          "dynamic": true
        }
      }
    }
  }
)
```

Ahora, tanto el campo `plot` como todos los subdocumentos de `released` están indexados.

#### Elección de los tipos de datos correctos

A estas alturas, probablemente ya habrá deducido que especificar el tipo de datos correcto es importante en un índice de Atlas Search, y estaría en lo cierto. Atlas Search admite una variedad de tipos de datos, cada uno de los cuales atiende a diferentes aspectos de sus datos. Aquí hay una pequeña muestra de los tipos de datos disponibles:

- **Tipos básicos:** `string`, `number`, `boolean` y `date`
- **Tipos complejos:** `array`, `objectId`, `document` y `embeddedDocument`

Estos tipos se asignan estrechamente a la estructura BSON de MongoDB, lo que garantiza una integración perfecta. Por ejemplo, el tipo de datos `number` se asigna a los tipos BSON `double`, entero de 32 bits y entero de 64 bits.

> **Figura 9.5:** Tipo de datos number asignado a tipos BSON en Atlas Search

Para necesidades de datos con matices, a los campos se les pueden asignar múltiples tipos de datos, lo que permite consultas complejas que mejoran la forma en que interactúa con su base de datos. Por ejemplo, supongamos que queremos asignar múltiples tipos de datos cuando indexamos el campo `directors` en la colección `movies`, de modo que pueda ser un nombre o un tipo `objectId`. Podemos hacer esto con el siguiente índice:

```javascript
db.movies.createSearchIndex(
  "directorsIndex",
  {
    "mappings": {
      "dynamic": false,
      "fields": {
        "directors": [
          { "type": "string" },
          { "type": "objectId" }
        ]
      }
    }
  }
)
```

Aquí, asignamos el campo `directors` a una matriz de objetos, donde cada objeto se define como un tipo `objectId` o `string` en un índice de búsqueda. Esto nos permitirá crear una consulta de búsqueda utilizando el nombre de un director o el valor `objectId`.

Esto fue solo una muestra de lo que podemos hacer con los índices de Atlas Search, pero al comprender y aprovechar estos componentes (tipos de índices, analizadores, mapeos de campos y tipos de datos), estará equipado para crear índices de búsqueda finamente ajustados que se adapten a las necesidades específicas de las aplicaciones.

Explore la documentación de MongoDB y experimente con diferentes configuraciones para ampliar su comprensión y habilidades con los índices de Atlas Search ([https://www.mongodb.com/docs/atlas/atlas-search/index-definitions/](https://www.mongodb.com/docs/atlas/atlas-search/index-definitions/)).

Ahora que el índice de búsqueda está configurado, el proceso `mongot` puede consultar los documentos que queremos buscar.

Entonces, ¿cómo ejecutamos una consulta de búsqueda? Aquí es donde entra en juego la agregación de MongoDB.

---

### Sección 6: Consultas de Atlas Search

Para ejecutar una consulta de Atlas Search, se utiliza la etapa `$search` o `$searchMeta` dentro de una canalización de agregación (*aggregation pipeline*). Es importante tener en cuenta que las etapas de búsqueda deben ser la **primera etapa** en la canalización de agregación para que una consulta se ejecute correctamente.

La etapa `$search` realiza una búsqueda de texto completo en un campo o campos específicos que están definidos en el índice de búsqueda, mientras que la etapa `$searchMeta` se utiliza para devolver un resumen de sus resultados.

Por ejemplo, si desea obtener el número de documentos de una consulta de búsqueda, sería más eficiente utilizar la etapa `$searchMeta` en lugar de agregar una etapa `$count` al final de su canalización de búsqueda.

Veamos ambas en acción para comprenderlas mejor, comenzando con la etapa `$search`. Ampliaremos nuestro ejemplo anterior construyendo una canalización de agregación que integre la etapa `$search` con el `plotIndex` que configuramos previamente:

```javascript
db.movies.aggregate([  
  {
    "$search": {
      "index": "plotIndex",
      "text": {
        "query": "space",
        "path": "plot"
      }
    }
  }
])
```

Desglosemos el código anterior:

- `db.movies.aggregate()`: Este es el método de MongoDB utilizado para ejecutar una canalización de agregación en la colección `movies`.
- `$search`: Esto inicia una operación de búsqueda de texto completo dentro de la canalización de agregación.
- `"index": "plotIndex"`: Esto especifica qué índice de búsqueda utilizar. En este caso, está utilizando `plotIndex`, que es un índice que creamos anteriormente y que se enfoca en el campo `plot` de los documentos.
- `"text"`: Esta palabra clave indica que la siguiente operación es una búsqueda de texto, lo que significa que buscará a través de datos basados en texto dentro de los campos especificados. No estamos limitados solo al operador de texto:
  - `"query": "space"`: Este es el término de búsqueda que está consultando dentro del campo `plot`. La búsqueda de texto buscará documentos que contengan la palabra "space".
  - `"path": "plot"`: Esto determina el campo dentro de cada documento en el que buscar. Aquí, está establecido en el campo `plot`, que contiene descripciones textuales o resúmenes de películas.

Después de ejecutar el comando, deberíamos recibir documentos similares a los siguientes:

```json
{
  _id: ObjectId("573a1398f29313caabcea2cc"),
  plot: 'The young attendees of a space camp find themselves in space for real when their shuttle is accidentally launched into orbit.',
  genres: [
    'Adventure',
    'Family',
    'Sci-Fi'
  ],
  runtime: 107,
  rated: 'PG',
  cast: [
    'Kate Capshaw',
    'Lea Thompson',
    'Kelly Preston',
    'Larry B. Scott'
  ],
  title: 'SpaceCamp',
  released: 1986-06-06T00:00:00.000Z,
  directors: [
    'Harry Winer'
  ]
  //...
}
//...
```

Podemos confirmar que los documentos devueltos incluyen efectivamente la palabra "space", alineándose con nuestro objetivo de búsqueda. Sin embargo, podemos mejorar la legibilidad de los resultados agregando las etapas `$limit` y `$project` para mostrar solo los campos `title` y `plot`. Además, al incorporar el operador de agregación `$meta` en la etapa de proyección, podemos obtener información sobre cómo se calificó cada resultado. Así es como puede lograr esto:

```javascript
db.movies.aggregate([  
  {
    "$search": {
      "index": "plotIndex",
      "text": {
        "query": "space",
        "path": "plot"
      }
    }
  },
  {
    "$limit": 3
  },
  {
    "$project": {
      "_id": 0,
      "title": 1,
      "plot": 1,
      "score": {
        "$meta": "searchScore"
      }
    }
  }
])
```

Después de ejecutar el comando, debería ver algo similar a lo siguiente:

```json
{
  plot: 'The young attendees of a space camp find themselves in space for real when their shuttle is accidentally launched into orbit.',
  title: 'SpaceCamp',
  score: 3.5627431869506836
}
```

Ahora, podemos ver fácilmente los campos `title` y `plot` en nuestros resultados. También vemos cómo se puntuó cada documento. Atlas Search tiene un sistema de puntuación basado en la relevancia. El documento con la puntuación más alta es el más relevante y se devuelve primero. Atlas Search otorga a los documentos una puntuación más alta si el término de consulta aparece con frecuencia en un documento, y una puntuación más baja si el término de consulta aparece en muchos documentos de la colección.

Anteriormente, mencionamos que también podemos utilizar la etapa `$searchMeta`. Actualicemos nuestra consulta de búsqueda anterior para usar la etapa `$searchMeta` y agregarle el campo `count`:

```javascript
db.movies.aggregate([  
  {
    "$searchMeta": {
      "index": "plotIndex",
      "text": {
        "query": "space",
        "path": "plot"
      },
      "count": {
        "type": "total"
      }
    }
  }
])
```

Deberíamos ver que cuenta el número total de documentos devueltos por la consulta. En este caso, se devuelven 97 documentos:

```json
{
  count: {
    total: 97
  }
}
```

---

### Sección 7: Operadores y recolectores de Atlas Search

Los **operadores** especifican el tipo de operación de búsqueda que desea realizar en la etapa de canalización de agregación `$search`, ya sea combinando múltiples condiciones de búsqueda con `compound` o implementando búsquedas de texto específicas. Mientras tanto, los **recolectores (*collectors*)** ayudan a organizar los resultados de búsqueda devueltos agregándolos según ciertos criterios, como agrupar por valores o rangos. Al aplicar cuidadosamente tanto operadores como recolectores, puede crear consultas de búsqueda precisas y reveladoras que, en última instancia, mejoren la recuperación y el análisis de datos.

Hasta ahora, solo hemos utilizado el operador `text` en nuestras consultas de búsqueda, pero tenemos bastantes otros operadores a nuestra disposición:

- **Operador de texto (`text`):** Permite capacidades de búsqueda de texto completo. Le permite buscar términos dentro de campos de cadena. Por ejemplo, se puede utilizar para buscar documentos que contengan palabras o frases específicas. Esto es útil cuando necesita coincidencias precisas o amplias.
- **Operador compuesto (`compound`):** Al combinar múltiples criterios de búsqueda en una sola consulta, el operador compuesto mejora la complejidad y la flexibilidad de las consultas. Facilita operaciones lógicas como `must`, `mustNot`, `should` y `filter` para crear consultas compuestas. Por ejemplo, puede buscar documentos que deban tener ciertos campos y no deban incluir otros.
- **Operador de rango (`range`):** Este operador ayuda a especificar rangos numéricos o de fechas para el filtrado. Es ideal cuando desea encontrar documentos dentro de un umbral numérico o rango de fechas específico.
- **Operador de documento incrustado (`embeddedDocument`):** Utilícelo cuando necesite realizar búsquedas dentro de estructuras anidadas. Permite consultar campos específicos dentro de documentos incrustados, lo cual es excelente para tratar con modelos de datos complejos.
- **Operador cercano (`near`):** El operador `near` es para consultas geoespaciales. Le permite buscar documentos que contengan coordenadas geográficas dentro de una proximidad específica. Ya sea para encontrar las ubicaciones más cercanas o clasificar los resultados por distancia, el operador `near` integra consideraciones geográficas sin problemas en su búsqueda.

#### Operador compuesto (*compound*)

Uno de los operadores más poderosos a nuestra disposición es el operador `compound`. El operador compuesto en Atlas Search está diseñado para combinar varios otros operadores mediante reglas llamadas cláusulas (*clauses*). Piense en estas cláusulas como condiciones o matrices de operadores de búsqueda que juntos pueden ayudar a formular consultas sofisticadas. A continuación se muestra un ejemplo de cómo se forma una consulta de operador compuesto:

```json
{
  "$search": {
    "index": "<index name>", // "default" when omitted
    "compound": {
      "<must|mustNot|should|filter>": [ { "<operator>" } ],
      "score": "<options>"
    }
  }
}
```

A continuación se muestra un desglose de cómo funciona cada cláusula, incluida la cláusula `should`:

- **La cláusula `must`:**
  - *Propósito:* Requiere que se cumplan todas las condiciones para la inclusión del documento en los resultados.
  - *Ejemplo de caso de uso:* Encontrar películas que contengan tanto `poet` como `Elizabeth` en la trama, garantizando coincidencias exactas, ordenadas por puntuación.
- **La cláusula `filter`:**
  - *Propósito:* Garantiza que se cumplan todas las condiciones para la inclusión, sin influir en la puntuación del documento.
  - *Beneficio de eficiencia:* Elimina eficazmente documentos sin una puntuación computacionalmente costosa.
  - *Ejemplo de caso de uso:* Filtrar documentos con `poet`, devolviendo solo aquellos que cumplen la condición con una puntuación de cero.
- **La cláusula `mustNot`:**
  - *Propósito:* Excluye documentos que cumplen con las condiciones especificadas.
  - *Impacto en la puntuación:* Asigna una puntuación de cero a los documentos coincidentes.
  - *Ejemplo de caso de uso:* Excluir películas con `poet` o `Elizabeth`.
- **La cláusula `should`:**
  - *Propósito:* Requiere que se cumpla cualquiera de las condiciones, lo que permite coincidencias parciales.
  - *Impacto en la puntuación:* La puntuación se ve afectada por cada condición coincidente; más coincidencias producen una puntuación más alta.
  - *Ejemplo de caso de uso:* Buscar documentos con `poet` o `Elizabeth`, devolviendo aquellos donde esté presente cualquiera de los términos, con puntuaciones más altas para más coincidencias.
  - *La opción `minimumShouldMatch`:* Ajusta con precisión la cantidad de condiciones coincidentes requeridas. Por ejemplo, con un valor de `minimumShouldMatch` de 2, solo se devuelven los documentos que contienen tanto `poet` como `Elizabeth`, similar a usar la cláusula `must`.

Cada cláusula desempeña un papel único en la configuración de sus consultas de búsqueda, lo que le brinda control sobre la inclusión de documentos, la puntuación y el rendimiento. Comprender estos elementos le permite adaptar su estrategia a las necesidades específicas de recuperación de datos. Veamos un ejemplo del operador compuesto en acción:

```javascript
db.movies.aggregate([  
  {
    "$search": {
      "index": "plotIndex",
      "compound": {
        "must": [{
          "text": {
            "query": "space",
            "path": "plot"
          }
        }],
        "mustNot": [{
          "text": {
            "query": "aliens",
            "path": "plot"
          }
        }]
      }
    }
  }
])
```

Desglosemos el código anterior:

- Dentro de la etapa `$search`, se utiliza un operador `compound` para combinar múltiples criterios de búsqueda, lo que permite la creación flexible de consultas con operadores lógicos como `must` y `mustNot`.
- La cláusula `must` especifica condiciones que deben cumplirse: incluye una búsqueda de texto donde la consulta es `"space"` sobre el campo `plot` de los documentos.
- La cláusula `mustNot` define condiciones que no deben cumplirse: incluye una búsqueda de texto donde la consulta es `"aliens"` aplicada al campo `plot`.

Este es solo un ejemplo simple de lo que puede lograr el operador `compound`. Recuerde actualizar su índice de búsqueda si decide realizar una consulta de búsqueda en un campo diferente del campo `plot`.

#### Facetas de búsqueda (*Search facets*)

Pasemos a las facetas, pero ¿qué son exactamente las facetas? Las facetas le permiten agrupar los resultados de búsqueda por una categoría o rango y devolver el recuento para cada grupo especificado. Estos grupos se denominan comúnmente contenedores (*buckets*). ¿Alguna vez ha buscado un producto en un sitio web y en la barra lateral ve una lista de marcas con un número asociado al lado? Esa es una búsqueda por facetas en acción, lo que facilita a los usuarios limitar los resultados.

Podemos ver cómo funcionan las facetas creando una para la colección `movies`. Creemos una consulta de búsqueda para averiguar cuántas películas dentro de un rango de fechas de lanzamiento pertenecen a cada género. Para hacer esto, necesitamos usar el operador `range` con facetas.

> **Figura 9.6:** Cómo operan las facetas en Atlas Search

Antes de que podamos crear nuestra consulta de búsqueda, debemos definir un nuevo índice de búsqueda que se pueda usar para facetas. Para que las facetas funcionen en un campo en particular, este debe tener uno de los siguientes mapeos de campos:

- `token`
- `date`
- `number`

Ejecutemos el siguiente comando para crear un nuevo índice de búsqueda llamado `genresFacetedIndex`:

```javascript
db.movies.createSearchIndex(
  "genresFacetedIndex",
  {
    "mappings": {
      "dynamic": false,
      "fields": {
        "genres": {
          "type": "token"
        },
        "released": {
          "type": "date"
        }
      }
    }
  }
)
```

Veamos el código anterior en detalle:

1. El índice se llama `genresFacetedIndex`.
2. Los índices facetados requieren tipos de campos específicos, por lo que establecemos la opción `dynamic` en `false`. Esto significa que los campos no se indexan automáticamente; solo se indexan los campos especificados.
3. Para el campo `genres`, el tipo se especifica como `token`. El campo de género es una cadena, pero debemos utilizar el tipo `token` de Atlas Search para habilitar la creación de facetas en estos campos. Con el tipo `token`, Atlas Search indexa los términos de la cadena como un token único para operaciones eficientes de filtrado u ordenación. Esta configuración permite la creación de facetas basada en valores de cadena, categorizando los resultados de búsqueda en diferentes contenedores de género.
4. Para el campo `released`, el tipo se especifica como `date`. Esta configuración permite almacenar y consultar datos de tipo fecha, lo cual es útil para filtrar películas por su rango de fechas de estreno.

Una vez creado con éxito nuestro nuevo índice de búsqueda, ahora podemos crear nuestra consulta de búsqueda que encuentra películas según su fecha de estreno y las agrupa por géneros. Para hacer esto, ejecute el siguiente comando:

```javascript
db.movies.aggregate([  
    {  
      "$searchMeta": {  
        "index": "genresFacetedIndex",  
        "facet": {  
          "operator": {  
            "range": {  
                "path": "released",  
                "gte": ISODate("2000-01-01T00:00:00.000Z"),  
                "lte": ISODate("2000-01-31T00:00:00.000Z")  
              },  
            },  
          "facets": {  
            "genresFacet": {  
              "type": "string",  
              "path": "genres"  
            }  
          }  
        }  
      }  
    }  
  ])
```

A continuación se muestra un desglose del código anterior:

- Utiliza la etapa `$searchMeta` para recuperar el recuento de cada género.
- Define el índice de búsqueda como `genresFacetedIndex`.
- Especifica un operador de faceta; puede elegir cualquier operador disponible en Atlas Search.
- Utiliza el operador `range` para buscar películas con fecha de estreno entre el 1 de enero de 2000 y el 31 de enero de 2000.
- Define los contenedores (*buckets*) para cada género para ver los recuentos de películas de cada género dentro del rango de fechas especificado.
- Dentro del operador de faceta, crea un campo de subdocumento llamado `facets`, donde puede especificar múltiples facetas o agrupaciones, cada una con su propio tipo de datos, ruta y campos opcionales.
- Define el nombre de la faceta, que es cómo se hará referencia a los recuentos de facetas en los resultados. Nómbrelo `genresFacet`.
- Proporciona parámetros, incluidos el campo que se consulta y su tipo de datos. Consulta el campo `genres`, que es de tipo `token`.

Cuando ejecutamos nuestra consulta de búsqueda, obtenemos un documento con el nombre de nuestra faceta y cada género en su propio contenedor:

```json
{
  count: {
    lowerBound: 57
  },
  facet: {  
    genresFacet: {
      buckets: [
        {
          _id: 'Drama',
          count: 39
        },
        {
          _id: 'Comedy',
          count: 13
        },
        {
          _id: 'Romance',
          count: 13
        },
        {
          _id: 'Biography',
          count: 11
        }
      ]
    }
  }
}
```

Recuerde que el recuento incluye únicamente películas del 1 de enero de 2000 al 31 de enero de 2000, ya que definimos el operador `range`.

Esto es solo arañar la superficie de lo que Atlas Search puede lograr. Más allá de estos ejemplos, se pueden implementar funciones como búsqueda difusa (*fuzzy search*), autocompletado, mapeo de sinónimos, paginación y más.

---

### Sección 8: Atlas Vector Search

¿Alguna vez ha intentado buscar algo cuando solo tenía una descripción vaga en lugar de palabras clave exactas? Si ha estado trabajando con la búsqueda de texto completo de Atlas, ya está familiarizado con su poderosa capacidad para comparar y clasificar documentos según palabras y frases. Pero ¿qué pasa con la búsqueda del significado detrás de esas palabras?

#### De las palabras clave al significado

Mientras que la búsqueda de texto completo de Atlas destaca en la coincidencia de palabras clave y frases (búsqueda léxica), **Atlas Vector Search** lleva sus capacidades de búsqueda al siguiente nivel al comprender el significado de su consulta (búsqueda semántica).

Piense en la diferencia de esta manera:

- **Búsqueda de texto completo:** Encuentra documentos que contienen palabras clave específicas, como `base de datos`, `escalable` y `nube`.
- **Búsqueda vectorial:** Comprende el significado y el contexto de una consulta de búsqueda, como "Necesito una solución de base de datos confiable que crezca con mi negocio", y puede encontrar documentos relevantes incluso sin coincidir con palabras clave exactas.

Esto abre la posibilidad de hacer cosas realmente interesantes con nuestros datos, como las siguientes:

- Encontrar documentos conceptualmente relacionados incluso sin coincidencias de palabras clave
- Mejorar los sistemas de recomendación centrándose en el significado en lugar de en coincidencias exactas
- Habilitar consultas en lenguaje natural que comprendan la intención del usuario
- Potenciar la búsqueda multimodal en texto, imágenes y otros tipos de datos

Esta no es una lista exhaustiva, pero le da una idea general de lo que puede hacer con Vector Search.

#### ¿Qué son los vectores y por qué son importantes?

Las incrustaciones vectoriales (*vector embeddings*) son representaciones numéricas de sus datos (ya sean texto, imágenes o audio) almacenadas como matrices de valores de punto flotante. Cada número de esta matriz representa una dimensión en un espacio de alta dimensión, capturando colectivamente el significado semántico de sus datos. Veamos un ejemplo:

```javascript
// Example of a dense vector embedding (simplified)  
const vectorExample = [0.023, -0.112, 0.438, 0.067, -0.291, ...]; // typically hundreds or thousands of dimensions
```

Ya ha estado trabajando con vectores en la búsqueda de texto completo de Atlas, aunque es posible que no se haya dado cuenta. Esos eran vectores dispersos (*sparse vectors*), donde la mayoría de las dimensiones son ceros, indicando principalmente la presencia o ausencia de palabras específicas (similar a los cálculos TF-IDF).

#### Denso frente a disperso

La búsqueda de texto tradicional se basa en medidas estadísticas de frecuencia y unicidad de palabras. Esto funciona bien para encontrar documentos que contienen términos específicos, pero pasa por alto la comprensión contextual que los humanos aportan naturalmente al lenguaje.

Atlas Vector Search utiliza vectores densos (*dense vectors*) en su lugar, donde se aplica lo siguiente:

- Prácticamente ninguna dimensión es cero
- Cada dimensión contribuye al significado semántico
- El vector captura relaciones complejas entre conceptos
- Conceptos similares se agrupan en el espacio vectorial

#### Cómo funciona Atlas Vector Search

A un alto nivel, Atlas Vector Search sigue estos pasos:

1. **Generación de incrustaciones (*Embedding generation*):** Los modelos de IA convierten sus datos en incrustaciones vectoriales densas.
2. **Indexación vectorial:** Atlas almacena estos vectores en un formato eficiente y apto para búsquedas.
3. **Coincidencia de similitud:** Cuando realiza una búsqueda, su consulta se convierte en un vector y se compara con los vectores almacenados.
4. **Clasificación de resultados:** Los documentos se devuelven según la similitud vectorial, no según la coincidencia de palabras clave.

Exploremos cada uno de estos pasos creando una consulta de búsqueda vectorial que nos ayude a encontrar películas según la consulta de un usuario.

#### Configuración del entorno

Para que podamos comenzar, primero necesitaremos configurar nuestro entorno. Para esto, utilizaremos un entorno virtual de Python, que se puede crear ejecutando el siguiente comando en el directorio en el que desea trabajar:

```bash
python -m venv vs-demo
```

Una vez que termine, podemos activar el entorno virtual ejecutando el siguiente comando:

- **En macOS/Linux:**

```bash
source vs-demo/bin/activate
```

- **En Windows:**

```cmd
.\vs-demo\Scripts\activate
```

Cuando el entorno virtual esté activado, verá el nombre del entorno en el símbolo del sistema. Una vez activado, podemos instalar paquetes usando `pip` que solo estarán disponibles en este entorno virtual.

Sigamos adelante e instalemos `pymongo` ya que lo usaremos para interactuar con nuestra base de datos MongoDB:

```bash
pip install pymongo
```

Lo último que debemos hacer es crear tres archivos: `data.py`, `index.py` y `search.py`. Usaremos estos archivos a lo largo de esta sección. Comencemos generando algunas incrustaciones vectoriales para nuestros datos.

---

### Sección 9: Generación de incrustaciones vectoriales con Voyage AI

Ahora que entendemos cómo funciona Atlas Vector Search a alto nivel, profundicemos en el primer paso: la generación de incrustaciones (*embeddings*). Aquí es donde convertimos las tramas de nuestras películas en incrustaciones vectoriales que se pueden buscar y comparar.

Generar incrustaciones requiere una clave de API para el modelo elegido. Esto generalmente requiere una tarjeta de crédito y tiene un costo. Los modelos gratuitos y de bajo costo están disponibles en [huggingface.com](https://huggingface.com/).

#### Comprensión de los modelos de incrustación

Para que nuestra búsqueda de películas funcione, necesitamos convertir las descripciones de texto en nuestros campos `plot` en vectores numéricos. Sin embargo, no todos los modelos de IA son iguales para esta tarea.

Los modelos de lenguaje grande (LLM) como GPT o Claude destacan en la generación de texto y conversaciones, pero no están optimizados específicamente para crear las representaciones numéricas coherentes que necesitamos para la búsqueda vectorial.

Los modelos de incrustación, sin embargo, están especialmente diseñados para transformar contenido en representaciones vectoriales densas que capturan el significado semántico. Estos modelos especializados garantizan que conceptos similares se posicionen cerca unos de otros en el espacio vectorial, independientemente de las palabras específicas utilizadas.

Si bien los LLM pueden generar incrustaciones, los modelos de incrustación dedicados producen vectores de mayor calidad para aplicaciones de búsqueda, con mejor rendimiento y resultados más coherentes.

#### Aprovechamiento de Voyage AI para incrustaciones

Utilizaremos Voyage AI para generar las incrustaciones de nuestros datos, ¡que recientemente pasó a formar parte de la familia MongoDB! La adquisición de Voyage AI por parte de MongoDB aporta tecnología de incrustación de vanguardia directamente a nuestro ecosistema de bases de datos.

Voyage AI ofrece varias ventajas para nuestra aplicación de búsqueda de películas:

- **Precisión líder en la industria:** Los modelos de Voyage AI son los modelos de incrustación *zero-shot* mejor calificados en Hugging Face.
- **Optimizado para búsqueda:** Estos modelos están diseñados específicamente para tareas de recuperación.
- **Integración perfecta:** Pronto se integrará de forma nativa en MongoDB Atlas.

Tenga en cuenta que no está limitado a utilizar únicamente Voyage AI con Atlas Vector Search. Tiene la capacidad de utilizar cualquier modelo de incrustación que desee con Atlas Vector Search.

#### Implementación de la generación de incrustaciones para la colección movies

Creemos los vectores de incrustación para nuestra colección `movies`. Nos centraremos específicamente en generar incrustaciones para el campo `plot` de cada documento. Hay varias formas de generar incrustaciones. Muchas incrustaciones vectoriales tienen paquetes que podemos instalar, lo que facilita la generación de incrustaciones para nuestros datos. Este es el enfoque que adoptaremos, lo que significa que tenemos que instalar el paquete para Voyage AI.

Para hacer esto, ejecute el siguiente comando en su entorno virtual:

```bash
pip install voyageai
```

Una vez completado esto, podemos agregar el siguiente código al archivo `data.py`:

```python
import os
import requests
import key_param
from pymongo import MongoClient
import voyageai

vo = voyageai.Client(api_key=key_param.voyage_api_key)

mongodb_client = MongoClient(key_param.mdb_uri)

#  Database name
DB_NAME = "sample_mflix"
# Name of the collection with full documents- used for summarization
FULL_COLLECTION_NAME = "movies"
db = mongodb_client[DB_NAME]
full_collection = db[FULL_COLLECTION_NAME]

chunked_docs = full_collection.find({}, {"plot": 1})

for doc in chunked_docs:
    embedding = vo.embed(doc["plot"], model="voyage-3-lite", input_type="document").embeddings[0]
    print(doc["plot"])
    print(embedding)
    full_collection.update_one({"_id": doc["_id"]}, {"$set": {"plot_embedding": embedding}})
```

Veamos qué sucede en el código anterior:

- **Importaciones de bibliotecas:** El código importa bibliotecas, incluidas `pymongo` para la conectividad con MongoDB y `voyageai` para generar incrustaciones.
- **Inicialización del cliente API:**
  - Crea una instancia de cliente de Voyage AI utilizando una clave de API almacenada en `key_param.voyage_api_key`.
  - Inicializa una conexión de cliente de MongoDB con la cadena de conexión de `key_param.mdb_uri`.
- **Configuración de la base de datos:** Se dirige a la base de datos `sample_mflix` y, específicamente, a la colección `movies`. Configura referencias tanto a la base de datos como a la colección para un fácil acceso.
- **Procesamiento de documentos:** Consulta la colección para recuperar todos los documentos, pero selecciona solo el campo `plot` para cada documento.
- **Bucle a través de documentos:** Para cada documento de película recuperado, el código hace lo siguiente:
  - Utiliza el método `embed()` de Voyage AI para generar incrustaciones vectoriales a partir del texto de la trama de la película.
  - Utiliza específicamente el modelo `voyage-3-lite`, optimizado para la incrustación de documentos.
  - Extrae la primera (y única) incrustación de los resultados devueltos.
- **Persistencia de datos:**
  - Actualiza cada documento en MongoDB agregando un nuevo campo llamado `plot_embedding` que contiene la representación vectorial.
  - Utiliza el método `update_one()` de MongoDB con el valor `_id` del documento como identificador.

Ahora que hemos generado incrustaciones para las tramas de nuestras películas, hemos completado el primer paso en nuestra implementación de Atlas Vector Search. Nuestra base de datos ahora contiene las representaciones vectoriales necesarias para la búsqueda semántica. En la siguiente sección, exploraremos cómo indexar estos vectores para una búsqueda eficiente y crear nuestras primeras consultas de búsqueda vectorial.

---

### Sección 10: Índices de Vector Search

Ahora que ha generado exitosamente incrustaciones vectoriales para las tramas de sus películas, necesita una forma de comparar eficientemente estos vectores para encontrar contenido similar. Pero ¿cómo se busca eficientemente a través de vectores con cientos o miles de dimensiones? Aquí es donde entran en juego los índices de Atlas Vector Search.

#### Comprensión de la tecnología detrás de Vector Search

Cuando trabajamos con datos vectoriales de alta dimensión, los métodos de búsqueda tradicionales no son eficientes. MongoDB Atlas utiliza gráficos de **Mundo Pequeño Jerárquico Navegable (*Hierarchical Navigable Small World* o HNSW)** para potenciar Vector Search. Pero ¿qué es exactamente HNSW y por qué es tan eficaz?

#### La evolución hacia HNSW

HNSW evolucionó a partir de dos estructuras de datos importantes:

1. **Listas por saltos (*Skip lists*):** Son listas enlazadas ordenadas con múltiples capas, cada una de las cuales contiene puntos de lista. Esta estructura aumenta la eficiencia de la búsqueda.

> **Figura 9.7:** Capas de lista de saltos para navegar al valor 9

2. **Gráficos de mundo pequeño navegable (*Navigable Small World* o NSW):** Los puntos (vértices) en estos gráficos están conectados por enlaces (bordes) que representan similitud. Permiten búsquedas de similitud eficientes, pero pueden devolver resultados subóptimos.

> **Figura 9.8:** Gráfico NSW que muestra el recorrido del vecino más cercano

HNSW combina lo mejor de ambos enfoques. Al igual que una lista por saltos, tiene múltiples capas y, al igual que NSW, utiliza gráficos de proximidad. La capa inferior contiene todos los puntos con conexiones más cortas, mientras que las capas superiores tienen menos puntos con conexiones más largas, similar a acercar y alejar el zoom en un mapa.

> **Figura 9.9:** Recorrido HNSW desde la capa superior hasta el punto objetivo

#### Creación de un índice de Vector Search

Hay un par de formas de crear un índice de búsqueda vectorial, pero en nuestro caso, utilizaremos el controlador de Python para crear el índice.

Dentro del archivo `index.py`, ejecute el siguiente código para crear un índice de búsqueda vectorial:

```python
import key_param
import pymongo
from pymongo.mongo_client import MongoClient
from pymongo.operations import SearchIndexModel

mongodb_client = MongoClient(key_param.mdb_uri)

#  Database name
DB_NAME = "sample_mflix"
# Name of the collection with full documents- used for summarization
FULL_COLLECTION_NAME = "movies"

db = mongodb_client[DB_NAME]
full_collection = db[FULL_COLLECTION_NAME]
search_index_model = SearchIndexModel(
  definition={
    "fields": [
      {
        "type": "vector",
        "numDimensions": 512,
        "path": "plot_embedding",
        "similarity": "cosine"
      },
      {
        "type": "filter",
        "path": "year"
      }
    ]
  },
  name="vector_plot_index",
  type="vectorSearch"
)

full_collection.create_search_index(search_index_model)
print("Index created successfully")
```

Desglosemos el código para comprender mejor lo que está sucediendo:

- `SearchIndexModel` crea un modelo para el índice de búsqueda e incluye:
  - Un campo `vector` que:
    - Apunta a `plot_embedding` en sus documentos.
    - Utiliza 512 dimensiones (que coinciden con el tamaño de salida de su modelo de incrustación).
    - Implementa la similitud de coseno para medir la cercanía del vector.
  - Un campo `filter` en `year` que:
    - Le permite limitar los resultados de búsqueda por año de estreno de la película.
    - Combina el filtrado tradicional con la similitud vectorial.
- `full_collection.create_search_index(search_index_model)` aplica la definición de índice a su colección. Una vez creado, MongoDB Atlas construye y optimiza el índice.
- Una declaración `print` confirma que el índice se creó correctamente.

#### Comprensión de las funciones de similitud

Al configurar su índice de búsqueda vectorial, deberá elegir una función de similitud, que afecta en gran medida sus resultados de búsqueda:

- **Similitud euclidiana (*Euclidean similarity*):** Mide la distancia en línea recta entre vectores en un espacio multidimensional. Esto funciona bien cuando las magnitudes absolutas de sus vectores importan.
- **Similitud del coseno (*Cosine similarity*):** Mide el ángulo entre vectores, ignorando la magnitud. Esto es útil cuando le importa la similitud direccional pero no el tamaño. Tenga en cuenta que el coseno no funciona con vectores de magnitud cero.
- **Producto punto (*Dot product*):** Similar al coseno, mide el ángulo pero también considera la magnitud. Para obtener resultados óptimos con el producto escalar, sus vectores deben normalizarse a una longitud unitaria.

¿Cuál debería elegir? Primero, consulte la documentación de su modelo de incrustación; utilice la función con la que se entrenó el modelo. Si esa información no está disponible, experimente con cada función para ver cuál ofrece los mejores resultados para su caso de uso específico.

#### Filtrado previo de sus búsquedas vectoriales

Una de las características más potentes de Atlas Vector Search es la capacidad de **prefiltrar (*pre-filter*)** sus datos antes de ejecutar comparaciones de similitud. Esto tiene los siguientes beneficios:

- Hace que sus búsquedas sean más eficientes al reducir el espacio de búsqueda.
- Mejora la relevancia de los resultados al excluir datos irrelevantes.
- Mejora el rendimiento de las aplicaciones multiinquilino al restringir las búsquedas a subconjuntos de datos específicos.

En el ejemplo anterior, agregamos un filtro en el campo `year`, lo que nos permitirá enfocar las búsquedas en películas de años específicos.

#### Consideraciones importantes y limitaciones

A continuación se detallan algunos puntos clave que debe recordar al trabajar con índices de Vector Search:

- **Requisitos de memoria:** HNSW requiere mucha memoria. Para cargas de trabajo de producción, utilice Search Nodes dedicados para separar Vector Search de las operaciones de su clúster principal.
- **Limitaciones de campo:** Solo puede usar el tipo de campo `vector` en campos que contengan incrustaciones vectoriales.
- **Limitaciones de matriz:** No puede indexar campos en subdocumentos dentro de un campo de matriz. No almacene incrustaciones en documentos que formen parte de una matriz.
- **Dimensiones del vector:** El número de dimensiones debe ser menor o igual a 8,192.
- **Tipos de vectores:** Su campo vectorial debe contener una matriz de números (`double` de BSON) o vectores `BinData` de BSON con subtipos específicos.
- **Estado del índice:** Supervise el estado de su índice a través de la interfaz de usuario de Atlas. Los índices solo son completamente operativos cuando muestran el estado "Active".

Otra cosa a considerar es cuánto espacio ocupa cada vector en la base de datos. Recientemente, MongoDB lanzó la **cuantificación vectorial (*vector quantization*)** para abordar este problema particular. La cuantificación vectorial es una técnica que comprime vectores de alta dimensión en representaciones más compactas al reducir la cantidad de bits utilizados para almacenar cada dimensión. Funciona mapeando valores flotantes continuos a un conjunto más pequeño de valores discretos, cambiando efectivamente una pequeña cantidad de precisión por ahorros significativos de almacenamiento. Piense en ello como algo similar a cómo funciona la compresión JPEG para imágenes: sacrifica un poco de calidad para reducir drásticamente el tamaño del archivo.

Al trabajar con MongoDB Atlas, tiene dos opciones de cuantificación a su disposición: **cuantificación escalar** y **cuantificación binaria**. La cuantificación escalar divide el rango de cada dimensión en contenedores iguales, convirtiendo valores flotantes en enteros discretos y reduciendo el uso de memoria a aproximadamente 1/4 del original. La cuantificación binaria es aún más agresiva, convirtiendo cada dimensión en un simple 0 o 1 según si está por encima o por debajo de un punto medio (normalmente 0), reduciendo los requisitos de memoria a aproximadamente 1/24 del tamaño original. Esto es particularmente efectivo para incrustaciones normalizadas como `text-embedding-3-large` de OpenAI.

Los beneficios de la cuantificación vectorial son sustanciales, especialmente a medida que crecen sus colecciones de vectores. Para aplicaciones con más de 10 millones de vectores, la cuantificación puede reducir drásticamente el consumo de recursos manteniendo la calidad de la búsqueda a través de técnicas como el recálculo de puntuación de candidatos (*candidate rescoring*). Experimentará consultas más rápidas, menor uso de memoria y costos reducidos, todo mientras escala sus capacidades de búsqueda vectorial para manejar conjuntos de datos más grandes.

---

### Sección 11: Realización de búsquedas vectoriales con $vectorSearch

Ahora que ha creado sus incrustaciones vectoriales y las ha indexado correctamente, ¡es hora de desatar el verdadero poder de Atlas Vector Search! ¿Está listo para ver cómo podemos consultar datos basados en el significado semántico en lugar de en coincidencias de texto exactas?

En esta sección, veremos cómo realizar búsquedas vectoriales utilizando la etapa de agregación `$vectorSearch`. Esta poderosa característica le permite encontrar documentos basados en la similitud semántica entre su consulta y las incrustaciones vectoriales en su base de datos.

Veamos un ejemplo completo de cómo realizar una búsqueda vectorial:

```python
import key_param
from pymongo import MongoClient
import voyageai

vo = voyageai.Client(api_key=key_param.voyage_api_key)

mongodb_client = MongoClient(key_param.mdb_uri)

#  Database name
DB_NAME = "sample_mflix"
# Name of the collection with full documents- used for summarization
FULL_COLLECTION_NAME = "movies"

db = mongodb_client[DB_NAME]
full_collection = db[FULL_COLLECTION_NAME]

query = "A movie about a life altering event that changes the protagonist's life forever."
query_embedding = vo.embed(query, model="voyage-3-lite", input_type="query").embeddings[0]

pipeline = [
    {
        '$vectorSearch': {
            'index': 'vector_plot_index',
            'path': 'plot_embedding',
            'queryVector': query_embedding,
            'numCandidates': 100,  
            'limit': 10,  
            'filter': {  
                'year': {  
                    '$lt': 1960  
                }  
            },  
        }  
    },  
    {  
      '$project': {  
          'title': 1,  
          'plot': 1,  
          'year': 1,  
          'score': {  
              '$meta': 'vectorSearchScore'  
          }  
      }  
    }  
]

results = full_collection.aggregate(pipeline)
for result in results:
    print(f"Title: {result['title']}, Year: {result['year']}, Score: {result['score']}")
    print(f"Plot: {result['plot']}")
```

Desglosemos lo que sucede en este código:

- **Importación de bibliotecas:** Comenzamos importando las bibliotecas necesarias, incluidas PyMongo para las interacciones con MongoDB y Voyage AI para generar incrustaciones.
- **Configuración de conexiones:** Creamos clientes tanto para MongoDB como para Voyage AI, utilizando parámetros de conexión almacenados en un archivo separado por seguridad.
- **Definición de la consulta:** Creamos una consulta en lenguaje natural que describe el tipo de película que estamos buscando, que trata sobre un evento que altera la vida.
- **Generación de incrustaciones de consulta:** Convertimos nuestra consulta de texto en una incrustación vectorial utilizando el mismo modelo que usamos para indexar nuestros datos (`voyage-3-lite`).
- **Creación de la canalización:** Definimos una canalización de agregación con dos etapas:
  - La etapa `$vectorSearch`, que realiza la búsqueda semántica.
  - Una etapa `$project` que da forma a nuestra salida.
- **Parámetros de Vector Search:**
  - `index`: El nombre de nuestro índice de búsqueda vectorial.
  - `path`: El campo que contiene nuestras incrustaciones vectoriales.
  - `queryVector`: Nuestra consulta convertida en una incrustación.
  - `numCandidates`: Cuántos candidatos considerar (100 en este caso).
  - `limit`: Cuántos resultados devolver (10).
  - `filter`: Un filtro previo para limitar los resultados a películas anteriores a 1960.
- **Recuperación de la puntuación:** Usamos el operador especial `$meta: 'vectorSearchScore'` para incluir la puntuación de similitud en nuestros resultados.
- **Ejecución y visualización:** Ejecutamos la canalización de agregación e imprimimos los resultados, mostrando el título, el año, la puntuación de similitud y la trama de cada película.

#### Comprensión de los resultados

Cuando ejecutamos este código, obtenemos estos fascinantes resultados:

```text
Title: White Shadows, Year: 1924, Score: 0.7545456886291504
Plot: In Paris a wild girl becomes possessed by the soul of her twin who died to save her life.

Title: For Heaven's Sake, Year: 1926, Score: 0.7396184802055359
Plot: An irresponsible young millionaire changes his tune when he falls for the daughter of a downtown minister.

[...additional results omitted for brevity...]
```

¿Nota algo sobre estos resultados? Nuestra consulta era sobre un evento que altera la vida y cambia la vida del protagonista para siempre, y el sistema devolvió películas con tramas que coinciden semánticamente con este concepto. El primer resultado presenta a un personaje cuya vida cambia drásticamente por una posesión sobrenatural, mientras que el segundo muestra a un personaje cuya vida se transforma al enamorarse.

Ninguna de estas coincidencias se habría encontrado con una búsqueda tradicional por palabras clave, ya que no comparten las palabras exactas de nuestra consulta. ¡Esto demuestra el poder de la búsqueda semántica: encontrar resultados basados en el significado en lugar de simplemente hacer coincidir texto!

#### ANN frente a ENN: comprensión de los enfoques de búsqueda

Atlas Vector Search admite dos enfoques de búsqueda distintos:

#### Búsqueda de vecino más cercano aproximado (ANN)

La búsqueda ANN (*Approximate Nearest Neighbor*) utiliza el algoritmo HNSW para encontrar de manera eficiente incrustaciones vectoriales que sean las más cercanas a su vector de consulta sin escanear cada vector individual en su conjunto de datos. Esto es lo que debe saber sobre ANN:

- Optimizado para velocidad y eficiencia, especialmente con grandes conjuntos de datos.
- No escanea todos los vectores, lo que lo hace más rápido pero potencialmente menos preciso.
- Se ajusta mediante el parámetro `numCandidates` para equilibrar velocidad frente a precisión.
- Recomendado para la mayoría de los casos de uso de producción, especialmente con colecciones grandes.

#### Búsqueda de vecino más cercano exacto (ENN)

Añadida recientemente a Atlas (disponible en MongoDB v6.0.16, v7.0.10, v7.3.2 o posterior), la búsqueda ENN (*Exact Nearest Neighbor*) compara exhaustivamente su vector de consulta con cada vector indexado para encontrar las coincidencias absolutamente más cercanas. Esto es lo que debe saber sobre ENN:

- Proporciona una precisión del 100% calculando la distancia entre todas las incrustaciones.
- Es más intensiva computacionalmente y puede afectar la latencia de las consultas.
- Ideal para evaluar comparativamente la precisión de ANN o para conjuntos de datos más pequeños.
- Perfecta para consultas con filtros previos altamente selectivos (que afectan a <5% de los datos).

Para habilitar la búsqueda ENN en nuestro ejemplo, modificaríamos nuestra etapa `$vectorSearch` para incluir el parámetro `exact: true`:

```python
'$vectorSearch': {
    'index': 'vector_plot_index',
    'path': 'plot_embedding',
    'queryVector': query_embedding,
    'exact': True,  # Enable Exact Nearest Neighbor search
    'limit': 10,
    'filter': {  
        'year': {  
            '$lt': 1960  
        }  
    }  
}
```

Al utilizar la búsqueda ENN, no necesita especificar `numCandidates` ya que el algoritmo examina todos los vectores.

#### Optimización del rendimiento de Vector Search

Para obtener los mejores resultados de sus búsquedas vectoriales, considere estas mejores prácticas:

- **Ajuste `numCandidates` para búsquedas ANN:** Comience estableciendo `numCandidates` en al menos 10 a 20 veces el valor de su límite (`limit`).
- **Utilice filtros previos sabiamente:** El filtrado previo con la opción `filter` mejora el rendimiento al reducir el espacio de búsqueda.
- **Elija entre ANN y ENN según sus necesidades:**
  - Utilice ANN para la mayoría de los escenarios de producción.
  - Utilice ENN para conjuntos de datos pequeños o al validar la precisión de ANN.
- **Garantice la coherencia del modelo de incrustación:** Utilice el mismo modelo de incrustación para la indexación y las consultas.

Vector Search abre interesantes posibilidades para crear aplicaciones conscientes del significado semántico. Puede encontrar documentos similares, implementar sistemas de recomendación o crear interfaces de lenguaje natural para sus datos, todo impulsado por la comprensión semántica de su contenido.

---

### Sección 12: Dar vida a sus datos con búsqueda híbrida, RAG y agentes de IA

Ahora que ha explorado las profundidades de Atlas Search y Atlas Vector Search, profundicemos en las emocionantes posibilidades que desbloquean estas poderosas herramientas. ¿Qué sucede cuando combina las capacidades de búsqueda tradicionales con incrustaciones vectoriales? ¡Le espera un mundo de aplicaciones inteligentes!

#### Búsqueda híbrida: lo mejor de ambos mundos

Al aprovechar conjuntamente Atlas Search y Atlas Vector Search, puede hacer lo siguiente:

- Capturar tanto el significado semántico como las coincidencias exactas de palabras clave.
- Ofrecer resultados más relevantes considerando múltiples dimensiones de búsqueda.
- Equilibrar la precisión y la recuperación (*recall*) según su caso de uso específico.
- Ajustar experiencias de búsqueda que realmente comprendan la intención del usuario.

La búsqueda híbrida le permite crear experiencias de búsqueda que comprenden no solo lo que los usuarios buscan, sino lo que realmente quieren decir.

#### Generación aumentada por recuperación (RAG)

RAG (*Retrieval-Augmented Generation*) representa una de las aplicaciones más prácticas de la tecnología de búsqueda vectorial. Al conectar sus datos de MongoDB a los LLM, puede hacer lo siguiente:

- Basar las respuestas de la IA en sus propios datos fidedignos.
- Reducir las alucinaciones y aumentar la precisión.
- Crear asistentes de IA específicos de un dominio que realmente comprendan su negocio.
- Mantener el control sobre las fuentes de información mientras aprovecha las capacidades de la IA.

Con Atlas Vector Search como su motor de recuperación, crear aplicaciones RAG se vuelve sencillo y escalable, garantizando que sus soluciones de IA tengan acceso a la información más relevante de sus bases de datos.

#### Agentes de IA: tomando medidas con sus datos

Los agentes de IA representan la próxima evolución: sistemas que no solo recuperan y generan, sino que realmente toman medidas basadas en sus datos. Atlas Vector Search permite a los agentes hacer lo siguiente:

- Comprender solicitudes de usuario complejas mediante la comprensión semántica.
- Recuperar el contexto relevante antes de ejecutar tareas.
- Aprender de interacciones anteriores almacenadas en su base de datos.
- Orquestar múltiples herramientas y sistemas con inteligencia.

#### Ecosistema de integración: construya más rápido con herramientas líderes de IA

Una de las mayores fortalezas de Atlas Vector Search es su extenso ecosistema de integración. ¡No necesita construir todo desde cero! MongoDB se ha asociado con plataformas de IA líderes para hacer que la implementación sea fluida:

- **AWS Bedrock:** Conecte sus datos vectoriales directamente a los modelos fundacionales de Amazon.
- **LangChain y LangGraph:** Cree canalizaciones RAG sofisticadas con un código mínimo.
- **LlamaIndex:** Cree patrones estructurados de acceso a datos sobre sus colecciones de MongoDB.
- **Microsoft Semantic Kernel:** Intégrese con el marco de orquestación de IA de Microsoft.
- **Haystack:** Cree aplicaciones de búsqueda y RAG listas para producción.
- **Spring AI:** Integre capacidades de búsqueda vectorial en sus aplicaciones Java Spring.
- **Google Vertex AI:** Conéctese sin problemas con el ecosistema de IA/aprendizaje automático (ML) de Google.

Con Atlas Search y Vector Search a su alcance, las posibilidades son casi infinitas. Desde la recuperación inteligente de documentos hasta asistentes de compras conversacionales y sistemas de gestión del conocimiento que realmente comprenden el contexto, sus datos ahora pueden impulsar experiencias que antes eran imposibles.

---

### Sección 13: Resumen

En este capítulo, exploramos dos características destacadas de MongoDB Atlas: Atlas Search y Atlas Vector Search. Ambas funciones están perfectamente integradas en Atlas, lo que permite a los usuarios aprovechar sus datos sin necesidad de software adicional. Estas herramientas permiten capacidades de búsqueda avanzadas que van más allá de las consultas básicas, ayudando a los desarrolladores a descubrir conocimientos más profundos y crear aplicaciones más inteligentes.

Atlas Search proporciona una funcionalidad de búsqueda de texto completo que permite una búsqueda eficiente basada en palabras clave en todas las colecciones. Al implementar índices de búsqueda y diseñar consultas, permite funciones como coincidencia difusa, autocompletado y mapeo de sinónimos, lo que garantiza experiencias de búsqueda altamente receptivas adaptadas a las necesidades de su aplicación.

Atlas Vector Search aporta una búsqueda semántica impulsada por IA a MongoDB Atlas al aprovechar las incrustaciones vectoriales generadas por modelos de aprendizaje automático. Esta función permite búsquedas basadas en el significado contextual en lugar de solo en palabras clave, lo que la hace ideal para casos de uso como sistemas de recomendación, búsquedas de similitud de imágenes o procesamiento de lenguaje natural. Cuando se combinan con Atlas Search, estas capacidades permiten búsquedas híbridas que fusionan la precisión de las palabras clave con la comprensión semántica, brindando resultados de búsqueda más ricos y matizados.

Demostramos cómo implementar tanto Atlas Search como Atlas Vector Search, recorriendo ejemplos prácticos de creación de índices, diseño de consultas y estrategias de búsqueda híbrida. Además, exploramos aplicaciones prácticas, mostrando cómo estas herramientas pueden transformar las experiencias de búsqueda y elevar las soluciones impulsadas por IA. Por lo tanto, ya sea que esté creando un sistema de recomendación o mejorando las interacciones de los usuarios, ¡MongoDB Atlas tiene las herramientas para dar vida a sus datos!
