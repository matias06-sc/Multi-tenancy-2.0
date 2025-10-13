
# Configuración y Perfil de Usuario
## Gestión de Usuarios:
### HU1: Implementar Endpoints CRUD para Usuario
Como Desarrollador, 
quiero implementar los **endpoints RESTful** (`GET`, `POST`, `PUT`, `DELETE`) 
para la entidad `Usuario`, para permitir la **creación, lectura, actualización y eliminación** (CRUD).

**Criterios de Aceptación:**
- El endpoint `POST /api/usuarios` debe crear un nuevo usuario y **hashear (cifrar)** la contraseña antes de almacenarla, devolviendo **201 Creado**.
- El endpoint `PUT /api/usuarios/{id}` debe permitir la actualización de campos no sensibles (ej. nombre, teléfono) y devolver **200 OK**.
- El endpoint `PUT /api/usuarios/{id}/password` debe existir por separado y **validar la contraseña actual** del usuario antes de actualizar la nueva.
- El endpoint `GET /api/usuarios` debe devolver datos de usuario **excluyendo el hash de la contraseña** y otros datos sensibles.
- Debe implementarse **control de acceso** que restrinja el uso de estos endpoints **solo a usuarios con el rol de `Administrador`**.

Si un usuario con rol `Empleado` intenta acceder a cualquier endpoint de gestión de usuarios (`/api/usuarios`), la API debe devolver un código de respuesta **403 Prohibido**.
### HU2: Implementar Lógica de Roles y Estados

Como Desarrollador, quiero implementar la **lógica de asignación de roles** (`Administrador`, `Empleado`) y **validación de estados** (`Activo`, `Inactivo`).

**Criterios de Aceptación:**

- La creación y edición de usuarios debe **validar que el `rol` sea uno de la lista definida** (`Administrador`, `Empleado`).
- La creación y edición de usuarios debe **validar que el `estado` sea uno de la lista definida** (`Activo`, `Inactivo`).
- El sistema debe impedir que un usuario intente **cambiar su propio rol** a través de la API, a menos que sea un super-administrador.
- Si un usuario intenta iniciar sesión con `estado: Inactivo`, la API de login debe **rechazar la autenticación** con un mensaje de error apropiado.
## Servicios y precios:
### HU1: Crear Tabla y Endpoints CRUD para Servicios
Como Desarrollador, 
quiero crear la **tabla `Servicios`** en la base de datos con campos para `nombre`, `precio_base`, y `estado`, y sus respectivos endpoints CRUD.

**Criterios de Aceptación:**
- El endpoint `POST /api/servicios` debe validar que el `nombre` del servicio sea **único** y que `precio_base` sea **mayor o igual a cero**.
- Los precios deben almacenarse en la base de datos utilizando un **tipo de dato numérico (ej. `DECIMAL` o `NUMERIC`)** para evitar errores de precisión en cálculos monetarios.
- El endpoint `PUT /api/servicios/{id}` debe poder **actualizar masivamente el precio** de un servicio y registrar la fecha de la última modificación.
- El endpoint `DELETE /api/servicios/{id}` debe **desactivar lógicamente** el servicio (cambiar `estado` a `Inactivo`) si existen **pedidos asociados** a este.

Si un usuario con rol `Empleado` intenta acceder a cualquier endpoint de modificación de servicios (`POST`/`PUT`/`DELETE /api/servicios`), la API debe devolver un código de respuesta **403 Prohibido**.
## Configuración del Negocio
### HU1: Crear Tabla de Configuración y Garantizar Unicidad
Como Desarrollador, quiero crear una **tabla `ConfiguracionNegocio`** (o utilizar una colección de clave-valor) para almacenar datos como `nombre`, `RUC`, `email`, y `moneda`, garantizando que solo un registro pueda existir.

