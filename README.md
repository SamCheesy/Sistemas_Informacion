# Tablero de retrasos · Soluciones & Partes

Tablero web estático (HTML + Chart.js) para las historias **HU-10** (tablero web de retrasos) y **HU-14** (análisis estadístico de retrasos). No necesita servidor ni base de datos: lee un archivo `alertas.csv`, que es el registro histórico de **HU-06**.

## Archivos

| Archivo | Para qué sirve |
|---|---|
| `index.html` | El tablero completo (estilos, lógica y gráficos). |
| `alertas.csv` | Registro de alertas. Hoy contiene **140 filas de ejemplo (datos ficticios)**. |

## Qué muestra

- **Filtros** (una sola fila, afectan todo lo de abajo): período (todo / 7 / 30 / 90 días / rango), tipo de alerta, asesor, proveedor y estado.
- **Indicadores:** alertas en el período, alertas abiertas, retraso promedio de las abiertas y tipo más frecuente.
- **Gráficos:** alertas por semana y por mes (separadas por tipo), alertas por proveedor y alertas por asesor. Cada gráfico tiene un botón **Ver tabla** con los mismos números.
- **Tabla de alertas abiertas** con reserva, equipo, asesor, proveedor, tipo y días de retraso, del mayor al menor retraso.
- Funciona en computador y celular, y respeta el modo claro/oscuro del dispositivo.

## Columnas de `alertas.csv`

La primera fila es el encabezado y los nombres deben ser exactamente estos. Acepta separador `,` o `;`.

| Columna | Ejemplo | Origen | Notas |
|---|---|---|---|
| `id_alerta` | `R-2026-1112` | HU-06 | Clave de la reserva o del equipo. **Una alerta = clave + tipo**: la misma clave puede tener alertas de tipos distintos y el tablero no las mezcla. |
| `fecha_alerta` | `2026-11-05` | HU-06 | Fecha en que se detectó por primera vez. Formato `AAAA-MM-DD` o `DD/MM/AAAA`. |
| `tipo_alerta` | `SLA de ensamble vencido` | HU-06 | Ver tipos abajo. |
| `reserva` | `R-2026-1112` | HU-06 | Vacío si el equipo no tiene reserva. |
| `equipo` | `Motobomba diésel 4 in #204` | HU-06 | |
| `asesor` | `Daniela Ochoa` | HU-06 | Vacío o `Sin asesor` si no hay reserva. |
| `dias_retraso` | `4` | HU-06 | Entero. Se **actualiza** cada día mientras la alerta siga activa. |
| `proveedor` | `Metalmecánica Andina` | **Agregar** | No está en HU-06, pero HU-14 lo necesita (retrasos por proveedor). Sale del Excel de control. |
| `ultima_actualizacion` | `2026-11-17` | **Agregar** | Fecha de la última ejecución del flujo que vio la alerta activa. Define si está abierta o cerrada. |

**Encabezados aceptados.** El tablero ignora tildes, mayúsculas y espacios, y reconoce los nombres que ya usan los flujos: `Clave` = `id_alerta`, `Tipo` = `tipo_alerta`, `Fecha Alerta` = `fecha_alerta`, `UltimaActualización` = `ultima_actualizacion`, `Días Retraso` = `dias_retraso`. Para Power Automate conviene renombrar las columnas del Excel a los nombres de la tabla (sin espacios ni tildes): el conector los codifica (`Fecha_x0020_Alerta`) y complica los filtros. Si prefieren dejarlos, se mapean en el paso *Create CSV table* con columnas personalizadas.

**Estado de la alerta.** HU-06 no guarda el estado, así que el tablero lo calcula: una alerta está **abierta** si su `ultima_actualizacion` es la más reciente de todo el registro (la última ejecución del flujo la vio activa) y **cerrada** si dejó de actualizarse. Si el registro trae una columna `estado` (`Abierta`/`Cerrada`), se usa esa.

**Tipos de alerta con color fijo:** `Sin envío a ensamble > 8 días` (HU-02, nombre real), `SLA de ensamble vencido`, `Equipo en producción sin reserva`, `Listo para facturar sin factura`, `Fila sin actualizar`. Los nombres de HU-03, 04, 07 y 12 son provisionales: deben coincidir exactamente con el texto que escriba cada flujo. Los tipos que no estén en la lista se agregan al final; solo hay 5 colores validados, así que con más de 5 tipos el sexto repite color.

**Defensas del tablero:** si el flujo repite una misma alerta en varias filas, cuenta solo la más reciente de cada `id_alerta` y avisa. Si falta `proveedor`, el gráfico de proveedores lo indica. Si falta `ultima_actualizacion` y `estado`, muestra todo como abierto y avisa. Una columna `etapa` es opcional: si existe aparece en la tabla.

