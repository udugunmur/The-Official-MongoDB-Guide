# Parte 3: Herramientas para Desarrolladores

## Capítulo 3: Herramientas para Desarrolladores

En el desarrollo de software moderno, la eficiencia y el éxito de la creación de aplicaciones están fuertemente influenciados por la disponibilidad y la calidad de las herramientas para desarrolladores. MongoDB ofrece un conjunto integral de dichas herramientas diseñadas para agilizar el proceso de desarrollo y mejorar la productividad. Estas herramientas permiten a los desarrolladores ensamblar, probar, optimizar y administrar sin problemas los diversos componentes necesarios para crear aplicaciones robustas.

Entre las herramientas más destacadas se encuentra MongoDB Compass, una interfaz gráfica de usuario que simplifica el proceso de creación y ejecución de consultas en la base de datos de MongoDB. Otra utilidad ampliamente adoptada es la extensión de MongoDB para Visual Studio Code (VS Code), que ayuda a los desarrolladores a crear consultas precisas y eficientes dentro de su entorno de desarrollo integrado preferido.

Más allá de la construcción de consultas, el conjunto de herramientas de MongoDB incluye capacidades para el análisis de datos, la automatización del flujo de trabajo y la integración de entornos, todo con el objetivo de mejorar la experiencia del desarrollador y la calidad del código. Si bien la gama completa de herramientas disponibles es amplia, este capítulo destaca aquellas con la utilidad más inmediata e impactante.

Este capítulo cubre las herramientas de desarrollo habilitadas para IA de MongoDB, que admiten la integración con marcos de inteligencia artificial (IA) y aprendizaje automático (*machine learning* o ML). Estas herramientas son particularmente beneficiosas para los desarrolladores que trabajan en aplicaciones inteligentes o predictivas, proporcionando un puente fluido entre la infraestructura de datos y el software web.

Este capítulo cubre las siguientes herramientas:

- Extensiones para entornos de desarrollo integrado (IDE)
- MongoDB Compass
- MongoDB MCP Server

---

### Sección 1: Extensiones para entornos de desarrollo integrado

Un entorno de desarrollo integrado (*Integrated Development Environment* o IDE) sirve como el centro neurálgico para la mayoría de los desarrolladores de software, proporcionando un conjunto integral de herramientas diseñadas para agilizar el proceso de codificación. Dentro del IDE, los desarrolladores pueden escribir, corregir y revisar minuciosamente su código de manera eficiente. Más allá de la edición de texto básica, los IDE suelen ofrecer características invaluables como el resaltado de sintaxis, que diferencia visualmente los elementos del código para mejorar la legibilidad e identificar errores. Las funcionalidades de autocompletado aceleran significativamente la codificación al sugerir y completar automáticamente fragmentos de código, reduciendo la escritura y los posibles errores tipográficos.

Los IDE a menudo se integran con varias otras herramientas de desarrollo, fomentando un flujo de trabajo más productivo y eficiente. Estas integraciones pueden incluir herramientas de depuración que permiten a los desarrolladores recorrer su código línea por línea, identificar errores lógicos e inspeccionar valores de variables en diferentes puntos de la ejecución. Las integraciones de sistemas de control de versiones (por ejemplo, Git) permiten a los desarrolladores gestionar cambios en su base de código, colaborar con miembros del equipo y revertir a versiones anteriores si es necesario. Los compiladores o intérpretes integrados permiten la prueba y ejecución inmediata del código, proporcionando retroalimentación instantánea sobre su funcionalidad.

Al consolidar estas funcionalidades esenciales en una interfaz única e intuitiva, los IDE permiten a los desarrolladores ser significativamente más productivos y rápidos en la creación de aplicaciones robustas y funcionales. Minimizan el cambio de contexto, reducen la necesidad de herramientas externas y proporcionan un entorno enriquecido que respalda todo el ciclo de vida del desarrollo de software, desde la codificación inicial hasta las pruebas y el despliegue.

