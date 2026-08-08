# RESEARCH_227 - Rate-card v4 research close-out

Date: 2026-08-09

Status: research close-out; no rate-card or policy changes applied

## Executive conclusion

The attempted live OpenRouter refresh produced useful demand evidence but did not
produce a trusted pricing snapshot. The engine failed closed while resolving a
dated ranking identity (notably `z-ai/glm-5.2-20260616`) against the catalog and
endpoint data. This is the correct outcome: no alias was guessed, no market price
was invented, and no proposal was generated from incomplete identity coverage.

The operator-provided top-50 table is therefore treated as **untrusted demand
evidence**, not as a schema-valid snapshot. It supports screening and research
triage only. On the evidence available in this close-out, there are **no new
policy additions ready for proposal**. The only clearly open-weight newcomer with
an MLX path that was independently checked, `mistral/mistral-nemo`, is a dense
12B model and fails the current broad-fleet active-parameter gate; it is not a
coding-specialist policy row either.

The existing four policy records remain supported. In particular,
`nvidia/nemotron-3-nano-30b-a3b` is commercially permitted under the NVIDIA Open
Model License and is **resolved, not blocked**.

## 1. Evidence boundary and attempted live run

The checked-in `live-rankings.json` contains:

- 1,530 daily ranking rows;
- window `2026-07-09` through `2026-08-07` (30 dates);
- 77 distinct `model_permaslug` values after excluding no `other` rows;
- the expected current policy models appearing in the demand data where their
  dated/source identity was available;
- ranking data including `z-ai/glm-5.2-20260616`, whose identity resolution was
  ambiguous during the live fetch.

This file is a record of the attempted source response, not a published engine
snapshot. The engine requires the normalized distinct top-50 cohort, exact
catalog identity coverage, and a complete endpoints response for every selected
model before it publishes a snapshot. Those conditions were not met.

### Reproduction of the authenticated fetch fail-close

On 2026-08-09 I reran the documented fetch command from the repository with the
operator-provided API key supplied only to the child process environment. The
key is intentionally not recorded here. The run used an isolated temporary
output directory and these parameters:

```text
python scripts/openrouter_pricing_engine.py fetch --output-dir <temporary-dir> \
  --top-n 50 --demand-window-days 30 --retries 3 \
  --timeout-seconds 20 --generation-timeout-seconds 900
```

The authenticated command exited with status `2` and failed closed while
resolving the dated ranking identity:

```text
openrouter pricing engine: catalog response cannot uniquely resolve ranked model 'z-ai/glm-5.2-20260616' to an endpoint model id
```

The temporary output directory contained no artifacts (`ARTIFACTS=[]`), so no
snapshot was published. This directly reproduces the GLM-5.2 failure described
above: the engine refuses to guess an endpoint alias, and the failure prevents
the fetch from proceeding to a publishable normalized snapshot.

The supplied top-50 display also contains repeated daily observations rather than
one normalized row per model. Some token magnitudes vary by roughly an order of
magnitude between rows, so the display must not be re-sorted or summed into a
replacement snapshot by hand. The apparent top rows include proprietary/API-only
families and `:free` variants, which are demand observations but not automatically
servable or undercuttable Macprovider products.

### Priced-row movements and proposal status

No live priced-row movement can be reported for any of the four policy models:

| Policy model | Shipped rate versus live proposal | Reason |
| --- | --- | --- |
| `openai/gpt-oss-20b` | Not computed | No validated snapshot was published, so there is no trusted cheapest-active-endpoint input for the undercut formula. |
| `google/gemma-4-26b-a4b-it` | Not computed | Same fail-closed snapshot prerequisite. |
| `nvidia/nemotron-3-nano-30b-a3b` | Not computed | Same fail-closed snapshot prerequisite; its license remains resolved and is not the blocker. |
| `qwen/qwen2.5-coder-32b-instruct` | Not computed | Same fail-closed snapshot prerequisite, including the coding baseline, market cap, and provider-payout floor calculations. |

Accordingly, `compute` was not run and no live proposal artifact exists to
archive. Reporting a numeric movement from raw rankings, a prior proposal, or a
hand-built price would violate the snapshot-digest and endpoint-provenance
contract. The clean hand-back is the exact fetch failure above; after identity
resolution is repaired, rerun `fetch`, then `compute` against that fresh
snapshot and the unchanged policy and rate-card references.

### Fail-closed/data-quality findings

The following separates what the pull actually established from what could not
be established because the run failed closed:

