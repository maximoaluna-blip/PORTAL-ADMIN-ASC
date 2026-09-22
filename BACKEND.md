# Backend — Portal Administrativo ASC

> Este portal **no tiene backend propio**. Es una interfaz que consume el Apps Script de cada línea formativa.
>
> Este documento explica cómo está conectado y cómo cambiar la URL si fuera necesario.

---

## 1. Identificadores clave (heredados del backend compartido)

Durante el piloto, todas las líneas formativas comparten **un solo Apps Script + un solo Google Sheet**:

| Campo | Valor |
|---|---|
| **PROD_SCRIPT_ID** | `1TTJ2VjNta0Vz4p6gAjwvsXggN8g8YfV-FrZuQtWvnUy0ZFRrYA-gCrqe` |
| **PROD_DEPLOYMENT_URL** | `https://script.google.com/macros/s/AKfycbxxZBp6XpmdRzZS0BXO02WMq31K5FUU8-Mqzc2Sj0PcwB3cMcrhIqbHQA0naUQb5mgBWw/exec` |
| **AUTH_TOKEN** | `ADULTOS_ASC_2026` |
| **Editor del script** | https://script.google.com/u/0/home/projects/1TTJ2VjNta0Vz4p6gAjwvsXggN8g8YfV-FrZuQtWvnUy0ZFRrYA-gCrqe/edit |
| **Repo dueño del código del backend** | [`INDUCCION-ADULTOS`](https://github.com/maximoaluna-blip/INDUCCION-ADULTOS) (mirar su `BACKEND.md` y `05-Generador-Cursos/google-apps-script.js`) |

---

## 2. Cómo se conecta este portal al backend

El usuario administrador, al abrir el dashboard por primera vez, debe **introducir manualmente la URL del Apps Script**. La URL queda guardada en `localStorage` del navegador (`dashboard_gas_url`) para futuras visitas.

```
dashboard.html
  → en init: lee localStorage['dashboard_gas_url'] y localStorage['dashboard_admin_key']
  → si no hay URL: muestra pantalla de "Conectar"
  → con clave:  POST <url>  {token, action:'stats', adminKey}   → agregados + detalle
  → sin clave:  GET  <url>?action=stats                          → solo agregados
```

Desde el **ADR-078** la clave de administración es **opcional**: sin ella el panel funciona
y muestra los totales, y las tablas de personas quedan vacías **diciendo por qué**. Si el
backend la rechaza, el panel reintenta sin ella y lo avisa, en vez de quedarse en blanco.

⚠️ La clave se guarda **solo en el navegador de quien administra** y va por **POST**, nunca
en la URL. Este panel es una página estática y pública: hornear la clave en su HTML habría
sido dejarla a la vista de cualquiera que lea su JavaScript.

El endpoint devuelve los datos agregados + arrays detallados, y el dashboard:

1. Carga `dashboards.json` para conocer los `courseIds` de cada línea.
2. Si hay `?linea=X` en el querystring, filtra los registros, certificados y módulos a esa línea.
3. Renderiza KPIs, tablas y gráfico.

---

## 3. Filtrado por línea

Cada línea tiene sus `courseIds` declarados en [`dashboards.json`](dashboards.json):

```json
{
  "id": "politica-adultos",
  "courseIds": [
    "bienvenida-adultos",
    "politica-marco",
    "ciclo-adulto",
    "competencias-esenciales",
    "plan-personal"
  ]
}
```

El dashboard al filtrar por línea recorre los `registros[]` del backend y descarta los que no estén en `courseIds`. Igual con certificados y módulos.

> **Cuando se agregue un curso nuevo a una línea**, hay que agregar su `courseId` al `dashboards.json` y hacer push.

---

## 4. Endpoint que consume el dashboard

```
GET <PROD_DEPLOYMENT_URL>?action=stats
```

Respuesta esperada (parche `handleStats con arrays detallados`):

```json
{
  "success": true,
  "data": {
    "totalUsers": 6,
    "totalCertificates": 2,
    "totalQuizzes": 12,
    "completionsByModule": { ... },
    "courseStats": { ... },
    "averageScore": 100,
    "registros": [ { fecha, nombre, grupo, region, email, curso, estado }, ... ],
    "certificados": [ { fecha, nombre, curso, grupo, region, codigo, puntuacion, email }, ... ],
    "modulos": [ { curso, modulo, nombre, completados }, ... ],
    "resumen": {
      "totalRovers": 6,
      "totalCertificados": 2,
      "tasaCompletacion": 33,
      "promedioPuntuacion": 100
    },
    "generatedAt": "2026-05-17T..."
  }
}
```

Si el endpoint NO devuelve los arrays detallados (`registros`, `certificados`, `modulos`, `resumen`), el dashboard solo mostrará los KPIs y dejará las tablas y el gráfico vacíos. **Eso indica que el deployment del Apps Script tiene código viejo.**

> ⚠️ **`totalCommitments` ya no existe en el payload** (hallazgo C4 de la auditoría del 20-sep-2026). El campo se publicaba y era **0 estructural**: el compromiso de cierre de cada curso se guarda solo en `localStorage` del navegador y ningún curso envía `action=commitment`, así que la métrica afirmaba un dato que nadie alimentaba. El dashboard **nunca la pintó** —sus cuatro KPIs salen de `resumen`—, así que retirarla no cambia nada de lo que se ve.

---

## 5. Si la URL del backend cambia

Si en el futuro cambia el deployment del Apps Script (ej. se separan las líneas, se crea un deployment nuevo por seguridad, etc.):

1. Actualizar `PROD_DEPLOYMENT_URL` en este archivo.
2. El admin debe abrir el dashboard, hacer "Desconectar" y pegar la URL nueva. No se requiere recompilar nada de este portal.
3. Si se quiere precargar la URL nueva, modificar `dashboard.html` para que el campo "URL del Apps Script" tenga el `value` por defecto apuntando a la nueva URL.

---

## 6. Cambios en `dashboards.json`

Cuando se agrega/modifica una línea:

| Cambio | Acción |
|---|---|
| Curso nuevo en línea existente | Agregar el `courseId` al array `courseIds` de esa línea. |
| Nueva línea formativa | Agregar un objeto nuevo con `id`, `name`, `icon`, `color`, `url`, `courseIds`. |
| Curso cambia de línea | Quitar del `courseIds` de la línea vieja, agregar al de la nueva. |

Cualquier cambio a `dashboards.json` se refleja en el dashboard al siguiente refresh (sin recompilación).

---

## 7. Seguridad

**Cerrado el 21-sep-2026 — ADR-078.** `?action=stats` entregaba `registros[]` con nombre,
correo, grupo, región y curso de **todas** las personas inscritas a quien tuviera la URL del
deployment — y esa URL viaja en el HTML publicado de los 32 cursos. Hoy el GET público
devuelve **solo agregados**; el detalle va por **POST** con `adminKey`.

⚠️ **La mejora que este documento proponía durante meses no habría cerrado nada.** Decía
«validar `params.token === AUTH_TOKEN`», y `AUTH_TOKEN` está impreso en el HTML de cada
curso publicado: cualquiera que abra el código fuente de una página lo tiene. Por eso la
clave nueva es **otra**, y vive donde el público no llega.

**Dónde vive `ADMIN_KEY`:** en el editor de Apps Script → *Configuración del proyecto* →
*Propiedades del script*. No está en ningún repositorio. Si no se configura, el detalle
**no se sirve**: el código falla cerrado, así que desplegarlo cierra la fuga aunque después
no se configure nada.

**Lo que esto SÍ hace:** deja de entregar el padrón a quien solo tiene una URL pública.
**Lo que NO hace:** autenticación de verdad. Quien tenga la clave la tiene; no hay usuarios,
ni caducidad, ni registro de quién consultó. Eso es otra decisión, y más grande.

**Rotar la clave:** cambiar el valor en las propiedades del script. Cada admin tendrá que
reconectar el panel con la nueva; las anteriores dejan de funcionar al instante.

**Cómo se comprueba:**

1. `node probar-stats.js` (en `INDUCCION-ADULTOS/05-Generador-Cursos/`) — 22 comprobaciones
   locales, sin red: el GET público no puede traer un nombre, un correo, un grupo ni un
   código de certificado.
2. `node verificar-backend.js` — su Paso 4 llama al endpoint de producción **sin clave** y
   falla si vuelve con gente dentro.

---

## 8. Documentación cruzada

- [`INDUCCION-ADULTOS/BACKEND.md`](https://github.com/maximoaluna-blip/INDUCCION-ADULTOS/blob/main/BACKEND.md) — fuente de verdad del backend.
- [`INDUCCION-DESARROLLO-INSTITUCIONAL/BACKEND.md`](https://github.com/maximoaluna-blip/INDUCCION-DESARROLLO-INSTITUCIONAL/blob/main/BACKEND.md) — referencia desde la línea DI.
- [`PORTAL-ADULTOS-ASC/ARQUITECTURA.md`](https://github.com/maximoaluna-blip/PORTAL-ADULTOS-ASC/blob/main/ARQUITECTURA.md) — vista panorámica de cómo se conectan los 4 repos.

---

_Documento operativo del Portal Administrativo. Heredado del backend compartido durante el piloto._
