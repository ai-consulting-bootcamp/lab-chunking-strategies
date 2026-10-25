# Lab Proof — Slice docs the way Google wishes you would

## Sources and workflow
- Podcast: Kamil Kowalczyk, Episode 01: Hello — why this podcast. The 59.4-second MP3 is downloaded from https://kamil.org.pl/wp-content/uploads/2026/10/kamil-podcast-odcinek-01.mp3. Faster-Whisper base detected Polish and saved a timestamped transcript locally. The audio is not committed.
- PDF: Gmyrek, Berg, and Bescond, ILO Working Paper 96 (2023), https://www.ilo.org/sites/default/files/2024-07/WP96_web.pdf. Open access, CC BY 4.0; 56 pages; 118,225 extracted characters. Page numbers stay in chunk metadata.
- Compare fixed (500/1,000/2,000 characters; overlaps 0/50/100), recursive (500/1,000/2,000 characters; overlap 50), and token-based (500/1,000 tokens; overlap 50) chunks. Inspect size statistics, histograms, and chunk endings.

## Query and path
Query: “O czym będzie podcast?” (“What will the podcast be about?”)
Path: audio → Polish timestamped transcript → RecursiveCharacterTextSplitter (500 characters, overlap 50) → word-overlap retrieval → inspect passage.

## Retrieved evidence
Top answer-bearing result: podcast chunk 1, overlap score 1, timestamp 38–47 seconds:

> Będzie o lokalnych modelach AI, automatizacji codziennej pracy, o narzędziach, które testuje.

The excerpt supports the topic list in the answer. The notebook prints the complete chunk, its source metadata, and score.

## Final output
The podcast will cover local AI models, automation of everyday work, tools the speaker tests, and his conclusions.

## Comparison snapshot
| Source | Strategy | Chunks | Avg chars | Min | Max | Clean endings |
|---|---|---:|---:|---:|---:|---:|
| Podcast | Fixed 500/50 | 3 | 360.7 | 90 | 497 | 33.3% |
| Podcast | Recursive 1,000/50 | 2 | 504.0 | 35 | 973 | 100.0% |
| PDF | Fixed 500/50 | 287 | 447.8 | 65 | 500 | 18.5% |
| PDF | Recursive 1,000/50 | 152 | 779.9 | 70 | 998 | 32.2% |

Recursive splitting gave cleaner endings in these samples. PDF extraction still creates awkward boundaries; the one-minute podcast sample is too small to generalize.

## Recommendation and limitation
Use recursive chunks around 1,000 characters with 50–100 overlap for PDFs, retaining page metadata. Use 500–1,000 characters with 50 overlap for transcripts, retaining timestamps and language. Word overlap is only a ranking signal: a score of 1 is weak evidence, and ASR may mishear words. Verify the cited passage against the audio.

## Reviewer checks
1. Confirm the answer topics occur in the transcript chunk at 38–47 seconds.
2. Check PDF page metadata and chunk tails for lost context.

## Grounding risks
1. Word overlap can retrieve a non-supporting passage.
2. Whisper can mis-transcribe Polish terms.
3. PDF extraction can flatten columns, tables, or headings.

Simplification: use recursive splitting for retrieval; keep fixed and token chunking for comparison.
