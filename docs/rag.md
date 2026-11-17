# Literature and RAG

> Status: design. The tables for documents and chunks exist; ingestion, embeddings and retrieval do not yet.

RAG (retrieval-augmented generation) is used to **ground and explain**, not to decide. The system finds relevant passages in a curated body of literature and lets a model write only from them, with citations.

## Principles

1. **The model never invents doses, products or authorizations.** Those come from official sources (labels, the national authority) and rules. Papers explain the *why*.
2. **Every statement has a source.** Answers cite concrete documents, and the system checks that the citations exist.
3. **Only text the licence allows.** Without an open licence, only metadata and the abstract are stored, and the answer links to the original.
4. **Applicability matters.** A trial from another region or climate may not apply, so crop, region and date are filters, not details.
5. **Admit ignorance.** Weak or out-of-context evidence produces an answer that says so.
6. **Start small.** One crop, one region, a few hundred curated documents beat millions of badly filtered ones.

## Ingestion (a Python worker, scheduled, offline)

```
 discover ─▶ triage ─▶ get text ─▶ convert ─▶ chunk ─▶ enrich ─▶ embed ─▶ index
```

| Step | What happens |
|---|---|
| Discover | Query open scholarly sources (OpenAlex, FAO AGRIS, Portuguese repositories) by crop and topic, in English and Portuguese. Deduplicate by DOI. Keep the query and date so the run is reproducible. |
| Triage | A small model plus a short human review: relevant to the crop and topic? Useful type (reviews, field trials, technical reports)? Old? Which region and climate? Which licence? |
| Get text | Open full text when legally available; otherwise abstract and metadata only. Paywalls are never bypassed. |
| Convert | Extract structure (sections, tables, captions) and clean headers, footers and hyphenation. |
| Chunk | Split by section, then paragraph, around 300 to 600 tokens with small overlap. Never cut a table in half. The abstract is its own chunk. |
| Enrich | Controlled tags (crop, pest or disease, practice, region, study type), an evidence level, and a short Portuguese summary. Output is schema-validated and the prompt version is stored. |
| Embed | A multilingual model, because questions are in Portuguese and much of the literature is in English. The model name is stored on every chunk. |
| Index | Vector index for meaning, full-text index for exact words, plain indexes for metadata. |

Maintenance: re-run discovery periodically and process only what changed; check for retracted papers and remove them from search.

## Retrieval and answering (real time)

1. **Understand** the question: intent, language, a rewrite with technical terms in both languages.
2. **Filter** by the context: the parcel's crop, region and stage, and only fully indexed, non-retracted documents.
3. **Hybrid search**: vector and keyword results fused with reciprocal rank fusion.
4. **Rerank** to the best handful of chunks, weighting document type, evidence level, age and region match.
5. **Assemble** the context with short ids, title, year, DOI, region and evidence level.
6. **Generate** from that context only, in Portuguese, with citations and explicit limitations. Fixed rules: no doses or products, abstain when unsupported, separate what sources say from inference.
7. **Validate**: every citation refers to a chunk that was provided; no invented doses; optionally a second pass checks each main claim against its cited text.
8. **Log** what was retrieved and cited, for audit and evaluation.

## Evaluation

Without measurement there is no quality. The plan:

- A gold set of 50 to 100 real Portuguese questions for the target crop and region, validated by an agronomist, including questions with **no answer in the sources** (the system must abstain) and out-of-scope questions.
- Retrieval metrics (is the expected document in the top results?) and answer metrics (faithfulness to the context, citation accuracy, correct abstention, no invented doses).
- An automatic judge for scale, with a human-reviewed sample for calibration.
- Run it on every change of model, prompt, chunking or embeddings, and keep results per version.

## Risks and care

- **Copyright and licences**: full text only from permitted open sources; the licence is stored per document.
- **Plant-protection advice** carries legal and environmental responsibility: clear warnings, and the responsible technician as the final decider.
- **Over-extrapolation** from another region or variety: always show the study's origin.
- **Stale information**: authorizations and practices change, so date and official sources are preferred.
- **Invented citations**: validate ids and, where possible, claims.
- **Privacy**: questions and parcels are personal data. Minimise what reaches a model, limit retention, avoid sending exact coordinates unless needed.
- **External dependencies**: scholarly APIs change terms, so keep a local copy of the metadata that matters and the access layer replaceable.
