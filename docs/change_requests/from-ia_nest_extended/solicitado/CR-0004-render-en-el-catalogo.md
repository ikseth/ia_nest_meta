# CR-0004: el catalogo publica como se presenta cada capacidad

- id: CR-0004
- fecha: 2026-08-19
- capa origen: `ia_nest_extended`
- capa destino: `ia_nest_core`
- estado: propuesto

## Caso de uso motor

El catalogo del core publica, por capacidad, sus parametros y su proyeccion en
cada interfaz: grupo, accion, alias, textos de ayuda y banderas. Con eso una capa
de encima puede construir sus subcomandos sin copiar nada, y es lo que
`ia_nest_extended` acaba de implementar (`extended ADR 0012`).

Lo que el catalogo NO publica es COMO se presenta la respuesta. Los
renderizadores del core viven en un diccionario de codigo de su propia CLI, de
modo que son inalcanzables para cualquier consumidor.

Efecto medido el 2026-08-19 sobre la misma capacidad y la misma peticion:

- por la CLI del core, `task.plan` imprime una linea por subtarea, legible;
- a traves de la capa de encima, la misma llamada imprime el JSON completo.

El operador que sube de capa recibe MENOS de lo que recibia abajo. Eso incumple
la regla 3 de `meta ADR 0007` -extension aditiva: una capa nunca degrada lo de
abajo- en la piel, aunque la capacidad y su respuesta viajen intactas.

## Que se pide

Que el catalogo publique lo suficiente para que otra piel presente una capacidad
como la presenta el core. No se pide un motor de plantillas ni que el core
renderice para nadie: se pide que la informacion deje de estar solo en su codigo.

## Forma sugerida (no vinculante)

Un descriptor de presentacion en la proyeccion de CLI de cada capacidad, con la
granularidad que el core considere sostenible. Incluso una forma gruesa resuelve
el caso: declarar que la respuesta lleva una lista en `plan[]` y que sus columnas
legibles son `index`, `domain` y `prompt` basta para reproducir la salida de
`task.plan` sin conocerla.

Para las capacidades cuya salida es un texto plano, declararlo explicitamente
tambien vale: hoy quien reenvia no puede distinguir "esto se imprime tal cual" de
"esto es una estructura que alguien sabe pintar".

## Impacto

- Core: adicion compatible -> PATCH o MINOR. No altera ninguna capacidad, ni sus
  parametros, ni la forma de sus respuestas.
- Extended: no bloquea. Mientras no exista, esta capa declara en su contrato que
  lo reenviado se presenta en JSON.

## Alternativas consideradas

- **Que cada capa escriba sus renderizadores.** Es codigo por capacidad, que es
  justo lo que `meta ADR 0007` prohibe: cada capacidad nueva del core obligaria a
  editar cada capa de encima. Ademas se desincroniza en silencio.
- **Aceptar JSON en lo reenviado.** Es lo que hay hoy y es honesto si se declara,
  pero degrada al operador cada vez que sube una capa, y el ente va a tener mas
  capas, no menos.
- **Que la capa de encima consuma la CLI del core en vez de su REST.** Ata una
  interfaz a otra y rompe que las capas se compongan por contrato.

## Tension doctrinal (honesta)

Existe y conviene nombrarla: la presentacion no es logica de motor, y el core
puede argumentar que su contrato describe capacidades, no como se pintan. Es un
argumento legitimo.

La otra cara: el propio core decidio en su ADR 0046 que su catalogo sea la FUENTE
UNICA de la que se derivan sus tres interfaces, precisamente para que la deriva
entre ellas sea imposible. Este CR observa que esa fuente unica se queda corta en
cuanto hay una cuarta piel, que es la de la capa de encima; y que las capas de
encima van a existir por diseno del ente.

Si el core prefiere no crecer por aqui, la respuesta util para esta capa es un
"no" explicito: entonces se declara en contrato que lo reenviado va en JSON y se
deja de esperar.

Extended propone; el core dispone.
