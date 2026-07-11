# Portal Administrativo · ASC

Landing unificada de acceso a los paneles administrativos de las plataformas formativas de la Asociación Scouts de Colombia.

🔐 **Producción:** https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/

## Paneles incluidos

| Línea | Estado | Dashboard |
|---|---|---|
| 📜 Política de Adultos en el Movimiento | Activo | [Abrir](https://maximoaluna-blip.github.io/INDUCCION-ADULTOS/dashboard-admin.html) |
| 🏛️ Desarrollo Institucional | Activo (piloto) | [Abrir](https://maximoaluna-blip.github.io/INDUCCION-DESARROLLO-INSTITUCIONAL/02-Plataforma-Web/dashboard-admin.html) |
| 🎒 Programa de Jóvenes | Activo (Nivel 1, 7 cursos) | [Abrir](https://maximoaluna-blip.github.io/PORTAL-ADMIN-ASC/dashboard.html?linea=programa-jovenes) |
| 🛡️ Políticas Transversales | Próximamente | — |

## Notas de seguridad

- Cada dashboard requiere la URL del Google Apps Script (se ingresa una vez y queda guardada en localStorage del navegador).
- El backend de las líneas activas está compartido durante el piloto (mismo token `ADULTOS_ASC_2026`). Los registros se diferencian por `courseId`.
- Idealmente añadir token de auth en el GET `?action=stats` antes de exposición pública.

## Mantenimiento

Para agregar o quitar paneles, editar `dashboards.json` y hacer commit.

---

© 2026 Asociación Scouts de Colombia
