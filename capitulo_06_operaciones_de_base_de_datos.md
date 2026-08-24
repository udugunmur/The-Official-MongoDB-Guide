# Parte 6: Operaciones de Base de Datos

## Capítulo 6: Operaciones de Base de Datos

La confiabilidad y la seguridad son factores clave que distinguen a un sistema de base de datos listo para producción de uno en desarrollo. Con las herramientas adecuadas, puede optimizar el rendimiento, solucionar posibles interrupciones, escalar recursos, actualizar flujos de trabajo y evitar amenazas antes de que afecten a sus usuarios. Asegurarse de contar con estas herramientas desde el inicio del desarrollo es el camino más seguro hacia una implementación sin problemas.

MongoDB y sus servicios de despliegue administrados asociados proporcionan paquetes robustos de monitoreo, respaldo y auditoría para simplificar el mantenimiento operativo. En este capítulo, explorará la importancia de las herramientas operativas; estrategias para garantizar un rendimiento constante, copias de seguridad precisas y restauraciones rápidas, y cómo configurar, solucionar problemas y analizar la auditoría y el registro (*logging*).

Este capítulo cubrirá los siguientes temas:

- La importancia del monitoreo de bases de datos
- Métricas clave para monitorear en MongoDB
- Herramientas de informes y monitoreo
- Métodos de copia de seguridad y recuperación de clústeres
- Principios de autenticación de bases de datos
- Monitoreo y copias de seguridad avanzadas

---

### Sección 1: Requisitos técnicos

Para seguir este capítulo, necesitará tener un clúster de MongoDB. Su clúster de MongoDB puede ser una instalación autogestionada o una administrada por MongoDB Atlas, MongoDB Cloud Manager o MongoDB Ops Manager. Donde se requiera una herramienta específica, las instrucciones lo especificarán.

---

### Sección 2: Monitoreo

> “Lo que no se mide no se puede mejorar.” – William Thomson, Lord Kelvin

El monitoreo es la base de cualquier estrategia proactiva de rendimiento de bases de datos. Por muy riguroso y detallado que sea el diseño inicial de su sistema, los patrones de uso de las personas reales siempre le sorprenderán. A falta de un monitoreo cuidadoso y constante, tendrá que adoptar una postura reactiva, reasignando recursos y rediseñando los flujos de trabajo solo después de que ya hayan impactado a los usuarios.

MongoDB permite un enfoque proactivo con un conjunto de herramientas de monitoreo repleto de funciones que ofrecen información granular sobre cómo se utiliza su sistema en el mundo real, con qué eficacia su arquitectura toma esto en cuenta y dónde tiene margen de mejora.

#### ¿Por qué monitorear?

Antes de analizar aspectos específicos del monitoreo, establezcamos qué queremos lograr con nuestras herramientas de monitoreo. Esto ayudará a motivar nuestras sugerencias estratégicas específicas y le brindará un marco dentro del cual diseñar sus propios refinamientos específicos para su caso de uso. Nuestros objetivos son los siguientes:

- **Detección temprana:** Piense en el conjunto de monitoreo como su sistema de alerta temprana. Mantener una vigilancia constante sobre las métricas clave le permite anticipar problemas e intervenir antes de que afecten a los usuarios. La proactividad ahorra tiempo, esfuerzo y frustración a los usuarios en el futuro.
- **Escalado preventivo:** Si la carga en su sistema aumenta sin un incremento correspondiente en la asignación de recursos, sus usuarios experimentarán interrupciones en el servicio que pueden provocar la pérdida de clientes. Incorpore métricas para planificar y ejecutar el escalado vertical o hacia arriba con anticipación.
- **Gestión de costos:** El mismo monitoreo que le indica cuándo escalar verticalmente puede indicarle cuándo su sistema está sobreaprovisionado y es hora de reducir la escala, optimizando los costos.
- **Optimización del rendimiento:** El funcionamiento eficiente de la base de datos es más que simplemente contar con recursos computacionales suficientes; también se trata de los índices y de las operaciones de recuperación en sí mismas. El monitoreo ayuda a identificar y corregir cuellos de botella específicos en las consultas.
- **Integridad de los datos:** El monitoreo regular garantiza que los procesos de copia de seguridad se comporten como se espera y que los datos permanezcan consistentes, precisos y disponibles.

Con estos objetivos en mente, veamos cómo MongoDB ayuda a lograrlos.

#### Monitoreo del clúster

El monitoreo en MongoDB ocurre principalmente a nivel del clúster, el componente fundamental de cualquier despliegue de base de datos respaldado por MongoDB. Cuando diseña e implementa su aplicación por primera vez, el uso de su clúster refleja sus suposiciones sobre el comportamiento y los requisitos de los usuarios; una estrategia de monitoreo sólida puede validar o corregir estas suposiciones y servir como base para futuros ajustes de diseño.

La Figura 6.1 ilustra varias métricas del rendimiento del clúster como se ve en MongoDB Atlas:

> **Figura 6.1:** Métricas de MongoDB Atlas

#### Métricas de memoria

Una de las métricas más importantes en MongoDB es el uso de memoria. Como todos los sistemas de bases de datos, MongoDB aprovecha la memoria del sistema para mejorar el rendimiento, lo que la convierte en una prioridad máxima para el monitoreo. Esta sección tiene como objetivo proporcionar resúmenes de alto nivel de conceptos clave de memoria y problemas comunes relacionados con cada concepto, así como estrategias de monitoreo y corrección para esos problemas.

#### Fallos de página (*Page faults*)

La mayoría de los datos en un sistema determinado normalmente residen "en disco", es decir, en unidades de disco duro (HDD) o unidades de estado sólido (SSD). Este es un valor predeterminado sensato, ya que estos medios son significativamente más asequibles por GB de almacenamiento y no son volátiles, lo que significa que los datos escritos en ellos persisten a menos que se eliminen explícitamente. Sin embargo, leer datos del disco es prohibitivamente lento para sistemas que reciben miles o millones de solicitudes por segundo.

Por el contrario, la memoria RAM proporciona velocidades de acceso adecuadas para aplicaciones de tráfico tan pesado, pero es mucho más costosa por GB y es volátil, lo que significa que no conserva los datos durante una interrupción del servicio. En consecuencia, MongoDB emplea una estrategia híbrida para recuperar datos:

1. Recibir una consulta.
2. Buscar en la RAM los datos solicitados.
3. Si se encuentran, devolver los datos a la aplicación; de lo contrario, continuar.
4. Buscar en el disco los datos solicitados.
5. Si se encuentran, cargar los datos en la RAM y devolverlos a la aplicación.

Cuando su sistema tiene que recuperar datos de su disco, se llama fallo de página (*page fault*), haciendo referencia a que los datos en la RAM están estructurados en páginas. Con cada fallo de página, los datos recién cargados consumen su reserva de RAM libre. Cuando su aplicación agota esta reserva, el sistema operativo descarta páginas, lo que se denomina un evento de expulsión de página (*page eviction*), para liberar espacio para datos adicionales más adelante.

Las expulsiones de páginas pueden aumentar la carga de E/S de su sistema; la próxima vez que se necesite una página que fue expulsada para una consulta, la base de datos tendrá que realizar una lectura lenta del disco nuevamente. Esto, a su vez, puede provocar que la latencia de las consultas se dispare y que el rendimiento de la CPU caiga mientras el sistema espera a que se completen las lecturas del disco.

Si bien es imposible eliminar por completo los fallos de página, especialmente con conjuntos de datos dinámicos, debe reducir su aparición tanto como sea posible. Una estrategia poderosa son las pruebas de estrés (*stress testing*). En este enfoque, crea un entorno de prueba que refleja su configuración en vivo y utiliza este entorno para llevar su sistema a sus límites. Realice un seguimiento de cuántos fallos de página puede manejar su sistema antes de que se degrade el rendimiento y compare los datos de las pruebas de estrés con las métricas reales de fallos de página en su sistema en vivo para determinar su margen de recursos disponible.

Para recopilar los datos relevantes de un despliegue autogestionado, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['extra_info']
```

Luego, puede consultar `extra_info.page_faults`. Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico de métricas de **Page Faults**.

También puede reducir los fallos de página asegurándose de que el conjunto de trabajo (*working set*), es decir, sus datos a los que accede con mayor frecuencia, permanezca en la RAM tanto como sea posible. Una estrategia práctica para lograr esto es refinar su esquema para que los documentos individuales sean más pequeños. Si observa fallos de página excesivos en un clúster que aloja una colección de documentos muy grandes, considere descomponer los documentos grandes en múltiples documentos más pequeños según los campos a los que se debe acceder juntos, y utilizar referencias de documentos para incrustar las relaciones entre las colecciones resultantes.

#### Memoria residente (*Resident memory*)

La memoria residente, es decir, la porción de RAM disponible que MongoDB utiliza activamente, es uno de los indicadores más sólidos de la idoneidad de la arquitectura de su base de datos. Si bien aprovechar las operaciones de alta velocidad de la memoria RAM es la forma más fácil de mejorar el rendimiento de la base de datos, debe tener cuidado de reservar suficiente memoria RAM para otros procesos del sistema.

Como regla general, la memoria residente no debe comprender más del 80% de la memoria total disponible. El riesgo de usar más es que obliga al sistema operativo a depender del espacio de intercambio (*swap space*), operando sobre datos en disco en lugar de en memoria, lo que ralentiza drásticamente su sistema. Si descubre que su base de datos se acerca regularmente o supera el umbral del 80%, considere asignar más RAM e investigar el diseño de su base de datos para posibles ahorros de memoria.

Para recopilar los datos relevantes de un despliegue autogestionado, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['mem']
```

Luego, puede consultar `mem.resident`. Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico de métricas de **Memory**.

#### Memoria virtual (*Virtual memory*)

La memoria virtual es una asignación de espacio correspondiente a una cierta cantidad de RAM física, espacio en disco o una combinación de ambos, con direcciones virtuales para acceder a los datos subyacentes. Esta es una abstracción de su sistema operativo para poner un límite general a la cantidad de memoria que pueden utilizar sus aplicaciones. Además de utilizar memoria virtual para operaciones ordinarias, MongoDB asigna una dirección virtual independiente para el registro en diario (*journaling*) cuando está habilitado.