MongoDB ofrece tres extensiones de IDE distintas que vale la pena destacar en este texto. La más popular es la extensión de VS Code, que es la que cubriremos con más detalle en las siguientes páginas, permitiéndole trabajar con MongoDB y sus datos directamente dentro de su entorno de codificación. También analizaremos brevemente el complemento de IntelliJ, diseñado para mejorar la experiencia de desarrollo para los desarrolladores de Java que trabajan con IntelliJ IDEA.

#### Extensión MongoDB para VS Code

La extensión MongoDB para VS Code integra las funcionalidades de MongoDB directamente en el entorno de VS Code. Le permite explorar e interactuar con sus datos de MongoDB sin problemas mientras programa. Algunas de sus características clave son las siguientes:

- **Exploración de datos:** Puede navegar a través de sus bases de datos, colecciones y documentos utilizando la extensión.
- **Prototipado de consultas:** Le permite crear prototipos de consultas y ejecutar comandos de MongoDB directamente dentro de VS Code.
- **Integración con GitHub Copilot:** La extensión incluye comandos específicos de MongoDB para asistir con GitHub Copilot, ayudándole a generar código e interactuar con sus despliegues de MongoDB.

Para comenzar, debe instalar la extensión desde VS Code Marketplace y conectarla a su despliegue de MongoDB. Una vez configurada, puede explorar sus datos y realizar varias operaciones directamente desde su entorno de codificación.

#### Primeros pasos con la extensión de VS Code

