# Respuesta del core a extended CR-0002

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Estado: reconciliado por el usuario (2026-08-14).
Base: `ia_nest_core` en `main`, commit `9310e04`, mas los cambios de contrato de
esta respuesta (`core ADR 0046`, sin publicar todavia).
Fecha: 2026-08-14
Responde a: `docs/handoff/cr_0002_brief_core.md`

## Resolucion

**ACEPTADO**, y ampliado. `core ADR 0046`.

El core acepta la necesidad y la forma sugerida en lo esencial -una capacidad
`capability.list`-, pero se queda con la parte del CR que el propio CR marcaba
como lo valioso: que el catalogo sea la FUENTE UNICA de la que se derivan las
interfaces, no un sitio mas donde repetir el dato.

## Lo que el core encontro al mirarse

La peticion es legitima; el motivo que obligo es interno. La regla de
compatibilidad del core -"MCP y REST no deben tener logica distinta a la CLI"-
no la comprobaba nadie, y la superficie ya habia derivado: `prompt.stream` y
`reasoning.stream` no existen en MCP, `task.run` solo tiene variante bloqueante
en MCP y solo SSE en REST, y `model.pull` -que es contrato publico- solo existe
en CLI.

La logica si era unica (las tres interfaces llaman a la misma capa de servicio);
lo triplicado era el catalogo.

## Que se entrega

- **Catalogo declarativo unico** en el core. Por capacidad: nombre canonico,
  descripcion corta, si transporta identidad, si es streaming, sus PARAMETROS
  (nombre, tipo, obligatoriedad, valores admitidos, defecto) y su proyeccion en
  cada interfaz: ruta y metodo REST, grupo y accion CLI, nombre de herramienta
  MCP, con NULO donde no se expone.
- **CLI y rutas REST se derivan** del catalogo; las herramientas MCP se
  ASERTAN contra el por test (el SDK de MCP deriva su esquema de la firma
  tipada; generarla exigiria sintetizar firmas en ejecucion). El efecto que
  pedia el CR es el mismo: divergir rompe el build.
- **`capability.list` en las TRES interfaces**, no solo REST: `ianest capability
  list`, `GET /capability/list` y herramienta MCP. Exponerla solo en REST habria
  creado la asimetria que el CR venia a cerrar. No requiere identidad.
- **`core_version`** en `capability.list` y tambien en `runtime.health`. Antes no
  era consultable en ejecucion: una capa que fija `core >=0.2 <0.4` no podia
  verificar contra que hablaba.
- **Los huecos se declaran, no se ocultan.** `model.pull` e `init` quedan
  solo-CLI POR DECISION (operacion de operador, seccion 5 de
  `ARQUITECTURA_DE_CAPAS.md`; abrir descargas de gigabytes por red no procede);
  `prompt.stream` y `reasoning.stream` quedan sin MCP por forma del protocolo.
  Se cierra el unico hueco real: `task.run` gana variante bloqueante en REST.

## Mas de lo que se pidio, y por que

El CR decia explicitamente que NO pedia esquema de argumentos. El core lo
publica igualmente: al derivar la CLI del catalogo hay que modelar los
parametros de todos modos, y esos parametros ya eran contrato publico (una
bandera CLI o un campo de cuerpo REST cuentan como tal en
`core docs/VERSIONADO.md`). Publicarlos no anade compromiso: los hace legibles
por maquina. El CLI de extended puede, por tanto, validar argumentos y no solo
pintar ayuda. Ignorarlos es igual de valido.

## Version y calendario

- Impacto: adicion compatible. Por `POLITICA_SEMVER.md` seccion 3 (serie
  pre-1.0) eso es PATCH; se publica dentro de la linea **v0.4.0**, que el
  usuario corta como MINOR por envergadura (precedente `core ADR 0034` y
  `core ADR 0038`).
- Es el tramo A de la linea v0.4 del core, ANTES del tramo B (`task.plan`, ver
  abajo): con el catalogo en pie, `task.plan` aterriza dentro y su paridad queda
  verificada por el gate.
- Sin urgencia impuesta a extended: su lista estatica interina sigue siendo
  valida hasta que la sustituya.

## Nota sobre CR-0001, que viaja en la misma linea

`extended CR-0001` es el tramo B de v0.4. Al preparar su implementacion, el core
detecto que `ADR 0040` (2026-07-27) no era implementable tal cual: es anterior a
`core ADR 0041`, `0044` y `0045`, que cambiaron la etapa PLAN por debajo.
`core ADR 0047` lo enmienda con las tres reglas que faltaban -esfuerzo heredado
del plan, presupuesto concedido al plan suministrado, requisitos que viajan con
el plan o degradacion declarada-. La decision de ADR 0040 no cambia.

## Lo que el core pregunta de vuelta

No se decide por inferencia sobre lo que extended necesita:

1. **Proyeccion CLI.** El CLI de extended derivara el subcomando del nombre
   canonico (`prompt.run` -> `prompt run`) o necesita la proyeccion CLI
   explicita en el payload? El core la publica de todas formas; la pregunta es
   si debe considerarla contrato estable o dato informativo.
2. **Esquema de parametros.** Lo consumira para validar argumentos, ahora que
   va a existir? Si la respuesta es si, el core lo trata como contrato bajo
   SemVer; si es no, se reserva el derecho de afinarlo sin ceremonia.
3. **Round-trip del plan (afecta a CR-0001, no a este).** `ia_nest_extended`
   devolvera a `task.run` el plan INTEGRO que le dio `task.plan` -con sus
   `requirements` y su `effort`- o solo los prompts enriquecidos? Si solo
   devuelve prompts, cada tarea entrara por la via degradada de `core ADR 0047`
   (`requirements_unavailable`), que funciona pero pierde la comprobacion de
   cobertura.
