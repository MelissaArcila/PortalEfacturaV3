# Glosario Funcional  
**Ecosistema de Facturación Electrónica DIAN**

Este glosario define los principales términos del negocio utilizados en la plataforma de facturación electrónica.  
Está orientado a **desarrolladores**, **QA** y **stakeholders no técnicos**, con el fin de asegurar un entendimiento común y consistente.

---

## Cliente

Entidad (empresa o persona natural) que tiene contratado con nosotros **al menos uno de los productos del ecosistema de facturación electrónica de la DIAN**, y que utiliza nuestra plataforma para la emisión, recepción, validación, transmisión o consulta de documentos electrónicos.

> Un cliente puede tener uno o varios productos activos dentro del ecosistema y uno o varios usuarios asociados.

---

## Usuario

Persona que accede a la plataforma en nombre de un cliente, con permisos definidos según su rol (administrativo, operativo, consulta, etc.).

---

## Facturación Electrónica

Sistema definido por la DIAN que permite la **emisión, validación, transmisión y conservación** de facturas y documentos equivalentes en formato electrónico, cumpliendo requisitos técnicos y normativos.

---

## Factura Electrónica

Documento electrónico que respalda una operación de venta de bienes o prestación de servicios.  
Debe cumplir con el formato XML definido por la DIAN y pasar por el proceso de validación previa.

---

## Documento Electrónico

Término general que agrupa los distintos tipos de documentos gestionados dentro del ecosistema DIAN, como facturas, notas, documentos soporte y eventos.

---

## DIAN

Dirección de Impuestos y Aduanas Nacionales de Colombia.  
Entidad gubernamental responsable de definir la normativa, validaciones y estados de los documentos de facturación electrónica.

---

## Validación DIAN

Proceso mediante el cual la DIAN revisa un documento electrónico y determina si cumple con las reglas técnicas y fiscales.

---

## Estado DIAN

Resultado del proceso de validación de un documento electrónico.  
Algunos estados comunes son:

- **Aprobado**: el documento cumple con los requisitos.
- **Rechazado**: el documento presenta errores técnicos o normativos.


---

## Detalle de Estados

Historial completo de los eventos y transiciones por los que pasa un documento electrónico, incluyendo respuestas técnicas, errores y confirmaciones.

---

## XML

Archivo estructurado que contiene la información de la factura electrónica u otro documento, siguiendo el esquema definido por la DIAN.

---

## PDF de Representación Gráfica

Archivo en formato PDF generado a partir del XML, utilizado como representación visual del documento electrónico para consulta o envío al cliente final.

---

## AttachedDocument (XML)

Archivo XML que contiene información adicional asociada a la factura electrónica, como eventos, respuestas o documentos relacionados, según la normativa DIAN.

---

## Evento

Registro electrónico que refleja una acción sobre una factura, especialmente en el contexto de RADIAN (aceptación, rechazo, endoso, etc.).

---

## RADIAN

Registro de la Factura Electrónica como Título Valor.  
Es el sistema de la DIAN que permite la **gestión de eventos asociados a la factura electrónica**, como aceptación expresa, aceptación tácita, rechazo y endoso.

---

## Prefijo

Código alfanumérico que identifica una serie de numeración de facturas electrónicas, autorizado por la DIAN.

---

## Resolución de Facturación

Autorización otorgada por la DIAN que define los rangos de numeración, prefijos, fechas de vigencia y tipos de documentos que un cliente puede emitir.

---

## Estado de Resolución

Indicador del estado operativo de una resolución de facturación en el sistema. Los estados posibles son:
- **Activa**: La resolución está habilitada para la emisión de documentos electrónicos.
- **Inactiva**: La resolución fue desactivada manualmente y no puede usarse para emisión de documentos.
- **Vencida**: La resolución alcanzó su fecha final de vigencia y ya no puede usarse (estado automático).
- **Agotada**: La resolución utilizó todos los consecutivos de su rango numérico y ya no puede usarse (estado automático).

---

## Prefijo de Resolución

Código alfanumérico de hasta 4 caracteres (letras mayúsculas y/o números) que identifica una serie de numeración de facturas electrónicas autorizada por la DIAN. El prefijo se antepone al número consecutivo para formar el número completo del documento.

---

## Rango de Consecutivos

Intervalo numérico definido por un consecutivo inicial y un consecutivo final que delimita los números disponibles para la emisión de documentos bajo una resolución de facturación específica.

---

## Traslape de Rangos

Situación no permitida en la cual dos resoluciones del mismo tipo de documento y prefijo tienen rangos de consecutivos que se superponen parcial o totalmente, independientemente de su estado.

---

## Punto de Venta o Sede

Ubicación física o lógica desde la cual un cliente emite documentos electrónicos. Cada resolución puede estar asociada opcionalmente a un punto de venta específico.

---

## Reenvío de Correo

Funcionalidad que permite volver a enviar al destinatario los correos asociados a una factura electrónica, incluyendo archivos como PDF y XML.

---

## Reporte

Archivo o vista generada por la plataforma que permite consultar información básica de facturas y documentos electrónicos, generalmente con filtros por fecha, estado o tipo.

---

## Portal Unificado

Plataforma web centralizada donde el cliente puede consultar el estado, detalle y archivos de sus documentos electrónicos, configurar información básica y realizar acciones operativas.

---

## Seguridad

Conjunto de mecanismos que garantizan la confidencialidad, integridad y disponibilidad de la información, incluyendo autenticación, autorización y control de accesos.

---

## Trazabilidad

Capacidad de seguir el recorrido completo de un documento electrónico desde su emisión hasta su estado final, incluyendo todos los eventos y respuestas de la DIAN.

