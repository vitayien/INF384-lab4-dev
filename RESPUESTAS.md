# Respuestas · Laboratorio 4
Alumno: 20200909
Presupuesto: 8 min  Métrica DORA: time to restore

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa:smoke, etapa build.
- Problema que causa en el pipeline:smoke depende de build-push y puede ejecutarse en paralelo con deploy. Por ello, puede probar una versión anterior de Lambda.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):mover de etapa.
- Evidencia (dato observado, con link al run o fragmento de la salida local): smoke terminó antes que deploy
### Defecto 2
- Job y etapa:smoke, etapa post-deploy.
- Problema que causa en el pipeline:No se configura EXPECTED_COMMIT, por lo que smoke.sh no verifica que la versión desplegada corresponda al commit actual.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local):El script solo valida /version dentro de if [ -n "${EXPECTED_COMMIT:-}" ], pero la variable no estaba definida en ci.yml
### Defecto 3
- Job y etapa: sca, etapa pre-build.
- Problema que causa en el pipeline: vulnerabilidades de paquetes que no se incluyen en la imagen final, porque el Dockerfile ejecuta npm prune --omit=dev
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local): --include-dev-deps --exit-code 1
### Defecto 4
- Job y etapa: build-push, etapa build
- Problema que causa en el pipeline: El escaneo de la imagen encuentra vulnerabilidades HIGH o CRITICAL, pero solo publica una advertencia y permite subir la imagen vulnerable a ECR y desplegarla.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):ajustar configuración.
- Evidencia (dato observado, con link al run o fragmento de la salida local): El run mostro "211 hallazgos HIGH o CRITICAL. Informativo, no detiene."
### Defecto 5
- Job y etapa: sast, etapa pre-build
- Problema que causa en el pipeline: La opción --error bloquea todo el pipeline ante cualquier hallazgo ERROR de Semgrep, lo que puede retrasar la restauración de una entrega cuando el hallazgo requiere revisión manual.
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): dejar como informativo.
- Evidencia (dato observado, con link al run o fragmento de la salida local): --severity ERROR --error

## Parte 2 · Implementación
- Link al run en verde en main:
- Link al pull request:
- Defecto de la aplicación introducido (archivo y descripción):
- Job que debería detenerlo:
- Link al run fallido:

## Parte 3 · Decisión bajo restricción
Link al run medido:

| Job | Duración (min:seg) |
|---|---|
| unit | |
| integration | |
| sast | |
| sca | |
| build-push | |
| deploy | |
| smoke | |
| e2e | |
| load | |
| **Total** | |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | |
| p95 | |
| Máximo | |
| Tasa de error | |
| Requests por segundo | |

Conclusión sobre el comportamiento bajo carga:

SLO propuesto (percentil, valor, ventana) y justificación:

SLA propuesto y diferencia con el SLO:

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|

Total resultante: [ ] min
Link al run en main con los cambios:

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:

## Declaración de uso de IA
Herramientas usadas y para qué:
