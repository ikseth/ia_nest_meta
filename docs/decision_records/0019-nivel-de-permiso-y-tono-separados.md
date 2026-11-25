# Decision 0019: nivel de permiso y tono del interlocutor, separados

Fecha: 2026-11-21
Estado: decidido por el usuario (2026-11-21), en meta H12P8, D2

Hito `H12` de la hoja de ruta (meta ADR 0014), paso H12P8. Matiza meta
ADR 0013 (punto 2 y criterio duro 3 del punto 6) y meta ADR 0017 (punto 10).

## Contexto

Desde meta ADR 0013 el borde afirma un perfil, `adult` o `child`
(`interlocutor_profile`). Hoy esa palabra hace tres trabajos:

1. **Permiso.** El core ofrece herramientas por perfil (`tools.allow`; `child`
   sin clima ni ventilador) y el borde limita el menu de `child` al modo
   directo.
2. **Tono.** Extended anade al modelo "El interlocutor es un nino o una nina."
   cuando el perfil es `child`.
3. **Respaldo.** El borde pone `child` a todo el que no esta en el grupo adulto
   del directorio y a quien habla sin que se sepa quien es (criterio duro 3 de
   0013; respaldo infantil de meta H12P5, D3).

El 1 y el 3 son un nivel de permiso minimo, y tiene sentido. El 2 es un hecho
sobre una persona concreta, y no se sigue del permiso. Por la voz de la casa, sin
persona, el ente trata como a un nino a cualquier adulto que hable. Lo senalo el
usuario el 2026-11-21: el nombre y el efecto no corresponden a lo que se quiere.

El motivo del criterio duro 3 de 0013 sigue en pie: por un dispositivo
compartido puede estar hablando un nino, y el ente no lo sabe.

## Practica habitual

- Asistentes de voz comerciales: una voz no reconocida es un invitado con acceso
  reducido; el perfil infantil es otra cosa, que se activa para una persona
  identificada.
- Dispositivos compartidos: con una voz no reconocida se aplican por defecto
  filtros aptos para todos los publicos, sin suponer que quien habla es un nino.
- Control de acceso por roles: el rol por defecto es el minimo, y se nombra por
  lo que puede hacer, no por un grupo de personas.

## Decision

### 1. Nivel de permiso: `adult` o `restricted`

- Sale solo del directorio, como hoy: `adult` si el sujeto esta en el grupo
  adulto y no en el infantil; en cualquier otro caso, `restricted`.
- `restricted` tiene exactamente lo que hoy tiene `child`: las mismas
  herramientas en el core y el mismo menu en el borde. Ningun permiso cambia.
- Quien habla sin persona (la cuenta como respaldo, meta H12P5 D3) queda en
  `restricted`, tenga la cuenta el grupo que tenga. Sigue fallando cerrado.

### 2. Tono: tres casos, aparte del permiso

| Quien habla | Tono |
|---|---|
| Persona validada del grupo infantil del directorio | infantil |
| Persona validada del grupo adulto | neutro |
| Cualquier otro caso (habla la cuenta, persona sin grupo de perfil) | apto para todos |

- "Apto para todos" no dice que quien habla sea un nino: dice que puede ser
  cualquiera de casa, ninos incluidos. Cumple el motivo del criterio duro 3 de
  0013 sin tratar como nino a un adulto.
- El tono orienta la respuesta y nunca concede.
- Lo dicho en la conversacion ("soy papa") no cambia ni el permiso ni el tono
  (criterio 7 de `H12`).
- Sale del directorio, el mismo que ya valida a la persona: no hay dato nuevo ni
  hecho nuevo en entities (meta H12P3, D1, sin cambios).

### 3. Una version de transicion

Las capas que reciben el perfil aceptan `child` como nombre antiguo de
`restricted` durante una version, para que el despliegue no dependa del orden.
Despues se retira.

## Motivos

- **El nombre dice lo que hace.** `restricted` es un nivel de permiso; ser nino es
  un hecho de una persona.
- **No se pierde la proteccion.** El permiso del respaldo es el mismo, y el tono
  apto para todos cubre al nino en un dispositivo compartido.
- **Es la practica habitual** de los asistentes de voz y del control por roles.
- **Sin dato nuevo.** El grupo infantil del directorio ya existe y el borde ya lo
  lee.

## Consecuencias

Entra en la tanda B de `H12` (meta H12P8), que ya toca las tres capas:

- **Borde:** emite el nivel de permiso `adult` o `restricted` y, aparte, el tono.
  La configuracion de grupos y menus cambia de nombre. Lo dispone en su ADR.
- **Core:** acepta `restricted` en la identidad y en `tools.allow` (y `child`
  durante la transicion). Lo dispone en su ADR, desde el CR que corresponda.
  Tocar el core requiere aprobacion del usuario.
- **Extended:** el tono deja de depender del permiso y sigue la tabla del
  punto 2. Lo dispone en su ADR.
- **Laboratorio:** la politica de herramientas del core y los grupos del borde,
  por configuracion.
- **Meta:** el criterio 7 de `H12` habla de una cuenta `restricted`; punteros en
  0013 y 0017.

Los identificadores exactos (nombre del campo del tono y sus valores) los fija
cada capa en su ADR, en ingles y `snake_case`.

## Lo que esta decision NO hace

- No cambia que herramientas tiene nadie.
- No anade grupos al directorio ni hechos a entities.
- No decide como debe tratar el ente a un nino: sigue siendo de conscience
  (meta ADR 0013, punto 2).

## Impacto de version

Ninguno en meta. En las capas, cambio del vocabulario del perfil con una
version de transicion; cada capa lo versiona en la suya.