**Criterios de Aceptación:**
- La tabla o colección de la base de datos debe tener una **restricción de unicidad (Unique Constraint)** que impida la inserción de una segunda fila de configuración.
- Debe existir un endpoint único (ej. `PUT /api/configuracion`) que permita actualizar cualquiera de los campos (`nombre`, `RUC`, `email`, `moneda`).
- El backend debe **validar la longitud o formato del `RUC`** conforme a la normativa local antes de persistirlo.
- El campo `moneda` debe restringirse a un **enumerador o lista controlada** para evitar valores arbitrarios.

Si un usuario con rol `Empleado` intenta acceder al endpoint de configuración (`PUT /api/configuracion`), la API debe devolver un código de respuesta **403 Prohibido**.
## Autenticación de Dos Factores
### HU1: Integrar Servicio de 2FA (Para Administradores en Configuración)
Como Desarrollador, 
quiero integrar un servicio de **2FA (Two-Factor Authentication)** que maneje la generación y validación de códigos temporales para el inicio de sesión.

**Criterios de Aceptación:**
- El proceso de **inicio de sesión** debe verificar si el 2FA está activo para el usuario; si lo está, debe **pausar la sesión** y solicitar la clave temporal.
- Se debe integrar una biblioteca o servicio para **generar un código temporal (TOTP)** de 6 dígitos que expire a los 30 o 60 segundos.
- El endpoint de validación de 2FA debe **rechazar la autenticación** si el código es incorrecto o si ha **expirado** el tiempo de vigencia.
- Después de una validación exitosa del 2FA, el sistema debe **emitir el token de sesión** (ej. JWT) al cliente.
## Exportar Datos

### HU1: Crear Endpoint de Exportación JSON
Como Desarrollador, 
quiero crear un **endpoint `/api/exportar/json`** que genere un archivo JSON con los datos de las tablas especificadas (ej. `Usuarios`, `Servicios`) y lo devuelva al cliente.

**Criterios de Aceptación:**
- El endpoint `GET /api/exportar/json` debe ser **accesible solo para el rol de `Administrador`**.
- La respuesta del endpoint debe incluir el **encabezado HTTP `Content-Disposition: attachment`** para forzar la descarga del archivo.
- El archivo JSON generado debe contener los datos de las **tablas principales especificadas** (`Usuarios`, `Servicios`, etc.) en un formato estructurado y legible.
- La generación y respuesta de la exportación debe ser **eficiente** y no exceder un tiempo de respuesta razonable (ej. 10 segundos) para un volumen de datos típico.

Si un usuario con rol `Empleado` intenta acceder al endpoint de exportación, la API debe devolver un código de respuesta **403 Prohibido**.
## Eliminar Cuenta / Zona de Peligro
### HU1: Implementar Proceso de Eliminación de Cuenta con Seguridad
Como Desarrollador, 
quiero implementar un **proceso de eliminación lógica o física** de la cuenta que requiera una **confirmación de seguridad adicional** (ej. reingreso de contraseña).

**Criterios de Aceptación:**
- El endpoint de eliminación de cuenta debe **requerir la reintroducción de la contraseña actual** del usuario para confirmar la acción.
- La eliminación debe ser **lógica**; el usuario y sus datos deben **marcarse como `Eliminado/Inactivo`** y no deben poder iniciar sesión.
- Antes de marcar la cuenta como eliminada, el sistema debe **revocar todos los tokens de sesión** activos para ese usuario.
- Si el usuario a eliminar es el **último `Administrador` activo** en el sistema, la eliminación debe ser **bloqueada** por seguridad.

Los endpoints de la "Zona de Peligro" (eliminar cuenta ajena, resetear configuración) deben estar **protegidos** y devolver **403 Prohibido** si son accedidos por un `Empleado`.
# Mi perfil 
## Gestión de Información Personal y Seguridad
### HU1: Implementar Endpoints de Perfil Personal (Lectura y Edición)
Como Desarrollador, 
quiero implementar los endpoints del perfil personal (`GET`, `PUT`), 
para que **cualquier usuario** pueda ver y actualizar su propia información personal.

