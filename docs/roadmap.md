# Roadmap

## Done

- Authentication with a replaceable seam, and a domain user created on first login.
- Parcels on a map: draw, edit, delete several at once, assign a crop.
- Field diary per parcel.
- Shared catalogue of crops, procedures and resources, including doses per hectare.
- Literature documents and text chunks, with the legal "full text allowed" switch.
- Roles from configuration, and the suggestions inbox: submit with evidence, duplicates handled, accept through the existing services, reject with a note. Useful with no AI at all.

## Next

**Phase 2: a local model and a first corpus.** Install Ollama, move the database to an image that has vector search, add embeddings to chunks, and ingest around 200 documents for **one crop** (the vine) with a multilingual embedding model.
*Done when* hybrid search returns sensible chunks for ten test questions.

## Planned

| Phase | What | Done when |
|---|---|---|
| 3 | Passive proposer and verifier bots posting through the API, plus a first evaluation set | After a week of runs a reasonable share is accepted and no quote is invented |
| 4 | The assistant tab and on-demand questions, with buttons on the crop, procedure and resource pages ("find missing information") | Answers cite real chunks, and "not enough evidence" appears when it should |
| 5 | A linker bot, a price watcher (supplies and prices), and a comparison of models | A measured improvement on the evaluation set |

## Analytics (to be planned)

A component for turning a farmer's own records into insight. Nothing is designed yet; the first step is to decide which questions matter. Likely candidates:

- Activity per parcel and per crop over the seasons, from the field diary.
- Consumption of inputs (doses applied against what the catalogue recommends) and the cost behind it.
- Comparing a parcel with the catalogue's recommended calendar: what was done, what is late.
- Combining diary data with weather and satellite data once the geospatial workers exist.
- For the technician, how useful the bots are: share of suggestions accepted, rejected or edited, by bot and by model.

## Also on the list

- **Mobile:** a PWA first, then a native wrap of the same React app.
- **Weather and satellite data per parcel** (vegetation index, water balance) from Python workers.
- **Email** for account confirmation and password reset (a no-op sender stands in today).
- **Editing and deleting** catalogue entries, with the roles to protect them.
- **A technician editing a suggestion** before accepting it, keeping the original to measure how often bots need fixing.
- **A self-hosted identity provider** instead of the built-in accounts.

## Why this order

The review workflow came first because it is the part that is useful whatever model is chosen, and the part where design mistakes are cheapest to fix. Models come after there is a safe place for their output to land and a measured way to judge it.
