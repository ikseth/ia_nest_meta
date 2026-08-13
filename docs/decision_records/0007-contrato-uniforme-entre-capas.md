# Decision 0007: contrato uniforme entre capas del ente

Fecha: 2026-08-13

## Decision

Se fija la FORMA en que las capas del ente se componen, en
`docs/ARQUITECTURA_DE_CAPAS.md`:

1. **Invariante**: anadir una capa no obliga a editar ninguna capa existente, ni
   el cliente. Criterio falsable, aplicable a cualquier propuesta.
2. **Contrato uniforme**: cada capa expone el contrato COMPLETO de la que
   envuelve, mas lo suyo propio.
3. **Reenvio por defecto, sobreescritura por excepcion**: lo que una capa no
   transforma lo reenvia con un mecanismo generico, no con una implementacion por
   capacidad.
4. **Extension aditiva**: subir de capa siempre suma; nunca quita ni degrada.
5. **Clientes del contrato, no de las capas**: un cliente se escribe contra el
   contrato y apunta a un extremo configurable.

La direccion de la dependencia NO cambia: cada capa depende solo de la de abajo,
y la de abajo sigue sin conocer a las de arriba.

## Motivo

Al disenar la interfaz de consumo de `ia_nest_extended` (su Fase 7) reaparecio,
por tercera vez en el proyecto, la misma discusion sin converger: si la capa
superior duplica el catalogo de la inferior, si el core debe exponer un puerto
hacia arriba, o si hace falta un repo de frontend unificado.

La causa de la no convergencia no era ninguna de las opciones: era que **no
estaba escrito el criterio** que una solucion debia cumplir. Cada propuesta
movia el acoplamiento a otro fichero y parecia razonable hasta que se preguntaba
que pasa con la capa siguiente.

Escrito el invariante, las cuatro opciones que se habian barajado se descartan
solas, cada una por el mismo motivo:

| Opcion | Al anadir una capa hay que editar |
|---|---|
| Capa superior con catalogo reducido | el habito del usuario: dos interfaces, capacidades perdidas |
| Capa superior con catalogo espejo a mano | una piel por capacidad, en cada capa |
| Puerto de enriquecimiento en el core | el core, que pasa a transportar conceptos de arriba |
| Repo de frontend unificado | el frontend, una vez por capa nueva |

El contrato uniforme es la unica familia que pasa el test, y es un patron
conocido y probado (proxy/decorador sobre interfaz uniforme; HTTP es su caso
canonico).

Precision sobre el puerto: la opcion se estudio en serio, no se descarto por
inercia. `core ADR 0031` dejo el punto explicitamente abierto y `core ADR 0035`
lo cerro por falta de consumidor, motivo hoy caducado. Pero `core ADR 0040` ya
disposo sobre una peticion equivalente (`extended CR-0001`) y aporto el argumento
que la cierra: los prompts de subtarea quedan fijados en la etapa PLAN, luego el
enriquecimiento NO necesita ocurrir dentro del bucle de orquestacion. Sin esa
necesidad, el puerto solo resolvia un problema de interfaz, y cambiar el
fundamento del core para eso es desproporcionado.

## Consecuencia

- Nace `docs/ARQUITECTURA_DE_CAPAS.md` (doctrina de ente, plano meta). Los repos
  de capa lo citan desde su `AGENTS.md`; no lo duplican.
- No se emite CR al core. Esta doctrina no toca su contrato ni reabre
  `core ADR 0031`, `0035` ni `0040`; se apoya en el ultimo.
- `ia_nest_extended` reencuadra su Fase 7: su interfaz de consumo pasa a ser el
  contrato del core reexpuesto (reenvio generico) mas sus capacidades propias, en
  lugar de un catalogo distinto y menor. Su Fase 7a (CLI) se redimensiona en
  consecuencia.
- `ia_nest_core_conscience` e `ia_nest_web` heredan la regla antes de existir:
  conscience envolvera a extended por el mismo patron, sin editar ni core ni
  extended.
- La doctrina se declara aplicable tambien a los clientes: un cliente con una
  rama por capa incumple el invariante.

## Alternativas consideradas y descartadas

Las cuatro de la tabla del motivo. Ninguna se descarta por preferencia
estetica: todas fallan el mismo test, y el test se escribio antes de aplicarlo.

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Es doctrina de construccion, no
contrato. Los cambios que induzca en cada capa se versionan en esa capa.