Antes de poder instalar la extensión MongoDB para VS Code, debe instalar VS Code. Puede descargar VS Code desde [https://code.visualstudio.com/](https://code.visualstudio.com/).

Luego, esto es lo que puede hacer:

1. Abra la vista de Extensiones en VS Code.
2. Haga clic en el icono de Extensiones en la navegación izquierda, como se muestra en la siguiente figura:

> **Figura 3.1:** El icono de Extensiones en VS Code

3. Busque `MongoDB for VS Code` en el Marketplace de Extensiones.
4. Haga clic en **Install**.
5. Una vez completada la instalación, el botón **Install** cambia al botón de engranaje **Manage**.

#### Exploración de datos

Para explorar sus datos en la extensión de VS Code, primero deberá conectar su entorno de VS Code a su base de datos de MongoDB. Esto se hace de forma muy sencilla siguiendo estos pasos:

1. Proporcione la cadena de conexión como una opción de configuración dentro de la extensión de VS Code en la paleta de comandos de VS Code (*Command Palette*), como se muestra en la Figura 3.2:

> **Figura 3.2:** La paleta de comandos de VS Code dentro de la extensión de MongoDB

2. Haga clic en **Connections**, luego expanda el menú **CONNECTIONS**:

> **Figura 3.3:** CONNECTIONS en la extensión de MongoDB para VS Code

3. Haga clic en el menú **More Actions**, indicado por tres puntos en la Figura 3.4:

> **Figura 3.4:** El menú More Actions en la interfaz de la extensión de MongoDB para VS Code

4. A continuación, pegue su cadena de conexión en la entrada del formulario.

Una vez que se haya conectado correctamente a su base de datos de MongoDB, podrá revisar sus datos y examinarlos más de cerca a través del menú de navegación del lado izquierdo.

En esta barra de navegación, puede ver la jerarquía completa, como las bases de datos, las colecciones dentro de la base de datos, los documentos dentro de la colección y los esquemas e índices que afectan a esos documentos.

Cuando expande el esquema de una colección, la extensión de VS Code enumera los campos de los documentos de la colección. Si un campo existe en todos los documentos y su tipo es consistente en toda la colección, la extensión de VS Code muestra un icono que indica el tipo de datos de ese campo. Luego puede pasar el cursor sobre el nombre del campo para obtener una descripción textual del tipo de datos del campo.

Para ver y editar campos individuales dentro de un documento, haga clic con el botón derecho en el ID del documento deseado. La extensión de VS Code abrirá el documento para su edición. Después de realizar los cambios deseados, presione Ctrl + S para guardar el documento.

Más allá de las funcionalidades principales, la extensión de VS Code para MongoDB ofrece un conjunto de características adicionales diseñadas para agilizar su flujo de trabajo y mejorar la productividad. Descubrirá herramientas potentes para administrar sus datos sin esfuerzo, incluida la capacidad de clonar documentos existentes con un solo clic, lo que facilita la iteración y las pruebas rápidas. Cuando se trata de la higiene de los datos, la extensión proporciona opciones intuitivas para eliminar documentos, ya sea individualmente o en masa, asegurando que sus colecciones permanezcan organizadas y relevantes.

Además, para situaciones que requieran una nueva entrada de datos o la creación de esquemas, puede insertar fácilmente nuevos documentos en blanco directamente dentro del entorno de VS Code, eliminando la necesidad de cambiar entre aplicaciones. Todas estas tareas, desde la manipulación compleja de datos hasta la creación de documentos fundamentales, están perfectamente integradas dentro de la conocida interfaz de VS Code, lo que permite a los desarrolladores mantener una experiencia de desarrollo consistente y eficiente.

La interfaz intuitiva hace que sea fácil de usar incluso para desarrolladores principiantes.

#### Prototipado de consultas

La extensión de VS Code también ofrece una función sólida para crear prototipos de consultas y agregaciones a través de MongoDB Playground. Este *playground* es un entorno de JavaScript equipado con funciones de resaltado de sintaxis y autocompletado.

Esta capacidad permite la experimentación inmediata con consultas, independientemente de su complejidad, y la verificación instantánea de los resultados frente a las expectativas. Todo esto se puede lograr sin modificaciones permanentes en el código de su aplicación. Los resultados satisfactorios se pueden guardar al instante.

A continuación se explica cómo utilizar MongoDB Playground en VS Code:

1. Abra la paleta de comandos de VS Code. En Windows y Linux, presione Ctrl + Shift + P. En macOS, presione ⌘ + Shift + P.
2. Seleccione el comando para crear un MongoDB Playground, que abrirá una plantilla de MongoDB Playground predeterminada y preconfigurada:

> **Figura 3.5:** Plantilla predeterminada de MongoDB Playground en VS Code

3. Para ejecutar un *playground*, simplemente haga clic en el botón **Play** en la barra de navegación superior de VS Code, y su *playground* se ejecutará contra el despliegue especificado en la conexión activa que pegó anteriormente:

> **Figura 3.6:** Ejecución de MongoDB Playground utilizando el botón Play en VS Code

#### Capacidades CRUD en MongoDB Playground para VS Code

Veamos un ejemplo de cómo podría realizar una operación CRUD en la extensión de VS Code dentro de MongoDB Playground:

1. Para comenzar, abra la paleta de comandos de VS Code.
2. Busque el comando **Create MongoDB Playground** desde la barra de búsqueda y ejecútelo.
3. Utilice el método `insertOne()` para insertar un documento en la colección, como en este ejemplo:

```javascript
db.cars.insertOne(
  { "_id" : 1, "item" : "corvette", "price" : 60000, "quantity" : 1, "dateofPurchase" : new Date("2019-03-01T08:00:00Z")}
);
```

En este ejemplo, estamos insertando un nuevo documento en la colección `cars`.

Para ejecutar el *playground*, después de haber ingresado el código, presione el botón **Play** en la parte superior derecha del *playground*. La extensión de VS Code dividirá su *playground*: en el lado izquierdo, verá el código que acaba de ingresar; en el lado derecho, verá los resultados del *playground*. Esto le brinda una vista dinámica en tiempo real de cómo se está ejecutando su código.

También puede leer los documentos en su colección en su base de datos existente utilizando el método `findOne()` o `find()` para encontrar más de un documento.

Como antes, esto requiere que ya haya ingresado su conexión a su base de datos de MongoDB y que ya haya abierto el *playground*, y que existan documentos en su base de datos previamente.

Por ejemplo, el siguiente código devolverá un documento de la colección `cars`:

```javascript
db.cars.findOne();
```

También puede utilizar los métodos `updateOne()` o `updateMany()` para modificar documentos existentes en la colección `cars` en la base de datos de MongoDB.

#### Prototipado de agregaciones en MongoDB Playground para VS Code

Dentro de la extensión de VS Code MongoDB Playground, puede ejecutar y probar canalizaciones de agregación (*aggregation pipelines*). Esta funcionalidad permite ejecutar canalizaciones de múltiples etapas que manipulan datos y generan resultados calculados. Estos resultados se muestran directamente en el *playground* de VS Code, lo que permite la verificación inmediata de la salida de la agregación. Los usuarios pueden entonces confirmar si los resultados se alinean con sus expectativas. Si los resultados son satisfactorios, se puede guardar la consulta de agregación o los resultados generados.

Si no es así, puede continuar refinando su consulta de agregación hasta que los resultados coincidan con sus expectativas. Esto es particularmente útil si está aprendiendo la sintaxis de agregación de MongoDB o intentando determinar en qué orden deben ocurrir ciertas operaciones de agregación.

Querrá utilizar las mejores prácticas generales en torno a la construcción de canalizaciones de agregación de MongoDB, con múltiples etapas definidas. Si no está familiarizado con las canalizaciones de agregación de MongoDB, consulte la documentación ubicada aquí: [https://www.mongodb.com/docs/manual/aggregation/](https://www.mongodb.com/docs/manual/aggregation/).

Por ahora, sin embargo, veamos un ejemplo simple. La sintaxis general de una canalización de agregación es la siguiente:

```javascript
db.<collection>.aggregate([
 {
  <$stage1>
 },
 {
  <$stage2>
 }  
//  ...  
])
```

Una canalización de agregación procesa datos a través de una serie de etapas, donde la salida de una etapa se convierte en la entrada de la siguiente. Un ejemplo sencillo es calcular el promedio de ventas. Si tiene 1,000 clientes, algunos con múltiples compras y muchos sin ninguna, determinaría el promedio de ventas por cliente dividiendo los ingresos totales por el número total de clientes.

Alternativamente, para calcular el promedio solo para aquellos clientes con al menos una compra, la etapa inicial implicaría filtrar a los clientes cuyas ventas superen los $0.00.

Para hacer esto, utilizará `$match` para encontrar a esos clientes y luego `$group` para calcular el valor promedio de sus compras.

La agregación resultante se vería así, en JavaScript:

```javascript
const pipeline = [
 // Stage 1: Match customers with purchases greater than 0.00
 {
  "$match": {
   "totalPurchases": { "$gt": 0.00 }
  }
 },  
 // Stage 2: Group by customerId and calculate the average purchase amount  
 {
  "$group": {
   "_id": "$customerId",
   "averagePurchaseAmount": { "$avg": "$totalPurchases" }
  }
 }
];

// Execute the aggregation pipeline  
return customersCollection.aggregate(pipeline).toArray()  
 .then(customers => {
   console.log(`Successfully calculated average purchase amount for ${customers.length} customers.`)  
  for(const customer of customers) {
   console.log(`customer: ${customer._id}`)
   console.log(`average purchase amount: ${customer.averagePurchaseAmount}`)
  }
  return customers
 })
```

Para profundizar en su comprensión de las agregaciones de MongoDB, puede consultar *Practical Aggregations* de Paul Done. Este libro, combinado con el prototipado de sus agregaciones en MongoDB Playground para VS Code, le permitirá verificar rápidamente si los resultados se alinean con sus expectativas.

#### Extensión IDE de MongoDB para IntelliJ

En 2025, MongoDB presentó su extensión IDE para IntelliJ. Con este complemento, los desarrolladores reciben autocompletado mejorado y validación de tipos para su código. IntelliJ ya tenía varias características útiles integradas:

- Conecta a la base de datos de MongoDB
- Explora bases de datos, colecciones y documentos
- Prototipa consultas

Pero con la extensión, recibe aún más capacidades. Ahora puede indicar al desarrollador cuándo se está ejecutando una consulta sin un índice, así como sugerirle una consulta más óptima a medida que escribe el código, en lugar de hacerlo a posteriori. Cuenta con soporte para marcos de trabajo y patrones de consulta comunes, como Java Spring. Para acceder al complemento, simplemente navegue a [https://plugins.jetbrains.com/plugin/24377-mongodb](https://plugins.jetbrains.com/plugin/24377-mongodb).

En el momento de escribir este texto, el complemento IDE para IntelliJ se encuentra en versión preliminar pública (*public preview*), lo que significa que aún no está disponible de forma general (*GA*).

---

### Sección 2: Compass

La mayoría de los desarrolladores preferiría dedicar tiempo a escribir código en lugar de hacer introspección en su base de datos. Con ese fin, MongoDB Compass es una herramienta para desarrolladores invaluable, que sirve como una interfaz gráfica de usuario (GUI) robusta que agiliza la interacción con su base de datos.

Ofrece un entorno fácil de usar donde puede crear y ejecutar consultas de manera intuitiva. Si no se siente seguro al escribir consultas en MongoDB, no se preocupe: su función de consulta en lenguaje natural tomará sus descripciones de texto y las convertirá en MQL funcional (¡o incluso en código Java!).

Más allá de las consultas, Compass proporciona funciones integrales para lo siguiente:

- Monitorear la calidad y consistencia de los datos, lo que le permite identificar y abordar fácilmente discrepancias o problemas de datos dentro de sus conjuntos de datos.
- Agregar índices y monitorear el rendimiento de la base de datos, incluidas las consultas subóptimas.
- Construir agregaciones en un formato visual de arrastrar y soltar.
- Importar y exportar datos.
- Crear y hacer cumplir esquemas en toda su base de datos.

Su naturaleza visual y sus potentes funcionalidades lo convierten en un activo esencial para los desarrolladores que buscan administrar, analizar y optimizar sus bases de datos de MongoDB de manera eficiente.

#### Primeros pasos

MongoDB Compass se puede descargar gratis y está disponible para Linux, macOS y Windows en este enlace: [https://www.mongodb.com/try/download/compass](https://www.mongodb.com/try/download/compass). Una vez que lo haya descargado, habrá un modal de instalación que lo guiará a través del proceso.

Luego, querrá conectar Compass a su base de datos de MongoDB para que pueda comenzar a experimentar con sus datos. Así es como puede hacerlo:

1. Abra el modal **New Connection**: en el panel inferior de la barra lateral **CONNECTIONS**, haga clic en **Add new connection** para abrir el modal New Connection.

> **Figura 3.7:** Modal Add New Connection de Compass

2. Pegue su cadena de conexión.
3. Ahora, puede nombrar su conexión. Esto es útil si tiene múltiples bases de datos a las que se conectará, como producción y pruebas.
4. Conéctese a su clúster. El botón predeterminado **Save & Connect** guarda su información y lo conecta a su clúster.

> **Figura 3.8:** Conexión a un clúster de MongoDB usando Save & Connect en Compass

Para acceder a varias funciones de Compass, es posible que se requieran roles de usuario específicos una vez conectado a su despliegue de MongoDB. Como ejemplo, si tiene acceso de solo lectura a la base de datos como individuo, no se le permitirá editar documentos a través de la interfaz de usuario de Compass. Si experimenta problemas al utilizar Compass, lo primero que debe hacer para solucionar el problema es verificar sus permisos de usuario en la base de datos.

#### Interactuar con sus datos

Interactuar con sus datos es una parte crítica para trabajar eficazmente con MongoDB, y Compass proporciona una interfaz intuitiva para ayudarle a hacerlo. Con Compass, puede visualizar, consultar y manipular fácilmente sus datos, lo que permite obtener conocimientos más profundos y flujos de trabajo más eficientes. Si bien Compass admite una amplia gama de actividades, esta sección se centrará en una selección de funciones clave:

- Administrar documentos (ver, insertar, actualizar y eliminar documentos en su base de datos)
- Consultar sus datos
- Administrar índices
- Analizar su esquema
- Ver el rendimiento de las consultas en tiempo real

Para obtener más información sobre cómo ayuda interactuar con sus datos, consulte la documentación de MongoDB aquí: [https://www.mongodb.com/docs/compass/manage-data/](https://www.mongodb.com/docs/compass/manage-data/).

#### Administrar documentos

Dentro de la GUI de Compass, navegue hasta la pestaña **Documents**. Esta sección le permite ver, insertar, modificar y eliminar documentos dentro de su base de datos de MongoDB. Por defecto, los documentos se muestran en una vista de lista (*List view*), mostrando cada documento junto con algunos de sus campos. En la siguiente figura, puede ver tres documentos de ejemplo mostrados en la vista de lista. Luego puede expandirlos para ver todos los diversos campos y valores por documento:

> **Figura 3.9:** Vista de lista (List view)

Para aquellos de ustedes que están más acostumbrados a las bases de datos tabulares y relacionales, pueden apreciar la vista de tabla (*Table view*) de sus documentos. La vista de tabla se puede seleccionar en el extremo derecho:

> **Figura 3.10:** Vista de tabla de Documentos (Table view)

Para insertar datos, haga clic en el botón **Add Data** en la parte superior derecha de la pantalla, como se muestra en la Figura 3.11. Esto lo llevará a una nueva pantalla donde podrá insertar un documento y definir sus campos y valores.

> **Figura 3.11:** Ver el documento en JSON

También puede modificar un documento haciendo clic en el icono de lápiz en la vista de Lista o Tabla, como se muestra aquí:

> **Figura 3.12:** El botón Edit document

Hacer clic en el icono de la papelera eliminará el documento, así que use ese botón en particular con precaución.

Como puede ver, es increíblemente fácil e intuitivo administrar sus documentos de esta manera. De hecho, para aquellos desarrolladores que son nuevos en MongoDB, Compass es fácilmente la mejor manera de comenzar a aprender sobre la base de datos y familiarizarse con cómo almacena los datos, así como evaluar cómo difieren sus documentos entre sí en una configuración de esquema flexible.

#### Consultar documentos en MongoDB Compass

Consultar documentos en MongoDB Compass también es igualmente fácil. Simplemente ingresa la consulta de MongoDB en la barra de filtros, así:

> **Figura 3.13:** El documento en Compass

Cualquier consulta de MongoDB correctamente formada funcionará dentro de esta barra de filtros. Una vez que tenga los resultados de su consulta, puede validar visualmente si los resultados son los esperados. También puede exportar los resultados de su consulta.

#### Consultas en lenguaje natural en Compass

En caso de que no conozca el lenguaje de consulta de MongoDB, no tema. También puede escribir consultas en lenguaje natural, utilizando IA generativa para interpretar su texto y luego reescribirlo en un lenguaje de consulta correctamente formado. Para hacer eso, primero necesitará tener una cuenta de MongoDB Atlas, que es la plataforma de base de datos como servicio de MongoDB. Luego, tendrá que habilitar la IA generativa en la configuración de Compass.

Esto se puede lograr fácilmente modificando el archivo de configuración en su computadora ubicado en `/etc/mongodb-compass.conf`. Configúrelo de la siguiente manera:

```json
{
  "enableGenAIFeatures": true,
  # ...
}
```

Regrese a la GUI de Compass y envíe su pregunta o solicitud sobre la colección en la barra de consulta, como `Which movies were released in 2000?` (*¿Qué películas se estrenaron en el año 2000?*).

Aprovechando el poder de la IA, Compass utiliza las capacidades avanzadas de Azure OpenAI para una revisión exhaustiva y un análisis en profundidad de su consulta. Además, basándose en esta comprensión completa, Azure OpenAI reescribirá y refinará inteligentemente su consulta, optimizándola para mayor claridad, eficiencia y precisión dentro del entorno de MongoDB. Este proceso garantiza que sus intenciones se traduzcan en las operaciones de base de datos más efectivas y de mayor rendimiento.

En conclusión, MongoDB Compass es una herramienta poderosa para desarrolladores que desean una interfaz de usuario simple para interactuar con su base de datos.

---

### Sección 3: MongoDB MCP Server

MongoDB Model Context Protocol (MCP) Server es una nueva característica introducida en versión preliminar pública (*public preview*), diseñada para mejorar la interacción entre los agentes de IA y los clústeres de MongoDB mediante instrucciones en lenguaje natural. Desarrollado originalmente por Anthropic, MCP facilita la conexión de modelos de lenguaje grande (LLM) de IA a los sistemas de datos, asegurando que estos modelos tengan acceso a la información más actualizada de la base de datos. Esta integración permite una gestión y consulta de datos más intuitiva y eficiente, aprovechando las capacidades de la IA para agilizar las operaciones.

MCP Server es particularmente útil para conectar varios servidores de MongoDB, incluidos Atlas, Community Edition, Enterprise Advanced e instalaciones locales (*on-premises*), a clientes compatibles con MCP. Esta conectividad permite a los usuarios utilizar herramientas impulsadas por IA para interactuar con sus bases de datos, lo que facilita la realización de consultas complejas y manipulaciones de datos sin necesidad de una gran experiencia técnica. Al proporcionar un puente entre las tecnologías de IA y los sistemas de bases de datos, MCP Server abre nuevas posibilidades para el análisis de datos y el desarrollo de aplicaciones.

Los clientes que actualmente admiten MCP incluyen Anthropic Claude Desktop, Windsurf Editor, Cursor y VS Code GitHub Copilot.

MongoDB MCP Server le permite realizar varias tareas aprovechando herramientas impulsadas por IA para interactuar con bases de datos de MongoDB. Estas son algunas de las cosas que puede hacer a través de MCP:

- **Consultas en lenguaje natural:** Puede utilizar instrucciones en lenguaje natural para consultar sus bases de datos de MongoDB. Esto facilita la realización de consultas complejas sin necesidad de escribir código tradicional, lo que permite una interacción más intuitiva con los datos.
- **Gestión de datos:** MCP facilita la gestión eficiente de datos al conectar agentes de IA a clústeres de MongoDB. Esta integración ayuda a agilizar las operaciones, facilitando la manipulación y el análisis de datos mediante capacidades de IA.
- **Desarrollo de aplicaciones mejorado:** Al unir las tecnologías de IA con los sistemas de bases de datos, MCP abre nuevas posibilidades para el desarrollo de aplicaciones. Puede utilizar herramientas impulsadas por IA para mejorar sus flujos de trabajo, haciendo que las interacciones con la base de datos sean más accesibles y eficientes.

El código también está disponible como código abierto aquí: [https://github.com/mongodb-js/mongodb-mcp-server/tree/main](https://github.com/mongodb-js/mongodb-mcp-server/tree/main).

Para comenzar con MCP Server, siga estos pasos:

1. **Instale un cliente compatible:** Elija un cliente que admita MCP, como Anthropic Claude Desktop, Windsurf Editor, Cursor o VS Code GitHub Copilot. Por ejemplo, si elige Claude Desktop, descárguelo e instálelo desde la página de descargas de Claude.
2. **Configure el cliente:** Una vez instalado, inicie el cliente y abra el archivo de configuración. Para Claude Desktop, iría a **Settings -> Developer -> Edit Config**. En el archivo de configuración, agregue los detalles del servidor MongoDB MCP. Puede copiar las siguientes líneas en el archivo de configuración:

```json
{
  "mcpServers": {
   "MongoDB": {
     "command": "npx",
     "args": ["-y", "mongodb-mcp-server"]
   }
  }
}
```

3. **Conéctese a MongoDB:** Asegúrese de que su servidor MongoDB se esté ejecutando y sea accesible. MCP Server puede conectarse a varios servidores MongoDB, incluidos Atlas, Community Edition, Enterprise Advanced e instalaciones locales. Una vez configurado, puede comenzar a interactuar con sus clústeres de MongoDB mediante instrucciones en lenguaje natural a través del cliente.

Si bien esta característica aún no está lista para producción en el momento de escribir este texto, puede obtener más información sobre ella aquí: [https://github.com/mongodb-js/mongodb-mcp-server/blob/main/README.md](https://github.com/mongodb-js/mongodb-mcp-server/blob/main/README.md).

---

### Sección 4: Resumen

Este capítulo exploró las herramientas esenciales para desarrolladores proporcionadas por MongoDB, diseñadas para mejorar la eficiencia y agilizar el proceso de desarrollo. Comenzamos analizando las extensiones de IDE, destacando la extensión MongoDB para VS Code, que integra perfectamente las funcionalidades de MongoDB directamente en el entorno de VS Code. Esta extensión capacita a los desarrolladores con sólidas capacidades de exploración de datos, creación de prototipos de consultas a través de MongoDB Playground y operaciones CRUD, todo dentro de su IDE preferido. La integración con GitHub Copilot aumenta aún más la productividad al ayudar con la generación de código y la interacción con los despliegues de MongoDB.

Luego profundizamos en MongoDB Compass, una potente GUI que simplifica la creación de consultas, el monitoreo de datos y la aplicación de esquemas. Su naturaleza visual y sus características intuitivas la convierten en un activo invaluable para administrar y optimizar bases de datos de MongoDB.

Finalmente, presentamos MongoDB MCP Server, una función de vanguardia en versión preliminar pública que une los agentes de IA con los clústeres de MongoDB. MCP permite consultas en lenguaje natural y gestión de datos, ofreciendo una forma más intuitiva y eficiente de interactuar con bases de datos mediante herramientas impulsadas por IA. Esta tecnología es muy prometedora para mejorar el desarrollo de aplicaciones y el análisis de datos al hacer más accesibles las interacciones complejas con las bases de datos.

Juntas, estas herramientas (Compass, la extensión de VS Code y MCP Server) forman un conjunto integral que permite a los desarrolladores compilar, probar, optimizar y administrar aplicaciones robustas con MongoDB. Al aprovechar estas herramientas, los desarrolladores pueden mejorar significativamente su flujo de trabajo, mejorar la calidad del código y abordar de manera eficiente las complejidades de la infraestructura de datos moderna.

En última instancia, la misión de MongoDB siempre ha sido crear una base de datos que sea fácil de usar para los desarrolladores. Como tal, estas herramientas tienen como objetivo mejorar la vida del desarrollador, razón por la cual vemos un enfoque tan marcado en las herramientas en entornos integrados para desarrolladores como VS Code e IntelliJ.

Para leer más sobre las herramientas de desarrollo de MongoDB, incluidos los controladores oficiales, las integraciones con marcos web como Spring y FastAPI, así como los marcos de IA/ML como LangChain, consulte la documentación oficial de MongoDB para herramientas de desarrollo ubicada en [https://www.mongodb.com/docs/drivers/](https://www.mongodb.com/docs/drivers/) y [https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/).
