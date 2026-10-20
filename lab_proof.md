# Lab Proof – Different Ways to Chunk Podcast and PDF

## 1. Learning Objective

The goal of this lab was to implement and compare different text chunking strategies for a podcast transcript and a PDF document.

I focused on understanding how chunk sizes, overlap, and text boundaries affect the structure of retrieved information and its potential usefulness in RAG applications.

**Main notebook:** [chunking_strategies.ipynb](chunking_strategies.ipynb)

## 2. Data Sources and Workflow

I worked with two source documents:

- **Podcast:** The Blueprint for Trustworthy AI, transcribed using OpenAI Whisper and saved as `transcript.txt` (16,313 characters).
- **PDF:** Ethics Guidelines for Trustworthy AI, published by the European Commission's High-Level Expert Group on AI (156,411 extracted characters).

I implemented three chunking strategies using LangChain:

1. **Fixed-Size Chunking:** Tested 500, 1000, and 2000 characters with different overlap values.
2. **Recursive Character Chunking:** Tested different token limits and separator priorities.
3. **Token-Based Chunking:** Used the `cl100k_base` tokenizer with 300 tokens and 50 tokens overlap.

I compared the resulting chunk counts, character lengths, size distributions, and sentence boundaries.

Semantic Chunking (Step 5) was optional and was not implemented.

## 3. Chunk Quality Results

To evaluate chunk quality, I implemented a function that calculates the percentage of chunks ending with a sentence-ending character (`.`, `?`, or `!`).

| Strategy | Podcast Clean Endings | PDF Clean Endings |
|---|---:|---:|
| Fixed-Size (500 characters) | 9.1% | 2.2% |
| Recursive (1000 tokens) | 25.0% | 30.6% |
| Token-Based (300 tokens) | 7.1% | 6.3% |

Recursive Character Chunking achieved the highest percentage of clean sentence endings for both documents.

However, these results were obtained using different chunk sizes and overlap settings, so they do not represent a controlled retrieval-performance comparison.

## 4. Grounding Test: Query and Retrieved Evidence

To demonstrate grounding, I performed a keyword-based retrieval step against the PDF document.

**Query:**

What are the three components of Trustworthy AI according to the EU Ethics Guidelines?

**Retrieval method:**

I used `PdfReader` to extract text page by page and searched for the keyword `lawful`. The script returned the first matching page and a surrounding text excerpt.

**Retrieved source:**

- Document: Ethics Guidelines for Trustworthy AI
- Location: PDF page 4 (printed page 2)
- Section: Executive Summary

**Retrieved evidence:**

> Trustworthy AI has three components, which should be met throughout the system's entire life cycle

The retrieved passage identifies the three components as:

- **Lawful:** Complying with applicable laws and regulations.
- **Ethical:** Ensuring adherence to ethical principles and values.
- **Robust:** Being technically and socially robust, since AI systems can cause unintended harm.

**Final Answer:**

According to the EU Ethics Guidelines, Trustworthy AI must be lawful, ethical, and robust. These three components are necessary throughout the entire life cycle of an AI system and should work together to ensure trustworthy AI.

This answer was formulated directly from the retrieved PDF evidence rather than generated through an LLM call.

## 5. Failure Cases and Limitations

During the experiments, I observed several limitations.

Fixed-size chunking frequently interrupted words and sentences. For example, a podcast chunk ended with `alg`, while a PDF chunk ended with `althoug`.

Recursive chunking generally preserved natural text boundaries better, but it was not perfect. When prioritizing sentence separators, the PDF heading `A. INTRODUCTION` was incorrectly split, resulting in `. INTRODUCTION`.

The grounding test also had an important limitation: it used keyword-based retrieval rather than semantic search. It worked because the searched keyword appeared in the relevant passage, but different wording could cause the method to miss useful evidence.

Finally, clean sentence endings are only a simple proxy for chunk quality and do not prove that the chunks would produce better RAG answers.

## 6. Final Recommendations

For **PDF documents**, I recommend Recursive Character Chunking with paragraph-first separators because it better preserves the structure of headings and paragraphs in our experiments.

For **podcast transcripts**, I would consider Token-Based Chunking with overlap as a practical starting point because it provides predictable token counts and repeats context between neighboring chunks.

These are starting recommendations rather than proven optimal configurations.

## 7. Conclusion

This lab helped me understand that chunking is not only about dividing text into smaller pieces. The way information is divided can influence how much context is preserved and how useful the retrieved evidence may be.

A useful next step would be to test the different strategies in a complete RAG pipeline and evaluate retrieval relevance and answer quality using the same questions.