**Criterios de Aceptación Clave:**
- El endpoint **`GET /api/perfil`** debe devolver los datos personales visibles (Nombre, Teléfono, Dirección, Biografía, Email, Rol).
- El endpoint **`PUT /api/perfil`** debe permitir actualizar campos personales y debe **ignorar** cualquier intento de cambiar campos sensibles (`rol`, `estado`).
- La lectura y edición deben estar protegidas, de modo que **solo el propio usuario** pueda acceder a su información.
### HU2: Implementar Endpoints para Seguridad del Perfil (Botones Superiores)
Como Desarrollador, 
quiero crear endpoints dedicados, 
para que el usuario pueda **modificar su email y contraseña** (`Cambiar Email`, `Cambiar Contraseña`).

**Criterios de Aceptación Clave:**
- El endpoint de cambio de contraseña (`PUT /api/perfil/password`) debe **requerir y validar la contraseña actual**.
- El endpoint de cambio de email (`PUT /api/perfil/email`) debe iniciar un **proceso de verificación por token/enlace** enviado a la nueva dirección.
## Estadísticas y Actividad Reciente
### HU1: Implementar Endpoints de Estadísticas Personalizadas
Como Desarrollador, 
quiero implementar endpoints que devuelvan las **métricas clave** (`Pedidos Procesados`, `Clientes Atendidos`, etc.),
para el usuario autenticado, en el periodo "Este mes".

**Criterios de Aceptación Clave:**
- El endpoint (`GET /api/perfil/estadisticas`) debe calcular:
    - **Pedidos Procesados:** Cuenta de pedidos procesados por el usuario en el mes actual.
    - **Clientes Atendidos:** Cuenta de clientes **únicos** asociados a esos pedidos.
    - **Sesiones Realizadas:** Cuenta de inicios de sesión del usuario en el mes actual.
### HU2: Implementar Endpoint de Actividad Reciente
Como Desarrollador, quiero implementar un endpoint para devolver la lista de **acciones clave recientes** realizadas por el usuario, para la sección "Actividad Reciente".

**Criterios de Aceptación Clave:**
- El endpoint (`GET /api/perfil/actividad`) debe devolver un array de los últimos **5 a 10 logs de actividad** del usuario.
- Los logs deben estar **filtrados exclusivamente por el ID del usuario autenticado**.
- Cada elemento debe incluir la **descripción de la acción** y el **timestamp** para el cálculo de tiempo relativo ("Hace X horas").
# Reportes
## Implementación de KPIs y Gráficos 
### HU1: Implementar Endpoint para Métricas Globales (KPIs)
Como Desarrollador, 
quiero crear un endpoint que calcule y devuelva los **Indicadores Clave de Rendimiento (KPIs)** del negocio (Ventas Totales, Órdenes Completadas, etc.) basándose en un rango de fechas.

**Criterios de Aceptación:**
- Debe existir un endpoint principal (`GET /api/reportes/kpis`) que acepte los parámetros **`fecha_inicio`** y **`fecha_fin`** para definir el período de análisis.
- El campo **Ventas Totales** debe calcular la suma total del valor de todas las órdenes en estado `Completado` dentro del rango de fechas.
- El campo **Órdenes Completadas** debe devolver el número total de órdenes con estado `Completado` dentro del rango de fechas.
- El campo **Nuevos Clientes** debe calcular el número de clientes que se registraron por primera vez y realizaron su primer pedido dentro del rango de fechas.
- El sistema debe devolver también los **valores de comparación** ("vs. mes anterior") para cada KPI, calculando la misma métrica para el período inmediatamente anterior al solicitado.
### HU2: Implementar Endpoint para Evolución de Ventas
Como Desarrollador, quiero crear un endpoint que devuelva los datos de **Evolución de Ventas** (Ventas por Mes) para alimentar el gráfico de línea.

