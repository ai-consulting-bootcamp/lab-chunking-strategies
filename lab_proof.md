# lab_proof — Slice docs the way Google wishes you would

## 1. Code / workflow

- Notebook: `chunking_strategies.ipynb` (run top to bottom with "Run All", kernel `bootcamp-semana2`, Python 3.11.16).
- Sources: `podcast_transcript.txt` (61,278 characters) and `rag_lesson.pdf` (8 pages, 11,150 characters).
- Strategies compared:
  - Fixed-size (`CharacterTextSplitter`, `separator=""`): sizes 500 / 1000 / 2000 characters × overlap 0 / 50 / 100.
  - Recursive (`RecursiveCharacterTextSplitter`): sizes 500 / 1000 / 2000 characters × overlap 0 / 100, plus a custom separator priority.
  - Token-based (`TokenTextSplitter`, `cl100k_base`): 500 and 1000 tokens.
- Retrieval check: TF-IDF + cosine similarity (local, no paid API), top-1 chunk.

**Limitación de datos:** el enunciado pide cargar el audio del pódcast y transcribirlo (pasos 4–5); en su lugar usé una transcripción ya hecha en texto (`podcast_transcript.txt`) y un PDF local (`rag_lesson.pdf`), para trabajar todo en local y sin coste de API.

## 2. Key results

**Overlap check (fixed, 500 characters, podcast).** Prediction before running: chunks ≈ text length ÷ (size − overlap).

| Overlap | Predicted | Actual |
|---|---|---|
| 0 | 61,278 ÷ 500 ≈ 123 | 123 |
| 50 | 61,278 ÷ 450 ≈ 137 | 137 |
| 100 | 61,278 ÷ 400 ≈ 153 | 153 |

**Chunk statistics (selection).**

| Strategy | Doc | # chunks | Avg chars | Min | Max |
|---|---|---|---|---|---|
| Fixed-1000-ov0 | podcast | 62 | 988 | 278 | 1000 |
| Fixed-1000-ov0 | pdf | 12 | 929 | 150 | 1000 |
| Recursive-1000-ov0 | podcast | 67 | 914 | 445 | 998 |
| Recursive-1000-ov0 | pdf | 12 | 928 | 480 | 990 |
| Token-500 | podcast | 30 | 2043 | 833 | 2204 |
| Token-500 | pdf | 5 | 2230 | 1080 | 2666 |

**% of chunks ending in `.` `?` `!` (clean boundaries).**

| Strategy | PDF | Podcast |
|---|---|---|
| Fixed-500-ov0 | 0.0 | 4.9 |
| Fixed-1000-ov0 | 0.0 | 6.5 |
| Recursive-500-ov0 | 12.5 | 26.2 |
| Recursive-1000-ov0 | 25.0 | 35.8 |
| Recursive-1000-sentences (custom separators) | 25.0 | 35.8 |
| Recursive-2000-ov100 | 33.3 | 42.4 |
| Token-500 | 20.0 | 6.7 |

**Where chunks break (last 50 characters, podcast, Fixed-1000-ov0):** `'...it will generate s'`, `'...So, in the TTS gene'` — words cut in half.

**Conclusión:** el troceo recursivo conserva mejor el contexto. El fijo corta cada 500 o 1000 caracteres exactos aunque esté a mitad de palabra («generate s»), mientras que el recursivo busca primero un salto de párrafo o de línea, y por eso sus trozos acaban limpios con más frecuencia (35,8 % frente a 6,5 % en el pódcast, con 1000 caracteres). Token-500 produce trozos de unos 2.000 caracteres, lo que indica que un token equivale a unos 4 caracteres en inglés. Los separadores personalizados no cambiaron nada porque los textos tienen muchos saltos de línea simples (el pódcast, 661 líneas y ningún doble salto): el divisor encuentra `\n` antes de llegar a `". "`, así que corta en los mismos sitios que el recursivo por defecto; y el máximo del 42 % se debe sobre todo a que el pódcast es habla transcrita, donde muchas líneas no terminan en punto, de modo que esta métrica penaliza al pódcast aunque el troceo sea bueno.

## 3. One query and the retrieved evidence

- Query: `"Why does each file do only one function?"`
- Retrieved (Recursive-1000-ov0): **podcast chunk #0, score 0.259**. With Fixed-1000-ov0: also chunk #0, score 0.236.
- Retrieved text (start):
  > So for each and every file will do only one function, and the output of one function given to input to the other. Now the question is, how we can do that, like the output of one dot py…

## 4. Final output

Según el trozo recuperado, cada archivo hace una sola función y la salida de una función se pasa como entrada a la siguiente: `data_processor` lee y limpia el texto, y `llm_processor` recibe ese texto limpio para crear el guion. El trozo describe **cómo** se conectan los archivos, pero no explica **por qué** se diseñó así.

## 5. One failure or limitation

La consulta pregunta **por qué** cada archivo hace una sola función, pero el trozo solo dice **que** lo hace y **cómo** se pasan los datos. Si un LLM respondiera «porque así el código es más fácil de mantener y de probar», sonaría correcto, pero esa razón vendría de su propio conocimiento y no del documento: es una respuesta plausible sin respaldo. Además, la puntuación es baja (0,259) y TF-IDF solo compara palabras, así que recuperar el trozo correcto no garantiza que contenga la respuesta.

## 6. Recommendations

### For PDF documents
**Recommended strategy:** Recursive (`RecursiveCharacterTextSplitter`), 1000 characters, overlap 100.
**Reasoning:** The PDF has paragraphs and headings, and the recursive splitter cuts there first, so ideas stay whole. 1000 characters keep one idea per chunk (good for precise retrieval), and a small overlap protects sentences that fall on a boundary. Fixed-size got 0 % clean endings on the PDF.

### For podcast transcripts
**Recommended strategy:** Recursive, 1000–2000 characters, overlap 100.
**Reasoning:** Spoken text has no clear paragraphs and many lines without a full stop, so ideas run across several lines. Larger chunks with overlap keep the speaker's explanation together; Recursive-2000-ov100 had the best clean-ending rate (42.4 %).

### Trade-offs
| Strategy | Pros | Cons | Best for |
|---|---|---|---|
| Fixed-size | Simple, predictable number of chunks | Cuts words and sentences in half | Quick tests, uniform text |
| Recursive | Respects paragraphs and lines; cleaner boundaries | Chunk sizes vary; depends on the text having separators | PDFs and structured documents |
| Token-based | Matches the model's real limits and cost | Chunks are much longer in characters; also cuts mid-sentence | Controlling context-window size |
| Semantic | (not implemented — optional) | Needs embeddings; slower and more expensive | Topic changes in long conversations |

## 7. AI use (transparency)

Un asistente de IA me guio en la instalación del entorno y en la predicción del overlap, y preparó este documento con los datos reales de mi ejecución del 2/11/2026. Los textos de conclusión (secciones 1, 2, 4, 5 y 6) los redactó el asistente de IA a petición mía, a partir de esos resultados.
