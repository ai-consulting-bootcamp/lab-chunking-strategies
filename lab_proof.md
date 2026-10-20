# Lab Proof – Chunking Strategies for RAG (Podcast + PDF)

## Sources
- PDF: *Ethics Guidelines for Trustworthy AI* (AI HLEG, 2019) – ~156,000 characters, extracted with pypdf
- Podcast: "The Blueprint for Trustworthy AI" – transcribed with Whisper (16,461 characters)

## Code
`chunking_strategies.ipynb` – fixed-size, recursive and token-based chunking, statistics, histogram, and a small retrieval test.

## Key results (clean ending = chunk ends with . ? !)
| Strategy | PDF chunks | PDF clean ends | Podcast chunks | Podcast clean ends |
|---|---|---|---|---|
| Fixed-500 (`separator=""`) | 348 | 3.2% | 37 | 8.1% |
| Recursive-1000 (cleaned, sentences first, `keep_separator="end"`) | 185 | 91.4% | 19 | 100% |
| Token-500 (`cl100k_base`) | 70 | 5.7% | 8 | 25.0% |

Findings that changed the numbers most:
- Default `separator="\n\n"` failed: the PDF has only 3 paragraph breaks → 4 chunks, max 74,680 chars (75× the target); the transcript became 1 chunk.
- Recursive splitting first scored *worse* on the podcast (2.4%) because the period was kept at the start of the next chunk; `keep_separator="end"` fixed it (100%).
- The PDF's 1,961 single line breaks are layout wraps inside sentences. Putting `". "` before `"\n"` raised clean ends from 21.8% to 95.7%; replacing the wraps with spaces gave 91.4% with cleaner chunk text.

## Query
"What are the seven key requirements for trustworthy AI?"

## Retrieved evidence (top 3, text-embedding-3-small, cosine similarity)
| Chunk | Similarity | Requirements found |
|---|---|---|
| 110 | 0.777 | [7/7] |
| 7 | 0.735 | [7/7] |
| 111 | 0.707 | [0/7] |

## Final output (gpt-4o-mini, temperature 0, "answer only from context")
"The seven key requirements for Trustworthy AI are: (1) human agency and oversight, (2) technical robustness and safety, (3) privacy and data governance, (4) transparency, (5) diversity, non-discrimination and fairness, (6) environmental and societal well-being, and (7) accountability (chunks 110 and 7)."

## Limitation / failure mode
[Write what the grounding check showed, e.g.: "The list of seven requirements is split across chunks 110 and 111, so the answer depends on retrieving both. With top-1 retrieval it would be incomplete." Or: "Not all seven terms appear in the retrieved chunks, so the model completed the list from its training knowledge – the answer is correct but not fully grounded in the evidence."]

Other limitations: PDF extraction noise (bullet symbol `\uf0fc`, lost paragraph structure); the transcript has no speaker labels; "clean ending" is a proxy (abbreviations like "e.g." also end with a period).


## Limitation / failure mode
The answer is fully grounded: chunks 110 and 7 each contain all seven requirements (keyword check: 7/7 and 7/7). Two limits remain:

1. **Similarity ≠ relevance.** The third retrieved chunk (111, similarity 0.707) contains none of the requirements; it is the continuation of chunk 110 and only shares vocabulary ("transparent", "AI system"). With a harder question, such chunks can dilute or mislead the answer.
2. **A correct answer does not prove grounding.** The seven requirements are a well-known list, so gpt-4o-mini could have produced the same answer from its training knowledge even if retrieval had failed. This is where the system could give a plausible but unsupported answer. I verified grounding by checking that every requirement appears in the cited chunks; a stronger test would use a question whose answer is specific to this document and not common knowledge.

Other limitations: PDF extraction noise (bullet symbol `\uf0fc`, only 3 paragraph breaks kept); no speaker labels in the transcript; "clean ending" and the keyword check are proxies (abbreviations, word variants).


## Chunking Strategy Recommendations

### For PDF Documents
**Recommended strategy:** Recursive character splitting on cleaned text
- Replace single line-wraps with spaces first (`pdf_clean`), then split with `["\n\n", ". ", "\n", " ", ""]` and `keep_separator="end"`
- Result: 91.4% clean sentence endings vs. 3.2% for fixed-size; chunks mostly 800–1000 chars (histogram)
- Settings: chunk_size 1000 chars (≈ 200 tokens), overlap 100. If exact token budgets matter, use `from_tiktoken_encoder` (300 tokens → 83% clean endings)

### For Podcast Transcripts
**Recommended strategy:** Recursive splitting with sentence separator, `keep_separator="end"`
- The transcript has no line breaks or speaker labels, so sentences are the only usable boundary
- Result: 100% clean endings vs. 8.1% fixed-size and 25% token-based
- Settings: chunk_size 1000 chars, overlap 100–200, because a question and its answer can sit in neighbouring chunks

### Trade-offs Summary
| Strategy | Pros | Cons | Best for |
|---|---|---|---|
| Fixed-size | Simple, predictable size | Breaks words and sentences (3–8% clean) | Quick baselines only |
| Recursive | Respects sentences/paragraphs (91–100% clean) | Needs tuning: separator order, keep_separator, text cleaning | Most documents |
| Token-based | Exact size for the model's context window | Cuts blindly (6–25% clean); 1 token ≈ 5 chars here | Hard token limits |
| Semantic | Splits where meaning changes | Slower, needs embeddings | Long, mixed-topic content |

**Main lesson:** the splitter matters less than understanding the text. Two settings (`keep_separator`, separator order) and one cleaning step moved the results from ~3% to 91–100%.