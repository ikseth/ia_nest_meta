# CR-0002: extended transporta el perfil del interlocutor y lo compone

- id: CR-0002
- fecha: 2026-10-22
- capa origen: `ia_nest_edge`
- capa destino: `ia_nest_extended`
- estado: propuesto

## Caso de uso motor

El mismo que `edge CR-0001`: el ente debe saber que habla con un nino, sin que el
borde le diga como comportarse. Mientras conscience no exista, alguien tiene que
convertir ese hecho en contexto para el modelo.

Extended es el sitio: saber con quien hablas es contexto del interlocutor, y la
capa ya arma el prompt con lo que sabe de el (`core ADR 0031`, "lo arma la
capa"). No es juicio: no decide como tratar a un nino, solo hace que el modelo
lo sepa. El juicio sigue destinado a conscience.

## Que se pide

1. **Transportar** el campo de identidad de `edge CR-0001`: leerlo en REST y MCP
   y reenviarlo al core. Hoy `MemoryIdentity` tiene cinco campos y descarta el
   resto en silencio.
2. **No usarlo como clave de memoria**: es procedencia, como `service`. Si fuera
   clave, los recuerdos de una misma persona se partirian
   (`docs/FORMA_ENRIQUECIMIENTO.md`, decision 1).
3. **Componerlo siempre**: con perfil infantil, el prompt lleva el contexto del
   interlocutor, con el texto que diga la configuracion de la capa. No es una
   fuente que se pueda apagar: se compone tambien con el enriquecimiento
   desactivado.
4. **No memorizarlo**: el write-back conserva la pregunta literal, como ya hace.
5. **Declararlo en telemetria**: que el evento diga si el contexto del
   interlocutor se compuso. Es como se comprueba el criterio duro dentro del ente.

## Impacto

- Extended: aditivo -> PATCH. Necesita el core con `edge CR-0001`.
- Borde: igual que `edge CR-0001`; bloquea la entrada de la familia, no la
  primera entrega.

## Limites que el borde ya asume

- Las capacidades que extended no enriquece (`task.stream`) no componen nada. El
  borde solo usa `prompt.*` para identidades infantiles.
- El contexto va dentro del mensaje de usuario, no como mensaje de sistema. Si el
  modelo lo atiende o no se comprueba en el laboratorio antes de que entre la
  familia.

## Tension doctrinal

Con el alcance de la capa (su `ALCANCE.md` manda el juicio a conscience): se
respeta si la composicion es mecanismo -un texto configurado- y no una decision
de la capa sobre como tratar al interlocutor. Extended propone la forma; el
operador reconcilia.

Coordinacion: el hilo de extended tiene abierto su hito siguiente (paso cero,
reranker, Fase 9). Esta pieza no es ruta critica: la familia espera ademas por
L5r y L4b.