---

## Ecosistema de Facturación Electrónica DIAN

Conjunto de productos y procesos regulados por la DIAN que soportamos como proveedor tecnológico, incluyendo:

- **Facturación Electrónica de Venta**
- **Notas Crédito y Débito**
- **Documento Soporte Electrónico**
- **RADIAN (Factura Electrónica como Título Valor)**

---

## Productos del Ecosistema

### Facturación Electrónica de Venta
Permite la emisión y validación de facturas electrónicas ante la DIAN.

### Notas Crédito y Débito
Documentos electrónicos utilizados para corregir, anular o ajustar facturas electrónicas previamente emitidas.

### Documento Soporte Electrónico
Documento que respalda costos y gastos cuando el proveedor no está obligado a facturar electrónicamente.

### RADIAN
Producto que permite la gestión de eventos de la factura electrónica como título valor, habilitando procesos como aceptación, rechazo y endoso.

---

## Autogestión

Capacidad del cliente para consultar información, descargar documentos y ejecutar acciones básicas sin intervención del equipo de soporte.

---

## Soporte

Equipo interno encargado de atender incidencias, consultas y problemas operativos relacionados con la plataforma y los documentos electrónicos.

---

## Sesión de Usuario

Período de tiempo durante el cual un usuario autenticado tiene acceso activo al sistema bajo el contexto de un cliente específico. La sesión inicia después de una autenticación exitosa y selección de cliente, y finaliza por cierre explícito del usuario, inactividad o expiración del tiempo de sesión.

---

## Bloqueo de Cuenta

Estado temporal de una cuenta de usuario que impide el acceso al sistema debido a intentos fallidos consecutivos de autenticación. El bloqueo es un mecanismo de seguridad para proteger contra ataques de fuerza bruta y adivinación de contraseñas.

---

## Desbloqueo Automático

Proceso mediante el cual una cuenta de usuario bloqueada por intentos fallidos de autenticación recupera automáticamente su capacidad de acceso después de transcurrido un período de tiempo definido, sin intervención manual de un administrador.

---

## Contexto de Cliente

Conjunto de información, permisos y configuraciones específicas asociadas a un cliente particular, que determinan el alcance de acceso y funcionalidades disponibles para un usuario durante su sesión en el sistema.

---

## Rol

Conjunto predefinido de permisos y capacidades que determinan las acciones que un usuario puede realizar en el sistema. Los roles se clasifican en dos tipos principales: roles de cliente (asociados a empresas contratantes) y roles internos (para personal de la organización).

---

## Administrador de Portal

Rol interno con máximo nivel de privilegios en el Portal Unificado. Tiene capacidad para crear y gestionar usuarios de cualquier tipo, administrar empresas, configurar productos, y acceder a todas las funcionalidades del sistema sin restricciones.

---

## Administrador de Cliente

Rol asignado a usuarios de una empresa cliente que tienen capacidad para gestionar usuarios de su propia organización, asignar permisos a sus colaboradores, y administrar la configuración de su empresa en el portal, pero sin acceso a funcionalidades de administración global del sistema.

---

## Gestor

Rol operativo asignado a usuarios de cliente que tienen permisos para ejecutar acciones específicas sobre uno o más productos contratados (emisión de facturas, recepción de documentos, generación de eventos RADIAN, etc.). Cada producto tiene su propio rol de gestor específico.

---

## Usuario Interno

Usuario que pertenece al personal de la organización proveedora del servicio (CDN Facturación) y tiene asignados roles internos como Administrador de Portal, Soporte Técnico, Auditor Interno, Desarrollador, o Consultor Funcional. Los usuarios internos pueden también tener roles de cliente asignados para propósitos de soporte.

---

## Usuario de Cliente

Usuario que pertenece a una empresa contratante del servicio y tiene asignados uno o más roles de cliente (Administrador de Cliente, Gestor) para acceder y operar sobre los productos contratados por su organización.

---

## Permiso

Asignación específica de un rol a un usuario para un cliente determinado. Un usuario puede tener múltiples permisos que le permiten acceder a diferentes clientes con diferentes roles en cada uno. Por ejemplo, un usuario puede ser Gestor Emisión FE en la Empresa A y Gestor RADIAN en la Empresa B.

---

## Producto

Servicio o módulo funcional del ecosistema de facturación electrónica DIAN que un cliente puede contratar. Los productos incluyen: Emisión Factura Electrónica (FE), Emisión POS, Emisión Documento Equivalente, Recepción FE, Nómina Electrónica, Emisión Documento Soporte, RADIAN, Notas Crédito, Notas Débito.

---

## Número de Identificación

Identificador único asociado a un usuario en el sistema, generalmente correspondiente a su documento de identidad (cédula, pasaporte, etc.). Este número debe ser único en toda la plataforma y se utiliza para autenticación y trazabilidad de acciones.

---

## Estado de Usuario

Indicador del estado operativo de una cuenta de usuario en el sistema. Los estados posibles son:
- **Activo**: El usuario puede autenticarse y acceder al sistema según sus permisos.
- **Inactivo**: El usuario no puede autenticarse ni acceder al sistema, pero su cuenta se mantiene en el sistema para fines de auditoría.
- **Bloqueado**: Estado temporal que impide el acceso debido a múltiples intentos fallidos de autenticación.

---

## Asignación de Permisos

Proceso mediante el cual un Administrador de Portal otorga acceso a un usuario sobre uno o más clientes con roles específicos, determinando las capacidades y alcance de acceso del usuario en el sistema. Cada asignación se registra con detalle de quién, cuándo y qué permiso fue otorgado para fines de auditoría.

---
