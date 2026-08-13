# Decision 0006: frontera de lo versionado y directorios no versionados del ente

Fecha: 2026-08-06

## Decision

La regla 5 de `CONVENCIONES_TRANSVERSALES.md` decia QUE no se versiona ("el
contexto local de cada maquina se mantiene fuera del control de versiones")
pero no DONDE vive lo que se excluye. Cada repo lo resolvia por su cuenta: el
core con `local/`, este taller solo con `.claude/` y `.codex/`. Se fija la
regla completa.

### Criterio: que se versiona

Se versiona el producto y su documentacion normativa: codigo, tests, contratos,
ADR, planes, plantillas de ejemplo. Nada mas.

NO se versiona nunca, en ningun repo del ente:

1. **Salida de ejecucion**: logs, telemetria, trazas, resultados de ejecuciones.
2. **Temporales y artefactos de build**: caches, entornos virtuales,
   empaquetado.
3. **Configuracion efectiva**: la config real de una instalacion. Se versiona
   SOLO su plantilla de ejemplo, mas sus copias de seguridad.
4. **Secretos y entorno**: credenciales, tokens, endpoints reales. Igual que la
   anterior: solo la plantilla.
5. **Contexto de la maquina o de la instalacion**: IPs, hostnames, nombres de
   equipos, arquitecturas de prueba, informes de laboratorio, rutas locales.
6. **Datos personales** de cualquier tipo.
7. **Ajustes de las herramientas de IA**, que ademas son de la herramienta y no
   del repo.

### El patron de la plantilla

Para las categorias 3 y 4, la regla es: **se versiona el ejemplo, se ignora el
efectivo**, y el nombre del ejemplo lo declara. Es el patron que el core ya
usaba sin haberlo escrito:

    config/core.example.yaml   versionado    <- la plantilla
    config/core.yaml           ignorado      <- la instalacion real
    .env.example               versionado
    .env                       ignorado

Un repo que publica su plantilla y oculta su efectivo permite que un tercero lo
instale sin aprender nada del entorno de quien lo desarrolla.

### Donde vive lo excluido

Dos hogares, con dueno distinto, y no se mezclan:

- **`local/`**: contexto operativo privado del repo. Es del USUARIO y neutral
  entre agentes. Ahi van los informes de instalacion, la telemetria, la
  documentacion de trabajo que no se publica y cualquier material compartido
  entre varios agentes. Se ignora la raiz entera (`/local/`), de modo que nada
  de lo que cuelgue pueda escaparse por olvido.
- **`.claude/`, `.codex/` y equivalentes**: ajustes de las herramientas de IA.
  Son de la HERRAMIENTA: las escribe y las reorganiza ella. No se usan como
  almacen de contexto del proyecto.

La distincion importa por el modo multi-IA (`DOCTRINA_MULTI_IA.md`): el
material que describe como un agente entrega trabajo a otro es compartido, y no
puede vivir en el cajon de uno de los participantes. Por eso `local/` es
neutral y las carpetas de herramienta no valen para eso.

### Obligacion por repo

Todo repo del ente ignora, como minimo: `/local/`, las carpetas de herramientas
de IA, sus temporales y artefactos de build, su configuracion efectiva y sus
secretos. Sembrar un repo incluye escribir ese `.gitignore`.

## Motivo

La regla 5 se escribio como prohibicion sin destino, y una prohibicion sin
destino se cumple por disciplina, no por construccion. Dos incidentes de la
misma clase lo mostraron el 2026-08-06 en el core:

- Un documento de contexto local llevaba dias en `docs/`, sin seguimiento pero
  en un directorio versionado: a un `git add .` de publicarse.
- La configuracion efectiva de una instalacion real, con sus copias de
  seguridad, estaba en `config/` sin estar ignorada, en la misma situacion.

Ninguno de los dos se habia publicado, y ninguno se habria publicado por mala
fe: se habrian publicado por un comando rutinario. Un directorio ignorado por
su raiz convierte la disciplina en construccion.

El segundo motivo es de reparto. Fijar el nombre AHORA, con dos repos, cuesta
un ADR; fijarlo cuando existan cinco obliga a mover directorios en todos ellos
y a reescribir las rutas que los citan. En el core, `local/` ya esta citado
desde doce ficheros versionados -plantillas de configuracion, fixtures de
evaluacion, documentacion de despliegue y briefs-, lo que hace de el la
eleccion barata y la unica que no rompe nada.

## Consecuencia

- `CONVENCIONES_TRANSVERSALES.md`: la regla 5 pasa de una frase a la regla
  completa (que se versiona, el patron de la plantilla, los dos hogares y la
  obligacion por repo). El documento sube a 1.1.
- Cada repo del ente revisa su `.gitignore` contra la obligacion minima.
- Este repo mantiene `/local/` ignorado y sigue siendo PUBLICO. Se descarta
  cerrarlo: los repos publicos del ente citan su doctrina por ruta, y un taller
  privado dejaria esas citas sin destino para cualquier lector externo. La
  alternativa correcta no es cerrar el taller, sino que lo privado no entre en
  el.
- Alternativas descartadas:
  - **Usar `.claude/` y `.codex/` como almacen del contexto privado.** Son de
    la herramienta, no del repo; una actualizacion puede reorganizarlas, y el
    material compartido entre agentes no cabe en el cajon de uno de ellos.
    Ademas obligaria a que la plantilla de configuracion publica de una capa
    escribiese su telemetria dentro de un directorio de herramienta, en
    maquinas que quiza no la tengan instalada.
  - **Un nombre nuevo y neutro** (`private/`, `workspace/`). Mas limpio sobre
    el papel, pero obligaria a reescribir las doce citas ya publicadas del core
    a cambio de nada.
  - **Dejarlo a criterio de cada repo.** Es el estado que produjo los dos
    incidentes, y garantiza que cada capa nueva invente su propia convencion.
