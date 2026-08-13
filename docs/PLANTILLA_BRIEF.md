# Plantilla de brief para el codificador

Estado: activo
Version: 1.0 - 2026-08-06

Esqueleto del brief con el que un disenador pasa trabajo a un codificador en
modo ciego (`DOCTRINA_MULTI_IA.md`, seccion Handoff). La doctrina fija QUE debe
llevar un brief; esta plantilla fija su FORMA, para que no haya que recordarlo
de memoria en cada entrega.

No sustituye a la doctrina ni anade obligaciones nuevas: es la misma lista,
escrita como formulario.

## Que NO va en un brief

- Diseno por decidir. Un brief se escribe sobre contrato YA reconciliado. Si al
  redactarlo aparece una decision sin tomar, el brief no esta listo: se
  reconcilia primero.
- Alcance abierto. "Y de paso mejora lo que veas" convierte al codificador en
  disenador, que es justo lo que el modo ciego evita.
- Detalles de la maquina o del canal por el que se entrega (rutas, credenciales,
  cuotas). Eso es contexto local y no se versiona (regla 5 de
  `CONVENCIONES_TRANSVERSALES.md`).

## Plantilla

    # Brief: <titulo corto de la tarea>

    Destinatario: <rol codificador>
    Autor: <rol disenador>
    Verifica: <quien revisa la entrega; NUNCA el mismo que implementa>
    Fecha: <AAAA-MM-DD>

    ## Base

    Repo y punto de partida: <repo, rama o commit>
    Estado de contrato: <ADR/ficha ya reconciliados que gobiernan la tarea>

    ## Lectura obligatoria

    <lista corta y VINCULANTE; si un documento esta en el working tree sin
    commitear, se dice explicitamente>

    ## Objetivo

    <una o dos frases: que debe ser cierto al terminar>

    ## Alcance

    Entra:
    - <...>

    NO entra:
    - <...>
    - <ficheros de otras reconciliaciones en curso que no se deben tocar>

    ## Criterios de aceptacion

    - <falsables, verificables por quien revisa, no por quien implementa>

    ## Entrega

    <como se deja el trabajo y que se lista al terminar>

    ## Regla que manda sobre las demas

    Ante ambiguedad, PARA y pregunta. No rellenes huecos por inferencia:
    eso introduce diseno no reconciliado.

## Notas de uso

- **Los criterios de aceptacion son el contrato de la entrega.** Si no son
  falsables, la revision se vuelve una opinion y el modo ciego pierde su
  garantia.
- **La seccion "NO entra" es la que mas trabajo ahorra.** Nombrar los ficheros
  de otras reconciliaciones en curso evita que una entrega pise trabajo ajeno
  (regla de la inconsistencia, `DOCTRINA_MULTI_IA.md`).
- **La verificacion no la hace quien implementa.** Es doctrina, no preferencia.
- El canal concreto por el que el brief llega al codificador -conversacion,
  fichero, invocacion automatizada- es indiferente a esta plantilla y no se
  describe aqui.
