# Guión de la presentación (10 minutos, 3 oradores)

Sistema de recomendación de películas basado en NLP — TFI

El hilo conductor sigue las preguntas orientadoras de la consigna (corpus → representación → modelado del usuario → resultados → evaluación y límites), pero hablado como un **informe a un jefe**, no como un cuestionario. Cada orador habla ~3:20.

Las notas entre corchetes `[ ]` son indicaciones de escena, no se dicen en voz alta.

---

## ORADOR 1 — Introducción + Estrategia 1 (LDA)
*Diapositivas 1 a 8 (~3:20)*

**[Diapositiva 1–2: portada y "De qué trata"]**

Buenas. Nos pidieron construir un sistema que, para 14 usuarios, recomiende cinco películas a partir de dos cosas: lo que cada uno ya vio y una frase en lenguaje natural diciendo qué quiere ver ahora. Es un recomendador **basado en contenido**: la película se representa por lo que dice, no por lo que otros usuarios votaron. Para la promoción implementamos y comparamos dos estrategias de representación distintas, y este informe justifica por qué funcionan y dónde fallan.

**[Diapositiva 3: los datos]** *(pregunta del corpus)*

Arrancamos por los datos. El corpus tiene casi 5.000 películas con sinopsis, keywords, género, año y director. No toda esa información sirve igual: las tres señales de contenido sin nulos son sinopsis, keywords y género. Las **keywords** son lo más valioso —son etiquetas ya curadas, como "violencia doméstica" o "psycho-thriller"— así que decidimos representar cada película juntando sinopsis, keywords y género.

**[Diapositiva 4: límites del corpus]** *(limitaciones estructurales)*

Pero el corpus condiciona lo que podemos hacer. Hay tres cosas a tener en cuenta: las keywords mezclan español e inglés, hay títulos duplicados, y siete películas de los historiales no están en el corpus. Lo importante es que esas ausencias **no son aleatorias**: caen todas en los perfiles ambiguos. O sea, los usuarios más difíciles de modelar son, encima, los que menos historial conservan. Esto va a explicar buena parte de los resultados.

**[Diapositiva 5: por qué dos estrategias]** *(pregunta de la representación)*

La pregunta de fondo del trabajo es cómo lograr que películas parecidas queden cerca, y que "parecido" signifique lo mismo para el sistema y para el usuario. Elegimos dos representaciones que entienden la similitud de maneras opuestas: una probabilística e interpretable, y otra semántica basada en un LLM.

**[Diapositiva 6–7: LDA y tópicos]**

La primera es **modelado de tópicos con LDA**, de la Unidad 3. Entrenamos un LDA de 20 tópicos sobre el texto combinado. Cada película queda descripta como una **mezcla de tópicos**, no como una etiqueta única. Y los tópicos salen legibles: hay uno claramente policial, uno bélico, uno de terror, uno de ciencia ficción, uno de drama familiar. Eso es lo que hace a esta estrategia defendible: podemos leer qué representa.

**[Diapositiva 8: modelado del usuario]** *(pregunta del modelado del usuario)*

Al usuario lo representamos en ese mismo espacio. El historial nos dice qué le gustó y la query qué quiere ahora: son señales distintas, así que las combinamos con un peso. Después recomendamos por similitud de coseno en el espacio de tópicos, sacando lo que ya vio. Les paso la palabra para ver cómo da esto.

---

## ORADOR 2 — Resultados de LDA + Estrategia 2 (LLM)
*Diapositivas 9 a 11 (~3:20)*

**[Diapositiva 9: resultados LDA]** *(pregunta de los resultados)*

Veamos dos casos opuestos. Rodrigo es un perfil definido: pide algo sobre corrupción y poder político. Su perfil carga en los tópicos policial, biografía y bélico, y le recomienda cosas como "Canción triste de Hill Street" o "Bajo el fuego". Captura bien el eje crimen-poder, aunque se le escapa el matiz de "basado en hechos reales". Paula, en cambio, es ambigua: dice "no quiero pensar mucho, algo intermedio". Su perfil mezcla romance y documental, señales que se contradicen. La diferencia entre los dos casos no está en el modelo: está en la **señal**. Cuando el perfil no apunta a un lugar único del espacio, no hay representación que lo salve.

