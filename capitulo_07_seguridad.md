# Parte 7: Seguridad

## Capítulo 7: Seguridad

En el mundo actual impulsado por la tecnología, la seguridad de los datos es primordial. Con el creciente volumen de información confidencial que se almacena y transmite, garantizar su protección es crucial para evitar posibles riesgos y consecuencias.

La seguridad es un proceso continuo que requiere un monitoreo y una mejora constantes. Como administradores, es esencial mantenerse informado sobre las últimas prácticas de seguridad, responder a los incidentes cuando ocurran y abordar de manera proactiva las posibles vulnerabilidades antes de que generen problemas.

MongoDB ofrece una variedad de medidas de seguridad para proteger sus datos, ya sea en reposo, en tránsito o en uso. La seguridad involucra dos componentes clave: la autenticación, que controla quién puede acceder a los datos, y la autorización, que define el nivel de acceso. Además, la protección de datos requiere cifrado, como Transport Layer Security (TLS) y Queryable Encryption de MongoDB.

Este capítulo cubrirá los siguientes temas:

- Seguridad de aplicaciones
- Seguridad de los datos
- Mecanismos de seguridad para despliegues autogestionados

---

### Sección 1: Requisitos técnicos

Para seguir los tutoriales de este capítulo, necesita acceso a lo siguiente:

- MongoDB instalado localmente
- Controlador de MongoDB para Node.js

---

### Sección 2: Seguridad de aplicaciones

La seguridad de las aplicaciones se refiere a las medidas y prácticas utilizadas para proteger las aplicaciones de amenazas y vulnerabilidades de seguridad. Esto incluye salvaguardar los datos, garantizar controles de acceso adecuados y prevenir acciones no autorizadas dentro de la aplicación.

Involucra dos pilares: la **autenticación**, o quién puede acceder a su base de datos y a sus datos, y la **autorización**, que determina el nivel de acceso permitido del usuario en la base de datos.

La seguridad requiere un monitoreo y una reevaluación constantes. Como administrador de MongoDB, debe permanecer atento a los métodos que utiliza para la protección de aplicaciones y datos, las mejores prácticas para estos métodos y sus actualizaciones. La mejor estrategia de seguridad responde a los incidentes y trata de prevenirlos de forma proactiva.

Para proteger la autenticación de sus datos, MongoDB ofrece soporte para múltiples métodos de autenticación, como SCRAM y la certificación X.509.

#### SCRAM

El método predeterminado de autenticación de MongoDB es SCRAM (*Salted Challenge Response Authentication Mechanism*), creado por el Grupo de Trabajo de Ingeniería de Internet (IETF). Cuando un usuario se autentica, MongoDB utiliza SCRAM para verificar las credenciales de usuario proporcionadas con respecto al nombre de usuario, la contraseña y la base de datos de autenticación. SCRAM permite la autenticación de usuarios sin enviar una contraseña en texto no cifrado a través de la red, donde podría ser interceptada por atacantes maliciosos.

La autenticación SCRAM en MongoDB utiliza los siguientes componentes:

- **Contraseña con sal (*Salted password*):** Una sal (*salt*) es una secuencia aleatoria generada para cada usuario. La sal se combina con la contraseña de cada usuario para hacer que las contraseñas sean más seguras al hacerlas más únicas y prevenir ataques de fuerza bruta.
- **Almacenamiento seguro de contraseñas:** Las contraseñas con sal y los datos de autenticación se almacenan en MongoDB utilizando métodos para mejorar la seguridad, como el cifrado de contraseñas.
- **Desafío-respuesta (*Challenge-response*):** SCRAM utiliza un método de autenticación denominado desafío-respuesta. Esencialmente, el cliente primero envía un desafío al servidor (en su caso, a MongoDB), incluyendo su nombre de usuario y un valor aleatorio único, conocido como *nonce*. El servidor responde con su propio *nonce* concatenado con el *nonce* del cliente, así como una sal. El cliente responde con una "prueba" (*proof*), que se calcula utilizando una función hash que toma los datos enviados por el servidor y la contraseña original del cliente. Finalmente, el servidor verifica la prueba del cliente y, si coincide con su propia prueba basada en la información almacenada en su base de datos, el servidor envía un mensaje confirmando que la autenticación fue exitosa.
- **Número de iteraciones:** Puede configurar el número de iteraciones entre el cliente y el servidor permitidas en la autenticación SCRAM. Más iteraciones significan que se requiere más tiempo para calcular la respuesta, lo que hace que los ataques de fuerza bruta sean difíciles de ejecutar.
- **Canal de comunicación seguro:** Para la autenticación más segura, la comunicación entre el cliente y el servidor debe estar cifrada, normalmente mediante el uso de TLS.

La idea principal detrás de SCRAM es que la contraseña del usuario nunca se transmite por la red. Por lo tanto, incluso si un atacante pudiera interceptar esta comunicación, no podría descubrir la contraseña del usuario porque no conoce la función hash utilizada ni la sal.

SCRAM es seguro, sencillo y fácil de entender. Sin embargo, requiere cierto trabajo de gestión. Dado que la autorización se produce localmente, se añaden responsabilidades adicionales al administrador. También debe tener un plan si una credencial llega a verse comprometida.

#### Certificación X.509

Además de las opciones de autenticación tradicionales, como las contraseñas, puede utilizar la autenticación basada en certificados X.509 en MongoDB. X.509 es un estándar de la Unión Internacional de Telecomunicaciones (UIT / ITU) que define el formato de los certificados de clave pública. Los certificados digitales que siguen el estándar X.509 se utilizan para autenticar clientes y servidores a través de conexiones seguras. Estos certificados contienen información como la identidad o la clave pública del usuario o del servidor, así como la firma de una Autoridad de Certificación (*Certificate Authority* o CA).

MongoDB admite la autenticación por certificado X.509 para clientes, así como para la autenticación interna de miembros individuales de un conjunto de réplicas. Para su uso en producción, su despliegue de MongoDB debe utilizar certificados válidos generados y firmados por una CA. Usted o su organización pueden generar y mantener una CA independiente o utilizar certificados generados por proveedores de TLS de terceros.

Para autenticarse en los servidores, los clientes pueden utilizar certificados X.509 en lugar de nombres de usuario y contraseñas. Para la autenticación interna entre miembros de clústeres fragmentados y conjuntos de réplicas, puede utilizar certificados X.509 en lugar de archivos de claves (*keyfiles*).

X.509 es un mecanismo de autenticación potente pero requiere una gestión cuidadosa de los certificados. Las mejores prácticas incluyen la renovación periódica de certificados, la restricción del acceso a las claves de los certificados de cliente y de servidor, y la supervisión de las conexiones a su base de datos.

La certificación X.509 requiere los siguientes aspectos fundamentales:

- **Certificados X.509:** Certificados digitales que siguen el estándar X.509. Estos contienen información como la identidad y la clave pública del usuario o del servidor, así como la firma de una CA. Se utilizan durante la autenticación.
- **CA:** Esta es una entidad de confianza responsable de emitir y firmar certificados digitales. La CA garantiza la autenticidad de los certificados y, por lo tanto, la identidad de los usuarios y servidores involucrados en la comunicación.
- **Certificado de cliente:** Este es el certificado digital utilizado por el cliente (aplicación o usuario) para autenticarse con MongoDB. Este certificado está firmado por la CA y contiene información específica del cliente.
- **Certificado de servidor:** Este es el certificado digital utilizado por el servidor MongoDB para autenticarse con los clientes, el cual también está firmado por la CA.

Estos son los pasos esenciales para implementar la autenticación mediante certificado X.509 en MongoDB:

