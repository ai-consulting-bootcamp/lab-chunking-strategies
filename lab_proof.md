# Lab Proof — Chunking Strategies


## 1. Workflow

**Objective:** Compare different chunking strategies for a PDF document and a podcast transcript to understand how they affect text boundaries and context preservation.

**Implementation:**
- Loaded and extracted text from a PDF document.
- Transcribed a podcast into text.
- Implemented three chunking strategies using LangChain:
  - Fixed-size character chunking (`CharacterTextSplitter`).
  - Recursive character chunking (`RecursiveCharacterTextSplitter`).
  - Token-based chunking (`TokenTextSplitter`).
- Experimented with different chunk sizes, overlaps, and separator priorities.
- Compared chunk counts, character lengths, and sentence-boundary preservation.
- Visualized chunk-size distributions using Matplotlib.

**Observed Results:**

| Strategy | PDF chunks | Podcast chunks | PDF clean endings | Podcast clean endings |
|---|---:|---:|---:|---:|
| Fixed-size (500 characters) | 348 | 37 | 3.16% | 5.41% |
| Recursive (500 characters) | 353 | 37 | 22.38% | 13.51% |
| Token-based (500 tokens) | 71 | 8 | 1.41% | 12.50% |

**Initial Finding:** Recursive character chunking achieved the highest percentage of clean sentence endings for both sources in the tested configurations. However, these results do not establish which strategy would produce the best retrieval performance.

## 2. Test Query

**Question:** What does the document say about human oversight of AI systems?

**Retrieval method:** Keyword search for "human oversight" across the PDF chunks generated using recursive character splitting (500 characters, 50-character overlap).

**Result:** Six matching chunks were found. Chunk 100 was selected for inspection, along with the following chunk (101).

## 3. Source Evidence

**Source:** PDF document — recursive chunk 100.

**Relevant evidence:**

> "The allocation of fu nctions between humans and AI systems should follow human -centric design principles and leave meaningful opportunity for human choice."

> "This means securing human oversight 28 over work processe s in AI systems."

**Additional context:** Chunk 101 discusses supporting humans in the workplace and preventing harm.

**Evidence assessment:** Chunk 100 directly supports the importance of human choice and oversight. However, the retrieved text does not specify particular technical or organizational oversight mechanisms.

## 4. Grounded Answer

**Question:** What does the document say about human oversight of AI systems?

**Answer:**

The document emphasizes that AI systems should support and empower humans rather than manipulate or subordinate them. It recommends human-centric design principles that preserve meaningful opportunities for human choice and ensure human oversight of AI-supported work processes.

**Supporting evidence:** PDF recursive chunk 100.

**Evidence limitation:** The retrieved chunk establishes the importance of human oversight but does not describe specific mechanisms for implementing it.

## 5. Failure or Limitation

**Observed limitation: Incomplete context at chunk boundaries.**

During the experiment, recursive chunk 100 ended in the middle of a sentence:

> "AI systems may also"

The following chunk continued with:

> "fundamentally change the work sphere."

This demonstrates that recursive chunking can still separate related information across chunks, even when structural separators and overlap are used.

Additionally, PDF text extraction introduced formatting artifacts, including broken words such as `"fu nctions"` and `"processe s"`.

**Potential impact on RAG:** Incomplete chunks or extraction errors may reduce retrieval quality and cause the LLM to generate answers based on incomplete information.

**Possible improvement:** Preserve source metadata, improve PDF text extraction, and test whether retrieving neighboring chunks provides sufficient context.

**Verification:** The grounded answer was checked against the actual text of chunk 100 rather than generated solely from the model's existing knowledge.