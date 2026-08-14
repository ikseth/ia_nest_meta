# Decision 0009: forma de errores y de traza entre capas

Fecha: 2026-08-14

## Decision

Nace `docs/FORMA_DE_ERRORES_Y_TRAZA.md`, que fija el minimo comun de lo que
cruza una frontera de capa:

1. **Forma del error**: `type`, `message`, `field` opcional, `origin` (que capa
   lo origino) y `request_id`.
2. **Un error reenviado no se re-envuelve**: se propaga tal cual, con su `origin`
   intacto. Una capa solo emite error propio cuando el fallo es suyo.
3. **La identidad del request viaja intacta**: ninguna capa la reescribe ni la
   completa por inferencia.
4. **Encadenado de traza**: cada capa es duena de su `request_id` y registra
   ademas `downstream_request_id`, el de la capa a la que llamo.

El CATALOGO de errores de cada capa se queda en esa capa. Aqui solo la forma.

## Motivo

Con el contrato uniforme (meta ADR 0007) una peticion atraviesa varias capas, y
el consumidor recibe errores y trazas que pueden originarse en cualquiera. Hasta
ahora cada capa resolvia esto sola: el core tiene su `CoreError` con
`type`/`message`/`field`, y la capa de enriquecimiento iba a definir el suyo por
separado, con otra forma. Dos taxonomias con formas distintas obligan a traducir
en cada salto, y traducir destruye informacion: con tres capas, el consumidor
acaba recibiendo un error que no describe el fallo real.

Se escribe AHORA, y no despues, por una razon de coste: la capa de
enriquecimiento esta a punto de fijar su taxonomia en implementacion. Definirla
primero y armonizarla despues seria deuda deliberada, y la forma correcta es
barata mientras no haya contrato cortado.

El campo `origin` es el unico que no existiria en un repo aislado, y es
precisamente el que un ente por capas necesita: sin el, un consumidor sabe que
algo fallo pero no a que altura de la pila.

## Consecuencia

- `ia_nest_extended` define su `ExtendedError` sobre esta forma, no en paralelo,
  y su brief de Fase 7a lo recoge. Su telemetria usa `downstream_request_id`
  donde hoy tiene un nombre especifico del vecino; es gratis, porque no tiene tag
  cortado.
- `ia_nest_core` cumple ya la parte de forma salvo `origin`, que no necesitaba
  siendo la capa mas baja. Se alineara cuando toque su taxonomia; no se le pide
  nada ahora y no se emite CR.
- `ia_nest_core_conscience`, `ia_nest_core_pulse` e `ia_nest_web` heredan la
  regla antes de existir. Pulse en particular depende de ella: observa la
  telemetria de todas las capas y sin encadenado no puede reconstruir un
  recorrido.
- Lo que NO se hace: mover el vocabulario de identidad del request al taller. Su
  hogar es el contrato del core y las demas capas lo referencian, que es lo que
  la convencion transversal 6 ya exige; moverlo obligaria al taller a editar el
  contrato de una capa, cosa que su charter prohibe.

## Impacto de version

Ninguno en el contrato publico de ninguna capa hoy. Cuando una capa alinee su
taxonomia, lo versiona en su repo.
