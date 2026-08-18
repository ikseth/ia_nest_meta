# Avisos al core: hallazgos del ejercicio de la fase 7b de extended

Destinatario: hilo de `ia_nest_core`, rol disenador.
Emite: `ia_nest_extended`, tras verificacion en laboratorio (2026-08-18).
Decide: el core, reconciliado por el usuario. Este documento informa; no decide.

No son Change Requests: ninguno pide una capacidad ni un cambio de contrato. Son
HALLAZGOS medidos. El unico que si cambia contrato salio por su canal,
`extended CR-0003`.

Nota de convencion: aqui no se nombran maquinas, rutas ni contenedores concretos.
Eso es contexto local y no se versiona (convencion transversal 5). Cada hallazgo
se describe por su forma, no por su instalacion.

## Contexto: la fase 7b ya es el consumidor real que la fase v0.4-B3 esperaba

El `PLAN` del core cierra su fase v0.4-B3 cuando "extended ejerce la via de punta
a punta con su RAG por subtarea", por la leccion de su ADR 0035. Eso ya ocurrio:
`task.plan` -> enriquecer cada subtarea con RAG gateado por su dominio ->
`task.run(plan)`, con corpus real y modelos reales.

Medido en tres brazos -sin plan, plan en eco y plan enriquecido- con tres
repeticiones cada uno: el enriquecimiento por subtarea NO degrada la respuesta.
Con n=3 no se afirma que mejore la convergencia. Los datos crudos son de
laboratorio y no se versionan. `plan_attempts` fue 0 y `requirements_covered` true en
todas las pasadas con plan suministrado, porque la capa copia `requirements` y
`effort` intactos y solo edita los `prompt`.

## Hallazgo 1: al gate de la fase B3 le falta una linea

El gate declarado mira tres cosas: `degradations` vacio, `requirements_covered`
true, y `params.effort` igual al planificado.

Medido sobre 18 ejecuciones con plan suministrado: **el gate paso 18 de 18, y dos
de esas 18 no convergieron.** Una corto con `replan_unavailable` y otra con
`max_iterations`. En las dos, las tres lineas daban verde.

Causa: ninguna de las tres mira COMO termino la tarea. Y el corte por
`replan_unavailable` es especifico del camino de plan suministrado, es decir del
camino que este gate existe para validar.

Sugerencia: una cuarta linea, `stop_reason == task_done`. Implicacion honesta: el
gate se vuelve mas estricto y dos de esas dieciocho pasadas habrian fallado, de
modo que la fase no se cierra declarando verde un conjunto que incluye tareas
cortadas. O se cierra con el dato delante, que tambien es una decision legitima,
pero distinta de no tenerlo.

Nota: esa linea no basta para todo. Ver el hallazgo 2.

## Hallazgo 2: una respuesta vaciada por exceso de contexto tambien pasa el gate

Midiendo cuanto contexto aguanta cada subtarea, con un factor controlado -de 0 a
3.000 tokens por subtarea- y tres repeticiones por tamano:

A 3.000 tokens por subtarea la respuesta se desploma a una decima parte, y el
core reporta `task_done`, `requirements_covered` true y `degradations` vacio en
las tres repeticiones.

Dos consecuencias, y la segunda es la que importa al core:

1. La concesion de presupuesto es fija -`base + per_subtask * n`- y no crece
   porque una capa de encima infle los prompts. A 1.500 por subtarea se consume el
   100% de lo concedido; a 3.000, el 133%. Cada token inyectado por subtarea
   cuesta del orden de tres tokens de gasto total, porque entra en la subtarea y
   ademas viaja al combinador.
2. **El core no tiene forma de declarar que la respuesta salio degradada por
   saturacion de contexto.** No es una degradacion de las que ya modela -nada se
   replego- ni un corte. Es una respuesta mala que el contrato describe como
   sana.

