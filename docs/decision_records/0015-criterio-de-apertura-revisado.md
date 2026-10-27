# Decision 0015: criterio de apertura revisado (identidad central, punto REST/MCP y TLS)

Fecha: 2026-10-27

Adenda a meta ADR 0013. Hito `H02` de la hoja de ruta (meta ADR 0014).

## Contexto

Meta ADR 0013 abrio el ente a uso real y fijo su criterio de salida antes que
el mecanismo, como debia. Pero ese criterio media tres clientes de la piel
OpenAI con "URL y credencial", y no recogia ni un directorio de identidades
comun ni la cara REST/MCP del borde que el propio ADR describia. Lo que no esta
en el criterio no se construye:

- Meta ADR 0013 no eligio mecanismo de autenticacion (correcto) y tampoco puso
  la identidad central en el criterio (el fallo).
- `edge ADR 0002` (punto 2) dejo el proveedor de identidad como "camino
  declarado" con disparador ("el segundo consumidor que lleve personas"), y
  TLS como aparcado con disparador (punto 3).
- El brief de la primera entrega dejo fuera LDAP, OIDC y TLS: consecuencia, no
  causa.

El resultado fue un webchat bien cerrado (borde v0.1.1) sin las dos cosas que
eran el punto de partida del operador: identidad en un solo sitio y un punto
comun para servicios y agentes.

## Decision

### 1. Tres puntos de partida entran en el criterio de salida

Ninguno es ya un camino con disparador:

1. **La identidad sale de un directorio central de IA_NEST.** Personas,
   servicios y agentes son cuentas del directorio; el borde y los consumidores
   la resuelven alli.
2. **El borde ofrece un punto REST/MCP comun** (reenvio del contrato de
   extended, la cara que meta ADR 0013 punto 1 ya preveia) para servicios y
   agentes, ademas de la piel OpenAI.
3. **TLS en los tramos que cruzan el perimetro**, antes de que entre la
   familia.

### 2. Criterio de salida de la apertura

Amplia el punto 6 de meta ADR 0013; C6 sustituye a su criterio 1. La apertura
esta hecha cuando:

| Id | Criterio | Lo cumple |
|---|---|---|
| C1 | Dar de alta o de baja a una persona en un solo sitio basta para que entre o deje de entrar en Open WebUI y para que el borde le asigne su perfil, sin editar Open WebUI ni la configuracion del borde. | H04, H05, H07 |
| C2 | La configuracion del borde no contiene ninguna identidad: ni personas ni credenciales de consumidor. | H05 |
| C3 | Open WebUI no tiene cuentas locales salvo un admin de emergencia, declarado como tal. | H07 |
| C4 | Con el directorio inalcanzable, toda peticion al borde se rechaza con 503 y queda en la auditoria. El borde no sirve a nadie desde otra fuente (ni fichero ni cache). | H05 |
| C5 | Un cliente MCP generico, con su cuenta del directorio, llama a una capacidad de extended a traves del borde; sin credencial recibe 401; core y extended siguen inalcanzables salvo a traves del borde. | H05 |
| C6 | Cada consumidor (Open WebUI, Home Assistant, Kodi, agente Linux) entra solo con configuracion y una cuenta del directorio, sin tocar el codigo de los demas ni el del borde. | H07 |
| C7 | Un `ldapsearch` por LDAPS desde la red de servicios y desde la critica devuelve personas y grupos; por LDAP en claro no responde. | H04 |
| C8 | Ningun tramo que cruce el perimetro va en claro: navegador a Open WebUI, consumidor al borde, cualquiera al directorio. | H06 |

Siguen vigentes, sin cambios, los criterios 2, 3 (criterio duro del perfil
infantil) y 4 de meta ADR 0013, y su requisito de entrada (punto 7: L5r y L4b).
C8 es puerta de la entrada de la familia (`H08`), junto a ellos.

### 3. Fallo cerrado

C4 lo decide el usuario: sin directorio no entra nadie. El operador conserva
su acceso a extended por loopback en la propia maquina, que no es una
superficie expuesta. Coste aceptado: mientras dure una parada del directorio,
Home Assistant y Kodi no hablan con el ente. Por eso la disponibilidad del
directorio es un requisito para `H03`, no una mejora.

Motivo: cualquier fuente alternativa (fichero de respaldo, cache) es una
segunda copia de las identidades, y rompe C1 y C2 justo en el caso que no se
prueba a diario.

### 4. El orden lo da la hoja de ruta

El punto 5 de meta ADR 0013 ("primero el borde con Open WebUI; despues el
agente Linux") queda sustituido por el orden de `docs/plan/HOJA_DE_RUTA.md`:
directorio (`H04`), borde con identidades del directorio y punto REST/MCP
(`H05`), TLS (`H06`), consumidores uno a uno (`H07`), familia (`H08`). Nacio de
tomar Open WebUI por un protocolo para servicios.

### 5. La leccion

**Si algo es punto de partida, va en el criterio del ADR que abre el trabajo.
Un camino con disparador es un aplazamiento.** Y el criterio se escribe desde
lo que el operador quiere poder hacer, no desde el primer cliente a mano.

Un disparador sigue siendo valido para lo que de verdad puede esperar (meta
ADR 0011 punto 4, `core ADR 0035`). La prueba: si el operador lo da por hecho
al describir el objetivo, no puede esperar.

## Consecuencias

- Meta ADR 0013 recibe un `## Estado posterior` con el puntero a esta decision.
  No se reescribe.
- Los puntos 2 y 3 de `edge ADR 0002` (proveedor de identidad y TLS con
  disparador) quedan superados por esta decision en lo que toca al criterio.
  Lo construido sigue valiendo: el unico punto de resolucion de identidades
  (punto 3) es justo donde entra el directorio, y el TLS del punto 8 es la base
  de C8. Si hace falta rastro en el borde, se le apendiza puntero en su repo,
  como commit de gobernanza.
- Entradas del inventario (`H01P1`): A09 la cubren C1 y C3; A10, C2; A11, C5;
  A12 (agente Linux por la piel OpenAI), C5 y C6, que la revisan en `H07`; A14,
  C8. A13 queda resuelta por el punto 4.
- Las cuentas locales de Open WebUI y las personas y claves en la configuracion
  del borde siguen hasta `H05` y `H07`, como estado de laboratorio. La familia
  no entra con ellas.
- `H03` hereda un requisito: el directorio se disena sabiendo que su parada
  para a todos los consumidores (C4).

## Lo que esta decision NO hace

- No elige producto ni maquina del directorio (`H03`, `H04`), ni como se
  autentican servicios y agentes contra el (clave, bind u otro): solo que sus
  cuentas viven alli.
- No elige mecanismo de autenticacion del borde ni de Open WebUI.
- No define la forma del punto REST/MCP: es diseno del borde (`H05`), dentro
  del contrato uniforme (meta ADR 0007).
- No pide ningun Change Request: ningun contrato cambia.
- No expone nada a Internet.

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Cambia el criterio de salida de
la apertura y el orden de trabajo.