Su sistema operativo rechazará solicitudes de mayor memoria virtual si la carga de trabajo de su aplicación actual ha alcanzado el límite. En consecuencia, monitorear el consumo de memoria virtual es una parte importante de su estrategia de implementación.

Para recopilar los datos relevantes de un despliegue autogestionado, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['mem']
```

Luego, puede consultar `mem.virtual`. Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico de métricas de **Memory**. Si sus patrones de uso están superando los límites de la memoria virtual disponible, debe escalar la memoria RAM de su sistema para aumentar la memoria virtual disponible.

Recomendamos encarecidamente no desactivar el *journaling*; si bien esto puede reducir significativamente su huella de memoria virtual, compromete la integridad de los datos y la recuperabilidad del sistema. A menos que haya analizado a fondo estos riesgos y haya desarrollado y probado una estrategia de mitigación específica, esto no debería considerarse una opción viable.

Asimismo, desaconsejamos aumentar el límite de memoria virtual a nivel del sistema operativo como una estrategia a largo plazo. Si bien puede servir como una solución de emergencia a corto plazo si no puede ampliar su hardware por cualquier motivo, un límite de memoria virtual elevado introduce los siguientes desafíos:

- **Mayor actividad de intercambio (*swap*):** Una mayor asignación de memoria virtual provendrá principalmente del espacio en disco, lo que significa que su sistema intercambiará operaciones al disco con mayor frecuencia. Esto da como resultado una degradación significativa del rendimiento.
- **Sobrecarga de intercambio (*Thrashing*):** Bajo cargas de trabajo pesadas, es posible que el sistema tenga que asignar más recursos a administrar páginas de memoria que a las operaciones deseadas.
- **Eliminación de procesos (*Process killing*):** Los sistemas operativos bajo una demanda extrema de memoria y que dependen en gran medida del intercambio pueden eliminar procesos de manera impredecible.
- **Tiempos de espera de transacciones (*Transaction timeouts*):** Las transacciones de larga duración tienen más probabilidades de agotar el tiempo de espera en condiciones de intercambio intensivo.

Si debe adoptar esta estrategia para responder a picos en la actividad de los usuarios, debe priorizar el escalado de hardware lo antes posible.

#### WiredTiger

Los motores de almacenamiento son los componentes de los sistemas de bases de datos responsables de la lectura, escritura y manipulación de datos, incluida la indexación, las consultas, las actualizaciones y el procesamiento de transacciones. Debido a que los motores de almacenamiento determinan la forma física de los datos en la memoria y en el disco, se pueden optimizar diferentes motores de almacenamiento para producir patrones de acceso físico más adecuados para cargas de trabajo específicas.

De forma predeterminada, MongoDB utiliza el motor de almacenamiento WiredTiger. WiredTiger establece una caché de memoria interna, que normalmente es igual a la mitad de su RAM disponible menos 1 GB. Por ejemplo, si tiene una máquina host con 16 GB de RAM, la caché de WiredTiger sería `(16 - 1) / 2`, o 7.5 GB, utilizando la configuración predeterminada. Luego, MongoDB preasigna memoria para manejar conexiones y operaciones de ordenación. Puede ajustar la caché de WiredTiger utilizando el parámetro `storage.wiredTiger.engineConfig.cacheSizeGB`.

Para la mayoría de los casos de uso, el tamaño de caché predeterminado es el mejor. Sin embargo, si maneja una gran proporción de datos fuertemente comprimidos, puede considerar reducir la asignación de caché para liberar memoria para otros procesos. Monitoree cuidadosamente los patrones de uso y rendimiento antes de realizar dicho ajuste para determinar un tamaño de caché apropiado y continúe observando después del ajuste para determinar su viabilidad a largo plazo.

#### Motor de almacenamiento en memoria (*In-memory storage engine*)

Como alternativa a WiredTiger, las compilaciones de 64 bits de MongoDB también admiten un motor de almacenamiento en memoria que, como su nombre indica, no mantiene datos en el disco. Esto le permite eludir la E/S de disco, lo que genera una latencia más predecible. De manera similar a la caché de WiredTiger, este motor de almacenamiento utiliza de forma predeterminada la mitad de la RAM disponible menos 1 GB.

Este motor de almacenamiento no conserva datos de ninguna manera; en consecuencia, es mejor en instancias que sirven como punto de contacto del usuario con datos a los que se accede con frecuencia. Estas instancias pueden luego ser respaldadas por otra instancia en el mismo clúster que ejecute el motor de almacenamiento WiredTiger y maneje la persistencia de datos.

#### Métricas de E/S (*I/O metrics*)

Las métricas de E/S a menudo pueden servir como indicadores útiles para problemas emergentes relacionados con la memoria. Monitorearlas de cerca puede permitirle anticipar la degradación del rendimiento e implementar estrategias de remediación lo antes posible.

#### Esperas y colas (*Waits and queues*)

La espera de E/S (*I/O wait*) se refiere a cuánto tiempo espera un sistema operativo para que se complete una operación de E/S determinada y puede correlacionarse con los fallos de página. El aumento de la espera de E/S con el tiempo probablemente indica que su sistema está lidiando con un volumen creciente de fallos de página. Puede utilizar herramientas de monitoreo integradas en su sistema operativo, como `top`, para verificar su tiempo de espera de E/S.

A su vez, estos períodos de espera de E/S pueden provocar acumulaciones de solicitudes de lectura y escritura. Si bien el monitoreo de la memoria es el núcleo de su estrategia de monitoreo proactiva, estas métricas de E/S pueden considerarse el núcleo de su estrategia de monitoreo reactiva. Un aumento sostenido en los tiempos de espera de E/S y en las colas de lectura y escritura es una señal clara de que se requiere una intervención activa.

Para recopilar datos sobre sus colas de lectura y escritura desde un despliegue autogestionado, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['queues']
```

Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico de métricas de **Queues**.

#### Espacio libre en disco

Las bases de datos comúnmente se quedan sin espacio libre en disco a medida que se acumulan los datos de los usuarios. Configurar alertas para cuando alcance hitos de uso de disco como 40%, 60% y 80% es crucial para ayudarlo a mantenerse al tanto de su escalado de hardware, especialmente para sistemas que agregan grandes cantidades de datos rápidamente.

Además de las necesidades puras de almacenamiento, el espacio libre en disco también es esencial para una indexación adecuada. MongoDB finaliza automáticamente las compilaciones de índices si no tiene suficiente espacio en disco. Puede ver el uso de disco de MongoDB en MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas navegando a la página de métricas de su despliegue y consultando el gráfico de métricas de **Disk Space Used**.

Para comprobar específicamente si el espacio limitado en disco está provocando que fallen las compilaciones de índices, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['indexBuilds']
```

Luego, puede consultar la propiedad `indexBuilds.killedDueToInsufficientDiskSpace`.

Debería considerar hacer de la compactación (*compaction*) parte de su estrategia de gestión del espacio en disco. Con el tiempo, las operaciones de eliminación y actualización, junto con las reconstrucciones de índices, pueden hacer que los bloques de almacenamiento de su colección queden sin uso. MongoDB y MongoDB Atlas proporcionan el comando `compact`, que analiza su base de datos en busca de bloques obsoletos que no se hayan devuelto al sistema operativo, consolida sus datos en bloques contiguos y libera los bloques no utilizados.

---

### Sección 3: Monitoreo de replicación

La replicación es una parte esencial para garantizar la continuidad del servicio en un sistema de base de datos. Los conjuntos de réplicas (*replica sets*) almacenan su estado sincronizado en el registro de operaciones (*oplog*). A medida que se lleva a cabo cada operación, se escribe en el *oplog* del nodo primario, que toma la forma de una colección limitada (*capped collection*). Los nodos secundarios transmiten este *oplog* de forma asíncrona, escribiendo operaciones en el suyo propio en el mismo orden.

Los grandes volúmenes de operaciones de escritura pueden dificultar la capacidad de su nodo primario para replicar eventos en el *oplog* en tiempo real, lo que a su vez ralentiza la replicación en instancias secundarias. El tiempo que lleva replicar una operación de escritura desde el primario a un secundario se denomina retraso de replicación (*replication lag*).

Si la hora es 4:30 P.M. y su nodo secundario acaba de aplicar una operación que se aplicó en su nodo primario a las 4:25 P.M., tiene un retraso de replicación de cinco minutos.

En un clúster de producción, el retraso de replicación debe ser cercano o igual a 0.

Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte los gráficos **Replication Oplog Window** y **Oplog GB/Hour**.

#### Tamaño del oplog

El *oplog* contiene un historial de las operaciones realizadas en su clúster de MongoDB que se utiliza principalmente para reconstruir el estado de su clúster en caso de una interrupción del servicio. Cada miembro del conjunto de réplicas de su clúster almacena una copia en `db.oplog.rs()`, de modo que si su nodo primario falla o renuncia a su rol, se puede elegir un secundario y garantizar que esté en el estado adecuado. El tamaño del *oplog* no afecta el uso de memoria.

Asegúrese de reservar suficiente espacio de almacenamiento en el disco para su *oplog* y de que el límite de tamaño que establezca para el *oplog* le permita capturar suficiente historial de operaciones para una replicación robusta a fin de protegerse contra las consecuencias del retraso de replicación. Con suficiente retraso de replicación, su sistema puede alcanzar un estado en el que el registro más antiguo que queda en el *oplog* de su nodo primario sea más nuevo que el registro más reciente que ha procesado el nodo secundario. Esta brecha en la continuidad entre los *oplogs* representa una pérdida de seguimiento de las operaciones que pueden haber ocurrido, lo que hace que el secundario detenga la replicación. Cuanto más pequeño sea el *oplog*, más probable será que surja esta situación.

Como regla general, debe dimensionar su *oplog* para almacenar al menos el valor de dos días de operaciones. También debe ser más largo que el tiempo que lleva la sincronización inicial dado su ancho de banda de red y de E/S actual.

Para configurar el tamaño de su *oplog* en un despliegue autogestionado, puede utilizar lo siguiente:

```javascript
db.adminCommand({ "replSetResizeOplog": 1, size: Double(<num>)})
```

Aquí, `<num>` es el tamaño en megabytes al que desea configurar el *oplog*. En MongoDB Atlas, puede configurar el tamaño del *oplog* utilizando el siguiente comando de la CLI de Atlas:

```bash
atlas clusters advancedSettings update <clusterName> --oplogSizeMB
```

---

### Sección 4: Monitoreo de red

Los patrones de actividad de la red proporcionan otro ángulo de conocimiento sobre el comportamiento del usuario y el rendimiento del sistema. El uso irregular de la red puede indicar la necesidad de rediseñar su arquitectura de almacenamiento y distribución de datos para brindar un mejor servicio a bases de usuarios geográficamente diversas. Además, los patrones de uso dependientes del tiempo pueden ayudarle a programar períodos de mantenimiento e implementación para minimizar la interrupción del servicio.

Para obtener información sobre la actividad de la red en un despliegue autogestionado, utilice lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['network']
```

Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico **Network**.

