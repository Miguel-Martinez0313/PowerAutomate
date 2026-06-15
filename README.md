# 🎫 Manual: Sistema de Tickets Helpdesk con Office 365 y Power Apps

> **Basado en el tutorial de [Christiano T.S.](https://www.youtube.com/@christianoTS)**  
> Video original: [Ver en YouTube](https://www.youtube.com/watch?v=2NRIp8fdc_w)  
> Duración aproximada: 1 hora 15 minutos

---

## 🧩 Herramientas necesarias

| Herramienta | Función en el sistema | Acceso gratuito |
|---|---|---|
| [Microsoft Forms](https://forms.office.com) | Formulario de solicitud para usuarios | ✅ Incluido en Office 365 |
| [Power Automate](https://make.powerautomate.com) | Automatización de flujos | ✅ Incluido en Office 365 |
| [SharePoint](https://www.microsoft.com/es-mx/microsoft-365/sharepoint/collaboration) | Base de datos de tickets | ✅ Incluido en Office 365 |
| [Power Apps](https://make.powerapps.com) | App para técnicos | ✅ Incluido en Office 365 |

> 💡 **¿No tienes Office 365?** Puedes crear una cuenta de prueba gratuita de 30 días en [microsoft.com/es-mx/microsoft-365/try](https://www.microsoft.com/es-mx/microsoft-365/try)

---

## 🏗️ Arquitectura del sistema

```
Usuario llena el Form
        ↓
Power Automate guarda la respuesta en SharePoint
        ↓
SharePoint almacena el ticket (Estado: "En proceso")
        ↓
Se envía email automático al usuario y al técnico
        ↓
Técnico abre la Power App, atiende el ticket y escribe la solución
        ↓
Power Automate detecta el cambio de estado y envía email de cierre al usuario
```

---

## Fase 1 — Crear el formulario de solicitud

**Herramienta:** [Microsoft Forms](https://docs.google.com/forms/u/2/?pli=1)

### Paso 1.1 — Acceder a Microsoft Forms

1. Ir a [forms.office.com](https://forms.office.com) con tu cuenta de Office 365.
2. Hacer clic en **+ Nuevo formulario**.
3. Darle un nombre descriptivo, por ejemplo: `Solicitud de Soporte Técnico`.

### Paso 1.2 — Agregar los campos del formulario

Agregar las siguientes preguntas haciendo clic en **+ Agregar nuevo**:

| Campo | Tipo | ¿Obligatorio? |
|---|---|---|
| Nombre | Texto | ✅ Sí |
| Apellido | Texto | ✅ Sí |
| Correo electrónico | Texto | ✅ Sí |
| Área | Opción múltiple | ✅ Sí |
| Descripción del problema | Texto (largo) | ✅ Sí |

Para el campo **Área**, agregar las opciones que correspondan a tu empresa, por ejemplo:
- Marketing
- Logística
- Recursos Humanos
- Tecnología

### Paso 1.3 — Configurar los permisos del formulario

1. Hacer clic en los **tres puntos (...)** en la esquina superior derecha → **Configuración**.
2. Ajustar según el caso de uso:
   - Si quieres que **cualquier persona** (sin cuenta Office 365) pueda enviar tickets → activar **"Cualquier persona puede responder"**.
   - Si los usuarios necesitan enviar más de una solicitud → desactivar **"Solo una respuesta por persona"**.

---

## Fase 2 — Crear la lista en SharePoint

**Herramienta:** [SharePoint](https://sharepoint.com)

### Paso 2.1 — Acceder al sitio de SharePoint

1. Ir a [sharepoint.com](https://sharepoint.com) y abrir el sitio de tu equipo o empresa.
2. Si no tienes un sitio, crear uno: **+ Crear sitio** → Sitio de equipo.

### Paso 2.2 — Crear una nueva lista

1. Dentro del sitio, ir a **Contenido del sitio** → **+ Nuevo** → **Lista**.
2. Seleccionar **Lista en blanco**.
3. Nombrarla: `Tickets_HelpDesk` y hacer clic en **Crear**.

### Paso 2.3 — Agregar columnas a la lista

Dentro de la lista, hacer clic en **+ Agregar columna** para crear cada una de las siguientes:

| Nombre de columna | Tipo de columna |
|---|---|
| Nombre | Texto de una línea |
| Apellido | Texto de una línea |
| Correo | Texto de una línea |
| Área | Opción (agregar las mismas opciones que el formulario) |
| Descripción | Texto de varias líneas |
| Estado_Ticket | Opción: `En proceso`, `Atendido`, `Resuelto` |
| Solución | Texto de varias líneas |

### Paso 2.4 — Ocultar la columna "Título"

La columna **Título** se crea por defecto y no la necesitamos. Para ocultarla:

1. Hacer clic en el encabezado de la columna **Título**.
2. Seleccionar **Configuración de columna** → **Ocultar**.

---

## Fase 3 — Flujo 1: Guardar respuestas del formulario en SharePoint

**Herramienta:** [Power Automate](https://make.powerautomate.com)

### Paso 3.1 — Crear un nuevo flujo automatizado

1. Ir a [make.powerautomate.com](https://make.powerautomate.com).
2. Hacer clic en **+ Crear** → **Flujo de nube automatizado**.
3. Darle un nombre: `Flujo_Guardar_Tickets`.
4. En el campo de búsqueda del desencadenador, escribir `Microsoft Forms`.
5. Seleccionar: **"Cuando se envía una respuesta nueva"**.
6. Hacer clic en **Crear**.

### Paso 3.2 — Configurar el desencadenador

1. En el paso del desencadenador, abrir el campo **Formulario** y seleccionar el formulario creado en la Fase 1.

### Paso 3.3 — Agregar acción: Obtener detalles de la respuesta

1. Hacer clic en **+ Nuevo paso**.
2. Buscar `Microsoft Forms` → seleccionar **"Obtener detalles de respuesta"**.
3. Configurar:
   - **Formulario**: seleccionar el mismo formulario.
   - **Id. de respuesta**: hacer clic en el campo → seleccionar el contenido dinámico **"Id. de respuesta"**.

### Paso 3.4 — Agregar acción: Crear elemento en SharePoint

1. Hacer clic en **+ Nuevo paso**.
2. Buscar `SharePoint` → seleccionar **"Crear elemento"**.
3. Configurar:
   - **Dirección del sitio**: seleccionar tu sitio de SharePoint.
   - **Nombre de lista**: seleccionar `Tickets_HelpDesk`.
4. Mapear cada campo usando contenido dinámico:

| Campo en SharePoint | Valor dinámico desde Forms |
|---|---|
| Nombre | Respuesta de la pregunta "Nombre" |
| Apellido | Respuesta de la pregunta "Apellido" |
| Correo | Respuesta de la pregunta "Correo electrónico" |
| Área | Respuesta de la pregunta "Área" |
| Descripción | Respuesta de la pregunta "Descripción del problema" |
| Estado_Ticket | Escribir el valor fijo: `En proceso` |

### Paso 3.5 — Guardar y probar el flujo

1. Hacer clic en **Guardar**.
2. Abrir el formulario y enviar una respuesta de prueba.
3. Ir a la lista de SharePoint y verificar que el nuevo ticket aparece correctamente.

---

## Fase 4 — Flujo 2: Enviar notificaciones por email al crear un ticket

**Herramienta:** [Power Automate](https://make.powerautomate.com)

> Agregar estos pasos al mismo flujo de la Fase 3, después del paso "Crear elemento".

### Paso 4.1 — Email de confirmación al usuario

1. Hacer clic en **+ Nuevo paso**.
2. Buscar `Office 365 Outlook` → seleccionar **"Enviar un correo electrónico (V2)"**.
3. Configurar:

```
Para:      [Correo electrónico] → contenido dinámico del campo Correo
Asunto:    Tu solicitud de soporte ha sido registrada
Cuerpo:    Hola [Nombre],

           Hemos recibido tu solicitud de soporte técnico.
           
           Descripción: [Descripción]
           
           Nuestro equipo te contactará a la brevedad.
           
           Saludos,
           Equipo de Soporte
```

### Paso 4.2 — Email de notificación al técnico

1. Agregar otro paso **"Enviar un correo electrónico (V2)"**.
2. Configurar:

```
Para:      correo-del-tecnico@tuempresa.com  (valor fijo)
Asunto:    Nuevo ticket de soporte — [Nombre] [Apellido]
Cuerpo:    Se ha registrado un nuevo ticket de soporte.
           
           Usuario: [Nombre] [Apellido]
           Correo:  [Correo]
           Área:    [Área]
           
           Descripción:
           [Descripción]
           
           Ingresa a la aplicación HelpDesk para atenderlo.
```

3. Hacer clic en **Guardar**.

---

## Fase 5 — Crear la aplicación en Power Apps

**Herramienta:** [Power Apps](https://make.powerapps.com)

### Paso 5.1 — Generar la app desde SharePoint

1. Ir a la lista `Tickets_HelpDesk` en SharePoint.
2. En el menú superior, hacer clic en **Integrar** → **Power Apps** → **Crear una aplicación**.
3. Dar un nombre a la app: `HelpDesk App`.
4. Hacer clic en **Crear**.

> Power Apps abrirá automáticamente el editor con una aplicación de 3 pantallas generada a partir de la lista de SharePoint.

### Paso 5.2 — Entender las 3 pantallas generadas

| Pantalla | Nombre | Función |
|---|---|---|
| 1 | BrowseScreen | Galería con todos los tickets |
| 2 | DetailScreen | Detalle de un ticket seleccionado |
| 3 | EditScreen | Formulario para editar el ticket |

### Paso 5.3 — Personalizar la galería (BrowseScreen)

1. En la pantalla **BrowseScreen**, seleccionar la galería de tickets.
2. En el panel derecho, cambiar los campos mostrados:
   - **Título** → columna `Nombre`
   - **Subtítulo** → columna `Correo` o `Estado_Ticket`
3. Esto hace que la lista sea más informativa para el técnico.

### Paso 5.4 — Verificar la pantalla de edición (EditScreen)

1. Ir a la pantalla **EditScreen**.
2. Confirmar que los campos `Estado_Ticket` y `Solución` están presentes y son editables.
3. El técnico usará esta pantalla para:
   - Cambiar el estado a `Atendido` o `Resuelto`.
   - Escribir la solución aplicada.

### Paso 5.5 — Guardar y publicar la aplicación

1. Hacer clic en **Archivo** → **Guardar**.
2. Hacer clic en **Publicar** → **Publicar esta versión**.
3. Para compartir con los técnicos: **Archivo** → **Compartir** → ingresar los correos del equipo de soporte.

---

## Fase 6 — Flujo 3: Notificar al usuario cuando el ticket es atendido

**Herramienta:** [Power Automate](https://make.powerautomate.com)

### Paso 6.1 — Crear un nuevo flujo

1. Ir a [make.powerautomate.com](https://make.powerautomate.com) → **+ Crear** → **Flujo de nube automatizado**.
2. Nombre: `Flujo_Notificar_Cierre_Ticket`.
3. Desencadenador: buscar `SharePoint` → seleccionar **"Cuando se crea o modifica un elemento"**.
4. Configurar:
   - **Dirección del sitio**: tu sitio de SharePoint.
   - **Nombre de lista**: `Tickets_HelpDesk`.

### Paso 6.2 — Agregar condición

1. Hacer clic en **+ Nuevo paso** → buscar **Condición**.
2. Configurar la condición:
   - Campo izquierdo: contenido dinámico **`Estado_Ticket`**
   - Operador: **es igual a**
   - Campo derecho: escribir `Atendido`

### Paso 6.3 — Rama "Sí": enviar email al usuario con la solución

Dentro de la rama **"En caso afirmativo (Sí)"**:

1. Agregar acción: **"Enviar un correo electrónico (V2)"**.
2. Configurar:

```
Para:      [Correo] → contenido dinámico de SharePoint
Asunto:    Tu ticket de soporte ha sido resuelto
Cuerpo:    Hola [Nombre],

           Tu solicitud de soporte ha sido atendida.
           
           Solución aplicada:
           [Solución] → contenido dinámico de SharePoint
           
           Si tienes alguna otra duda, no dudes en crear un nuevo ticket.
           
           Saludos,
           Equipo de Soporte
```

### Paso 6.4 — Guardar y probar el flujo completo

1. Hacer clic en **Guardar**.
2. Abrir la **HelpDesk App**, seleccionar un ticket y cambiar el estado a `Atendido`, escribir una solución y guardar.
3. Verificar que el usuario recibe el email con la solución.

---

## ✅ Resultado final — Funcionalidades del sistema

- [x] Formulario público de solicitud de soporte
- [x] Registro automático de tickets en SharePoint
- [x] Asignación automática del estado "En proceso"
- [x] Email de confirmación automático al usuario
- [x] Notificación automática al técnico de soporte
- [x] Aplicación móvil/web para gestión de tickets
- [x] Cambio de estado desde la aplicación (Power Apps)
- [x] Registro de la solución aplicada por el técnico
- [x] Email automático al usuario al cerrar el ticket

---

## 🔗 Enlaces de acceso directo a cada herramienta

| Herramienta | Enlace |
|---|---|
| Microsoft Forms | [forms.office.com](https://forms.office.com) |
| Power Automate | [make.powerautomate.com](https://make.powerautomate.com) |
| SharePoint | [sharepoint.com](https://sharepoint.com) |
| Power Apps | [make.powerapps.com](https://make.powerapps.com) |
| Office 365 (prueba gratuita) | [microsoft.com/es-mx/microsoft-365/try](https://www.microsoft.com/es-mx/microsoft-365/try) |
| Video tutorial original | [youtube.com/watch?v=2NRIp8fdc_w](https://www.youtube.com/watch?v=2NRIp8fdc_w) |

---

*Manual generado con base en el tutorial de Christiano T.S. — [@christianoTS](https://www.youtube.com/@christianoTS)*
