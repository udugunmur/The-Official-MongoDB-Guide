# Parte 8: MongoDB Atlas

## Capítulo 8: MongoDB Atlas

MongoDB Atlas es una solución integral diseñada para optimizar y mejorar la gestión de datos para los desarrolladores. Esta plataforma está diseñada para satisfacer las necesidades de las aplicaciones modernas, que a menudo implican el manejo de grandes volúmenes de datos, requieren automatización y demandan una rápida escalabilidad y adaptabilidad al cambio.

MongoDB Atlas proporciona una interfaz sencilla y fácil de usar para el mantenimiento de bases de datos, junto con copias de seguridad automatizadas e instantáneas de datos en un momento dado (*point-in-time snapshots*). Atlas incorpora una amplia gama de funciones, incluido el escalado automatizado, un monitoreo y alertas robustos, Vector Search y búsqueda de texto completo, disparadores (*triggers*), modelos de incrustación (*embedding models*) y herramientas de optimización del rendimiento para diagnosticar y mejorar consultas mal construidas.

El objetivo de estas funciones es crear una plataforma única que una diferentes cargas de trabajo y atienda a los desarrolladores de toda la organización. MongoDB Atlas se adapta tanto a empresas emergentes (*start-ups*) como a empresas bien establecidas, garantizando que se satisfagan todas las necesidades de bases de datos basadas en la nube.

A medida que examinemos las nuevas funciones lanzadas con MongoDB 8.0, obtendrá una comprensión clara de cómo MongoDB Atlas no es solo un servicio de base de datos, sino un activo versátil y potente diseñado para satisfacer las demandas de los desarrolladores modernos.

Este capítulo cubrirá los siguientes temas:

- Configuración de su organización
- Exploración de herramientas para interactuar con sus despliegues de Atlas
- Dimensionamiento de un clúster de Atlas
- Escalado en Atlas
- Configuración de autenticación y autorización
- Protección de su despliegue de Atlas
- Servicios obsoletos en Atlas

Esta descripción general no cubrirá todas y cada una de las funciones de MongoDB Atlas, ya que eso sería demasiado extenso. En su lugar, combina funcionalidades introducidas recientemente con conceptos fundamentales. El objetivo es proporcionarle los conocimientos necesarios para comenzar a utilizar Atlas rápidamente y acelerar su innovación.

---

### Sección 1: Requisitos técnicos

Para seguir este capítulo, necesitará tener lo siguiente:

- Cuenta de MongoDB Atlas
- Python 3.7+
- Atlas CLI