**Criterios de Aceptación:**
- Debe existir un endpoint (`GET /api/reportes/evolucion`) que devuelva un array de objetos (ej. `[{mes: 'Enero', ingreso: 10000}]`) para el período solicitado.
- La data debe agruparse por **mes calendario** o por el intervalo de tiempo seleccionado en el _frontend_ (si es diferente a meses).
- Cada punto de datos debe reflejar la **suma total de ingresos** de las órdenes completadas en ese intervalo.
### HU3: Implementar Endpoint para Distribución por Servicios
Como Desarrollador, quiero implementar un endpoint para calcular la **Distribución Por Servicios** para el gráfico circular, mostrando qué servicios generan mayor porcentaje de ventas.

**Criterios de Aceptación:**
- Debe existir un endpoint (`GET /api/reportes/distribucion/servicios`) que devuelva una lista de servicios y el **monto total de ventas** asociado a cada uno en el período.
- El backend debe calcular el **porcentaje de participación** de cada servicio sobre el total de ventas del período.
- La data devuelta debe permitir al _frontend_ identificar el nombre del servicio y su porcentaje para el gráfico (ej. "Impresión Minilab: 33%").
## Exportación de Reportes

### HU1: Implementar Endpoint de Exportación a Formato Excel
Como Desarrollador, 
quiero crear un endpoint que genere los datos de las ventas en formato **Excel (.xlsx)**, 
para que el usuario pueda exportarlos para análisis detallado.
**Criterios de Aceptación:**

- Debe existir un endpoint (`GET /api/reportes/exportar/excel`) que acepte los parámetros de **rango de fechas**.
- El endpoint debe generar una respuesta con el **header `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`** para forzar la descarga del archivo Excel.
- El archivo Excel generado debe contener **al menos una hoja** con los detalles de las órdenes completadas en el período (Fecha, Cliente, Servicio, Monto).
- La solicitud solo debe ser exitosa si el usuario tiene el **rol de `Administrador`**.
### HU2: Implementar Endpoint de Exportación a Formato PDF
Como Desarrollador, 
quiero crear un endpoint que genere un resumen visual de los reportes en formato **PDF**, 
para compartir fácilmente los resultados.

**Criterios de Aceptación:**
- Debe existir un endpoint (`GET /api/reportes/exportar/pdf`) que acepte los parámetros de **rango de fechas**.
- El endpoint debe generar una respuesta con el **header `Content-Type: application/pdf`**.
- El PDF generado debe incluir los **KPIs principales** (Ventas Totales, Órdenes Completadas) y un resumen de las **Ventas por Mes (Tabla)** del período.
- Se debe usar una biblioteca de generación de PDF confiable y eficiente en el backend.
# Contratos
## Gestión de Entidades
### HU1: Implementar Endpoints CRUD para Contratos
Como Desarrollador, 
quiero implementar los **endpoints RESTful (`GET`, `POST`, `PUT`, `DELETE`)**,
para la entidad `Contrato`, para permitir la creación, lectura, edición y gestión de cotizaciones.

**Criterios de Aceptación:**
- El endpoint **`POST /api/contratos`** debe permitir la creación de un nuevo contrato, asociándolo con un **`cliente_id`** y un **`usuario_creador_id`** (el usuario autenticado).
- El sistema debe validar que los campos obligatorios del contrato (ej. `nombre_entidad`, `valor_total`, `fecha_inicio`, `estado`) no estén vacíos en la creación.
- El endpoint **`PUT /api/contratos/{id}`** debe permitir la actualización de la mayoría de los campos del contrato, excepto el `ID` y el `usuario_creador_id`.
- El endpoint **`DELETE /api/contratos/{id}`** debe implementar una **eliminación lógica** (cambiar `estado` a `Archivado` o `Eliminado`) para preservar la trazabilidad histórica.
- El acceso a la creación (`POST`) debe estar disponible para los roles **`Administrador` y `Empleado`**, ya que ambos pueden estar a cargo de ventas.
### HU2: Implementar la Lógica de Estados de Contrato
Como Desarrollador, 
quiero implementar la lógica para gestionar los diferentes **estados del ciclo de vida** de un contrato, incluyendo el control del pago.

