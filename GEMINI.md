# Reglas de Trabajo - ElectroFrío V2

Este documento establece las directrices operativas del negocio, las reglas funcionales de cada módulo del sistema y los estándares técnicos de desarrollo para ElectroFrío.

---

## 1. Control de Versiones y Directrices de Desarrollo (Asistente AI)

- **CERO COMMITS Y PUSH AUTOMÁTICOS:** El asistente NUNCA debe ejecutar `git commit` ni `git push`.
- **REVISIÓN DEL USUARIO:** Todos los cambios deben permanecer en el árbol de trabajo local (*unstaged*) para que el usuario los verifique manualmente con `git diff` / `git status` y realice sus propios commits.
- **ARQUITECTURA OFFLINE-FIRST Y TIEMPO REAL:** Toda acción en la interfaz debe actualizar inmediatamente la memoria local (`localStorage`) y sincronizarse en segundo plano con Google Firestore (`merge: true`).
- **COMUNICACIÓN ENTRE PESTAÑAS (IFRAMES):** Al crear, editar o eliminar registros (Clientes, Órdenes, Planillas, Fichajes), siempre emitir el evento correspondiente vía `window.parent.postMessage` y retransmitirlo a todos los módulos activos.
- **CONFIRMACIÓN DE SEGURIDAD:** Las operaciones destructivas (eliminar clientes, planillas u órdenes) requieren confirmación explícita o clave de seguridad administrativa (`123`).

---

## 2. Módulo de Clientes

1. **Identificador Único Correlativo:**
   - Todo cliente nuevo recibe automáticamente un código correlativo con el formato `CLI-xxxx` (ej. `CLI-0001`, `CLI-0015`), calculado a partir del valor más alto registrado.
2. **Campos Obligatorios Estrictos:**
   - **Nombre o Razón Social:** Obligatorio.
   - **CUIT / DNI:** Obligatorio.
   - **Teléfono de Contacto:** Obligatorio.
   - **Domicilio(s):** Al menos un domicilio físico válido es estrictamente obligatorio.
3. **Múltiples Domicilios y Enlace con Google Maps:**
   - Un cliente puede tener múltiples sucursales o domicilios.
   - Cada domicilio puede contar con su dirección física escrita y su link/coordenadas de Google Maps.
   - Si se pega un enlace de Google Maps en el campo de dirección, el sistema debe detectarlo automáticamente.
4. **Orden de Visualización:**
   - La tabla de clientes debe mostrarse siempre en **orden descendente** (del cliente más reciente o último registrado al más antiguo).
5. **Sincronización Total con Órdenes y Planillas:**
   - Cualquier cliente nuevo o modificado debe reflejarse en tiempo real en los buscadores de Órdenes de Trabajo y Planillas sin necesidad de recargar la página.

---

## 3. Módulo de Fichajes y Control Horario

1. **Geolocalización GPS Satelital:**
   - Todo fichaje de Entrada y Salida captura las coordenadas GPS del dispositivo para verificar si el técnico se encuentra en la base de la empresa o en campo.
2. **Regla de Auditoría de Edición Manual:**
   - Si un fichaje original tenía el badge de "GPS Verificado", pero es **editado o corregido posteriormente** por el administrador, su estado debe cambiar automáticamente a **Manual (Sin GPS)** con registro de edición.
   - Queda prohibido que un fichaje modificado manualmente conserve la certificación de GPS satelital en vivo.
3. **Orden Cronológico en la Tabla de Fichajes:**
   - En la vista de entradas y registros diarios, los registros deben ordenarse claramente: el fichaje más antiguo de la jornada abajo y el último/más reciente arriba (o en orden cronológico natural de jornada laboral).

---

## 4. Módulo de Órdenes de Trabajo (OT)

1. **Numeración y Trazabilidad:**
   - Identificadores únicos correlativos con formato `OT-xxxx` (ej. `OT-0025`).
2. **Asignación de Clientes y Ubicaciones:**
   - Al seleccionar un cliente en el buscador (por nombre, código `CLI-xxxx`, teléfono o calle), se deben desplegar inmediatamente todos sus domicilios registrados en tarjetas de selección rápida.
   - Cada domicilio dispone de su botón `🗺️ Maps` para que los técnicos puedan abrir la ruta satelital directamente desde su celular.
   - Siempre debe estar disponible la opción de ingresar "+ Otro Domicilio" o escribir una dirección manual si fuera necesario.
3. **Control de Superposiciones Horarias de Técnicos:**
   - El sistema debe impedir asignar al mismo colaborador a dos servicios en el mismo día y rango horario.
4. **Franja Horaria de Almuerzo:**
   - El horario de **12:30 a 13:30 hs** se respeta como horario de almuerzo operativo.
5. **Ciclo de Estados:**
   - `Pendiente` (o `Sin Coordinar` si no tiene horario fijado) -> `En Proceso` -> `Completada` / `Reprogramar`.

---

## 5. Módulo de Planillas de Trabajo Digitales (PDT)

1. **Identificador del Documento:**
   - Formato estandarizado: `PDT-YYMMxxx` (Año + Mes + Correlativo, ej. `PDT-2611014`).
2. **Bloqueo Estricto Anti-Duplicados (Doble Clic):**
   - Una vez presionado el botón "Enviar Planilla" o "Guardar Borrador", el sistema bloquea inmediatamente los botones con la leyenda `Enviando...` y activa un semáforo de guardado (`isSavingPlanilla`).
   - El sistema rechaza cualquier intento de registrar una planilla con el mismo cliente, fecha, horario y domicilio en un lapso de 15 segundos para evitar registros gemelos.
3. **Contenido Obligatorio del Informe Técnico:**
   - Detalle de mano de obra y horas trabajadas por colaborador.
   - Materiales e insumos utilizados tomados del catálogo/inventario.
   - Áreas especializadas y equipos intervenidos (refrigeración, climatización, electricidad).
   - Firma digital del técnico o cliente y conformidad del trabajo.
4. **Vinculación con Órdenes:**
   - Al completar la planilla vinculada a una OT, la orden puede pasar automáticamente a estado de revisión o completada.

---

## 6. Roles y Permisos del Personal

- **Administrador:**
  - Control total sobre tarifas, precios, costos, liquidación de jornales y presupuestos.
  - Creación, edición y eliminación de clientes, colaboradores, órdenes y planillas (eliminaciones protegidas con clave `123`).
  - Asignación y reasignación de colaboradores a servicios.
  - Vista del mapa satelital global de la cuadrilla.
- **Colaborador / Técnico:**
  - Acceso restringido a sus servicios asignados en el día.
  - Fichaje de ingreso y egreso con GPS desde el celular.
  - Llenado, edición y firma de planillas de trabajo asignadas.
  - Visualización de domicilios de clientes con acceso directo a Google Maps para navegación.