1. **Elegir una CA:** Seleccionar o crear una CA de confianza es el primer paso. En entornos corporativos, suele haber una CA interna, pero también puede utilizar CA públicas reconocidas, como AWS o Google Cloud.
2. **Generación de certificados:** La CA es responsable de generar los certificados de servidor y de cliente. Para ello, debe crear un par de claves (pública y privada) para cada entidad, firmar las claves con la CA y almacenarlas correctamente.
3. **Distribución de certificados:** Todas las máquinas que accederán a MongoDB deben tener un certificado de cliente. Además, asegúrese de configurar el certificado del servidor en MongoDB.

Hasta ahora, hemos analizado dos métodos diferentes para la autenticación de usuarios, o quién puede acceder a su base de datos. La siguiente sección describe cómo puede controlar la autenticación y la autorización, o a qué datos puede acceder un usuario, mediante el control de acceso basado en roles (RBAC).

#### Autorización y control de acceso basado en roles (RBAC)

Mientras que la autenticación se trata de controlar quién puede acceder a su base de datos, la autorización se trata de definir cómo un usuario puede interactuar con su base de datos y sus datos. MongoDB controla la autorización de los usuarios mediante RBAC (*Role-Based Access Control*), que se utiliza para definir permisos para los usuarios de forma eficiente.

En MongoDB, puede gestionar eficazmente los permisos de los usuarios con RBAC. Un rol en MongoDB es una designación asignada a un usuario que define ciertos permisos. Los roles permiten una granularidad de acceso a nivel de base de datos; en otras palabras, los roles se asocian a bases de datos y, al conceder a un usuario un rol para una base de datos específica, define sus permisos sobre cómo interactúa con esa base de datos.

Por ejemplo, un usuario con un rol de acceso de lectura en una determinada base de datos puede leer datos en esa base de datos, pero no puede modificarlos ni eliminarlos. Este rol no afecta la forma en que el usuario puede interactuar con otras bases de datos. Hay algunas excepciones a esta definición, como cuando se asignan roles en la base de datos `admin`.

En MongoDB, los roles pueden ser integrados (*built-in*) o definidos por el usuario (*user-defined*). Los roles integrados ya están definidos por MongoDB. Los roles definidos por el usuario pueden ser creados y definidos por el usuario según sea necesario. Asignar roles a los usuarios, en lugar de conceder permisos individuales para cada usuario, simplifica el proceso de gestión de acceso, especialmente en entornos grandes y complejos.

El uso de RBAC ofrece varias ventajas:

- Proporciona un control granular sobre los permisos. Las organizaciones pueden definir una amplia gama de roles para reflejar las diferentes responsabilidades y necesidades de acceso de los usuarios.
- Ayuda a hacer cumplir el principio de mínimo privilegio (*principle of least privilege*), una práctica de seguridad que consiste en conceder a los usuarios únicamente los permisos que necesitan estrictamente para realizar sus tareas.
- Puede ayudar a simplificar el proceso de auditoría al facilitar la verificación de quién tiene acceso a qué.

Para comprender RBAC, es importante entender los tres aspectos del acceso a la base de datos:

- **Recursos (*Resources*):** Las entidades involucradas en la autorización de la base de datos, incluidas las bases de datos, las colecciones, los clústeres y los propios usuarios.
- **Acciones de base de datos (*Database actions*):** Las posibles acciones que se pueden realizar dentro o sobre una base de datos, colección o clúster. Estas incluyen acciones como creación/lectura/actualización/eliminación (CRUD), acciones de replicación, acciones de fragmentación y despliegue de bases de datos.
- **Roles integrados en la base de datos (*Database built-in roles*):** Los roles preexistentes definidos por MongoDB. Los roles están asociados a bases de datos específicas. Cada rol contiene diferentes permisos y algunos roles reemplazan a otros. Por ejemplo, un usuario con solo un rol `read` solo puede leer datos en una base de datos determinada, pero un usuario con un rol `readWrite` puede leer datos y escribir en colecciones en una base de datos determinada. Los roles integrados siguen esta jerarquía:
  - **Nivel 0 – Usuario de base de datos (*Database user*):** Este nivel incluye roles básicos que cada base de datos puede proporcionar. El nivel 0 incluye los siguientes roles:
    - `read`
    - `readWrite`
  - **Nivel 1 – Administrador de base de datos (*Database administrator*):** Este nivel incluye roles que permiten la administración de bases de datos. Estos roles se pueden asociar a cualquier base de datos. El nivel 1 incluye los siguientes roles:
    - `dbAdmin`
    - `dbOwner`
    - `userAdmin`
  - **Nivel 2 – Administrador de clúster (*Cluster administrator*):** Este nivel incluye roles que permiten funciones básicas y de administración en múltiples bases de datos. Estos roles solo se pueden asociar a la base de datos `admin` en un clúster. El nivel 2 incluye los siguientes roles:
    - `dbAdminAnyDatabase`
    - `clusterAdmin`
    - `clusterManager`
    - `directShardOperations`
    - `enableSharding`
    - `hostManager`
    - `userAdminAnyDatabase`
    - `readAnyDatabase`
    - `readWriteAnyDatabase`
    - `clusterMonitor`
    - `backup`
    - `restore`
    - `root`

Los roles de nivel superior pueden heredar permisos de roles de nivel inferior, lo que significa que puede otorgar roles a usuarios específicos, simplificando la gestión de permisos porque no necesita otorgar múltiples permisos diferentes a múltiples usuarios. Por ejemplo, si concede el rol `readAnyDatabase` a un usuario, no necesita concederle adicionalmente el rol `read`, ya que `readAnyDatabase` ya incluye los permisos que contiene `read`.

#### Roles definidos por el usuario

Si los roles integrados de MongoDB no describen completamente los privilegios que desea para un usuario, puede crear roles personalizados definidos por el usuario mediante el método `db.createRole()` de MongoDB en `mongosh`. De forma similar a los roles integrados, los roles creados por el usuario se definen en una base de datos específica y solo son aplicables a esa base de datos, con la excepción de los roles creados en la base de datos `admin`, que pueden aplicarse y heredar de roles definidos en otras bases de datos.

Puede utilizar el método `db.createRole()` en el Shell de MongoDB para crear un nuevo rol. Por ejemplo, supongamos que desea definir un rol de usuario muy reducido en el que una aplicación solo pueda leer metadatos de colección o realizar consultas sobre datos. El rol integrado `read` ofrece más permisos que estos, por lo que necesitamos crear un rol básico:

```javascript
db.createRole( {  
   role: "my-role",  
   privileges: [   
     {   
       resource: {   
         db: "my-db",  
         collection: ""  
       },  
       actions: [ "find" ]  
     }  
   ],  
   roles: [ ]  
} )
```

Al llamar a este método, hemos creado un rol llamado `my-role` que opera en todas las colecciones de la base de datos llamada `my-db`. Este rol solo contiene la acción de privilegio `find`, que le permite realizar consultas, listar colecciones y más tareas de solo lectura.

`db.createRole()` le permite personalizar los roles que crea. Esto es importante para limitar el acceso a los datos según los usuarios. Al igual que con los roles integrados, querrá asegurarse de seguir el principio de mínimo privilegio al crear un rol; es decir, conceder a un usuario o aplicación el rol mínimo posible que necesita para realizar su tarea.

Si es necesario, puede utilizar esto junto con los mecanismos de autenticación externa descritos a lo largo de este capítulo. Sin embargo, para garantizar el máximo nivel de control sobre sus roles, es una mejor práctica crear roles de autorización dentro de su despliegue de MongoDB mientras utiliza un Proveedor de Identidad Externo (*External Identity Provider* o EIP) únicamente para la autenticación.

