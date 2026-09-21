# Decision 0011: quien autentica, y donde vive esa frontera

Fecha: 2026-09-21

## Decision

1. **Autenticar es propiedad del BORDE, no de una capa.** Autentica la superficie
   mas externa expuesta, y afirma la identidad hacia dentro. Las capas
   interiores siguen consumiendo identidad AFIRMADA, exactamente como hoy.
2. **Invariante de despliegue:** la identidad afirmada solo es creible si llega
   desde el borde autenticador. Toda superficie que no sea el borde debe ser
   inalcanzable salvo a traves de el.
3. **Excepcion declarada, para laboratorio y desarrollo.** Un entorno de
   desarrollo puede saltarse el invariante, con una condicion: **declararlo**.
   Donde, desde cuando, que queda expuesto, con que perimetro alrededor y con
   que caducidad. Un incumplimiento declarado es una decision; uno silencioso es
   una brecha.
4. **No se implementa ahora.** Disparador: el primer consumidor que no sea el
   operador -la GUI, o la capa de administracion-.
5. **Hoy no hace falta ningun Change Request**, porque ningun contrato cambia. El
   CR aparecera si el borde necesita que alguien distinga identidad AFIRMADA de
   identidad VERIFICADA, y no antes.

## Motivo

`CAPAS_FUTURAS.md` registraba este concern con dos hogares candidatos y sin
decidir. Se decide el HOGAR -no el mecanismo- porque la primera superficie
expuesta ya esta aqui, y porque la capa web nacera con esta dependencia encima:
mejor que nazca sabiendolo.

Por que el borde y no una capa intermedia:

- **`user_id` segmenta, no autoriza.** La autoridad la dan el principal en codigo
  (`extended ADR 0002`) y el GRANT del motor (`extended ADR 0010`). Autenticar no
  cambia quien puede escribir que; cambia quien puede hablar.
- **Autenticar en el medio contamina hacia arriba.** Si autenticara una capa
  intermedia, cada capa por encima tendria que decidir si reautentica o confia, y
  esa duda no se resuelve despues.
- **El contrato uniforme lo permite sin tocar a nadie** (meta ADR 0007): el borde
  entra como una capa por encima, reenviando lo que no es suyo.
- **Y una costura sin consumidor se pudre** (`core ADR 0035`). Por eso se fija el
  hogar y se difiere el mecanismo, en vez de construir un login que nadie usa.

## Consecuencias

- El concern deja de estar sin dueno en `CAPAS_FUTURAS.md`: el hogar es el borde.
  El MECANISMO sigue abierto y se elige cuando haya consumidor.
- `ia_nest_extended` mantiene su alcance: no autentica, consume la identidad
  afirmada. No se le anade la frontera.
- La capa de interfaz web, y la de administracion cuando exista, nacen sabiendo
  que el borde es suyo o esta delante de ellas.
- **Estado del laboratorio al tomar esta decision**, bajo la excepcion 3: la capa
  de enriquecimiento y el core se publican en la subred del lab, sin autenticar,
  tras perimetro de red y en desarrollo. Queda declarado en el repo de la capa y
  en las notas de maquina, con su caducidad: **hasta que exista un consumidor que
  no sea el operador**.

## Lo que esta decision NO hace

- No elige mecanismo: ni OIDC, ni mTLS, ni proxy con credenciales, ni tokens.
- No autentica maquina a maquina, que es otro problema y tiene otra respuesta.
- No cambia la forma de la identidad del request ni el contrato de nadie.
- No convierte la autenticacion en autorizacion: seguiran siendo dos cosas.
