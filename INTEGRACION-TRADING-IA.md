# Integración con Trading IA - Trader Alerts

Este documento define cómo aprovechar este repositorio dentro del proyecto ChatGPT **Trading IA - Trader Alerts**, sin confundir las capacidades de ChatGPT con las de Claude Code/Desktop.

## Estado y alcance

- Este repositorio es un paquete de **11 skills para Claude**. Cada skill combina un `SKILL.md` con una herramienta Python.
- Los scripts requieren Python 3.8 o superior y usan la biblioteca estándar.
- Un proyecto de ChatGPT puede consultar instrucciones y archivos que se le proporcionen, pero **no ejecuta automáticamente** los scripts Python de este repositorio ni obtiene acceso a sus archivos locales.
- Para ejecución real desde ChatGPT hace falta un entorno externo autorizado (por ejemplo, un servicio/API propio o un flujo de automatización) que ejecute los scripts y devuelva resultados. La conexión GitHub por sí sola no es un entorno Python.
- Este fork es la copia de trabajo. El repositorio upstream de Jake Schincariol no debe modificarse.

## Decisión principal: un solo motor de alertas

El proyecto Trading IA - Trader Alerts ya tiene instrucciones de uso para `alerts.py` (comandos `clock`, `add`, `check`, `quote`, `list` y `reset`). Este repositorio incluye otro motor en `trader-alerts/alerts.py`.

**No ejecutar ambos motores sobre el mismo conjunto de alertas.** Antes de sustituir o fusionar código hay que comparar el script que utiliza actualmente el proyecto con `trader-alerts/alerts.py`, incluidos sus formatos JSON, estados, identificadores de reglas, calendario y pruebas. Hasta completar esa comparación, el motor actual del proyecto se considera el motor principal y este repositorio es una fuente de funcionalidades, no un reemplazo automático.

## Mapa funcional recomendado

| Skill | Uso dentro del flujo de Trading IA |
| --- | --- |
| `trader-scanner` | Descubrir configuraciones y construir una watchlist. |
| `trader-news` | Contexto de noticias y filings antes de evaluar una entrada. |
| `trader-entry` | Vigilar la confirmación de entrada; no enviar órdenes. |
| `trader-exit` | Definir stop, objetivo y hora límite antes de entrar. |
| `trader-timemachine` | Simular escenarios históricos; no tratar probabilidades como predicción. |
| `trader-budget` | Calcular tamaño a partir del presupuesto y pérdida máxima autorizados. |
| `trader-risk` | Bloquear propuestas que incumplan los límites configurados. |
| `trader-broker` | Preparar tickets; nunca enviar órdenes sin todas las aprobaciones exigidas. |
| `trader-alerts` | Alertas informativas de precio, stop, objetivo y cierre. Mantener un único motor activo. |
| `trader-analyst` | Resumir resultados y métricas fundamentales. |
| `trader-pelosi` | Consultar divulgaciones públicas de operaciones del Congreso. |

### Secuencia de análisis de una operación

```text
scanner -> news -> entry -> exit -> timemachine -> budget -> risk
                                                        |
                                             broker (aprobación explícita)
alerts: stop / target / hora límite (motor único)
```

Cada etapa debe informar de la fuente, la hora de consulta, los datos faltantes y cualquier limitación. Una coincidencia del scanner no es una recomendación; una simulación no predice el futuro; un PASS de riesgo no equivale a autorización para operar.

## Archivos compartidos y contratos

Las herramientas comparten archivos bajo `trader/`. Mantener los nombres y contratos existentes al integrar:

- `trader/watchlist.json`: símbolos y configuraciones detectadas.
- `trader/entry.json`: estado de confirmación de entrada.
- `trader/plan.json`: entrada, stop, objetivo y hora límite.
- `trader/orders.json`: propuestas de órdenes, no órdenes ejecutadas.
- `trader/config.json`: límites y preferencias de riesgo.
- `trader/prices.json`: cotizaciones con fuente y marca de tiempo.
- `trader/portfolio.json`: efectivo y posiciones.
- `trader/state/`: estado operativo, alertas, verificaciones y registro del broker.

No guardar claves, tokens, contraseñas ni secretos en GitHub, archivos de proyecto o mensajes de chat. Los datos de ejemplo no son configuración personal del usuario.

## Controles obligatorios

1. Mantener `paper: true` como valor predeterminado.
2. No habilitar trading real, margen, opciones, ventas en corto ni apalancamiento por inferencia.
3. No colocar, cancelar, modificar ni cerrar órdenes desde una alerta o tarea programada.
4. Una orden solo puede pasar al broker después de superar la verificación de riesgo para esa orden exacta y recibir autorización explícita del usuario en el momento de ejecutarla, además de la aprobación de la herramienta del broker.
5. Nunca solicitar ni copiar claves API en el chat. Configurarlas únicamente en el entorno seguro del proveedor.
6. Tratar cotizaciones gratuitas de Yahoo/Stooq como no oficiales y potencialmente retrasadas; verificar precio y estado de la orden con el broker.
7. Si faltan datos, el calendario de mercado no está cubierto o una fuente falla, indicarlo y no inventar resultados.

## Configuración pendiente antes de activar operaciones

No copiar los valores de `config.example.json` como si fueran límites personales. El usuario debe definir expresamente, como mínimo:

- presupuesto destinado a paper trading;
- pérdida máxima por operación y pérdida máxima diaria;
- número máximo de operaciones al día;
- tamaño máximo de posición y reserva de efectivo;
- horario permitido de entrada y hora límite de cierre;
- lista de símbolos bloqueados;
- broker y confirmación de que la cuenta es paper.

Las reglas de ejemplo del repositorio (por ejemplo, presupuesto de USD 5.000 y pérdida máxima de USD 50) son demostrativas, no recomendaciones ni datos confirmados del usuario.

## Próximos pasos técnicos

1. Obtener una copia exacta del `alerts.py` que utiliza actualmente Trading IA - Trader Alerts y sus archivos JSON de ejemplo (sin información sensible).
2. Comparar ambos motores y decidir si se conserva el actual, se adapta el del repositorio o se hace una migración con compatibilidad y pruebas.
3. Ejecutar la suite offline del repositorio: `python3 -m unittest discover -s tests -v`.
4. Preparar un adaptador de ejecución externo si se desea invocar los scripts desde ChatGPT.
5. Probar primero con datos de fixtures y paper trading; no activar una cuenta real durante la integración.

---

**Nota:** Este documento es una guía de arquitectura. Su presencia en el fork no instala el paquete en el equipo del usuario, no configura Claude y no habilita ejecución Python dentro de ChatGPT.
