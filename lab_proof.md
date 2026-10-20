# Lab Proof: Different Ways to Chunk a Podcast and a PDF
**Author:** Ayhan Demir · **Notebook:** `chunking_strategies.ipynb` (entry point for the code)

## 1. Workflow
```
INPUT   podcast (.m4a -> Whisper -> transcript.txt) + AI HLEG Ethics Guidelines PDF (41 pages)
DECIDE  retrieval path (embedding similarity, top-3, no LLM rewriting -> extractive answer)
BUILD   chunk with 4 strategies (fixed, recursive, token, semantic) -> measure -> pick recommended chunker
        PDF: Recursive 1000 chars / 150 overlap, per page (page metadata kept)
        Podcast: sentence-aware Recursive 1000 chars / 200 overlap (chunk id metadata)
        embed 215 chunks with all-MiniLM-L6-v2
VERIFY  top-3 chunks printed with score + source + page; string check for the 7 requirement names
EXPLAIN unanswerable query -> retriever still returns confident-looking chunks
```

## 2. Chunking results (from the notebook, Step 6/7)
| Strategy | Doc | Avg chunk size (chars) | Min | Max | # of chunks | % clean endings |
|---|---|---|---|---|---|---|
| Fixed-500 | Podcast | 493 | 261 | 500 | 37 | 5.4 |
| Fixed-500 | PDF | 498 | 258 | 500 | 348 | 3.2 |
| Fixed-1000 | Podcast | 961 | 261 | 1000 | 19 | 15.8 |
| Fixed-1000 | PDF | 997 | 708 | 1000 | 174 | 1.7 |
| Recursive-1000 | Podcast | 957 | 232 | 999 | 19 | 21.1 |
| Recursive-1000 | PDF | 933 | 229 | 998 | 170 | 21.8 |
| Recursive-1000-sent | Podcast | 922 | 189 | 996 | 19 | 100.0 |
| Recursive-1000-sent | PDF | 933 | 229 | 998 | 170 | 21.8 |
| Token-300 | Podcast | 1387 | 1089 | 1503 | 14 | 14.3 |
| Token-300 | PDF | 1478 | 748 | 1724 | 127 | 6.3 |

Figures: `fig_size_distributions.png`, `fig_clean_boundaries.png`, `fig_semantic_similarity.png` · Data: `chunk_stats.csv`, `clean_boundaries.csv`

## 3. Query
**"What are the seven key requirements for Trustworthy AI?"**

## 4. Retrieved evidence
**Hit 1** · score `0.808` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `16`

> 14  Acknowledge that, while bringing substantial benefits to individuals and society, AI systems also pose certain risks and may have a negative impact, including impacts which may be difficult to anticipate, identify or measure (e.g. on democracy, the rule of law and distributive justice, or on the human mind itself.) Adopt adequate measures to mitigate these risks when appropriate, and proportionately to the magnitude of the risk. II. Chapter II: Realising Trustworthy AI This Chapter offers guidance on the implementation and realisation of Trustworthy AI, via a list of seven requirements that should be met, building on the principles outlined in Chapter I. In addition, avail ...

**Hit 2** · score `0.781` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `10`

> 8 Figure 1: The Guidelines as a framework for Trustworthy AI ...

**Hit 3** · score `0.767` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `4`

> identify or measure (e.g. on democracy, the rule of law and distributive justice , or on the human mind itself.) Adopt adequate measures to mitigate these risks when appropriate, and proportionately to the magnitude of the risk. II. Drawing upon Chapter I , Chapter II provides guidance on how Trustworthy AI can be realised, by listing seven requirements that AI systems should meet . Both technical and non -technical methods can be used for their implementation. Key guidance derived from Chapter II:  Ensure that the development, deployment and use of AI systems meets the seven key requirements for Trustworthy AI: (1) human agency and o versight, (2) technical robustness and s afety, ...

## 5. Final output (extractive, from the retrieved text only)
Grounding check: **4/7** requirement names appear verbatim in the retrieved chunks: privacy and data governance, transparency, diversity, non-discrimination and fairness, accountability.
The answer is the list as it appears in Hit 1 (`ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf`, page/chunk `16`). A reviewer can open that page and confirm it.

## 6. Failure / limitation
**Query:** "What fines does the EU AI Act impose for non-compliance?"
The 2019 guidelines predate the EU AI Act and contain no fine amounts, yet the retriever still returns 3 chunks (top score `0.630` vs `0.808` for the answerable query). Penalty figure present in retrieved text: **True**.

**Hit 1** · score `0.630` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `8`

> The law provides both positive and negative obligations, whi ch means that it should not only be interpreted with reference to what cannot be done, but also with reference to what should be done and what may be done. The law not only prohibits certain actions but also enables others. In this regar ...

**Hit 2** · score `0.615` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `8`

> international level already apply or are relevant to the development, deployment and use of AI systems today. Legal sources include, but are not limit ed to : EU p rimary law (the Treaties of the European Union and its Charter of Fundamental Rights ), EU s econdary law (such as the General Data P ...

**Hit 3** · score `0.586` · `ai_hleg_ethics_guidelines_for_trustworthy_ai-en.pdf` · page/chunk `5`

> aim to deter the introduction thereof . They should be seen as a living document to be reviewed and updated over time to ensure their continuous relevance as the technology , our social environments , and our knowledge evolve. This document is a starting point for the discussion about “Trustworthy ...

**Where a plausible but unsupported answer appears:** an LLM given these chunks would likely blend "accountability / redress" language with its own training knowledge of AI Act fines and present it as if it came from the source. Guardrails: a minimum similarity threshold, an instruction to answer "not in the sources" when no chunk contains the asked fact, and mandatory citations (source + page) checked against the retrieved text.

**Chunking-specific limitations:** (1) the clean-endings metric treats PDF headings and bullets without a period as "unclean"; (2) semantic chunks have no size cap, so long ones are truncated by MiniLM's 256 word-piece limit; (3) the Whisper transcript has no speaker labels or timestamps, so podcast evidence is traceable to a chunk id but not yet to a minute in the audio.
