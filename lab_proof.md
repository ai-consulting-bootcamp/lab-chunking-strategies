# Lab Proof

## Scope and workflow

Executed chunking_strategies.ipynb loads the assigned PDF and podcast, caches Whisper transcription, evaluates fixed/recursive/token configurations, writes metrics and figures, and retrieves inspectable evidence. Semantic chunking is explicitly optional and not run.

## Reproduce

Install requirements.txt in a virtual environment, put the two instructor files in the parent folder (or edit SOURCE in the notebook), configure local .env, and run the notebook from top to bottom. Cached transcription avoids repeat API calls.

## Query

What are the three components of trustworthy AI?

## Retrieved evidence

See results/retrieval_trace.json for complete ranked chunks, cosine scores, character spans, PDF physical page numbers, and approximate podcast timestamps. Source hashes are in data/source_manifest.json.

Selected evidence: pdf-recursive-0004, physical PDF pages [7, 8]; the full output below is an exact substring of both the retrieved chunk and the extracted PDF.

## Final output (extractive, verbatim)

```text
5

benefits that they can bring. To help Europe realise those benefits, our vision is to ensure and scale Trustworthy AI.
Trust in the development, deployment and use of AI systems concern s not only the technology’s inherent
properties, but also the qualities of the socio -technical systems involving AI applications.9 Analogous to questions of
(loss of) trust in aviation, nuclear power or food safety, it is not simply components of the AI system but the system
in its overall context that may or may not engender trust. Striving towards Trustworthy AI hence concerns not only
the trustworthiness of the AI system itself, but requires a holistic and systemic approach, encompassing the
trustworthiness of all actors and processes that are part of the system’s socio -technical context throughout its
entire life cycle.
Trustworthy AI has three components, which should be met throughout the system's entire life cycle:
1. it should be lawful, complying with all applicable laws and regulations;
2. it should be ethical, ensuring adherence to ethical principles and values; and
3. it should be robust, both from a technical and social perspective, since, even with good intentions, AI
systems can cause unintentional harm.
Each of these three components is necessary but not sufficient in itself to achieve Trustworthy AI.10 Ideally, all three
work in harmony and overlap in their operation. In practice, however, there may be tension s between these
elements (e.g. at times the scope and content of existing law might be out of step with ethical norms). It is our
individual and collective responsibility as a society  to work towards ensuring that all three components help to
secure Trustworthy AI.11
A trustworthy approach is key to enabling  “responsible competitiveness”, by providing the foundation  upon which
all those affected by AI systems can trust that their design, development and use are lawful, ethical and robust.
These Guidelines are int ended to foster responsible and sustainable AI innovation in Europe. They seek to make
ethics a core pillar for developing a unique approach to AI, one that aims to benefit, empower and protect both
individual human flourishing and the common good  of society. We believe that this will enable Europe to position
itself as a global leader in cutting -edge AI worthy of our individual and collective trust. Only by ensuring
trustworthiness will European individuals fully reap AI systems’ benefits, secure in the knowledge that measures are
in place to safeguard against their potential risks.
Just as the use of AI systems does not stop at national borders, neither does their impact. Global solutions are
therefore required for the global opportunities and challenges that AI systems bring forth. We therefore encourage
all stakeholders to work towards a global framework for Trustworthy AI, building international consensus while
promoting and upholding our fundamental rights-based approach.

Audience and Scope
These guidelines are addressed to all AI stakeholders designing, developing, deploying, implementing, using or being
affected by  AI, including but not limited to companies, organisations, researchers, public services, government
agencies, institutions, civil society organisations, individuals, workers  and consumers. Stakeholders committed
towards achieving Trustworthy AI can voluntarily opt to use these Guidelines as a method to operationalise their
commitment, in particular by using the practical assessment list of Chapter III when developing, deploying or using
AI systems . This assessment list can also complement – and hence be incorporated in – existing assessment
processes.
The Guidelines aim to provide guidance for AI applications in general, building a horizontal foundation to achieve
Trustworthy AI. However, different situations raise different challenges . AI music recommendation systems do not

9  These systems comprise humans, state actors, corporations, infrastructure, software, protocols, standards, governance, existi ng
laws, oversight mechanisms, incentive structures, auditing procedures, best practices reporting and others.
10  This does not exclude the fact that additional conditions may be(come) necessary.
11  This also means that the legislature or policy-makers may need to review the adequacy of existing law where these might be out
of step with ethical principles.

6
```

## Verification

Executed 70 configurations on both sources. Assertions check nonempty chunks, size ceilings except the intentional paragraph diagnostic, source-span provenance, and retrieved support for lawful/ethical/robust. All figures are saved under results/.

## Failure and limitation

The default paragraph separator can produce oversized chunks when there are no paragraph breaks. See the measured paragraph diagnostic in results/chunk_statistics.csv. Terminal punctuation is not semantic coherence; top-3 lexical phrase coverage on one query is not a retrieval benchmark. No LLM answer is generated: the final output is intentionally extractive.

## Deliverables

chunking_strategies.ipynb; requirements.txt; data/transcript.txt and timestamp JSON; extracted PDF and page JSON; source manifest; results/chunk_statistics.csv; retrieval_probe.csv; boundary_examples.json; figures; retrieval_trace.json; final_output.txt; recommendations.md; environment.json.

## Recommendations

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
| --- | --- | --- | --- |
| Fixed-size | Simple character budget | Cuts words and sentences | Baselines |
| Recursive | Prioritizes structural separators | No guaranteed sentence or heading preservation | Structured text |
| Token-based | Explicit token windows | Can split sentences and Unicode sequences | Context budgets |
| Semantic (not run) | Potential topic-based boundaries | Model dependency and extra computation; unmeasured here | Optional future comparison |

### Interpretation

Clean-ending percentage only checks terminal punctuation. PDF headings and footnotes can lower it without indicating a bad chunk. Sentence-first separators may leave punctuation at the next chunk start. Inspect boundary_examples.json alongside the plots.
The retrieval probe reports phrase coverage at top 3 and retrieved characters. It has one query, no independent relevance labels, unequal context budgets, and lexical matching; it cannot establish a universal winner.
The podcast is explanatory secondary material. ASR timestamps are approximate, ten-minute piece boundaries can lose context, and audio should be checked before relying on a disputed quotation.