**Criterios de Aceptación:**
- El campo `estado` debe aceptar solo valores definidos por el sistema (ej. `Activo`, `Pendiente`, `Pagado`, `Completado`, ).
- Debe existir un endpoint dedicado (ej. **`PUT /api/contratos/{id}/pagar`**) que, al ser llamado, actualice el estado a `Pagado` y registre el monto y la fecha del pago en una tabla de `Pagos`.
- El sistema debe calcular y exponer el estado `Pendiente` cuando el contrato ha sido creado pero el pago no se ha completado.
- El sistema debe **calcular el `Valor Total`** del contrato (sumando los servicios o productos asociados) y registrar el `Total Pagado` (`Valor Total Pagado`).
## Lectura, Búsqueda y Filtros
### HU1: Implementar Endpoint de Listado y Búsqueda de Contratos
Como Desarrollador, 
quiero implementar un endpoint para **listar y buscar contratos** que soporte filtros dinámicos.

**Criterios de Aceptación:**
- El endpoint **`GET /api/contratos`** debe devolver una **lista paginada** de los contratos.
- El endpoint debe aceptar parámetros de consulta para **filtrar por `estado`** (ej. `Activo`, `Pendiente`, `Pagado`,`Completado`) y por `tipo` (`Anual`, `Semestral`, `Anual`, `Por Proyecto`).
- El endpoint debe aceptar un parámetro de búsqueda de texto libre (ej. `search=cliente_o_servicio`) que permita buscar coincidencias en los campos `nombre_entidad` y `servicio_principal`.
- Para el rol **`Empleado`**, la lista devuelta solo debe incluir los contratos donde el `usuario_creador_id` sea su propio ID, a menos que tenga un permiso especial.
### HU2: Implementar Endpoint para Métricas del Módulo 
Como Desarrollador, 
quiero crear un endpoint que calcule y devuelva las métricas clave de contratos (`Total Contratos`, `Activos`, `Valor Total`, `Total Pagado`) 
para la vista del dashboard.

**Criterios de Aceptación:**
- Debe existir un endpoint (ej. **`GET /api/contratos/kpis`**) que devuelva los 4 indicadores superiores de forma rápida.
- **Total Contratos:** Debe calcular la cuenta total de contratos en la base de datos (excluyendo los eliminados lógicamente).
- **Activos:** Debe calcular la cuenta de contratos con el estado `Activo`.
- **Valor Total (Global):** Debe calcular la suma del campo `valor_total` de **todos** los contratos.
- **Total Pagado:** Debe calcular la suma total de todos los pagos registrados en la tabla `Pagos` para los contratos en el sistema.

Si el usuario es un **`Empleado`**, las métricas de **`Valor Total`** y **`Total Pagado`** deben ser **ocultadas** o **filtradas** para mostrar solo los valores asociados a sus propios contratos, a menos que el negocio defina lo contrario.
## Utilidades
### HU2: Implementar funcionalidad de Reset de Datos
Como Desarrollador, 
quiero implementar la lógica de backend para el botón **"Reset Datos"** en la vista de Contratos.

**Criterios de Aceptación:**
- Debe existir un endpoint dedicado (ej. **`POST /api/contratos/reset`**) que solo sea accesible para el rol **`Administrador`**.
- La llamada a este endpoint debe **eliminar físicamente todos los contratos** y sus pagos asociados de la base de datos (para fines de prueba o limpieza).
- El endpoint debe requerir una **confirmación de seguridad adicional** (ej. reingreso de contraseña del Administrador) antes de ejecutar la acción.
- Debe enviarse una **respuesta 200 OK** después de la eliminación masiva.
# Activos
## Inventario de Equipos
### HU1: Implementar Endpoints CRUD para Activos
Como Desarrollador, 
quiero implementar los **endpoints RESTful (`GET`, `POST`, `PUT`, `DELETE`)** para la entidad `Activo`, que representa cada equipo físico.