---

### Sección 5: Cursores y conexiones

Los recuentos de conexiones y cursores pueden actuar como indicadores del tráfico general de usuarios y de la capacidad de respuesta del sistema bajo carga. Esto los convierte en métricas útiles para realizar un seguimiento al evaluar y refinar su arquitectura de implementación. Demasiadas conexiones y cursores abiertos pueden degradar el rendimiento de la base de datos, y los aumentos en los cursores que han agotado el tiempo de espera reflejan cuellos de botella en el rendimiento que ya están ocurriendo. Muy pocas conexiones sugieren una base de datos infrautilizada, lo que puede justificar la reducción del tamaño de su sistema para evitar gastos excesivos.

Para obtener información sobre cursores en un despliegue autogestionado, utilice lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['metrics.cursor']
```

Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte los gráficos **Connections** y **Cursors**.

#### Contadores de operaciones de replicación (*Replication opcounters*)

El seguimiento de las operaciones CRUD es obviamente útil para discernir patrones de utilización de la base de datos, pero también puede ser una herramienta poderosa para analizar el comportamiento de red de su sistema. Específicamente, la métrica `opcountersRepl`, que rastrea la replicación de operaciones desde el *oplog* primario a los secundarios, se puede utilizar para identificar la desincronización entre nodos. Si los dos nodos secundarios en un conjunto de réplicas muestran una disparidad significativa en los recuentos de operaciones de replicación, podría indicar problemas de configuración o de ancho de banda de red entre el nodo rezagado y el primario.

Para obtener información sobre los contadores de operaciones de replicación en un despliegue autogestionado, utilice lo siguiente:

```javascript
db.adminCommand({ "serverStatus" : 1 })['opcountersRepl']
```

Si utiliza MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas, navegue a la página de métricas de su despliegue y consulte el gráfico **Opcounters - Repl**.

---

### Sección 6: Consideraciones sobre el conjunto de trabajo (*Working set*)

Tocamos brevemente el concepto del conjunto de trabajo (*working set*) anteriormente, pero merece una discusión más detallada debido a su influencia fundamental sobre el rendimiento de su base de datos. Asegurarse de que su conjunto de trabajo, es decir, sus datos a los que accede más comúnmente, quepa completamente dentro de la RAM debe ser una prioridad al diseñar su sistema para minimizar los fallos de página, las expulsiones y la sobrecarga de intercambio (*thrashing*). Esta faceta de su sistema requerirá una observación constante, pero a medida que su aplicación crezca y la base de usuarios tome una forma definida, los ajustes necesarios deberían ser cada vez más pequeños.

El desafío obvio al incorporar este conocimiento en su diseño es que, antes de tener una gran cantidad de datos del entorno de producción con los que trabajar, puede que no sea inmediatamente evidente cómo se verá el conjunto de trabajo y, por extensión, cuáles serán sus necesidades de RAM. Puede formar una estimación aproximada para comenzar a planificar su implementación sumando lo siguiente:

- Suficientes datos para satisfacer el 95% de las solicitudes de los usuarios
- Registros (*logs*) de MongoDB
- Planes de consulta almacenados en caché (*Cached query plans*)
- Un margen adicional del 50% por encima de la suma de los anteriores para tener en cuenta los índices

#### Índices y el conjunto de trabajo

Los índices son cruciales para el acceso rápido a los datos. Si no puede diseñar su implementación de manera que todo su conjunto de trabajo resida en la RAM, priorizar que sus índices residan en la RAM aún puede contribuir en gran medida a mejorar el rendimiento. Incluso si los datos reales deben recuperarse del disco, MongoDB puede localizarlos rápidamente con la ayuda de índices en la RAM.

---

### Sección 7: Consideraciones para clústeres fragmentados

Además de las técnicas descritas hasta ahora, su estrategia de monitoreo para clústeres fragmentados debe incorporar verificaciones regulares del estado del servidor de configuración (*config server*), la distribución de datos y el bloqueo (*locking*).

#### Estado del servidor de configuración (*Config server health*)

El servidor de configuración de su clúster fragmentado orquesta operaciones como la distribución de fragmentos de datos (*chunks*) y la inicialización de instancias `mongos`. Si su servidor de configuración se vuelve inaccesible, sus fragmentos se desequilibrarán gradualmente a medida que algunos se sobrecarguen con datos entrantes y otros queden infrautilizados. Para un despliegue autogestionado, puede verificar la conectividad con el servidor de configuración usando la siguiente línea:

```javascript
db.adminCommand({ "serverStatus" : 1 })['sharding']
```

`sharding.lastSeenConfigServerOpTime` indica la última vez que una instancia o fragmento `mongos` determinado ha visto una operación en el servidor de configuración. Si hay una discrepancia entre este valor y el último tiempo de operación conocido en el servidor de configuración, esto puede indicar un problema de conectividad.

MongoDB Ops Manager, MongoDB Cloud Manager y MongoDB Atlas monitorean los servidores de configuración de forma predeterminada, y puede configurar alertas relativas al estado del servidor de configuración a través de cualquiera de estas herramientas.

#### Distribución de datos

Una ventaja clave de los despliegues de clústeres fragmentados es la distribución automática y equilibrada de fragmentos de datos entre los fragmentos constituyentes. Esto es manejado por un proceso equilibrador (*balancer*) en segundo plano que observa colecciones fragmentadas y migra datos entre fragmentos cuando se cumplen los umbrales de migración, intentando alcanzar una distribución de datos lo más uniforme posible entre los fragmentos. El monitoreo de este proceso puede proporcionar información valiosa sobre la idoneidad del diseño de su sistema.

La migración de datos crea una sobrecarga adicional en el sistema. Si observa un rendimiento reducido en sus clústeres fragmentados, use el comando `sh.isBalancerRunning()` en `mongosh` para verificar si se debe a una migración activa. Si las migraciones frecuentes afectan constantemente a su clúster fragmentado, puede realizar cualquiera de las siguientes acciones:

- Aumente el tamaño del rango del clúster, lo que reduce la frecuencia de las migraciones a cambio de una distribución de datos menos uniforme.
- Establezca una ventana de equilibrio de fragmentos (*shard balancing window*), que limite al equilibrador a ejecutar migraciones solo durante horas específicas.

También debe monitorear cuidadosamente la eliminación de rangos (*range deletion*). Las eliminaciones de rangos son operaciones costosas y, si son demasiado frecuentes o afectan a demasiados documentos a la vez, pueden degradar el rendimiento del clúster fragmentado. Si nota una degradación después de las migraciones de rango, considere limitar el tamaño del lote de eliminación de rango e implementar retrasos de tiempo entre eliminaciones.

Cuando realice una ingesta masiva de datos o una exportación para una colección fragmentada específica, considere deshabilitar el equilibrador durante la duración de la operación; las operaciones masivas tienen más probabilidades de activar migraciones, desviando los recursos del clúster. Al deshabilitar el equilibrador, permite que la operación masiva continúe de manera eficiente mientras mitiga el impacto en los usuarios. Vuelva a habilitar el equilibrador cuando se complete la operación.

Monitorear el uso del disco en todos los fragmentos es crucial para garantizar que el equilibrador siempre tenga espacio para asignar datos al ejecutar migraciones. Además, discrepancias inesperadamente grandes en el uso del disco entre fragmentos pueden indicar problemas de conectividad dentro de su clúster fragmentado o problemas con el proceso del equilibrador en sí.

#### Bloqueos (*Locks*)

Para evitar conflictos entre migraciones, el equilibrador registra un bloqueo de base de datos especial en la base de datos `config` de un clúster fragmentado. Si observa que las migraciones esperadas no se inician, use este comando para verificar el estado del bloqueo del equilibrador y determinar si está obsoleto (*stale*):

```javascript
db.locks.find({ _id: "balancer"})
```

#### Consistencia de datos

La manipulación manual de su servidor de configuración o las operaciones de mantenimiento como actualizaciones o degradaciones pueden provocar metadatos de fragmentación inconsistentes si no prepara adecuadamente su clúster fragmentado de antemano. Los metadatos inconsistentes pueden provocar resultados de consulta incorrectos o pérdida de datos. Una estrategia de mantenimiento adecuada evita que surja este problema, pero si sospecha que un clúster fragmentado sufre de inconsistencia de metadatos, ejecute lo siguiente:

```javascript
db.runCommand({ checkMetadataConsistency: 1 })
```

---

### Sección 8: Herramientas de generación de informes de MongoDB

Ahora que sabe qué buscar, analicemos cómo buscarlo. Las herramientas de generación de informes de MongoDB se pueden dividir en tres categorías amplias: herramientas locales, herramientas alojadas de origen (*first-party*) y herramientas alojadas de terceros (*third-party*).

#### Herramientas locales

Las herramientas locales para informes de métricas incluyen utilidades de línea de comandos dedicadas y comandos de base de datos. Estas son opciones ideales para una retroalimentación rápida y proporcionan datos de rendimiento en formatos fácilmente asimilables para flujos de trabajo de análisis personalizados.

Ejemplos de herramientas locales de generación de informes de MongoDB incluyen:

- **`mongostat`:** Proporciona información sobre la frecuencia y el uso de memoria por operación, lo que le permite analizar el comportamiento y las necesidades del usuario.
- **`mongotop`:** Informa los niveles de actividad de lectura y escritura por colección, revelando patrones de utilización y cuellos de botella que pueden fundamentar su arquitectura de datos.

Los comandos de base de datos proporcionan una mayor granularidad que las utilidades dedicadas y se recomiendan para usuarios que conocen las métricas que más les interesan:

- **`serverStatus`:** Proporciona informes completos sobre todas las facetas de su servidor de base de datos. Como se detalla a lo largo de este capítulo, puede aislar facetas específicas de la operación de su sistema pasando argumentos correspondientes a los subdocumentos del informe general.
- **`dbStats`:** Proporciona informes de almacenamiento y capacidad para una base de datos determinada.
- **`collStats`:** Proporciona informes sobre métricas a nivel de colección. Este comando es una parte esencial del monitoreo de la indexación.
- **`replSetGetStatus`:** Proporciona un informe sobre el estado del conjunto de réplicas en su conjunto.

#### Herramientas de origen (*First-party tools*)

Además de las herramientas de monitoreo integradas directamente en el servidor de base de datos de MongoDB, MongoDB proporciona herramientas de monitoreo listas para usar con cada una de sus soluciones de bases de datos alojadas:

- **MongoDB Cloud Manager:** Un paquete alojado en la nube para administrar implementaciones de MongoDB, que proporciona monitoreo, copias de seguridad y automatización.
- **MongoDB Ops Manager:** Un equivalente local (*on-premises*) a Cloud Manager.
- **MongoDB Atlas:** Una solución de despliegue de MongoDB totalmente administrada y alojada en la nube, que ofrece varias extensiones a la funcionalidad de MongoDB, como Atlas Search, Atlas Vector Search y Atlas Stream Processing.

#### Herramientas de terceros (*Third-party tools*)

MongoDB también admite una variedad de herramientas de monitoreo de terceros que interactúan con su base de datos de MongoDB para proporcionar métricas enriquecidas y/o interfaces útiles como paneles de control e interfaces móviles:

- **VividCortex:** Un agente que analiza las estadísticas de la base de datos de MongoDB y el tráfico de la red para proporcionar información sobre el rendimiento segundo a segundo.
- **Scout:** Una solución de monitoreo de aplicaciones que incluye soporte para métricas de MongoDB.
- **Server Density:** Proporciona un panel específico de MongoDB, alertas, cronología de conmutación por error de replicación y aplicaciones móviles.
- **Application Performance Management:** Una oferta de software como servicio (SaaS) de IBM que monitorea el rendimiento de las aplicaciones. Esto incluye herramientas de monitoreo para MongoDB.
- **Datadog:** Herramienta de monitoreo de infraestructura para visualizar el rendimiento de implementaciones de MongoDB. Se integra con MongoDB Ops Manager, MongoDB Cloud Manager o MongoDB Atlas.
- **SPM Performance Monitoring:** Monitorea todas las métricas cruciales de MongoDB y proporciona una correlación de métricas y registros.
- **Pandora FMS:** Proporciona el complemento `PandoraFMS-mongodb-monitoring` para el monitoreo de MongoDB.

#### Monitoreo con la CLI de Atlas

Además de las herramientas de monitoreo similares a las que se ofrecen en MongoDB Ops Manager y MongoDB Cloud Manager, MongoDB Atlas proporciona varios comandos para inspeccionar el rendimiento de su implementación a través de la CLI de Atlas. Para flujos de trabajo de monitoreo programáticos, considere los siguientes comandos:

- **`Atlas alerts`:** Una familia de comandos para interactuar con las alertas de métricas que ha configurado para su despliegue de Atlas. Los comandos `describe` y `list` recuperan información sobre las alertas existentes, mientras que la subfamilia `settings` le permite crear y configurar alertas.
- **`Atlas metrics`:** Una familia de comandos para obtener métricas de rendimiento a nivel de base de datos, disco o proceso.

---

### Sección 9: Errores comunes de monitoreo

Un potente conjunto de monitoreo solo es tan útil como la estrategia que construya a su alrededor. Además de las pautas generales descritas hasta ahora, exploremos algunos antipatrones de uso comunes y cómo evitarlos:

- **Descuidar el estado del conjunto de réplicas:** Los conjuntos de réplicas de MongoDB ofrecen redundancia y alta disponibilidad. Sin embargo, no vigilar el estado de su conjunto de réplicas puede provocar una posible pérdida de datos o problemas de disponibilidad:
  - *Cómo evitarlo:* Verifique periódicamente el estado de cada miembro de su conjunto de réplicas. Configure alertas para cualquier cambio en el estado del conjunto de réplicas.
- **Usar la configuración predeterminada:** MongoDB proporciona una configuración predeterminada destinada a ofrecer valores predeterminados sensatos para la mayoría de los escenarios, pero no optimizados para casos particulares. Depender de esta configuración para implementaciones de producción puede generar cuellos de botella en el rendimiento:
  - *Cómo evitarlo:* Analice periódicamente las tendencias de uso a medio y largo plazo de sus aplicaciones y ajuste sus implementaciones para adaptarse a estas tendencias.
- **Ignorar las advertencias de almacenamiento:** Quedarse sin almacenamiento puede detener su servidor MongoDB. Las advertencias de almacenamiento pueden parecer menos importantes que las advertencias relacionadas con la memoria debido a la diferencia en la urgencia, pero descuidarlas provocará degradaciones del rendimiento en cascada:
  - *Cómo evitarlo:* Configure alertas para sus métricas de almacenamiento e incorpore el escalado de almacenamiento y el archivado de datos en sus estrategias de implementación a largo plazo desde el principio.
- **Ignorar los límites de conexión:** Cada instancia de MongoDB tiene un límite en la cantidad de conexiones de cliente simultáneas que puede mantener. Ignorar este límite puede provocar conexiones rechazadas, cuyas consecuencias pueden ser significativamente más complejas de solucionar que si hubiera respondido preventivamente a sus crecientes demandas de conectividad:
  - *Cómo evitarlo:* Ajuste la configuración de su grupo de conexiones (*connection pool*) o escale sus implementaciones de MongoDB antes de que las cargas de trabajo superen sus límites de conexión existentes.

---

### Sección 10: Copias de seguridad de MongoDB

> “Más vale prevenir que curar.”

Una de las formas más fáciles y seguras de mejorar la integridad de los datos de su sistema es implementar una estrategia de copias de seguridad. Los imprevistos comprometen sus datos, hasta e incluyendo la pérdida absoluta de datos. Si bien la replicación integrada en el modelo de clúster de MongoDB puede mitigar la mayoría de estos problemas, las violaciones de seguridad física o digital, los errores humanos y los desastres naturales como incendios, inundaciones y terremotos pueden afectar a todas las réplicas de un conjunto simultáneamente. Para protegernos verdaderamente contra estos escenarios, recurrimos a las copias de seguridad. MongoDB proporciona varios métodos para capturar y restaurar copias de seguridad.

#### Copias de seguridad con Atlas

MongoDB Atlas proporciona la función Cloud Backups. Este servicio de administración de copias de seguridad aprovecha las capacidades de instantáneas (*snapshots*) de su proveedor de nube elegido (AWS, Azure o GCP) para respaldar sus datos y proporciona controles intuitivos a través de la GUI, la API o la CLI. Atlas admite las siguientes comodidades para la gestión de copias de seguridad:

- Los usuarios pueden iniciar copias de seguridad a pedido y configurar copias de seguridad automáticas y continuas. Las copias de seguridad continuas funcionan según políticas de copia de seguridad definidas por el usuario, lo que le permite establecer la granularidad, la frecuencia y el tiempo de retención de las instantáneas.
- Los usuarios pueden configurar clústeres multirregionales para copiar instantáneas automáticamente de una región a otras, lo que le permite restaurar datos incluso en caso de interrupciones del servicio a escala regional dentro de su proveedor de la nube.
- Los usuarios pueden aplicar una protección de datos estricta configurando una política de cumplimiento de copias de seguridad (*backup compliance policy*). Esto restringe en gran medida la capacidad de los usuarios para reducir manualmente varias configuraciones de respaldo o eliminar datos de respaldo.
- Atlas puede aplicar cifrado en reposo a sus datos utilizando un proveedor de Servicio de Administración de Claves (KMS).
- Los usuarios pueden exportar instantáneas a depósitos de AWS S3 o Azure Blob Storage.
- Atlas archiva automáticamente los datos a los que se accede con poca frecuencia en dispositivos de almacenamiento de objetos.
- Los usuarios pueden restaurar bases de datos en Atlas utilizando instantáneas, copias de seguridad continuas, instantáneas descargadas localmente, instantáneas de Cloud Manager e incluso implementaciones de MongoDB autogestionadas (con la herramienta `mongoimport`).

Además, puede administrar las copias de seguridad de Atlas a través de la CLI de Atlas con la familia de comandos `Atlas backups` o con el punto final de OpenAPI de Cloud Backups.

#### Copia de seguridad en plataformas autogestionadas

Para aquellos que administran MongoDB en su propia infraestructura, tanto MongoDB Ops Manager como MongoDB Cloud Manager brindan herramientas de copia de seguridad:

- **MongoDB Ops Manager:** Para organizaciones que tienen reservas sobre conectar sus servidores a un SaaS externo debido a problemas de seguridad u otras inquietudes, Ops Manager es la solución ideal. Se puede configurar para escribir instantáneas en almacenes de bloques (*blockstores*), depósitos de AWS S3 o un sistema de archivos. También admite copias de seguridad regionales, donde cada clúster o fragmento de su implementación lee y escribe en almacenes de instantáneas en una región de su elección para admitir el aislamiento de datos.
- **MongoDB Cloud Manager:** Un equivalente en la nube a la funcionalidad local de Ops Manager, con el mismo soporte para comodidades de copia de seguridad y restauración.

Ambos servicios también proporcionan copias de seguridad consultables (*queryable backups*). Estas se aprovisionan como instancias de MongoDB de solo lectura con una vida útil de 24 horas durante las cuales puede consultar el contenido de la copia de seguridad sin restaurarlo directamente en su implementación. Esto le permite restaurar un subconjunto de los datos contenidos en su interior o determinar el mejor punto en el tiempo al que restaurar un sistema comparando datos entre instantáneas.

#### Copias de seguridad locales

Para aquellos con implementaciones de MongoDB totalmente autogestionadas, existen varios métodos para crear una copia de los archivos de datos subyacentes de MongoDB. Antes de discutir enfoques específicos, examinemos algunas consideraciones prácticas.

#### Requisitos de consistencia

Su despliegue debe cumplir con ciertos requisitos previos para poder tomar una instantánea válida de una instancia `mongod`. El más básico es que debe tener habilitado el registro en diario (*journaling*) y el diario debe residir en el mismo volumen lógico que todos los demás archivos de datos relevantes para su implementación de MongoDB. No puede garantizar la coherencia o validez de su instantánea sin cumplir primero estas condiciones.

Los clústeres fragmentados requieren algunas consideraciones adicionales para garantizar la coherencia. Primero, debe deshabilitar el proceso del equilibrador para asegurarse de que no se inicien migraciones de fragmentos de datos durante el proceso de instantánea; de lo contrario, corre el riesgo de duplicación o pérdida de datos. También debe ejecutar el comando `fsync`, que se analiza con más detalle más adelante en este capítulo, en su instancia `mongos` para ayudar a garantizar la coherencia en todo el clúster. Este comando bloquea las operaciones de escritura entrantes en el clúster, lo que garantiza que su copia de seguridad capture un estado único y estable.

#### Copia de seguridad completa frente a incremental

A medida que crecen sus implementaciones de MongoDB, llega un punto en el que ejecutar copias de seguridad completas cada vez resulta poco práctico debido a los requisitos de tiempo y recursos. Afortunadamente, MongoDB admite instantáneas incrementales: después de tomar la primera instantánea de una implementación determinada, las instantáneas posteriores capturan solo los datos que han cambiado en el período intermedio. Esto reduce el uso de disco, red y memoria, aunque estas instantáneas incrementales dependen de las instantáneas completas subyacentes para la funcionalidad de restauración.

Estratégicamente, debe evaluar el patrón de uso de su aplicación para determinar una frecuencia adecuada para las copias de seguridad completas y complementarlas con una política de instantáneas incrementales nocturnas. Considere comenzar con una copia de seguridad completa mensual y ajústela a medida que desarrolle una comprensión más clara de lo que sus usuarios necesitan y lo que su sistema puede manejar.

Ops Manager, Cloud Manager y Atlas brindan un soporte sólido para copias de seguridad incrementales y agilizan enormemente el proceso. Aún puede realizar copias de seguridad incrementales utilizando una implementación de MongoDB autogestionada aprovechando el *oplog*. Para hacerlo, siga este procedimiento:

1. Realice una copia de seguridad completa inicial utilizando cualquiera de los métodos de copia de seguridad locales admitidos que se describen más adelante.
2. Bloquee temporalmente las escrituras en los secundarios de su conjunto de réplicas.
3. Identifique la entrada más reciente en el *oplog*.
4. Exporte las entradas posteriores del *oplog*.
5. Desbloquee las escrituras en los secundarios de su conjunto de réplicas.

Si desea restaurar a partir de esta instantánea incremental manual, utilice `mongorestore` con la opción `--oplogReplay`, utilizando el archivo exportado `oplog.rs`.

#### Copias de seguridad con mongodump

`mongodump` es una utilidad de línea de comandos que crea una copia de seguridad binaria de los datos en su clúster de MongoDB. Está diseñada para combinarse con la utilidad `mongorestore` para restaurar desde una instantánea existente.

`mongodump` no es bloqueante, lo que significa que puede escribir datos en su clúster mientras lo ejecuta; para agregar estas escrituras al *oplog* de salida, simplemente pase el indicador `--oplog`. Luego puede restaurar desde esta copia de seguridad compuesta usando el indicador `--oplogReplay` cuando ejecute `mongorestore`. Esto no se aplica a los clústeres fragmentados, que deben bloquearse contra escritura mediante `fsync` antes de iniciar un `mongodump`.

Si bien este enfoque es eficaz para implementaciones más pequeñas, a medida que su sistema crezca, es probable que encuentre limitaciones de rendimiento al utilizar estas herramientas. Cuando restaura una base de datos con `mongorestore`, debe iniciar una reconstrucción de todos sus índices, lo que puede imponer demandas significativas de recursos del sistema durante mucho tiempo. Además, estas operaciones obligan a la base de datos a leer todos los datos a través de la memoria, lo que puede provocar la expulsión de datos en su conjunto de trabajo, ralentizando el rendimiento más allá de la duración de la operación.

#### Instantáneas del sistema de archivos (*Filesystem snapshots*)

Si su sistema de almacenamiento admite instantáneas, aprovechar esta funcionalidad puede ser uno de los enfoques más rápidos para una copia de seguridad local. La siguiente tabla describe algunos sistemas de almacenamiento que admiten instantáneas del sistema de archivos:

| Soluciones de Sistema Operativo | Soluciones de Almacenamiento en la Nube |
| :--- | :--- |
| Logical Volume Manager (LVM) | Amazon EC2 Elastic Block Store (EBS) |
| ZFS | Azure Disk Storage |
| BTRFS | Oracle Cloud Block Volumes |
| Microsoft Storage Spaces | Google Cloud Persistent Disk |

> **Tabla 6.1:** Soluciones de almacenamiento habilitadas para instantáneas del sistema de archivos

Cada uno de estos sistemas viene con sus propias herramientas para interactuar con volúmenes lógicos, pero el flujo de trabajo básico para crear instantáneas del sistema de archivos se puede resumir de la siguiente manera:

1. Asegúrese de que el registro en diario (*journaling*) esté habilitado para su implementación de MongoDB.
2. Determine a qué volúmenes lógicos en el sistema de almacenamiento de destino está vinculado su `dbPath`.
3. Dirija la creación de su instantánea a esos volúmenes lógicos.
4. Valide su copia de seguridad montando la instantánea e iniciando un proceso `mongod` contra los datos. Establezca una conexión con esa instancia `mongod` e inspeccione el conjunto de datos.

#### Copias de seguridad con cp o rsync

Si su sistema de almacenamiento no admite instantáneas, puede copiar los archivos directamente usando `cp`, `rsync` o una herramienta similar. Dado que copiar varios archivos no es una operación atómica, debe detener todas las escrituras entrantes en el `mongod` de destino antes de copiar los archivos. De lo contrario, producirá una copia de seguridad no válida.

Las copias de seguridad producidas al copiar los datos subyacentes no admiten la recuperación en un momento dado (*point-in-time recovery*) para conjuntos de réplicas y pueden resultar engorrosas para clústeres grandes y/o fragmentados. Además, estas copias de seguridad ocupan un espacio de almacenamiento considerable porque incluyen índices y duplican la fragmentación y el relleno del almacenamiento subyacente.

#### Sincronización de clúster a clúster (*Cluster-to-cluster sync*)

MongoDB admite la replicación de datos entre clústeres mediante la utilidad `mongosync`. Si bien no es estrictamente una forma de copia de seguridad, esta operación ocupa un lugar importante en su estrategia de redundancia y continuidad, lo que le permite automatizar el proceso de transferencia de datos e índices entre clústeres. El proceso de sincronización se lleva a cabo en dos fases: la fase de sincronización y la fase de confirmación (*commit phase*). Durante la fase de sincronización, `mongosync` altera varias propiedades de los índices y de las colecciones limitadas para facilitar las escrituras en el clúster de destino; estas alteraciones solo se revierten durante la fase de confirmación, lo que significa que no puede garantizar la idempotencia de los datos en los clústeres si interrumpe el proceso durante la fase de sincronización. `mongosync` no admite compilaciones de índices continuas (*rolling index builds*), así que asegúrese de planificar la realización de una compilación de índices en el clúster de origen antes de la sincronización o en el clúster de destino después de la sincronización como parte de su migración.

---

### Sección 11: Errores comunes en copias de seguridad

Las copias de seguridad son una herramienta crucial para proteger la integridad de los datos, pero no permita que la tranquilidad que brindan se convierta en complacencia. Las copias de seguridad en sí deben administrarse con cuidado para garantizar que realmente cumplan el propósito previsto. Examinemos algunos problemas comunes que pueden surgir en las estrategias de copia de seguridad:

- **Hacer copias de seguridad sin datos del oplog:** El *oplog* rastrea todas las operaciones que modifican datos. Es la base de las restauraciones en un momento dado, así como una parte crucial de las copias de seguridad incrementales autogestionadas; sin el *oplog*, estas funciones no funcionarán:
  - *Cómo evitarlo:* Asegúrese de que sus datos del *oplog* estén respaldados con el mismo rigor que los datos reales de su aplicación.
- **No verificar las copias de seguridad:** Las copias de seguridad no verificadas pueden convertir las interrupciones temporales del acceso en una pérdida permanente de datos:
  - *Cómo evitarlo:* Verifique las copias de seguridad realizando restauraciones de prueba en entornos controlados e inspeccionando el contenido para asegurarse de que coincida con sus expectativas.

El monitoreo y las copias de seguridad son esenciales para la confiabilidad de las implementaciones de MongoDB. El primero le permite planificar y diseñar para un rendimiento óptimo, mientras que el segundo protege contra la pérdida de datos. MongoDB proporciona métodos que van desde la línea de comandos hasta el extenso conjunto de herramientas GUI en Atlas, lo que permite a los usuarios elegir cómo conceptualizan e implementan sus estrategias. Ya sea localmente o en la nube, tendrá la información y la confiabilidad necesarias para aprovechar al máximo sus datos.

Tan importante como poder analizar el comportamiento actual del sistema y recuperarse de las interrupciones actuales del servicio es la capacidad de rastrear el historial de eventos dentro de su sistema. Aquí es donde entra en juego la auditoría.

---

### Sección 12: Auditoría

La auditoría de bases de datos es el seguimiento y registro de operaciones, ya sean operaciones de lectura y escritura sobre datos, autenticaciones de usuarios u otras acciones administrativas. Mantener dicho historial ayuda a garantizar que su sistema proporcione al usuario seguridad, cumplimiento normativo, diagnóstico y rendición de cuentas.

Los registros de auditoría pueden revelar patrones sospechosos de actividad que apuntan a una autenticación o autorización comprometida, lo que le permite remediar las amenazas de seguridad existentes y ajustar el diseño de su sistema para evitar más amenazas. De la mano del fortalecimiento de su postura de seguridad, una auditoría adecuada garantiza que la protección de datos del usuario y los requisitos de acceso coincidan con los estándares regulatorios más estrictos, cumpliendo con las obligaciones de su organización hacia los usuarios y los reguladores.

Los administradores de sistemas pueden leer los registros de auditoría para diagnosticar problemas de rendimiento e integridad de datos con extrema granularidad, más allá de las métricas más precisas vinculadas al tiempo. Esta misma granularidad garantiza la responsabilidad de poder atribuir todas las operaciones de la base de datos a usuarios específicos.

Dada su importancia, su estrategia de auditoría justifica un enfoque reflexivo que equilibre el rendimiento y la capacidad del sistema con políticas de retención selectiva. Antes de implementar su estrategia, asegúrese de tener una idea clara de qué tipos de actividad justifican este nivel de granularidad.

#### Registro del sistema frente a auditoría (*System logging versus auditing*)

El registro (*logging*) y la auditoría son conceptos estrechamente relacionados en MongoDB, pero no son intercambiables. Ambos tienen propósitos diferentes pero igualmente importantes como parte de su estrategia de administración y monitoreo y producen resultados diferentes. Existe cierta superposición entre los conceptos, pero un vistazo a algunos ejemplos debería ayudar a aclarar las diferencias.

Primero, tenemos un ejemplo de un mensaje de registro típico:

```json
{  
 "t": {  
   "$date": "2020-05-01T15:16:17.180+00:00"  
 },  
 "s": "I",  
 "c": "NETWORK",  
 "id": 12345,  
 "ctx": "listener",  
 "svc": "R",  
 "msg": "Listening on",  
 "attr": {  
   "address": "127.0.0.1"  
 }  
}
```

Esta entrada le indica que el servidor MongoDB se inició con los parámetros especificados, como se muestra en el atributo `attr`. En este caso, el atributo contiene parámetros del archivo de configuración (`mongod.conf`).

En comparación, aquí hay un registro de auditoría típico:

```json
{
  "atype": "createCollection",
    "ts": {
      "$date": "2025-04-21T16:09:33.810-04:00"
    },
    "uuid": {
      "$binary": "ZF9aa2odPqSg74C/Lf1yJk==",
      "$type": "04"
    },
    "local": {
      "ip": "127.0.0.1",
      "port": 27017
    },
    "remote": {
      "ip": "127.0.0.1",
      "port": 56292
    },
    "users": [
      {
        "user": "admin",
        "db": "admin"
      }
    ],
    "roles": [
      {
        "role": "root",
        "db": "admin"
      }
    ],
    "param": {
        "ns": "audit.coll1"
    },
    "result": 0
}
```

Esta entrada nos dice qué sucedió, cuándo sucedió, dónde sucedió y quién tomó la acción.

Con estos ejemplos en mano, podemos ver las distinciones clave entre el registro general y la auditoría:

| Característica | Registro del Sistema (*System Logging*) | Auditoría (*Auditing*) |
| :--- | :--- | :--- |
| **Objetivo** | Facilitar el monitoreo, el diagnóstico y la optimización del rendimiento capturando eventos y el estado del servidor | Facilitar el cumplimiento normativo, la seguridad y la investigación capturando eventos de acceso y modificación de datos |
| **Granularidad** | Menos granular, centrándose en una amplia variedad de actividades diferentes y facetas del estado del sistema | Más granular, rastreando a nivel de acciones específicas, usuarios o incluso documentos |
| **Naturaleza de los datos** | Estructurado como JSON | Estructurado como JSON, con soporte adicional integrado para el esquema OCSF |
| **Retención** | Diseñado para rotarse a intervalos regulares para evitar el consumo excesivo de capacidad de almacenamiento | Diseñado para retenerse durante períodos arbitrariamente largos para mantener las regulaciones de acceso y seguridad de datos y facilitar investigaciones |
| **Destino** | Normalmente se almacena localmente; puede ser consumido por herramientas de monitoreo y análisis | Reenviado a sistemas de Gestión de Eventos e Información de Seguridad (SIEM) y/o medios de almacenamiento a largo plazo |

> **Tabla 6.2:** Diferencias entre el registro general y la auditoría

En resumen, el registro del sistema proporciona un monitoreo y diagnóstico integral del estado del sistema. Proporciona la amplitud a su estrategia de registro. Por el contrario, la auditoría observa acciones específicas con fines de seguridad y cumplimiento. Proporciona la profundidad.

#### Registro del sistema (*System logging*)

Si bien nos centraremos principalmente en el registro de auditoría, vale la pena considerar algunos aspectos del registro del sistema para obtener el mayor valor de ellos.

#### Nivel de detalle (*Verbosity*)

Los registros del sistema de MongoDB se pueden configurar para diferentes niveles de detalle, aumentando o disminuyendo el volumen de registro realizado por MongoDB. Los niveles de detalle corresponden a las categorías de gravedad de las entradas de registro: un nivel de detalle más alto muestra mensajes para elementos de menor gravedad, mientras que un nivel de detalle más bajo limita la salida solo a elementos de mayor gravedad. Al determinar el nivel de detalle de su registro del sistema, considere lo siguiente:

- **¿Para qué componentes de su sistema desea un registro más o menos detallado?** MongoDB le permite establecer diferentes niveles de detalle para varios componentes, como `ACCESS`, `COMMAND` y `ELECTION`.
- **¿Qué tan seguro está de la corrección de su sistema?** Cuando todavía está desarrollando su base de datos en un entorno de ensayo (*staging*), un nivel de detalle más alto mejora la utilidad de sus registros para la depuración general. Sin embargo, los registros ocupan espacio de almacenamiento y requieren operaciones de escritura, por lo que para una implementación de producción completamente validada, considere establecer un nivel de detalle bajo para minimizar el consumo de recursos.

Para ver sus niveles de detalle actuales, puede utilizar lo siguiente:

```javascript
db.getLogComponents()
```

Puede configurar sus niveles de detalle editando la configuración `systemLog.verbosity` en su archivo `mongod.conf`.

#### Redacción (*Redaction*)

MongoDB proporciona redacción de registros integrada con el parámetro `redactClientLogData`. Si habilita esta configuración en su `mongod.conf`, todo, excepto los metadatos, los archivos de origen y los números de línea, se redacta en sus mensajes de registro. Esto agrega una capa adicional de seguridad de datos a su sistema, aunque reduce la utilidad diagnóstica de sus registros. Si su sistema maneja datos confidenciales, considere implementar esto como parte de su estrategia de cumplimiento.

#### Registro en despliegues administrados

Además de los registros de procesos de MongoDB disponibles para implementaciones de MongoDB autogestionadas, MongoDB Atlas registra los intentos de autenticación realizados contra sus clústeres, que puede ver con el comando `atlas accessLogs list` en la CLI de Atlas, o visitando la página **View Database Access History** para su clúster.

MongoDB Ops Manager y MongoDB Cloud Manager proporcionan registros para cada agente de MongoDB en su implementación, así como registros de acceso, copias de seguridad, operaciones y eventos de inicio. Estos se pueden encontrar navegando al panel **Deployment**, seleccionando la pestaña **Agents** y luego haciendo clic en **Agent Logs** en la GUI de Ops Manager o Cloud Manager.

---

### Sección 13: Registro de auditoría (*Audit logging*)

Al configurar la auditoría, los administradores pueden filtrar los eventos que son relevantes para sus necesidades, asegurándose de tener una vista detallada de las actividades críticas y minimizando el desorden de información.

#### Formatos de registro de auditoría

El mensaje de registro de auditoría de ejemplo que se muestra anteriormente utilizó el esquema de `mongo`. Si bien MongoDB utiliza este esquema de forma predeterminada, también admite el esquema OCSF, un formato estándar ampliamente utilizado y compatible con muchas herramientas de procesamiento de registros. Puede configurar el esquema utilizado para la auditoría con la opción `auditLog.schema` en `mongod.conf`.

El siguiente mensaje de ejemplo describe una acción `authenticate` según el esquema OCSF:

```json
{
  "activity_id" : 1,
  "category_uid" : 3,
  "class_uid" : 3002,
  "time" : 1710715316123,
  "severity_id" : 1,
  "type_uid" : 300201,
  "metadata" : {
     "correlation_uid" : "20ec4769-984d-445c-aea7-da0429da9122",
     "product" : "MongoDB Server",
     "version" : "1.0.0"
  },
  "actor" : {
     "user" : {
        "type_id" : 1,
        "groups" : [ { "name" : "admin.root" } ]
     }
  },
  "src_endpoint" : { "ip" : "127.0.0.1", "port" : 56692 },
  "dst_endpoint" : { "ip" : "127.0.0.1", "port" : 20040 },
  "user" : { "type_id" : 1, "name" : "admin.admin" },
  "auth_protocol" : "SCRAM-SHA-256",
  "unmapped" : { "atype" : "authenticate" }
}
```

Debido a la mayor compatibilidad del esquema OCSF, le recomendamos encarecidamente que considere adoptar este esquema para sus registros de auditoría.

#### Tipos de eventos auditables

La siguiente es una lista no exhaustiva de los tipos de eventos que audita MongoDB:

- **Eventos de autenticación y autorización:**
  - `authenticate` y `authCheck`: Eventos relacionados con la autenticación de usuarios.
  - `logout`: Eventos de cierre de sesión de usuarios.
- **Eventos de gestión de usuarios:**
  - `createUser`, `dropUser`, `dropAllUsersFromDatabase` y `updateUser`: Eventos asociados con la creación, modificación y gestión de usuarios.
  - `createRole`, `dropRole`, `grantRolesFromUser` y `revokeRolesFromUser`: Eventos asociados con la creación, modificación y gestión de roles.
- **Eventos de operaciones de datos:**
  - `createCollection`, `dropCollection` y `renameCollection`: Eventos asociados con la gestión de colecciones.
  - `createDatabase` y `dropDatabase`: Eventos asociados con la gestión de bases de datos.
  - `createIndex` y `dropIndex`: Eventos asociados con la gestión de índices.
- **Eventos de operaciones administrativas:**
  - `enableSharding`, `addShard`, `removeShard` y `shardCollection`: Eventos relacionados con operaciones de fragmentación.
  - `replSetReconfig`: Eventos asociados con la configuración de conjuntos de réplicas.
- **Eventos de inicio y apagado:**
  - `startup` y `shutdown`.

#### Habilitar la auditoría en implementaciones autogestionadas

Antes de habilitar la auditoría en MongoDB, debe elegir dónde y cómo almacenar sus registros de auditoría. MongoDB admite una variedad de formatos de salida para adaptarse a diferentes requisitos organizacionales. Cada uno de los siguientes ejemplos utiliza el parámetro `auditLog` para la configuración:

#### Consola:
Escribe los registros de auditoría directamente en la salida estándar (`stdout`) de su consola. Este método de salida no conserva los datos de su registro de auditoría y no es adecuado como una solución independiente. Esto es útil para realizar pruebas o para pasar los registros directamente a otra herramienta para un procesamiento adicional antes de escribirlos en el disco:

```yaml
storage:
  dbPath: data/db