Otro caso de uso de los roles definidos por el usuario es que puede definir privilegios de acceso restringiéndolos en función de la dirección IP del cliente. Para hacer esto, define los parámetros `clientSource` y `serverAddress` al crear un rol definido por el usuario. El parámetro `clientSource` es la dirección IP del cliente que intenta conectarse al servidor MongoDB, y `serverAddress` es la dirección IP del servidor MongoDB al que se conecta. Si se establece cualquiera de estos parámetros, el usuario solo podrá conectarse al servidor MongoDB si la dirección IP correspondiente se encuentra dentro de las restricciones.

Por ejemplo, si establece `clientSource` en `194.154.1.1`, solo los usuarios con la dirección IP `192.168.1.1` podrán conectarse al servidor MongoDB. Si establece `serverAddress` en `192.154.1.1`, solo los usuarios que se conecten al servidor MongoDB con la dirección IP `192.154.1.1` podrán conectarse. Estos dos parámetros también se pueden utilizar conjuntamente para restringir aún más el acceso: por ejemplo, estableciendo tanto `clientSource` como `serverAddress` para garantizar que solo los usuarios con la dirección IP `192.168.1.1` puedan conectarse a un servidor MongoDB que utilice la dirección IP `192.154.1.1`.

Ahora que ha aprendido a utilizar los mecanismos de seguridad para configurar quién puede acceder a su base de datos y a qué puede acceder, continúe leyendo para aprender cómo proteger sus datos en todos los puntos de su ciclo de vida mediante métodos de cifrado y canalizaciones de agregación.

---

### Sección 3: Seguridad de los datos

Con una creciente necesidad de proteger la información confidencial y con normativas de privacidad de datos cada vez más estrictas, las organizaciones están poniendo mayor énfasis en la seguridad de los datos. Normativas como el Reglamento General de Protección de Datos de la Unión Europea (GDPR), que tiene como objetivo proteger la privacidad de los ciudadanos europeos; la Ley de Portabilidad y Responsabilidad del Seguro Médico (HIPAA) en los Estados Unidos, que regula la seguridad de los datos en el sector de la salud; y el Estándar de Seguridad de Datos para la Industria de Tarjetas de Pago (PCI DSS), un marco de cumplimiento para los datos y la seguridad de los titulares de tarjetas de crédito, dictan que las organizaciones prioricen la seguridad y protección de los datos personales.

Los datos deben protegerse en todos los puntos de su ciclo de vida: en reposo en el disco físico, en tránsito entre el cliente y el servidor, y en uso dentro de un cliente. MongoDB proporciona funciones de seguridad como gestión de claves, soporte de TLS, Queryable Encryption y Client-Side Field Level Encryption para garantizar que sus datos estén protegidos en todo momento. Si utiliza un despliegue autogestionado, también puede utilizar algunos de estos métodos junto con los descritos en la siguiente sección, que cubre la seguridad para despliegues autogestionados de MongoDB.

#### Cifrado en reposo con gestión de claves

El cifrado en reposo protege los datos almacenados en un disco físico. Los datos se cifran automáticamente a nivel de archivo, lo que garantiza que incluso si alguien obtiene acceso físico al disco, sus datos permanezcan seguros.

MongoDB Enterprise Advanced ofrece soporte para cifrado en reposo. MongoDB utiliza el algoritmo de cifrado AES-256 para cifrar datos. AES-256 es un algoritmo de cifrado robusto y se utiliza en una amplia variedad de estándares de cumplimiento. Este algoritmo utiliza una clave simétrica, lo que significa que utiliza la misma clave para cifrar y descifrar datos.

El cifrado en reposo utiliza múltiples niveles de claves para proporcionar una combinación de seguridad y rendimiento. Implica una clave maestra (*master key*), que es una clave de cifrado altamente secreta y protegida que se utiliza para cifrar otras claves, conocidas como claves de datos (*data keys*). Las claves de datos, o claves de base de datos, se utilizan para cifrar los datos dentro de una base de datos específica, o incluso conjuntos específicos de datos dentro de una base de datos.

El cifrado en reposo generalmente implica los siguientes pasos:

1. **Generar la clave maestra** utilizando un Generador de Números Pseudoaleatorios Criptográficamente Seguro (*Cryptographically Secure Random Number Generator* o CSRNG). Esta clave debe almacenarse de forma segura, preferiblemente mediante un Servicio de Gestión de Claves (*Key Management Service* o KMS) externo. Si la clave maestra se ve comprometida, todas las claves de base de datos/datos también se ven comprometidas. Normalmente, las claves son de 256 bits.
2. **Generar claves para cada base de datos**, o (opcionalmente) para cada conjunto de datos, también con un CSRNG. Estas claves se utilizan directamente para cifrar datos, por lo que deben ser legibles por la base de datos.
3. **Cifrar datos con las claves de datos.** Cuando los datos se escriben en la base de datos, el sistema utiliza la clave correspondiente a la base de datos (o conjunto de datos) para cifrar los datos antes de escribirlos en el disco. De manera similar, cuando se leen los datos, el proceso se invierte: los datos cifrados se leen del disco y luego se descifran utilizando la clave de la base de datos.
4. **Cifrar las claves de datos con la clave maestra.** Para aumentar la seguridad, las claves de datos se cifran con la clave maestra. Incluso si las claves de datos se ven comprometidas, son inútiles sin la clave maestra. Las claves de datos deben almacenarse en una ubicación segura, pero accesible para la base de datos.

La ventaja de este sistema multinivel es que la clave maestra más crítica se puede almacenar de forma segura y rara vez se accede a ella. Las claves de base de datos, a las que se accede con más frecuencia, son menos críticas en términos de exposición porque siempre se cifran cuando se almacenan.

Se deben tener en cuenta las siguientes mejores prácticas al utilizar el cifrado en reposo:

- **Almacenamiento seguro:** Debe utilizar un sistema de archivos o dispositivo de almacenamiento seguro que admita el cifrado de datos.
- **Rendimiento:** El cifrado puede afectar el rendimiento de operaciones intensivas de lectura o escritura. Asegúrese de tener en cuenta el impacto en el rendimiento al elegir los algoritmos de cifrado.
- **Gestión de claves:** Asegúrese de almacenar de forma segura sus claves de cifrado.
- **Copia de seguridad y recuperación:** Al crear copias de seguridad de datos cifrados, es importante asegurarse de que las copias de seguridad también estén cifradas.
- **Compatibilidad de versiones:** Verifique la compatibilidad de la versión de MongoDB con el cifrado en reposo, ya que esta característica puede variar entre versiones.
- **Recuperación de claves perdidas:** Asegúrese de contar con mecanismos en caso de que pierda una clave de cifrado. Recuperar datos sin una clave de cifrado puede resultar imposible.

La gestión de claves es importante para proteger los datos en reposo mediante cifrado. El concepto de gestión de claves también es importante para proteger los datos cuando están en uso, particularmente en CSFLE y Queryable Encryption, sobre los cuales leerá más adelante en esta sección.

#### Cifrado en tránsito con TLS

El cifrado en tránsito protege los datos durante la transmisión entre clientes y servidores de MongoDB. Esto se logra mediante el protocolo TLS, que cifra los datos antes de enviarlos a través de la red.

El cifrado de transporte protege MongoDB cifrando todo el tráfico de red, lo que significa que un paquete transmitido mediante TLS/SSL solo puede ser leído por el cliente final. El cifrado de red debe utilizarse siempre que sea posible. TLS proporciona los siguientes beneficios de seguridad:

