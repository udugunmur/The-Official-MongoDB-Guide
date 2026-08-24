# Parte 2: Arquitectura de MongoDB

## Capítulo 2: Arquitectura de MongoDB

MongoDB está diseñado para satisfacer las necesidades cambiantes de las aplicaciones modernas a través de una plataforma de datos centrada en el desarrollador, basada en un conjunto de principios arquitectónicos fundamentales. En este capítulo, exploraremos el marco arquitectónico que impulsa a MongoDB, con un enfoque particular en dos conceptos fundamentales: la replicación y el *sharding* (fragmentación).

La replicación desempeña un papel vital en el diseño distribuido de MongoDB al mejorar tanto la disponibilidad como la tolerancia a fallos. Al mantener varias copias de sus datos en múltiples servidores, MongoDB garantiza que su aplicación pueda continuar funcionando incluso si un servidor falla.

El *sharding* es una estrategia de escalado horizontal para distribuir datos a través de varias máquinas, llamadas fragmentos (*shards*). Cada *shard* almacena una porción del conjunto de datos total en una instancia de servidor de base de datos independiente. A medida que las aplicaciones crecen en popularidad y aumenta el volumen de datos que producen, el escalado entre máquinas se vuelve esencial para garantizar un rendimiento suficiente de lectura y escritura. Cuando se utiliza junto con la replicación, el *sharding* puede ayudar a crear un despliegue de MongoDB robusto y escalable.

Este capítulo cubre los siguientes temas:

- Cómo aumentar la confiabilidad y la disponibilidad
- Mejores prácticas para la replicación y el *sharding*
- Nuevas características en MongoDB 8.0

---

### Sección 1: Replicación

Un conjunto de réplicas (*replica set*) en MongoDB es un grupo de procesos `mongod` que mantienen el mismo conjunto de datos. Proporciona redundancia y alta disponibilidad, y es la base de todos los despliegues en producción. Al tener varias copias de sus datos en distintos servidores de bases de datos, la replicación garantiza un grado de tolerancia a fallos, protegiendo contra la pérdida de un único servidor de base de datos.

Mantener copias de datos en diferentes centros de datos puede aumentar la localidad y disponibilidad de los datos para aplicaciones distribuidas. En algunos casos, la replicación puede proporcionar una mayor capacidad de lectura, ya que los clientes pueden enviar operaciones de lectura a diferentes servidores.

Un conjunto de réplicas incluye un nodo primario, que gestiona todas las operaciones de escritura y registra todos los cambios del conjunto de datos en su registro de operaciones (*operations log* o `oplog`). Los nodos secundarios replican el `oplog` del primario e implementan las operaciones en sus propios conjuntos de datos. Esto asegura que reflejen el conjunto de datos del primario. En caso de que el primario deje de ser accesible, un secundario calificado iniciará una elección para convertirse en el nuevo primario.

Al almacenar datos en múltiples servidores, la replicación aumenta la confiabilidad del sistema. Además, el soporte para actualizaciones graduales (*rolling upgrades*) le permite actualizar el software o hardware de servidores individuales sin interrupciones, garantizando la disponibilidad continua de la base de datos. La replicación mejora significativamente el rendimiento de las aplicaciones con un uso intensivo de lectura, distribuyendo la carga de lectura en múltiples servidores para asegurar una rápida recuperación de datos.

#### Elecciones de conjuntos de réplicas

MongoDB emplea un protocolo construido sobre el algoritmo de consenso Raft para ejecutar las elecciones de conjuntos de réplicas, asegurando la consistencia de los datos en sistemas distribuidos. Este protocolo incluye un mecanismo de votación utilizado por los conjuntos de réplicas para seleccionar qué miembro asumirá el rol primario.

Los conjuntos de réplicas pueden desencadenar una elección en respuesta a una variedad de eventos, como los siguientes:

- Agregar un nuevo nodo al conjunto de réplicas
- Iniciar un conjunto de réplicas
- Realizar mantenimiento en el conjunto de réplicas utilizando un método como `rs.stepDown()` o `rs.reconfig()`
- Miembros secundarios que pierden la conectividad con el primario durante más tiempo del tiempo de espera configurado (10 segundos por defecto para hosts autogestionados o 5 segundos en MongoDB Atlas) pueden iniciar una elección para seleccionar un nuevo primario, siempre que la mayoría de los nodos con derecho a voto coincidan en que el primario actual es inalcanzable.

Debe diseñar la lógica de manejo de conexiones de su aplicación para tener en cuenta las conmutaciones por error automáticas (*failovers*) y las elecciones resultantes. Los controladores de MongoDB pueden identificar la pérdida del primario y reintentar automáticamente operaciones específicas de lectura o escritura, ofreciendo una capa adicional de resiliencia integrada ante elecciones.

Cuando un primario deja de estar disponible, los nodos secundarios votan por un nuevo primario. El nodo con mayor probabilidad de ser elegido se determina por múltiples factores, como su prioridad configurada y la marca de tiempo de escritura más reciente. Esta estrategia minimiza la posibilidad de reversión (*rollback*) cuando un primario anterior se vuelve a unir al conjunto. Tras una elección, los nodos entran en un período de congelación (*freeze period*) que les impide iniciar otra elección. Esto está diseñado para evitar elecciones continuas y rápidas, que pueden desestabilizar el sistema.

> **Figura 2.1:** Elección de un nuevo primario

Los conjuntos de réplicas no pueden ejecutar operaciones de escritura hasta que la elección haya concluido con éxito. Sin embargo, los conjuntos de réplicas aún pueden gestionar operaciones de lectura si están configurados para dirigirse a nodos secundarios.

En circunstancias normales, con la configuración predeterminada del conjunto de réplicas, el tiempo promedio que tarda un clúster en elegir un nuevo primario no debe exceder los 12 segundos. Esta duración abarca el tiempo necesario para declarar al primario como inaccesible y para iniciar y finalizar una elección. Este período de tiempo se puede ajustar alterando la opción de configuración de replicación `settings.electionTimeoutMillis`.

#### Protocolo de elección de replicación

Actualmente, los conjuntos de réplicas de MongoDB solo admiten la versión de protocolo 1 (pv1). pv1 reduce el tiempo de conmutación por error del conjunto de réplicas, acelera la detección de múltiples primarios simultáneos y garantiza la preservación de escrituras confirmadas con `{w: "majority"}`.

Con pv1, puede configurar la opción `settings.catchUpTimeoutMillis` del conjunto de réplicas para priorizar entre conmutaciones por error más rápidas y la preservación de escrituras independientes (*standalone writes*). Valores infinitos o más grandes para `settings.catchUpTimeoutMillis` pueden reducir la cantidad de datos que los otros miembros necesitarían revertir tras una elección, pero pueden aumentar el tiempo de conmutación por error.

En caso de que dos nodos en un conjunto de réplicas crean transitoriamente que son el primario, uno de ellos podrá completar escrituras con la confirmación de escritura (*write concern*) `{w: "majority"}`. Esto puede ocurrir cuando el nodo que puede completar las operaciones de escritura actúa como el primario actual y el otro nodo aún no ha reconocido su degradación, normalmente debido a una partición de red. En este caso, los clientes que se conectan al antiguo primario podrían leer datos obsoletos a pesar de haber solicitado una preferencia de lectura primaria, y las nuevas escrituras en el antiguo primario eventualmente se revertirán (*rollback*).

Además, pv1 realiza un intento de "mejor esfuerzo" para que el secundario disponible con mayor prioridad convoque una elección. Esto puede conducir a elecciones consecutivas, ya que los miembros elegibles con mayor prioridad pueden solicitar una elección. Sin embargo, pv1 limita las elecciones de prioridad para que solo ocurran si el nodo de mayor prioridad está dentro de los 10 segundos respecto al primario actual, y hace que los árbitros voten "no" en las elecciones si detectan un primario en buen estado de prioridad igual o mayor que la del candidato.

#### Latidos (*Heartbeats*)

Los miembros del conjunto de réplicas se envían latidos (*pings*) entre sí cada dos segundos. Si un latido no regresa dentro de los 10 segundos, los demás miembros marcan al miembro infractor como inaccesible.

#### Prioridad de los miembros

Una vez que se establece un primario estable, el algoritmo de elección permite que el secundario de mayor prioridad inicie una elección. La prioridad de los miembros afecta tanto el momento como el resultado de las elecciones, donde los secundarios con mayor prioridad tienen más probabilidades de iniciar una elección antes y ganar. Los miembros de menor prioridad pueden servir brevemente como primarios, a pesar de que haya miembros de mayor prioridad disponibles. El proceso de elecciones continúa hasta que el miembro de mayor prioridad se convierte en el primario. Los miembros con un valor de prioridad de 0 no pueden convertirse en primarios y no buscan ser elegidos.

Dado que un conjunto de réplicas puede admitir un máximo de 50 miembros, con solo 7 de ellos como miembros votantes, la inclusión de miembros no votantes permite que el conjunto supere el límite de 7. Los miembros no votantes, caracterizados por tener cero votos, deben poseer una prioridad de 0.

