# Portal Administrativo · ASC

**Panel administrativo unificado** de las plataformas formativas de la Asociación Scouts de Colombia — indicadores, registros y certificados por línea, sobre el backend compartido.

🔐 **Producción:** https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/

Es el contrapunto de [`PORTAL-ADULTOS-ASC`](https://github.com/maximoaluna-blip/PORTAL-ADULTOS-ASC): ese portal es para el **estudiante** (elegir en qué línea formarse), este es para **administración** (ver quién se inscribió, qué certificó, cómo va cada línea). Comparten backend, pero cada uno lee su propio catálogo (`lineas.json` allá, `dashboards.json` aquí) — mantenerlos sincronizados es manual.

## ¿Qué es esto?

Una landing con una tarjeta por línea formativa, más una **vista global** (`dashboard.html`) que agrega los indicadores de todas. Cada tarjeta enlaza al panel filtrado de esa línea (`dashboard.html?linea=<id>`). No tiene backend propio: consume el mismo Apps Script + Sheet que usan las 3 líneas de cursos — ver [`BACKEND.md`](BACKEND.md) para el detalle completo del contrato, el endpoint `?action=stats` y cómo filtra por `courseIds`.

## Paneles incluidos

| Línea | Estado | Dashboard |
|---|---|---|
| 📜 Política de Adultos en el Movimiento | Activo · 5 cursos | [Abrir](https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/dashboard.html?linea=politica-adultos) |
| 🏛️ Desarrollo Institucional | Activo · 6 cursos (Nivel 1 completo) | [Abrir](https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/dashboard.html?linea=desarrollo-institucional) |
| 🎒 Programa de Jóvenes | Activo · 8 cursos (Nivel 1 completo + Nivel 2 en marcha) | [Abrir](https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/dashboard.html?linea=programa-jovenes) |
| 🛡️ Políticas Transversales | Próximamente | — |

> La fuente de verdad de esta tabla es [`dashboards.json`](dashboards.json) — específicamente `coursesActive` y `courseIds` por línea. Si editas el JSON, actualiza también esta tabla. **`courseIds` debe coincidir exactamente** con los `courseId` reales del `cursos.json` de cada línea — un desajuste ahí hace que el dashboard filtre datos reales como si no existieran (pasaba con Desarrollo Institucional hasta el 12-jul-2026: los 6 `courseIds` no coincidían con los publicados).

## Estructura del repo

```
PORTAL-ADMIN-ASC/
├── index.html          ← Landing con tarjetas por línea
├── dashboard.html       ← Panel (global o filtrado por ?linea=)
├── dashboards.json      ← Catálogo de líneas (courseIds, colores, estado)
├── assets/
│   ├── logo-asc.png
│   ├── logo-vallescout.png
│   ├── favicon.svg
│   └── theme-toggle.js
├── README.md            ← Este archivo
└── BACKEND.md            ← Contrato con el Apps Script compartido, endpoint, seguridad
```

## ¿Cómo agregar o actualizar una línea?

1. Confirmar los `courseId` reales publicados en `02-Plataforma-Web/cursos.json` de esa línea (no adivinarlos ni copiarlos de la propuesta original — pueden haber cambiado durante la construcción).
2. Editar `dashboards.json`: `coursesActive`, `courseIds` (la lista completa y exacta), `audience`, y `status`/`url` si la línea recién se activa.
3. Actualizar la tabla "Paneles incluidos" de este README para que coincida.
4. Commit + push. El dashboard lee el JSON en cada carga, sin recompilación.

## Notas de seguridad

- Cada dashboard requiere la URL del Google Apps Script (se ingresa una vez y queda guardada en `localStorage` del navegador).
- El backend de las líneas activas está compartido durante el piloto (mismo token `ADULTOS_ASC_2026`). Los registros se diferencian por `courseId` — por eso un `courseIds` desactualizado en `dashboards.json` es un bug silencioso, no solo cosmético.
- El endpoint `?action=stats` hoy es público (sin token). Mitigación y mejora futura sugerida en `BACKEND.md` §7.

---

© 2026 Asociación Scouts de Colombia