- **Autenticación:** Los servidores y clientes se autentican entre sí mediante certificados.
- **Confidencialidad:** Garantiza que los datos en tránsito no puedan ser leídos ni decodificados por actores maliciosos que puedan estar monitoreando la red.
- **Integridad:** Garantiza que los datos permanezcan intactos durante la transmisión a través de la red.

En MongoDB Atlas, el cifrado de red está habilitado de forma predeterminada y no se puede deshabilitar. Para clústeres locales (*on-premises*), puede optar por implementar TLS configurando el parámetro `net.tls.mode` en el archivo de configuración con uno de los siguientes valores:

- `requireTLS`: El cliente debe usar TLS.
- `preferTLS`: El cliente puede usar TLS y la replicación se utilizará siempre que sea posible.
- `allowTLS`: El cliente puede usar TLS.
- `disable`: El servidor no admite TLS.

Para utilizar TLS/SSL con MongoDB, debe tener certificados TLS/SSL como archivos PEM (*Privacy-Enhanced Mail*), que son contenedores de certificados concatenados. Estos archivos pueden ser emitidos por una CA o pueden ser autofirmados. Usted o su organización pueden generar y mantener una CA independiente, o utilizar certificados generados por proveedores de TLS externos como AWS o Google Cloud.

Estas son algunas consideraciones para habilitar el cifrado en tránsito para MongoDB:

- **Requisitos del sistema:** Debe configurar un certificado TLS/SSL de una CA de confianza.
- **Consumo de recursos:** El cifrado en tránsito puede tener un impacto en el rendimiento de operaciones grandes de lectura o escritura. Tenga en cuenta el rendimiento para asegurarse de que su sistema pueda manejar la sobrecarga adicional.
- **Compatibilidad de controladores:** Asegúrese de que los controladores de MongoDB utilizados en su aplicación admitan el cifrado en tránsito. Algunos controladores pueden requerir una configuración específica para utilizarlo.
- **Latencia:** El cifrado introduce cierta latencia debido al cifrado y descifrado de datos. Esto puede ser insignificante en la mayoría de los casos, pero es algo a considerar en aplicaciones sensibles a la latencia.
- **Configuración adecuada:** Debe configurar MongoDB para usar conexiones TLS/SSL y asegurarse de que todas las conexiones sean seguras.
- **Seguridad de archivos:** Mantenga el archivo `.pem` (certificado) en un directorio seguro y tenga en cuenta los permisos del directorio que contiene el archivo. Asegúrese de que el directorio solo sea accesible para el usuario de MongoDB y que otros usuarios no tengan permisos de lectura, escritura o ejecución.

#### Cifrado a nivel de campo del lado del cliente (Client-Side Field Level Encryption o CSFLE)

El cifrado en uso (*in-use encryption*) protege los datos que se transmiten, almacenan y procesan, y admite consultas sobre esos datos cifrados. MongoDB proporciona CSFLE y Queryable Encryption para facilitar el cifrado en uso, que funciona junto con el cifrado en tránsito y en reposo para proteger los datos a lo largo de todo su ciclo de vida. Ambos métodos utilizan el método de gestión de claves de cifrado descrito anteriormente, que implica claves de datos para cifrar los datos y una clave maestra para cifrar las claves de datos. No puede utilizar Queryable Encryption en una colección en la que esté utilizando CSFLE, y viceversa.

CSFLE permite a las aplicaciones cifrar campos de documentos específicos en una colección antes de enviar esos datos a MongoDB. Con esta técnica, los procesos de cifrado y descifrado tienen lugar exclusivamente en el lado del cliente, lo que garantiza que el servidor de la base de datos nunca tenga acceso a datos no cifrados. Los datos confidenciales se almacenan y procesan de forma segura mediante la gestión de claves, lo que garantiza que, aunque un atacante pudiera obtener datos cifrados de campos confidenciales, no tendría las claves ni el contexto necesario para descifrarlos.

CSFLE es una excelente opción para organizaciones que desean cifrar un campo, pero no todos los campos de su base de datos. Al proteger los datos a nivel de campo o documento, CSFLE se puede utilizar en situaciones en las que ciertos datos, como la Información de Identificación Personal (*Personally Identifiable Information* o PII), requieren protección adicional debido a regulaciones o requisitos de privacidad.

MongoDB ofrece flexibilidad a la hora de adoptar CSFLE. Para realizar CSFLE, MongoDB puede utilizar dos tipos de algoritmos de cifrado:

1. **Algoritmo de cifrado determinista (*Deterministic encryption*):** Garantiza que un valor de entrada específico siempre dará como resultado el mismo valor cifrado cada vez que se ejecute el algoritmo. Sin embargo, si los datos no cifrados tienen una cardinalidad baja, o un número pequeño de valores distintos, el cifrado determinista puede facilitar la realización de análisis de frecuencia sobre los datos cifrados, esencialmente para aplicarles ingeniería inversa hasta su forma no cifrada. Además, el cifrado determinista no admite el cifrado de objetos y matrices completos. Si necesita realizar consultas sobre sus datos cifrados, es posible hacerlo cuando utiliza el algoritmo determinista. Sin embargo, es mejor utilizar Queryable Encryption si necesita realizar consultas.
2. **Algoritmo de cifrado aleatorio (*Randomized encryption*):** Garantiza que un valor de entrada determinado siempre se cifre en un valor de salida diferente cada vez que se ejecute el algoritmo. Si bien el cifrado aleatorio proporciona garantías más sólidas de confidencialidad de los datos que el cifrado determinista, también impide la compatibilidad con cualquier operación de lectura que deba operar en el campo cifrado para evaluar la consulta. El cifrado aleatorio admite el cifrado de objetos y matrices completos.

MongoDB le permite cifrar automáticamente sus datos antes de escribirlos en la base de datos. Para las operaciones de lectura, el controlador cifra los valores de campo en la consulta antes de emitir la operación de lectura. Para las operaciones de lectura que devuelven campos cifrados, el controlador descifra automáticamente los valores cifrados solo si el controlador se configuró con acceso a las claves de datos y a la clave maestra utilizadas para cifrar esos valores.

MongoDB utiliza la validación de esquemas para hacer cumplir el cifrado de campos específicos en una colección. Sin un esquema del lado del cliente, el cliente descarga el esquema del lado del servidor para la colección para determinar qué campos cifrar. Debido a que CSFLE y Queryable Encryption no proporcionan un mecanismo para verificar la integridad de un esquema, depender de un esquema del lado del servidor significa confiar en que el esquema del servidor no ha sido manipulado. Si un adversario compromete el servidor, puede modificar el esquema para que un campo previamente cifrado ya no esté etiquetado para cifrado. Esto hace que el cliente envíe valores en texto plano para ese campo. Para evitar este problema, utilice la validación de esquemas del lado del cliente.

También tiene la opción de cifrar campos explícitamente cuando utiliza CSFLE. Si bien esto permite un control más detallado sobre la seguridad que el cifrado automático, puede aumentar la complejidad al configurar colecciones o escribir código del lado del cliente. Debe especificar explícitamente cuándo cifrar los datos durante las operaciones de lectura y escritura. Incluso si cifra explícitamente sus campos, tiene la opción de descifrar automática o explícitamente sus campos cifrados.