**Criterios de Aceptación:**
- El endpoint **`POST /api/activos`** debe permitir crear un nuevo activo con campos obligatorios: `nombre`, `categoria`, `proveedor`, y `costo_total`.
- El sistema debe validar que el `costo_total` sea un **valor numérico positivo**.
- El endpoint **`GET /api/activos`** debe devolver el listado de activos, permitiendo **búsqueda por texto** y filtrado por `Categoría`, `Proveedor` y `Estado`.
- El endpoint **`PUT /api/activos/{id}`** debe permitir actualizar los datos del activo y modificar el `estado` (ej. `Activo`, `Inactivo`, `Mantenimiento`).
- El endpoint **`DELETE /api/activos/{id}`** debe implementar una **eliminación lógica** del activo.
### HU2: Implementar la Lógica de Estado y Tipo de Pago de Activo
Como Desarrollador, 
quiero implementar la lógica de backend para manejar el estado operativo y el tipo de adquisición del activo.

**Criterios de Aceptación:**
- El campo **`tipo_pago`** de la entidad `Activo` debe restringirse a un conjunto de valores definidos (ej. `Contado`, `Financiado`, `Leasing`).
- El backend debe permitir que el **`estado`** de un activo se actualice automáticamente a `Mantenimiento` cuando se crea un nuevo registro de mantenimiento para ese activo.
- El endpoint de creación (`POST /api/activos`) debe registrar el **`usuario_creador_id`** para la trazabilidad.
## Financiamientos

### HU1: Implementar Endpoints CRUD para Financiamientos
Como Desarrollador, 
quiero implementar los endpoints CRUD para la entidad **`Financiamiento`**, que rastrea la deuda asociada a los activos financiados.

**Criterios de Aceptación:**
- El endpoint **`POST /api/financiamientos`** debe requerir la asociación con un **`activo_id`** válido.
- El endpoint debe validar que la `cuota_mensual` y el `monto_financiado` sean valores positivos.
- El campo **`cuotas_pagadas`** debe ser un valor editable y debe ser **menor o igual** al total de `cuotas_totales`.
- El endpoint **`GET /api/financiamientos`** debe devolver el listado, incluyendo la información del `Activo` asociado (ej. Nombre del Activo).
- El backend debe calcular el **estado de la deuda** (ej. `Activo`, `Pagado`, `Demora`) basándose en la comparación de `cuotas_pagadas` vs. `cuotas_totales`.
### HU2: Implementar la Lógica de Deuda y Estado Financiero
Como Desarrollador, 
quiero implementar la lógica
para gestionar la progresión del pago del financiamiento.

**Criterios de Aceptación:**
- Debe existir un endpoint dedicado (ej. `PUT /api/financiamientos/{id}/pagar_cuota`) que **incremente la `cuota_pagada`** en 1 unidad.
- El sistema debe **prohibir** que se incremente la `cuota_pagada` si ya es igual al total de `cuotas_totales`.
- Al alcanzar la última cuota, el sistema debe **actualizar el `tipo_pago`** del `Activo` asociado a `Contado` (o `Pagado`).
## Mantenimientos

### HU1: Implementar Endpoints CRUD para Mantenimientos
Como Desarrollador, 
quiero implementar los endpoints CRUD 
para la entidad **`Mantenimiento`**, que rastrea las reparaciones y el estado de servicio de los activos.

**Criterios de Aceptación:**
- El endpoint **`POST /api/mantenimientos`** debe requerir la asociación con un **`activo_id`** válido, la `fecha_programada` y el `costo` del servicio.
- El backend debe validar que el `tipo` de mantenimiento sea de una lista predefinida (ej. `Preventivo`, `Correctivo`).
- El endpoint **`PUT /api/mantenimientos/{id}`** debe permitir actualizar el `estado` (ej. `Programado`, `Completado`, `Cancelado`).
- Cuando un mantenimiento se registra, el backend debe **actualizar automáticamente el `estado` del `Activo`** asociado a `Mantenimiento` 
- Cuando un mantenimiento se marca como `Completado`, el backend debe **revertir el `estado` del `Activo`** asociado a `Activo` (o el estado anterior).

