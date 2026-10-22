# Decision 0013: abrir el ente a uso real (el borde y la capa agentica)

Fecha: 2026-10-22

## Contexto

Hasta hoy el unico interlocutor del ente es el operador. Se quiere abrirlo a uso
real, todo interno y propio, sin nada expuesto a Internet:

- **Personas**: la familia del operador, desde un webchat, Home Assistant y
  Kodi. El uso abierto de los ninos interesa de forma expresa: es dato para
  `ia_nest_extended` y para `ia_nest_conscience`.
- **Consumidores internos**: Home Assistant que conversa con el ente, el
  recomendador de Kodi, y agentes de solo lectura (Linux y Nextcloud).

La familia es el primer consumidor que no es el operador: dispara meta ADR 0011,
que fijo el hogar de la autenticacion (el borde) y difirio el mecanismo hasta
este momento.

## Decision

### 1. El borde tiene repo propio: `ia_nest_edge`, zona Exterior

Autentica a cada consumidor, afirma la identidad hacia dentro (meta ADR 0011) y
traduce cada conversacion de un consumidor a una sesion de `ia_nest_extended`
(`extended ADR 0014` y `0015`).

Va en la zona **Exterior**, no en la zona Ente. La prueba es la de
`core ADR 0033`: una pieza es del ente si infiere, recuerda, delibera o
presenta. El borde no hace ninguna de las cuatro: decide quien habla con el ente
y en nombre de quien, pero no le da identidad. Consume el contrato publico de
extended y lo re-expone con autenticacion.

No vive en `ia_nest_extended`: autenticar no es enriquecer (meta ADR 0011). No
vive en `ia_nest_agents`: los agentes son consumidores del borde, no su puerta.
No vive en `ia_nest_web`: ver punto 3.

Dos caras, ambas dentro del contrato uniforme (meta ADR 0007):

- **Reenvio del contrato de extended** (REST y MCP), con autenticacion. Es la
  cara de los agentes.
- **Piel en el dialecto OpenAI** (`/v1/chat/completions`, `/v1/models`) como
  extension aditiva (regla 3 de `ARQUITECTURA_DE_CAPAS.md`). Es la cara de Home
  Assistant, Kodi y el webchat.

### 2. El perfil infantil: el borde lo afirma, el juicio es de conscience

Decidir como debe comportarse el ente con un nino es control etico, y eso es de
`ia_nest_conscience` (`core ADR 0034`). Hasta que exista:

- El borde **afirma** el perfil (adulto o infantil) como atributo de la
  identidad, y garantiza que ninguna peticion infantil sale sin el.
- **Como** se aplica el perfil vive en la configuracion del borde, declarado
  como provisional y con destino escrito: conscience, cuando exista.

Asi el borde no dirige el pensar del ente (`core ADR 0033`) y el criterio duro
no espera a una capa sin codigo.

### 3. La web sigue siendo una capa del ente, aun prevista

`ia_nest_web` es la cara del ente (`core ADR 0033`): toda la interfaz de usuario
y de administracion, incluida la del propio borde. Se mantiene en "prevista" y
se siembra cuando haga falta mostrar algo que una interfaz de terceros no sepa.

Mientras, el webchat lo cubre **Open WebUI**, local, tras el borde y con todo lo
que sale a Internet apagado (busqueda web, telemetria). Sus valoraciones de
usuario son dato para extended y conscience.

Frontera falsable entre las dos: sustituir Open WebUI por `ia_nest_web` no edita
ningun fichero de `ia_nest_edge`. Si hay que editarlo, el borde ha absorbido
parte de la cara.

### 4. La capa agentica tiene repo propio: `ia_nest_agents`, zona Exterior

Ya figuraba en `REGISTRO_CAPAS.md`. Se le da forma: un gestor (registro de
agentes, policy y auditoria) y, por este orden, un agente Linux y un agente
Nextcloud, los dos de solo lectura. Consume el ente a traves del borde, con su
propia credencial. Base previa: AGENTS-001 y el plano de agentes Linux del repo
`ia_nest` antiguo.

