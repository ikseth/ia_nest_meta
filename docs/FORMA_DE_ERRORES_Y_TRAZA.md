# Forma de errores y de traza entre capas

Estado: activo
Version: 1.0 - 2026-08-14

Con el contrato uniforme (`ARQUITECTURA_DE_CAPAS.md`, meta ADR 0007), una
peticion atraviesa varias capas antes de resolverse. El consumidor recibe
respuestas, errores y trazas que pueden haberse originado en cualquiera de ellas.

Este documento fija la FORMA de esos errores y de esa traza. El CATALOGO de
errores de cada capa se queda en esa capa: aqui no se enumera ninguno.

Origen: meta ADR 0009.

## 1. Forma del error

Un error que cruza una frontera de capa lleva, como minimo:

| campo | que es |
|---|---|
| `type` | identificador estable del tipo de error, del catalogo de la capa que lo origina |
| `message` | texto legible, en la convencion de idioma del ente |
| `field` | opcional: que entrada lo provoco |
| `origin` | QUE CAPA lo origino |
| `request_id` | el de la capa que lo origina, si lo tiene |

`origin` es el campo que solo tiene sentido en un ente por capas, y el unico que
no existiria si cada capa se mirase el ombligo. Sin el, un consumidor sabe que
algo fallo pero no a que altura de la pila, y depura a ciegas.

## 2. Un error reenviado NO se re-envuelve

La capa que reenvia un error de una capa inferior lo propaga TAL CUAL, con su
`origin` intacto. No lo traduce, no lo re-tipa, no lo envuelve en un error propio
ni lo degrada a un mensaje generico.

Motivo: re-envolver destruye informacion en cada salto y, con tres capas, el
consumidor recibe un error que no dice nada del fallo real. Ademas obligaria a
cada capa a mantener una tabla de traduccion de los errores de la de abajo: un
catalogo ajeno replicado, contra la convencion transversal 6.

Una capa SOLO emite error propio cuando el fallo es suyo.

**Excepcion acotada: completar `origin` ausente.** Si la capa inferior no emite
`origin`, la que reenvia lo COMPLETA con la identidad de a quien llamo, que
conoce con certeza. No es traducir: `type`, `message` y `field` se dejan
intactos. Sin esta excepcion, `origin` seria inutil hasta que TODAS las capas lo
emitiesen, y la mas baja de la pila es justo la que menos motivo tiene para
declararse a si misma.

Completar es rellenar un hueco; sobrescribir un `origin` ya presente es
falsificar la procedencia y no esta permitido.

## 3. La identidad del request viaja intacta

El contexto de identidad del request (definido en `core docs/CORE_CONTRACT.md`;
aqui no se copia) atraviesa todas las capas SIN alterarse. Ninguna capa lo
reescribe, lo completa por inferencia ni lo sustituye por el suyo.

Una capa puede DERIVAR de el sus propias claves internas -es lo que hace la capa
de memoria-, pero lo que envia hacia abajo es lo que recibio.

## 4. Encadenado de la traza

Cada capa emite su propia traza y es duena de su `request_id`. Para que la cadena
se pueda reconstruir, cada capa registra ademas el identificador de la llamada
que hizo hacia abajo:

    request_id             el propio, de esta capa
    downstream_request_id  el de la capa inmediatamente inferior, si la llamo

Con esos dos campos, un consumidor (o `ia_nest_core_pulse`, que observa la
telemetria de todos) reconstruye el recorrido completo de una peticion sin que
ninguna capa tenga que conocer a las demas.

El nombre `downstream_request_id` es el generico, para que la cadena se lea igual
en toda la pila. Una capa que hoy use un nombre especifico de su vecino lo
renombra cuando toque su telemetria.

## 5. Que se queda en cada capa

- El CATALOGO de tipos de error: que errores existen y que significan.
- La decision de que cuenta como error propio frente a fallo reenviado.
- El formato de presentacion en su CLI y los codigos de salida.
- Sus contadores y eventos de telemetria propios.

Esta forma es un minimo comun, no un esquema cerrado: una capa puede anadir
campos, nunca quitar los de la tabla ni cambiar su significado.
