### Módulo: Producción

## Gestión de Órdenes

### **HU1: Implementar Registro de Órdenes de Producción**
Como Desarrollador,  
quiero implementar el registro de órdenes de producción con los datos del cliente y materiales utilizados,  
para iniciar y controlar el proceso de fabricación dentro del sistema.

**Criterios de Aceptación:**
- El formulario debe permitir registrar los campos: **`Cliente`**, **`Moldura`**, **`Vidrio`**, **`MDF`**, **`Paspartú`**, **`Accesorios`**, **`Mermas`** y **`Fecha de Inicio`**.  
- Al guardar la orden, el backend debe generar automáticamente un **ID de producción único** y establecer el **estado inicial `Pendiente`**.  
- Los campos obligatorios deben validarse antes del guardado; si falta alguno, se mostrará un mensaje de error.  
- Los datos deben almacenarse correctamente en la base de datos (o enviarse al endpoint **`POST /api/produccion`**).  

---

### **HU2: Implementar Visualización de Órdenes Activas**
Como Desarrollador,  
quiero implementar la vista que muestra las órdenes de producción activas,  
para permitir al usuario supervisar el avance y el estado de cada trabajo.

**Criterios de Aceptación:**
- La tabla debe incluir las columnas: **Cliente**, **Moldura**, **Vidrio**, **MDF**, **Paspartú**, **Accesorios**, **Mermas**, **Fecha de Inicio** y **Estado**.  
- Los estados deben mostrarse con **etiquetas o badges de color** (`Pendiente`, `En proceso`, `Terminado`, `Entregado`).  
- El endpoint **`GET /api/produccion`** debe soportar **búsqueda por cliente o ID de orden**.  
- La vista debe **actualizarse dinámicamente** tras registrar o editar una orden.  

---

### **HU3: Implementar Edición y Actualización de Órdenes**
Como Desarrollador,  
quiero permitir la edición de materiales y estados en órdenes existentes,  
para reflejar cambios en tiempo real durante la producción.

**Criterios de Aceptación:**
- Debe existir un endpoint **`PUT /api/produccion/{id}`** para modificar los materiales y el estado de la orden.  
- No se debe permitir eliminar ni modificar órdenes con estado **`Entregado`**.  
- Cada actualización debe registrar la **fecha y hora del cambio** y el **usuario responsable**.  
- La interfaz debe mostrar una **notificación de confirmación** al actualizar.  

---

### **HU4: Implementar Eliminación de Órdenes**
Como Desarrollador,  
quiero implementar la eliminación lógica de órdenes,  
para mantener la base de datos limpia y consistente.

**Criterios de Aceptación:**
- Solo los usuarios con rol **`Administrador`** pueden eliminar órdenes.  
- Antes de eliminar, debe mostrarse una **ventana modal de confirmación**.  
- Si la orden tiene estado `En proceso`, `Terminado` o `Entregado`, el sistema debe **impedir su eliminación** y mostrar una advertencia.  
- El endpoint **`DELETE /api/produccion/{id}`** debe marcar la orden como **eliminada lógicamente**, sin borrarla físicamente.  

---

## Control de Estados y Submódulos

### **HU5: Implementar Control de Estados de Producción**
Como Desarrollador,  
quiero implementar la lógica de control de estados en el módulo de producción,  
para reflejar el flujo real de trabajo dentro del sistema.

**Criterios de Aceptación:**
- Los estados válidos deben ser: **`Pendiente`**, **`En proceso`**, **`Terminado`**, **`Entregado`**.  
- Cada cambio debe registrar la **fecha**, **hora** y **usuario** que lo realizó.  
- La interfaz debe actualizar las etiquetas visuales y colores de estado automáticamente.  
- Solo los roles autorizados pueden cambiar a estado **`Entregado`**.  

---

### **HU6: Implementar Acceso a Submódulos de Producción**
Como Desarrollador,  
quiero desarrollar el acceso a los submódulos del área de producción,  
para que los empleados puedan navegar entre secciones específicas según su rol.

**Criterios de Aceptación:**
- En la interfaz principal deben mostrarse pestañas o botones de navegación a los submódulos:  
  **`Minilab`**, **`Recordatorios`**, **`Corte Láser`**, **`Accesorios`**, **`Edición Digital`**.  
- Cada pestaña debe redirigir a su vista correspondiente (por ejemplo, `/produccion/minilab`).  
- El sistema debe conservar la **sesión del usuario** al cambiar de submódulo.  
- Los permisos de acceso deben depender del **rol** del usuario autenticado.  

---

## Mermas y Reportes

### **HU7: Implementar Registro y Control de Mermas**
Como Desarrollador,  
quiero implementar el registro de materiales desperdiciados,  
para permitir el control de eficiencia y el cálculo de pérdidas.

**Criterios de Aceptación:**
- El campo **`Mermas`** debe aceptar solo valores numéricos y positivos.  
- Cada registro de merma debe almacenar **fecha**, **cantidad** y **motivo**.  
- Debe existir un endpoint **`POST /api/produccion/mermas`** y uno de listado **`GET /api/produccion/mermas`**.  
- El sistema debe permitir generar un **resumen de mermas** filtrado por fecha u orden.  

---

### **HU8: Implementar Reportes de Producción**
Como Desarrollador,  
quiero implementar la generación de reportes de producción,  
para analizar el rendimiento general del taller.

**Criterios de Aceptación:**
- El sistema debe permitir **filtrar por fechas y estado de orden**.  
- El reporte debe incluir: número total de órdenes, materiales usados y cantidad de entregas.  
- Debe poder **exportarse a PDF o Excel** desde la interfaz.  
- El endpoint **`GET /api/produccion/reportes`** debe devolver la información consolidada para el periodo seleccionado.  