Cubre en parte el concern "quien decide y quien orquesta" de
`CAPAS_FUTURAS.md`: el bucle de un encargo que llega desde fuera es del agente
que lo lleva. Lo que el ente hace por iniciativa propia sigue sin dueno, y el
agente no dirige COMO piensa el ente (`core ADR 0033`).

### 5. Orden: el borde primero

Primero el borde con Open WebUI; despues el agente Linux. Motivo: el borde es el
disparador de meta ADR 0011, se puede construir y probar en paralelo al hito
siguiente de extended, y los agentes nacen ya autenticados en vez de hablar con
extended por la excepcion del laboratorio.

### 6. Criterio de salida del borde

Se fija antes que el mecanismo. El borde esta hecho cuando:

1. Home Assistant (conversacion), el recomendador de Kodi y Open WebUI responden
   a traves del borde cambiando solo su configuracion (URL y credencial), sin
   codigo propio en ninguno.
2. Desde otra maquina de la red de casa no responde ninguna superficie del core
   ni de extended; solo el puerto del borde.
3. **Criterio duro.** Ninguna peticion de una identidad con perfil infantil sale
   del borde sin el perfil infantil aplicado. Si el borde no puede determinar el
   perfil -un dispositivo compartido, una identidad sin perfil-, aplica el
   infantil. Se comprueba en la auditoria del borde; una peticion infantil sin
   perfil es un fallo, no un aviso.
4. Dos conversaciones distintas nunca comparten sesion de extended, y una misma
   conversacion conserva su sesion entre turnos.

El mecanismo de autenticacion y la traduccion de conversaciones a sesiones se
deciden en `ia_nest_edge`, no aqui.

### 7. Requisito de entrada para personas que no sean el operador

El borde y Open WebUI se construyen y prueban antes, pero la familia no entra
hasta que extended cierre dos lineas de su puerta de laboratorio (meta ADR 0010),
que se arreglan en extended y no desde aqui:

- **L5r**: una frase de cortesia no trae fragmentos del corpus de conocimiento.
- **L4b**: si el usuario se corrige, el sistema se queda con la correccion.

## Consecuencias

- `REGISTRO_CAPAS.md` gana la fila de `ia_nest_edge` (prevista) y precisa las de
  `ia_nest_agents` y `ia_nest_web`.
- El concern de autenticacion sale de `CAPAS_FUTURAS.md`: ya tiene repo. El de
  "quien decide y quien orquesta" queda con su estado precisado.
- **La excepcion del laboratorio de meta ADR 0011 (punto 3) caduca** cuando entre
  el primer usuario que no sea el operador. Desde entonces el criterio 2 es
  invariante de despliegue, no objetivo.
- Riesgo aceptado: el laboratorio es tambien el banco de medidas de extended, y
  con usuarios reales sus memorias y su trafico se mezclan con las pruebas.
  Mitigacion: `user_id` propios para pruebas y aviso a la familia antes de cada
  parada.
- Los repos sin uso del webchat antiguo (Jarvis) quedan para archivar: Open
  WebUI los sustituye.

## Lo que esta decision NO hace

- No elige mecanismo de autenticacion.
- No decide que capacidad de extended atiende `/v1/chat/completions`. Una
  correspondencia fija suple la pregunta 1 de "quien decide" de
  `CAPAS_FUTURAS.md`, pero no la resuelve.
- No pide ningun Change Request: ningun contrato cambia. El obstaculo de
  re-planificar con enriquecimiento (`extended docs/PLAN.md`, Fase 7b) llegara
  como CR al core cuando un agente necesite iterar, no antes.
- No expone nada a Internet.

## Impacto de version

Ninguno en el contrato publico de ninguna capa. Registra repos nuevos y un orden
de trabajo.
