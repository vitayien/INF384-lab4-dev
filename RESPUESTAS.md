# Respuestas · Laboratorio 4
Alumno: 20200909
Presupuesto: 8 min  Métrica DORA: time to restore

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa:smoke, etapa build.
- Problema que causa en el pipeline:smoke depende de build-push y puede ejecutarse en paralelo con deploy. Por ello, puede probar una versión anterior de Lambda.
- Link al commit: https://github.com/vitayien/INF384-lab4-dev/commit/69b9378bd87c8b6eee420d42312bee333eeb4c7f 
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):mover de etapa.
- Evidencia (dato observado, con link al run o fragmento de la salida local): smoke terminó antes que deploy
### Defecto 2
- Job y etapa:smoke, etapa post-deploy.
- Problema que causa en el pipeline:No se configura EXPECTED_COMMIT, por lo que smoke.sh no verifica que la versión desplegada corresponda al commit actual.
- Link al commit: https://github.com/vitayien/INF384-lab4-dev/commit/ea5a847c315a39d8ef8fdeb6760b361406d81a1e
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local):El script solo valida /version dentro de if [ -n "${EXPECTED_COMMIT:-}" ], pero la variable no estaba definida en ci.yml
### Defecto 3
- Job y etapa: sca, etapa pre-build.
- Problema que causa en el pipeline: vulnerabilidades de paquetes que no se incluyen en la imagen final, porque el Dockerfile ejecuta npm prune --omit=dev
- Link al commit: https://github.com/vitayien/INF384-lab4-dev/commit/2727bfafdb86299eab7d6be70fb1dd14f32943b6
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local): --include-dev-deps --exit-code 1
### Defecto 4
- Job y etapa: build-push, etapa build
- Problema que causa en el pipeline: El escaneo de la imagen encuentra vulnerabilidades HIGH o CRITICAL, pero solo publica una advertencia y permite subir la imagen vulnerable a ECR y desplegarla.
- Link al commit: https://github.com/vitayien/INF384-lab4-dev/commit/6ac72a4ed95958d6c340f1b4f0806b0458b0c4dd
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local): El run mostro "211 hallazgos HIGH o CRITICAL. Informativo, no detiene."
### Defecto 5
- Job y etapa: sast, etapa pre-build
- Problema que causa en el pipeline: La opción --error bloquea todo el pipeline ante cualquier hallazgo ERROR de Semgrep, lo que puede retrasar la restauración de una entrega cuando el hallazgo requiere revisión manual.
- Link al commit: https://github.com/vitayien/INF384-lab4-dev/commit/d3e49a121bcda880beafd208504d00cd3db0f2f6
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): dejar como informativo.
- Evidencia (dato observado, con link al run o fragmento de la salida local): --severity ERROR --error

## Parte 2 · Implementación
- Link al run en verde en main: https://github.com/vitayien/INF384-lab4-dev/actions/runs/40964620735
- Link al pull request: https://github.com/vitayien/INF384-lab4-dev/pull/1
- Defecto de la aplicación introducido (archivo y descripción): Se modificó src/wallet.js para devolver HTTP 400 cuando debería ser 402 debido a que el origen
  y el destino de una transferencia son el mismo usuario. 
- Job que debería detenerlo: unit
- Link al run fallido: https://github.com/vitayien/INF384-lab4-dev/actions/runs/40966769815/job/123773143256?pr=1

## Parte 3 · Decisión bajo restricción
Link al run medido: https://github.com/vitayien/INF384-lab4-dev/actions/runs/40964620735

| Job | Duración (min:seg) |
|---|---|
| unit | 12s|
| integration | 14s|
| sast | 18s|
| sca | 21s|
| build-push | 45s|
| deploy | 18s|
| smoke | 8s|
| e2e | 38s|
| load | 12m 12s|
| **Total** | 13m 57s|

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | 68.25ms|
| p95 | 70.3ms|
| Máximo | 264.09 ms|
| Tasa de error | 0.00 %|
| Requests por segundo | 48.782978 requests/s|

Conclusión sobre el comportamiento bajo carga: Monedero Andino se comportó de manera estable bajo una carga sostenida de 30 usuarios virtuales. La tasa de error fue de 0.00 %. El p95 de latencia fue 70.30 ms y el p90 fue 68.25 ms. Aunque se observó un máximo de 264.09 ms, este valor aislado no representa el comportamiento típico del servicio. El sistema procesó aproximadamente 48.78 requests por segundo.

SLO propuesto (percentil, valor, ventana) y justificación: p95 de latencia menor que 100 ms durante cada ventana de medición de carga de 5 minutos, con una tasa de error inferior al 1 %. Se propone 100 ms porque
el p95 observado fue 70.30 ms y este margen permite absorber la variación normal entre ejecuciones.

SLA propuesto y diferencia con el SLO: Se propone un SLA de p95 menor que 200 ms y una tasa de error inferior al 1 %, medido mensualmente. El SLA es más amplio que el SLO porque constituye un compromiso externo con los cliente contemplando variaciones como infraestructura, mantenimiento. El SLO es mas exigente pues propone un objetivo operativo para el equipo.

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|
| Reducir la carga constante de 10 min a 4 min | load | 7.00 |
| Mover SAST para que corra solo en pull requests | sast | 20s |

Total resultante: [6m 42s] min
Link al run en main con los cambios: https://github.com/vitayien/INF384-lab4-dev/actions/runs/40968989551

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada): Al ejecutar SAST solo en pull requests, un defecto de seguridad introducido directamente en main podría detectarse más tarde, durante una revisión o un escaneo posterior. Esto podría aumentar el time to restore, porque el defecto podría llegar a desplegarse y requerir una corrección y un nuevo despliegue.

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:La decisión permite reducir la duración del pipeline de main y cumplir el presupuesto de 8 minutos, manteniendo el control de seguridad antes de aceptar cambios mediante un pull request

## Declaración de uso de IA
Herramientas usadas y para qué:
Se utilizó ChatGPT como herramienta de apoyo para comprender las instrucciones del laboratorio, interpretar la configuración del pipeline y mejorar la redacción de las respuestas.
