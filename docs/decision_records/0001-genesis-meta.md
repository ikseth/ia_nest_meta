# Decision 0001: genesis de ia_nest_meta (el taller del ente)

Fecha: 2026-07-26

## Decision

Se crea `ia_nest_meta` como repo de GOBERNANZA del ente IA_NEST, con estas
propiedades:

1. **No es una capa.** Meta queda FUERA del mapa ente/exterior del core
   (core ADR 0033): no es parte del ente ni un consumidor exterior. Es un plano
   distinto, el taller donde el ente se disena y se gobierna. Su zona propia es
   "Taller".
2. **Charter estricto** (`README.md`) como regla de corte, para evitar el
   repo-Dios: aqui vive lo del ENTE o lo de COMO se construye el ente; en cada
   capa se queda su contrato, sus ADR, su PLAN y su codigo.
3. **Arranque minimo.** Solo se siembra lo que tiene consumidor hoy: el proceso
   de Change Request y CR-0001. El resto del charter (registro de capas,
   convenciones transversales, doctrina multi-IA, SemVer de ente, capas
   futuras, plantillas de brief) se re-hoga por pasos reconciliados.
4. **Los ADR son historia.** Ningun ADR se reescribe ni se mueve entre repos.
   Cuando una regla viva se re-hoga, al ADR de origen se le APENDIZA una
   seccion `## Estado posterior` con el puntero, dejando su cuerpo intacto, y
   meta registra el movimiento con un ADR propio.
5. **Diferimiento explicito: meta no adopta SemVer todavia.** Los docs
   normativos llevan cabecera `Estado:` y `Version: X.Y - fecha`. Se revisara
   cuando un segundo consumidor real del contrato documental de meta (la
   anatomia del CR) lo justifique.

## Motivo

El core, por ser el primer repo, absorbio gobernanza que no es del motor. Eso
ya se estaba pagando en dos sitios observables, no hipoteticos:

- `ia_nest_extended` cita `core ADR 0016` para la regla de idioma de
  identificadores: una regla transversal del ecosistema que nada tiene que ver
  con enrutar o inferir. La capa se dobla hacia el core para encontrar doctrina.
- El proceso de Change Request nacio sin hogar dentro de `ia_nest_extended`,
  con su README declarando que su version autoritativa debia "elevarse al
  core". Un artefacto BILATERAL (entre dos capas) no tiene sitio natural en
  ninguna de ellas, y meterlo en el motor habria vuelto a engordar el core con
  gobernanza.

El argumento decisivo es estructural: el registro de capas del core tiene zonas
`Ente` y `Exterior`, y `ia_nest_meta` no cabe en ninguna. Un registro que no
puede listar al repo que lo hospeda esta en el sitio equivocado. Ademas, ese
registro ya se ha quedado obsoleto donde vive (declara el core en `v0.1.0` y a
`extended` "en diseno" cuando extended ya depende de `>=0.2 <0.3` y tiene su
memoria verificada): se pudre porque lo mantiene quien no lo consume.

Alternativa considerada y descartada: seguir elevando la doctrina de ente al
core. Se descarta porque contradice la regla anti-entropia del propio core
(`IA_NEST_CORE_CONTEXT.md`) y porque el core no puede ser a la vez una pieza
del ente y el arbitro entre piezas.

## Consecuencia

- El hogar autoritativo del proceso de CR pasa a `docs/change_requests/`, con
  estado proyectado por directorio (`from-<origen>/solicitado|resuelto/`).
  CR-0001 se porta con su texto intacto.
- `ia_nest_extended` conserva un puntero de una linea en
  `docs/change_requests/README.md` y actualiza su `AGENTS.md`. Su `CHANGELOG.md`
  no se toca: es historia.
- El core NO se toca en este paso. La re-hogacion de su doctrina de ente
  (registro de capas, convenciones transversales, doctrina multi-IA, split de
  SemVer y de `CAPAS_FUTURAS.md`) queda propuesta, por pasos, y se registrara en
  ADRs posteriores de este repo conforme se reconcilie cada paso.
- Convencion nueva, transversal: los ADR se citan como `<repo> ADR NNNN`, nunca
  `ADR NNNN` a secas, porque la numeracion es por repo y colisiona. La practica
  ya era de facto en `ia_nest_extended`; aqui se hace normativa.
