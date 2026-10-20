# Chunking Strategies — Lab Proof

## Workflow
El laboratorio utiliza como fuentes 2 archivos: un pdf y un m4a.
Primero se extrajo el texto del PDF con PdfReader y transcribió el audio con whisper-1, y se guardó en transcript.txt.
Luego para la comparación se aplicó chunking fijo, recursivo y por tokens tanto al PDF como al podcast. Se comparó tamaños, finales con puntuación y cortes concretos.
Se buscó fragmentos del PDF con las palabras lawful, ethical y robust, leí las frases completas y cité sus identificadores en la respuesta.

## Code and experiments
Se trabajó con chunking fijo de **500, 1000 y 2000 caracteres**, con overlaps de **0, 50 y 100 caracteres**,(1000 caracteres no equivale a 1000 tokens).
El recursivo con **500, 1000 y 2000 tokens**, comparando prioridad de estructura y de frases, con overlap objetivo de **50 tokens**.
Por tokens se trabajó: **500 y 1000 tokens**, con overlap de **50 tokens**.
Y se generaron tablas, histogramas y ejemplos de cortes para ambas fuentes
La tabla conjunta y los trade-offs están al final del notebook.
[Notebook con código y resultados](chunking_strategies.ipynb)

## Query and retrieved evidence
¿Para que la IA sea confiable, basta con cumplir uno de los tres componentes: ser lícita, ética o robusta?
Busqué en el PDF, fragmentos que contuvieran los tres términos lawful, ethical y robust. El filtro devolvió 7 candidatos, los cuales revisé manualmente. Aclaro que fue una busqueda por palabra clave, no semántica y la pregunta traducida al inglés quedó registrada pero no se utilizó automáticamente para recuperar los fragmentos.
[Consulta, configuración y fragmentos recuperados](evidence/retrieval_trace.json)

## Grounded answer
Redacté la respuesta manualmente, después de revisar los fragmentos `pdf_recursive_1000_0` y `pdf_recursive_1000_3`
[Respuesta y referencias a los fragmentos](evidence/grounded_answer.md)

## Failure or limitation
Encontrar los tres términos no demuestra su relación: podría interpretarse erróneamente como “uno u otro”. Para evitarlo, revisé las frases completas y cité el fragmento que explica que cada componente es necesario, pero ninguno suficiente por sí solo. Además, el filtro por palabras clave puede omitir pasajes relevantes que utilicen otras expresiones.