Puede encontrar los ejemplos de código utilizados en GitHub: [https://github.com/PacktPublishing/The-Official-MongoDB-Guide](https://github.com/PacktPublishing/The-Official-MongoDB-Guide).

---

### Sección 2: Configuración de su organización

En MongoDB Atlas, comprender las organizaciones, los proyectos y los clústeres es fundamental, ya que estos elementos forman el marco básico para administrar sus bases de datos. Las organizaciones son los contenedores de nivel superior que abarcan proyectos, mientras que los proyectos pueden alojar múltiples clústeres. Cada nivel, es decir, los niveles de organización, proyecto y clúster, tiene un rol distinto en la gestión de recursos, acceso y datos dentro de Atlas.

Exploremos estos elementos clave.

#### Gestión de organizaciones en MongoDB Atlas

A nivel de organización, MongoDB Atlas le proporciona herramientas para gestionar la estructura general que gobierna todos los proyectos. Esto implica establecer políticas globales, definir reglas operativas y mantener parámetros de seguridad que se aplican de manera uniforme en todos los proyectos. Las siguientes son capacidades clave proporcionadas por Atlas para facilitar una gestión eficiente a nivel de organización:

- **Facturación centralizada y gestión de acceso:** Atlas proporciona facturación centralizada, ofreciendo una vista integral de los gastos en todos los proyectos, lo que agiliza la gestión financiera. La gestión del acceso es primordial para asignar roles y permisos de usuario, garantizando tanto la seguridad como la eficiencia operativa.
- **Establecimiento de políticas de recursos:** Como propietario de una organización, tiene la responsabilidad de crear políticas de recursos. Estas dictan cómo los proyectos y los clústeres utilizan sus recursos, como establecer límites en las configuraciones de los clústeres, elegir regiones de la nube y determinar estándares para el escalado y el rendimiento. Estas medidas garantizan una gobernanza eficaz alineada con estrategias operativas y financieras más amplias.
- **Flexibilidad:** Los usuarios pueden participar en múltiples organizaciones simultáneamente. Esto fomenta la colaboración entre equipos y proyectos, admitiendo interacciones fluidas mientras se mantienen límites claros y controles de seguridad sólidos para cada organización distinta.
- **Mejora de la eficiencia con etiquetas de recursos (*resource tags*):** MongoDB Atlas ha introducido etiquetas de recursos para organizaciones y facturación, una nueva característica que le permite categorizar sus proyectos y clústeres. Con las etiquetas, puede filtrar y organizar sistemáticamente los recursos según criterios específicos, como departamento, entorno o centro de costos. Esta adición simplifica la gestión, mejora la visibilidad y conecta las etiquetas directamente con la facturación, facilitando un mejor seguimiento financiero y rendición de cuentas.

#### Configuración a nivel de proyecto

Al centrarse en el nivel de proyecto, el énfasis cambia a la configuración y gestión de componentes y aplicaciones individuales relacionados con iniciativas o equipos específicos. Dentro de una organización, puede tener hasta 250 proyectos, lo que ofrece un amplio espacio para crecer y segmentar diferentes cargas de trabajo. Esta estructura mejora el aislamiento del equipo y del entorno, asegurando que cada proyecto funcione de forma independiente sin interferencias de los demás.

La separación de organizaciones y proyectos ofrece importantes beneficios, como segregación de funciones, mayor aislamiento, gestión optimizada y mejor escalabilidad. Este enfoque estructurado es escalable y da cabida a proyectos e infraestructura adicionales a medida que evolucionan las necesidades de su organización.

#### Comprensión de los clústeres

En Atlas, un clúster sirve como su despliegue de base de datos, compuesto por conjuntos de réplicas (*replica sets*) que ejecutan la base de datos de MongoDB para almacenar y gestionar sus datos. Los clústeres proporcionan alta disponibilidad y escalabilidad, lo cual es ideal para soportar diferentes necesidades de aplicaciones.

Atlas le permite implementar clústeres en múltiples regiones geográficas y proveedores de nube, incluidos Amazon Web Services (AWS), Microsoft Azure y Google Cloud Platform (GCP). Esta capacidad multinube ayuda a evitar la dependencia del proveedor (*vendor lock-in*), mejorando la resiliencia y el poder de negociación. Proporciona redundancia y garantiza operaciones continuas incluso durante interrupciones del proveedor, al tiempo que accede a las fortalezas y ofertas únicas de cada plataforma en la nube.

Una estrategia multinube también fortalece los planes de recuperación ante desastres al distribuir datos entre diferentes entornos, minimizando los riesgos y manteniendo la disponibilidad. Atlas permite a las organizaciones alinear su arquitectura de base de datos con los objetivos de continuidad del negocio.

Atlas ofrece varios niveles de clúster adaptados a casos de uso específicos; el nivel gratuito M0 es perfecto para el aprendizaje y proyectos de pequeña escala, mientras que los clústeres dedicados gestionan cargas de trabajo de producción y aplicaciones a gran escala. Dentro de un solo proyecto, puede establecer hasta 25 clústeres, admitiendo diversos entornos como pruebas, desarrollo y producción. Sin embargo, solo se permite un clúster de nivel gratuito M0 por proyecto.

A medida que avancemos en este capítulo, profundizaremos en los detalles de los clústeres y le guiaremos en la configuración del suyo propio.

---

### Sección 3: Exploración de herramientas para interactuar con MongoDB Atlas

Después de configurar su organización, proyectos y clústeres en MongoDB Atlas, el siguiente paso es aprovechar las herramientas de la plataforma para desbloquear todo el potencial de sus despliegues de bases de datos. Con una gran selección de opciones, MongoDB Atlas equipa tanto a nuevos usuarios como a desarrolladores experimentados con herramientas diseñadas para agilizar la gestión, mejorar la eficiencia y optimizar el rendimiento. En esta sección, presentaremos herramientas clave, que van desde la Interfaz de Usuario (UI) de Atlas para una navegación intuitiva hasta opciones de acceso programático como la API de administración de Atlas (*Atlas Admin API*), que respaldan desde operaciones cotidianas hasta flujos de trabajo de automatización avanzados. Exploremos cómo estas herramientas pueden mejorar su experiencia con MongoDB Atlas.

#### Atlas UI

Para quienes se embarcan en su viaje por MongoDB Atlas, la interfaz de usuario de Atlas sirve como un punto de partida intuitivo. Las mejoras recientes en la interfaz de usuario se centran en alinearse con el flujo de trabajo del desarrollador, lo que da como resultado una mayor eficiencia y una navegación fácil de usar. Esta actualización garantiza que cada interacción con sus datos sea fluida y reveladora.

> **Figura 8.1:** Atlas UI

Con una navegación intuitiva, la interfaz de usuario de Atlas brinda fácil acceso a organizaciones, proyectos y clústeres, ofreciendo una visión integral de sus recursos. Las características notables incluyen:

- **Data Explorer:** Perfecto para una rápida inspección y actualización de datos sin necesidad de escribir código. Empodera a las organizaciones facilitando el monitoreo, la gestión y el ajuste de datos en todos los proyectos y clústeres.
- **Atlas Search:** Capacidades de búsqueda integradas dentro de sus datos.
- **Paneles de monitoreo (*Dashboards*):** Visualice las métricas del clúster y el estado de la base de datos en tiempo real.
- **Performance Advisor:** Una herramienta diseñada para clústeres que ejecutan niveles M10 o superiores. Ayuda a las organizaciones a optimizar el rendimiento de las consultas analizando las consultas de ejecución lenta y recomendando estrategias de indexación eficientes. Disponible dentro de la interfaz de Atlas, el Performance Advisor mejora el desarrollo de proyectos y las operaciones de clústeres al identificar y eliminar cuellos de botella, reducir los costos operativos y garantizar una utilización óptima de los recursos, lo que permite a los equipos ofrecer mejores experiencias de usuario y mejorar el rendimiento general del sistema.

Cada función proporciona perspectivas únicas para gestionar e interactuar con sus datos, lo que convierte a la interfaz de usuario de Atlas en una herramienta poderosa para usuarios de cualquier nivel.

#### Atlas CLI

Para quienes prefieren utilizar la terminal, la interfaz de línea de comandos de Atlas (Atlas CLI) es una herramienta invaluable para administrar despliegues de MongoDB Atlas directamente desde la línea de comandos. Las mejoras recientes han elevado aún más sus capacidades, particularmente con la adición de soporte para entornos de desarrollo local. Cuando se utiliza junto con Docker, la configuración de conjuntos de réplicas de un solo nodo se vuelve muy sencilla, lo que reduce significativamente el tiempo y la complejidad de la configuración.

Este soporte de desarrollo local es especialmente útil ya que permite a los desarrolladores crear, probar y perfeccionar sus aplicaciones en un entorno que refleja fielmente el de producción. Esto garantiza transiciones más fluidas al implementar en la nube y reduce el riesgo de problemas inesperados que surjan de discrepancias en el entorno.

Lo mejor de todo es que es fácil de poner en marcha. Todo lo que necesita tener instalado es la CLI de Atlas y Docker. Después de eso, ejecute el siguiente comando en su terminal:

```bash
atlas deployments setup
```

Esto abrirá un mensaje de configuración interactivo que le permitirá configurar su implementación local de Atlas.

Si está interesado en obtener más información sobre los despliegues locales de Atlas, asegúrese de visitar la documentación de Atlas ([https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/)), donde podrá encontrar más información y tutoriales relacionados con esta nueva función.

Además, la CLI de Atlas ofrece varias funcionalidades clave que agilizan su flujo de trabajo:

- **Gestión de recursos:** Crear, modificar y eliminar organizaciones, proyectos y clústeres
- **Procesos automatizados:** Crear scripts y automatizar tareas repetitivas para ahorrar tiempo y reducir errores
- **Seguridad integrada:** Configurar y administrar parámetros de seguridad, incluidos niveles de acceso y autenticación
- **Personalización del despliegue:** Adaptar las implementaciones de clústeres a las necesidades cambiantes de sus aplicaciones
- **Monitoreo y análisis:** Acceder a métricas y datos de rendimiento en tiempo real

En resumen, la CLI de Atlas proporciona una forma sólida de administrar MongoDB Atlas de manera eficiente desde la línea de comandos, admitiendo con facilidad tanto el desarrollo local como los despliegues en la nube.

#### Acceso programático y automatización

Para los desarrolladores interesados en el control programático y la automatización, MongoDB Atlas proporciona herramientas para simplificar la gestión de bases de datos:

- **Atlas Admin API:** Una API RESTful que ofrece un control integral sobre su entorno de MongoDB Atlas. Permite la automatización y la integración de operaciones con sistemas existentes, permitiendo flujos de trabajo de DevOps fluidos.
- **Operador de Kubernetes (*Kubernetes Operator*):** Diseñado para despliegues nativos de la nube, integra clústeres de MongoDB dentro de un entorno de Kubernetes utilizando gestión declarativa y Definiciones de Recursos Personalizados (*Custom Resource Definitions* o CRD) para una automatización consistente.
- **Proveedor de HashiCorp Terraform para MongoDB Atlas:** Permite administrar recursos de Atlas como infraestructura como código utilizando HashiCorp Configuration Language (HCL). Esto le permite definir, implementar y actualizar su entorno de Atlas mediante programación, integrando Atlas en sus flujos de trabajo de entrega continua.
- **Recursos de AWS CloudFormation para Atlas:** Simplifica el aprovisionamiento y la gestión de funciones de Atlas en AWS mediante plantillas basadas en YAML o JSON. Esto permite el despliegue rápido y confiable de servicios o aplicaciones (*stacks*) que se pueden actualizar o replicar fácilmente según sea necesario.

El operador de Kubernetes de Atlas representa un avance significativo para las organizaciones que adoptan enfoques nativos de la nube para el despliegue de aplicaciones. Pero ¿qué hace exactamente y cómo puede agilizar las operaciones de su base de datos? Exploremos esta poderosa herramienta en profundidad.

---

### Sección 4: ¿Qué es el operador de Kubernetes de Atlas?

En esencia, el operador de Kubernetes de Atlas le permite administrar sus recursos de MongoDB Atlas directamente desde su entorno de Kubernetes. Piense en él como un puente que conecta sus clústeres de Kubernetes con MongoDB Atlas, permitiéndole definir y administrar recursos de Atlas utilizando el mismo enfoque declarativo que utiliza para el resto de su infraestructura de Kubernetes.

¿Cómo funciona esto en la práctica? Cuando implementa el operador de Kubernetes de Atlas en su clúster de Kubernetes, introduce CRD que representan recursos de Atlas, es decir, proyectos, despliegues de bases de datos, usuarios y más. Estas CRD le permiten describir el estado deseado de Atlas en archivos YAML, tal como definiría Pods, Services o Deployments de Kubernetes.

#### El poder de la gestión declarativa

¿Qué hace que este enfoque sea tan poderoso? Considere este escenario: necesita aprovisionar un nuevo clúster de MongoDB para su aplicación. Sin el operador de Kubernetes, podría hacer lo siguiente:

1. Iniciar sesión en la interfaz de usuario de Atlas.
2. Navegar a través de varias pantallas para crear un proyecto.
3. Configurar e implementar un clúster.
4. Configurar usuarios de base de datos.
5. Configurar el acceso a la red.

Con el operador de Kubernetes de Atlas, simplemente define estos recursos en YAML:

```yaml
apiVersion: atlas.mongodb.com/v1  
kind: AtlasProject  
metadata:  
  name: my-project  
spec:  
  name: "Production Application"  
  projectIpAccessList:  
    - ipAddress: "192.0.2.0/24"  
      comment: "Production network"  
---  
apiVersion: atlas.mongodb.com/v1  
kind: AtlasDeployment  
metadata:  
  name: my-cluster  
spec:  
  projectRef:  
    name: my-project  
  deploymentSpec:  
    name: "Production Cluster"  
    providerSettings:  
      instanceSizeName: M10  
      providerName: AWS
```

El código anterior es solo un ejemplo visual y no está diseñado para ejecutarse localmente. Para comenzar con el operador de Kubernetes de Atlas, consulte la documentación de Atlas sobre él ([https://www.mongodb.com/docs/atlas/operator/v2.7/ak8so-quick-start/](https://www.mongodb.com/docs/atlas/operator/v2.7/ak8so-quick-start/)).

El operador concilia continuamente esta definición con Atlas, asegurando que sus recursos reales siempre coincidan con su estado deseado. Esto habilita flujos de trabajo GitOps, donde las definiciones de su infraestructura residen junto con el código de su aplicación en el control de versiones.

Una de las mayores ventajas es la capacidad de automatizar flujos de trabajo complejos de múltiples pasos que de otro modo requerirían intervención manual o scripts personalizados utilizando la API de Atlas.

Con la versión 2.0, el operador de Kubernetes de Atlas introdujo una salvaguarda importante: la protección contra eliminación (*deletion protection*). Esta función cambia la forma en que el operador gestiona las eliminaciones de recursos.

Anteriormente, si eliminaba un recurso personalizado en Kubernetes (quizás accidentalmente), el operador eliminaba obedientemente el recurso correspondiente en Atlas. Esto podría provocar pérdidas involuntarias de datos o interrupciones del servicio si alguien no familiarizado con el comportamiento del operador eliminaba un recurso.

Ahora, de forma predeterminada, cuando elimina un recurso personalizado en Kubernetes, ocurre lo siguiente:

- El recurso en Atlas permanece intacto.
- El operador simplemente deja de administrar ese recurso.
- Puede continuar administrándolo directamente a través de la interfaz de usuario o la API de Atlas.

Este comportamiento proporciona una red de seguridad. Por ejemplo, si alguien elimina accidentalmente un recurso `AtlasProject` de Kubernetes, su proyecto real de Atlas (junto con todos sus clústeres y datos) permanece intacto.

#### Primeros pasos con el operador de Kubernetes de Atlas

Aquí están los pasos para comenzar con el operador de Kubernetes de Atlas:

1. Instale el operador en su clúster de Kubernetes siguiendo la documentación oficial o los gráficos de Helm.
2. Configure el acceso a Atlas creando las claves de API y los secretos de Kubernetes necesarios.
3. Defina sus recursos utilizando las CRD proporcionadas por el operador.
4. Aplique sus configuraciones y deje que el operador las sincronice con Atlas.

A medida que Kubernetes continúa siendo la plataforma preferida para aplicaciones modernas, el operador de Kubernetes de Atlas desempeñará un papel cada vez más vital en la gestión de bases de datos. MongoDB continúa mejorando el operador con nuevas capacidades, convirtiéndolo en una piedra angular de las implementaciones de MongoDB Atlas nativas de la nube.

Al adoptar el patrón de operador y la gestión declarativa para sus recursos de Atlas, no solo simplifica las operaciones de la base de datos; se alinea con las mejores prácticas de la industria para la gestión de infraestructura y posiciona a su organización para operaciones de bases de datos más eficientes, escalables y confiables.

#### Exploración de datos a través del conector Atlas BI

El conector de MongoDB Atlas para Power BI proporciona otra forma de interactuar con sus datos de MongoDB. Esta solución certificada por Microsoft cierra la brecha entre los equipos de desarrollo y análisis, permitiendo a los usuarios de Power BI transformar, analizar y compartir de forma nativa paneles que incorporan datos de MongoDB Atlas en vivo. Con este conector, puede aprovechar el modelo de documentos de MongoDB mientras trabaja dentro de las interfaces familiares de Power BI que los analistas conocen y aprecian.

El modo DirectQuery es una mejora significativa para el conector de Power BI de MongoDB Atlas. DirectQuery proporciona una conexión directa a su base de datos de MongoDB Atlas, lo que permite a Power BI consultar datos en tiempo real en lugar de importarlos y almacenarlos dentro de Power BI. Esto significa que sus visualizaciones e informes siempre reflejan los datos más actuales en su base de datos de MongoDB Atlas.

#### Cómo funciona DirectQuery con MongoDB Atlas

El conector de Power BI funciona a través de la interfaz SQL de Atlas de MongoDB, impulsada por Atlas Data Federation. Así es como funciona el proceso:

1. Habilite la interfaz SQL de Atlas desde la consola de Atlas.
2. Utilice el punto de enlace/URL de SQL proporcionado en el cuadro de diálogo de conexión SQL de MongoDB Atlas dentro de Power BI Desktop.
3. Seleccione DirectQuery como su modo de conectividad (en lugar de Importar).
4. Aproveche el *query folding* con Power Query para optimizar la recuperación y transformación de datos.
5. Cree, guarde y publique informes en la aplicación en línea de Power BI.

Al utilizar DirectQuery con MongoDB Atlas, podemos crear informes que siempre muestren los datos más actuales sin necesidad de importar o actualizar conjuntos de datos. Este enfoque nos brinda acceso directo a nuestros datos de MongoDB mientras aprovechamos las capacidades de visualización de Power BI. A continuación, echemos un vistazo más de cerca a los beneficios de DirectQuery para Atlas.

#### Beneficios de DirectQuery para MongoDB Atlas

DirectQuery se desarrolló para abordar la creciente necesidad de acceso en tiempo real a datos operativos dentro de plataformas de análisis, especialmente a medida que las empresas dependen cada vez más de información actualizada para la toma de decisiones. Los métodos tradicionales, como la importación de grandes conjuntos de datos en herramientas de análisis, a menudo introducían desafíos de rendimiento, complejidades de almacenamiento y retrasos derivados de datos obsoletos. DirectQuery mitiga estos problemas permitiendo una conexión en vivo a su base de datos, garantizando que los análisis siempre reflejen el estado más reciente de sus datos.

Esta solución es particularmente beneficiosa en los siguientes casos:

- Necesita analizar grandes conjuntos de datos donde la importación no sería práctica
- La información actualizada es importante para la toma de decisiones
- Desea evitar problemas de rendimiento derivados de importaciones repetitivas de datos
- Necesita simplificar las complejidades de almacenamiento para grandes conjuntos de datos

Con la compatibilidad con DirectQuery, puede conectarse fácilmente a datos de MongoDB Atlas en tiempo real mientras conserva la flexibilidad de utilizar el modo Importar cuando sea necesario un modelado detallado para volúmenes de datos manejables. Esta mejora ayuda a las empresas a utilizar todas las capacidades de MongoDB Atlas dentro de Microsoft Power BI, permitiendo una integración fluida entre las bases de datos operativas y las plataformas de análisis para impulsar decisiones informadas.

#### Atlas Charts

Atlas Charts es la herramienta de visualización de datos nativa de MongoDB que le permite crear representaciones visuales atractivas directamente a partir de sus datos de Atlas. En lugar de exportar sus datos a herramientas de visualización externas, puede transformar sin problemas sus colecciones de MongoDB en gráficos y paneles informativos, todo dentro del ecosistema de Atlas.

Esto es lo que hace que Atlas Charts sea particularmente valioso:

- **Integración perfecta con MongoDB Atlas:** Conecte Charts directamente a sus proyectos de Atlas y visualice los datos del clúster con una configuración mínima
- **Manejo de datos de documentos:** A diferencia de las herramientas de BI tradicionales, Charts comprende de forma nativa el modelo de documentos de MongoDB, incluidos los objetos y matrices incrustados
- **Funcionalidad de agregación integrada:** Procese los datos de su colección utilizando diversas métricas y cálculos sin escribir consultas complejas
- **Diversos tipos de gráficos:** Cree visualizaciones que van desde gráficos de barras y diagramas de dispersión hasta gráficos geoespaciales, cada uno diseñado para mostrar diferentes aspectos de sus datos

Charts organiza sus visualizaciones a través de conceptos clave:

- **Fuentes de datos (*Data sources*):** Colecciones de MongoDB que contienen los datos que desea visualizar
- **Gráficos (*Charts*):** Visualizaciones individuales que se asignan a una única fuente de datos
- **Paneles (*Dashboards*):** Colecciones de uno o más gráficos que proporcionan una vista completa de sus datos

> **Figura 8.2:** Panel de control de Atlas Charts que muestra visualizaciones de muestra sobre el volumen de ventas y las tendencias de ingresos

#### Compartir de forma segura con paneles protegidos por contraseña

¿Cómo comparte información valiosa con partes interesadas externas a su organización manteniendo al mismo tiempo la seguridad de los datos? Atlas Charts ahora ofrece paneles protegidos con contraseña como una función de seguridad mejorada para paneles compartidos.

Esta característica agrega una capa adicional de protección, garantizando que solo los usuarios autorizados con la contraseña correcta puedan acceder a sus conocimientos visuales. La implementación de esta medida de seguridad es sencilla.

Para implementar esta medida de seguridad, siga estos pasos:

1. Marque la casilla para proteger su enlace público con un código de acceso al compartir.
2. Se generará automáticamente un código de acceso (puede regenerarlo según sea necesario).
3. Comparta tanto el enlace del panel como la contraseña con los destinatarios previstos.
4. Se solicitará a los espectadores que ingresen la contraseña antes de acceder a su panel.

> **Figura 8.3:** Panel de Atlas Charts con opciones para configurar sus gráficos

Esta característica cierra la brecha entre accesibilidad y seguridad, haciendo más fácil compartir visualizaciones de datos más allá de su organización de Atlas mientras mantiene el control sobre quién las ve.

#### Mejora de gráficos de tablas con hipervínculos

¿Cómo puede hacer que sus gráficos de tablas sean más interactivos y navegables? Atlas Charts ahora ofrece personalización de hipervínculos, una de las funciones más solicitadas por los usuarios. Esta potente mejora transforma los datos estáticos en contenido procesable, creando una experiencia de usuario más dinámica.

Con la personalización de hipervínculos, puede hacer lo siguiente:

- Formatear datos de columnas como enlaces interactivos utilizando los protocolos `http`, `https`, `mailto` o `tel`
- Construir enlaces dinámicamente utilizando valores de campo de sus documentos
- Crear conexiones entre sus visualizaciones y recursos externos
- Proporcionar acceso directo a información relacionada sin salir de su panel

Veamos un ejemplo práctico. Imagine que tiene una colección de películas en su base de datos de MongoDB. Con Atlas Charts, puede crear una tabla que muestre títulos de películas, géneros y calificaciones. Con la personalización de hipervínculos, puede convertir cada título de película en un enlace en el que se puede hacer clic que dirija a los usuarios a su página correspondiente de IMDb.

#### Creación de gráficos con el modo de lenguaje natural (*Natural Language Mode*)

¿Qué pasaría si pudiera crear visualizaciones de datos sofisticadas simplemente haciendo preguntas en lenguaje natural? ¡Con el Modo de Lenguaje Natural en Atlas Charts, ahora puede!

El Modo de Lenguaje Natural reduce las barreras técnicas al permitirle hacer lo siguiente:

- Generar gráficos haciendo preguntas como: "Muéstrame el rendimiento de ventas por país y producto para el cuarto trimestre del año fiscal 2023"
- Crear visualizaciones sin conocimientos especializados de herramientas de BI
- Explorar sus datos a través de conversaciones en lugar de configuraciones complejas

Atlas Charts transforma la forma en que interactúa con sus datos de MongoDB proporcionando herramientas de visualización intuitivas que se integran perfectamente con su entorno de Atlas existente. Ya sea que esté creando paneles para monitorear métricas comerciales, compartiendo conocimientos de forma segura mediante protección por contraseña, mejorando tablas con hipervínculos interactivos o explorando datos a través de consultas en lenguaje natural, Charts le permite extraer el máximo valor de sus datos de MongoDB.

#### Herramientas familiares

Además, puede utilizar herramientas con las que quizás ya esté familiarizado si ha estado administrando sus propios despliegues de MongoDB, como las siguientes:

- **MongoDB Shell (`mongosh`):** Una herramienta familiar para el análisis exploratorio de datos y la prueba de consultas
- **MongoDB Compass:** Una interfaz gráfica que ofrece una vista visual para la gestión de bases de datos

A lo largo de este capítulo, utilizaremos Atlas CLI y `mongosh` para muchos ejemplos. Sin embargo, todo lo discutido se puede adaptar a la herramienta que prefiera.

---

### Sección 5: Dimensionamiento de un clúster de Atlas

Un clúster de Atlas es la unidad de despliegue fundamental dentro de este servicio. Se puede configurar como un conjunto de réplicas o como un clúster fragmentado. Estos clústeres operan en los principales proveedores de nube, incluidos AWS, Azure y Google Cloud, brindándole flexibilidad en el lugar donde residen sus datos y eliminando la carga de la gestión de la infraestructura.

Los clústeres de Atlas incluyen numerosas capacidades que benefician a los equipos de desarrollo. Obtendrá copias de seguridad automatizadas con recuperación en un momento dado, herramientas de monitoreo integradas y funciones de seguridad integrales, que incluyen aislamiento de red, cifrado y control de acceso basado en roles (RBAC). El servicio gestiona tareas rutinarias de mantenimiento de bases de datos, como actualizaciones y parches, de forma automática. Esto permite a su equipo concentrarse en el desarrollo de aplicaciones en lugar de dedicar tiempo a tareas de administración de bases de datos.

Dimensionar eficazmente su clúster de MongoDB Atlas es un paso importante para cualquier proyecto. Si bien no existe una configuración universal que se adapte a todas las necesidades, esta sección le proporcionará un punto de partida y le guiará a través del proceso de toma de decisiones en función de sus requisitos específicos. Es una buena idea documentar la lógica detrás de sus elecciones de tamaño, ya que este registro será invaluable para futuros esfuerzos de escalado y optimización.

Al planificar su despliegue de Atlas, deberá seleccionar entre varios niveles de clúster disponibles. Estos niveles difieren en sus asignaciones de recursos, lo que afecta directamente el rendimiento, la capacidad y el costo. Examinemos qué recursos proporciona cada nivel para ayudarle a hacer coincidir las necesidades de su aplicación con la configuración adecuada.

#### Almacenamiento, RAM, CPU, IOPS y conexiones por nivel

Los clústeres de Atlas vienen en varios niveles, desde pequeñas instancias de desarrollo (M0, M2 y M5) hasta grandes despliegues de nivel empresarial (M300+). Cada nivel proporciona asignaciones específicas en cinco aspectos clave:

- **Almacenamiento (*Storage*):** La cantidad de espacio en disco disponible para sus datos e índices
- **RAM:** Memoria disponible para la caché y las operaciones de WiredTiger
- **CPU:** Potencia de procesamiento para la ejecución de consultas y operaciones de bases de datos
- **IOPS:** Operaciones de entrada/salida por segundo, que es una medida del rendimiento del disco
- **Conexiones (*Connections*):** Número máximo de conexiones simultáneas de clientes

Por ejemplo, un clúster M30 proporciona 8 GB de RAM por nodo, mientras que un M80 ofrece 64 GB de RAM por nodo, un aumento de 8 veces que puede cambiar drásticamente la forma en que su conjunto de trabajo opera en la memoria.

A medida que avanza de niveles inferiores a superiores, estos recursos aumentan de una manera no del todo lineal. Algunos recursos pueden aumentar exponencialmente entre ciertos saltos de nivel, mientras que otros crecen más gradualmente.

Una de las características de Atlas es la capacidad de aprovisionar almacenamiento independientemente de otros recursos. Para los clústeres M10+, puede seleccionar la capacidad de almacenamiento que coincida con su volumen de datos sin cambiar necesariamente otras especificaciones.

Al seleccionar el almacenamiento, considere no solo el tamaño actual de sus datos sino también su crecimiento proyectado. ¿Cuántos datos espera acumular durante los próximos 6 a 12 meses? Incorporar cierto margen de maniobra evita ajustes frecuentes y al mismo tiempo mantiene los costos razonables.

#### Requisitos de almacenamiento

Predecir con precisión cuánto almacenamiento requerirá su despliegue de MongoDB puede resultar un desafío, especialmente para aplicaciones nuevas. Sin embargo, podemos hacer estimaciones razonables utilizando varias métricas clave que le acercarán lo suficiente para la planificación inicial.

Desglosemos las métricas específicas que deberá considerar al calcular sus requisitos totales de almacenamiento.

#### Cálculo del tamaño del documento

La base de su estimación de almacenamiento comienza con la comprensión del tamaño promedio de sus documentos. Si ya tiene una colección con datos, puede utilizar el siguiente comando en el Shell de MongoDB:

```javascript
db.collection.stats().avgObjSize  // Returns average object size in bytes
```

Si sus datos ya existen en una base de datos de MongoDB, puede obtener la mayor parte de la siguiente información ejecutando `db.stats()` y `db.col.stats()` en el Shell de MongoDB.

Para nuevas aplicaciones, puede estimar esto creando documentos de muestra que coincidan con su modelo de datos esperado y midiendo su tamaño BSON en el Shell de MongoDB:

```javascript
let doc = {'_id': ObjectId('600e8fcf36b07f77b6bc8ecf'), 'name': 'Mina'};
print(bsonsize(doc));
```

#### Cálculo del tamaño de la colección

Una vez que conoce el tamaño promedio de sus documentos, puede estimar el tamaño de una colección:

```text
Collection Size = Average Document Size × Number of Documents
```

Por ejemplo, si su documento promedio es de 2 KB y espera 1 millón de documentos, tendría lo siguiente:

```text
2KB × 1,000,000 = 2GB (raw data size)
```

#### Tamaño total de las colecciones

Es probable que su base de datos conste de varias colecciones. Sume todos los tamaños de las colecciones para obtener sus necesidades de almacenamiento de datos sin procesar:

```text
Total Raw Data Size = Sum of All Collection Sizes
```

#### Tamaño de los índices

Los índices consumen espacio adicional en disco además del espacio utilizado por sus datos sin procesar. Un enfoque para estimar el tamaño del índice es calcularlo como una fracción o porcentaje del tamaño total de sus datos:

```text
Index Size = Raw Data Size × (0.2 to 0.5)
```

Para este ejemplo, estimar los índices en un 30% del tamaño de sus datos proporciona un punto de partida razonable:

```text
Index Size = 2GB × 0.3 = 600MB
```

Esto puede variar significativamente según la cantidad y los tipos de índices que cree. Algunos índices ocupan más espacio que otros. Es importante recordar que esta es una estimación muy aproximada.

#### Compresión de WiredTiger

El motor de almacenamiento WiredTiger de MongoDB proporciona ahorros sustanciales de almacenamiento mediante compresión. De forma predeterminada, WiredTiger aplica lo siguiente:

- Compresión de datos de colección
- Compresión de índices

El nivel de compresión que se puede lograr con MongoDB es muy variable y depende de numerosos factores, lo que dificulta predecir la cantidad exacta. Para una estimación prudente, se puede suponer una reducción del 20% en el tamaño. En condiciones óptimas, las tasas de compresión podrían alcanzar hasta el 80%.

Esto significa que el uso real del disco puede ser significativamente menor de lo que sugieren los cálculos de datos sin procesar:

```text
Compressed Data Size = Raw Data Size ÷ Compression Ratio
Compressed Index Size = Index Size ÷ Index Compression Ratio
```

Para nuestro ejemplo, esto se ve de la siguiente manera:

```text
Compressed Data Size = 2GB ÷ 4 = 500MB
Compressed Index Size = 600MB ÷ 2 = 300MB
```

#### Tamaño del búfer

MongoDB necesita espacio de trabajo más allá del tamaño de sus datos comprimidos. Una buena práctica es agregar un búfer:

```text
Total Storage Needed = (Compressed Data Size + Compressed Index Size) × 1.5
```

Utilizando nuestro ejemplo, esto se ve de la siguiente manera:

```text
Total Storage Needed = (500MB + 300MB) × 1.5 = 1.2GB
```

Este búfer del 50% satisface las necesidades del motor de almacenamiento WiredTiger y proporciona espacio para operaciones temporales.

#### Proyecciones de crecimiento futuro

¿Cómo crecerán sus datos con el tiempo? Considere estos enfoques:

- **Proyección lineal:** Calcule la tasa de crecimiento mensual basada en datos históricos
- **Ajustes estacionales:** Tenga en cuenta los períodos de mayor actividad si su aplicación tiene un uso cíclico
- **Proyección basada en usuarios:** Estime el almacenamiento por usuario y multiplíquelo por el crecimiento proyectado de usuarios

Por ejemplo, si actualmente está creciendo a un ritmo de 5 GB por mes, tendría lo siguiente:

```text
Year 1 Additional Storage = 5GB × 12 = 60GB
```

Si espera que el crecimiento de usuarios acelere esto en un 20% en el segundo año, tendría lo siguiente:

```text
Year 2 Additional Storage = 60GB × 1.2 = 72GB
```

#### Consideraciones adicionales

Si bien los cálculos anteriores proporcionan una base sólida, varios otros factores pueden influir en sus necesidades totales de almacenamiento:

- **Políticas de retención de datos:** ¿Archivará o eliminará datos antiguos? ¿Con qué frecuencia?
- **Variaciones de la carga de trabajo:** Los entornos de desarrollo, pruebas y producción pueden tener diferentes patrones de almacenamiento.
- **Estrategia de fragmentación:** La forma en que distribuye los datos entre los fragmentos afecta la distribución del almacenamiento.

Es beneficioso documentar el razonamiento detrás de sus opciones de estimación. No se preocupe excesivamente por estimaciones precisas, ya que Atlas ofrece opciones de escalado flexibles, que se analizarán más adelante.

#### Requisitos de RAM

Comprender su conjunto de trabajo (*working set*) es fundamental para dimensionar adecuadamente los requisitos de memoria de su clúster de MongoDB Atlas. El conjunto de trabajo representa el subconjunto de sus datos e índices a los que su aplicación accede activamente durante las operaciones regulares. Cuando su conjunto de trabajo cabe dentro de la RAM disponible, MongoDB puede atender la mayoría de las consultas desde la memoria, lo que resulta en tiempos de respuesta significativamente más rápidos en comparación con las consultas que dependen del acceso al disco.

Su conjunto de trabajo normalmente consta de dos componentes principales:

- **Documentos a los que se accede con frecuencia:** Estos son los datos que su aplicación lee o escribe periódicamente durante las operaciones normales.
- **Índices:** Es importante mantener los índices en la memoria porque afectan directamente la velocidad de ejecución de las consultas.

#### Asignación de memoria de WiredTiger

MongoDB Atlas utiliza el motor de almacenamiento WiredTiger, que dedica una parte de la RAM disponible a la caché y a las operaciones intensivas de memoria. La asignación de caché depende del nivel del clúster:

- **Clústeres M40 o superiores:** WiredTiger dedica el 50% de la RAM física a su caché de forma predeterminada. La memoria restante se reserva para otras operaciones, como ordenaciones, agregaciones, cálculos, el sistema operativo subyacente y otros servicios que se ejecutan en el host.
- **Clústeres más pequeños (M30 o inferiores):** WiredTiger asigna el 25% de la RAM física a su caché. Esta diferencia en el porcentaje de asignación es importante al estimar la disponibilidad efectiva de memoria para su conjunto de trabajo.

Para calcular la RAM disponible para su conjunto de trabajo, multiplique la RAM total por 0.5 (para niveles M40+) o 0.25 (para niveles M30 e inferiores), según el nivel de su clúster.

Si la caché no puede acomodar completamente el conjunto de trabajo, WiredTiger expulsará las páginas utilizadas con menos frecuencia para liberar memoria. Esto garantiza que las consultas sigan funcionando, pero puede depender de una E/S de disco más lenta bajo cargas de trabajo pesadas.

#### Ejemplo de cálculo de RAM

Para calcular los requisitos de RAM, debe hacer lo siguiente:

1. Comience con el tamaño total de su índice (15 GB en nuestro ejemplo).
2. Agregue datos a los que se accede con frecuencia (200 GB × 20% = 40 GB). Tenga en cuenta que el 20% es una suposición y puede variar mucho según las aplicaciones.
3. Conjunto de trabajo = 15 GB + 40 GB = 55 GB.
4. Tenga en cuenta la asignación de WiredTiger (55 GB ÷ 0.5 = 110 GB para M40+).

Para una aplicación de catálogo de productos con 200 GB de datos totales, 15 GB de índices y 20% de datos frecuentes (*hot data*), necesitaría al menos 110 GB de RAM en su clúster. Esto sugiere que un nivel M80 (128 GB de RAM) sería apropiado.

#### Análisis de rendimiento e IOPS

Las IOPS representan la capacidad del sistema de almacenamiento de su base de datos para operaciones de lectura y escritura. Para elegir la configuración óptima del clúster de Atlas, es útil comprender los requisitos de IOPS de su aplicación.

Diferentes operaciones consumen distintas cantidades de IOPS. Las operaciones de lectura que acceden a datos que ya están en la memoria consumen un mínimo de IOPS, haciendo un uso eficiente de su conjunto de trabajo. Por el contrario, cualquier operación de escritura, incluidas inserciones, actualizaciones y eliminaciones, requiere significativamente más IOPS porque modifica tanto los datos como los índices asociados.

Un enfoque valioso es perfilar su aplicación registrando todas las operaciones de base de datos que realiza durante el uso típico. Esto ayuda a determinar si su carga de trabajo es intensiva en lectura, intensiva en escritura o equilibrada. Por ejemplo, una plataforma de análisis puede ser intensiva en lectura con escrituras por lotes ocasionales, mientras que un sistema de registro de eventos sería predominantemente intensivo en escritura.

#### Aprovisionamiento y escalado de IOPS

MongoDB Atlas vincula automáticamente los umbrales de IOPS a la capacidad de almacenamiento aprovisionada. Más almacenamiento significa un mayor valor base de IOPS:

- Los clústeres configurados con 1 TB o más de almacenamiento garantizan una relación de 3:1 de IOPS a almacenamiento (por ejemplo, 3,000 IOPS con 1 TB de almacenamiento o 16,000 IOPS con 16 TB).
- Los clústeres más pequeños tienen líneas base específicas de cada nivel, a partir de un mínimo de 3,000 IOPS para clústeres M30 y superiores.

Este escalado automático garantiza un rendimiento predecible y evita cuellos de botella. La capacidad de almacenamiento también afecta el rendimiento de ráfaga (*burst performance*), ya que los clústeres con discos más grandes acumulan más créditos de ráfaga para manejar picos temporales de actividad.

#### IOPS aprovisionadas para cargas de trabajo con uso intensivo de escritura

Para clústeres que se ejecutan en AWS, los niveles M30 y superiores ofrecen la capacidad de aprovisionar IOPS adicionales independientemente del tamaño del almacenamiento. Esta característica es especialmente útil para aplicaciones con uso intensivo de escritura que requieren un alto rendimiento de IOPS pero no necesitan grandes volúmenes de almacenamiento.

Para los clústeres alojados en Azure (M40 y superiores) y en regiones seleccionadas, los usuarios pueden incluso escalar IOPS de forma independiente y personalizar el rendimiento sin aprovisionar almacenamiento en exceso. Esta flexibilidad permite ajustar los clústeres a los requisitos de carga de trabajo específicos, ya sean de lectura intensiva o de escritura intensiva.

#### Uso de herramientas de monitoreo para escalar eficazmente

La estimación de IOPS no necesita ser perfecta desde el principio. MongoDB Atlas proporciona herramientas para monitorear el uso real de IOPS e identificar cuellos de botella en el rendimiento:

- Utilice el gráfico de **Disk IOPS** en el panel de métricas de Atlas para realizar un seguimiento del uso durante los períodos de mayor actividad y de inactividad de su carga de trabajo.
- Observe métricas como **Disk Latency** y **Disk Queue Depth** para determinar si su carga de trabajo está sobrecargando su configuración actual. Esta información le ayuda a evaluar si su clúster puede soportar las cargas de trabajo proyectadas o requiere escalado.

Atlas admite el autoescalado y el ajuste manual de IOPS mediante aumentos de almacenamiento o capacidad aprovisionada (M30+ para AWS, M40+ para Azure). Esto le permite comenzar con una estimación, monitorear el uso en el mundo real y escalar dinámicamente para mantener el rendimiento de la aplicación mientras optimiza los costos.

Este enfoque, comenzando con una estimación razonable, monitoreando el uso real y escalando según sea necesario, le permite ajustar las características de rendimiento de su clúster basándose en datos operativos reales en lugar de proyecciones teóricas.

#### Rendimiento de red y conexiones

Los clústeres de MongoDB Atlas manejan dos tipos principales de tráfico de red: comunicación de cliente a servidor (cómo se conectan sus aplicaciones a su base de datos) y comunicación dentro del clúster (coordinación entre nodos para la replicación y la coherencia operativa). Ambos afectan el rendimiento, y los clústeres multirregionales requieren especial atención debido a una mayor latencia y un mayor consumo de ancho de banda.

La capacidad de la red afecta directamente el rendimiento de su base de datos. Un ancho de banda suficiente garantiza la entrega eficiente de grandes conjuntos de resultados y reduce la latencia de las operaciones. Durante los picos de tráfico, una capacidad limitada puede provocar conexiones caídas. Un ancho de banda inadecuado puede provocar retrasos en la replicación, especialmente en clústeres distribuidos geográficamente.

#### Cálculo del ancho de banda

Estime sus requisitos de ancho de banda utilizando esta sencilla fórmula:

```text
Bandwidth = Average Document Size × Peak Documents Per Second
```

Por ejemplo, si su documento promedio es de 10 KB y procesa 100,000 documentos por segundo, necesitará aproximadamente 1 GB/seg de ancho de banda.

El uso real varía según la configuración de preferencias de lectura, las opciones de replicación y la estrategia de particionamiento. La distribución de operaciones de lectura entre nodos secundarios puede optimizar el uso del ancho de banda y reducir la carga del nodo primario.

#### Gestión de conexiones

MongoDB Atlas impone límites de conexión basados en el nivel de su clúster, aplicados por nodo en lugar de por clúster. Los niveles gratuitos y compartidos (M0, M2 y M5) admiten de 500 a 3,000 conexiones por nodo. Los niveles dedicados escalan desde 6,000 conexiones (M10) hasta 128,000 conexiones (M200+). Seleccione un nivel que se adapte a sus requisitos máximos con margen de crecimiento.

#### Agrupación de conexiones (*Connection pooling*)

La agrupación de conexiones implica mantener conexiones reutilizables que las aplicaciones pueden tomar prestadas y devolver, en lugar de establecer repetidamente nuevas conexiones. Esta estrategia reduce la sobrecarga, optimiza el uso de recursos, reduce la latencia y disminuye la carga del servidor. Para una agrupación de conexiones eficaz, es importante dimensionar el grupo adecuadamente, configurar tiempos de espera para evitar conexiones bloqueadas e implementar monitoreo para detectar cualquier problema de conexión. La mayoría de los controladores de MongoDB ofrecen soporte integrado para la agrupación de conexiones con opciones configurables.

Al estimar las necesidades de ancho de banda, comprender los límites de conexión, implementar una agrupación de conexiones eficiente y distribuir estratégicamente el tráfico, puede optimizar el rendimiento de la red de MongoDB Atlas. Estas prácticas son importantes para construir una base de datos receptiva que pueda escalar eficazmente con el crecimiento de su aplicación y al mismo tiempo prevenir de forma proactiva cuellos de botella en el rendimiento.

#### Selección de región

Seleccionar la región geográfica adecuada para su clúster de Atlas implica algo más que consideraciones técnicas. Los requisitos de soberanía de datos pueden exigir legalmente dónde se pueden almacenar y procesar sus datos. Diferentes países y regiones tienen regulaciones específicas que rigen el almacenamiento de datos, particularmente para información personal o confidencial.

Al evaluar su panorama regulatorio, considere marcos como GDPR (Europa), HIPAA (atención médica de EE. UU.) o CCPA (California). Estas regulaciones pueden requerir que los datos permanezcan dentro de límites geográficos específicos. Atlas ofrece regiones en los principales proveedores de nube y continentes para ayudarle a satisfacer estos requisitos manteniendo el rendimiento.

Para aplicaciones multinacionales, es posible que deba implementar una estrategia de segregación de datos, donde diferentes clústeres en diferentes regiones manejen datos de acuerdo con las regulaciones aplicables. La estructura de proyectos y organizaciones de Atlas puede ayudar a gestionar esta complejidad de forma eficaz.

#### Optimización de latencia

Más allá del cumplimiento, la proximidad física entre los servidores de aplicaciones y la base de datos afecta el rendimiento. La latencia de la red aumenta con la distancia e incluso los milisegundos importan para las aplicaciones con alta capacidad de respuesta. Como regla general, ubique su clúster de Atlas en la misma región que sus servidores de aplicaciones siempre que sea posible.

Para aplicaciones distribuidas globalmente, considere estos enfoques:

- **Selección de región primaria:** Coloque su clúster en la región con la mayor concentración de usuarios o las operaciones más críticas.
- **Estrategias multirregionales:** Para despliegues avanzados, los clústeres globales de Atlas (*Global Clusters*) proporcionan fragmentación con reconocimiento de ubicación, enrutando automáticamente los datos a la zona geográfica adecuada. Esto permite que las escrituras se realicen en la región más cercana a sus usuarios, minimizando la latencia.
- **Patrones de lectura local (*Read-local patterns*):** Configure su aplicación para leer desde secundarios geográficamente cercanos mientras envía escrituras al primario en otra región. Este enfoque equilibra la coherencia de los datos con el rendimiento de lectura.

Al implementar una estrategia multirregión, tenga en cuenta los costos adicionales y la complejidad que implica. Realice pruebas exhaustivas para asegurarse de que su aplicación maneje adecuadamente las fallas regionales y las particiones de red.

Para la mayoría de las aplicaciones, un buen punto de partida es seleccionar una sola región con el mejor equilibrio entre cumplimiento normativo y proximidad a su base de usuarios. Más adelante podrá evolucionar hacia una arquitectura más distribuida a medida que crezcan las necesidades de su aplicación y madure su comprensión de los patrones de uso reales.

#### Selección final del clúster

Después de analizar sus diversos requisitos, cree una vista consolidada de sus hallazgos. La clave aquí no es sólo recopilar los números, sino documentar cada decisión y suposición tomada a lo largo del camino:

- **Necesidades de almacenamiento:** Documente tanto el tamaño actual como las suposiciones sobre la tasa de crecimiento.
- **Requisitos de RAM:** Registre las estimaciones del conjunto de trabajo y las suposiciones sobre los patrones de acceso.
- **Proyecciones de IOPS:** Anote la relación de lectura/escritura utilizada en los cálculos.
- **Necesidades de red:** Documente las suposiciones de rendimiento máximo y las estimaciones de concurrencia.
- **Restricciones regionales:** Enumere los requisitos de cumplimiento que influyen en las decisiones de ubicación.

Mantener este registro sirve para múltiples propósitos: ayuda a justificar sus elecciones ante las partes interesadas, proporciona contexto para futuras decisiones de escalado y crea conocimiento institucional para los miembros del equipo que puedan heredar el sistema.

Al seleccionar la configuración inicial de su clúster, recuerde que Atlas ofrece una flexibilidad notable; nada está escrito en piedra. Esto debería influir en su enfoque de toma de decisiones:

- **Comience de forma práctica, no perfecta:** Dado que Atlas puede escalar fácilmente hacia arriba o hacia abajo, no necesita encontrar el nivel perfecto de inmediato. Documente su razonamiento para cada elección.
- **Equilibre las necesidades actuales con la flexibilidad:** En lugar de aprovisionar en exceso para necesidades proyectadas a meses de distancia, elija un punto de partida razonable con un camino claro hacia el escalado.
- **Registre las compensaciones entre costo y rendimiento:** Anote qué concesiones se hicieron y el razonamiento comercial detrás de ellas, creando responsabilidad para las decisiones.

Atlas le permite ajustar casi todos los parámetros del clúster (nivel, almacenamiento, IOPS e incluso región) con una interrupción mínima. Esta flexibilidad significa que su selección inicial es un punto de partida, no un compromiso permanente.

Antes de finalizar su despliegue de producción, haga lo siguiente:

- **Documente escenarios de prueba:** Cree un registro claro de las cargas de trabajo utilizadas en las pruebas y su relación con las cargas de producción esperadas.
- **Registre métricas de referencia:** Establezca puntos de referencia de rendimiento que servirán como puntos de comparación después de futuras operaciones de escalado.
- **Pruebe operaciones de escalado:** Practique realmente escalar su clúster hacia arriba y hacia abajo para comprender el proceso y el impacto en su aplicación.

Durante la validación, documente cualquier discrepancia entre el rendimiento esperado y el real. Estas observaciones proporcionan información valiosa para una optimización futura.

La verdadera ventaja de Atlas es que le permite adaptarse a medida que aprende. La selección inicial de su clúster no necesita ser perfecta; sólo necesita ser un punto de partida razonable. A medida que recopila datos de uso del mundo real, puede ajustar fácilmente su configuración para que coincida con los requisitos reales en lugar de con proyecciones teóricas. Este enfoque a menudo da como resultado un mejor rendimiento, menores costos y menos estrés que intentar predecir perfectamente todas sus necesidades por adelantado.

---

### Sección 6: Escalado en Atlas

¿Alguna vez ha experimentado ese momento de emoción mezclado con ansiedad cuando su aplicación de repente gana tracción? Si bien el crecimiento de usuarios es el objetivo de la mayoría de las aplicaciones, a menudo trae consigo un desafío: escalar su base de datos para manejar una mayor carga sin sacrificar el rendimiento o la confiabilidad.

A medida que se expande la base de usuarios de su aplicación, es probable que observe varios indicadores que señalan la necesidad de escalar:

- Tiempos de respuesta de consultas más lentos durante períodos de uso máximo
- Aumento de la utilización de CPU y memoria en los nodos de su base de datos
- Requisitos de almacenamiento que crecen más rápido de lo previsto
- Expansión geográfica de su base de usuarios, creando problemas de latencia

MongoDB Atlas aborda estos desafíos de escalado mediante un enfoque integral y flexible que combina estrategias de escalado tanto vertical como horizontal. Pero ¿qué significa esto exactamente para su aplicación?

El **escalado vertical** implica aumentar los recursos disponibles para los nodos de su base de datos existentes, brindando a su infraestructura actual más potencia para manejar cargas de trabajo. Piense en ello como actualizar de un automóvil compacto a un automóvil deportivo sin cambiar de ruta.

El **escalado horizontal**, por otro lado, distribuye sus datos entre varias máquinas, lo que permite que su base de datos maneje conjuntos de datos más grandes y un mayor rendimiento agregando más nodos en lugar de hacer que los existentes sean más potentes. Esto es como agregar más carriles a una carretera en lugar de intentar que los automóviles vayan más rápido.

Atlas proporciona herramientas y capacidades para ambos enfoques, lo que le brinda la flexibilidad de elegir la estrategia adecuada, o combinación de estrategias, para sus necesidades específicas.

#### ¿Por qué es tan importante elegir la estrategia de escalado adecuada?

La ruta de escalado que seleccione afecta directamente varios aspectos de su aplicación:

- **Eficiencia de costos:** Diferentes enfoques de escalado tienen diferentes estructuras de costos. Elegir sabiamente evita pagar de más por recursos no utilizados.
- **Características de rendimiento:** Algunas cargas de trabajo se benefician más del escalado vertical, mientras que otras experimentan mayores mejoras con enfoques horizontales.
- **Complejidad operativa:** El escalado horizontal introduce una complejidad adicional que podría requerir una mayor sobrecarga de gestión.
- **Potencial de crecimiento futuro:** Las decisiones iniciales de escalado pueden facilitar o restringir opciones futuras a medida que la aplicación continúa creciendo.

A lo largo de esta sección, exploraremos las capacidades de escalado de Atlas, brindándole el conocimiento necesario para tomar decisiones informadas sobre el escalado de sus despliegues de MongoDB. Comenzaremos examinando las opciones de escalado vertical, incluidas las potentes funciones de escalado automático de Atlas, antes de pasar a enfoques horizontales como la fragmentación (*sharding*) y los clústeres globales (*Global Clusters*).

Al comprender la gama completa de opciones de escalado disponibles en Atlas, estará equipado para crear una estrategia personalizada que se alinee con los requisitos únicos y la trayectoria de crecimiento de su aplicación.

#### Escalado vertical en Atlas

El escalado vertical, a menudo llamado "escalado hacia arriba" (*scaling up*), es el proceso de agregar más recursos a los nodos de su base de datos existentes. En Atlas, esto se traduce en aumentar la capacidad de CPU, memoria y almacenamiento de su clúster sin cambiar su arquitectura básica.

Piense en el escalado vertical como actualizar su computadora en lugar de comprar computadoras adicionales. Está haciendo que lo que ya tiene sea más potente, lo que le permite manejar mayores cargas de trabajo sin cambios arquitectónicos.

El escalado vertical destaca en varios escenarios comunes:

- **Cargas de trabajo moderadas pero crecientes:** Cuando su aplicación experimenta un crecimiento constante que aún no justifica la complejidad de la fragmentación.
- **Operaciones con uso intensivo de memoria:** Aplicaciones que realizan agregaciones complejas o mantienen grandes conjuntos de trabajo en la memoria.
- **Requisitos de simplicidad:** Cuando la simplicidad operativa es una prioridad sobre la escala absoluta.
- **Entornos de desarrollo o pruebas:** Donde necesita ajustar rápidamente los recursos sin rediseñar su modelo de datos.

Por ejemplo, una plataforma de comercio electrónico que experimenta un crecimiento mensual del 30% podría comenzar con un clúster M30 y luego escalar verticalmente a M40 y M50 a medida que aumentan los volúmenes de transacciones, antes de considerar eventualmente la fragmentación.

#### Compensaciones y limitaciones

Si bien el escalado vertical ofrece simplicidad, conlleva consideraciones importantes:

- **Límites superiores:** Los proveedores de la nube tienen tamaños máximos de instancia, lo que eventualmente limita el escalado vertical.
- **Curva de rentabilidad:** Los niveles superiores tienen una relación precio-rendimiento menos favorable; duplicar los recursos puede más que duplicar los costos en algunos casos.

El escalado vertical funciona mejor como parte de una estrategia integral. Si bien ofrece ganancias rápidas para aplicaciones en crecimiento, la mayoría de los sistemas exitosos eventualmente combinan el escalado vertical con la distribución horizontal a medida que alcanzan una escala mayor.

#### Opciones de implementación del escalado vertical

Cuando esté listo para escalar verticalmente su clúster de Atlas, tiene varias opciones de implementación que se pueden adaptar a los requisitos de su carga de trabajo específica. Exploremos algunos de los enfoques que podríamos adoptar.

#### Aprovisionamiento de hardware más potente

El enfoque de escalado vertical más sencillo es actualizar el nivel de su clúster a instancias más potentes.

Al actualizar el nivel de su clúster, Atlas realiza un proceso de actualización continua (*rolling upgrade*) que mantiene la disponibilidad durante toda la transición. Su nodo primario permanece accesible mientras que los nodos secundarios se actualizan uno por uno.

#### Implementación de nodos adicionales en conjuntos de réplicas existentes

Otra estrategia de escalado vertical es agregar más nodos a su conjunto de réplicas existente. Este enfoque mejora lo siguiente:

- **Escalabilidad de lectura:** Más nodos secundarios distribuyen las operaciones de lectura
- **Tolerancia a fallos:** Mayor resiliencia contra fallas de nodos
- **Distribución geográfica:** Los nodos en diferentes regiones reducen la latencia

Agregar nodos no aumenta la capacidad de su nodo primario, pero descarga las operaciones de lectura y mejora la disponibilidad. Por ejemplo, una aplicación de servicios financieros podría agregar nodos de solo lectura en múltiples regiones para respaldar las necesidades de informes locales manteniendo centralizadas las operaciones de escritura.

#### Descarga de operaciones de lectura y cargas de trabajo de análisis

Para cargas de trabajo con operaciones de lectura intensivas y requisitos analíticos, Atlas ofrece tipos de nodos especializados:

- **Nodos de análisis (*Analytics nodes*):** Nodos secundarios optimizados para consultas analíticas
- **Nodos de solo lectura (*Read-only nodes*):** Nodos secundarios no elegibles dedicados a atender operaciones de lectura

Estos nodos especializados le permiten escalar verticalmente diferentes aspectos de su base de datos de forma independiente. Sus transacciones operativas pueden utilizar nodos estándar, mientras que los análisis que consumen muchos recursos se ejecutan en hardware dedicado optimizado para esas cargas de trabajo.

Considere una plataforma de comercio electrónico que necesita mantener un rendimiento de pago rápido mientras ejecuta análisis de inventario complejos. Al agregar nodos de análisis, pueden ejecutar consultas de informes pesadas sin afectar la experiencia del cliente.

Cada una de estas opciones de escalado vertical se puede implementar por separado o en combinación, lo que le brinda formas flexibles de abordar los cuellos de botella en el rendimiento antes de considerar la complejidad del escalado horizontal.

#### Capacidades de autoescalado

Una de las características más poderosas de Atlas es su capacidad de escalar automáticamente sus recursos en respuesta a cargas de trabajo cambiantes, lo que reduce la necesidad de una intervención manual constante.

#### Autoescalado de almacenamiento

El autoescalado de almacenamiento de Atlas ayuda a prevenir emergencias de espacio en disco aumentando proactivamente la capacidad de almacenamiento cuando sea necesario.

**Cómo funciona:**
1. Atlas monitorea la utilización del disco en todos los nodos de su clúster.
2. Cuando cualquier nodo alcanza el 90% de utilización del disco, Atlas aumenta automáticamente el almacenamiento.
3. Después del escalado, el objetivo es reducir la utilización a aproximadamente el 70%.
4. Esta función está habilitada de forma predeterminada para los clústeres nuevos.

**Cuándo no se producirá el escalado automático del almacenamiento:**
- Durante la actividad de escritura a alta velocidad (como inserciones masivas) donde Atlas no tiene tiempo suficiente para preparar y copiar datos.
- Cuando especifica diferentes clases de nivel de clúster para nodos base y nodos de análisis (por ejemplo, General para nodos operativos y Low-CPU para análisis).
- Cuando los nodos de nivel base y de nivel de análisis se implementan en diferentes regiones de proveedores de la nube.

Planifique escalar manualmente el almacenamiento antes de operaciones masivas de datos para evitar alcanzar los límites de capacidad durante actividades intensivas de escritura.

#### Autoescalado de nivel de clúster

Atlas puede ajustar automáticamente todo el nivel de su clúster según los patrones de utilización de CPU y memoria.

**Desencadenadores de escalado:**
- Para escalar hacia arriba (clústeres M30+):
  - La utilización de la CPU supera el 75% durante una hora o el 90% durante 10 minutos.
  - La utilización de la memoria supera el 75% durante una hora o el 90% durante 10 minutos.
- Para escalar hacia abajo:
  - La utilización de la CPU está por debajo del 45% durante al menos 4 horas.
  - No hubo actividad de escalado en las 24 horas anteriores.
  - El uso de memoria proyectado en el nivel inferior se mantendría por debajo del 60%.

#### Establecer límites de nivel adecuados

Dentro de la interfaz de usuario de Atlas, puede establecer niveles de clúster mínimos y máximos para controlar el comportamiento del autoescalado y evitar costos inesperados.

#### Consideraciones de autoescalado

Tomemos un momento para ver algunas de las consideraciones que debemos tener al usar el autoescalado para nuestros despliegues de Atlas:

- **Filosofía de diseño conservadora:** El autoescalado es intencionalmente conservador para evitar la oscilación entre niveles o reacciones a picos temporales. Para lanzamientos de productos o aumentos de tráfico planificados, considere el preescalado manual.
- **Relación entre almacenamiento y nivel:** Si el autoescalado de almacenamiento aumenta más allá de lo que admite su nivel actual, Atlas ajustará automáticamente su nivel mínimo hacia arriba, deshabilitando potencialmente el escalado hacia abajo.
- **Monitoreo:** Atlas registra todos los eventos de autoescalado en su feed de actividad (*Activity Feed*) y proporciona alertas configurables para actividades de autoescalado, reemplazando las notificaciones heredadas por correo electrónico.

Al comprender estas capacidades y limitaciones de escalado automático, puede lograr un equilibrio óptimo de rendimiento, rentabilidad y simplicidad operativa, evitando al mismo tiempo posibles fallas de escalado durante las operaciones.

#### Escalado horizontal en MongoDB Atlas

El escalado horizontal, a menudo llamado "escalado hacia afuera" (*scaling out*), permite que las bases de datos amplíen su capacidad distribuyendo datos y cargas de trabajo entre múltiples nodos. Este enfoque es ideal para adaptarse a altas cargas de tráfico, conjuntos de datos masivos y demandas fluctuantes de carga de trabajo en aplicaciones modernas. MongoDB Atlas hace que el escalado horizontal sea fluido a través de una gestión automatizada y capacidades sólidas de fragmentación (*sharding*).

MongoDB logra el escalado horizontal a través del particionamiento o fragmentación, donde los datos se dividen entre múltiples conjuntos de réplicas, llamados fragmentos (*shards*). Cada fragmento almacena un subconjunto del total de datos, lo que hace posible escalar horizontalmente a medida que crecen las necesidades de capacidad.

Las ventajas clave de la fragmentación en Atlas son las siguientes:

- Distribuye operaciones de lectura/escritura en múltiples fragmentos, permitiendo la escalabilidad para aplicaciones de alto rendimiento
- Supera las limitaciones de las máquinas individuales distribuyendo las cargas de trabajo
- Logra un mejor equilibrio de carga y eficiencia en comparación con depender únicamente del escalado vertical

#### Componentes de un clúster fragmentado

Un clúster fragmentado de MongoDB incluye lo siguiente:

- **Fragmentos (*Shards*):** Conjuntos de réplicas que contienen subconjuntos particionados de datos.
- **Servidores de configuración (*Config servers*):** Almacenan metadatos sobre la distribución y fragmentación de datos.
- **Enrutadores mongos (*mongos routers*):** Enrutan las solicitudes de consulta a los fragmentos correctos.

A partir de la versión 8.0, Atlas proporciona fragmentos de configuración (*config shards*), que reducen los costos para despliegues con hasta tres fragmentos mientras mantienen una arquitectura fragmentada completamente funcional.

#### Selección de clave de fragmentación (*Shard key*)

Elegir la clave de fragmentación correcta es importante para un escalado horizontal efectivo:

- **Alta cardinalidad:** Garantizar muchos valores únicos posibles
- **Distribución de escritura:** Evitar puntos calientes (*hotspots*) distribuyendo uniformemente las escrituras
- **Eficiencia de consulta:** Diseñar claves para admitir patrones de acceso comunes, permitiendo que las consultas se dirijan a fragmentos específicos

Atlas también admite tres estrategias de fragmentación:

- **Fragmentación por rango (*Ranged sharding*):** Divide los datos en rangos contiguos basados en valores de clave de fragmentación
- **Fragmentación con hash (*Hashed sharding*):** Emplea valores hash de clave de fragmentación para lograr una distribución uniforme de datos y cargas de trabajo
- **Fragmentación por zonas (*Zoned sharding*):** Conecta valores de rango específicos a regiones geográficas designadas

#### Clústeres globales para distribución geográfica

Los clústeres globales (*Global Clusters*) amplían los principios de fragmentación combinando la distribución geográfica y de carga de trabajo. Esta capacidad permite a Atlas proporcionar baja latencia para usuarios ubicados en diferentes regiones.

**Beneficios de los clústeres globales:**
- **Lecturas y escrituras de baja latencia:** Mejora el rendimiento enrutando solicitudes a la zona geográfica más cercana
- **Soberanía de datos:** Garantiza el cumplimiento regional de las regulaciones de almacenamiento de datos
- **Disponibilidad mejorada:** Aísla las interrupciones regionales, minimizando el impacto de las fallas
- **Experiencia de usuario perfecta:** Optimiza los tiempos de respuesta para cargas de trabajo distribuidas globalmente

Por ejemplo, una plataforma global de comercio electrónico puede utilizar la fragmentación por zonas para atender a clientes europeos desde fragmentos alojados en la UE por motivos de cumplimiento normativo, manteniendo al mismo tiempo consultas de inventario de alta velocidad en otras regiones.

Los siguientes son requisitos clave para implementar clústeres globales:

- **Colecciones fragmentadas:** Las colecciones deben estar fragmentadas para aprovechar la distribución global de datos
- **Definición de zona:** Asignar hasta nueve zonas geográficas para la ubicación de los datos
- **Mapeo de prioridades:** Definir regiones para nodos primarios, secundarios, de solo lectura y de análisis

#### Consideraciones administrativas

El escalado horizontal con MongoDB Atlas introduce varias opciones administrativas para la planificación y el mantenimiento.

A diferencia del escalado vertical (que puede aprovechar el autoescalado), el escalado horizontal implica decisiones de diseño deliberadas:

- **Selección de clave de fragmentación:** Elija claves según los patrones de acceso
- **Estrategia de fragmentación:** Seleccione entre enfoques con hash, por rangos o por zonas
- **Momento del clúster:** Introduzca la fragmentación una vez que la utilización de recursos alcance un pico de aproximadamente el 70%

#### Compensaciones operativas y de costos

El escalado horizontal ofrece escalabilidad a costos más bajos pero aumenta la sobrecarga administrativa:

- **Costos del servidor de configuración:** Aunque Atlas los reduce con fragmentos de configuración, siguen siendo una consideración
- **Transferencias multirregionales:** Los clústeres globales pueden incurrir en costos de transferencia de datos entre regiones
- **Riesgos de utilización desigual:** Una mala selección de la clave de fragmentación puede provocar puntos calientes

#### Escalado horizontal frente a escalado vertical

| Aspecto | Escalado Vertical | Escalado Horizontal |
| :--- | :--- | :--- |
| **Complejidad** | Baja (el autoescalado gestiona las actualizaciones) | Mayor (requiere selección de clave de fragmentación) |
| **Dinámica de costos** | Costoso en niveles grandes | A menudo más rentable para cargas de trabajo grandes |
| **Capacidad máxima** | Limitada por el tamaño de máquina más grande | Agregue fragmentos según sea necesario |
| **Impacto en modelado de datos** | Mínimo | Requiere clave de fragmentación y estrategia de fragmentación |
| **Eficiencia de consulta** | Adecuado para la mayoría de los patrones de consulta | Mejor para consultas específicas de fragmentos |

> **Tabla 8.1:** Comparación de escalado vertical y horizontal

MongoDB Atlas integra ambos enfoques, lo que permite un escalado automático detallado junto con un escalado horizontal para una elasticidad integral. Esto nos permite aprovechar fácilmente el escalado tanto horizontal como vertical a medida que evolucionan las cargas de trabajo.

Por ejemplo, durante los picos de tráfico navideño o eventos virales, Atlas puede escalar temporalmente los clústeres hacia arriba y al mismo tiempo distribuir las cargas de trabajo horizontalmente para optimizar el costo y el rendimiento.

#### Creación de una estrategia de escalado integral

Los enfoques de escalado más eficaces suelen combinar técnicas tanto verticales como horizontales, adaptándose a medida que evoluciona su aplicación.

Las estrategias de escalado exitosas a menudo superponen diferentes enfoques en lugar de elegir entre ellos:

- El escalado vertical proporciona simplicidad y ajustes inmediatos de recursos
- Los nodos de solo lectura ofrecen un paso intermedio antes de la fragmentación completa
- La fragmentación de colecciones específicas permite un escalado horizontal específico
- La distribución global aborda las necesidades de rendimiento geográfico

Muchas aplicaciones comienzan con el autoescalado vertical para mayor flexibilidad y luego introducen elementos horizontales a medida que surgen cuellos de botella específicos. Este enfoque híbrido equilibra la simplicidad con la escalabilidad.

Atlas proporciona herramientas para evaluar su estrategia de escalado a través de lo siguiente:

- Paneles de rendimiento en tiempo real
- Análisis de métricas históricas
- Estadísticas de rendimiento de consultas
- Seguimiento del uso de recursos

Una evaluación eficaz requiere establecer líneas base y objetivos claros para sus iniciativas de escalado. La revisión periódica de estas métricas ayuda a identificar cuándo ajustar su enfoque.

Una estrategia de escalado flexible se adapta tanto al crecimiento esperado como a los patrones de acceso cambiantes:

- Considere las implicaciones del modelo de datos para futuras necesidades de escalado
- Equilibre los requisitos inmediatos con la flexibilidad a largo plazo
- Establezca umbrales claros para escalar su enfoque
- Tenga en cuenta posibles cambios en la funcionalidad de la aplicación

Las estrategias más resilientes mantienen opciones de expansión tanto vertical como horizontal, lo que permite que su infraestructura se adapte junto con la evolución natural de su aplicación.

Al combinar cuidadosamente técnicas de escalado, monitorear métricas de rendimiento y mantener la flexibilidad en su enfoque, puede construir una estrategia de escalado que crezca con su aplicación mientras optimiza tanto el rendimiento como el costo.

---

### Sección 7: Configuración de autenticación y autorización

En el mundo de las bases de datos en la nube, la seguridad es una preocupación primordial y MongoDB Atlas no es una excepción. Proteger su despliegue de Atlas implica comprender e implementar eficazmente dos conceptos clave: autenticación y autorización. Estos mecanismos son vitales para crear una arquitectura de seguridad sólida que proteja sus datos y limite el acceso solo a quienes lo necesitan.

#### Comprensión de la autenticación y la autorización

La **autenticación** se trata de verificar identidades y responder a la pregunta: "¿Quién eres?". Garantiza que solo los usuarios y servicios legítimos puedan acceder a su sistema, sirviendo como la barrera inicial contra la entrada no autorizada.

Una vez que un usuario está autenticado, la **autorización** toma el control y responde a la pregunta: "¿Qué tienes permitido hacer?". La autorización rige los permisos del usuario dentro del sistema, otorgando acceso a recursos y acciones específicos según roles o políticas definidas. Juntos, estos procesos forman la columna vertebral de un entorno seguro de MongoDB Atlas.

#### El plano de control y el plano de datos

Para comprender completamente cómo funcionan la autenticación y la autorización en Atlas, primero debemos alejarnos y comprender las funciones del plano de control (*control plane*) y el plano de datos (*data plane*). Al distinguir entre estos dos planos, obtendrá claridad sobre las responsabilidades dentro de sus despliegues de Atlas.

El **plano de control** actúa como el corazón administrativo de su entorno de MongoDB Atlas. Este es el dominio de los **usuarios de Atlas**: aquellos que son responsables de administrar la infraestructura general. Configuran clústeres, configuran parámetros de seguridad y manejan operaciones de facturación. Piense en los usuarios de Atlas como los estrategas que definen cómo se organizan y mantienen los recursos.

Por el contrario, el **plano de datos** es donde se lleva a cabo la gestión práctica de los recursos de la base de datos. Este es el reino de los **usuarios de la base de datos**, personas que interactúan directamente con las colecciones y documentos dentro de sus clústeres de Atlas. A estos usuarios se les asignan las operaciones diarias, asegurando que se acceda a las bases de datos y se utilicen de manera efectiva, ya sea en fases de desarrollo o en entornos de producción de alta demanda.

Al definir claramente los roles de los usuarios de Atlas y los usuarios de bases de datos dentro de los planos de control y datos, comprenderá mejor cómo se distribuyen las responsabilidades, lo que permitirá una gestión más fluida y segura de sus implementaciones de Atlas.

#### Fuerza laboral frente a carga de trabajo (*Workforce versus workload*)

Una distinción más importante que se debe hacer antes de configurar la autenticación y la autorización es la diferencia entre usuarios de la fuerza laboral (*workforce*) y usuarios de la carga de trabajo (*workload*). Saber esto fundamentará su decisión sobre qué método de autenticación elegir.

Los **usuarios de la fuerza laboral** son personas que interactúan con un sistema. Esto incluye roles como administradores, desarrolladores o analistas que utilizan herramientas e interfaces diseñadas para uso humano.

Por otro lado, los **usuarios de carga de trabajo** se refieren a entidades automatizadas, como programas o scripts, que realizan tareas dentro de un sistema sin intervención humana directa. Estos pueden incluir aplicaciones o procesos en segundo plano. Por lo general, los usuarios de carga de trabajo utilizan la API de administración de Atlas para administrar e interactuar de manera eficiente con los recursos de MongoDB Atlas, automatizar operaciones rutinarias y agilizar los flujos de trabajo.

Si está interesado en utilizar la API de administración de Atlas, asegúrese de consultar la documentación sobre ella ([https://www.mongodb.com/docs/atlas/api/atlas-admin-api/](https://www.mongodb.com/docs/atlas/api/atlas-admin-api/)).

Hacer la distinción entre usuarios de la fuerza laboral y usuarios de carga de trabajo es importante porque ayuda a adaptar las estrategias de seguridad, autenticación y acceso a las necesidades únicas de cada grupo. Los usuarios de la fuerza laboral, al ser individuos, generalmente requieren mecanismos de autenticación que prioricen la facilidad de uso y la integración con los sistemas de gestión de identidades de la organización, como el inicio de sesión único (SSO). Por el contrario, los usuarios de carga de trabajo, como aplicaciones o scripts automatizados, necesitan medidas de seguridad sólidas que garanticen un acceso seguro y fluido, a menudo utilizando métodos como claves de API o cuentas de servicio.

Cambiemos nuestro enfoque hacia la autenticación.

#### Autenticación

Podríamos llenar un libro entero con material práctico sobre la creación de un sistema de autenticación sólido. En cambio, esta sección ofrecerá una descripción general de alto nivel del panorama de autenticación, lo que permitirá una toma de decisiones informada para su caso de uso individual. Una vez que identifique un enfoque adecuado que satisfaga sus necesidades, la documentación de MongoDB incluye tutoriales para guiar el proceso de implementación.

Hay dos tipos de usuarios en MongoDB Atlas: usuarios de Atlas y usuarios de bases de datos, cada uno con distintos métodos de autenticación. Para empezar, nos centraremos en los usuarios de Atlas, que gestionan el plano de control y desempeñan un papel clave en los flujos de trabajo de autenticación.

#### Autenticación de usuarios de la fuerza laboral de Atlas

Para los usuarios de Atlas, la autenticación se puede lograr mediante un nombre de usuario y una contraseña, complementados con autenticación multifactor para mayor seguridad. Alternativamente, puede utilizar SSO para integrarse con plataformas populares como GitHub o Google.

En muchos casos, las organizaciones requieren un enfoque de autenticación más integrado. Para abordar esta necesidad, puede configurar la autenticación federada. Esto le permite conectarse con el proveedor de identidad elegido, como Okta o Microsoft Entra ID, lo que garantiza un acceso fluido para su equipo. Para obtener una lista completa de los proveedores de identidad admitidos, asegúrese de consultar la documentación de Atlas.

Estos métodos de autenticación son excelentes para los usuarios de la fuerza laboral, pero ¿qué pasa con los usuarios de carga de trabajo?

#### Autenticación de usuarios de carga de trabajo de Atlas

Hasta hace poco, la única opción para la autenticación de usuarios de carga de trabajo en Atlas era utilizar claves de API de Atlas, que aún se pueden generar y utilizar en la actualidad. Pero la introducción de cuentas de servicio (*service accounts*) proporciona una solución mejorada.

Las cuentas de servicio en MongoDB Atlas ofrecen una forma moderna para que las aplicaciones se autentiquen con la API de administración de MongoDB Atlas. A diferencia de las claves de API programáticas (*Programmatic API Keys* o PAK) tradicionales que podría haber utilizado para proporcionar acceso a los usuarios, las cuentas de servicio utilizan el estándar OAuth 2.0, lo que agiliza y automatiza los procesos de autenticación. Esto significa que sus aplicaciones, en lugar de usuarios individuales, pueden interactuar de forma segura y eficiente con los recursos de MongoDB Atlas, permitiendo a los equipos de desarrollo elegir los flujos de trabajo de autenticación que mejor se adapten a sus necesidades.

Una de las mejoras que proporcionan las cuentas de servicio sobre las claves de API se encuentra en el ámbito de la seguridad y la automatización. Mientras que las PAK requieren una gestión y rotación diligentes para mantener la seguridad, las cuentas de servicio automatizan gran parte de esto a través del flujo de credenciales de cliente de OAuth 2.0. Esto significa menos trabajo manual para usted, ya que simplemente puede regenerar los secretos del cliente sin tener que cambiar las demás configuraciones de la cuenta de servicio. El enfoque de OAuth 2.0 también es ampliamente compatible, lo que facilita la integración de diversos servicios y componentes en toda su pila tecnológica.

La capacidad de las cuentas de servicio para sincronizarse con su proveedor de identidades existente marca un antes y un después. Esto permite a su organización gestionar la autenticación utilizando sistemas de identidad nativos de la nube familiares, adaptándose perfectamente a su configuración actual. Al permitirle utilizar medidas de control de acceso e identidad establecidas, MongoDB Atlas facilita la incorporación de cuentas de servicio en las estrategias de autenticación que ya tiene, todo mientras alivia la carga de sus equipos de TI y seguridad.

#### Autenticación de usuarios de la base de datos

Habiendo cubierto los métodos de autenticación para los usuarios de Atlas, profundicemos en los mecanismos de autenticación disponibles para los usuarios de bases de datos en MongoDB Atlas.

- **Salted Challenge Response Authentication Mechanism (SCRAM):** Proporciona una configuración sencilla de nombre de usuario y contraseña. Este método es ideal para entornos de desarrollo que priorizan la velocidad, pero puede no ser adecuado para aplicaciones de producción debido a su simplicidad. Incluso en el desarrollo, las prácticas de seguridad sólidas son importantes. Utilice contraseñas seguras y considere herramientas como HashiCorp Vault para la gestión segura de credenciales.
- **Federación de identidades (*Identity federation*):** Para una experiencia de autenticación más integrada, la federación de identidades se conecta con los proveedores de identidades existentes de su organización, de forma similar a la autenticación federada para los usuarios de Atlas. Admite dos tipos principales de usuarios de bases de datos:
  - *Usuarios de la fuerza laboral:* Son usuarios humanos que acceden y administran MongoDB Atlas a través de interfaces como la interfaz de usuario web o las herramientas de línea de comandos. La federación de identidades les permite autenticarse utilizando credenciales administradas por el proveedor de identidades de su organización, como Okta o Microsoft Entra ID, lo que agiliza la autenticación y reduce los esfuerzos de administración manual.
  - *Federación de identidades de carga de trabajo (*Workload identity federation*):* Le permite configurar identidades desde cada proveedor de nube o cualquier servicio de autorización compatible con OAuth 2.0. Los siguientes son ejemplos:
    - *Azure:* Utilice entidades de servicio e identidades administradas de Azure
    - *Google Cloud:* Aproveche las cuentas de servicio de Google
    - *AWS:* Utilice roles de AWS IAM
- **Pasos para configurar la federación de identidades en MongoDB Atlas:**
  1. Agregue y verifique sus dominios.
  2. Configure su proveedor de identidad con Atlas.
  3. Conecte sus dominios con su proveedor de identidad.
  4. Active su proveedor de identidad.
  *(Puede encontrar guías detalladas para cada proveedor de identidad en la documentación de Atlas).*
- **Certificados X.509:** Permiten Transport Layer Security mutuo (mTLS). Al utilizar certificados X.509, podemos aprovechar mTLS. TLS mutuo se basa en el protocolo de enlace TLS estándar al requerir que tanto el cliente como el servidor autentiquen el certificado del otro. Esta autenticación mutua garantiza que ambas partes involucradas en una comunicación sean de confianza. X.509 es ideal para autenticar aplicaciones que necesitan conectarse a sus clústeres.

Seleccionar el mejor método de autenticación para los usuarios de su base de datos depende de sus necesidades operativas y de seguridad específicas. Al evaluar estas opciones, podrá proteger mejor sus datos y operaciones.

#### Autorización

Habiendo discutido la autenticación en Atlas, es natural pasar a la autorización, que responde a la pregunta: "¿Qué tienes permitido hacer?". En Atlas, esto se gestiona a través de RBAC.

La implementación de la autorización está estrechamente relacionada con el modelo de seguridad de confianza cero (*zero-trust*) y el principio de mínimo privilegio. La confianza cero enfatiza que no se debe confiar en nadie de forma predeterminada, validando continuamente los derechos de acceso. Mientras tanto, el principio de mínimo privilegio garantiza que los usuarios tengan solo los permisos necesarios para sus funciones. Juntos, estos principios guían la configuración de RBAC en Atlas, mejorando la seguridad mediante un control y una verificación estrictos del acceso.

#### Control de acceso basado en roles (RBAC)

RBAC define un marco para administrar y restringir el acceso a la base de datos según roles predefinidos. Cada rol abarca privilegios específicos, que determinan qué acciones puede realizar un usuario. Al asignar roles a los usuarios, puede controlar su acceso y sus acciones dentro de la base de datos y Atlas de manera eficiente. Este enfoque mejora la seguridad al garantizar que los usuarios solo tengan los permisos necesarios para su trabajo, adhiriéndose al principio de mínimo privilegio.

Es importante comprender que RBAC se aplica tanto a los usuarios de Atlas como a los usuarios de bases de datos. Sin embargo, sus funciones y privilegios no se comparten. Como exploraremos en breve, los usuarios de Atlas y los usuarios de bases de datos tienen cada uno sus propios roles y privilegios distintos.

Visualicemos cómo se puede aplicar RBAC en un escenario del mundo real tanto para los usuarios de Atlas como para los usuarios de bases de datos:

Imagine una empresa de desarrollo de software mediana que aprovecha MongoDB Atlas para gestionar los datos de sus proyectos. Dentro de la empresa, hay diferentes miembros del equipo con distintas necesidades de acceso:

- **Usuarios de Atlas:**
  - *Administradores de plataforma:* Estos usuarios, responsables de administrar la infraestructura de la cuenta de MongoDB Atlas, tienen roles que les otorgan permisos para implementar nuevos clústeres, administrar configuraciones y supervisar los protocolos de seguridad. Con amplios privilegios, garantizan un rendimiento óptimo y la seguridad de la infraestructura de base de datos en la nube de la empresa.
- **Usuarios de base de datos:**
  - *Desarrolladores:* Como usuarios de bases de datos, requieren acceso para crear, leer y actualizar documentos dentro de colecciones específicas. Se les asigna un rol de base de datos de desarrollador (*Developer*), lo que les permite realizar operaciones Create, Read, Update, Delete (CRUD) relevantes para sus tareas.
  - *Analistas de datos:* También usuarios de bases de datos, dependen del acceso de lectura en varias colecciones para extraer información y analizar tendencias de datos. Su rol de analista (*Analyst*) proporciona privilegios de lectura alineados con sus tareas analíticas, permitiéndoles ejecutar consultas complejas utilizando las potentes herramientas de MongoDB, como el marco de agregación.

Si bien estos roles no se asignan directamente a los roles de Atlas, puede obtener una idea básica de cómo puede verse RBAC en un entorno de equipo.

Ahora que sabemos qué es RBAC, comencemos a explorar sus detalles específicos en Atlas.

#### Roles de usuario de Atlas

Un buen lugar para comenzar son los roles integrados para los usuarios de Atlas. Los roles integrados son conjuntos predefinidos de permisos que puede asignar a los usuarios para controlar su acceso y acciones dentro de Atlas. Estos roles ayudan a simplificar la gestión de usuarios al proporcionar una forma rápida de otorgar niveles adecuados de acceso sin tener que definir roles personalizados manualmente.

Para los usuarios de Atlas, tenemos roles tanto para organizaciones como para proyectos. Las organizaciones administran elementos como el acceso de usuarios de Atlas, la facturación y diversas tareas administrativas. Dado esto, los roles de organización están vinculados a este tipo de tareas. Los siguientes son ejemplos:

- **Organization Owner:** Como cumbre de la gestión, el Organization Owner tiene la autoridad para gobernar todos los aspectos de la organización. Esto incluye supervisar las configuraciones, manejar la facturación y administrar usuarios y proyectos, proporcionando un control integral sobre las funciones administrativas.
- **Billing Admin:** Enfocado exclusivamente en operaciones financieras, este rol maneja detalles de facturación y administra métodos de pago. Si bien es crucial para la transparencia financiera, el Billing Admin no posee derechos para modificar la configuración del proyecto o tareas administrativas no financieras.
- **Organization Member:** Estos miembros tienen una vista limitada de la organización, con privilegios administrativos restringidos. Pueden acceder a los detalles de la organización pero carecen de derechos para crear, modificar o eliminar clústeres, o para administrar el acceso de los usuarios, lo que garantiza operaciones organizativas seguras.

Dentro de cada organización, los proyectos actúan como entornos individuales diseñados para aplicaciones específicas. Los roles de proyecto están diseñados para complementar los roles organizacionales y proporcionar un control granular dentro de los proyectos. Cada proyecto puede tener roles únicos:

- **Project Owner:** Con acceso total, el Project Owner puede configurar y administrar todos los aspectos del proyecto. Desde la creación de clústeres hasta la configuración de herramientas de monitoreo y copias de seguridad, este rol permite una gestión integral, incluido el manejo de permisos de usuario y la integración de herramientas de observabilidad como Query Profiler para el análisis de datos.
- **Project Data Access Read Only:** Ideal para usuarios que necesitan visibilidad de las bases de datos sin la capacidad de modificarlas, este rol proporciona privilegios de visualización en Data Explorer y herramientas de observabilidad, garantizando la integridad de los datos y evitando cambios no autorizados.
- **Project Cluster Admin:** Especializado en operaciones de clústeres, este rol permite pausar, editar y reanudar clústeres, pero no crear nuevos. Garantiza un control preciso sobre las configuraciones de clústeres existentes, al tiempo que deja de lado tareas de creación más amplias reservadas para roles superiores.

Los roles de acceso a datos de proyecto (*Project Data Access*) son únicos porque otorgan a los usuarios del plano de control acceso al plano de datos. Esto debe tenerse en cuenta al conceder estos roles a los usuarios.

Es importante tener en cuenta que estos roles son jerárquicos. Los roles de nivel superior en la organización tienen permisos generales que pueden afectar a múltiples proyectos, mientras que los roles específicos del proyecto son más específicos y de alcance restringido.

> **Figura 8.4:** La relación jerárquica y las diferencias entre los roles de organización y de proyecto

Si bien esta no es una lista exhaustiva, le da una idea de los tipos de roles integrados disponibles. Si desea una lista más completa de todos los roles integrados, puede consultar la documentación de Atlas ([https://www.mongodb.com/docs/atlas/reference/user-roles/](https://www.mongodb.com/docs/atlas/reference/user-roles/)).

Entonces, ¿cómo asignamos un rol a un usuario de Atlas? He aquí una manera: podemos asignar roles cuando invitamos a alguien a nuestra organización, de la siguiente manera:

```bash
atlas users invite --email user@example.com --username user@example.com --orgRole <orgId>:ORG_MEMBER --projectRole <projectId>:GROUP_READ_ONLY --firstName Example --lastName User --country US --output json
```

#### Roles de usuario de base de datos

Cambiemos nuestra atención al usuario de la base de datos, que se encarga de gestionar los datos directamente. En MongoDB, un usuario de base de datos sirve como credencial para autenticarse en su clúster, actuando como una entidad única con permisos específicos que determinan las acciones que puede realizar dentro de la base de datos. De manera similar a los usuarios de Atlas, los usuarios de bases de datos tienen su propio conjunto de roles integrados que podemos aprovechar, como los siguientes:

- **`atlasAdmin`:** El rol `atlasAdmin` puede realizar una amplia gama de tareas administrativas. Esto incluye administrar fragmentos, configurar ajustes de bases de datos y realizar operaciones en todas las bases de datos dentro del clúster. Es ideal para usuarios que necesitan un control y supervisión integrales.
- **`readWriteAnyDatabase`:** Este rol permite a los usuarios ejecutar operaciones de lectura y escritura en cualquier base de datos dentro del clúster. Es una opción versátil para quienes necesitan un acceso amplio para la manipulación de datos pero no requieren permisos administrativos.
- **`readAnyDatabase`:** Diseñado para usuarios que solo necesitan leer datos, el rol `readAnyDatabase` otorga permiso para acceder y ver datos en todas las bases de datos del clúster. Restringe cualquier modificación de datos, lo que lo hace perfecto para analistas o miembros del equipo centrados en el examen de datos sin alterarlos.

Esta no es una lista exhaustiva de roles de usuarios de bases de datos, pero le da una idea de los tipos de roles integrados disponibles para nosotros.

Necesitaremos un usuario de base de datos para cuando comencemos a trabajar con Atlas Search y Vector Search, así que adelante, creemos ese usuario ahora para que podamos ver cómo se utilizan los roles de base de datos.

Para crear un nuevo usuario de base de datos, ejecute el siguiente comando:

```bash
atlas dbusers create readWriteAnyDatabase --username searchDemoUser --projectId <projectId>
```

- `atlas dbusers create`: Este es el comando para crear un nuevo usuario de base de datos en un proyecto de Atlas.
- `readWriteAnyDatabase`: Esto especifica el rol que se asignará al usuario.
- `--username searchDemoUser`: Este indicador establece el nombre de usuario para el nuevo usuario de base de datos. En este ejemplo, el nombre de usuario proporcionado es `searchDemoUser`. Este es el nombre que el usuario utilizará para autenticarse en las bases de datos del proyecto especificado.
- `--projectId <projectId>`: El indicador `--projectId` especifica el identificador único para el proyecto de MongoDB Atlas donde se creará este usuario. `<projectId>` debe reemplazarse con la cadena de ID del proyecto real. Este ID garantiza que el usuario se agregue al entorno de proyecto correcto dentro de su organización de Atlas.

Ahora tenemos un usuario de base de datos que podemos usar para el resto de este capítulo para conectarnos a nuestro clúster. En nuestro caso, los roles integrados funcionaron muy bien. Pero ¿qué sucede si necesitamos un control más granular sobre los privilegios de un determinado rol?

#### Roles personalizados

Tenemos la capacidad de crear roles personalizados para usuarios de bases de datos. Los roles de usuario de bases de datos personalizados le permiten adaptar el control de acceso para que se ajuste a las necesidades específicas de su aplicación u organización. Si bien MongoDB Atlas proporciona un conjunto de roles integrados que cubren casos de uso comunes, la creación de roles personalizados proporciona flexibilidad cuando necesita permisos más granulares.

Con los roles personalizados, podemos hacer lo siguiente:

- **Definir privilegios específicos:** Asignar privilegios precisos a un rol, como acceso de lectura o escritura a bases de datos o colecciones específicas, o incluso operaciones administrativas como la gestión de índices.
- **Mejorar la seguridad:** Limitar a los usuarios a solo los recursos y acciones que requieren, reduciendo la exposición potencial de un acceso excesivamente permisivo.
- **Mejorar la gestión:** Agilizar la gestión de roles agrupando permisos de usuario, lo que facilita la asignación y actualización de controles de acceso a medida que su equipo o aplicación evoluciona.

La creación de un rol personalizado en MongoDB Atlas implica identificar las acciones y los recursos necesarios para un usuario o aplicación en particular y luego ejecutar estas definiciones a través de la interfaz de usuario o la API de Atlas.

Para crear un rol de base de datos personalizado, ejecute el siguiente comando:

```bash
atlas customDbRoles create customRole --privilege FIND@databaseName,UPDATE@databaseName.firstCollectionName,UPDATE@databaseName.secondCollectionName
```

- `atlas customDbRoles create`: Este es el comando para crear un rol de base de datos personalizado.
- `customRole`: Este es el nombre del rol de base de datos personalizado que se está creando. Puede elegir cualquier nombre que tenga sentido para el propósito y alcance del rol.
- `--privilege`: Este indicador se utiliza para especificar los privilegios que tendrá el rol. Cada privilegio se define con una acción y un objetivo, con el formato `ACTION@TARGET`.
  - `FIND@databaseName`: Esto especifica un privilegio que permite al rol personalizado realizar operaciones de lectura (como consultar documentos) dentro de toda la base de datos `databaseName`. `FIND` es la acción y `databaseName` es la base de datos de destino en la que se puede realizar esta acción.
  - `UPDATE@databaseName.firstCollectionName`: Esto otorga el privilegio de realizar operaciones de actualización en `firstCollectionName` dentro de `databaseName`. `UPDATE` es la acción y `databaseName.firstCollectionName` especifica la colección en la que este privilegio es aplicable.
  - `UPDATE@databaseName.secondCollectionName`: De manera similar, esto otorga el privilegio de realizar operaciones de actualización en `secondCollectionName` dentro de `databaseName`. Nuevamente, `UPDATE` es la acción y `databaseName.secondCollectionName` es la colección de destino.

Podemos hacer que los roles sean aún más granulares especificando acciones concretas, como en el siguiente comando:

```bash
atlas customDbRoles create customRole --privilege GET_CMD_LINE_OPTS
```

`GET_CMD_LINE_OPTS` es un privilegio específico que permite al usuario acceder a las opciones de línea de comandos con las que se inició la instancia de MongoDB.

O podemos hacer que los roles de base de datos personalizados hereden privilegios de otros roles, como en el siguiente comando:

```bash
atlas customDbRoles create customRole --inheritedRole read@databaseName
```

- `--inheritedRole`: Este indicador se utiliza para especificar que el nuevo rol personalizado heredará permisos de otro rol predefinido. En MongoDB, los roles predefinidos vienen con un conjunto de privilegios integrados específicos que se pueden aprovechar para crear un rol personalizado con permisos similares.
- `read@databaseName`: Esto indica el rol heredado del cual el rol personalizado derivará sus permisos. El rol `read` es un rol de MongoDB predefinido que otorga privilegios para leer datos de todas las colecciones dentro de una base de datos específica. `@databaseName` especifica la base de datos particular donde se aplica este acceso de lectura.

Como podemos ver, Atlas ofrece soluciones amplias y sólidas para autenticar y autorizar a los usuarios. Con suerte, esto le dará una idea de lo que es posible con Atlas y podrá aplicarlo a sus propios despliegues. Pero la seguridad no empieza ni termina sólo con la autenticación y la autorización. En la siguiente sección, veremos más funciones de seguridad disponibles en Atlas para mantener sus datos protegidos.

---

### Sección 8: Protección de su despliegue de Atlas

Ahora que hemos cubierto la autenticación y la autorización, echemos un vistazo a otras características que mantienen seguros sus despliegues de Atlas para que pueda estar tranquilo. Pero primero, ¿alguna vez se ha preguntado por qué tantas organizaciones luchan contra las violaciones de seguridad, incluso después de invertir fuertemente en herramientas de seguridad? La respuesta suele residir en su enfoque: la seguridad se añade como una ocurrencia tardía en lugar de integrarse en los cimientos de sus sistemas. Cuando se trata de su plataforma de datos, esta distinción marca la diferencia.

MongoDB Atlas adopta una filosofía de **seguridad por diseño (*security by design*)**, donde se integran medidas de seguridad sólidas desde cero en lugar de agregarse como una capa después del hecho. Este enfoque proactivo garantiza que sus datos permanezcan protegidos a lo largo de todo su ciclo de vida, desde la creación hasta la eliminación.

#### Por qué es importante la seguridad desde el principio

Piense en la seguridad como los cimientos de una casa. Es mucho más fácil (y más eficaz) construirla adecuadamente desde el principio que intentar adaptarla más tarde. Cuando la seguridad es una ocurrencia tardía, podría ocurrir lo siguiente:

- Las vulnerabilidades pueden quedar profundamente integradas en su arquitectura.
- La remediación se vuelve exponencialmente más compleja y costosa.
- Los requisitos de cumplimiento normativo pueden forzar cambios disruptivos.
- Sus datos permanecen en riesgo durante la brecha entre el despliegue y la implementación de la seguridad.

Al incorporar la seguridad desde el primer día, crea una estrategia de defensa en profundidad que protege sus datos en múltiples niveles. Este enfoque se alinea con las mejores prácticas de seguridad modernas y reduce su exposición general al riesgo.

#### Herramientas de seguridad de Atlas: Protección integral

MongoDB Atlas proporciona un extenso conjunto de herramientas de seguridad que cubre todos los aspectos de la seguridad de bases de datos. Exploremos lo que esto significa para sus despliegues:

- **Seguridad de red:** Atlas proporciona múltiples capas de protección de red para proteger la infraestructura de su base de datos:
  - *Cifrado TLS obligatorio para todas las conexiones:* Todos los datos transmitidos entre su aplicación y Atlas se cifran mediante TLS. Esto evita la interceptación no autorizada de datos durante la transmisión y protege contra ataques de intermediario (*man-in-the-middle*).
  - *Listas de acceso IP para controlar quién puede conectarse:* Puede especificar qué direcciones IP o bloques CIDR pueden conectarse a sus clústeres. Esto crea una primera línea de defensa al evitar intentos de conexión desde redes o ubicaciones no autorizadas.
  - *Puntos de enlace privados (*Private endpoints*) y emparejamiento de VPC (*VPC peering*) para aislamiento de la Internet pública:* Estas características permiten que sus aplicaciones se conecten a Atlas sin exponer el tráfico a la Internet pública. Al establecer conexiones de red privadas, reduce la superficie de ataque y minimiza la exposición a amenazas externas.
- **Autenticación y autorización:** Implementamos una gestión integral de identidades y accesos para garantizar que solo los usuarios autorizados puedan acceder a nuestros datos:
  - *RBAC con permisos granulares:* RBAC le permite asignar permisos específicos a los usuarios según sus responsabilidades. Esto implementa el principio de mínimo privilegio, garantizando que los usuarios tengan acceso solo a los recursos que necesitan para realizar sus funciones laborales.
  - *Autenticación de usuarios de base de datos con múltiples mecanismos:* Atlas admite varios métodos de autenticación, incluidos nombre de usuario/contraseña, certificados X.509, OpenID Connect (OIDC)/OAuth 2.0 y LDAP. Esta flexibilidad le permite implementar la estrategia de autenticación que mejor cumpla con sus requisitos de seguridad.
  - *Integración con proveedores de identidad a través de OIDC:* La integración de OIDC le permite utilizar su proveedor de identidad existente (como Entra ID, Okta o Auth0) para administrar el acceso a Atlas. Esto centraliza la gestión de usuarios y habilita capacidades de inicio de sesión único (SSO).
  - *Control de acceso a nivel de organización y proyecto:* Atlas organiza los recursos de forma jerárquica, lo que le permite establecer permisos en diferentes niveles. Esta estructura le permite implementar políticas de acceso consistentes entre equipos manteniendo una separación adecuada de funciones.
- **Protección de datos:** Atlas protege sus datos con múltiples opciones de cifrado y funciones de seguridad:
  - *Cifrado en tránsito mediante TLS/SSL:* Todas las comunicaciones entre clientes y Atlas se cifran utilizando protocolos TLS modernos. Esto garantiza la confidencialidad de los datos durante la transmisión y ayuda a satisfacer los requisitos de cumplimiento para la protección de datos.
  - *Cifrado en reposo mediante cifrado AES-256:* Todos los datos almacenados en Atlas se cifran automáticamente mediante el cifrado estándar de la industria AES-256. Esto protege sus datos del acceso no autorizado si el medio de almacenamiento se ve comprometido o es robado.
  - *Claves de cifrado opcionales administradas por el cliente (BYOK):* Para un mayor control, puede utilizar sus propias claves de cifrado administradas en un servicio de gestión de claves en la nube. Esto agrega una capa de separación entre sus datos y el proveedor de servicios de base de datos.
  - *Client-Side Field Level Encryption y Queryable Encryption para datos confidenciales:* Estas funciones avanzadas cifran campos específicos en sus documentos antes de que salgan de su aplicación. Esto protege la información altamente confidencial, como la Información de Identificación Personal (PII), de los operadores de bases de datos.
- **Monitoreo y auditoría:** Nuestra plataforma ofrece una amplia visibilidad de la actividad de la base de datos para ayudarle a mantener el cumplimiento de la seguridad:
  - *Monitoreo continuo de actividades sospechosas:* Atlas monitorea activamente patrones de acceso inusuales, intentos de autenticación y otros eventos de seguridad. Esto ayuda a detectar posibles incidentes de seguridad de forma temprana para que pueda responder rápidamente.
  - *Registros de auditoría completos para rastrear accesos y cambios:* Los registros detallados registran quién accedió a su base de datos, qué acciones realizó y cuándo. Estos registros proporcionan una pista de auditoría esencial para las investigaciones de seguridad y los informes de cumplimiento.
  - *Integración con soluciones SIEM líderes:* La integración de Gestión de Información y Eventos de Seguridad (SIEM) le permite incorporar datos de seguridad de Atlas en su infraestructura de monitoreo de seguridad más amplia. Esto crea una vista unificada de su postura de seguridad.
  - *Seguimiento de actividad a nivel de organización y de proyecto:* Atlas registra acciones administrativas tanto a nivel de organización como de proyecto. Este seguimiento de múltiples niveles proporciona visibilidad tanto de los cambios de configuración estratégicos como de las actividades operativas tácticas.

La belleza del enfoque de Atlas es que muchas de estas características de seguridad están habilitadas de forma predeterminada y no requieren configuración adicional. Esto reduce el riesgo de una mala configuración al tiempo que garantiza que sus despliegues comiencen con una sólida postura de seguridad.

Al construir sobre esta base segura, puede concentrarse en crear valor con sus aplicaciones en lugar de preocuparse por las vulnerabilidades de seguridad en su capa de datos. A medida que avancemos en esta sección, exploraremos cada uno de estos aspectos de seguridad con mayor detalle, brindándole una comprensión integral de cómo maximizar la seguridad de sus implementaciones de Atlas.

#### Cumplimiento y estándares

MongoDB Atlas mantiene rigurosas certificaciones de cumplimiento para cumplir con los requisitos regulatorios en todas las industrias. Estas certificaciones garantizan que Atlas cumpla con los marcos de seguridad globales al manejar sus datos confidenciales:

- **SOC-2:** Verifica controles sólidos para seguridad, disponibilidad, integridad de procesamiento, confidencialidad y privacidad.
- **ISO/IEC 27001:** Confirma la implementación de un sistema integral de gestión de seguridad de la información.
- **PCI DSS:** Cumple con los requisitos de PCI DSS para aplicaciones que procesan información de pagos.
- **HIPAA:** Respalda el cumplimiento con salvaguardas para la Información de Salud Protegida (PHI) para organizaciones de atención médica.
- **GDPR:** Proporciona herramientas y controles para ayudar a cumplir con los requisitos de GDPR para el manejo de datos personales de ciudadanos de la UE.
- **FedRAMP Moderate:** Cumple con los estándares de seguridad para servicios en la nube utilizados por agencias y contratistas del gobierno federal de EE. UU.

Si está interesado en obtener más información sobre las regulaciones y estándares que cumple Atlas, visite el Centro de confianza de MongoDB (*MongoDB Trust Center*) ([https://www.mongodb.com/products/platform/trust](https://www.mongodb.com/products/platform/trust)).

MongoDB mantiene continuamente estas certificaciones a través de auditorías periódicas de terceros, lo que le permite crear aplicaciones compatibles sin duplicar los controles de seguridad.

#### Modelo de responsabilidad compartida

¿Se ha preguntado dónde terminan sus responsabilidades de seguridad y dónde comienzan las de MongoDB? El modelo de responsabilidad compartida aclara este límite, garantizando que comprenda exactamente quién maneja qué aspectos de la seguridad en su despliegue de Atlas.

#### Comprensión de la división de funciones de seguridad

El modelo de responsabilidad compartida es un marco de seguridad que divide las responsabilidades de protección entre MongoDB (el proveedor de servicios en la nube) y usted (el cliente). Esta clara delimitación ayuda a prevenir brechas de seguridad al tiempo que evita esfuerzos redundantes.

**MongoDB Atlas asume la responsabilidad de lo siguiente:**
- Seguridad de la infraestructura física
- Infraestructura de red
- Seguridad y parches del software de base de datos
- Cifrado de datos en tránsito
- Cifrado predeterminado de datos en reposo
- Mecanismos de autenticación
- Infraestructura de alta disponibilidad y recuperación ante desastres
- Certificaciones y auditorías de cumplimiento

**Como cliente de Atlas, usted es responsable de lo siguiente:**
- Configuración adecuada del acceso a la red (listas de acceso IP, puntos de enlace privados)
- Gestión del acceso de usuarios y controles de identidad
- Selección de métodos de autenticación adecuados
- Políticas de clasificación y protección de datos
- Controles de seguridad a nivel de aplicación
- Implementación de cifrado del lado del cliente (si es necesario)
- Monitoreo y respuesta a eventos de seguridad
- Gestión y prueba de copias de seguridad

Esta división le permite concentrarse en proteger su aplicación y sus datos mientras MongoDB se encarga de la seguridad de la infraestructura subyacente. Piense en ello como si MongoDB asegurara la casa, mientras usted controla quién obtiene las llaves y a qué puede acceder dentro.

Comprender dónde recaen estas responsabilidades es clave para mantener un despliegue seguro. Cuando ocurren incidentes de seguridad, a menudo explotan brechas en esta comprensión; por ejemplo, cuando los clientes asumen que MongoDB maneja ciertos controles de acceso que en realidad son su responsabilidad.

Al reconocer sus obligaciones de seguridad dentro de este modelo, puede implementar procesos y controles adecuados para garantizar una protección integral en todo el entorno de su base de datos.

#### Cifrado en MongoDB Atlas

La seguridad de los datos no termina con el cumplimiento y las responsabilidades compartidas. La forma en que sus datos están realmente protegidos a medida que se mueven a través de sus sistemas es igualmente importante. MongoDB Atlas emplea una estrategia de cifrado integral que protege sus datos a lo largo de su ciclo de vida: en tránsito, en reposo y en uso.

#### Cifrado en tránsito

¿Alguna vez le ha preocupado que su información confidencial sea interceptada mientras viaja a través de las redes? El cifrado en tránsito aborda esta preocupación protegiendo los datos a medida que se mueven entre su aplicación y la base de datos.

**¿Qué es el cifrado en tránsito?**
El cifrado en tránsito protege sus datos mientras se mueven a través de las redes: entre los servidores de sus aplicaciones y los clústeres de MongoDB Atlas o entre nodos dentro de un clúster. Garantiza que incluso si alguien intercepta el tráfico de su red, no pueda leer ni modificar sus datos.

En MongoDB Atlas, el cifrado en tránsito se implementa mediante TLS, el sucesor de SSL. TLS crea un canal cifrado seguro para la transmisión de datos al hacer lo siguiente:
- Autenticar la identidad del servidor
- Establecer una conexión cifrada segura
- Garantizar la integridad de los datos durante la transmisión

**Protección siempre activa (*Always-on protection*):**
A diferencia de muchos sistemas de bases de datos donde el cifrado en tránsito es opcional, Atlas adopta un enfoque de seguridad primero haciendo que el cifrado TLS sea obligatorio. Esto significa lo siguiente:
- Todas las conexiones a sus clústeres de Atlas están siempre cifradas
- TLS está habilitado de forma predeterminada y no se puede desactivar
- Ningún dato confidencial viaja jamás "en texto claro" a través de las redes

Este cifrado obligatorio protege contra varios ataques de red, incluidos los siguientes:
- Ataques de intermediario (*man-in-the-middle*), donde los atacantes intentan interceptar las comunicaciones
- Rastreo de paquetes (*packet sniffing*), que podría exponer consultas o resultados confidenciales
- Intentos de secuestro de sesión (*session hijacking*)

Atlas utiliza TLS 1.2+ de forma predeterminada, lo que representa el estándar de seguridad actual para comunicaciones cifradas. Esta implementación garantiza que sus datos permanezcan protegidos de acuerdo con las prácticas de seguridad modernas, sin requerir ninguna configuración adicional de su parte.

Al hacer que el cifrado en tránsito no sea opcional, Atlas elimina una brecha de seguridad común y garantiza que sus datos estén siempre protegidos a medida que se mueven entre sistemas, lo que le brinda una preocupación de seguridad menos de la que ocuparse.

#### Cifrado en reposo

¿Qué sucede con sus datos cuando se almacenan en el disco? ¿Siguen estando protegidos contra el acceso no autorizado? Con el cifrado en reposo de Atlas, puede estar seguro de que sus datos permanecerán seguros incluso cuando no se estén procesando activamente.

**¿Qué es el cifrado en reposo?**
El cifrado en reposo protege sus datos mientras están almacenados en medios físicos: discos duros, SSD o almacenamiento de respaldo. Garantiza que si alguien obtiene acceso físico a los dispositivos de almacenamiento o extrae archivos de datos sin procesar, no pueda acceder al contenido real sin la debida autorización y claves de descifrado.

En MongoDB Atlas, se aplica lo siguiente para el cifrado en reposo:
- Habilitado de forma predeterminada en todos los clústeres
- Implementado mediante el cifrado estándar de la industria AES-256
- Aplicado a todos los archivos de datos, índices y registros

Esto garantiza una protección completa de sus datos almacenados sin requerir ninguna configuración o gestión adicional de su parte.

**Cómo implementa Atlas el cifrado en reposo:**
Atlas utiliza una arquitectura de cifrado de dos niveles:
1. *Cifrado a nivel de volumen:* El cifrado de disco de su proveedor de la nube protege todo el volumen de almacenamiento
2. *Cifrado a nivel de base de datos:* MongoDB cifra archivos de datos individuales antes de que se escriban en el disco

Este enfoque por capas proporciona defensa en profundidad, garantizando que sus datos permanezcan protegidos incluso si una capa de seguridad se ve comprometida.

#### Traiga su propia clave (Bring Your Own Key o BYOK)

Para organizaciones con estrictos requisitos de seguridad o cumplimiento normativo, Atlas ofrece cifrado Bring Your Own Key (BYOK). Esta característica le brinda control total sobre sus claves de cifrado al permitirle hacer lo siguiente:

- Utilizar sus propias claves de cifrado almacenadas en el servicio de administración de claves de su proveedor de nube (AWS KMS, Azure Key Vault o Google Cloud KMS)
- Administrar la rotación de claves según sus políticas de seguridad
- Revocar el acceso a los datos revocando la clave, lo que proporciona un "interruptor de apagado" (*kill switch*) de emergencia si es necesario

Así es como BYOK mejora su postura de seguridad:

```text
Customer Key Management Service → Master Key Encryption → Data Encryption Keys → Encrypted Data
```

Con BYOK, usted mantiene el control de la clave maestra que cifra todas las claves de cifrado de datos. Si revoca el acceso a esta clave, los datos se vuelven inaccesibles: incluso para MongoDB.

Al implementar el cifrado en reposo con BYOK opcional, Atlas garantiza que sus datos permanezcan seguros durante todo su ciclo de vida, protegiéndolos contra el acceso no autorizado incluso si los medios de almacenamiento físico se ven comprometidos. Recuerde: perder su clave significa perder sus datos.

#### Cifrado en uso (*Encryption in use*)

Con Atlas, puede aprovechar el CSFLE y el Queryable Encryption de MongoDB, que cubrimos anteriormente.

#### Mejora de la seguridad de la red

Más allá del cifrado, proteger las rutas de red a su base de datos es importante para una estrategia de seguridad integral. MongoDB Atlas ofrece múltiples enfoques para la seguridad de la red, cada uno de los cuales proporciona diferentes niveles de aislamiento y protección. Exploremos cómo puede mejorar la seguridad de las conexiones de red de su base de datos.

#### Emparejamiento de redes (*Network peering*)

¿Alguna vez ha necesitado conectar de forma segura la infraestructura de su aplicación a su base de datos sin exponerla a la Internet pública? El emparejamiento de red crea una conexión privada directa entre su nube privada virtual (VPC) y la VPC de Atlas.

Con el emparejamiento de red, el tráfico fluye directamente entre las dos redes, eludiendo por completo la Internet pública. Este enfoque hace lo siguiente:

- Reduce la latencia de la red creando una ruta más directa
- Mejora la seguridad manteniendo el tráfico fuera de la Internet pública
- Permite la comunicación por IP privada entre sus recursos y Atlas
- Mantiene el aislamiento de la red de otros clientes de Atlas

La configuración del emparejamiento de red es sencilla a través de la interfaz de usuario o la API de Atlas. El proceso implica crear una conexión de emparejamiento en su proveedor de la nube y aceptarla en Atlas, con actualizaciones automáticas de la tabla de enrutamiento para permitir la comunicación.

Considere el emparejamiento de red en los siguientes casos:

- Necesita el máximo rendimiento para aplicaciones de alto rendimiento
- Desea utilizar direccionamiento IP privado para las conexiones de su base de datos
- Tiene requisitos de cumplimiento que requieren comunicación de red privada
- Opera en el mismo proveedor de la nube y región que su despliegue de Atlas

#### Puntos de enlace privados (*Private endpoints*)

¿Qué sucede si necesita los beneficios de seguridad de las redes privadas pero no puede configurar el emparejamiento de VPC? Los puntos de enlace privados proporcionan una alternativa más sencilla, creando una conexión privada a sus clústeres de Atlas sin requerir un emparejamiento de VPC completo.

Los puntos de enlace privados funcionan estableciendo un enlace privado entre su VPC y Atlas utilizando el servicio de conectividad privada de su proveedor de la nube:

- **AWS:** AWS PrivateLink
- **Azure:** Azure Private Link
- **Google Cloud:** Private Service Connect

Las ventajas clave de los puntos de enlace privados incluyen las siguientes:

- Configuración más sencilla que el emparejamiento completo de VPC
- Soporte para conectividad entre nubes
- Control de acceso más granular
- Sin necesidad de direcciones IP superpuestas ni modificaciones en la tabla de rutas

Los puntos de enlace privados son ideales en los siguientes casos:

- Necesita conectividad privada con una configuración mínima
- Sus aplicaciones abarcan múltiples proveedores de nube
- Requiere múltiples conexiones privadas a la misma implementación de Atlas
- Tiene permisos limitados para modificar configuraciones de VPC

#### Lista de acceso IP (*IP access list*)

Atlas proporciona listas de acceso IP como una medida de seguridad básica pero eficaz. Esta función restringe el acceso a la base de datos a direcciones IP específicas o bloques CIDR que usted permite explícitamente.

Las listas de acceso IP controlan qué conexiones pueden acceder a la base de datos:

- Solo las direcciones IP enumeradas pueden establecer conexiones con sus clústeres
- Puede agregar entradas de acceso temporal que caducan automáticamente
- Se admiten tanto direcciones IP individuales como rangos CIDR
- Los cambios surten efecto inmediatamente para un control de acceso sólido

Si bien son menos seguras que las opciones de redes privadas, las listas de acceso IP brindan protección al hacer lo siguiente:

- Evitar intentos de conexión desde fuentes desconocidas
- Limitar la superficie de ataque para intentos de fuerza bruta
- Crear una capa adicional de defensa más allá de la autenticación
- Proporcionar acceso temporal para necesidades de mantenimiento o desarrollo

Las listas de acceso IP son particularmente útiles en los siguientes casos:

- Entornos de desarrollo con necesidades de acceso cambiantes
- Escenarios de trabajo remoto que requieren acceso temporal
- Agregar una capa de seguridad a aplicaciones orientadas al público
- Implementación rápida de seguridad de red básica

Las listas de acceso IP ahora se pueden bloquear, lo que ayuda a evitar cambios accidentales en quién puede acceder a sus clústeres de Atlas.

Al combinar estos enfoques de seguridad de red, o seleccionar el que mejor se adapte a sus necesidades, puede mejorar significativamente la postura de seguridad de su implementación de Atlas, garantizando que solo los sistemas autorizados puedan intentar conectarse a su base de datos.

#### Capacidades de auditoría

La auditoría es una característica vital en MongoDB Atlas que ayuda a garantizar la seguridad, el cumplimiento y la transparencia de las operaciones de su base de datos. Con múltiples herramientas disponibles para monitorear las actividades de los usuarios y los eventos del sistema, Atlas permite a los administradores mantener un control estricto sobre sus entornos.

#### Registro de auditoría (*Audit log*)

El registro de auditoría en MongoDB Atlas está diseñado para capturar y registrar eventos clave para ayudarle a monitorear y analizar acciones operativas y administrativas. Disponible para clústeres de tamaño M10 y mayores, los registros de auditoría rastrean actividades como operaciones CRUD, intentos de autenticación y cambios de configuración como actualizaciones de claves de cifrado o roles de usuario. Puede aplicar filtros de auditoría para centrarse en eventos específicos, asegurándose de capturar solo la información más relevante para sus necesidades de cumplimiento o seguridad. Estos registros se almacenan de forma segura dentro de Atlas, preservando su integridad y accesibilidad para análisis posteriores.

#### Feeds de actividad (*Activity feeds*)

MongoDB Atlas también proporciona feeds de actividad que ofrecen información visual en tiempo real sobre eventos a nivel de sistema y específicos del proyecto:

- **Feed de actividad de la organización:** Realiza un seguimiento de las actividades de alto nivel a nivel organizacional, como actualizaciones de facturación, gestión de equipos o cambios administrativos.
- **Feed de actividad del proyecto:** Se centra en eventos dentro de proyectos individuales de Atlas, cubriendo tareas como configuraciones de bases de datos, operaciones de clústeres e intentos de autenticación de usuarios.

Estos feeds permiten a los administradores revisar y monitorear rápidamente las acciones dentro de sus implementaciones sin necesidad de profundizar en registros detallados. La capacidad de distinguir entre actividades de toda la organización y actividades centradas en proyectos garantiza que se pueda acceder a la información relevante con el mínimo esfuerzo.

#### Uso conjunto de registros de auditoría y feeds de actividad

Al aprovechar juntos los registros de auditoría y los feeds de actividad, los administradores obtienen perspectivas tanto granulares como de alto nivel sobre la actividad de la base de datos. Los registros de auditoría proporcionan información detallada sobre eventos específicos, mientras que los feeds de actividad ayudan a identificar tendencias o patrones más amplios en el comportamiento del sistema. Juntas, estas herramientas permiten a las organizaciones cumplir con los requisitos reglamentarios, detectar posibles riesgos de seguridad y mantener la claridad operativa en sus entornos de bases de datos en la nube.

#### Configuración de la auditoría en Atlas

Atlas simplifica la implementación de la auditoría. Disponible en clústeres M10+, las funciones de auditoría se pueden adaptar a sus necesidades específicas:

- **Auditoría de autenticación predeterminada:** Habilitada automáticamente para todos los clústeres M10+, rastreando intentos de autenticación sin ninguna configuración adicional
- **Filtros de auditoría personalizados:** Cree filtros con formato JSON para especificar exactamente qué eventos desea rastrear, enfocando sus registros de auditoría en las operaciones más críticas para la seguridad
- **Acceso al registro de auditoría:** Recupere y analice registros a través de la interfaz de usuario, la API o la CLI de Atlas con 30 días de retención

#### La conexión con el cumplimiento normativo

Para las industrias reguladas, la auditoría no es sólo útil; es obligatoria. Las capacidades de auditoría de Atlas ayudan a satisfacer los requisitos de múltiples marcos de cumplimiento:

- Requisitos SOC-2 para monitorear el acceso al sistema
- Mandatos de HIPAA para rastrear quién accedió a PHI
- Requisitos de PCI DSS para el monitoreo de la actividad de los usuarios
- Disposiciones de GDPR para demostrar medidas de protección de datos

Al implementar una auditoría integral en Atlas, crea un registro verificable de la actividad de la base de datos que protege sus datos y demuestra el cumplimiento ante auditores y reguladores.

Recuerde que la auditoría introduce cierta sobrecarga en el rendimiento, así que configure sus filtros para capturar lo necesario para la seguridad y el cumplimiento sin detalles excesivos que puedan afectar el rendimiento. Con una auditoría cuidadosamente implementada, obtiene visibilidad de las operaciones de la base de datos mientras mantiene el rendimiento que requieren sus aplicaciones.

#### Protección de su despliegue de Atlas: Mejores prácticas

Ahora que hemos explorado las funciones de seguridad integrales de MongoDB Atlas, ¿cómo reunimos todo para crear una estrategia de seguridad holística? Examinemos algunas mejores prácticas y recomendaciones prácticas para maximizar su postura de seguridad en Atlas.

Las estrategias de seguridad más efectivas implementan múltiples capas de protección: ninguna medida de seguridad es suficiente por sí sola. Para MongoDB Atlas, considere implementar lo siguiente:

- **Defensa en profundidad:** Combine seguridad de red, cifrado y controles de acceso para crear capas superpuestas de protección
- **Principio de mínimo privilegio:** Otorgue a los usuarios y aplicaciones solo los permisos que necesitan absolutamente
- **Arquitectura de confianza cero (*Zero-trust architecture*):** Verifique cada solicitud de acceso independientemente de la fuente, sin otorgar confianza implícita

#### Lista de verificación de implementación de seguridad

Comience con esta práctica lista de verificación para mejorar la seguridad de su implementación de Atlas:

- **Base de seguridad de red:**
  - Utilice puntos de enlace privados o emparejamiento de VPC siempre que sea posible
  - Restrinja las listas de acceso IP a direcciones específicas en lugar de rangos CIDR amplios
- **Autenticación y control de acceso:**
  - Implemente políticas de contraseñas seguras para los usuarios de la base de datos
  - Utilice la federación OIDC con su proveedor de identidades para una gestión centralizada de usuarios
  - Cree controles de acceso basados en roles con roles personalizados para permisos detallados
- **Protección de datos:**
  - Considere BYOK para entornos de producción confidenciales
  - Identifique campos que contienen datos confidenciales e implemente CSFLE o Queryable Encryption según corresponda
  - Rote las claves de cifrado con regularidad según sus políticas de seguridad
- **Monitoreo y detección:**
  - Configure alertas para patrones sospechosos de autenticación o acceso
  - Implemente filtros de auditoría personalizados para colecciones y operaciones críticas
  - Revise periódicamente los registros de auditoría en busca de actividades inesperadas

Al implementar metódicamente estas medidas de seguridad, creará una postura de seguridad sólida que protegerá sus datos y al mismo tiempo permitirá que sus aplicaciones funcionen de manera eficiente y segura. Recuerde que la seguridad es un viaje, no un destino; la mejora continua y la vigilancia son clave para mantener su postura de seguridad frente a las amenazas en evolución.

---

### Sección 9: Servicios obsoletos en Atlas

A medida que MongoDB Atlas continúa creciendo y adaptándose a las necesidades cambiantes de los clientes, a veces nos enfrentamos a la difícil decisión de dar por obsoletos ciertos servicios. Estas decisiones no se toman a la ligera; implican una cuidadosa consideración de los patrones de uso, los comentarios de los clientes y la dirección estratégica.

Para el **30 de septiembre de 2025**, varias características de MongoDB Atlas llegarán al final de su vida útil y se eliminarán de la plataforma. Si utiliza alguno de estos servicios, es importante comprender qué está cambiando y planificar su estrategia de migración con suficiente antelación.

Las características afectadas incluyen las siguientes:

- Atlas Data API y puntos de enlace HTTPS personalizados
- Atlas Device Sync
- SDK de Atlas Device (Realm)
- Atlas Data Lake (versión preliminar / *preview*)

Además, Atlas Edge Server (*preview*) se suspendió en septiembre de 2024.

#### Clústeres sin servidor (*Serverless clusters*)

Los clústeres sin servidor se diseñaron para proporcionar una experiencia de base de datos bajo demanda con escalado automático según la carga de trabajo. Sin embargo, estos clústeres pasarán al nivel de clúster Flex, que ofrece lo siguiente:

- Capacidades de escalado automático similares
- Modelo de precios más predecible
- Mayor control sobre la asignación de recursos
- Rendimiento mejorado para cargas de trabajo variadas

Si actualmente utiliza clústeres sin servidor, sus instancias se migrarán automáticamente al nivel Flex antes de la fecha de finalización de su vida útil. Esta transición debería ser fluida, con un impacto mínimo en sus aplicaciones y cargas de trabajo.

Antes de que estas obsolescencias surtan efecto, aquí hay algunas consideraciones clave a tener en cuenta:

- **Evalúe su uso actual:** Haga un inventario de qué servicios obsoletos está utilizando activamente
- **Evalúe alternativas con anticipación:** Comience a probar posibles soluciones de reemplazo mucho antes de la fecha de finalización de su vida útil
- **Planifique los costos de migración:** Presupueste el tiempo de desarrollo y los posibles cambios de infraestructura
- **Considere los impactos arquitectónicos:** Algunas alternativas pueden requerir ajustes en la arquitectura de su aplicación
- **Pruebe exhaustivamente:** Asegúrese de que las alternativas elegidas cumplan con sus requisitos de rendimiento y confiabilidad
- **Comuníquese con las partes interesadas:** Mantenga a su equipo informado sobre los próximos cambios y planes de migración

---

### Sección 10: Próximos pasos

Recuerde que el equipo de soporte de MongoDB está disponible para guiarle a través de estas transiciones. No dude en comunicarse a través del Centro de soporte de MongoDB para obtener ayuda con su estrategia de migración.

Al planificar su migración para abandonar los servicios obsoletos de Atlas, considere estos factores clave:

- **Evaluación del cronograma:** Cree un calendario realista que complete la migración mucho antes de la fecha límite del 30 de septiembre de 2025
- **Planificación de recursos:** Asigne los recursos de desarrollo adecuados para cada componente afectado
- **Mapeo de dependencias:** Identifique todos los puntos de integración con servicios obsoletos en toda la pila de su aplicación
- **Estrategia de prueba:** Desarrolle un plan de prueba integral que cubra todas las fases de migración
- **Opciones de respaldo (*Fallback options*):** Prepare planes de contingencia en caso de que encuentre desafíos inesperados

Al comenzar su planificación de migración de manera temprana, aprovechar los recursos disponibles y trabajar estrechamente con el equipo de soporte de MongoDB, puede garantizar una transición sin problemas a soluciones alternativas. Recuerde que MongoDB y su ecosistema de socios están desarrollando activamente recursos para que estas migraciones sean lo más fluidas posible.

Ya sea que elija crear soluciones personalizadas utilizando controladores de MongoDB, aprovechar arquitecturas sin servidor o adoptar ofertas de socios, la clave es comenzar su proceso de evaluación ahora para disponer de suficiente tiempo para la implementación y las pruebas antes de que lleguen los plazos de obsolescencia.

Para obtener la información y orientación más actualizada, no dude en ponerse en contacto con el soporte técnico de MongoDB. Pueden proporcionarle recomendaciones basadas en sus casos de uso y requisitos específicos.

---

### Sección 11: Resumen

En este capítulo, hemos explorado MongoDB Atlas y su papel como una solución integral de gestión de datos para desarrolladores modernos. Comenzamos configurando una organización y examinando las diversas herramientas disponibles para interactuar con los despliegues de Atlas. Estas herramientas proporcionan la base para gestionar eficientemente su entorno de datos.

Luego, analizamos las consideraciones importantes para dimensionar un clúster de Atlas, asegurando que pueda tomar decisiones informadas basadas en las necesidades específicas de su aplicación. A partir de esto, exploramos cómo Atlas maneja el escalado, tanto automáticamente como bajo demanda, permitiendo que su base de datos crezca junto con su aplicación sin interrupciones del servicio.

La seguridad formó una parte importante de nuestra discusión, cubriendo tanto los mecanismos de autenticación como de autorización. Examinamos cómo implementar controles de acceso adecuados y los métodos disponibles para proteger su despliegue de Atlas contra el acceso no autorizado y posibles amenazas.

Finalmente, revisamos los servicios obsoletos en Atlas, ayudándole a planificar futuros cambios y actualizaciones en la plataforma.

MongoDB Atlas se destaca como más que un simple servicio de base de datos; es una plataforma integrada que admite diversas cargas de trabajo al tiempo que atiende las necesidades de los equipos de desarrollo en organizaciones de todos los tamaños. Desde empresas emergentes hasta grandes corporaciones, Atlas proporciona las herramientas necesarias para crear, escalar y mantener aplicaciones modernas de manera eficiente.

A medida que continúa su viaje con MongoDB, le recomendamos que experimente con las funciones que hemos analizado. Comience poco a poco, pruebe en entornos que no sean de producción e incorpore gradualmente estas potentes capacidades en sus sistemas de producción. Cuanto más explore Atlas, mejor comprenderá cómo puede ayudar a resolver sus desafíos de datos únicos.

¿Qué construirá con MongoDB Atlas?
