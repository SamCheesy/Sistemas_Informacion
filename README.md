# Tablero de retrasos · Soluciones & Partes

Tablero web que muestra las alertas de retraso de los equipos: cuántas hay, cuáles siguen abiertas, con qué proveedor y asesor se relacionan y cómo evolucionan por semana y por mes. Es una sola página estática (`index.html`, con Chart.js) que lee el archivo `alertas.csv`. No necesita servidor ni base de datos.

Tablero publicado: https://samcheesy.github.io/Sistemas_Informacion/

## Qué muestra

- **Filtros** por período (todo, 7, 30 o 90 días, o un rango), tipo de alerta, asesor, proveedor y estado. Afectan todo lo que aparece debajo.
- **Indicadores:** alertas en el período, alertas abiertas, retraso promedio de las abiertas y tipo de alerta más frecuente.
- **Gráficos:** alertas por semana y por mes, separadas por tipo; alertas por proveedor y por asesor. Cada gráfico tiene un botón **Ver tabla** con los mismos números.
- **Tabla de alertas abiertas**, ordenada del mayor al menor retraso, con clave, equipo, asesor, proveedor, tipo y días de retraso.
- Funciona en computador y celular, y respeta el modo claro u oscuro del dispositivo.

## Archivos

| Archivo | Para qué sirve |
|---|---|
| `index.html` | El tablero completo: estilos, lógica y gráficos. |
| `alertas.csv` | Registro de alertas que lee el tablero. Se reemplaza cada día. |
| `.github/workflows/actualizar-alertas.yml` | Flujo de GitHub Actions que recibe el CSV nuevo y lo guarda en el repositorio. |

## Columnas de `alertas.csv`

La primera fila es el encabezado. El separador puede ser coma o punto y coma.

| Columna | Ejemplo | Descripción |
|---|---|---|
| `id_alerta` | `9102` | Clave de la alerta: serial del equipo o número de reserva. Una alerta se identifica por clave más tipo, así que un mismo equipo puede tener alertas de tipos distintos sin que se mezclen. |
| `fecha_alerta` | `2026-11-05` | Fecha en que se detectó por primera vez. Se acepta `AAAA-MM-DD` o `DD/MM/AAAA`. |
| `ultima_actualizacion` | `2026-11-20` | Última fecha en que un flujo vio la alerta activa. Con ella el tablero decide si está abierta o cerrada. |
| `tipo_alerta` | `SLA de ensamble vencido` | Tipo de alerta. |
| `equipo` | `Motobomba diésel 4 in` | Descripción del equipo. |
| `asesor` | `Daniela Ochoa` | Asesor responsable. |
| `dias_retraso` | `4` | Días de retraso. Se actualiza mientras la alerta siga activa. |
| `proveedor` | `Metalmecánica Andina` | Proveedor relacionado con la alerta. |

La columna `reserva` es opcional. Si existe, la primera columna de la tabla muestra la reserva en lugar de `id_alerta`.

El tablero ignora tildes, mayúsculas y espacios en los encabezados, y reconoce los nombres del registro en Excel (`Clave`, `Tipo`, `Fecha Alerta`, `UltimaActualización`, `Días Retraso`). Las fechas que llegan como número de serie de Excel también se leen bien.

### Estado de una alerta

El registro no guarda el estado, así que el tablero lo calcula: una alerta está **abierta** si su `ultima_actualizacion` es la más reciente de todo el registro, es decir, si la última ejecución de los flujos todavía la vio activa. Si deja de actualizarse, pasa a **cerrada**. Si el CSV trae una columna `estado` (`Abierta` o `Cerrada`), se usa esa.

### Tipos de alerta

| Tipo | Qué detecta |
|---|---|
| `Sin envío a ensamble > 8 días` | Equipos que llevan más de 8 días desde la reserva sin enviarse a ensamble. |
| `SLA de ensamble vencido` | Equipos en ensamble con proveedor que superaron el SLA acordado. |
| `Equipo en producción sin reserva` | Equipos en ensamble, alistamiento o listos para facturar sin reserva o sin cliente. |

Cada tipo tiene un color fijo. Un tipo nuevo se agrega al final de la lista con el siguiente color disponible.

### Datos repetidos o incompletos

- Si una misma alerta aparece en varias filas, el tablero cuenta solo la más reciente y avisa.
- Si falta la columna `proveedor`, el gráfico de proveedores lo indica.
- Si faltan `ultima_actualizacion` y `estado`, todas las alertas se muestran como abiertas y se avisa.

## Cómo se actualiza

El registro vive en un Excel de Microsoft 365 (`TablaAlertas`) que llenan tres flujos de Power Automate. Cada mañana otro flujo, «Publicar tablero», convierte ese registro en CSV y avisa a GitHub. Desde ahí todo es automático:

1. **Power Automate** lee `TablaAlertas`, genera el CSV y envía un evento de repositorio (`actualizar-alertas`) con el contenido codificado en base64.
2. **GitHub Actions** recibe el evento, comprueba que el CSV empiece por `id_alerta` y reemplaza `alertas.csv` con un commit del usuario `alertas-bot`.
3. **GitHub Pages** vuelve a publicar el tablero con el cambio.

Si algo falla, el error aparece en la pestaña **Actions** del repositorio.

## Publicar en GitHub Pages

1. Subir `index.html` y `alertas.csv` a la raíz del repositorio, y el flujo en `.github/workflows/actualizar-alertas.yml`.
2. En **Settings → Pages → Build and deployment**, elegir **Deploy from a branch**, la rama `main` y la carpeta `/ (root)`.
3. Esperar uno o dos minutos. El tablero queda en `https://<usuario>.github.io/<repositorio>/`.

## Probar en local

Abrir `index.html` con doble clic funciona. Para probar con otro CSV, usar **Cargar otro archivo de alertas (CSV)** al final de la página. Los gráficos necesitan conexión a internet porque Chart.js se carga desde un CDN.

## Límites

- La información se actualiza una vez al día, no en tiempo real.
- Si un flujo falla un día, sus alertas aparecen como cerradas hasta la siguiente ejecución.
- El filtro de período usa `fecha_alerta` (primera detección): una alerta abierta desde hace meses no aparece en «últimos 7 días».
- El repositorio es público, así que cualquiera con el enlace puede ver el tablero y `alertas.csv`. Con datos reales de la empresa hay que anonimizarlos o acordar qué se publica.
- El tablero no tiene control de acceso.