auditLog:
  destination: console
```

#### JSON:
Escribe registros de auditoría como archivos con formato JSON. Este método de salida es un enfoque altamente compatible para almacenar sus registros de auditoría. Además de admitir las numerosas herramientas que tienen métodos integrados para ingerir y procesar datos JSON, le permite importar registros directamente a una colección de MongoDB o a una colección en cualquier otra base de datos de modelo de documentos:

```yaml
storage:
  dbPath: data/db
auditLog:
  destination: file
  format: JSON
  path: data/db/auditLog.json
```

*Nota:* Este formato requiere dos parámetros adicionales: uno para especificar el tipo de salida del archivo y el otro para especificar la ruta del archivo en la que desea escribir el registro.

#### BSON:
Escribe registros de auditoría como una representación binaria de datos tipo JSON. Esta opción optimiza la integración con MongoDB, sacrificando la compatibilidad genérica por la eficiencia:

```yaml
storage:
  dbPath: data/db
auditLog:
  destination: file
  format: BSON
  path: data/db/auditLog.json
```

#### Syslog:
Escribe registros de auditoría directamente en `syslog`, una herramienta de registro estándar de UNIX/tipo UNIX. Esto facilita la integración con una clase de herramientas de monitoreo y gestión de registros creadas en torno a la ingesta de eventos de `syslog`:

```yaml
storage:
  dbPath: data/db
