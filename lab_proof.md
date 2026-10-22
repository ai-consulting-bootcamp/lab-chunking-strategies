# Lab Proof: Chunking Strategies for Podcast and PDF

## 1. Workflow

**Sources**
- PDF: *Ethics Guidelines for Trustworthy AI* (AI HLEG), 156,411 characters / 31,710 tokens, loaded with `pypdf`
- Podcast: *The Blueprint For Trustworthy AI* (.m4a), transcribed with OpenAI `whisper-1` into `transcript.txt`, 16,463 characters / 3,509 tokens

**Path:** load sources -> chunk with four strategies -> compare chunk statistics and clean endings -> embed chunks -> retrieve the top 3 chunks for one query -> inspect whether the evidence answers the query.

**Chunking strategies tested**

```python
CharacterTextSplitter(separator="", chunk_size=500, chunk_overlap=50)          # Fixed-500
RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=50)              # Recursive-1000
TokenTextSplitter(encoding_name="cl100k_base", chunk_size=500, chunk_overlap=50)  # Token-500
semantic_chunks(text, percentile=10)   # sentence embeddings (all-MiniLM-L6-v2), cut at the 10% lowest similarities
```

**Retrieval step**

```python
model = SentenceTransformer("all-MiniLM-L6-v2")

def search(chunks, query, k=3):
    chunk_embeddings = model.encode(chunks, normalize_embeddings=True)
    query_embedding = model.encode(query, normalize_embeddings=True)
    scores = []
    for i in range(len(chunks)):
        scores.append(float(np.dot(chunk_embeddings[i], query_embedding)))
    top = np.argsort(scores)[::-1][:k]
    for i in top:
        print("chunk", i, "| score", round(scores[i], 3))
        print(chunks[i])

search(all_chunks[("PDF", "Fixed-500")], query)
```

## 2. Chunking results

| Strategy | PDF chunks | PDF avg (min–max) chars | PDF clean endings | Podcast chunks | Podcast avg (min–max) chars | Podcast clean endings |
|---|---|---|---|---|---|---|
| Fixed-500 | 348 | 498 (258–500) | 3.2% | 37 | 493 (263–500) | 2.7% |
| Recursive-1000 | 168 | 932 (232–998) | 22.6% | 18 | 958 (307–999) | 16.7% |
| Token-500 | 71 | 2,443 (748–2,776) | 1.4% | 8 | 2,267 (1,576–2,499) | 25.0% |
| Semantic | 104 | 1,490 (2–9,682) | 100% | 36 | 456 (23–2,328) | 100% |

Recursive-1000 with `". "` added as a separator reached 100% clean endings on the podcast (18 of 18 chunks), but stayed at 22.6% on the PDF, because the PDF has a `\n` at the end of almost every line.

## 3. Query

> What are the seven key requirements for Trustworthy AI?

Run against the PDF, Fixed-500 chunks.

## 4. Retrieved evidence (top 3)

| Rank | Chunk | Score | Content |
|---|---|---|---|
| 1 | 117 | 0.811 | Start of Chapter II. Says the chapter gives "a list of seven requirements", but does not list them. Cut mid-word at the end ("technica"). |
| 2 | 7 | 0.800 | The **three components** of Trustworthy AI (lawful, ethical, robust). Not the seven requirements. |
| 3 | 13 | 0.787 | The actual list: "(1) human agency and oversight, (2) technical robustness and safety, (3) privacy and data governance, (4) transparency, (" — cut after requirement 4. |

## 5. Final output

Based only on the retrieved evidence, the answer is incomplete. The evidence supports 4 of the 7 requirements:

1. Human agency and oversight
2. Technical robustness and safety
3. Privacy and data governance
4. Transparency

Requirements 5–7 are not in any retrieved chunk, so they cannot be stated as grounded in this retrieval.

## 6. Failure / limitation

**The chunk boundary split the answer.** Fixed-size chunking cut the list of seven requirements after requirement 4, so the chunk with requirements 5–7 was not retrieved. Retrieval found the right place in the document (chunk 13), but a fixed 500-character cut ignored the list's structure.

A second problem: rank 2 (chunk 7) is a near-miss. It scores high because it shares vocabulary ("Trustworthy AI", "should be met throughout the system's entire life cycle", a numbered list), but it answers a different question (three components, not seven requirements). A high similarity score is not proof that a chunk answers the query.

**Grounding check:** every claim in the final output can be traced to chunk 13. Nothing was added from outside the retrieved text.

**Possible fixes (not tested):** larger chunks or more overlap so the full list fits in one chunk; recursive or semantic chunking that cuts at sentence ends; retrieving more than 3 chunks or adding the neighbouring chunk of each hit.