**[Diapositiva 10: evaluación]** *(pregunta de la evaluación)*

¿Cómo sabemos si recomienda bien sin tener la respuesta correcta? Usamos una proxy: cuánto se solapan los géneros de lo recomendado con los del historial. Da 0.76 en promedio para los definidos contra 0.42 para los ambiguos. El sistema se comporta como esperábamos, más coherente donde hay señal clara. Pero ojo: esta métrica **no vale igual para todos**, porque en los ambiguos el propio historial es contradictorio. Es un diagnóstico, no una verdad.

**[Diapositiva 11: transición al LLM]**

¿Dónde queda corta LDA? Captura el tema, pero no la intención. No entiende "algo que te haga pensar sobre qué es real", ni las negaciones tipo "que no sea muy larga". Por eso la segunda estrategia es un **LLM local con Ollama**. Trabaja en dos pasos: primero TF-IDF preselecciona un grupo de candidatos, y después el LLM lee historial, query y sinopsis, y elige las cinco con un motivo. Lo limitamos a esos candidatos para que no invente títulos, y forzamos la salida a un esquema para que sea parseable. Con esto paso a los resultados.

---

## ORADOR 3 — Resultados del LLM + Conclusiones
*Diapositivas 12 a 15 (~3:20)*

**[Diapositiva 12: resultados LLM]** *(pregunta de los resultados)*

Sobre los mismos dos usuarios. En Rodrigo, el perfil definido, el LLM recomienda cosas como "Buenas noches y buena suerte", "La verdad oculta" o "El príncipe de la ciudad". Lo interesante es que cuatro de las cinco son específicamente sobre corrupción y poder político, y varias basadas en hechos reales. Ese matiz de "basado en hechos reales" que LDA no distinguía, el LLM sí lo captó: no se quedó en el tema policial genérico, leyó el pedido completo. En Paula, el perfil ambiguo, el LLM verbaliza lo que pedía: elige películas "para reflexionar sin sentirse abrumado", que es exactamente el "algo intermedio" de la query. Pero las elecciones quedan heterogéneas, de un anime a cine de autor: cuando no hay señal clara, el LLM la explica mejor pero no la inventa.

**[Diapositiva 13: comparación]**

Si las ponemos lado a lado: LDA compara tópicos, es interpretable, determinista y rápida, pero de grano grueso. El LLM compara significado, capta matices y negaciones, pero es caja negra, no determinista y más lento, con el riesgo de alucinar. No compiten: LDA aporta estructura que se puede auditar, el LLM aporta comprensión de la query.

**[Diapositiva 14: límites del enfoque]** *(pregunta de los límites)*

Hay un techo que más datos del mismo tipo no resuelven. Sin señal de otros usuarios no hay descubrimiento por gustos parecidos, que es lo que haría el filtrado colaborativo. Una query vaga es vaga para cualquier representación. Y evaluar sin ground truth nos obliga a proxies imperfectas. Lo que sí ayudaría es tener keywords más completas o algún feedback del usuario.

**[Diapositiva 15: conclusión]**

Para cerrar, la recomendación al "jefe": las dos estrategias funcionan bien en perfiles definidos y se degradan en los ambiguos —el límite es la señal, no el método—. Propondríamos usar el LLM como recomendador principal por su lectura de la intención, con LDA como capa interpretable para explicar y auditar. Y para usuarios con poca señal, que el sistema pida más información antes de arriesgar una recomendación pobre. Gracias.

---

### Reparto rápido

| Orador | Cubre | Diapositivas | Preguntas de la consigna |
|---|---|---|---|
| 1 | Intro + presentación LDA | 1–8 | Corpus, representación, modelado del usuario |
| 2 | Resultados LDA + presentación LLM | 9–11 | Resultados, evaluación, representación 2 |
| 3 | Resultados LLM + conclusiones | 12–15 | Resultados, evaluación y límites |
