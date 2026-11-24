# Decision 0020: el contexto va aparte de lo que dice la persona

Fecha: 2026-11-24
Estado: propuesta (direccion elegida por el usuario el 2026-11-24, H12P8 D5;
pendiente de decidir los puntos abiertos)

Depende de: meta ADR 0007 (regla 3: sin el campo nuevo, como antes), meta ADR
0017 (marco de la conversacion), core ADR 0054 (`turn_context`, efecto), core
ADR 0055 (marco en el turno, preguntar lo que falta).

## Contexto

Extended enriquece cada turno con memoria, RAG, sintesis del hilo y contexto del
interlocutor. Hoy lo pega delante de la frase de la persona, en un solo texto:

    <enrichment_context>
    ## rag
    [wiki_geografia/...] Madrid (Cultura / Deporte): Desde 2003, se celebraba ...
    </enrichment_context>

    en Madrid

El core recibe ese texto como `prompt` y no puede distinguir lo que dijo la
persona de lo que anadio extended. Todo lo que el core decide sobre la frase lo
decide sobre la mezcla:

- La clasificacion de lo que falta (core ADR 0055, D4). En el humo de H12P8, el
  RAG sobre Madrid le hizo pedir el lugar en "en Madrid".
- La clasificacion del efecto, lectura u orden (core ADR 0054, D3). Tiene el
  mismo defecto desde el 2026-11-20.
- El modelo que responde lee un texto del RAG en el mismo mensaje que la persona.
  Puede tomarlo como dicho por ella, tambien si trae instrucciones.

## Decision (propuesta)

1. **El `prompt` es lo que dijo la persona**, tal cual. Extended no le pega nada.
2. **El contexto va como dato aparte**, en una parte nueva de `turn_context`
   (por ejemplo `context`). Es una lista de bloques con su origen (`interlocutor`,
   `memory`, `rag`, `thread`) y su texto, ya recortados al presupuesto por
   extended como hoy.
3. **El core lo pone donde va el contexto de datos**: un mensaje `system`
   aparte, despues de las instrucciones del perfil y antes del mensaje de la
   persona. Es el mismo sitio que el bloque del marco (core ADR 0055, D3).
4. **Todo lo que clasifica mira la frase**: el enrutador, el efecto y lo que
   falta.
5. **Sin la parte nueva, como antes** (meta ADR 0007, regla 3). Extended solo la
   manda si el catalogo del core la declara; si no, pega el contexto como hoy.

Es la practica habitual: las API de chat separan el contexto en mensajes
propios, y A2A y MCP mandan mensajes con partes. No se inventa un mecanismo
nuevo, se generaliza el del marco.

## Puntos abiertos (para decidir)

- **Alcance.** Lo conversacional (`prompt.run`, `prompt.stream`, y
  `domain.route` con la frase limpia) entra seguro. `task.run` tambien pega
  contexto en cada subtarea (`compose_prompt` en extended). Falta decidir si
  entra en este ADR o va despues, cuando el planificador y el combinador del
  core sepan recibir contexto aparte. `reasoning.*` no recibe hoy contexto de
  extended.
- **Presupuesto.** Extended recorta hoy el contexto a un presupuesto de
  caracteres que cuenta el envoltorio. Hay que ver si el core lo cuenta igual o
  si basta con lo que mide extended.
- **Telemetria.** Que guarda el core del contexto: tamano por origen o nada,
  sin el texto, porque la memoria es de la persona.

## Lo que esta decision NO hace

- No cambia que se recuerda ni como se busca (memoria, RAG, hilo).
- No mueve el marco: sigue en `turn_context.frame`.
- No resuelve el catalogo de herramientas (meta `H13`).

## Consecuencias por capa

- Core: parte nueva de `turn_context` y mensaje de contexto. MINOR.
- Extended: deja de pegar el contexto cuando el core declara la parte. MINOR.
- Borde e integraciones: nada.