## Qué debe hacer el flujo de HU-06

1. Tomar la clave de la alerta (reserva, o equipo cuando no hay reserva).
2. Buscar en el registro una fila con **esa clave y ese tipo** (dos filtros, no solo la clave).
3. Si **no existe**: insertar fila con `fecha_alerta` = hoy, `dias_retraso` y `ultima_actualizacion` = hoy.
4. Si **ya existe**: actualizar solo `dias_retraso` y `ultima_actualizacion` (no tocar `fecha_alerta` y no borrar nada).
5. Incluir `proveedor` tomado del Excel de control.

> Con la hoja de Google Sheets, columnas de fecha: **Formato → Número → Fecha personalizada → `aaaa-mm-dd`**, o dejarlas como texto. El tablero también entiende `DD/MM/AAAA`.

## Publicar gratis en GitHub Pages

1. Crear un repositorio en GitHub (o usar el del proyecto) y subir `index.html` y `alertas.csv` a la raíz.
2. En el repositorio: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, rama `main`, carpeta `/ (root)` y **Save**.
3. Esperar uno o dos minutos. El tablero queda en `https://<usuario>.github.io/<repositorio>/`.

**Importante:** en el plan gratuito de GitHub, Pages solo funciona con repositorios **públicos**, así que `alertas.csv` también sería público. Con datos reales de la empresa (reservas, asesores, proveedores) hay que anonimizarlos o acordar con el cliente qué se puede publicar.

## Pasar de datos de ejemplo a datos reales

1. Reemplazar `alertas.csv` por el registro real, con las columnas de arriba (o cambiar `CSV_URL` en `index.html` por el enlace del CSV publicado de Google Sheets).
2. En `index.html`, cambiar `var DATOS_DE_EJEMPLO = true;` por `false`.
3. Recargar la página para ver los datos nuevos.

## Actualización automática (Power Automate → GitHub Actions → GitHub Pages)

Evita subir el CSV a mano. Usa solo conectores estándar y servicios gratuitos.

**En el repositorio:** subir `.github/workflows/actualizar-alertas.yml` (respetando esa ruta). Ese flujo de GitHub Actions recibe el CSV, valida que empiece por `id_alerta` y reemplaza `alertas.csv`. Pages se republica con el cambio.

**En Power Automate**, un flujo nuevo «Publicar tablero» (programado después de los flujos de alerta, por ejemplo a las 8:00 a. m.):
1. **Recurrence**, una vez al día.
2. **List rows present in a table** sobre `Registro_Alertas`. En ⋯ → Settings activar **Pagination** (umbral 5000); por defecto devuelve solo 256 filas. Dejar las columnas de fecha del Excel con formato **Texto** (si no, Excel entrega números de serie en vez de fechas).
3. **Create CSV table**, *From* = `value` del paso anterior, *Columns* = **Custom** con las 9 columnas (`id_alerta`, `fecha_alerta`, `tipo_alerta`, `reserva`, `equipo`, `asesor`, `proveedor`, `dias_retraso`, `ultima_actualizacion`). Sin esto se cuelan columnas internas de Excel.
4. **Create a repository dispatch event** (conector GitHub): propietario y repositorio, *Event Name* = `actualizar-alertas`, *Event Payload* = `{"csv_b64": "@{base64(body('Create_CSV_table'))}"}`. El base64 evita que comillas, tildes y saltos de línea rompan el JSON.

**Límites:** GitHub no publica el tamaño máximo de `client_payload` (puede responder «client_payload is too large»). Con unas 140 alertas el mensaje pesa ~25 KB. Si el registro crece mucho, enviar solo las alertas de los últimos 90 días. Un error del flujo se ve en la pestaña **Actions** del repositorio. La acción de GitHub aparece como *Preview* en Power Automate.

## Probar en local

Abrir `index.html` con doble clic funciona: si el navegador no puede leer `alertas.csv`, la página usa una copia de los datos de ejemplo incluida en el mismo archivo (y lo avisa). Para probar con otro CSV, usar **Cargar otro archivo de alertas (CSV)** al final de la página. Los gráficos necesitan internet, porque Chart.js se carga desde un CDN.

## Límites conocidos

- La página no se actualiza sola: depende de que `alertas.csv` (o la hoja publicada) se actualice.
- Una alerta se considera cerrada cuando deja de actualizarse. Si el flujo falla un día, todas las alertas de ese día quedarían como cerradas hasta la siguiente ejecución.
- El filtro de período usa `fecha_alerta` (primera detección): una alerta abierta desde septiembre no aparece en «últimos 7 días».
- No tiene control de acceso: quien tenga el enlace puede ver el tablero.
