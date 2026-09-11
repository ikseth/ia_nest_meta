# Decision 0010: que significa "validada", y la puerta de laboratorio

Fecha: 2026-09-11

## Decision

Se anade a `DOCTRINA_MULTI_IA.md` la **regla de la puerta de laboratorio**, que
define lo que hasta hoy cada capa interpretaba por su cuenta:

1. **Una fase que cambia comportamiento observable no se cierra sin una puerta
   de laboratorio**: una ejecucion contra un despliegue REAL, por la superficie
   que consumen las capas de encima, con veredicto.
2. **El criterio se declara ANTES de medir**, y es falsable: que se mide, con
   que valor pasa y con cual no. Un criterio escrito despues de ver el resultado
   no es criterio.
3. **Se ejecuta, no se narra.** Vale un script con codigo de salida; no vale una
   tabla redactada. Un informe que NARRA una verificacion es peor que no
   tenerlo, porque la sustituye en lugar de guiarla.
4. **Se cruza dos veces, por dos agentes independientes.** La discrepancia entre
   las dos pasadas no es ruido: es el hallazgo.
5. **Una fase cuya afirmacion sea sobre la CALIDAD de lo producido necesita un
   control externo** a las senales que emite el propio sistema. Un sistema no es
   su propio criterio de aceptacion.
6. **Lo que la puerta no cubre se declara.** Un "PASA" de forma no es un "PASA"
   de contenido, y decirlo forma parte del resultado.

El PROCEDIMIENTO -que script, contra que host, con que umbrales- es de cada capa
y vive en su repo. Esta regla fija que la puerta existe y que forma tiene, no
como se ejecuta en cada sitio.

## Motivo

El criterio de avance del core dice, y decia solo, esto:

    No se abre una fase nueva si la anterior no esta documentada y validada.

**"Validada" no estaba definido en ninguna parte.** El `PLAN` del core nombra el
laboratorio veintidos veces y cada linea se inventaba su propia puerta; las
convenciones del core llegaron a legislar que hacer SI tocas configuracion al
cerrar una puerta de laboratorio -es decir, se legislo el efecto secundario de
una regla que nunca se escribio-.

Lo que esa indefinicion ha costado, con casos reales y todos del mismo ano:

- Un smoke declarado "sobre main exacto" que por REST no lo fue: el servicio
  llevaba en memoria codigo de veintiun commits antes. Lo destapo la capa de
  encima, no el core, y costo tres horas de medidas invalidas (regla 1 y 3).
- Una linea dada por cerrada en su ficha y muda en el PLAN, que es donde se mira
  para saber si algo esta cerrado (regla 1).
- Un gate de fase que paso 18 de 18 incluyendo dos tareas que no convergieron, y
  mas tarde cuatro respuestas falsas en verde, porque ninguna senal miraba el
  contenido (regla 5).
- Una puerta que la primera vez dio "NO PASA" por un error del operador y no del
  producto, porque exigia calentar el backend a mano antes (regla 3).

Ninguno de esos casos fue un descuido de alguien distraido. Los cuatro son la
misma ausencia: nadie habia escrito que significa validar.

## Por que en el taller y no en cada repo

`DOCTRINA_MULTI_IA.md` dice, en su apartado "Fuera de este documento", que el
METODO de trabajo no se re-hoga aqui porque es metodo y no protocolo entre
agentes. Esta regla es la excepcion, y conviene argumentarla en vez de colarla:

**"Validada" es un termino del que dependen las OTRAS capas.** Cuando una capa
declara una fase cerrada, las de al lado lo creen y construyen encima: el vinculo
por SemVer entre capas (core ADR 0032) se apoya en que "entregado" signifique lo
mismo para todos. Un termino compartido que cada capa interpreta a su manera no
es metodo propio: es vocabulario comun mal definido, que es exactamente lo que
este taller existe para fijar (meta ADR 0008, criterio de ubicacion).

El apartado "Fuera de este documento" se enmienda para decirlo: el metodo general
sigue viviendo en cada repo; lo que se re-hoga aqui es la definicion de validada,
y solo eso.

## Consecuencia

- `DOCTRINA_MULTI_IA.md` gana la seccion y enmienda su apartado final.
- Cada capa referencia la regla desde su criterio de avance; no la copia
  (meta ADR 0008). En el core, `docs/LINEA_DE_ACTUACION.md`.
- La regla 5 ya existia en el core como regla de criterio de salida de su PLAN
  (core ADR 0051): al subir aqui, el core la referencia en vez de conservarla
  por duplicado.
- No hay efecto retroactivo sobre fases ya cerradas. La regla gobierna las que
  se cierren a partir de hoy; revisar las pasadas seria reescribir historia con
  una vara que entonces no existia.

## Impacto de version

Doctrina del taller: no mueve la version de ninguna capa. Cambia lo que se exige
para cerrar una fase, no el contrato de nadie.
