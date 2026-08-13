# Decision 0005: levantar la plantilla de brief (aparecio el consumidor)

Fecha: 2026-08-06

## Decision

Se crea `docs/PLANTILLA_BRIEF.md`: el esqueleto del brief con el que un
disenador pasa trabajo a un codificador en modo ciego.

El charter ya listaba "plantillas de brief y handoff" en "Vive aqui", y el
README las marcaba **diferidas por falta de consumidor**. Esta decision no
amplia el charter: activa un elemento que ya estaba dentro y que estaba
esperando exactamente a esto.

Alcance deliberadamente estrecho:

- La plantilla fija la FORMA del brief. El QUE debe llevar ya lo fija
  `DOCTRINA_MULTI_IA.md` (seccion Handoff) y no se duplica: la plantilla es esa
  misma lista escrita como formulario.
- No describe el CANAL por el que un brief llega a su destinatario. El canal es
  contexto local de cada maquina y no se versiona (regla 5 de
  `CONVENCIONES_TRANSVERSALES.md`).

De la pareja "brief y handoff" se levanta solo el brief. La plantilla de handoff
sigue diferida: no tiene consumidor.

## Motivo

El consumidor aparecio en `ia_nest_core` el 2026-08-06: una tanda de trabajo de
implementacion sobre contrato ya reconciliado (`core ADR 0041` y las fichas
`core ficha v0.3/0004` a `0007`) que se entrega a un codificador en ciego.

Hasta hoy el contenido del brief se reconstruia de memoria a partir de la
doctrina en cada entrega. Eso funciona una vez; a la tercera, cada brief lleva
una lista distinta, y la seccion que se cae primero es "que NO entra", que es
justo la que protege el trabajo en curso de otros agentes.

La tercera pregunta del charter -"tiene consumidor HOY?"- se cumple. Las otras
dos ya se cumplian: no es de una sola capa (cualquier capa entrega trabajo a un
codificador) y es de COMO se construye el ente, no de que hace una capa.

## Consecuencia

- `README.md` y `IA_NEST_META_CONTEXT.md`: "plantillas de brief y handoff" deja
  de figurar como diferido en bloque; queda diferida solo la de handoff.
- No cambia `DOCTRINA_MULTI_IA.md`: la plantilla la cita, no la sustituye ni la
  modifica. Su version sigue en 1.0.
- No afecta al contrato publico de ninguna capa: meta no versiona contratos.
- Alternativas descartadas:
  - **Dejarla diferida hasta tener dos capas entregando trabajo.** La regla del
    charter pide "consumidor hoy", no "dos consumidores": exigir dos seria una
    regla distinta, y mas estricta, que la escrita.
  - **Que la plantilla viva en el repo que la estrena (`core`).** Falla la
    pregunta 1 del charter: no es de una sola capa. Ademas repetiria el patron
    que motivo este repo, que es doctrina de ente alojada dentro de una capa.
  - **Fundir la plantilla dentro de `DOCTRINA_MULTI_IA.md`.** Mezcla norma con
    formulario: la doctrina dice que debe cumplirse y perdura; la plantilla es
    ergonomia y se ajustara con el uso. Separadas, cada una se revisa a su
    ritmo.
  - **Levantar tambien la plantilla de handoff.** No tiene consumidor. Es la
    leccion de `core ADR 0035`, que el charter cita expresamente.
