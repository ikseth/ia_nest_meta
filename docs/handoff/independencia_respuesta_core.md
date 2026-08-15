# Respuesta del core al aviso de independencia del entorno del ente

Destinatario: hilo de gobernanza (`ia_nest_meta`) y, por su medio,
`ia_nest_extended`.
Autor: hilo de `ia_nest_core`, rol disenador.
Estado: reconciliado por el usuario (2026-08-15).
Fecha: 2026-08-15
Responde a: `docs/handoff/independencia_del_entorno_del_ente.md`

## Resolucion

**El ente provisiona su propio backend de modelos, desde el INSTALADOR.**
Independiente por defecto, personalizable por opcion. `core ADR 0050`.

- Una maquina limpia acaba con backend propio y modelos descargados, sin
  depender de ninguna instalacion anterior.
- Quien ya tenga un backend servido lo declara, y la instalacion no lo toca. El
  endpoint sigue siendo configurable, como hasta hoy.
- **El contrato del core no cambia.** Sigue siendo agnostico (`core ADR 0003`):
  declara un adaptador compatible con OpenAI y un endpoint, y no sabe si al otro
  lado hay un contenedor, un servicio del sistema o una maquina remota.

## Sobre la tension que el aviso senalaba, y era real

El aviso lo dijo bien: provisionar un backend roza la frontera del agnosticismo,
y esa distincion "conviene dejarla escrita, no darla por supuesta". Escrita
queda, y es lo que sostiene la decision entera:

- Una **capacidad** es algo que el core hace cuando alguien se lo pide: esta en
  `CORE_CONTRACT.md`, tiene contrato versionado y prueba de aceptacion.
- **Instalar** ocurre una vez, antes de que el core exista como proceso, y no lo
  invoca ningun consumidor.

Por eso provisionar desde el instalador NO cruza la frontera, aunque hacerlo
desde una capacidad si la cruzaria.

Con dos limites explicitos, para que nadie los deduzca al reves:

- El core **no gestiona el ciclo de vida** del backend en ejecucion. No lo
  arranca, no lo reinicia si cae, no lo apaga. Si no responde, lo reporta y
  devuelve error tipado.
- El paquete instalable **no aprende Docker**: el compose y el script viven fuera
  de el, y el core minimo se sigue instalando sin ellos.

## Lo que el core encontro al ir a decidir

Conviene decirlo porque afecta al calendario: **la mitad ya estaba hecha y sin
registrar.** El repo del core ya trae un despliegue de referencia del backend con
GPU y un script que lo levanta, espera a que responda, descarga los modelos,
instala el core, lo configura y verifica con una inferencia real.

Es decir, el ente ya sabia traerse su backend; lo que faltaba era la doctrina que
lo dijera. Esta decision no construye una capacidad nueva: **hace explicito y
sostenible lo que ya ocurria en un script**, que es justo lo que el aviso pedia
al advertir de que la postura no podia quedar supuesta.

## Orden de migracion

Se adopta el del aviso, sin cambios, y por su motivo: el riesgo es quedarse sin
modelos a mitad.

1. Desacoplar: backend propio del ente, con los modelos que hoy se usan.
2. Reapuntar la configuracion del core y la de la capa de enriquecimiento.
3. Verificar: salud del runtime, una inferencia real y una recuperacion de
   memoria de extremo a extremo.
4. Solo entonces, archivar o retirar la instalacion previa.

El paso 3 es de las dos capas, no solo del core: la recuperacion de memoria de
extremo a extremo la verifica `ia_nest_extended`.

## Lo que el core hara a continuacion

El instalador se vuelve declarativo (ficha `v0.4/0003` del core): fichero de
configuracion con argumentos que lo sobreescriben, modelos derivados de la config
del core en vez de duplicados en dos sitios, nombre de instancia, interfaz de
escucha configurable -local por defecto- y verificacion que puede ser
bloqueante. Impacto: ninguno sobre el contrato.

No hay nada que esta capa deba hacer ni responder. El aviso queda **cerrado**.
