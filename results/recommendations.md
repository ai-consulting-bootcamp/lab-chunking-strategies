## Chunking Strategy Recommendations

### For PDF
Fixed 1000 characters / overlap 50 produced 161 chunks and 1.9% clean endings. Recursive 1000 tokens / overlap 50 produced 38 chunks and 23.7% clean endings. Different units and resulting context lengths confound a direct quality ranking.
The default paragraph splitter produced 105 chunks with maximum 4173 characters, including 52 chunks above the 1000-character target. The source contains 169 paragraph separators.
Recommended starting configuration: recursive splitting, 1000 tokens and 50-token target overlap, paragraph-first separators. Preserve PDF page metadata and inspect section boundaries; line-based extraction does not guarantee heading preservation.
Use token windows when strict model budgets matter; use fixed character windows as a cheap baseline. Increase overlap only if retrieval gains justify repeated context and indexing cost.

### For Podcast
Fixed 1000 characters / overlap 50 produced 18 chunks and 5.6% clean endings. Recursive 1000 tokens / overlap 50 produced 4 chunks and 25.0% clean endings. Different units and resulting context lengths confound a direct quality ranking.
The default paragraph splitter produced 1 chunks with maximum 16313 characters, including 1 chunks above the 1000-character target. The source contains 0 paragraph separators.
Recommended starting configuration: recursive splitting, 1000 tokens and 50-token target overlap, paragraph-first separators. Preserve timestamp spans and inspect conversational transitions; the transcript has no reliable speaker labels.
Use token windows when strict model budgets matter; use fixed character windows as a cheap baseline. Increase overlap only if retrieval gains justify repeated context and indexing cost.

### Trade-offs Summary
| Strategy | Pros | Cons | Best for |
|---|---|---|---|
| Fixed-size | Simple character budget | Cuts words and sentences | Baselines |
| Recursive | Prioritizes structural separators | No guaranteed sentence or heading preservation | Structured text |
| Token-based | Explicit token windows | Can split sentences and Unicode sequences | Context budgets |
| Semantic (not run) | Potential topic-based boundaries | Model dependency and extra computation; unmeasured here | Optional future comparison |

### Interpretation
Clean-ending percentage only checks terminal punctuation. PDF headings and footnotes can lower it without indicating a bad chunk. Sentence-first separators may leave punctuation at the next chunk start. Inspect boundary_examples.json alongside the plots.
The retrieval probe reports phrase coverage at top 3 and retrieved characters. It has one query, no independent relevance labels, unequal context budgets, and lexical matching; it cannot establish a universal winner.
The podcast is explanatory secondary material. ASR timestamps are approximate, ten-minute piece boundaries can lose context, and audio should be checked before relying on a disputed quotation.