# Clientes
## Gestión de Entidades
### HU1: Implementar Endpoints CRUD para Clientes
Como Desarrollador, 
quiero implementar los **endpoints RESTful (`GET`, `POST`, `PUT`, `DELETE`)** 
para la entidad `Cliente`, para gestionar la base de datos de clientes.

**Criterios de Aceptación:**
- El endpoint **`POST /api/clientes`** debe permitir la creación de un nuevo cliente y generar automáticamente un **`ID` único** y legible (ej. C001, C002).
- El sistema debe validar que el campo **`Nombre`** y al menos un campo de **`Contacto`** (ej. teléfono o email) sean obligatorios en la creación.
- El endpoint **`PUT /api/clientes/{id}`** debe permitir la actualización de todos los campos del cliente (Dirección, Detalles Adicionales, etc.).
- El endpoint **`DELETE /api/clientes/{id}`** debe implementar una **eliminación lógica** (cambiar `estado` a `Inactivo` o `Archivado`) para mantener la trazabilidad de pedidos históricos.
- La creación y modificación de clientes debe estar disponible para los roles **`Administrador`** y **`Empleado`**.
### HU2: Implementar la Lógica de Tipo de Cliente
Como Desarrollador, 
quiero implementar la lógica 
para clasificar a los clientes según su tipo (Colegio, Particular, Empresa, etc.).

**Criterios de Aceptación:**
- El campo **`Tipo`** de la entidad `Cliente` debe restringirse a un conjunto de valores definidos (ej. `Colegio`, `Particular`, `Empresa`).
- En el caso de los tipos `Colegio` o `Empresa`, el campo **`Nombre I.E.`** o **`Razón Social`** (mostrado como `Nombre I.E.` en la tabla) debe ser **obligatorio** y validado.
- El campo `Contacto` debe aplicar una **validación de formato** (ej. 9 dígitos para teléfono, formato de email) según el tipo de dato ingresado.
## Lectura, Búsqueda y Filtros
### HU1: Implementar Endpoint de Listado, Búsqueda y Filtros
Como Desarrollador, 
quiero implementar un endpoint 
para **listar clientes** que soporte búsqueda por texto y filtros dinámicos.

**Criterios de Aceptación:**
- El endpoint **`GET /api/clientes`** debe devolver una **lista paginada** de los clientes.
- El endpoint debe aceptar un parámetro de consulta de texto libre (ej. `search=`) que filtre por coincidencias en los campos **`Nombre`**, **`Contacto`**, y **`Dirección`**.
- El endpoint debe aceptar un parámetro de filtro por **`tipo_cliente`** (ej. `GET /api/clientes?tipo=Colegio`).
- Los resultados de la lista deben devolver todas las columnas visibles en la tabla (ID, Nombre, Tipo, Contacto, Dirección, Detalles Adicionales).
### HU4: Implementar Lógica de Paginación
Como Desarrollador, 
quiero implementar la lógica de paginación 
para garantizar la eficiencia al listar grandes volúmenes de clientes.

**Criterios de Aceptación:**
- El endpoint **`GET /api/clientes`** debe aceptar los parámetros **`limit`** (ej. 20) y **`offset`** o **`page`** (ej. 1) para controlar la paginación.
- La respuesta de la API debe incluir metadatos de paginación: **`total_registros`**, **`pagina_actual`** y **`total_paginas`**, para que el _frontend_ pueda mostrar la barra de navegación ("Anterior" / "Siguiente").
- La paginación y el filtrado deben trabajar de manera **conjunta y eficiente** (es decir, el conteo total de registros debe reflejar el filtro aplicado).

