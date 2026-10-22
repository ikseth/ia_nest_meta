# CR-0003: un modo iterativo de uso normal

- id: CR-0003
- fecha: 2026-10-22
- capa origen: `ia_nest_edge`
- capa destino: `ia_nest_extended`
- estado: propuesto

## Caso de uso motor

El borde ofrece a cada consumidor un menu de modos (edge ADR 0002, punto 5). El
modo **iterativo** -descomponer, iterar, combinar (`task.run`)- es el que el
operador quiere por defecto en el webchat. Hoy solo puede ofrecerse como
PROTOTIPO, por dos huecos ya declarados en extended:

1. **Sin streaming con memoria.** `task.stream` se reenvia sin enriquecer
   (`extended docs/EXTENDED_CONTRACT.md`): el core no acepta `plan` ni
   `requirements` en esa capacidad. Un webchat en streaming perderia memoria y
   RAG, que es justo lo que se quiere estudiar con la familia. El borde lo suple
   con `task.run` y la respuesta entera al final: minutos sin ver nada.
2. **Sin re-planificar con enriquecimiento.** `task.run` enriquecido por subtarea
   no re-planifica: si EVALUATE lo pide, el core corta con `replan_unavailable`
   (`extended docs/PLAN.md`, coste declarado de la Fase 7b; tambien en
   `meta docs/CAPAS_FUTURAS.md`, "quien decide y quien orquesta").

## Que se pide

Que el modo iterativo pueda usarse de forma normal a traves de extended:

1. `task.stream` enriquecido, con la misma garantia que `task.run`.
2. Re-planificacion compatible con el enriquecimiento por subtarea.

## Forma sugerida (no vinculante)

Las dos piezas tocan la costura con el core, asi que lo natural es que extended
disponga la forma y abra su propio CR al core (`extended CR-NNNN`). El borde no
pide al core directamente: depende de extended.

## Impacto

- Extended y core: lo que dispongan. Previsiblemente aditivo.
- Borde: sin cambios en ningun cliente. El modo iterativo pasa de `task.run` a
  `task.stream` y sale del menu del operador a los demas.
- No bloquea nada hoy: el iterativo es prototipo por decision del operador.

## Alternativas consideradas

- **`task.stream` sin enriquecer.** Descartada: un modo de conversacion sin
  memoria contradice para que existe extended.
- **Simular el progreso en el borde.** Mensajes de espera inventados por el
  borde: parece streaming y no lo es. Descartada por la misma regla que
  `extended CR-0005` aplica a los datos plausibles y falsos.

## Tension doctrinal

Ninguna nueva. Es el obstaculo que `meta docs/CAPAS_FUTURAS.md` ya anunciaba:
"quien aborde esto se lo encontrara, y el canal para resolverlo es un CR al
core". Llega ahora con consumidor real.