| Requested check | Live-pull result | Operational meaning |
| --- | --- | --- |
| Schema drift | The raw rankings response was readable enough to expose rows and dates, but no schema-valid normalized snapshot was published. The complete catalog/endpoint contract was not established for the selected cohort. | Treat the run as schema/provenance-incomplete; do not bypass allowlists or manually repair fields. |
| Missing endpoints | Endpoint completeness for every selected model was not established; the run stopped during identity/catalog resolution before a complete endpoint-backed snapshot could be produced. | Do not infer a missing endpoint or price. Re-run `fetch` and retain the failed-run logs. |
| Free-only pricing | `:free` ranking variants were present in the raw demand evidence. A free-only endpoint has no positive paid completion benchmark. | Keep the demand row visible for screening, but block it from paid-market undercut calculations. |
| Demand-cohort gaps | The evidence has 1,530 daily rows and a 30-day date range, but it is not itself the engine's normalized distinct top-50 snapshot. Completeness of the required selected cohort cannot be claimed. | Do not sum or re-rank the raw artifact by hand; a successful engine fetch is required before `compute`. |
| Identity resolution | `z-ai/glm-5.2-20260616` could not be safely resolved against the current catalog/endpoint identity. | Fail closed; do not alias it to `z-ai/glm-5-20260211` or another model. |

| Finding | Consequence |
| --- | --- |
| 1,530 rows returned | Useful raw demand coverage, but not proof that the required normalized cohort was publishable. |
| Dated/permaslug identities | A ranking slug can be absent or represented differently in the current catalog. Alias guessing is prohibited. |
| Ambiguous GLM-5.2 resolution | Fetch correctly failed closed rather than mapping it to `z-ai/glm-5` or another identity. |
| Catalog/endpoint coupling | Every selected model needs exact endpoint data; partial coverage cannot become a snapshot. |
| `:free` rows | A free endpoint has no paid market price to undercut; it is blocked for pricing analysis. |
| Duplicate daily observations | Raw rows require engine aggregation; manual top-50 reconstruction would lose provenance and digest guarantees. |
| No published snapshot | `compute` must not be run against a hand-edited or reconstructed snapshot. |

## 2. Candidate screening

The distinct high-demand newcomers observed in the supplied cohort include
`xiaomi/mimo-v2.5-20260422`, `deepseek/deepseek-v4-flash-20260423`,
`tencent/hy3-20260706`, `minimax/minimax-m3-20260531`,
`deepseek/deepseek-v4-pro-20260423`, `z-ai/glm-5.2-20260616`,
`stepfun/step-3.7-flash-20260528`, `moonshotai/kimi-k3-20260715`,
`openai/gpt-oss-120b`, and `mistral/mistral-nemo`.

Most are blocked at this stage because they are proprietary/API identities,
free-only rows, unresolved dated identities, too large for the Mac fleet, or lack
verified MLX/GGUF and commercial-license evidence. No candidate may be admitted
from demand alone.

### Component-2 verification: nearest candidate, not an addition

`mistral/mistral-nemo` has an Apache-2.0 base card and an MLX conversion:

- Base: https://huggingface.co/mistralai/Mistral-Nemo-Instruct-2407
- MLX: https://huggingface.co/mlx-community/Mistral-Nemo-Instruct-2407-4bit
- OpenRouter identity: https://openrouter.ai/mistralai/mistral-nemo

The HF metadata reports approximately 12.25B base-model parameters and an MLX
artifact of approximately 1.91 GB. The model is dense, so active parameters are
approximately 12B. A rough bandwidth-bound estimate is about 8-12 TPS on an
M-base-class system and 25-35 TPS on M-Max-class hardware, before runtime and
context overhead. It therefore exceeds the broad-fleet ≤8B active-parameter
gate, and it is not marked as a coding specialist. It is a blocked near-miss,
not a proposed policy addition.

### Proposed policy additions

```json
[]
```

This empty proposal is intentional. The live fetch did not satisfy the evidence
contract, and no newcomer passed all serving-path, license, hardware, demand,
and commercial-pricing gates without inference. Re-run `fetch` successfully
before considering any addition.

## 3. Existing policy license confirmations

- `openai/gpt-oss-20b`: Apache-2.0 on the model card; commercial use is allowed
  under Apache-2.0 obligations. Source: https://huggingface.co/openai/gpt-oss-20b
- `google/gemma-4-26b-a4b-it`: the Gemma 4 model card identifies Apache-2.0;
  commercial use remains permitted subject to that license and Google’s stated
  terms. Source: https://ai.google.dev/gemma/docs/core/model_card_4
- `nvidia/nemotron-3-nano-30b-a3b`: commercially permitted under the NVIDIA Open
  Model License, subject to its conditions. **Resolved; not blocked.** Source:
  https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-license/
- `qwen/qwen2.5-coder-32b-instruct`: Apache-2.0 on the model card; commercial
  use is allowed under Apache-2.0 obligations. Source:
  https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct

License permission does not replace serving-path verification, performance
benchmarks, or legal review of downstream packaging.

## 4. V4 decision and next action

Keep the current policy unchanged. Do not modify
`phase3-binary/catalog/autotune/rate-card.json`. Repair or clarify the OpenRouter
dated-model/catalog/endpoint identity contract, then run a fresh authenticated
`fetch`. Only a successfully validated snapshot should be passed to `compute`.

## Sources

- OpenRouter daily rankings: https://openrouter.ai/api/v1/datasets/rankings-daily
- OpenRouter catalog: https://openrouter.ai/api/v1/models
- V3 baseline and prior eligibility research:
  `docs/research/RESEARCH_227_RATE_CARD_V3_MEMO.md`
- Attempted-run evidence: `live-rankings.json` (not a trusted snapshot)