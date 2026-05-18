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
  → en init: lee localStorage['dashboard_gas_url']
  → si está vacío: muestra pantalla de "Conectar"
  → si existe: hace GET <url>?action=stats
```

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
    "totalCommitments": 0,
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

⚠️ **Punto a mejorar:** el endpoint `?action=stats` actualmente es público (sin token de auth). Cualquiera con la URL podría leer los registros (nombres, emails, grupos).

**Mitigación actual:** la URL no está indexada ni difundida públicamente, solo se comparte entre admins.

**Mejora futura sugerida:** agregar validación de token al GET en el Apps Script (similar al que ya tiene el POST). Cuando se haga:

1. Modificar `handleStats()` en `google-apps-script.js` para validar `params.token === AUTH_TOKEN`.
2. Modificar `dashboard.html` para pedir el token junto a la URL al conectar (guardarlo en localStorage también).
3. Push y verificar con `verificar-backend.js`.

---

## 8. Documentación cruzada

- [`INDUCCION-ADULTOS/BACKEND.md`](https://github.com/maximoaluna-blip/INDUCCION-ADULTOS/blob/main/BACKEND.md) — fuente de verdad del backend.
- [`INDUCCION-DESARROLLO-INSTITUCIONAL/BACKEND.md`](https://github.com/maximoaluna-blip/INDUCCION-DESARROLLO-INSTITUCIONAL/blob/main/BACKEND.md) — referencia desde la línea DI.
- [`PORTAL-ADULTOS-ASC/ARQUITECTURA.md`](https://github.com/maximoaluna-blip/PORTAL-ADULTOS-ASC/blob/main/ARQUITECTURA.md) — vista panorámica de cómo se conectan los 4 repos.

---

_Documento operativo del Portal Administrativo. Heredado del backend compartido durante el piloto._
