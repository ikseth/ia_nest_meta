# Aviso al core: independencia del entorno de ejecucion del ente

Destinatario: hilo de `ia_nest_core`, rol disenador.
Emite: `ia_nest_extended`, tras verificacion en laboratorio (2026-08-14).
Decide: el core, reconciliado por el usuario. Este documento informa; no decide.

No es un Change Request: no se pide una capacidad ni un cambio de contrato. Es un
HALLAZGO con una decision de postura detras, que es del core.

Nota de convencion: aqui no se nombran maquinas, rutas ni contenedores concretos.
Eso es contexto local y no se versiona (convencion transversal 5). El hallazgo se
describe por su forma, no por su instalacion.

## Hallazgo

El ente depende, para su backend de modelos, de un despliegue que NO forma parte
del ente: una instalacion anterior y ajena, marcada para retirar.

- El core apunta su endpoint compatible con OpenAI a ese backend.
- `ia_nest_extended` apunta ahi su cliente de embeddings.

Es decir, **no es una dependencia de una capa: es del ente entero**. Retirar esa
instalacion previa sin un paso de desacople deja al core y a la capa de
enriquecimiento sin modelos a la vez.

Lo que si esta resuelto: la capa de enriquecimiento ya provisiona su propio
almacen (PostgreSQL con extension vectorial) en su propio despliegue. El backend
de modelos es lo unico compartido con la instalacion ajena.

## La decision que le corresponde al core

Si el core debe PROVISIONAR su propio backend de modelos por defecto, manteniendo
el endpoint configurable para quien quiera apuntar a uno ya existente. Es decir:
**independiente por defecto, personalizable por opcion**.

Objetivo de fondo del usuario, que conviene tener presente al decidir: el ente
debe ser autonomo. Sus recursos -modelos, almacenes, conocimiento- se instalan
con el y no dependen de ninguna instalacion previa, salvo que se le indique una
existente de forma explicita durante la instalacion.

## Tension honesta, y es del core

El core es deliberadamente AGNOSTICO del backend: declara un adaptador
compatible con OpenAI y un endpoint configurable, y su contrato dice que no
implementa capacidades ajenas al motor. Provisionar un backend por defecto roza
esa frontera.

Puede resolverse situandolo en el INSTALADOR y no en el contrato -instalar no es
una capacidad-, pero esa distincion conviene dejarla escrita, no darla por
supuesta. Es exactamente el tipo de decision que el core debe tomar por si mismo.

## Orden de ejecucion, no negociable

Sea cual sea la forma elegida:

1. **Desacoplar**: backend propio del ente, con los modelos que hoy se usan.
2. **Reapuntar** la configuracion del core y la de la capa de enriquecimiento.
3. **Verificar**: salud del runtime, una inferencia real y una recuperacion de
   memoria de extremo a extremo.
4. **Solo entonces**, archivar o retirar la instalacion previa.

Hacerlo en otro orden deja el ente sin modelos.

## Urgencia

Ninguna inmediata: hoy funciona. Pero es deuda con fecha de caducidad ajena -el
dia que alguien retire esa instalacion previa, el ente se para-, y esa fecha no
la controla el ente.