No se pide nada: la mitigacion es de la capa que inyecta, y ya esta aplicada con
un tope por subtarea calibrado en 500. Se informa porque el mismo dia el core
anadio la segunda linea de su gate al descubrir que `requirements_covered` false
podia convivir con `degradations` vacio. Esto es el mismo patron un nivel mas
arriba, y el metodo que lo encuentra es comparar contra un control, no mirar el
gate.

## Hallazgo 3: el smoke por REST de la linea v0.4 conviene rehacerlo

El informe de laboratorio del core del 2026-08-18 declara un smoke "sobre main
exacto" tras poner el entorno en el commit de `main`. La suite y la CLI leen del
disco en cada invocacion, asi que esa parte es valida.

Lo que atraviesa la REST, no: el servicio llevaba en memoria el codigo anterior,
porque actualizar el arbol de trabajo no reinicia un servicio ya arrancado. La
diferencia era de veintiun commits e incluia un campo publico anadido a la
respuesta de `task.plan`.

Y es justo la superficie que consumen las capas de encima, que hablan por red y
no por CLI.

Sugerencias, ambas baratas: que el procedimiento de despliegue reinicie los
servicios tras actualizar el codigo, y que el smoke sea un script que se ejecuta
en vez de una tabla que se redacta. Un informe que narra una verificacion es peor
que no tenerlo, porque sustituye la comprobacion en lugar de guiarla. El
identificador de build de `extended CR-0003` cierra la misma puerta por el otro
lado.

## Hallazgo 4: la CLI ignora la variable de configuracion que la REST respeta

La REST resuelve su ruta de configuracion desde la variable de entorno de
configuracion del core cuando no se le pasa una explicita. La CLI no: cae siempre
a su ruta relativa por defecto, aunque la variable este definida y apunte a un
fichero valido.

Verificado: con la variable exportada a la ruta correcta, la CLI sigue fallando
con "configuration file not found" sobre su ruta por defecto.

Es una asimetria entre interfaces, y la regla de compatibilidad del propio core
dice que MCP y REST no deben tener logica distinta a la CLI. Es la segunda
asimetria de este tipo que aparece; la otra esta en el hallazgo 6.

## Hallazgo 5: un endpoint sin resolver se reporta como modelo no disponible

La configuracion del core toma el endpoint del backend de una variable de
entorno. Cuando esa variable no esta resuelta, el endpoint queda vacio, todos los
modelos quedan sin destino y el error que llega al operador es que no hay modelo
disponible para el dominio por defecto.

El sintoma senala al catalogo de modelos; la causa esta en una variable sin
resolver. Un mensaje que nombrara la variable ausente ahorraria el rodeo entero.

Evidencia de que no es un tropiezo aislado: le ocurrio al propio core el
2026-08-18 durante su gate de laboratorio -su informe lo registra como error de
metodo propio- y le volvio a ocurrir al operador el mismo dia por la misma causa.
Dos veces en un dia es un defecto de diagnostico, no mala suerte.

## Hallazgo 6: `task.stream` no acepta plan suministrado, y su funcion interna si

`task.stream` declara en el catalogo `prompt`, `mode` y `effort`. No acepta `plan`
ni `requirements` por ninguna interfaz. Sin embargo, la funcion de servicio que
la sirve SI declara esos dos parametros en su firma, y ninguna interfaz puede
llegar a ellos: son parametros muertos.

Consecuencia para las capas de encima: el enriquecimiento de tareas es
necesariamente BLOQUEANTE. Una capa puede ofrecer `task.run` enriquecida, pero su
`task.stream` reenviada va sin enriquecer, de modo que la misma tarea da
resultados distintos segun se pidan checkpoints o no.

**No se pide cambiarlo.** Extended lo ha declarado como hueco conocido en su
contrato, y por la leccion del ADR 0035 no procede abrir una costura sin
consumidor real: cuando la presentacion la necesite, se pedira por su canal. Se
informa por los parametros muertos, que si son deuda hoy: una firma que promete
algo que ninguna interfaz alcanza acabara induciendo a error a quien la lea.