Si bien MongoDB abstrae gran parte de la complejidad asociada con la gestión de claves, usted debe administrar y proteger adecuadamente las claves de cifrado, especialmente la clave maestra. Cuando configura CSFLE, debe definir qué campos de sus colecciones deben cifrarse y con qué algoritmo. El cliente genera automáticamente claves de cifrado de datos para cada campo cifrado. Cada cliente tiene sus propias claves de cifrado de datos. Cuando un cliente desea acceder a datos cifrados, MongoDB proporciona automáticamente las claves de descifrado adecuadas para descifrar los campos protegidos. Este proceso es transparente para el cliente, lo que significa que el cliente no necesita gestionar claves de descifrado ni realizar operaciones manuales de descifrado. MongoDB se encarga de esto automáticamente; incluso si cifra y descifra explícitamente sus campos, simplemente puede llamar a un método de cifrado o descifrado para realizar la operación.

Si implementa CSFLE, tenga en cuenta las siguientes consideraciones:

- **Rendimiento:** El cifrado y descifrado del lado del cliente pueden afectar el rendimiento, especialmente en operaciones intensivas de lectura o escritura. Tenga esto en cuenta al elegir su algoritmo de cifrado.
- **Consulta de datos cifrados:** Consultar campos cifrados puede agregar cierta sobrecarga, especialmente si el campo está cifrado mediante cifrado aleatorio. En este caso, es posible que necesite consultar un campo cifrado aleatoriamente mediante un proxy. Consulte la documentación de MongoDB para obtener más información.
- **Clave maestra:** Mantenga copias de seguridad seguras de su clave maestra. Perder o comprometer la clave maestra significa perder el acceso a los datos.
- **Compatibilidad de controladores:** No todos los controladores de lenguajes de programación admiten CSFLE, o al menos no en la misma medida.
- **Regulaciones de datos:** Puede utilizar CSFLE para cumplir con normativas de protección de datos como GDPR, ya que MongoDB nunca ve datos no cifrados. Si necesita cumplir con una regulación determinada, asegúrese de tener en cuenta todos los requisitos de la misma.
- **KMS externo:** Puede utilizar un KMS externo como AWS KMS o Google Cloud KMS para gestionar sus claves de cifrado.