auditLog:
  destination: syslog
```

La opción más adecuada para usted depende del equilibrio entre el cumplimiento normativo, el conjunto de herramientas que desea crear en torno a sus auditorías y el grado de dependencia del sistema (*system lock-in*) con el que se sienta cómodo.

MongoDB Ops Manager y MongoDB Cloud Manager también admiten la auditoría, ofreciendo convenientes opciones de interfaz gráfica de usuario para configurarla a través de las opciones de configuración avanzada de un proceso de MongoDB determinado.

#### Habilitar la auditoría en Atlas

Atlas también facilita la auditoría de MongoDB para todos los clústeres en el nivel M10 o superior. Siga este procedimiento para habilitar la auditoría:

1. Inicie sesión en su proyecto de Atlas y seleccione **Advanced** en la sección **SECURITY** del menú de navegación de la izquierda:

> **Figura 6.2:** Configuración de seguridad avanzada

2. Cambie el botón **Database Auditing** a la posición **On**:

> **Figura 6.3:** Habilitación de la auditoría de la base de datos

3. Después de habilitar la función, aparecen las opciones de configuración a continuación. Estas opciones le permiten configurar filtros de auditoría utilizando un menú de selección de forma predeterminada:

> **Figura 6.4:** Uso del generador de filtros

4. Alternativamente, puede hacer clic en el botón **USE CUSTOM JSON FILTER** para definir manualmente su filtro mediante una expresión JSON:

> **Figura 6.5:** Definición de un filtro JSON personalizado

5. Independientemente de la opción que elija, haga clic en el botón **Save** para aplicar los cambios.

Además de estos registros de auditoría de MongoDB disponibles de forma general, Atlas mantiene un registro de eventos de disparadores (*triggers*), funciones y flujos de cambios (*change streams*), que publica en el feed de actividad de su proyecto de Atlas. Estos registros se conservan durante 10 días, tras los cuales se eliminan. Para conservar estos registros más allá de 10 días, descargue sus registros disponibles actualmente como se describe más adelante en este capítulo.

#### Filtros de auditoría

Auditar todo el tráfico de eventos en su sistema consumiría rápidamente su almacenamiento e incurriría en importantes penalizaciones de rendimiento. Con la misma importancia, el abrumador volumen de datos capturados de esta manera podría hacer que las investigaciones o cualquier tipo de análisis significativo resulten más laboriosos al ahogar la señal en el ruido. Los filtros de auditoría permiten a los administradores evitar estos inconvenientes al especificar qué eventos desean auditar y cuáles no.

Para filtrar su auditoría, defina criterios para campos específicos en los documentos de auditoría. Los criterios pueden filtrar por usuarios específicos, colecciones específicas, tipos de operaciones específicos o cualquier combinación de estos.

Una vez que haya habilitado la auditoría en su implementación como se describió anteriormente, la forma en que defina los filtros de auditoría varía según el tipo de implementación:

- Para implementaciones totalmente autogestionadas, edite manualmente `mongod.conf` de acuerdo con uno de los patrones de configuración descritos anteriormente.
- Para Ops Manager o Cloud Manager, haga lo siguiente:
  1. Haga clic en **Deployment**, luego en **Processes**.
  2. Haga clic en **Modify** junto al proceso `mongod` para el cual desea configurar la auditoría.
  3. Haga clic en **Advanced Configuration Options**.
  4. Haga clic en la opción **+ Add**.
  5. Agregue la propiedad `auditLogFilter`.
- Para Atlas, haga lo siguiente:
  1. Seleccione su proyecto y haga clic en **Advanced**.
  2. En **Database Auditing**, seleccione **Configure Your Auditing Filter**, luego haga clic en **Use Custom JSON Filter**.

Los filtros toman la forma de documentos JSON. Por ejemplo, para auditar todas las acciones del usuario `jim` que se autentica en la base de datos `admin`, utilice el siguiente filtro:

```json
{ "users" : [ { "user" : "jim", "db": "admin" } ] }
```

A continuación se muestran algunos ejemplos de patrones de filtro de auditoría:

- **Auditar operaciones de lectura para cualquier usuario en la colección `collA` de la base de datos `db1`:**

```json
{
  "atype": "authCheck",
  "param.ns": "db1.collA",
  "param.command": {
    "$in": [
      "find"
    ]
  }
}
```

- **Auditar todas las operaciones de actualización para cualquier usuario en cualquier colección:**

```json
{ "atype": "update" }
```

- **Combinar criterios de auditoría:** Al combinar múltiples criterios de auditoría, puede refinar su filtro haciéndolo más granular. El siguiente ejemplo captura solo las operaciones de eliminación contra la colección `collA` realizadas por el usuario `jim`:

```json
{
  "users": [
    {
      "user": "jim",
      "db": "admin"
    }
  ],
  "atype": "delete",
  "param.ns": "db1.collA"
}
```

- **Cambiar parámetros en tiempo de ejecución:** El comando `db.adminCommand`, cuando se llama con el argumento `setAuditConfig`, le permite modificar filtros en tiempo de ejecución:

```javascript
db.adminCommand(
  {
    setAuditConfig: 1,
    filter: {
      "atype": "authCheck",
      "param.command": {
        "$in": [
          "find",
          "insert",
          "delete",
          "update",
          "findAndModify"
        ]
      }
    }
  }
)
```

Este ejemplo reconfigura los filtros de auditoría para operaciones CRUD en cualquier base de datos. Tenga en cuenta que cuando cambia la configuración en tiempo de ejecución de esta manera, vuelve a la definida en su `mongod.conf` cuando reinicia el servidor. Si desea que estos cambios sean permanentes, persístalos siempre escribiéndolos en `mongod.conf`.

- **Verificar los parámetros de auditoría actuales:**

```javascript
db.adminCommand({ getAuditConfig: 1 })
```

Es importante recordar que la auditoría puede introducir una sobrecarga de rendimiento. Puede mitigar esta inquietud con el uso cuidadoso de filtros de auditoría para capturar solo los eventos que necesita. También debe probar sus filtros en un entorno de desarrollo antes de implementarlos en un despliegue de producción, verificando lo siguiente:

- Impacto aceptable en el rendimiento
- Captura de eventos esperados
- Ausencia de efectos secundarios no deseados

---

### Sección 14: Generación de perfiles de la base de datos (*Database profiling*)

Para completar su estrategia de análisis del sistema, MongoDB proporciona el generador de perfiles de bases de datos (*database profiler*), que registra información detallada sobre los comandos de base de datos ejecutados contra una instancia `mongod` en ejecución en una colección limitada llamada `system.profile`. Esta función está desactivada de forma predeterminada, pero se puede activar configurando el parámetro `operationProfiling.mode` con el valor entero apropiado en su `mongod.conf`.

Cuando se activa, el generador de perfiles introduce cierta sobrecarga de rendimiento y almacenamiento. Cuando utiliza el generador de perfiles como parte de su estrategia de pruebas en un entorno de ensayo (*staging*), estas consideraciones se pueden equilibrar sobre la marcha. Si desea realizar la generación de perfiles de base de datos en un entorno de producción, es importante evaluar cuidadosamente sus necesidades y prioridades antes de habilitar el generador de perfiles.

- Para la mayoría de los casos de uso en los que es deseable la generación de perfiles de bases de datos, considere establecer el nivel de generación de perfiles en `1`. Si utiliza un umbral de tiempo `slowms`, configúrelo en el valor máximo que su sistema puede esperar en un evento determinado. Si utiliza un filtro, seleccione solo aquellos eventos que sean de mayor interés.
- Un nivel de generación de perfiles de `2` captura todos los eventos y, por tanto, introduce la mayor sobrecarga. Este nivel de perfilador solo se recomienda para entornos de prueba.

---

### Sección 15: Visualización y descarga de registros

Existen múltiples formas de acceder a los registros según la naturaleza de su despliegue de MongoDB. Para implementaciones autogestionadas de MongoDB, Ops Manager o Cloud Manager, su despliegue escribe registros en una ruta de archivo de su elección, en el formato de su elección.

En MongoDB Atlas, puede hacer lo siguiente:

- **Descargar registros a través de la GUI:** En la página de descripción general de su proyecto, para un clúster determinado, haga clic en el icono de puntos suspensivos (...) y seleccione **Download Logs** en el menú desplegable:

> **Figura 6.6:** La acción Download Logs

- **Descargar registros con una llamada a la API:**
  1. Cree o recupere sus claves de API de Atlas públicas y privadas a través de la página Access Manager de su organización.
  2. Recupere su ID de proyecto utilizando la acción **Copy Project ID** en la página de Proyectos:

> **Figura 6.7:** Recuperación de su ID de proyecto

  3. Recupere el nombre de host de su clúster de MongoDB desde su página Clústeres.
  4. Una vez que tenga los cuatro parámetros, realice una llamada a la API con el siguiente formato:

```bash
curl --user "{public_key:private_key}" \
      --digest \
      --header "Accept: application/gzip" \
    GET  "https: //cloud.mongodb.com/api/atlas/v2/groups/{groupId}/clusters/
{hostname}/logs/{logName}.gz"
```

Aquí, `groupId` es su `projectId` y `hostname` es el nombre de host del clúster.

Alternativamente, puede descargar registros del clúster con la CLI de Atlas usando `atlas logs download`.

---

### Sección 16: Rotación de registros

Con el tiempo, sus implementaciones acumularán grandes volúmenes de datos de registro, incluso con una filtración agresiva. Los requisitos de almacenamiento y memoria para conservar o leer archivos de registro muy grandes pueden interferir con el rendimiento del sistema, por lo que una estrategia de registro completa implicará una rotación regular de registros. En términos generales, tiene tres enfoques para la rotación de registros para implementaciones de MongoDB autogestionadas:

1. Puede utilizar el comando integrado de MongoDB `logRotate`. Archiva el archivo de registro actual, agregando una marca de tiempo con formato `ISODate` al nombre para facilitar la referencia, y luego abre un nuevo archivo de registro.
2. Puede configurar el parámetro `systemLog.logRotate` en `reopen` en su `mongod.conf`, en cuyo caso MongoDB aprovechará la utilidad `logrotate` de Linux/UNIX. Esta utilidad cierra su archivo de registro actual y vuelve a abrir un archivo de registro con el mismo nombre, esperando que otro proceso haya cambiado el nombre del archivo antes de la rotación.
3. Puede configurar `mongod` para escribir datos de registro en el `syslog`. En este caso, puede utilizar cualquier utilidad de rotación de registros que utilizaría con un `syslog` normal.

Independientemente del método que elija, tenga en cuenta que la rotación de registros no se replica. Debe asegurarse de que cada instancia de un conjunto de réplicas se someta a una rotación de registros de forma individual.

Para implementaciones administradas en Ops Manager o Cloud Manager, puede configurar la rotación de registros automatizada para cualquier clúster administrado por un agente de MongoDB. Para hacerlo, navegue a la página **MongoDB Log Settings** de su despliegue y cambie **System Log Rotation** a activado, luego configure los ajustes de rotación de registros. Puede establecer umbrales de tiempo y de tamaño de archivo más allá de los cuales sus registros se rotan automáticamente. Atlas rota los registros automáticamente de forma predeterminada.

---

### Sección 17: Caso de estudio: Auditoría para cumplimiento normativo

Considere la auditoría en el contexto de un sistema de gestión de información médica. El sistema proporciona programación de citas, gestión de registros de pacientes, gestión de recetas y más. La información de atención médica se encuentra entre la información personal más confidencial que existe y debe manejarse con cuidado para garantizar los más altos estándares de seguridad, integridad y privacidad de los datos. Regulaciones como la Ley de Portabilidad y Responsabilidad del Seguro Médico (HIPAA) codifican esta necesidad. La auditoría de dicho sistema debe hacer lo siguiente:

- Rastrear el acceso y las modificaciones a los registros de los pacientes
- Monitorear las acciones de todos los usuarios administradores
- Garantizar el cumplimiento de las regulaciones y estándares de privacidad de datos
- Permitir una rápida investigación y respuesta a actividades sospechosas

Teniendo en cuenta los requisitos anteriores, una estrategia de auditoría sólida podría incluir las siguientes características:

- **Configuración y filtrado:**
  - Filtros para auditar operaciones CRUD en colecciones clave, por ejemplo, registros médicos, recetas y horarios.
  - Filtros para monitorear el comportamiento de usuarios administradores y otros con acceso a datos confidenciales.
- **Atribución e identificación:**
  - Autenticación segura, Identificación Única de Usuario (UUID).
  - Registros de información detallada del usuario, como ID, dirección IP y acciones tomadas.
- **Revisión y análisis periódicos:**
  - Revisión rutinaria de registros de auditoría para detectar patrones anómalos y actividades sospechosas.
  - Herramientas de análisis de registros para resaltar anomalías agudas o patrones crónicos.
- **Retención y copia de seguridad:**
  - Políticas de retención de registros de auditoría de conformidad con las regulaciones pertinentes (por ejemplo, un plazo de retención de seis años según HIPAA).
  - Copia de seguridad periódica de los registros de auditoría para garantizar la integridad de los datos mediante redundancia.
- **Respuesta a incidentes:**
  - Plan de respuesta aprovechando los registros de auditoría como fuentes primarias en las investigaciones.
  - Procedimiento de análisis de incidentes basado en registros de auditoría.

Si bien esto no pretende ser una lista exhaustiva ni una recomendación de políticas, esta lista ilustra las muchas formas en que una práctica de auditoría sólida puede reforzar la salud y la seguridad de un sistema de información de salud.

---

### Sección 18: Resolución de problemas de auditoría

La auditoría es un componente esencial de una arquitectura de sistema sólida, pero solo es tan efectiva como la estrategia que la rodea; el simple hecho de tener registros de auditoría no garantiza una "auditoría robusta". Por lo tanto, es importante revisar periódicamente su propia práctica de auditoría, del mismo modo que revisa periódicamente los registros de auditoría, para asegurarse de que está aprovechando al máximo los datos que ha recopilado. Algunos factores que debe considerar durante estas reevaluaciones estratégicas incluyen los siguientes:

- **Frecuencia de revisión de auditoría:** ¿La frecuencia con la que revisa sus registros de auditoría es adecuada a la carga de actividad de su sistema? Si es demasiado infrecuente, se vuelve más difícil encontrar la información que busca, se pueden acumular más incidentes entre revisiones para dividir la atención y es más probable que olvide incidentes más antiguos.
- **Priorización de la revisión:** Cuando utiliza herramientas de análisis, ¿qué tan claras son sus prioridades de búsqueda? Los patrones son más fáciles de detectar cuando sabe lo que está buscando y puede reducir el alcance de la información a investigar en las primeras etapas del proceso. Si sus sesiones de revisión son demasiado amplias, considere examinar sus prioridades.
- **Alertas y respuesta en tiempo real:** Si bien los registros de auditoría están destinados a ser artefactos a largo plazo, esto no excluye su utilidad para solucionar problemas actuales. Asegúrese de contar con planes de respuesta en tiempo real para anomalías específicas.
- **Realizar revisiones de cumplimiento interno:** Evalúe periódicamente cómo su régimen de auditoría se alinea o se desvía de las regulaciones pertinentes en su jurisdicción.
- **Monitorear el rendimiento:** La auditoría puede afectar el rendimiento de su sistema si se usa en exceso. Si experimenta cuellos de botella, considere reescribir sus filtros de auditoría para que sean más específicos y reducir el volumen general de actividad de auditoría.
- **Mantenerse informado:** Manténgase actualizado sobre las mejores prácticas y actualizaciones relacionadas con la auditoría en MongoDB para evitar problemas conocidos.

---

### Sección 19: Herramientas para el análisis de auditoría

Con un régimen de auditoría sólido y un proceso de revisión periódico implementados, la pieza final del rompecabezas son las herramientas que utilizará para analizar sus registros de auditoría. Sus opciones se pueden dividir en tres categorías:

- **Herramientas de origen (*First-party tools*):** MongoDB proporciona Compass, una herramienta gráfica para ver y consultar intuitivamente datos del modelo de documentos. Si configuró su implementación para escribir registros de auditoría como JSON o BSON, puede importar sus registros de auditoría directamente a una colección de MongoDB y usar Compass para verlos tal como vería cualquier otro dato en una colección.
- **Herramientas de terceros (*Third-party tools*):** Cualquier herramienta de terceros que pueda ingerir datos JSON, BSON o syslog puede integrarse con su despliegue de MongoDB. Por ejemplo, podría utilizar Splunk, una popular plataforma de análisis de datos para indexar y analizar registros de auditoría.
- **Herramientas del sistema y scripts personalizados:** Finalmente, puede utilizar utilidades de línea de comandos comunes a la mayoría de los sistemas, como `grep`, o escribir sus propios scripts en un lenguaje de su elección para leer, filtrar y analizar sus datos de auditoría.

---

### Sección 20: Resumen

En este capítulo, exploró el monitoreo, las copias de seguridad, la auditoría y la configuración de seguridad en MongoDB y cómo cada uno de estos es una pieza crucial del rompecabezas general que es el diseño operativo, el análisis y el refinamiento. Aprendió sobre los diversos desafíos, oportunidades y herramientas disponibles en cada faceta, y cómo desarrollar e implementar estrategias que garanticen que su sistema de base de datos satisfaga y anticipe las necesidades de sus usuarios.

MongoDB ofrece un alto grado de control granular sobre estas facetas de su sistema, lo que requiere y recompensa un enfoque reflexivo por parte de los arquitectos de sistemas. Armado con el conocimiento de este capítulo, puede diseñar un sistema que sea rápido, confiable y seguro para los usuarios, y fácil de administrar y escalar para los desarrolladores.

En el próximo capítulo, aprenderá sobre MongoDB Atlas en detalle, explorando sus múltiples comodidades para el desarrollo y despliegue de MongoDB, así como potentes extensiones a la funcionalidad principal de la base de datos.
