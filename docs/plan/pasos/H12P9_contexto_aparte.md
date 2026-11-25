# H12P9: el contexto va aparte de lo que dice la persona

Estado: en curso; D1-D3 decididos por el usuario el 2026-11-25; ADR del core y de
extended aceptados el mismo dia
Version: 0.1 - 2026-11-25

Paso de `H12`. Entrega meta ADR 0020 (H12P8, D5): extended deja de pegar el
contexto delante de la frase y el core lo coloca segun la confianza de su
origen. Es construccion del producto: la aplicacion en el laboratorio, la medida
y el cierre de `H12` siguen en H12P8, despues de este paso.

## Punto de partida

Core `v0.7.1`, extended `v0.8.2`, borde `v0.3.0`. Meta ADR 0020 decidido (los
ocho puntos). Al bajarlo al codigo salieron tres casos que 0020 no resolvia.

## D1: la reescritura de `task.run` se mantiene

Extended reescribe con la memoria la peticion de `task.run` antes de planificar,
para que se entienda sola (deuda D8 de extended, medida 9 de 10). Choca con la
letra del punto 1 de 0020 ("el prompt es lo que dijo la persona").

El usuario elige mantenerla: la reescritura es una peticion, no contexto pegado,
esta medida y es la practica habitual ("condense question" en RAG
conversacional). El planificador recibe la peticion reescrita y, aparte, el
contexto. Quitarla se mide aparte, mas adelante.

Descartados: quitarla (sin medir, puede volver el fallo de D8); quitarla detras
de una opcion y medir antes (alarga el paso sin un fallo que lo pida).

## D2: `domain.route` con la frase y el hilo como pista

Extended pega hoy el ultimo intercambio delante de la frase al enrutar, para que
"y manana?" vaya al dominio del turno anterior. 0020 pide la frase limpia.

El usuario elige: el `prompt` de `domain.route` es la frase; el ultimo
intercambio va como bloque `thread` y el core lo lee como pista delimitada,
igual que la pista del marco (core ADR 0055, D2).

Descartados: solo la frase ("y manana?" pierde el dominio); dejarlo como esta
(el enrutador seguiria clasificando una mezcla).

## D3: en `task.run`, la confianza baja a las subtareas y el marco no

Al declarar `turn_context` en `task.run`, extended mandara tambien
`trusted_providers` y `frame`. El usuario elige: `trusted_providers` baja a cada
subtarea (la memoria solo resta, como en `prompt.run`); `frame` se ignora en
`task.*`, para que ninguna subtarea pregunte a mitad de tarea.

Descartados: solo el contexto (una restriccion que llega y se ignora); confianza
y marco (una subtarea podria terminar en `input_required`; diseno nuevo).

## Precisiones que se siguen de lo decidido

- `task.plan` acepta `turn_context` igual que `task.run`: extended pide el plan
  con su propia llamada, y es el planificador.
- Extended manda aparte lo mismo que hoy pega, ni mas ni menos: el planificador
  y el combinador reciben el interlocutor; cada subtarea, el interlocutor y su
  RAG; la memoria solo alcanza la reescritura.
- `entity.unresolved` lista tambien el dispositivo sembrado sin casa, por la
  misma causa que la persona (el lugar queda `unknown`).

## Entregables

| Capa | ADR | Estado |
| --- | --- | --- |
| core | core 0056 (`v0.8.0`) | aceptado, rama local `feat/contexto-aparte` |
| extended | extended 0018 (`v0.9.0`), enmienda a extended 0017 punto 6 | aceptado, rama local `feat/contexto-aparte` |

## Por hacer

1. Aceptar los ADR del core y de extended (hecho, 2026-11-25).
2. Briefs de Codex y revision: core primero, extended despues.
3. Tags con aprobacion del usuario; despliegue por instalador y repeticion del
   humo de H12P8.
4. Aviso por issue a cada capa de destino (ajuste de H12P6), con aprobacion del
   usuario.
5. Cierre del paso: fila H12P9, comentario en Deck #200 y traspaso de la fase 2
   de H12P8.