Para obtener más información sobre los miembros votantes y no votantes, consulte la siguiente documentación: [https://www.mongodb.com/docs/manual/core/replica-set-elections/](https://www.mongodb.com/docs/manual/core/replica-set-elections/).

#### Lecturas reflejadas (*Mirrored reads*)

Las lecturas reflejadas en MongoDB reducen el impacto de las elecciones de primarios después de una interrupción o mantenimiento planificado. Después de una conmutación por error en un conjunto de réplicas, el secundario que asume el control como nuevo primario actualiza su caché a medida que llegan nuevas consultas. Las lecturas reflejadas precalientan las cachés de los miembros secundarios elegibles del conjunto de réplicas haciendo que el primario refleje una muestra de las operaciones compatibles que recibe hacia un subconjunto de secundarios elegibles. Las lecturas reflejadas admiten las siguientes operaciones:

- `count`
- `distinct`
- `find`
- `findAndModify` (específicamente, cuando el filtro se envía como una lectura reflejada)
- `update` (específicamente, cuando el filtro se envía como una lectura reflejada)

Las lecturas reflejadas utilizan una tasa de muestreo predeterminada de 0.01. Puede configurar el tamaño del subconjunto de miembros secundarios elegibles del conjunto de réplicas que reciben lecturas reflejadas configurando el parámetro `mirrorReads`. Si establece una tasa de muestreo entre 0.0 y 1.0, el primario reenvía una muestra aleatoria de las lecturas admitidas a la tasa de muestreo especificada a los secundarios elegibles. Si establece una tasa de muestreo de 1.0, el primario reenvía todas las lecturas admitidas a los secundarios elegibles.

Para deshabilitar las lecturas reflejadas, puede establecer el parámetro `mirrorReads` en 0.0:

```javascript
db.adminCommand({ setParameter: 1, mirrorReads: { samplingRate: 0.0 } })
```

#### Pérdida de un centro de datos

Con un conjunto de réplicas distribuido, la pérdida de un centro de datos puede afectar la capacidad de los miembros restantes en otros centros de datos para elegir un primario. Si es posible, distribuya los miembros del conjunto de réplicas entre centros de datos para maximizar la probabilidad de que, incluso con la pérdida de un centro de datos, uno de los miembros restantes del conjunto de réplicas pueda convertirse en el nuevo primario.

#### Partición de red

Una partición de red puede aislar a un primario en una partición con una minoría de nodos. Cuando el primario detecta que solo puede ver una minoría de nodos votantes en el conjunto de réplicas, el primario renuncia (*steps down*) y se convierte en secundario. Si esto sucede, un miembro en la partición que puede comunicarse con la mayoría de los nodos votantes puede celebrar una elección para convertirse en el nuevo primario.

#### Oplog del conjunto de réplicas

El `oplog` es una colección limitada (*capped collection*) única que mantiene un registro continuo de todas las operaciones que alteran los datos alojados en las bases de datos de MongoDB. Puede expandirse más allá de su límite de tamaño establecido para evitar la eliminación del punto de confirmación por mayoría (*majority commit point*).

Las operaciones de escritura en MongoDB se ejecutan en el primario y posteriormente se registran en el `oplog` del primario. Los miembros secundarios replican y aplican estas operaciones de forma asíncrona. Cada miembro del conjunto de réplicas contiene una copia del `oplog`, ubicada en la colección `local.oplog.rs`, lo que les permite mantenerse al día con el estado actual de la base de datos.

Para admitir la replicación, todos los miembros del conjunto de réplicas intercambian latidos entre sí. Cualquier miembro secundario puede importar entradas de `oplog` de cualquier otro miembro. Cada operación dentro del `oplog` es idempotente, lo que significa que, ya sea que se aplique una o varias veces al conjunto de datos de destino, las operaciones del `oplog` producen el mismo resultado.

#### Búferes del oplog

A partir de MongoDB 8.0, los secundarios escriben y aplican entradas de `oplog` para cada lote de datos en paralelo. Para cada lote de entradas de `oplog`, MongoDB utiliza dos búferes:

- El **búfer de escritura** recibe nuevas entradas de `oplog` del primario, agrega las entradas al `oplog` local y las envía al búfer de aplicación.
- El **búfer de aplicación** recibe nuevas entradas de `oplog` del búfer de escritura y utiliza estas entradas para actualizar la base de datos local.

#### La ventana del oplog (*Oplog window*)

Las entradas del `oplog` llevan marcas de tiempo. La ventana del `oplog` se refiere al intervalo de tiempo entre las marcas de tiempo más reciente y más antigua en el `oplog`. Si un nodo secundario pierde la conectividad con el primario, solo podrá resincronizarse mediante replicación si la conexión se restablece dentro de la duración de la ventana del `oplog`. MongoDB le permite definir tanto una duración mínima (en horas) como un tamaño específico para retener una entrada de `oplog`.

Si el `oplog` se ha llenado hasta su capacidad máxima configurada y la entrada del `oplog` ha superado el período de retención especificado según el reloj del sistema host, el sistema descartará el `oplog`. Además, sin un período mínimo de retención de `oplog` especificado, MongoDB comienza a truncar el `oplog` a partir de las entradas más antiguas, asegurando que el `oplog` no exceda el tamaño máximo configurado.

#### Tamaño del oplog

Cuando inicia un miembro del conjunto de réplicas por primera vez, MongoDB crea un `oplog` de un tamaño predeterminado si no especifica el tamaño del `oplog` con la opción `oplogSizeMB`.

Consulte la siguiente tabla para conocer el tamaño predeterminado del `oplog` en diferentes sistemas operativos:

| Sistema Operativo | Motor de Almacenamiento | Tamaño Predeterminado del Oplog |
| :--- | :--- | :--- |
| Unix | WiredTiger | 5% del espacio libre en disco |
| Unix | In-memory | 5% de la memoria física |
| Windows | WiredTiger | 5% del espacio libre en disco |
| Windows | In-memory | 5% de la memoria física |
| macOS de 64 bits | WiredTiger | 192 MB de espacio libre en disco |
| macOS de 64 bits | In-memory | 192 MB de memoria física |

> **Tabla 2.1:** Tamaño predeterminado del oplog en diferentes sistemas operativos

El tamaño predeterminado del `oplog` tiene las siguientes restricciones:

- El tamaño mínimo del `oplog` es de 990 MB. Si el 5% del espacio libre en disco o la memoria física, según su motor de almacenamiento, es inferior a 990 MB, el tamaño predeterminado del `oplog` se establece en 990 MB.
- El tamaño máximo predeterminado del `oplog` es de 50 GB. Si el 5% del espacio libre en disco o la memoria física, según su motor de almacenamiento, es superior a 50 GB, el tamaño predeterminado del `oplog` se establece en 50 GB.

En la mayoría de los casos, el tamaño predeterminado del `oplog` es suficiente. Por ejemplo, si un `oplog` representa el 5% del espacio libre en disco y se llena en 24 horas de operaciones, entonces los secundarios pueden dejar de copiar entradas del `oplog` hasta por 24 horas sin quedar demasiado obsoletos para continuar replicando. Sin embargo, la mayoría de los conjuntos de réplicas tienen volúmenes de operación mucho más bajos y sus `oplogs` pueden contener cantidades mucho mayores de operaciones.

Si puede prever que la carga de trabajo de su conjunto de réplicas se asemejará a uno de los siguientes patrones, es posible que desee crear un `oplog` más grande que el tamaño predeterminado. Alternativamente, si su aplicación realiza predominantemente lecturas con una cantidad mínima de operaciones de escritura, un `oplog` más pequeño puede ser suficiente. Para cambiar el tamaño del `oplog` después de iniciar un miembro del conjunto de réplicas, utilice el comando administrativo `replSetResizeOplog`.

Considere establecer un tamaño de `oplog` mayor en los siguientes escenarios de carga de trabajo:

- **Operaciones de actualización masiva (*bulk updates*):** El `oplog` debe traducir las actualizaciones masivas en operaciones individuales para mantener la idempotencia. Esto puede utilizar una gran cantidad de espacio de `oplog` sin un aumento correspondiente en el tamaño de los datos o el uso del disco.
- **Las eliminaciones equivalen a la misma cantidad de datos que las inserciones:** Si elimina aproximadamente la misma cantidad de datos que inserta, el tamaño del registro de operaciones puede volverse grande sin un aumento significativo en el uso del disco.
- **Gran cantidad de actualizaciones en el lugar (*in-place updates*):** Si una parte significativa de la carga de trabajo se compone de actualizaciones que no aumentan el tamaño de los documentos, la base de datos registra una gran cantidad de operaciones que no cambian la cantidad de datos en el disco.

#### Arquitectura de despliegue de conjuntos de réplicas

En entornos de producción, un enfoque común es desplegar MongoDB como un conjunto de réplicas de tres miembros, lo que ofrece tanto redundancia como alta disponibilidad. Si bien es mejor evitar configuraciones excesivamente complejas, la arquitectura debe alinearse en última instancia con los requisitos de su aplicación.

A continuación se presentan algunas mejores prácticas generales a seguir:

- Utilice un número impar de nodos votantes para evitar escenarios de cerebro dividido (*split-brain*) durante una partición de red. Esto asegura que aún se pueda formar una mayoría y continuar aceptando escrituras.
- Si su conjunto de réplicas incluye un número par de miembros votantes, intente agregar otro nodo que contenga datos. Si eso no es factible, puede introducir un árbitro, que participa en las elecciones pero no almacena datos.
- Para tareas especializadas como copias de seguridad o consultas analíticas, considere agregar nodos ocultos (*hidden*) o retrasados (*delayed*). Estos miembros no atenderán lecturas de clientes, pero son valiosos para descargar ciertas cargas de trabajo.
- Para protegerse contra interrupciones en los centros de datos, asegúrese de que al menos un miembro del conjunto de réplicas esté alojado en un centro de datos independiente, lo que mejora la resiliencia frente a fallas regionales.

#### Estrategias para desplegar conjuntos de réplicas

Un conjunto de réplicas puede tener hasta 50 miembros, pero solo 7 miembros votantes. Cuando despliegue su conjunto de réplicas, asegúrese de que haya un número impar de miembros votantes. Si tiene un número par de miembros votantes, despliegue otro miembro votante que contenga datos o, si las limitaciones impiden otro miembro votante que contenga datos, despliegue un árbitro.

#### Árbitro del conjunto de réplicas

Un árbitro participa en las elecciones de primarios, pero no almacena una copia de los datos y no puede convertirse en primario. Como resultado, puede ejecutar un árbitro en un servidor de aplicaciones u otro recurso compartido. Al no tener una copia de los datos, puede ubicar un árbitro en entornos en los que no ubicaría a otros miembros del conjunto de réplicas, según sus políticas de seguridad.

Evite desplegar múltiples árbitros en su conjunto de réplicas. Si bien los árbitros contribuyen a la cantidad de nodos en un conjunto de réplicas utilizados para calcular una mayoría en relación con la confirmación de escritura `{w: "majority"}`, la presencia de múltiples árbitros hace que sea menos probable que una mayoría de nodos que contienen datos esté disponible tras la falla de un nodo. Como resultado, la presencia de múltiples árbitros impide el uso confiable de la confirmación de escritura por mayoría y, en consecuencia, no puede garantizar que las escrituras persistan tras la falla de un nodo primario.

#### Miembros ocultos del conjunto de réplicas (*Hidden replica set members*)

Los miembros ocultos no pueden asumir el rol primario y permanecen invisibles para las aplicaciones cliente. A pesar de ser invisibles, los miembros ocultos conservan la capacidad de participar y votar en las elecciones. Los nodos ocultos se pueden utilizar como miembros dedicados del conjunto de réplicas para realizar operaciones de copia de seguridad o para ejecutar algunas consultas de informes. Para configurar un nodo oculto y evitar que un miembro sea promovido a primario, puede establecer su prioridad en 0 y también configurar el parámetro `hidden` en `true`, como se muestra en el siguiente ejemplo:

```javascript
cfg = rs.conf()
cfg.members[n].hidden = true
cfg.members[n].priority = 0
rs.reconfig(cfg)
```

También puede configurar miembros retrasados (*delayed members*), que son miembros ocultos que replican y aplican operaciones del `oplog` de origen con un retraso deliberado, representando un estado anterior del conjunto. Los miembros retrasados actúan como una copia de seguridad continua o un registro histórico en vivo del conjunto de datos que puede proporcionar una red de seguridad contra errores humanos. Pueden facilitar la recuperación de actualizaciones fallidas de aplicaciones y errores de operadores, como la eliminación accidental de bases de datos y colecciones.

Por ejemplo, si la hora actual es 08:01 y un miembro tiene un retraso establecido en una hora, la operación más reciente en el miembro retrasado no será posterior a las 07:01:

```javascript
cfg = rs.conf()
cfg.members[n].hidden = true
cfg.members[n].priority = 0
cfg.member[n].secondaryDelaySecs = 3600
rs.reconfig(cfg)
```

#### Tolerancia a fallos (*Fault tolerance*)

La tolerancia a fallos se refiere a la capacidad de un conjunto de réplicas para continuar funcionando en caso de que varios miembros no estén disponibles. Puede calcular la tolerancia a fallos de un conjunto como la diferencia entre el número de miembros en el conjunto y la mayoría de miembros votantes necesarios para elegir un primario. Si un conjunto de réplicas no puede elegir un nuevo primario, no puede aceptar operaciones de escritura.

La consideración de la tolerancia a fallos puede ayudar a garantizar una alta disponibilidad, integridad de datos y flexibilidad para su despliegue de MongoDB. Por ejemplo, si tiene una aplicación que requiere un tiempo de actividad continuo y no puede permitirse interrupciones, la tolerancia a fallos garantiza que la base de datos permanezca accesible para su aplicación y sus usuarios.

La tolerancia a fallos se ve afectada por el tamaño del conjunto de réplicas, pero la relación no es directa. Aunque agregar un miembro al conjunto de réplicas no siempre aumenta la tolerancia a fallos, la adición de miembros al conjunto de réplicas puede brindar soporte para funciones dedicadas, como copias de seguridad o informes.

Consulte la siguiente tabla para calcular la tolerancia a fallos para un conjunto de réplicas:

| Número de Miembros | Mayoría Requerida para Elegir un Nuevo Primario | Tolerancia a Fallos |
| :--- | :--- | :--- |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 6 | 4 | 2 |

> **Tabla 2.2:** Tolerancia a fallos calculada para un conjunto de réplicas

#### Agregar capacidad antes de la demanda

Los miembros existentes en un conjunto de réplicas deben tener capacidad disponible para soportar la adición de un nuevo miembro. Siempre debe agregar nuevos miembros antes de que la demanda actual sature la capacidad del conjunto de réplicas.

#### Distribución geográfica

Si bien los conjuntos de réplicas brindan protección básica contra fallas de una sola instancia, los conjuntos de réplicas cuyos miembros están todos ubicados en un solo centro de datos son susceptibles a fallas en el centro de datos. Para proteger sus datos en caso de una falla en el centro de datos, distribuya los miembros del conjunto de réplicas entre centros de datos geográficamente distintos para agregar redundancia y brindar tolerancia a fallos si un centro de datos deja de estar disponible.

Si es posible, utilice un número impar de centros de datos y elija una distribución de miembros que maximice la probabilidad de que, incluso con la pérdida de un centro de datos, los miembros restantes del conjunto de réplicas puedan formar una mayoría o, como mínimo, proporcionar una copia de sus datos.

Al distribuir miembros del conjunto de réplicas entre dos centros de datos en lugar de uno, se aplica lo siguiente:

- Si uno de los centros de datos se cae, los datos aún están disponibles para lecturas.
- Si el centro de datos con la minoría de miembros se cae, el conjunto de réplicas aún puede atender operaciones de escritura y de lectura.
- Si el centro de datos con la mayoría de los miembros se cae, el conjunto de réplicas pasa a ser de solo lectura.

Cuando sea posible, distribuya los miembros entre al menos tres centros de datos. Si el costo del tercer centro de datos es prohibitivo, puede distribuir equitativamente los miembros que contienen datos entre los dos centros de datos y almacenar al miembro restante en la nube, si está permitido.

#### Aplicaciones con uso intensivo de lectura (*Read-heavy applications*)

Los conjuntos de réplicas están diseñados para alta disponibilidad y redundancia. Los miembros secundarios generalmente operan bajo cargas similares a las del primario; sin embargo, debe evitar dirigir lecturas a los secundarios.

Si tiene una aplicación con uso intensivo de lectura y necesita replicar datos en otro clúster para lectura, considere usar otras herramientas, como Mongosync. Mongosync es una herramienta que replica datos y escrituras de un clúster a otro hasta que finaliza la sincronización. Puede utilizar `mongosync` para realizar migraciones de datos únicas entre clústeres de MongoDB con un tiempo de inactividad mínimo.

Para obtener más información sobre `mongosync`, consulte la siguiente documentación: [https://www.mongodb.com/docs/mongosync/current/](https://www.mongodb.com/docs/cluster-to-cluster-sync/current/).

Alternativamente, puede utilizar el modo de preferencia de lectura `secondary` o `secondaryPreferred`. Aunque los datos que lee en un secundario pueden estar obsoletos en comparación con el primario, estas preferencias de lectura pueden resultar útiles en los siguientes escenarios:

- Si tiene un conjunto de réplicas con miembros distribuidos en diferentes ubicaciones geográficas, puede usar `secondaryPreferred` para dirigir las lecturas al nodo secundario más cercano. Esto puede reducir la latencia para los usuarios que están geográficamente más cerca de un nodo secundario que del nodo primario.
- Si tiene una aplicación para análisis o informes, leer desde nodos secundarios puede ayudar a evitar que estas operaciones afecten el rendimiento del nodo primario. Esto puede ser especialmente útil cuando la carga de trabajo de análisis requiere un uso intensivo de lectura y no requiere los datos más actualizados.

#### Ejemplos de arquitecturas de despliegue

La siguiente sección incluye arquitecturas de despliegue de muestra que aprovechan algunas de las estrategias para desplegar conjuntos de réplicas enumeradas en la sección anterior. Cada ejemplo describe los roles de cada miembro, las estrategias utilizadas en la arquitectura y los posibles casos de uso para el tipo de despliegue específico.

##### Primario con dos miembros secundarios

Considere el siguiente ejemplo, donde un conjunto de réplicas de tres miembros utiliza la arquitectura Primario-Secundario-Secundario (PSS):

> **Figura 2.2:** Un conjunto de réplicas con un primario y dos secundarios

Este ejemplo representa el despliegue estándar de un conjunto de réplicas para un sistema de producción. Con dos secundarios, este despliegue proporciona dos copias completas del conjunto de datos en todo momento y tendrá una mayoría si el primario se cae. Cuando un secundario detecta que el primario no está disponible, puede iniciar una elección. Los dos secundarios votan cada uno por uno de los secundarios para ser el primario (generalmente el que tiene la mayor prioridad o el `oplog` más actualizado) y continúan con la operación normal. Cuando el antiguo primario vuelve a estar en línea, se vuelve a unir al conjunto de réplicas como miembro secundario.

##### Primario con un miembro secundario y un árbitro

Considere el siguiente ejemplo, donde un conjunto de réplicas de tres miembros utiliza la arquitectura Primario-Secundario-Árbitro (PSA):

> **Figura 2.3:** Un conjunto de réplicas con un primario, un secundario y un árbitro

Este ejemplo representa una alternativa a la arquitectura de despliegue estándar PSS. Este despliegue se puede utilizar en casos donde circunstancias, como el costo, le impiden agregar un tercer miembro que contenga datos. Dado que el árbitro no contiene una copia de los datos, estos despliegues proporcionan solo una copia completa de los datos. Si el primario no está disponible, el secundario y el árbitro elegirán al secundario para convertirse en el nuevo primario.

Aunque el uso de un árbitro requiere menos recursos, considere los siguientes impactos en el rendimiento antes de decidir desplegar esta configuración:

- Esta configuración puede equivaler a una redundancia y tolerancia a fallos más limitadas, ya que los árbitros no conservan una copia de los datos.
- La confirmación de escritura predeterminada, `{w: "majority"}`, puede causar problemas de rendimiento en los despliegues PSA si un secundario deja de estar disponible o experimenta retrasos.
- Si utiliza una preferencia de lectura global predeterminada de `"majority"` y la confirmación de escritura es un valor numérico menor que el tamaño de la mayoría (2 en este ejemplo), sus consultas pueden devolver datos obsoletos.

##### Conjunto de réplicas de tres miembros distribuido en dos centros de datos

Considere el siguiente ejemplo, donde un conjunto de réplicas de tres miembros distribuye sus miembros en dos centros de datos:

> **Figura 2.4:** Un conjunto de réplicas de tres miembros distribuido en dos centros de datos

Este ejemplo aprovecha los miembros del conjunto de réplicas distribuidos geográficamente. Aunque normalmente recomendamos utilizar un número impar de centros de datos, esta configuración de dos centros de datos tiene más sentido para un conjunto de réplicas de tres miembros para garantizar que haya al menos una copia de los datos disponible para lectura (si no una mayoría) en caso de que se pierda un centro de datos debido a un corte de energía, una interrupción de la red o un desastre natural.

Por ejemplo, si el Centro de Datos 1 se cae, el conjunto de réplicas pasa a ser de solo lectura ya que la mayoría de los miembros se han caído, pero aún se conserva una copia de los datos en el Centro de Datos 2. Debido a que el miembro secundario en el Centro de Datos 2 se ha configurado con una prioridad de 0, no puede convertirse en el primario. Esto es para evitar tener un solo nodo que acepte escrituras como un nuevo primario, lo que podría causar inconsistencia de datos cuando el Centro de Datos 1 vuelva a estar en línea. Si el Centro de Datos 2 se cae, el conjunto de réplicas permanece apto para escritura ya que los miembros en el Centro de Datos 1 constituyen la mayoría con un primario en ejecución.

##### Conjunto de réplicas de cinco miembros distribuido en tres centros de datos

Considere el siguiente ejemplo, donde un conjunto de réplicas de cinco miembros distribuye sus miembros en tres centros de datos:

> **Figura 2.5:** Un conjunto de réplicas de cinco miembros distribuido en tres centros de datos

Similar al ejemplo anterior, esta configuración hace uso de miembros de conjuntos de réplicas distribuidos geográficamente. Sin embargo, con más miembros, podemos distribuir los nodos entre un número impar de centros de datos, lo que aumenta la probabilidad de que, incluso con la pérdida de un solo centro de datos, los miembros restantes del conjunto de réplicas puedan formar una mayoría.

En este ejemplo, si algún centro de datos individual se cae, el conjunto de réplicas permanece apto para escritura ya que los miembros restantes forman una mayoría y pueden celebrar una elección, si es necesario.

Alternativamente, es posible que desee configurar su despliegue para que los miembros de un centro de datos puedan ser elegidos como primarios antes que los miembros de otros centros de datos. La siguiente figura muestra cómo puede establecer valores de prioridad, de modo que los miembros en el Centro de Datos 1 tengan una prioridad más alta que aquellos en el Centro de Datos 2 y, posteriormente, en el Centro de Datos 3:

> **Figura 2.6:** Un conjunto de réplicas de cinco miembros distribuido en tres centros de datos con valores de prioridad modificados

#### Confirmación de escritura (*Write concern*)

Una confirmación de escritura (*write concern*) controla cómo MongoDB reconoce las operaciones de escritura dentro de un conjunto de réplicas. Define el nivel de garantía que el sistema debe proporcionar antes de confirmar que una escritura fue exitosa. Esencialmente, determina cuántos nodos deben registrar la escritura antes de que se considere completa.

Una confirmación de escritura se configura utilizando las siguientes opciones:

```javascript
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

Cada campo en esta estructura desempeña un papel específico en el control de la durabilidad y el comportamiento de las operaciones de escritura:

- **`w`:** Especifica el número requerido de miembros del conjunto de réplicas que deben confirmar la escritura. Puede establecer el campo `w` con los siguientes valores:
  - `0`: Sin confirmación. Esto proporciona la menor durabilidad pero el mayor rendimiento.
  - `1`: Confirmación del nodo primario.
  - `majority`: Confirmación de la mayoría de los miembros del conjunto de réplicas (predeterminado).
  - `<number>`: Confirmación de un número específico de miembros del conjunto de réplicas.
- **`j`:** Solicita confirmación de que la operación de escritura se ha escrito en el diario en disco (*journal*). Si se establece en `true`, la operación de escritura espera la confirmación del diario.
- **`wtimeout`:** Establece un límite de tiempo, en milisegundos, para la confirmación de escritura. Si el nivel de confirmación especificado no se alcanza dentro de este tiempo, la operación de escritura devuelve un error. Este error no significa que la escritura no se completará ni se revertirá; evita que la escritura se bloquee indefinidamente.

Por defecto, MongoDB utiliza la confirmación de escritura `{ w: "majority" }`. La mayoría se calcula como la mitad de los miembros votantes más uno (redondeado hacia abajo). Sin embargo, si el número de nodos votantes que contienen datos es menor que esta mayoría, como en los despliegues que incluyen árbitros, la confirmación de escritura predeterminada efectivamente recurre a `{ w: 1 }`.

A partir de MongoDB 8.0, las operaciones de escritura que utilizan la confirmación de escritura `"majority"` devuelven una confirmación cuando la mayoría de los miembros del conjunto de réplicas han escrito la entrada del `oplog` para el cambio. Esto mejora el rendimiento de las escrituras `"majority"`. En versiones anteriores, estas operaciones esperarían y devolverían una confirmación después de que la mayoría de los miembros del conjunto de réplicas aplicaran el cambio.

También puede establecer una confirmación de escritura global predeterminada tanto en conjuntos de réplicas como en clústeres fragmentados. Este valor predeterminado se aplica a todas las operaciones que no definen explícitamente su propia confirmación de escritura. Para configurar la confirmación de lectura o escritura predeterminada globalmente, utilice el siguiente comando:

```javascript
db.adminCommand({
  setDefaultRWConcern: 1,
  defaultReadConcern: { <read concern> },
  defaultWriteConcern: { <write concern> },
  writeConcern: { <write concern> },
  comment: <any>
})
```

Una confirmación de escritura puede influir tanto en el rendimiento como en la durabilidad de sus datos. Tome los siguientes ejemplos:

- Una confirmación de escritura más baja, como `{ w: 0 }`, puede mejorar el rendimiento al reducir la latencia de la operación de escritura. Sin embargo, pone en riesgo la durabilidad de los datos.
- Una confirmación de escritura más alta, como `{ w: "majority" }`, garantiza que los datos sean duraderos al esperar confirmaciones de la mayoría de los nodos. Sin embargo, esto puede introducir cierta latencia en la operación de escritura.

#### Preferencia de lectura (*Read preference*)

En MongoDB, la preferencia de lectura controla cómo los clientes eligen qué miembros del conjunto de réplicas utilizar para las operaciones de lectura. Por defecto, todas las lecturas se envían al miembro primario en un conjunto de réplicas. Sin embargo, puede configurar la preferencia de lectura para enrutar consultas a miembros secundarios en su lugar, lo que puede ayudar a distribuir la carga, mejorar el rendimiento y aumentar la disponibilidad, especialmente en aplicaciones con uso intensivo de lectura.

MongoDB admite cinco modos de preferencia de lectura:

- **`primary`:** Todas las operaciones de lectura se leen desde el primario actual del conjunto de réplicas (predeterminado).
- **`primaryPreferred`:** Las lecturas se dirigen al miembro primario si está disponible; de lo contrario, se enrutan a miembros secundarios.
- **`secondary`:** Todas las operaciones se leen desde los miembros secundarios del conjunto de réplicas.
- **`secondaryPreferred`:** Las lecturas se dirigen a los miembros secundarios disponibles, si los hay; de lo contrario, se enrutan al miembro primario.
- **`nearest`:** Las operaciones de lectura se dirigen al miembro con la latencia de red más baja, independientemente del estado del miembro.

Cambiar la preferencia de lectura puede influir tanto en el rendimiento como en la disponibilidad de sus datos. Por ejemplo, si utiliza la preferencia de lectura `secondary` o `secondaryPreferred`, se aplica lo siguiente:

- Distribuir las operaciones de lectura a los secundarios puede mejorar el rendimiento al reducir la carga en el primario.
- Si el primario no está disponible, las operaciones de lectura aún pueden ser atendidas por miembros secundarios si el modo de preferencia de lectura lo permite.

Además de los modos de preferencia de lectura, también puede especificar las siguientes opciones:

- **Conjuntos de etiquetas (*Tag sets*):** Los conjuntos de etiquetas permiten asociar preferencias de lectura personalizadas con etiquetas personalizadas para miembros de conjuntos de réplicas. Los clientes pueden entonces orientar las operaciones de lectura a miembros con etiquetas específicas. Considere lo siguiente al crear etiquetas:
  - Si bien siempre se aplican con la preferencia de lectura `secondary`, los conjuntos de etiquetas con preferencias de lectura `primaryPreferred` y `secondaryPreferred` solo se aplican si su aplicación selecciona un secundario para una operación de lectura.
  - Al configurar listas de etiquetas con la preferencia de lectura `nearest`, MongoDB selecciona el miembro coincidente con la latencia de red más baja.
  - No puede configurar esta opción con la preferencia de lectura `primary`.
- **Obsolescencia máxima (*Max staleness*):** La obsolescencia se refiere al retraso en la replicación desde el primario al secundario. Esta opción le permite especificar cuán obsoleto puede ser un secundario antes de que el cliente deje de usarlo para operaciones de lectura. Puede establecer el valor máximo de obsolescencia con la opción `maxStalenessSeconds`. Considere lo siguiente al configurar el valor máximo de obsolescencia:
  - Esta opción está diseñada para aplicaciones que pueden leer de secundarios y buscan evitar leer de secundarios que se han quedado demasiado atrás al replicar las escrituras del primario.
  - No puede configurar esta opción con la preferencia de lectura `primary`.
  - Si la preferencia de lectura incluye tanto un valor `maxStalenessSeconds` como una lista de conjuntos de etiquetas, el cliente filtra primero por obsolescencia y luego por las etiquetas especificadas.

Para aprender cómo configurar conjuntos de etiquetas y el valor de obsolescencia máxima, consulte la siguiente documentación:

- [https://www.mongodb.com/docs/manual/core/read-preference-tags/](https://www.mongodb.com/docs/manual/core/read-preference-tags/)
- [https://www.mongodb.com/docs/manual/core/read-preference-staleness/](https://www.mongodb.com/docs/manual/core/read-preference-tags/)

#### Confirmación de lectura (*Read concern*)

Una confirmación de lectura (*read concern*) determina las propiedades de consistencia y aislamiento de los datos leídos de conjuntos de réplicas y clústeres fragmentados. Al ajustar la confirmación de lectura, puede controlar la visibilidad de los datos en las operaciones de lectura para garantizar el nivel deseado de consistencia y aislamiento para su despliegue.

Consulte la siguiente tabla para obtener información sobre los niveles de confirmación de lectura compatibles:

| Nivel de Read Concern | Descripción | Disponibilidad |
| :--- | :--- | :--- |
| `local` | Devuelve los datos más recientes disponibles para la instancia de MongoDB al inicio de la consulta. El read concern local no garantiza que los datos se hayan escrito en la mayoría de los miembros del conjunto de réplicas. | Predeterminado para lecturas en el primario y secundarios.<br>Disponible para su uso con o sin sesiones causalmente consistentes y transacciones. |
| `available` | Devuelve datos que están disponibles actualmente en el sistema distribuido en el momento de la consulta.<br>Este nivel proporciona la menor latencia pero no garantiza consistencia. | No disponible para su uso con sesiones causalmente consistentes y transacciones. |
| `majority` | Devuelve datos que han sido confirmados por la mayoría de los miembros del conjunto de réplicas.<br>Este nivel garantiza que los documentos devueltos por la operación de lectura sean duraderos, incluso en caso de fallo.<br>Para operaciones en transacciones de múltiples documentos, el read concern majority proporciona sus garantías solo si la transacción se confirma con el write concern majority. | Disponible para su uso con o sin sesiones causalmente consistentes y transacciones. |
| `linearizable` | Devuelve datos que reflejan todas las escrituras exitosas confirmadas por la mayoría que se completaron antes del inicio de la operación de lectura.<br>Este nivel proporciona el nivel más alto de consistencia.<br>Las garantías de read concern linearizable solo se aplican si las operaciones de lectura especifican un filtro de consulta que identifica un único documento.<br>Además, se deben cumplir los siguientes criterios para que el read concern linearizable lea de una instantánea consistente:<br>- La consulta utiliza un campo inmutable como clave de búsqueda.<br>- Ninguna actualización concurrente modifica la clave de búsqueda de la consulta.<br>- La clave de búsqueda tiene un índice único que utiliza la consulta.<br>- Establezca siempre `maxTimeMS` con el read concern linearizable en caso de que la mayoría de los nodos que contienen datos no estén disponibles. | No disponible para su uso con sesiones causalmente consistentes y transacciones.<br>Puede especificar este read concern únicamente para operaciones de lectura en el primario. |
| `snapshot` | Devuelve datos confirmados por la mayoría desde un punto específico en el tiempo a través de todos los miembros del conjunto de réplicas.<br>Este nivel proporciona sus garantías solo si la transacción se confirma con el write concern majority.<br>Si una transacción es parte de una sesión causalmente consistente, al confirmarse la transacción, se garantiza que las operaciones de la transacción habrán leído de una instantánea de datos confirmados por la mayoría que proporciona consistencia causal con la operación inmediatamente anterior al inicio de la transacción.<br>Si una transacción no es parte de una sesión causalmente consistente, al confirmarse la transacción, se garantiza que las operaciones de la transacción habrán leído de una instantánea de datos confirmados por la mayoría. | Disponible para todas las operaciones de lectura dentro de transacciones de múltiples documentos con el read concern establecido a nivel de transacción.<br>También disponible para los siguientes métodos fuera de transacciones de múltiples documentos:<br>- `find`<br>- `aggregate`<br>- `distinct` (en colecciones no fragmentadas) |

> **Tabla 2.3:** Niveles de read concern en MongoDB y su disponibilidad

Para aprender sobre consistencia causal y transacciones en MongoDB, consulte la siguiente documentación:

- [https://www.mongodb.com/docs/manual/core/causal-consistency-read-write-concerns/](https://www.mongodb.com/docs/manual/core/causal-consistency-read-write-concerns/)
- [https://www.mongodb.com/docs/manual/core/transactions/](https://www.mongodb.com/docs/manual/core/causal-consistency-read-write-concerns/)

La confirmación de lectura puede influir tanto en la consistencia como en el aislamiento de sus datos. Tome los siguientes ejemplos:

- Si establece un nivel más alto de confirmación de lectura, como `majority` o `linearizable`, puede asegurarse de que los datos devueltos sean consistentes en todos los miembros del conjunto de réplicas.
- Si utiliza la confirmación de lectura `snapshot`, puede aislar una transacción de escrituras concurrentes, asegurando una vista consistente de los datos durante toda la transacción.

---

### Sección 2: Compactación

Puede aprovechar la compactación para liberar espacio en disco innecesario, ocupado por datos e índices de colecciones, hacia el sistema operativo. Para hacer esto, ejecute el comando `compact` en uno de los nodos secundarios de su despliegue. El comando tiene la siguiente sintaxis:

```javascript
db.runCommand({
  compact: <string>,
  dryRun: <boolean>,
  force: <boolean>,
  freeSpaceTargetMB: <int>,
  comment: <any>
})
```

Puede especificar las siguientes opciones de `compact`:

- **`compact`:** El nombre de la colección en la que desea ejecutar la compactación.
- **`dryRun`:** Si se establece en `true`, devuelve una estimación de cuánto espacio, en bytes, puede reclamar la compactación de la colección de destino. Si `dryRun` está habilitado, el comando `compact` no realiza ningún tipo de compactación y solo devuelve el valor estimado.
- **`force`:** (Opcional) Si está habilitado, obliga a que `compact` se ejecute en el miembro primario de su conjunto de réplicas.
- **`freeSpaceTargetMB`:** (Opcional) Especifica la cantidad mínima de espacio de almacenamiento, en megabytes, que debe ser recuperable para que la compactación continúe. Si no hay suficiente espacio de almacenamiento, la compactación no se ejecuta.
- **`comment`:** (Opcional) Un comentario para adjuntar al comando `compact`. Este comentario aparece junto a los registros del comando `compact` en las salidas de registro.

Para ejecutar la compactación en un clúster que exige autenticación, debe autenticarse como un usuario con la acción de privilegio `compact` en la colección de destino. Antes de ejecutar la compactación, considere lo siguiente:

- El primario no replica el comando de compactación a los nodos secundarios.
- Un nodo secundario puede replicar datos mientras se ejecuta la compactación.
- Las lecturas y escrituras están permitidas mientras se ejecuta la compactación.
- La compactación no es compatible con clústeres Atlas M0 y Flex.

Además, tenga en cuenta que ejecutar la compactación puede generar una sobrecarga de sincronización debido al hecho de que realiza regularmente puntos de control (*checkpoints*) en la base de datos. En bases de datos con mucho tráfico, esto puede retrasar o impedir tareas operativas como la realización de copias de seguridad. Para evitar interrupciones inesperadas, deshabilite la compactación antes de realizar una copia de seguridad.

#### Compactación en segundo plano (*Background compaction*)

Además del comando `compact` independiente, a partir de MongoDB 8.0, puede habilitar la compactación en segundo plano con el comando `autoCompact`. Si hay suficiente espacio de almacenamiento libre disponible, la compactación en segundo plano itera periódicamente a través de todos los archivos disponibles y ejecuta continuamente la compactación sin necesidad de que usted ejecute manualmente el comando `compact` de forma continua. Mientras que el comando `compact` se dirige a una sola colección a la vez para la compactación, el comando `autoCompact` ejecuta la compactación en todas las colecciones del nodo. El comando `autoCompact` tiene los mismos requisitos y consideraciones de rendimiento que el comando `compact`.

El comando tiene la siguiente sintaxis:

```javascript
db.runCommand({
  autoCompact: <boolean>,
  freeSpaceTargetMB: <int>,
  runOnce: <boolean>
})
```

Puede especificar las siguientes opciones con el comando `autoCompact`:

- **`autoCompact`:** Especifica si se debe habilitar la compactación en segundo plano.
- **`freeSpaceTargetMB`:** (Opcional) Especifica la cantidad mínima de espacio de almacenamiento, en megabytes, que debe ser recuperable para que la compactación continúe. Si no hay suficiente espacio de almacenamiento, la compactación no se ejecuta.
- **`runOnce`:** (Opcional) Si está habilitado, la compactación en segundo plano se ejecuta solo una vez a través de cada colección en el nodo. Si una colección no se puede compactar cuando `runOnce` está establecido en `false`, MongoDB omite temporalmente esa colección y continúa compactando las colecciones restantes.

---

### Sección 3: Optimización del rendimiento con TCMalloc

A partir de MongoDB 8.0, MongoDB utiliza una versión actualizada de TCMalloc que emplea cachés por CPU (*per-CPU caches*), las cuales almacenan memoria localmente para un núcleo de CPU específico y reducen drásticamente la fragmentación de la memoria en comparación con las cachés por hilo (*per-thread caches*) utilizadas anteriormente. Como resultado, el nuevo TCMalloc ayuda a reducir la fragmentación de la memoria y hace que su base de datos sea más resistente a cargas de trabajo de alto estrés. Esta fragmentación reducida puede mejorar el rendimiento de lectura de su conjunto de réplicas, especialmente los secundarios que atienden consultas de lectura.

#### Soporte de plataformas

Todos los sistemas operativos compatibles con MongoDB 8.0 también admiten el TCMalloc actualizado, excepto los siguientes:

- RHEL 8/Oracle 8 en las arquitecturas PPC64LE y s390x
- RHEL 9/CentOS 9 / Oracle 9 en la arquitectura PPC64LE
- Todos los sistemas operativos Windows

#### Habilitación de Transparent Huge Pages (THP)

Si está ejecutando un despliegue autogestionado de MongoDB en un sistema Linux, asegúrese de habilitar *Transparent Huge Pages* (THP). Antes de MongoDB 8.0, recomendábamos deshabilitar THP debido a sus impactos negativos en el rendimiento en las cargas de trabajo de bases de datos con la versión anterior de TCMalloc. Sin embargo, la nueva versión de TCMalloc depende en gran medida de THP.

Esto se debe a que la nueva versión de TCMalloc cuenta con un asignador consciente de páginas enormes (*hugepage-aware allocator*), que libera páginas enormes de manera más agresiva al sistema operativo para reducir el tamaño total del *pageheap* de TCMalloc. En otras palabras, esto puede conducir a un uso de memoria más eficiente, una asignación de memoria más rápida y un menor consumo general de memoria por parte de MongoDB.

Para obtener instrucciones detalladas sobre cómo habilitar THP para su sistema Linux, consulte la documentación oficial: [https://www.mongodb.com/docs/manual/administration/tcmalloc-performance/#enable-transparent-hugepages--thp-](https://www.mongodb.com/docs/manual/administration/tcmalloc-performance/#enable-transparent-hugepages--thp-).

#### Habilitación de cachés por CPU

Para verificar si TCMalloc se está ejecutando con cachés por CPU, ejecute el siguiente comando:

```javascript
db.runCommand({ serverStatus: 1 }).tcmalloc
```

En la salida del comando `serverStatus`, asegúrese de lo siguiente:

- `tcmalloc.usingPerCPUCaches` es `true`
- `tcmalloc.tcmalloc.cpu_free` es un valor mayor que 0

Si descubre que las cachés por CPU no están habilitadas para su despliegue, asegúrese de haber deshabilitado `glibc rseq` y de estar utilizando la versión del kernel de Linux 4.18 o posterior. El nuevo TCMalloc requiere `rseq` para implementar cachés por CPU, y si otra aplicación, como la biblioteca `glibc`, registra una estructura `rseq` antes de TCMalloc, TCMalloc no puede usar `rseq`.

#### Métodos de replicación

Además de las funciones de replicación integradas, el MongoDB Shell (`mongosh`) proporciona un conjunto de métodos auxiliares (*helper methods*) diseñados para facilitar la interacción con un conjunto de réplicas. Estos métodos auxiliares sirven como herramientas cruciales para gestionar un despliegue replicado de MongoDB. Examinemos algunos de los métodos importantes necesarios para gestionar la replicación en MongoDB:

- **`rs.add()`:** Agrega un miembro a un conjunto de réplicas.
- **`rs.addArb()`:** Agrega un árbitro a un conjunto de réplicas.
- **`rs.conf()`:** Muestra el documento de configuración del conjunto de réplicas.
- **`rs.initiate()`:** Inicializa un nuevo conjunto de réplicas.
- **`rs.printReplicationInfo()`:** Resumen del estado del conjunto de réplicas visto desde el primario.
- **`rs.status()`:** Proporciona un documento que detalla el estado actual del conjunto de réplicas.
- **`rs.reconfig()`:** Reconfigura un conjunto de réplicas existente, sobrescribiendo la configuración existente.
- **`rs.stepDown()`:** Desencadena una transición donde el primario se convierte en secundario y los secundarios elegibles celebran una elección.

Para obtener una lista detallada de los métodos de replicación, visite el siguiente recurso: [https://www.mongodb.com/docs/manual/reference/method/js-replication/](https://www.mongodb.com/docs/manual/reference/method/js-replication/).

#### Comandos de replicación

Además de los métodos de replicación anteriores, también puede utilizar comandos de replicación para operaciones específicas. Por ejemplo, `replSetResizeOplog` cambia el tamaño del `oplog` variable en un conjunto de réplicas. El comando toma los siguientes campos:

- **`replSetResizeOplog`:** Establecido en `1`.
- **`size`:** Especifica el tamaño máximo del `oplog` en megabytes. El tamaño mínimo que se puede establecer es de 990 MB y el máximo es de 1 PB. Convierta explícitamente el tamaño como un valor `double` en `mongosh` utilizando `Double()`.
- **`minRetentionHours`:** Representa el número mínimo de horas para retener una entrada de `oplog`, donde los valores decimales indican fracciones de una hora (por ejemplo, 1.5 significa 1 hora y 30 minutos). El valor debe ser igual o mayor que 0. Un valor de 0 significa que `mongod` debe truncar el `oplog` comenzando con las entradas más antiguas para mantener el tamaño máximo configurado del `oplog`.

Puede utilizar la siguiente sintaxis con los campos relevantes configurados como se muestra a continuación:

```javascript
db.adminCommand({
  replSetResizeOplog: <int>,
  size: <double>,
  minRetentionHours: <double>
})
```

Para obtener más información sobre los comandos de replicación, visite el siguiente recurso: [https://www.mongodb.com/docs/manual/reference/command/nav-replication/](https://www.mongodb.com/docs/manual/reference/command/nav-replication/).

---

### Sección 4: Replicación frente a sharding (fragmentación)

La replicación y el *sharding* a menudo se confunden entre sí, pero cumplen funciones fundamentalmente diferentes. La replicación implica crear múltiples copias de los mismos datos en diferentes servidores. Su objetivo principal es proporcionar tolerancia a fallos, alta disponibilidad y redundancia, asegurando que su aplicación permanezca operativa incluso si un nodo se cae. Por otro lado, el *sharding* es un método de escalado horizontal que divide un conjunto de datos grande en partes más pequeñas y manejables, llamadas fragmentos (*shards*), y distribuye esos fragmentos en múltiples instancias de servidor. Cada *shard* contiene solo un subconjunto de los datos generales, lo que mejora el rendimiento y permite que el sistema maneje más datos de los que un solo servidor podría gestionar.

Sin embargo, la replicación y el *sharding* trabajan juntos en el sentido de que cada fragmento debe implementar replicación para mantener la integridad y disponibilidad de los datos. Cada *shard* en un clúster fragmentado debe configurarse como un conjunto de réplicas para protegerse contra la pérdida de datos y las interrupciones del sistema. Si el servidor primario de un *shard* falla y no existe una copia replicada, cualquier dato almacenado exclusivamente en ese *shard* se vuelve temporalmente inaccesible. Al replicar los datos de cada *shard* en múltiples nodos, MongoDB garantiza que el clúster pueda continuar funcionando incluso si servidores individuales se desconectan.

Esta arquitectura también admite actualizaciones graduales (*rolling upgrades*), que son actualizaciones que se pueden aplicar a un nodo a la vez sin tiempo de inactividad, lo que permite un mantenimiento fluido del sistema sin interrupción del servicio.

La siguiente sección examinará en profundidad el *sharding* y sus componentes.

---

### Sección 5: Sharding (Fragmentación)

MongoDB admite el escalado horizontal a través del *sharding*, lo que implica la distribución de datos en múltiples servidores, divididos en partes más pequeñas conocidas como *shards*. El *sharding* juega un papel esencial en la gestión y organización de datos a gran escala.

Además, el *sharding* permite la creación de bases de datos distribuidas para admitir aplicaciones distribuidas geográficamente, habilitando políticas que hacen cumplir la residencia de datos dentro de regiones específicas.

#### ¿Por qué necesita sharding?

Imagine que los datos de su aplicación están creciendo rápidamente y comienzan a superar los límites de su base de datos. A medida que aumenta el volumen de datos, es posible que encuentre una serie de problemas, siendo el más inmediato la degradación del rendimiento. Las consultas que antes arrojaban resultados rápidamente ahora pueden tardar notablemente más, ralentizando su aplicación y frustrando a los usuarios con respuestas lentas o demoradas.

El almacenamiento es otra preocupación crítica. Cada sistema tiene límites prácticos sobre la cantidad de datos que puede almacenar y gestionar de manera eficiente. Si el crecimiento de sus datos supera la capacidad de su infraestructura, corre el riesgo de alcanzar los límites del sistema o incluso sufrir interrupciones. Para manejar este tipo de crecimiento, existen dos estrategias principales:

- **Escalado vertical:** Implica mejorar los recursos de un solo servidor, por ejemplo, agregando más RAM, actualizando a una CPU más rápida o aumentando el espacio en disco. Si bien esto puede hacerle ganar tiempo, existe un límite respecto a cuánto puede escalar una sola máquina. Con el tiempo, las limitaciones del hardware se convierten en un cuello de botella.
- **Escalado horizontal:** Distribuye la carga en múltiples máquinas. En lugar de depender de un servidor potente, el sistema divide los datos y la carga de trabajo entre varios nodos. Este enfoque puede ofrecer una mejor escalabilidad y, a menudo, resulta más rentable que las actualizaciones de hardware de alta gama. Sin embargo, también introduce una mayor complejidad en términos de arquitectura del sistema, distribución de datos y mantenimiento continuo.

#### Elementos clave de un clúster fragmentado

Un clúster fragmentado de MongoDB consta de los siguientes elementos clave:

- **Shard:** Un conjunto de réplicas que almacena una porción de los datos en el clúster. El clúster puede tener de 1 a *n* shards, y cada shard contiene un subconjunto distinto de los datos generales. MongoDB Atlas ofrece diferentes capacidades de almacenamiento para shards según el nivel del clúster (*tier*).
- **mongos:** Un enrutador de consultas para aplicaciones cliente que gestiona eficientemente operaciones de lectura y escritura. Dirige las solicitudes de los clientes a los shards correspondientes y consolida los resultados de múltiples shards en una respuesta de cliente coherente. Los clientes establecen conexiones con instancias de `mongos`, en lugar de directamente con shards individuales. Se recomienda ejecutar múltiples instancias de `mongos` en despliegues de producción para garantizar una alta disponibilidad.
- **Servidores de configuración (*Config servers*):** Un conjunto de réplicas que sirve como repositorio principal para metadatos de fragmentación y ajustes de configuración. Estos metadatos abarcan el estado y la estructura de los datos fragmentados, incluida información esencial como la lista de colecciones fragmentadas y los detalles de enrutamiento. Desempeña un papel crucial al permitir una gestión eficiente de datos y enrutamiento de consultas dentro del clúster. Los servidores de configuración pueden adoptar la forma de un servidor de configuración dedicado o un shard de configuración:
  - Un **servidor de configuración dedicado** es un servidor de configuración independiente sin funcionalidad de servidor de fragmentos.
  - Introducido en MongoDB 8.0, un **shard de configuración (*config shard*)** es un servidor de configuración que también proporciona funcionalidad de servidor de fragmentos, lo que significa que puede almacenar datos de aplicaciones además de los metadatos habituales del clúster fragmentado.

> **Figura 2.7:** Componentes de un clúster fragmentado típico

En MongoDB, la fragmentación de datos ocurre a nivel de colección, lo que significa que los datos de una colección se distribuyen entre múltiples shards dentro del clúster. Es importante tener en cuenta que cada base de datos en un clúster fragmentado tiene su propio shard primario (*primary shard*), que es responsable de almacenar todas las colecciones no fragmentadas dentro de esa base de datos.

Tenga en cuenta también que los shards primarios en clústeres fragmentados son diferentes de los nodos primarios en los conjuntos de réplicas; tienen propósitos diferentes y no están relacionados entre sí.

Al crear una nueva base de datos, el proceso `mongos` selecciona el shard primario. Elige el shard del clúster que contiene la menor cantidad de datos. El campo `totalSize` devuelto por el comando `listDatabases` se utiliza como uno de los factores en los criterios de selección.

Puede mover el shard primario después de su asignación inicial utilizando el comando `movePrimary`. El comando `movePrimary` modifica inicialmente el shard primario en los metadatos del clúster y luego migra todas las colecciones no fragmentadas al shard designado.

#### Shards de configuración (*Config shards*)

A partir de MongoDB 8.0, puede configurar un servidor de configuración para que sea un shard de configuración que almacene los datos de su aplicación además de los metadatos habituales del clúster fragmentado. Cada clúster fragmentado debe tener un servidor de configuración, pero puede adoptar la forma de un shard de configuración o de un servidor de configuración dedicado. No puede utilizar el mismo shard de configuración para múltiples clústeres fragmentados.

Al elegir si utilizar un shard de configuración o un servidor de configuración dedicado, considere lo siguiente:

- Utilice un **shard de configuración** si su clúster tiene tres o menos shards.
- Utilice un **servidor de configuración dedicado** en los siguientes casos:
  - Planea utilizar más de tres shards.
  - Planea utilizar colecciones de series temporales o *queryable encryption*.
  - Planea utilizar copias de seguridad consultables (*queryable backups*).
  - Su aplicación tiene requisitos exigentes de disponibilidad y resiliencia.

#### Conversión de un conjunto de réplicas en un clúster fragmentado con un shard de configuración

Si bien no puede convertir directamente un conjunto de réplicas en un shard de configuración, puede convertir su conjunto de réplicas en un clúster fragmentado con un servidor de configuración dedicado. Una vez que complete la conversión inicial de su conjunto de réplicas a un clúster fragmentado, puede ejecutar el comando `transitionFromDedicatedConfigServer` para configurar su servidor de configuración dedicado para que se ejecute como un shard de configuración:

```javascript
db.adminCommand({ transitionFromDedicatedConfigServer: 1 })
```

Después de ejecutar el comando `transitionFromDedicatedConfigServer`, puede verificar que su servidor de configuración se esté ejecutando como un shard de configuración mediante el comando `listShards` en la base de datos `admin`, mientras está conectado a una instancia de `mongos`:

```javascript
db.adminCommand({ listShards: 1 })["shards"].find(element => element._id === "config")
```

Si su clúster utiliza un shard de configuración, la salida del comando `listShards` debe ser similar al siguiente documento, que contiene un valor `_id` de `"config"`:

```json
{
  _id: "config",
  host: "configRepl/localhost:27018",
  state: 1,
  topologyTime: Timestamp({ t: 1732218671, i: 13 }),
  replSetConfigVersion: Long('-1')
}
```

Para conocer los procedimientos detallados sobre cómo convertir un conjunto de réplicas en un clúster fragmentado con un shard de configuración, consulte el siguiente tutorial: [https://www.mongodb.com/docs/manual/tutorial/convert-replica-set-to-embedded-config-server/](https://www.mongodb.com/docs/manual/tutorial/convert-replica-set-to-embedded-config-server/).

#### Transición de un shard de configuración a un servidor de configuración dedicado

Si su clúster fragmentado utiliza un shard de configuración, pero luego descubre que un servidor de configuración dedicado puede adaptarse mejor a la carga de trabajo de su aplicación, puede ejecutar el comando `transitionToDedicatedConfigServer` en la base de datos `admin` de una instancia de `mongos` para convertir su shard de configuración en un servidor de configuración dedicado:

```javascript
db.adminCommand({ transitionToDedicatedConfigServer: 1 })
```

Cuando convierte un shard de configuración en un servidor de configuración dedicado, MongoDB mueve todos los datos de la aplicación desde el shard de configuración a los otros shards del clúster.

Esta capacidad de realizar la transición entre el shard de configuración y el servidor de configuración dedicado proporciona una mayor flexibilidad para que configure su aplicación con respecto a los requisitos cambiantes de la carga de trabajo y el volumen de datos.

#### Ventajas del sharding

Con una comprensión de sus componentes clave, podemos comprender mejor los beneficios que puede proporcionar el *sharding* al gestionar y optimizar su despliegue de MongoDB. Con la capacidad de distribuir datos entre shards (a diferencia de un conjunto de réplicas independiente, donde cada nodo contiene los mismos datos), el *sharding* proporciona varias ventajas, tales como:

- **Velocidad de lectura y escritura mejorada:** Distribuir su conjunto de datos en múltiples shards permite el procesamiento en paralelo. Cada shard adicional aumenta su rendimiento total y un conjunto de datos bien distribuido mejora el rendimiento de las consultas. Múltiples shards pueden procesar consultas simultáneamente, acelerando los tiempos de respuesta.
- **Capacidad de almacenamiento ampliada:** Aumentar la cantidad de shards mejora su almacenamiento total. Si un shard contiene 4 TB de datos, cada shard adicional agrega otros 4 TB. Esto permite una capacidad de almacenamiento casi ilimitada, facilitando la escalabilidad a medida que aumentan sus necesidades de datos.
- **Localidad de datos:** El *zone sharding* le permite distribuir bases de datos en diferentes ubicaciones geográficas, lo que lo hace ideal para aplicaciones distribuidas. Las políticas pueden confinar datos dentro de regiones específicas, donde cada región contiene uno o más shards, mejorando la eficiencia y la adaptabilidad en la gestión de datos. En el *zone sharding*, se pueden asociar diferentes rangos de valores de claves de fragmentación con cada zona, lo que permite un acceso a los datos más rápido y preciso según la relevancia geográfica.
- **Mayor disponibilidad:** Si uno o más shards no están disponibles, el clúster fragmentado puede continuar realizando lecturas y escrituras parciales en los shards restantes disponibles. Esto difiere de los conjuntos de réplicas en que la falla de un primario en un conjunto de réplicas puede afectar temporalmente la disponibilidad de escritura para todo el conjunto hasta que se elija un nuevo primario. Dado que los clústeres fragmentados distribuyen datos en múltiples shards, los otros shards aún pueden atender solicitudes para el subconjunto de datos que contienen.

#### Distribución de datos

La distribución correcta de datos en un clúster fragmentado de MongoDB es vital para evitar cuellos de botella y garantizar un uso eficiente de los recursos.

Algunos componentes clave para la distribución de datos fragmentados incluyen los siguientes:

- **Clave de fragmentación (*Shard key*):** Un campo que MongoDB utiliza para distribuir documentos entre los miembros de un clúster fragmentado.
- **Zonas (*Zones*):** Una agrupación de documentos basada en rangos de valores de clave de fragmentación para una colección fragmentada.
- **Trozo (*Chunk*):** Un rango contiguo de valores de clave de fragmentación dentro de un shard.
- **Rango (*Range*):** Un rango contiguo de valores de clave de fragmentación dentro de un chunk. Un rango puede ser una porción de un chunk o el chunk completo.
- **Balanceador (*Balancer*):** Un proceso interno de MongoDB que monitorea la cantidad de datos en cada shard y migra datos entre shards para intentar lograr una distribución equitativa entre ellos.

Las siguientes secciones proporcionan información más detallada sobre estos componentes clave.

#### Clave de fragmentación (*Shard key*)

MongoDB realiza la fragmentación a nivel de colección, lo que le permite elegir qué colecciones fragmentar. Las claves de fragmentación determinan la distribución de los documentos de la colección entre los shards del clúster. Pueden adoptar la forma de un único campo indexado o de múltiples campos cubiertos por un índice compuesto. Aprenderá más sobre los índices en el [Capítulo 4](https://subscription.packtpub.com/book/data/9781837021970/4), Modelado de datos y optimización de índices.

Elegir la clave de fragmentación adecuada es una de las decisiones más críticas al fragmentar. Una clave de fragmentación mal elegida puede provocar una distribución de datos ineficiente, una distribución de carga desequilibrada entre shards y un menor rendimiento de las consultas. Estos problemas pueden sobrecargar algunos shards y subutilizar otros, reduciendo la eficiencia del sistema. En casos extremos, un solo shard, conocido como *hot shard*, podría convertirse en un cuello de botella que afecte significativamente el rendimiento del clúster. Para evitar estos impactos en el rendimiento, elija la clave de fragmentación adecuada para optimizar el rendimiento de su clúster fragmentado.

Para abordar los problemas derivados de una clave de fragmentación subóptima, MongoDB le permite cambiar su clave de fragmentación ya sea refragmentando una colección (*resharding*) o refinando una clave de fragmentación.

#### Refragmentar una colección (*Resharding a collection*)

Las correcciones en la distribución de datos son más efectivas cuando se refragmenta una colección. Si desea mejorar la distribución de datos y su clúster cumple con los siguientes criterios, debe refragmentar la colección:

- Su aplicación puede tolerar un período de dos segundos en el que la colección afectada bloquea las escrituras debido a un aumento en la latencia.
- El espacio de almacenamiento disponible en cada shard en el que se distribuirá la colección es al menos el doble del tamaño de la colección que desea refragmentar y el tamaño total de sus índices, dividido por el número de shards:

```javascript
storage_req = ((collection_storage_size + index_size) * 2) / shard_count
```

- Asegúrese de que la capacidad de E/S (*I/O capacity*) de su base de datos sea inferior al 50%.
- Asegúrese de que la carga de su CPU sea inferior al 80%.

La base de datos no impone estos requisitos. No asignar suficientes recursos puede provocar lo siguiente:

- La base de datos se queda sin espacio y se apaga
- Disminución del rendimiento
- La operación tarda más de lo esperado: en este caso, considere desplegar un miembro de cada conjunto de réplicas en un sitio adecuado como ubicación de recuperación ante desastres

Si su clúster no cumple con los criterios para refragmentar, considere refinar la clave de fragmentación en su lugar.

Puede utilizar el comando `reshardCollection` para cambiar su clave de fragmentación o redistribuir sus datos en su clúster. Antes de MongoDB 8.0, tenía que especificar una nueva clave de fragmentación al ejecutar el comando `reshardCollection`; de lo contrario, el comando no tendría ningún efecto. Sin embargo, a partir de MongoDB 8.0, puede especificar la opción `forceRedistribution` para refragmentar una colección sobre la misma clave de fragmentación.

Refragmentar una colección sobre la misma clave de fragmentación puede resultar útil en el escenario en el que acaba de agregar un shard a su clúster y necesita redistribuir sus datos en el nuevo shard. Anteriormente, para redistribuir datos sin cambiar la clave de fragmentación, MongoDB ejecutaba una migración de rango, que era un proceso mucho más lento que la versión actual de *resharding*.

#### Refinar una clave de fragmentación (*Refining a shard key*)

Refinar la clave de fragmentación de una colección permite una distribución de datos más granular y puede abordar situaciones en las que la clave existente ha dado lugar a *jumbo chunks* debido a una cardinalidad insuficiente. Para refinar la clave de fragmentación de una colección, utilice el comando `refineCollectionShardKey`. Este comando agrega uno o varios campos sufijo a la clave existente para crear una nueva clave de fragmentación.

Por ejemplo, es posible que tenga una colección `orders` existente en una base de datos `test` con la clave de fragmentación `{ customer_id: 1 }`. Puede utilizar el comando `refineCollectionShardKey` para cambiar la clave de fragmentación a la nueva clave `{ customer_id: 1, order_id: 1 }`:

```javascript
db.adminCommand({
  refineCollectionShardKey: "test.orders",
  key: { customer_id: 1, order_id: 1 }
})
```

#### Chunks (Trozos)

MongoDB organiza los datos fragmentados en secciones distintas llamadas *chunks*. Cada *chunk* se caracteriza por un límite inferior inclusivo (`minKey`) y un límite superior exclusivo (`maxKey`), que están definidos por la clave de fragmentación. Estos *chunks* contienen una secuencia ininterrumpida de valores de clave de fragmentación dentro de un shard específico y solo se dividen cuando se mueven entre shards.

#### Chunks gigantes e indivisibles (*Jumbo and indivisible chunks*)

En MongoDB, cuando un *chunk* crece más allá del tamaño máximo configurado pero no se puede dividir automáticamente, se etiqueta como *jumbo*. Este estado *jumbo* indica que el *chunk* puede causar problemas de balanceo, ya que no se puede dividir y mover fácilmente.

La forma preferida de borrar el indicador *jumbo* es dividir manualmente el *chunk*. Si el *chunk* se puede dividir con éxito en partes más pequeñas, MongoDB eliminará la designación *jumbo*. Puede realizar esta operación utilizando el método `sh.splitAt()` o `sh.splitFind()`, lo que le permite especificar dónde debe ocurrir la división.

A veces, MongoDB no puede dividir más un *chunk* gigante. Esto sucede a menudo cuando el *chunk* contiene documentos que comparten todos el mismo valor de clave de fragmentación, lo que lo hace indivisible. En estas situaciones, tiene un par de opciones: puede modificar la clave de fragmentación y realizar una refragmentación de la colección para crear *chunks* más granulares, o puede borrar manualmente el indicador *jumbo* para reconocer el tamaño del *chunk* sin dividirlo.

Para borrar manualmente el indicador, ejecute `clearJumboFlag` en la base de datos `admin`, proporcione el espacio de nombres (*namespace*) de la colección fragmentada y proporcione uno de los siguientes:

- Los límites (*bounds*) del chunk gigante:

```javascript
db.adminCommand({ clearJumboFlag: "sample.customers", bounds: [{ "x": 5 }, { "x": 6 }] })
```

- Un documento `find` con una clave de fragmentación y un valor que se encuentre dentro del chunk gigante:

```javascript
db.adminCommand({ clearJumboFlag: "sample.customers", find: { "x": 5 } })
```

En el caso de que la colección emplee una clave de fragmentación con hash (*hashed shard key*), evite usar el campo `find` con el comando `clearJumboFlag`. Para colecciones con claves de fragmentación hash, es más apropiado especificar el campo `bounds`.

#### Rangos (*Ranges*)

Los rangos en MongoDB están estrechamente relacionados con los *chunks* en el sentido de que un rango puede ser una porción del *chunk* o el *chunk* completo en sí. MongoDB migra rangos de datos automáticamente cuando hay una distribución desigual de datos de colecciones fragmentadas entre los shards.

El tamaño predeterminado de un rango en MongoDB 8.0 es de 128 MB. Aunque este tamaño de rango predeterminado funciona de manera suficiente para la mayoría de los despliegues, puede considerar modificar este valor debido a requisitos de hardware o de carga de trabajo. Por ejemplo, si nota que las migraciones automáticas utilizan más E/S de la que su hardware puede manejar, es posible que desee reducir el tamaño del rango para permitir migraciones más rápidas y frecuentes. Puede especificar un valor de tamaño de rango entre 1 y 1,024 megabytes, ambos inclusive.

#### Rangos pre-divididos (*Pre-split ranges*)

En la mayoría de los casos, un clúster fragmentado de MongoDB crea, divide y distribuye rangos de datos automáticamente entre los shards para equilibrar la carga. Sin embargo, hay situaciones en las que es posible que MongoDB no genere suficientes rangos con la suficiente rapidez para mantenerse al día con las demandas de rendimiento de su aplicación.

Si se está preparando para cargar un gran volumen de datos en un nuevo clúster fragmentado, pre-dividir los rangos puede ayudar a garantizar una distribución equitativa de los datos desde el principio. Este paso proactivo evita que cualquier shard individual se convierta en un cuello de botella durante la ingesta masiva de datos.

Solo debe pre-dividir rangos para una colección que esté actualmente vacía. Intentar dividir rangos manualmente para una colección que ya contiene datos puede generar tamaños y límites de rango irregulares. También puede provocar que el comportamiento de equilibrio funcione de manera ineficiente o no funcione en absoluto.

Para dividir manualmente rangos vacíos, use el comando `moveRange` para distribuir los rangos vacíos entre los shards de su clúster. Para obtener un ejemplo detallado de cómo mover rangos manualmente, consulte [https://www.mongodb.com/docs/manual/tutorial/create-chunks-in-sharded-cluster/#std-label-create-ranges-in-a-sharded-cluster](https://www.mongodb.com/docs/manual/tutorial/create-chunks-in-sharded-cluster/#std-label-create-ranges-in-a-sharded-cluster).

#### Zonas (*Zones*)

Puede crear zonas de datos fragmentados según la clave de fragmentación de un clúster. Cada shard en un clúster puede estar en una o más zonas. En un clúster equilibrado, MongoDB dirige las lecturas y escrituras de una zona solo a los shards que están dentro de esa zona. Debería considerar aplicar zonas en su despliegue si necesita hacer lo siguiente:

- Aislar un subconjunto específico de datos en un conjunto específico de shards.
- Asegurarse de que los datos más relevantes residan en shards ubicados geográficamente más cerca de los servidores de aplicaciones.
- Enrutar datos a shards según el hardware o su rendimiento.

Cada zona cubre uno o más rangos de valores de clave de fragmentación para una colección, y cada rango que cubre una zona incluye su límite inferior y excluye su límite superior. Cuando define zonas y rangos de zonas antes de fragmentar una colección vacía o inexistente, la operación de fragmentación de colección crea *chunks* para los rangos de zonas definidos, así como cualquier *chunk* adicional que cubra el rango completo de valores de clave de fragmentación. Luego realiza una distribución inicial de *chunks* basada en los rangos de zona, lo que permite una configuración más rápida del *zoned sharding*.

#### Balanceador (*Balancer*)

Los datos en clústeres fragmentados se distribuyen según el tamaño de los datos. Para garantizar una distribución equilibrada de los datos, un proceso en segundo plano llamado balanceador realiza un seguimiento del volumen de datos en cada shard para cada colección fragmentada. Una vez que la cantidad de datos para una colección fragmentada en un shard particular alcanza un límite de migración específico, el balanceador intenta redistribuir los datos entre los shards, buscando una distribución equitativa de datos por shard mientras respeta las zonas previamente definidas. Por defecto, este proceso de balanceo siempre está habilitado y se ejecuta en el primario del conjunto de réplicas del servidor de configuración (*Config Server Replica Set* o CSRS).

El proceso de balanceo para clústeres fragmentados es completamente invisible para el usuario y la capa de aplicación, aunque se pueden observar efectos menores en el rendimiento durante la ejecución. El balanceador intenta mitigar este impacto limitando un shard para que participe en solo una migración a la vez. El balanceador realiza migraciones de rango una tras otra. Con un clúster fragmentado que consta de *n* shards, MongoDB puede realizar hasta *n*/2 (redondeado hacia abajo) migraciones concurrentes, siempre que no se realicen en el mismo shard.

Otra forma en que el balanceador intenta minimizar el impacto en el rendimiento es iniciando una ronda de balanceo solo cuando la discrepancia de datos entre el shard más cargado y el shard menos cargado para una colección fragmentada alcanza el umbral de migración.

Una colección se considera equilibrada cuando la variación de datos entre shards para esa colección específica es inferior a tres veces el tamaño de rango establecido para la colección. Por ejemplo, si el tamaño del rango se establece en el valor predeterminado de 128 MB, se desencadenará una migración si la diferencia de tamaño de datos entre dos shards cualesquiera para una determinada colección es de al menos 384 MB.

#### Ventana de balanceo (*Balancing window*)

También puede establecer un período de tiempo específico para el funcionamiento del balanceador a fin de evitar que interfiera con el tráfico de producción. Esto se denomina ventana de balanceo. Es posible que desee programar una ventana de balanceo en situaciones en las que su conjunto de datos crezca lentamente y una migración pueda afectar el rendimiento. Si especifica una ventana de balanceo diferente, debe ser suficiente para completar la migración de todos los datos insertados durante el día.

Para especificar un intervalo de tiempo de la ventana de balanceo, realice los siguientes pasos:

1. Conéctese a cualquier instancia de `mongos` en el clúster utilizando `mongosh`.
2. Cambie a la base de datos `config`:

```javascript
use config
```

3. Ejecute el comando `sh.startBalancer()` para asegurarse de que el balanceador no esté detenido.
4. Establezca el campo `activeWindow` utilizando el comando `updateOne()`:

```javascript
db.settings.updateOne(
  { _id: "balancer" },
  { $set: { activeWindow: { start: "<start-time>", stop: "<end-time>" } } },
  { upsert: true }
)
```

Reemplace `<start-time>` y `<end-time>` con valores de tiempo utilizando valores de horas y minutos de dos dígitos (es decir, `HH:MM`) que especifiquen los límites inicial y final de la ventana de balanceo:

- Para los valores `HH`, utilice valores de hora que van de 00 a 23.
- Para los valores `MM`, utilice valores de minutos que van de 00 a 59.

Para clústeres fragmentados autogestionados, MongoDB evalúa las horas de inicio y finalización en relación con la zona horaria del miembro primario en el CSRS. Para clústeres de Atlas, MongoDB evalúa las horas de inicio y finalización en relación con la zona horaria UTC.

Para obtener información más detallada sobre la administración del balanceador, consulte la siguiente documentación: [https://www.mongodb.com/docs/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-schedule-balancing-window](https://www.mongodb.com/docs/manual/tutorial/manage-sharded-cluster-balancer/#std-label-sharding-schedule-balancing-window).

#### Estrategias de sharding

MongoDB admite dos estrategias de fragmentación para distribuir datos en clústeres fragmentados: fragmentación por rango (*ranged sharding*) y fragmentación por hash (*hashed sharding*).

#### Fragmentación por rango (*Ranged sharding*)

La fragmentación por rango implica dividir los datos en secuencias continuas según los valores de la clave de fragmentación. Es probable que los documentos con valores de clave de fragmentación similares estén en el mismo shard o chunk. Esto permite realizar consultas eficientes donde las lecturas se dirigen a documentos dentro de un rango contiguo. La fragmentación basada en rangos es el método predeterminado a menos que se configuren opciones para fragmentación por hash o zonas.

La fragmentación por rango es más eficiente cuando la clave de fragmentación tiene una combinación de estas características:

- **Alta cardinalidad:** La alta cardinalidad permite a MongoDB crear más *chunks*, lo que le da al balanceador una mayor flexibilidad para distribuir equitativamente los datos y la carga de trabajo entre los shards. Siempre que sea posible, elija una clave de fragmentación con alta cardinalidad. Una clave de baja cardinalidad limita la cantidad de *chunks* que se pueden crear, lo que a su vez restringe la efectividad del escalado horizontal y puede generar cuellos de botella en el rendimiento.
- **Baja frecuencia:** La frecuencia se refiere a la frecuencia con la que aparece cada valor de clave de fragmentación en el conjunto de datos. Idealmente, los valores de las claves de fragmentación deberían distribuirse uniformemente. Si una gran parte de los documentos comparte solo unos pocos valores comunes, esto puede dar lugar a *chunks* de gran tamaño concentrados alrededor de esos valores. Estos *chunks* desequilibrados pueden sobrecargar shards específicos y convertirse en cuellos de botella de rendimiento. Si estos *chunks* no se pueden dividir más porque todos los documentos comparten el mismo valor de clave de fragmentación, se convierten en *jumbo chunks*, lo que impide un reequilibrio eficiente y reduce la escalabilidad general del clúster.
- **Valores que no cambian monótonamente:** Siempre que sea posible, elija una clave de fragmentación con valores que cambien de manera impredecible para fomentar una distribución más uniforme de las inserciones en todo el clúster. El uso de una clave de fragmentación basada en un valor que aumenta o disminuye constantemente puede provocar una distribución deficiente de los datos. Si utiliza una clave de fragmentación que cambia monótonamente, las nuevas inserciones probablemente se dirigirán a un solo *chunk*, a menudo el que contiene el rango más alto de valores. Este *chunk* se convierte en un punto caliente (*hotspot*) para las escrituras, lo que genera una carga desigual en todo el clúster.

Para ayudar a mitigar el riesgo de puntos calientes, MongoDB garantiza que los *chunks* `MinKey` y `MaxKey` no se coloquen en el mismo shard. Sin embargo, eso por sí solo no es suficiente para resolver el sesgo de escritura provocado por valores que cambian monótonamente.

Puede realizar fragmentación por rango tanto en colecciones pobladas como vacías. Si fragmenta una colección poblada, inicialmente solo se crea un *chunk*. Luego, el balanceador migra rangos de ese *chunk*, si es necesario, de acuerdo con el tamaño de rango configurado.

Si fragmenta una colección vacía, el comportamiento de MongoDB dependerá de si especifica zonas y rangos de zonas. Si no especifica zonas para la colección vacía, la operación de fragmentación crea un único *chunk* vacío para cubrir todo el rango de valores de clave de fragmentación, y el balanceador luego migra el *chunk* inicial a través de los shards según corresponda. Si especifica zonas y rangos de zonas, la operación de fragmentación crea *chunks* vacíos para los rangos de zonas definidos, así como cualquier *chunk* adicional para cubrir todo el rango de valores de clave de fragmentación. Luego realiza una distribución inicial de *chunks* basada en los rangos de zonas especificados. Esta creación y distribución inicial de *chunks* permite una configuración más rápida del *zoned sharding*.

#### Fragmentación por hash (*Hashed sharding*)

La fragmentación por hash implica calcular un hash del valor del campo de la clave de fragmentación. MongoDB calcula automáticamente los hashes al resolver consultas que utilizan índices con hash, por lo que su aplicación no necesita calcularlos. A cada *chunk* se le asigna luego un rango basado en los valores hash de la clave de fragmentación. Aunque un rango de claves de fragmentación pueda parecer cercano en términos de sus valores, es poco probable que sus valores hash caigan dentro del mismo *chunk*.

La fragmentación por hash es ideal para claves de fragmentación con campos que cambian monótonamente, como valores `ObjectId` o marcas de tiempo. La fragmentación por hash proporciona una distribución de datos más uniforme en clústeres fragmentados a costa de reducir las operaciones dirigidas (*targeted operations*), que son aquellas que `mongos` enruta a un solo shard utilizando la clave de fragmentación y los metadatos del clúster.

Puede realizar fragmentación por hash tanto en colecciones pobladas como vacías. Si fragmenta una colección poblada con una clave de fragmentación hash, la operación crea un *chunk* inicial para cubrir todos los valores de la clave de fragmentación.

Si fragmenta una colección vacía con una clave de fragmentación hash, el comportamiento de MongoDB dependerá de si especifica zonas y rangos de zonas. A partir de MongoDB 7.2, si no especifica zonas para la colección vacía, la operación de fragmentación crea un *chunk* por shard de forma predeterminada y migra los datos a través del clúster. Anteriormente, la operación creaba dos *chunks* de forma predeterminada. Puede especificar un número diferente de *chunks* iniciales con la opción `numInitialChunks` para iniciar una distribución inicial de *chunks*, lo que permite una configuración más rápida del *sharding*.

Si especifica zonas y rangos de zonas, la operación de fragmentación crea *chunks* vacíos para los rangos de zonas definidos, así como cualquier *chunk* adicional para cubrir todo el rango de valores de clave de fragmentación. Luego realiza una distribución inicial de *chunks* basada en los rangos de zonas especificados.

También puede crear índices compuestos con un único campo con hash. Para crear un índice hash compuesto, especifique `"hashed"` como el valor de cualquier clave de índice individual durante la creación del índice. Un índice hash compuesto calcula el valor hash para un solo campo en el índice compuesto. Este valor, junto con otros campos del índice, se convierte en su clave de fragmentación.

El siguiente ejemplo crea un índice hash compuesto en el campo `name` en orden ascendente, establece `_id` como el campo con hash y fragmenta la colección `planets`:

```javascript
db.planets.createIndex({ "name": 1, "_id": "hashed" })
sh.shardCollection("sample_guides.planets", { "name": 1, "_id": "hashed" })
```

Las características como *zone sharding* son compatibles con el *compound hashed sharding*, donde el campo o campos prefijo no hash definen rangos de zonas, mientras que el campo con hash garantiza una distribución más equitativa de los datos fragmentados. Además, el *compound hashed sharding* puede admitir claves de fragmentación con un prefijo con hash, lo que ayuda a abordar los problemas de distribución de datos vinculados a campos que aumentan monótonamente.

#### Escenarios de sharding: Hash frente a Rango

La siguiente tabla describe varios escenarios de ejemplo de datos fragmentados, recomienda la mejor estrategia de fragmentación y explica el razonamiento por el cual la estrategia elegida es mejor que la alternativa:

| Escenario | Mejor Estrategia de Sharding | Razonamiento de la Estrategia de Sharding |
| :--- | :--- | :--- |
| **Plataforma de redes sociales:** Usted gestiona una base de datos para una plataforma de redes sociales que maneja un gran volumen de publicaciones de usuarios. Cada publicación, almacenada en una colección `app.posts`, está asociada con un `post_id` único, que es un identificador generado secuencialmente.<br>La plataforma experimenta una alta carga de escritura debido a la creación continua de publicaciones.<br>Las consultas comunes a la base de datos incluyen recuperar las publicaciones más recientes, obtener publicaciones de usuarios específicos y realizar análisis sobre los datos de las publicaciones. | **Hashed** | La fragmentación por hash se adaptaría mejor a este escenario porque `post_id` es un valor que aumenta monótonamente. La fragmentación por hash utiliza un hash calculado automáticamente del valor de `post_id` para determinar la ubicación del shard. Esto garantiza que las publicaciones se distribuyan uniformemente entre todos los shards, independientemente de su naturaleza secuencial. Esta distribución uniforme ayuda a evitar que cualquier shard individual se convierta en un cuello de botella debido a una concentración de escrituras o lecturas.<br>Si usáramos la fragmentación por rango, al ser `post_id` un valor que aumenta monótonamente, el chunk con el límite superior de valores clave recibiría la mayoría de las escrituras entrantes. Esto restringiría las operaciones de inserción al único shard que contiene este chunk. |
| **Aplicación de viajes compartidos (*Rideshare*):** Usted gestiona la base de datos de una aplicación global de viajes compartidos que administra un gran volumen de datos de viajes, almacenados en una colección `application.rides`. Cada viaje está asociado con un `ride_id` único, que es una clave de fragmentación compuesta que consta de un valor `city_code` (que representa diferentes ciudades donde opera el servicio) y un valor `_id` único para cada viaje.<br>La aplicación experimenta una combinación de operaciones de lectura y escritura, con consultas frecuentes dirigidas a valores específicos de `city_code` para recuperar todos los viajes o analizar patrones y demanda de viajes en una ciudad determinada. | **Ranged** | Dado que la aplicación consulta con frecuencia viajes con valores de `city_code` específicos, la fragmentación por rango sería mejor ya que se dirige a rangos específicos de la clave de fragmentación.<br>Además, la alta cardinalidad del campo `city_code` garantiza que los datos estén bien distribuidos entre los shards. Cada shard puede manejar viajes para diferentes ciudades, equilibrando la carga y evitando que cualquier shard individual se convierta en un cuello de botella.<br>La fragmentación por hash no sería tan efectiva aquí porque distribuiría los viajes aleatoriamente entre los shards. Como resultado, esto podría provocar un aumento de la latencia y del uso de recursos al buscar valores específicos de `city_code`, ya que sería necesario consultar múltiples shards. |
| **Plataforma de transmisión (*Streaming*):** Usted gestiona la base de datos de una plataforma de transmisión de contenido que administra un gran catálogo de contenido multimedia, incluidas películas, programas de televisión y música. Cada elemento multimedia está asociado con un `media_id` único, que es una clave compuesta que consta de un género y un valor `_id` único para cada pieza de contenido.<br>Las consultas comunes en esta base de datos incluyen recuperar todos los elementos multimedia de un género específico, ya que a los usuarios les gusta filtrar por categorías de medios específicas, y analizar tendencias de popularidad dentro de los géneros. | **Ranged** | La fragmentación por rango sería mejor para este escenario debido a la naturaleza no monótona del campo de género. Esto garantiza que los datos se puedan distribuir entre shards según el género, en lugar de en un patrón secuencial, y puede permitir que los datos relacionados se almacenen en el mismo shard. De esta manera, es probable que las consultas sobre el campo de género solo necesiten acceder a un shard.<br>Similar al escenario anterior de viajes compartidos, la fragmentación por hash distribuiría innecesariamente cada elemento de contenido al azar entre los shards. Esto podría provocar una mayor latencia y uso de recursos al buscar valores de género específicos, ya que la consulta tendría que dirigirse a múltiples shards. |

> **Tabla 2.4:** Estrategias de sharding recomendadas según los escenarios de aplicación

#### Configuraciones de producción

En un clúster de producción, asegúrese de que los datos sean redundantes y de que sus sistemas tengan alta disponibilidad. Considere lo siguiente para un despliegue de clúster fragmentado en producción:

- Despliegue los servidores de configuración como un conjunto de réplicas de tres miembros.
- Despliegue al menos dos shards como conjuntos de réplicas de tres miembros.
- Despliegue uno o más enrutadores `mongos`.

Cuando sea posible, considere desplegar un miembro de cada conjunto de réplicas en un sitio adecuado para la recuperación ante desastres.

Tener múltiples instancias de `mongos` puede respaldar la alta disponibilidad y la escalabilidad. Si hay un proxy o equilibrador de carga entre la aplicación y los enrutadores `mongos`, debe configurarlo para la afinidad del cliente (*client affinity*). La afinidad del cliente permite que cada conexión de un solo cliente llegue a la misma instancia de `mongos`. Para una alta disponibilidad a nivel de shard, haga una de las siguientes acciones:

- Agregue instancias de `mongos` en el mismo hardware donde ya se están ejecutando otras instancias de `mongos`.
- Incruste enrutadores `mongos` a nivel de aplicación.

Si aumenta la cantidad de enrutadores `mongos`, el rendimiento puede degradarse. Su despliegue no debe tener más de 30 enrutadores `mongos`.

El siguiente diagrama muestra una arquitectura común de clúster fragmentado utilizada en producción:

> **Figura 2.8:** Ejemplo de arquitectura de clúster fragmentado en producción

#### Configuración de desarrollo

Para pruebas y desarrollo, puede desplegar un clúster fragmentado con un número mínimo de componentes. Estos clústeres que no son de producción normalmente tendrán los siguientes componentes:

- Una instancia de `mongos`
- Un conjunto de réplicas de un solo shard
- Un servidor de configuración de conjunto de réplicas

El siguiente diagrama muestra una arquitectura de clúster fragmentado utilizada solo para desarrollo:

> **Figura 2.9:** Ejemplo de arquitectura de clúster fragmentado para pruebas y desarrollo

#### Colecciones fragmentadas y no fragmentadas

Una base de datos puede tener una combinación de colecciones fragmentadas y no fragmentadas. Las colecciones fragmentadas se particionan y distribuyen entre los shards del clúster. Las colecciones no fragmentadas pueden ubicarse en cualquier shard, pero no pueden abarcar múltiples shards.

La siguiente figura muestra la distribución de datos entre shards para una colección fragmentada (Colección1) y una colección no fragmentada (Colección2):

> **Figura 2.10:** Una colección fragmentada y una no fragmentada

Para interactuar tanto con colecciones fragmentadas como no fragmentadas en el clúster fragmentado, debe conectarse a un enrutador `mongos`. Los clientes nunca deben conectarse directamente a un solo shard para realizar operaciones de lectura o escritura.

#### Desfragmentar una colección (*Unsharding a collection*)

A partir de MongoDB 8.0, puede desfragmentar colecciones fragmentadas existentes con el comando `unshardCollection` o el método auxiliar `sh.unshardCollection()`. Cuando desfragmenta una colección, MongoDB mueve los datos de la colección a un solo shard y actualiza los metadatos para reflejar el estado no fragmentado. Por defecto, cuando desfragmenta una colección, MongoDB mueve los datos de la colección al shard con la menor cantidad de datos; sin embargo, también puede especificar en qué shard colocar los datos.

Puede desfragmentar una colección en los siguientes casos:

- Puede almacenar la colección en su totalidad en un solo shard.
- La colección requiere aislamiento de recursos y los patrones de acceso se admiten mejor si la colección reside en un solo shard.
- La colección fue fragmentada previamente pero ya no necesita estar fragmentada.

Antes de desfragmentar su colección, asegúrese de cumplir con los siguientes requisitos:

- Si su despliegue tiene el control de acceso habilitado, se le ha otorgado el rol `enableSharding`.
- Su aplicación puede tolerar un período de dos segundos en el que la colección afectada bloquea las escrituras. Durante el período de tiempo en que las escrituras están bloqueadas, su aplicación experimenta un aumento en la latencia.
- Su base de datos cumple con los siguientes requisitos de recursos:
  - El shard al que está moviendo la colección tiene suficiente espacio de almacenamiento para la colección y sus índices. El shard de destino requiere que haya al menos el valor de `(Collection Storage Size + Index Size) * 2` bytes disponibles.
  - Asegúrese de que su capacidad de E/S esté por debajo del 50%.
  - Asegúrese de que su carga de CPU esté por debajo del 80%.

Para desfragmentar una colección, siga estos pasos:

1. Obtenga el nombre de su shard de destino. Opcionalmente, puede especificar a qué shard se moverán sus datos cuando desfragmente una colección. Para generar una lista de nombres de shards, use el comando `listShards`:

```javascript
db.adminCommand({ listShards: 1 })
```

2. Ejecute el comando `unshardCollection`. El siguiente código de muestra desfragmenta una colección llamada `us_users` en la base de datos `clients` hacia `shard1`:

```javascript
db.adminCommand({ unshardCollection: "clients.us_users", toShard: "shard1" })
```

Si omitiera el campo `toShard`, MongoDB movería los datos de la colección al shard con la menor cantidad de datos.

3. Confirme que la colección esté desfragmentada. Para confirmar que la colección `clients.us_users` está desfragmentada, utilice la etapa de canalización de agregación `$shardedDataDistribution` y consulte una coincidencia en el espacio de nombres de la colección no fragmentada:

```javascript
db.aggregate([
  { $shardedDataDistribution: {} },
  { $match: { "ns": "clients.us_users" } }
])
```

Si la colección está desfragmentada, la operación de agregación no devuelve ningún dato.

#### Mover una colección no fragmentada

A partir de MongoDB 8.0, puede mover una colección no fragmentada a un shard diferente utilizando el comando `moveCollection`. En versiones anteriores de MongoDB, las colecciones no fragmentadas solo residían en el shard primario y tenía que usar el comando `movePrimary` para migrar colecciones no fragmentadas. Mover una colección no fragmentada a cualquier shard puede ayudar a optimizar el rendimiento en cargas de trabajo más grandes y complejas, lograr una mejor utilización de recursos y distribuir los datos de manera más uniforme entre los shards.

Podría considerar mover una colección no fragmentada en los siguientes escenarios de ejemplo:

- Tiene tres colecciones no fragmentadas en un solo shard y una de ellas comienza a crecer significativamente más que las demás, lo que provoca una degradación del rendimiento en el shard. Para mejorar el rendimiento y equilibrar la carga en todo el clúster, puede mover las dos colecciones más pequeñas a un shard diferente.
- Su aplicación almacena datos en tres colecciones no fragmentadas independientes ubicadas en América del Norte, Europa y Asia en un solo shard. Para reducir la latencia de una aplicación, puede mover estas colecciones a un shard ubicado en cada región respectiva en el mismo clúster.
- Su aplicación realiza frecuentemente operaciones `$lookup` entre dos colecciones no fragmentadas que residen en diferentes shards. Para mejorar el rendimiento de las consultas, puede mover ambas colecciones al mismo shard.

Antes de mover su colección, asegúrese de cumplir con los siguientes requisitos:

- Su aplicación puede tolerar un período de dos segundos en el que la colección afectada bloquea las escrituras. Durante el período de tiempo en que las escrituras están bloqueadas, su aplicación experimenta un aumento en la latencia.
- Su base de datos cumple con los siguientes requisitos de recursos:
  - El shard al que está moviendo la colección tiene suficiente espacio de almacenamiento para la colección y sus índices. El shard de destino requiere que haya al menos el valor de `(Collection Storage Size + Index Size) * 2` bytes disponibles.
  - Asegúrese de que su capacidad de E/S esté por debajo del 50%.
  - Asegúrese de que su carga de CPU esté por debajo del 80%.

#### Consultar datos fragmentados

Consultar datos desde un clúster fragmentado de MongoDB difiere de consultar datos en un despliegue de un solo servidor o en un conjunto de réplicas. En lugar de conectarse al servidor único o al primario del conjunto de réplicas, se conecta a `mongos`.

Las instancias de `mongos` proporcionan la única interfaz y punto de entrada a su clúster de MongoDB. Las aplicaciones se conectan a `mongos` en lugar de conectarse directamente a los shards. `mongos` ejecuta consultas, recopila resultados y los pasa a la aplicación.

El proceso `mongos` no mantiene ningún estado persistente y normalmente no utiliza muchos recursos del sistema. Actúa como un proxy para las solicitudes. Cuando llega una consulta, `mongos` la examina, decide qué shards deben ejecutarla y establece un cursor en todos los shards de destino.

#### Operaciones find

Si una consulta incluye la clave de fragmentación o un prefijo de la clave de fragmentación, `mongos` realiza una operación dirigida (*targeted operation*), que solo consulta los shards que contienen las claves que está buscando.

Por ejemplo, supongamos que la clave de fragmentación compuesta para la siguiente colección `user` indexa `_id`, `email` y `country`:

```javascript
db.user.find({ _id: 1 })
db.user.find({ _id: 1, "email": "packt@packt.com" })
db.user.find({ _id: 1, "email": "packt@packt.com", "country": "UK" })
```

Estas consultas contienen un prefijo o la clave de fragmentación completa. Una consulta sobre `{email, country}` o `{country}` no podría dirigirse a los shards correctos, lo que daría como resultado una operación de difusión (*broadcast operation*). Una operación de difusión es cualquier operación que no incluye la clave de fragmentación ni un prefijo de la clave de fragmentación, y hace que `mongos` consulte cada shard.

#### Operaciones sort(), limit() y skip()

Si desea ordenar resultados, tiene las siguientes opciones:

- Si utiliza una clave de fragmentación en los criterios de ordenación, `mongos` puede determinar el orden en el que debe consultar un shard o shards. Esto da como resultado una operación eficiente y dirigida.
- Si no está utilizando la clave de fragmentación en los criterios de ordenación, `mongos` ejecutará una operación de difusión. Para ordenar los resultados cuando no utiliza la clave de fragmentación, el shard primario ejecuta una ordenación por fusión distribuida (*distributed merge sort*) localmente antes de pasar los resultados ordenados a `mongos`.

Se aplica un límite en las consultas en cada shard individual y luego nuevamente en el nivel de `mongos`, ya que puede haber resultados de múltiples shards.

Por otro lado, un operador `skip()` no se puede pasar a shards individuales y `mongos` lo aplicará después de recuperar los resultados localmente.

Si combina los métodos de cursor `skip()` y `limit()`, `mongos` optimiza la consulta pasando ambos valores a shards individuales. Esto puede resultar útil en casos como la paginación. Si realiza una consulta sin `sort()` y los resultados provienen de más de un shard, `mongos` utilizará el método *round robin* entre los shards para obtener los resultados.

#### Operaciones de actualización y eliminación

Para las operaciones `updateOne()`, `deleteOne()` y `findAndModify()`, puede usar cualquier campo para hacer coincidir documentos, similar a una colección no fragmentada. No necesita tener la clave de fragmentación ni el valor `_id`; sin embargo, usar la clave de fragmentación será más eficiente ya que permite realizar consultas dirigidas.

Por ejemplo, la siguiente consulta ejecuta una operación `updateOne()` en la colección `cities` sin tener que pasar la clave de fragmentación, que utiliza el campo `city`:

```javascript
db.cities.updateOne({ "population": 293200 }, { $set: { "areaSize": 211 } });
```

La siguiente tabla resume las operaciones en MongoDB 8.0 que puede utilizar para el *sharding*:

| Operación | Notas |
| :--- | :--- |
| `insert()` | Debe incluir la clave de fragmentación |
| `update()` | Puede incluir la clave de fragmentación, pero no es obligatorio |
| Consulta con clave de fragmentación | Operación dirigida (*Targeted operation*) |
| Consulta sin clave de fragmentación | Operación de difusión (*Broadcast operation*) |
| Índice ordenado, consulta sin clave de fragmentación | Fusión de ordenación distribuida (*Distributed sort merge*) |
| `updateOne()`, `deleteOne()`, `replaceOne()`, `findAndModify()` | Puede utilizar cualquier campo para coincidir, pero es más eficiente con una clave de fragmentación |

> **Tabla 2.5:** Comportamiento de las operaciones de sharding en MongoDB 8.0

#### Lecturas con cobertura (*Hedged reads*)

Las instancias de `mongos` pueden realizar lecturas con cobertura (*hedged reads*) que utilizan preferencias de lectura no primarias. Las instancias de `mongos` dirigen las operaciones de lectura a dos miembros del conjunto de réplicas para cada shard consultado y luego devuelven los resultados del primer nodo en responder por shard.

Las operaciones que admiten lecturas con cobertura incluyen las siguientes:

- `collStats`
- `count`
- `dataSize`
- `dbStats`
- `distinct`
- `filemd5`
- `find`
- `listCollections`
- `listIndexes`
- `planCacheListFilters`

#### Métodos de sharding

Para gestionar la distribución de datos, MongoDB proporciona un conjunto de métodos auxiliares. Puede utilizar estos métodos para habilitar la fragmentación, definir cómo se deben distribuir los datos y monitorear el estado del *sharding*.

Algunos métodos auxiliares clave del shell `mongosh` incluyen los siguientes:

- **`sh.shardCollection()`:** Fragmenta una colección utilizando la clave de fragmentación especificada.
- **`sh.status()`:** Devuelve un informe formateado de la configuración de fragmentación e información sobre los *chunks* existentes en un clúster fragmentado. El nivel de detalle se puede ajustar con el parámetro `verbose`, lo que permite obtener una descripción general de alto nivel o un informe detallado. La información proporcionada por este comando incluye la versión de fragmentación, detalles sobre cada shard, el estado de las instancias activas de `mongos`, el estado del balanceador e información sobre bases de datos y colecciones fragmentadas.
- **`sh.reshardCollection()`:** Cambia la clave de fragmentación de una colección y cambia la distribución de sus datos. A partir de MongoDB 8.0, puede volver a fragmentar una colección con la misma clave de fragmentación pasando la opción `forceDistribution`.
- **`sh.getBalancerstate()`:** Devuelve un valor booleano que indica si el balanceador está actualmente activo.
- **`sh.moveCollection()`:** Mueve una sola colección no fragmentada a un shard diferente sin tener que cambiar los shards primarios.
- **`sh.getShardedDataDistribution()`:** Devuelve información sobre la distribución de datos en un clúster fragmentado.
- **`sh.unshardCollection()`:** Desfragmenta una colección fragmentada existente y mueve los datos de la colección a un solo shard.

---

### Sección 6: Resumen

En este capítulo, cubrimos los elementos fundamentales de la arquitectura de MongoDB, definiendo la replicación y el *sharding*, detallando las características clave que puede utilizar en producción y presentando algunas características nuevas en MongoDB 8.0 que pueden ayudar a maximizar el rendimiento de su despliegue.

Con respecto a la replicación, aprendimos sobre el papel fundamental que desempeña para garantizar la disponibilidad de datos y la tolerancia a fallos dentro de MongoDB, incluido el proceso de elección del conjunto de réplicas, la importancia del `oplog` y cómo puede configurar su conjunto de réplicas para mejorar la escalabilidad de lectura. Además, presentamos nuevas características, como la compactación en segundo plano y el TCMalloc actualizado, que ayudan a gestionar el uso de la memoria y, a su vez, aumentan la eficiencia de su despliegue.

Con el *sharding*, este capítulo proporcionó una descripción general completa del *sharding* como estrategia para el escalado horizontal. Aprendimos cómo el *sharding* distribuye datos entre múltiples servidores para gestionar grandes conjuntos de datos y aplicaciones de alto tráfico. Conceptos clave como las claves de fragmentación y los servidores de configuración ayudaron a demostrar cómo el *sharding* permite que los despliegues de MongoDB escalen eficientemente mientras mantienen el rendimiento. Además, cubrimos nuevos comandos de *sharding* que puede aprovechar para reducir la complejidad de su despliegue.

Al final de este capítulo, debería tener una comprensión sólida de cómo los mecanismos de replicación y *sharding* de MongoDB trabajan juntos para proporcionar una solución de base de datos escalable, confiable y de alto rendimiento.

En el próximo capítulo, aprenderá sobre las Herramientas para desarrolladores de MongoDB, que pueden ayudarle a aprovisionar medidas de seguridad y más para su despliegue.
