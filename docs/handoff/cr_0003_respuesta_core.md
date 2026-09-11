# Respuesta del core a extended CR-0003

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Estado: reconciliado por el usuario (2026-09-11).
Base: `ia_nest_core` en `main`, commit `acc3c8b`, mas los cambios de contrato de
esta respuesta (`core ADR 0052`, sin publicar todavia).
Fecha: 2026-09-11
Responde a: `docs/change_requests/from-ia_nest_extended/solicitado/CR-0003-identificador-de-build-en-la-respuesta.md`

## Resolucion

**ACEPTADO, REFORMULADO.** `core ADR 0052`, decision D1.

La necesidad se acepta entera. La forma sugerida se cambia, y conviene explicar
por que, porque no es una preferencia de estilo.

## Por que la forma sugerida no valia

El CR pide "el identificador del commit desplegado". Verificado antes de
responder: hoy no hay nada -`core_version` sale del manifiesto del paquete, y ni
`install.sh` ni `deploy/setup.sh` escriben informacion de build-, asi que la
premisa del CR es correcta y el hueco es real.

Pero un identificador leido de git describe **el arbol de trabajo**, y el arbol
de trabajo es exactamente lo que mintio en el episodio que motiva este CR: decia
`v0.4.0` mientras el proceso servia codigo de veintiun commits antes. Un
consumidor que hubiera preguntado por el commit habria recibido la misma
respuesta tranquilizadora y falsa.

Dicho de otra forma: el CR pide poder afirmar contra que artefacto mide, y un
commit no describe el artefacto CARGADO, sino el directorio que hay al lado.

## Que se entrega en su lugar

`build_id`, hermano de `core_version`, en `runtime.health` y en
`capability.list`: un digest del PAQUETE QUE EL PROCESO IMPORTO.

- No depende de git: vale instalado desde un wheel y en editable.
- Dos builds de la misma version dan digests distintos, que es la pregunta del
  CR.
- El mismo build en dos maquinas da el mismo digest, que es lo que permite a una
  capa declarar contra que midio.
- Si no se puede calcular, `unknown` explicito, como ya hace `runtime.health` con
  lo que no puede sondear.

Y la distincion que el propio CR pedia preservar pasa a ser CONTRATO, no
costumbre: **`core_version` gobierna el vinculo por SemVer; `build_id` es
observabilidad y no se usa para fijar dependencias.**

## Lo que se pierde, dicho claro

Un digest no le dice a un humano QUE version del codigo esta corriendo. Para eso
esta el repo. El campo responde a si dos cosas son la misma, que es justo lo que
un consumidor remoto no puede averiguar por su cuenta.

Si extended necesita ademas el commit legible, es otra peticion y merece su
propio CR, con su caso de uso: no se cuela dentro de este.

## Version y entrega

Adicion compatible: **PATCH**. Linea "la respuesta que se describe a si misma"
del PLAN del core, con puerta de laboratorio explicita: actualizar el arbol SIN
reiniciar el servicio y comprobar que `build_id` delata al proceso viejo mientras
`core_version` sigue diciendo lo mismo. Si las dos respuestas coinciden antes y
despues, el campo no sirve y se rehace.