Para obtener más información sobre la implementación de CSFLE, consulte la documentación de MongoDB: [https://www.mongodb.com/docs/manual/core/csfle/](https://www.mongodb.com/docs/manual/core/csfle/).

#### Cifrado consultable (*Queryable Encryption*)

Queryable Encryption permite que una aplicación cliente cifre datos y los envíe a través de la red manteniendo la capacidad de consulta. Al igual que en CSFLE, los datos confidenciales son cifrados y descifrados de forma transparente por el cliente y solo se comunican hacia y desde el servidor en formato cifrado. A diferencia de CSFLE, sin embargo, Queryable Encryption utiliza esquemas de cifrado con capacidad de búsqueda basados en cifrado estructurado, lo que significa que los datos se cifran de tal manera que se pueden ejecutar consultas sobre los datos cifrados. Utiliza una clave maestra y una clave de datos como se describió anteriormente en la sección de Gestión de claves.

Queryable Encryption funciona de la siguiente manera:

1. Cuando se envía una consulta, el controlador de MongoDB la analiza. Si el controlador reconoce que la consulta se realiza sobre un campo cifrado, solicita las claves de cifrado a un proveedor de claves aprovisionado por el cliente, como AWS KMS. Luego envía la consulta al servidor de MongoDB con los campos cifrados.
2. Queryable Encryption implementa un esquema de búsqueda rápido que permite al servidor procesar consultas sobre datos totalmente cifrados, sin saber nada sobre los datos. Los datos y la consulta en sí permanecen cifrados en todo momento en el servidor.
3. El servidor MongoDB devuelve los resultados cifrados de la consulta al controlador.
4. Los resultados de la consulta se descifran con las claves que posee el controlador y se devuelven al cliente, mostrándose como texto no cifrado.

Para habilitar Queryable Encryption en los datos de su aplicación, debe crear un esquema de cifrado. Al crear este esquema, cuando elija los campos que se cifrarán, podrá determinar cómo desea consultar estos campos. Puede utilizar consultas de igualdad, que prueban la igualdad, o consultas de rango, que prueban valores dentro de un rango específico.

El siguiente tutorial ofrece una descripción general de cómo utilizar Queryable Encryption con MongoDB. Este tutorial utiliza cifrado automático. Para obtener más información, incluidos detalles y las instrucciones más actualizadas, consulte el Inicio rápido de Queryable Encryption en la documentación de MongoDB: [https://www.mongodb.com/docs/manual/core/queryable-encryption/quick-start/](https://www.mongodb.com/docs/manual/core/queryable-encryption/quick-start/). Este tutorial utiliza el controlador de MongoDB para Node.js. Para ver el código utilizado en este tutorial, puede consultar [https://github.com/mongodb/docs/tree/master/source/includes/qe-tutorials/node](https://github.com/mongodb/docs/tree/master/source/includes/qe-tutorials/node), que contiene todo el código para la aplicación de muestra de inicio rápido de Queryable Encryption.

Para comenzar con Queryable Encryption utilizando MongoDB y el controlador de Node.js, siga estos pasos esenciales:

1. **Utilice el controlador correcto:** Queryable Encryption no es compatible con todas las versiones de todos los controladores de MongoDB. Para obtener información actualizada sobre qué controladores y qué versiones admiten Queryable Encryption, consulte la documentación de MongoDB.
2. **Descargue la biblioteca libmongocrypt:** Esta biblioteca realiza el cifrado y descifrado y gestiona la comunicación entre el controlador y el KMS. Dependiendo del controlador que utilice, es posible que deba descargar `libmongocrypt` si es una dependencia para ese controlador.
3. **Descargue la biblioteca compartida de cifrado automático (*Automatic Shared Encryption Library*):** La biblioteca compartida de cifrado automático es una biblioteca dinámica (lo que significa que se accede a su funcionalidad en tiempo de ejecución) que permite a su aplicación realizar el cifrado automático. La biblioteca lee el esquema de cifrado, que creará más adelante, para determinar qué campos cifrar o descifrar. No realiza ningún cifrado o descifrado real.
4. **Genere sus claves maestras y de datos:** El siguiente fragmento de código genera una clave maestra y la almacena en un archivo llamado `./customer-master-key.txt`. Si utiliza Queryable Encryption en producción, querrá asegurarse de almacenar de forma segura sus claves maestras y de datos en un KMS.

El siguiente fragmento de código genera una clave maestra aleatoria de 96 bytes:

```javascript
if (!existsSync("./customer-master-key.txt")) {
  try {
    writeFileSync("customer-master-key.txt", randomBytes(96));
  } catch (err) {
    throw new Error(
      `Unable to write Customer Master Key to file due to the following error: ${err}`
    );
  }
}
```

5. **Cree su esquema de cifrado:** Para cifrar un campo, agréguelo a su esquema de cifrado. Si desea realizar consultas en este campo, debe especificar si desea realizar consultas de igualdad o consultas de rango en el campo.

Por ejemplo, supongamos que tenemos un conjunto de registros médicos que queremos cifrar. El siguiente fragmento de código define tres campos que se cifrarán utilizando el esquema de cifrado. Un campo, `name`, se puede consultar por igualdad, el campo `bill` se puede consultar por desigualdad (consulta de rango) y el último no se puede consultar porque no hemos definido un tipo de consulta para el campo:

```javascript
const encryptedFieldsMap = {
  encryptedFields: {
    fields: [
      {
        path: "name",
        bsonType: "string",
        queries: { queryType: "equality" },  
      },  
      {  
        path: "bill",  
        bsonType: "int",  
        queries: { queryType: "range" },  
      },  
      {  
        path: "procedure",  
        bsonType: "object",  
      },  
    ],  
  },  
};
```

6. **Cree la colección cifrada:** Debe proporcionar `encryptedFieldsMap` como una opción al crear esta colección, así como la información para la clave maestra. Esto creará su base de datos y colección cifradas, y generará automáticamente claves de datos para su colección que se pueden usar para descifrar los datos:

```javascript
await clientEncryption.createEncryptedCollection(
  encryptedDatabase,
  encryptedCollectionName,
  {
    provider: kmsProviderName,
    createCollectionOptions: encryptedFieldsMap,
    masterKey: customerMasterKeyCredentials,
  }
);
```

7. **Inserte sus documentos:** Puede observar sus datos y ver cómo se cifran sus campos. Al igual que con los datos no cifrados, puede utilizar el método `insertOne()` o `insertMany()`. El controlador se encargará de cifrar los datos y subirlos de forma segura al servidor.

Por ejemplo, podemos insertar los siguientes tres documentos:

```javascript
const patient1 = {
   name: "Jon Doe",
   bill: 1500,
   procedure: {
     code: 174839,
     recoveryTime: 30,
     name: "appendectomy"
   },
};
const patient2 = {
   name: "Jane Doe",
   bill: 1800,
   procedure: {
     code: 149174,
     recoveryTime: 60,
     name: "ankle surgery"
  },
};
const patient3 = {
   name: "Jake S",
   bill: 700,
   procedure: {
     code: 198347,
     recoveryTime: 90,
     name: "LASIK"
   },
};
await myColl.insertMany([ patient1, patient2, patient3 ]);
```

Después de insertar los documentos anteriores, puede verlos en su base de datos y observar que los campos `name`, `bill` y `procedure` están todos cifrados.

8. **Realice consultas de rango sobre sus datos:** Supongamos que queremos encontrar a todos los pacientes cuyas facturas médicas superaron los $1,000, lo que significa que el valor del campo `bill` en su registro es mayor que 1000 y menor que 2000. Podemos hacerlo realizando la siguiente consulta de rango:

```javascript
const findResult = await encryptedCollection.find({
  "bill": { $gt: 1000, $lt: 2000 },
});
for await (const doc of findResult) {
  console.log(doc);
}
```

Los resultados de la consulta contendrán los siguientes dos documentos, que han sido descifrados por el servidor:

```json
{
   name: "Jon Doe",
   bill: 1500,
   procedure: {
     code: 174839,
     recoveryTime: 30,
     name: "appendectomy"
   },  
},
{
   name: "Jane Doe",
   bill: 1800,
   procedure: {
     code: 149174,
     recoveryTime: 60,
     name: "ankle surgery"
   },
}
```

Ahora, ha creado una aplicación que utiliza Queryable Encryption. Puede realizar consultas de igualdad y de rango de valores sobre los datos de su base de datos mientras se asegura de que los datos permanezcan cifrados hasta que lleguen a su aplicación. Puede ampliar esto agregando más datos y probando diferentes algoritmos de cifrado para ver cómo mejoran o afectan el rendimiento de su aplicación.

Ahora que ha aprendido sobre CSFLE y Queryable Encryption, consulte la siguiente sección para saber cómo se comparan estos métodos cuando se trata de proteger los datos de su aplicación.

#### Comparación entre CSFLE y Queryable Encryption

CSFLE y Queryable Encryption son mecanismos sólidos de seguridad de datos proporcionados por MongoDB. Ambos métodos cifran los datos en el lado del cliente antes de enviarlos a través de la red. Se pueden utilizar en la misma aplicación; sin embargo, no se pueden utilizar en la misma colección. Por lo tanto, debe tomar decisiones sobre qué mecanismo de cifrado utilizar según las necesidades de su aplicación.

**Queryable Encryption se utiliza mejor en los siguientes escenarios:**
- Está desarrollando una nueva aplicación.
- Espera que los usuarios ejecuten consultas, incluidas consultas de rango, prefijo, sufijo o subcadena, sobre datos cifrados.
- Su aplicación puede utilizar una única clave para un campo determinado.

**CSFLE se puede utilizar en los siguientes escenarios:**
- Su aplicación ya utiliza CSFLE.
- Necesita utilizar claves diferentes para el mismo campo. Esto se puede encontrar al separar inquilinos (*tenants*) o al utilizar claves específicas de usuario.
- Necesita ser flexible con el esquema de sus datos y potencialmente agregar más campos cifrados. Agregar campos cifrados para Queryable Encryption requiere reconstruir colecciones de metadatos e índices.

#### Operador de canalización $redact

Un método más de seguridad de datos que no implica cifrado es el operador de canalización `$redact`. Cuando construye una canalización de agregación, puede utilizar la etapa de canalización de agregación `$redact` para restringir el contenido de la información almacenada en los documentos mismos.

> **Figura 7.1:** Flujo de trabajo del operador de canalización $redact

`$redact` restringe el acceso al contenido de un documento según los criterios de acceso almacenados en el propio documento. Esta es una forma útil y fácil de implementar para restringir el acceso de los usuarios a los datos; sin embargo, debido a que los datos todavía se almacenan en MongoDB y no están cifrados, existe la posibilidad de que un actor malicioso comprometa los datos. Al utilizar `$redact`, asegúrese de emplear también otros mecanismos de seguridad para proteger sus datos.

Por ejemplo, podemos controlar el acceso de los usuarios a los datos en función de un campo `tags`. Imagine una colección que contiene datos de la siguiente forma:

```json
{
   _id: 1,
   title: "123 Department Report",
   tags: [ [ "bar" ], [ "foo" ] ],
   year: 2014,
   subsections: [
       {
           subtitle: "Section 1: Overview",
           tags: [ [ "bar", "baz" ], [ "foo" ] ],
           content:  "Section 1: This is the content of section 1."
       },
       {
           subtitle: "Section 2: Analysis",
           tags: [ [ "bar" ] ],
           content: "Section 2: This is the content of section 2."
       },
   ]
}
```

Queremos que el campo `tags` determine la capacidad de un usuario para acceder a los datos. Las etiquetas deben coincidir con la matriz más interna. Para el ejemplo anterior, si un usuario tiene la etiqueta de acceso `bar`, puede acceder al documento de alto nivel y al documento de nivel incrustado con el campo de subtítulo `Section 2: Analysis`. No puede acceder a la Subsección 1 porque no tiene la etiqueta `foo` ni las etiquetas `bar` y `baz`. Si tiene las etiquetas de acceso `bar` y `foo`, puede acceder al documento completo.

Ahora, imagine a un usuario con la etiqueta de acceso `bar`. Para ejecutar una consulta de canalización para este usuario, utilizamos la función `db.collection.aggregate()` con la etapa de canalización `$redact`:

```javascript
var userAccess = [ "bar" ];
db.collection.aggregate(
   [
     { $redact: {
         $cond: {
           if: { $gt: [ { $size: { $setIntersection: [ "$tags", userAccess ] } }, 0 ] },  
           then: "$$DESCEND",  
           else: "$$PRUNE"  
         }  
       }  
     }  
   ]  
);
```

La operación de agregación devuelve el siguiente documento "redactado" para el usuario:

```json
{
   _id: 1,
   title: "123 Department Report",
   tags: [ [ "bar" ], [ "foo" ] ],
   year: 2014,
   subsections: [
       {
           subtitle: "Section 2: Analysis",
           tags: [ [ "bar" ] ],
           content: "Section 2: This is the content of section 2."
       },
   ]
}
```

De esta manera, puede controlar la información de un documento que el usuario puede ver. Esto es diferente del cifrado a nivel de campo, que cifra campos específicos y requiere múltiples claves para descifrar; más bien, el usuario nunca verá los datos a los que no tiene acceso.

Al utilizar `$redact` junto con CSFLE o Queryable Encryption, puede garantizar la máxima seguridad y protección de sus datos. Continúe leyendo para conocer los métodos que puede utilizar en una implementación autogestionada.

---

### Sección 4: Mecanismos de seguridad para despliegues autogestionados

La seguridad en las implementaciones de MongoDB es esencial para garantizar la integridad y confidencialidad de sus datos. Al proteger su aplicación, datos e implementación, puede salvaguardar todo su sistema contra posibles amenazas.

Esta sección explora varios métodos de seguridad disponibles para proteger su despliegue autogestionado de MongoDB. Cubriremos la obsolescencia del soporte del Protocolo ligero de acceso a directorios (*Lightweight Directory Access Protocol* o LDAP) en MongoDB 8.0 y cómo hacer la transición a Microsoft OpenID Connect (OIDC) para una autenticación externa más moderna y eficiente.

Además, analizaremos la autenticación Kerberos, la creación de roles de usuario personalizados y el fortalecimiento de su implementación con el blindaje de la red (*network hardening*). Estos métodos se pueden utilizar juntos para mejorar la seguridad de su base de datos desde múltiples ángulos.

#### Obsolescencia de LDAP (*LDAP deprecation*)

MongoDB Enterprise Advanced proporcionaba soporte tanto para la autenticación como para la autorización mediante LDAP. Como se mencionó anteriormente en este capítulo, la autenticación se refiere a quién puede acceder a su base de datos, mientras que la autorización se refiere a qué puede acceder un usuario. A partir de MongoDB 8.0, el soporte para la autenticación y autorización LDAP está obsoleto (*deprecated*). Todavía está disponible para su uso, pero se eliminará en una versión futura. Los usuarios deberían considerar el uso de la autenticación OIDC de Microsoft en su lugar.

Para obtener más información sobre LDAP, consulte la documentación de MongoDB: [https://www.mongodb.com/docs/manual/core/security-ldap-external/](https://www.mongodb.com/docs/manual/core/security-ldap-external/).

#### ¿Por qué la obsolescencia?

En MongoDB 8.0, el soporte para la autenticación y autorización LDAP quedó obsoleto tanto en MongoDB Enterprise Advanced como en MongoDB Atlas. Los desarrolladores aún pueden usarlo; sin embargo, el soporte para LDAP no se actualizará en el futuro y se eliminará en una versión posterior, por lo que se recomienda comenzar a migrar sus aplicaciones para usar autenticación y autorización OIDC. Aunque LDAP se utiliza ampliamente, no está diseñado inherentemente para entornos de nube, lo que genera la posibilidad de que las credenciales queden expuestas al utilizar la autorización LDAP con MongoDB Atlas. Además, acceder a un servidor LDAP a través de Internet genera una configuración de red compleja que puede provocar problemas de conectividad y aumentar el riesgo de problemas de producción.

MongoDB 7.0 introdujo soporte para la Federación de Identidad de la Fuerza Laboral/Lugar de Trabajo (*Workforce/Workplace Identity Federation*) mediante OIDC, que ofrece una alternativa moderna y optimizada a LDAP. En esta sección, puede aprender sobre OIDC y cómo migrar su aplicación para usarlo.

#### Descripción general de OIDC

El soporte para OIDC se introdujo en MongoDB 7.0. De manera similar a LDAP, OIDC verifica a los usuarios en un servidor de autorización, siguiendo las especificaciones de autorización de OAuth 2.0. OIDC permite el acceso de inicio de sesión único (*Single Sign-On* o SSO) a su base de datos de MongoDB utilizando cualquier proveedor de identidad que admita OIDC, como Servicios de federación de Active Directory de Microsoft (ADFS), Microsoft Entra ID, Okta y Ping Identity.

A alto nivel, el proceso de autenticación OIDC sigue este flujo de trabajo:

1. Su aplicación cliente, o usuario, envía credenciales de autenticación a MongoDB.
2. MongoDB envía una solicitud a un proveedor de OpenID que se ha implementado siguiendo OIDC y el protocolo OAuth 2.0, como ADFS, Microsoft Entra ID u Okta.
3. El proveedor de OpenID autentica y autoriza al usuario. Si tiene éxito, el proveedor responde a MongoDB con un token de identidad y, a veces, un token de acceso. Estos tokens identifican al usuario y también pueden incluir información sobre cuándo y cómo se autenticó el usuario.
4. MongoDB envía una solicitud al usuario con el token de acceso.
5. El usuario obtiene acceso a MongoDB utilizando la información proporcionada en el token de acceso.

MongoDB ofrece dos formas de OIDC: **Workforce Identity Federation**, que permite a los usuarios humanos autenticarse mediante un EIP, y **Workload Identity Federation**, que permite a las aplicaciones autenticarse mediante identidades programáticas externas, como cuentas de servicio de Google. Algunas de las ventajas de Workforce y Workload Identity Federation son las siguientes:

- **No se almacenan credenciales en MongoDB:** Con LDAP, las credenciales de usuario se almacenan en MongoDB, lo que significa que pueden ser comprometidas por un actor malicioso.
- **Riesgo reducido entre aplicaciones:** En una conexión LDAP, las credenciales LDAP del usuario se envían a MongoDB dentro de la cadena de conexión, lo que puede exponer las credenciales a través de una red. Sin embargo, con Workforce y Workload Identity Federation, MongoDB nunca recibe un secreto. En su lugar, OIDC y OAuth 2.0 otorgan tokens de acceso para recursos específicos. Si un token se ve comprometido, no se puede utilizar para acceder a otras aplicaciones.
- **Seguridad mejorada:** Identity Federation otorga acceso a través de tokens de acceso, que normalmente solo son válidos durante una hora.
- **Autenticarse sin contraseñas:** Si sus aplicaciones se ejecutan en recursos de nube específicos como GCP o Azure, puede autenticarse sin contraseñas, lo que significa que no tiene que renovar periódicamente las credenciales.

#### Uso de OIDC con MongoDB

MongoDB y OIDC ofrecen dos tipos de autenticación de identidad: Workload Identity Federation, que autentica aplicaciones, y Workforce Identity Federation, que autentica usuarios. Este tutorial se centra en Workload Identity Federation, que utiliza el protocolo OAuth 2.0. Como siempre, para obtener la información más detallada y actualizada, consulte la documentación oficial de MongoDB. El soporte para OIDC solo está disponible cuando se ejecuta una implementación de MongoDB Enterprise en Linux.

Para configurar OIDC con MongoDB para la autenticación de aplicaciones, siga estos pasos:

1. Asegúrese de que su implementación de MongoDB Enterprise ejecute MongoDB 7.0 o posterior.
2. Configure un EIP, como Microsoft Azure o Google Cloud. Este EIP emite tokens de identidad y acceso compatibles con OAuth 2.0. Al configurar su EIP, puede crear grupos de usuarios, si desea definir diferentes permisos para diferentes grupos. Sin embargo, las mejores prácticas dictan que defina los permisos de usuario dentro de MongoDB en lugar de en su EIP. Para obtener instrucciones paso a paso sobre cómo configurar un EIP, consulte la documentación de MongoDB.
3. Configure su servidor para usar OIDC mediante su archivo de configuración. En la opción `setParameter`, establezca el parámetro `authenticationMechanisms` en `MONGODB-OIDC` y establezca el parámetro `oidcIdentityProviders` para especificar la configuración de su EIP. Para Workload Identity Federation, establezca el campo `supportsHumanFlows` en `false`:

```yaml
setParameter:
   authenticationMechanisms: MONGODB-OIDC
   oidcIdentityProviders: [ {
        "issuer": "https://okta-test.okta.com",
        "audience": "example@kernel.mongodb.com",
        "authNamePrefix": "okta-issuer",
        "matchPattern": "@mongodb.com$",
        "JWKSPollSecs": 86400,
        "supportsHumanFlows": false
   } ]
```

Si desea configurar múltiples proveedores de identidad, puede incluir la información de configuración como documentos adicionales en la matriz pasada a `oidcIdentityProviders`. La prioridad de cada EIP está determinada por el orden de la matriz, siendo el primer EIP de la matriz el primero seleccionado.

4. Autorice a los usuarios mediante la creación de roles. En la base de datos `admin`, utilice el comando `db.createRole()` para crear roles que asignen el grupo de EIP (consulte el paso 2) a un rol de MongoDB.

Puede especificar un rol de EIP utilizando el siguiente formato:

```text
<authNamePrefix>/<authorizationClaim>
```

Por ejemplo, si utilizamos Okta como nuestro EIP, podemos crear roles basados en el grupo `Everyone` en el EIP que puedan leer y escribir en cualquier base de datos, ejecutando el siguiente comando:

```javascript
db.createRole( {
   role: "okta/Everyone",
   privileges: [ ],
   roles: [ "readWriteAnyDatabase" ]
} )
```

Así es como puede habilitar OIDC para su implementación de MongoDB para una autenticación basada en tokens rápida, segura y eficiente.

#### Kerberos

MongoDB Enterprise Advanced ofrece el método Kerberos para la autenticación. Con origen en el Proyecto Athena del MIT, Kerberos proporciona una autenticación sólida para aplicaciones cliente-servidor a través de un enfoque basado en criptografía para emitir tickets de acceso. Los tickets son conjuntos de datos cifrados, como la información del cliente. Kerberos se ha convertido en una herramienta esencial para gestionar identidades de forma segura en entornos distribuidos.

La integración de Kerberos en los mecanismos de autenticación de MongoDB garantiza que solo los usuarios autenticados puedan acceder e interactuar con los datos almacenados de MongoDB. Sin embargo, el propio MongoDB debe realizar la autorización. Puede utilizar la autenticación Kerberos con una forma diferente de autorización, como OIDC.

Para utilizar Kerberos para autenticarse contra MongoDB, primero debe comprender y configurar algunos componentes, tanto en Kerberos como en MongoDB:

- **Los componentes de MongoDB son los siguientes:**
  - **Archivo de configuración:** Utilice el archivo de configuración de MongoDB para elegir las opciones de seguridad de Kerberos.
- **Los componentes de Kerberos son los siguientes:**
  - **Centro de distribución de claves (*Key Distribution Center* o KDC):** Este sistema gestiona claves y distribuye tickets. Incluye dos partes:
    - *Servidor de autenticación:* Este servidor autentica a los usuarios y les emite tickets de concesión de tickets (*Ticket-Granting Tickets* o TGT).
    - *Servicio de concesión de tickets:* Una vez que un usuario tiene un TGT, puede solicitar tickets de servicio. Un usuario de MongoDB puede solicitar un ticket para el servicio de MongoDB.
  - **Principal:** Los principales incluyen usuarios, servicios y hosts; cualquier entidad que esté presente durante el proceso de autenticación. Cada principal requiere un nombre único.
  - **Ticket:** Datos cifrados que se utilizan para demostrar la identidad de un cliente ante un servidor. Un ticket contiene datos como información del cliente y una clave de sesión. Debido a que estos datos están cifrados con la clave del servidor, el servidor puede descifrarlos.
  - **Reino (*Realm*):** Un dominio en Kerberos que gestiona las entidades principales.

Puede configurar Kerberos siguiendo los siguientes pasos:

1. Configure el centro de distribución de claves y defina todos los principales para los usuarios y servicios que participarán en la operación.
2. Configure MongoDB para utilizar la autenticación Kerberos especificando la opción en el archivo de configuración y reiniciando su servidor.
3. Cree usuarios en MongoDB si aún no lo ha hecho.
4. Los usuarios obtienen un TGT del servidor de autenticación.
5. Mediante el uso del TGT, el cliente puede solicitar un ticket de servicio del servicio de concesión de tickets para MongoDB.
6. El controlador de MongoDB se autentica con MongoDB utilizando este ticket de servicio.
7. Al utilizar la clave de cifrado del servicio, MongoDB descifra la información del ticket, valida el ticket y establece una sesión autenticada.

La integración de MongoDB con Kerberos proporciona un método de autenticación sólido y seguro para entornos que ya utilizan Kerberos. Si utiliza una red de área local (LAN) o un entorno que no es Linux, puede utilizar Kerberos para autenticarse en su implementación autogestionada a través de OIDC.

Aquí hay una página de documentación para leer más: [https://www.kantega-sso.com/articles/the-difference-between-kerberos-saml-og-openid-connect-oidc](https://www.kantega-sso.com/articles/the-difference-between-kerberos-saml-og-openid-connect-oidc).

#### Fortalecimiento de la red (*Network hardening*)

El fortalecimiento de la red implica diferentes técnicas para minimizar las vulnerabilidades en su implementación. Con una implementación autogestionada de MongoDB, puede utilizar lo siguiente:

- **Cortafuegos (*Firewalls*):** Le permiten filtrar el acceso a su red a un nivel granular. Un administrador puede limitar qué usuarios obtienen acceso a una red, lo que aumenta la protección de su red contra atacantes maliciosos.
- **Red Privada Virtual (*Virtual Private Network* o VPN):** Oculta su dirección IP y reenvía el tráfico a una nueva. De esta manera, los atacantes maliciosos no pueden obtener acceso a su dirección IP.
- **Reenvío de IP (*IP forwarding*):** Permite a los servidores reenviar paquetes a otros sistemas desde el origen hasta el destino. Sin embargo, es susceptible a ataques maliciosos y puede permitir que actores malintencionados obtengan acceso no autorizado a su red o eludan los cortafuegos. Para evitar riesgos derivados del reenvío de IP, puede desactivarlo en la máquina que aloja su implementación.

Generalmente, puede utilizar cortafuegos junto con una VPN, o cortafuegos junto con la desactivación del reenvío de IP. También puede utilizar una VPN junto con la desactivación del reenvío de IP, pero esto tiene algunas excepciones y limitaciones según dónde esté enrutando el tráfico su VPN, ya que esto puede implicar el reenvío de IP. Si el tráfico se enruta más allá de su servidor, es posible que deba mantener habilitado el reenvío de IP.

---

### Sección 5: Resumen

En este capítulo, aprendió sobre los diversos mecanismos de seguridad que ofrece MongoDB. Esto incluye la seguridad de las aplicaciones, que protege el acceso a su aplicación; la seguridad de los datos, que protege los datos mismos a lo largo de su ciclo de vida; y otros mecanismos de seguridad adicionales que se pueden aplicar a una implementación autogestionada de MongoDB. Con el conocimiento de todos estos mecanismos, puede elegir la combinación adecuada de medidas de seguridad para garantizar que su aplicación cuente con la protección que necesita.

En el próximo capítulo, profundizaremos en MongoDB Atlas, la plataforma en la nube para la gestión moderna de bases de datos. Aprenderá cómo configurar organizaciones, proyectos y clústeres, interactuar con Atlas mediante la interfaz de usuario, la CLI y las API, e implementar controles de acceso seguros. El capítulo también explora estrategias de escalado, herramientas de automatización y visualización de datos con Charts y Power BI.