# Pedidos
## Gestión de Entidades
### HU1: Implementar Endpoints CRUD para Pedidos
Como Desarrollador, 
quiero implementar los **endpoints RESTful (`GET`, `POST`, `PUT`, `DELETE`)** 
para la entidad `Pedido`, para gestionar las órdenes de trabajo.

**Criterios de Aceptación:**
- El endpoint **`POST /api/pedidos`** debe permitir la creación de un nuevo pedido, requiriendo un **`cliente_id`** válido y registrando el **`usuario_creador_id`** (el usuario autenticado).
- El sistema debe validar que los campos obligatorios (ej. `cliente_id`, `tipo_producto`, `fecha_pedido`) no estén vacíos.
- El endpoint **`PUT /api/pedidos/{id}`** debe permitir la modificación de los datos del pedido, incluyendo la actualización de los campos `Estado`, `Subestado` y `Progreso (%)`.
- El endpoint **`DELETE /api/pedidos/{id}`** debe implementar una **eliminación lógica** del pedido para mantener el registro histórico y la integridad de los reportes financieros.
- La creación y edición de pedidos debe estar disponible para los roles **`Administrador` y `Empleado`**.
## Lógica Financiera y de Progreso
### HU1: Implementar Lógica de Costos, Precios y Utilidad
Como Desarrollador, 
quiero implementar la lógica de backend para calcular y almacenar las métricas financieras clave de cada pedido.

**Criterios de Aceptación:**
- El sistema debe requerir que los campos **`Costo Estimado`** y **`Precio Venta`** sean valores numéricos (tipo `DECIMAL` o `NUMERIC` en BD) y positivos.
- El campo **`Utilidad`** debe ser calculado automáticamente en el backend como: **`Precio Venta - Costo Estimado`**.
- El campo **`Avance (%)`** (Mostrado como `Avance`) debe ser calculado automáticamente como: **`(Utilidad / Precio Venta) * 100`** si el Precio Venta es mayor a cero.
- El backend debe manejar correctamente la visualización y el cálculo de **`Utilidad` en negativo** (ej. si el costo excede el precio de venta).
### HU2: Implementar Lógica de Estados y Fechas
Como Desarrollador, 
quiero implementar la lógica de los estados y subestados 
para reflejar el flujo de trabajo del pedido.

**Criterios de Aceptación:**
- La entidad `Pedido` debe tener un campo **`Estado`** restringido a un catálogo de valores predefinidos (ej. `Listo para entrega`, `Entregado`, `En Producción`, `Pendiente de confirmación`).
- El campo **`Subestado`** debe permitir una descripción de texto o un valor de catálogo que detalle el punto exacto del `Estado` (ej. "Retoque Final", "Esperando Pago").
- El campo **`Progreso (%)`** debe ser un valor numérico editable entre 0 y 100, y su actualización debe registrar la fecha y hora de modificación.
- El sistema debe permitir el registro de la **`Fecha Pedido`** y la **`Fecha Compromiso`** (la fecha límite acordada para la entrega).
## Lectura, Búsqueda y Filtros

### HU3: Implementar Endpoint de Listado, Búsqueda y Filtros de Pedidos
Como Desarrollador, 
quiero implementar un endpoint 
para **listar pedidos** que soporte búsqueda y filtrado, esencial para la operatividad diaria.

**Criterios de Aceptación:**
- El endpoint **`GET /api/pedidos`** debe devolver una **lista paginada** de los pedidos.
- El endpoint debe aceptar un parámetro de búsqueda de texto libre que filtre pedidos por el **nombre del cliente/colegio** asociado.
- El endpoint debe aceptar parámetros de filtro para **`Cliente`**, **`Tipo de Producto`** y **`Estado`**.
- La paginación debe ser implementada (`limit` y `offset`/`page`) y debe incluir metadatos de paginación (`total_registros` filtrados).

Si el usuario es **`Empleado`**, la lista debe priorizar o restringir los pedidos donde su `usuario_creador_id` sea el propio ID, a menos que el `Administrador` le conceda acceso global.
