# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** ___/___/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Analicé la estructura y los tipos de variables de una base de fidelización de 12 clientes. Después utilicé una base de 400 clientes como población para estimar el ingreso mensual promedio mediante muestreo por conveniencia, aleatorio simple y estratificado. El promedio poblacional fue de 1330,72 USD. En esta ejecución, el muestreo aleatorio simple produjo la estimación más cercana, pero me di cuenta que la selección de clientes y la variabilidad entre sorteos influyen en los resultados. Por eso, recomiendo elegir el diseño según el objetivo del análisis y no únicamente por el tamaño de la muestra.

## Parte A — Clasificación de las variables

Cargué data/clientes.csv en Google Colab con pandas para Python y con la extensión necesaria para R, comprobé que contenía 12 filas y 8 columnas (un identificador y siete variables de interés). Revisé los tipos inferidos y los valores únicos para apoyar la clasificación estadística.

La revisión de los tipos de datos permitió distinguir entre la forma en que Python y R almacenan las variables y su significado estadístico. En ambos lenguajes se identificaron cuatro columnas de tipo numérico y cuatro de texto en la base original. Sin embargo, el tipo de almacenamiento no determina por sí solo la clasificación estadística: por ejemplo, `satisfaccion` se almacenó como entero, pero representa categorías ordinales.

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` |Identificador|Identifica a cada cliente, pero no es una cantidad que se pueda promediar u ordenar numéricamente.|Tipo texto (`str`); 12 valores únicos. |
| `edad` | Cuantitativa discreta en el registro | Está registrada en años cumplidos y toma valores enteros. Si se midiera con precisión temporal, la edad subyacente podría tratarse como continua. | Tipo `int64`; 12 valores únicos. |
| `numero_hijos` | Cuantitativa discreta | Cuenta personas y solo puede tomar valores enteros no negativos. No tiene sentido registrar medio hijo en este conteo. | Tipo `int64`; 5 valores únicos: 0, 1, 2, 3 y 4. |
| `genero` | Categórica nominal | Sus categorías identifican grupos sin un orden natural. | Tipo `str`; 3 categorías: Femenino, Masculino y Otro. |
| `nivel_educacion` | Categórica ordinal | Los niveles tienen un orden educativo: Primaria, Secundaria, Universidad y Posgrado. Las diferencias entre niveles no son cantidades numéricas equivalentes. | Tipo `str` originalmente; 4 valores únicos. Después declaré las categorías ordenadas y comprobé que el mínimo fuera Primaria, con 0 faltantes. |
| `ciudad` | Categórica nominal | Identifica la ciudad de cada cliente; son categorías sin jerarquía. | Tipo `str`; 3 categorías: Quito, Guayaquil y Cuenca.|
| `ingresos_mensuales` | Cuantitativa continua | El ingreso puede expresarse con fracciones de una unidad monetaria, aunque este archivo lo registre en valores enteros. Tiene sentido calcular promedios y diferencias monetarias. | Tipo `int64`; 12 valores únicos. |
| `satisfaccion` | Categórica ordinal | Los números representan niveles ordenados de satisfacción, no cantidades con distancias necesariamente equivalentes. Una puntuación 4 no significa el doble de satisfacción que una puntuación 2. | Tipo `int64`; 4 valores observados: 2, 3, 4 y 5. Declaré el orden de 1 a 5; el mínimo observado fue 2 y no hubo faltantes. |

**P1.** **¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?**

El dtype indicó que `satisfaccion` llegó como un entero (`int64`). Sin embargo, el tipo de almacenamiento no refleja que sea una variable cuantitativa: los números representan categorías ordenadas de una escala de satisfacción. Por eso declaré explícitamente los niveles del 1 al 5 mediante `CategoricalDtype` con `ordered=True`.

Si una variable ordinal queda como texto sin declarar su orden, el programa no conoce necesariamente su jerarquía natural. Por ejemplo, un orden alfabético de los niveles educativos no coincide con el orden educativo. Declararlo permite realizar comparaciones respetando la secuencia correcta.

También probé el error tipográfico Postgrado en lugar de Posgrado. La prueba produjo 2 valores faltantes porque esa etiqueta no coincidía con la categoría utilizada en los datos. Demostrando que los nombres de las categorías deben escribirse bien para evitar datos faltantes por una clasificación incorrecta.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes |
| Variable de interés | `ingresos_mensuales` |
| Parámetro poblacional (media) | 1330,72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

En una situacion real, la empresa podría usar muestreo porque revisar a todos los clientes puede requerir más tiempo y recursos. Una muestra estima el comportamiento poblacional, pero su calidad depende del diseño de selección y variabilidad muestral.

### Los tres diseños

En Python ejecuté los tres diseños con muestras de 20 clientes y comparé sus medias con el parámetro poblacional de 1330,72 USD.

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1348,50 USD | Quito: 20 | +17,78 USD |
| Aleatorio simple (`sample`) | 20 | 1336,00 USD | Cuenca: 8; Quito: 8; Guayaquil: 4 | +5,28 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1287,50 USD | Quito: 10; Guayaquil: 6; Cuenca: 4 | −43,22 USD |

La muestra por conveniencia incluyó únicamente clientes de Quito. El muestreo aleatorio simple seleccionó clientes de las tres ciudades, pero no garantizó que sus proporciones coincidieran con las de la población. El estratificado seleccionó el 5 % de cada ciudad y mantuvo la composición proporcional de la población.

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media | 1336,00 USD | 1405,00 USD | 1235,00 USD | 1491,50 USD | 1288,00 USD |

Las estimaciones variaron entre 1235,00 y 1491,50 USD. Esta diferencia se debe a que cada sorteo selecciona un conjunto distinto de clientes. La semilla 1 produjo una estimación cercana al parámetro, mientras que la semilla 4 produjo una media considerablemente mayor.

También ejecuté los tres diseños de muestreo en **R**. Los resultados obtenidos fueron:

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1348,50 USD | Quito: 20 | +17,78 USD |
| Aleatorio simple (`sample`) | 20 | 1318,00 USD | Cuenca: 5; Quito: 8; Guayaquil: 7 | −12,72 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1226,00 USD | Quito: 10; Guayaquil: 6; Cuenca: 4 | −104,72 USD |

En las cinco ejecuciones del muestreo aleatorio simple en R, las medias fueron de 1318,00; 1287,50; 1233,00; 1385,50 y 1283,00 USD, respectivamente.

Los resultados de Python y R son similares en el parámetro poblacional y en las características generales de los diseños. Sin embargo, las medias de las muestras aleatorias y estratificadas no lo son, porque cada implementación puede seleccionar clientes distintos. En ambos lenguajes, la muestra por conveniencia estuvo formada únicamente por clientes de Quito, mientras que el muestreo estratificado incluyó las tres ciudades dependiendo de su presencia en la población.

Por tanto, los resultados permiten comprobar que las estimaciones dependen tanto del diseño de selección como de los clientes que entran en cada muestra. 

**P2.** **¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?**

El parametro poblacional fue de 1330,72 USD. En Python, la muestra por conveniencia estimó 1348,50 USD; el muestreo aleatorio simple estimó 1336,00 USD y el estratificado estimó 1287,50 USD. El aleatorio simple quedó más cerca del parámetro, con una diferencia de +5,28 USD.

Sin embargo, esto no es suficiente para concluir que este diseño será el ganador. Los resultados dependen de los clientes seleccionados y de cuanto varían las estimaciones entre distinas muestras.

**P3.** **La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?**

El defecto es que seleccioné los primeros 20 registros del archivo, en vez de seleccionarlos de manera aleatoria. Esto mostró 20 clientes de Quito y ninguno de Guayaquil o Cuenca, a pesar de que estas últimas representan el 50% de la poblacion total.

Tomar mas filas no garantiza que el sesgo desaparezca, más aún cuando el archivo esta ordenado por ciudad. Por tanto, aparte de revisar el temaño de muestra también hay que considerar el mecanismo de selección.

**P4.** **El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?**

La variacion entre semillas muestra que una estimación también depende de los grupos de clientes que se seleccionen aleatoriamente. La variabilidad muestral implica que un empresa que toma decisiones a partir de una sola muestra tiene el riesgo de obtener una estimación que se aleja del parámetro poblacional.

Las cinco semillas por sí solas no influyen en la precision estadística del caso, por ello para evaluar la incertidumbre de la estimación, se debe considerar el tamaño de muestra y evitar pensar una única media como un valor exacto. 

**P5.** **¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).**

- **Conveniencia:** la utilizaría en una exploración preliminar o un piloto rápido cuando se requiera una primera aproximación y se sabe que la muestra puede estar sesgada. No la eligiría como única base para generar conclusiones a toda la población.

- **Aleatorio simple:** lo utilizaría cuando se necesite estudiar a toda la población y tenga un listado completo de clientes. Cada cliente puede tener la misma probabilidad de ser seleccionado. 

- **Estratificado:** lo utilizaría cuando sea importante la participación de grupos específicos. Si se selecciona clientes de cada ciudad de acuerdo a su proporción en la población, se obtiene una muestra mejor distribuida y el análisis por subgrupos resulta más sencillo.

## Conclusiones

1. Se clasificaron las siete variables de interés y se justificaron sus tipos según su comportamiento. Además, se comprobó que el tipo de almacenamiento no siempre coincide con la clasificación estadística, como ocurre con `satisfaccion`, que se almacenó como entero, pero representa una variable categórica ordinal. La declaración del orden de las categorías permitió interpretar correctamente las variables ordinales.

2. Una estimación cercana al parámetro no garantiza que un diseño sea el más adecuado. La media poblacional fue de 1330,72 USD y; en la ejecución de Python, el muestreo aleatorio simple obtuvo 1336,00 USD. Sin embargo, las cinco semillas produjeron estimaciones diferentes. Por eso, la comparación debe considerar la variabilidad y como se haga el proceso de selección, no solo la diferencia entre distintas muestras.

3. La representatividad depende también de cómo se seleccionan los clientes. La muestra por conveniencia incluyó únicamente clientes de Quito, mientras que el muestreo estratificado de las tres ciudades según su proporción en la población. Se recomienda a RetailAndina S.A., elegir el diseño adecuado según el objetivo del análisis para obtener conclusiones más acertadas sobre sus clientes. 
