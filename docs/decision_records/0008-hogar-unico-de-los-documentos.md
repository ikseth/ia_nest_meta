# Decision 0008: hogar unico de los documentos y criterio de ubicacion

Fecha: 2026-08-14

## Decision

Se anade la regla 6 a `CONVENCIONES_TRANSVERSALES.md`:

1. **Hogar unico.** Un documento normativo tiene UN hogar. Las demas capas lo
   referencian; nunca lo copian.
2. **Criterio de ubicacion.** Un documento vive donde vive su unidad de cambio:
   con el codigo si cambia siempre a la vez que ese codigo (contrato publico de
   una capa); en el taller si cambia con independencia de cualquier
   implementacion y lo consumen todos (doctrina, vocabulario compartido,
   indices).
3. **Presencia fisica generada, no copiada.** Si un documento debe aparecer en
   otro repo, se genera, con cabecera que lo declara y nombra su origen.
4. **Los indices del taller se generan** desde los manifiestos de cada capa, que
   son su fuente de verdad.

## Motivo

Al declarar el contrato publico de `ia_nest_extended` se planteo centralizar los
contratos de todas las capas en este taller, para no duplicar documentacion. La
discusion resulto ser dos preguntas distintas mezcladas: donde vive el TEXTO de
cada contrato, y donde vive el INDICE que apunta a ellos.

Sobre el texto: ninguna ubicacion elimina la desincronizacion; cada una ELIGE que
pareja debe mantenerse en sincronia. El contrato en el taller sincroniza
contrato-con-contrato, pero abre contrato-con-codigo. El contrato en su capa
sincroniza contrato-con-codigo, que ademas es la unica pareja con verificacion
automatica: la bateria de conformidad de la capa desmiente al codigo que se
desvie de lo que su contrato promete. Un contrato alojado lejos de su
implementacion no tiene ese test al lado, y espera a que un humano note la
deriva.

Sobre el indice: la preocupacion que abrio la discusion es real y ya esta
materializada. `REGISTRO_CAPAS.md` declara reflejar los manifiestos
`DEPENDENCIAS.md` de cada capa, y ese reflejo se mantiene a mano. Es un dato
replicado sostenido por disciplina humana, y por tanto se desactualizara. La
correccion no es mover el texto de los contratos: es que los indices no se
escriban a mano.

De ahi la regla, que separa las dos cosas y ataca la que de verdad falla.

## Consecuencia

- El contrato publico de cada capa se queda en su repo (confirma el charter, que
  ya lo decia, y ahora con un criterio y no solo por reparto).
- El taller gana un deber: sus indices se generan. `REGISTRO_CAPAS.md` y el
  futuro indice de contratos pasan a ser artefactos generados desde los
  manifiestos. Mientras la generacion no exista, se mantienen a mano y esa deuda
  queda declarada aqui.
- Ningun documento del ente se replica a mano. Un documento encontrado en dos
  sitios es un defecto, no una comodidad.
- Queda pendiente, sin bloquear a nadie: llevar al taller el vocabulario de
  identidad del request (hoy en `core CORE_CONTRACT.md`, consumido por todas las
  capas) y la forma transversal de errores y traza entre saltos (hoy inexistente;
  cada capa iba a inventar la suya).

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Es convencion de construccion.
