# Gestalt Syntax — Test Results
**Status: Ongoing**

---

## Methodology

This document contains two generations of test results:

**Generation 1 — Character-based estimation (initial)**
Token counts approximated using character-to-token ratios (~4 chars per token for English/code, ~1.5 for Chinese). Used for early exploration and directional findings. Results are preserved for posterity and marked with `~` to indicate estimation.

**Generation 2 — Deterministic tokenizer testing (current)**
Token counts produced by running actual tokenizer libraries against document files. Results are exact counts, not estimates. Community contributed test scripts — see `tests/STRESS_TEST.md` and `tests/AUDIT_SYNTAX.md`.

*Thank you to the community member who contributed the deterministic test scripts and the tokenizer methodology that made Generation 2 possible.*

---

## Deterministic Results Summary

Results from `gestalt_bench.py` run against actual repo files across four major tokenizer families.

### SYNTAX_SPEC.md vs GST_SYNTAX_SPEC.md

| Tokenizer Family | GST Tokens | Control Tokens | Compression |
|---|---|---|---|
| OpenAI (o200k / GPT-5) | 2455 | 3029 | **19.0%** |
| Meta (Llama 3.1) | 2436 | 3035 | **19.7%** |
| Google (Gemma 3) | 2161 | 3034 | **28.8%** |
| Mistral (Codestral) | 3252 | 3791 | **14.2%** |
| **Cross-model average** | | | **~20.4%** |

### CODE_EXAMPLE.md vs GST_CODE_EXAMPLE.md

| Tokenizer Family | GST Tokens | Control Tokens | Compression |
|---|---|---|---|
| OpenAI (o200k / GPT-5) | 2263 | 4808 | **52.9%** |
| Meta (Llama 3.1) | 2253 | 4801 | **53.1%** |
| Google (Gemma 3) | 2160 | 5890 | **63.3%** |
| Mistral (Codestral) | 2860 | 6253 | **54.3%** |
| **Cross-model average** | | | **~55.9%** |

### Key observations from deterministic testing

- OpenAI and Meta produce nearly identical results — compression behavior appears to be a property of the content, not the tokenizer
- Google Gemma consistently shows the strongest compression across both tests
- Mistral Codestral, optimized for code syntax, tokenizes Gestalt delimiters less efficiently than general-purpose tokenizers — this is consistent with the delimiter fertility finding below
- NLP compression (~20%) is materially lower than code compression (~55%) under real tokenization
- Our initial character-based NLP estimate of 36% was optimistic; our code estimate of 63% was accidentally accurate

---

## Delimiter Fertility Test

**Script:** `tests/STRESS_TEST.md`
**Finding:** Gestalt Unicode delimiters (`☩` `☸` `☥`) tokenize at **2.00 tokens per character** under the OpenAI o200k tokenizer — twice the cost of standard English text.

This means each delimiter is being split into sub-tokens rather than recognized as a single atomic unit. The practical implication is that the syntax tax is higher than character-based estimates assumed.

**The offset:** Despite this cost, code compression achieves ~55% cross-model average. The delimiter overhead is real and measurable — and compression wins anyway at sufficient document length.

**Design implication:** Future versions of Gestalt may benefit from evaluating alternative delimiter characters with better tokenizer vocabulary coverage. This is identified as a research direction.

---

## Generation 1 — Character-Based Estimates (Preserved for Reference)

*These results were produced using character-to-token ratio approximation. They are preserved as part of the honest development record of this project. They should not be cited as validated compression figures.*

| Test | Input tokens | Output tokens | Reduction | Notes |
|---|---|---|---|---|
| Single NLP paragraph | ~260 | ~243 | ~7% | Syntax tax visible |
| Two identical paragraphs | ~520 | ~243 | ~53% | Maximum redundancy elimination |
| Two distinct paragraphs | ~434 | ~445 | ~-3% | Syntax tax dominates |
| 78-line code script | ~546 | ~384 | ~30% | English semantic content |
| Executive summary (553 words) | ~1032 | ~868 | ~16% | Domain specific prose |
| SYNTAX_SPEC.md | ~3498 | ~2246 | ~36% | Superseded by deterministic test |
| 566-line code script | ~4817 | ~1768 | ~63% | Consistent with deterministic result |

---

## Test 001 — Code Compression, Short Script (78 lines)
*(Character-based estimation — Generation 1)*

**Purpose:** Establish baseline compression behavior for a short Python script under v3.2 spec

### Test A — Incorrect Implementation (Mandarin semantic content applied to code)

This test was run with Mandarin Chinese as the semantic content language for code blocks. This is a spec violation — v3.2 specifies English or language-agnostic identifiers for code content.

| Metric | Value |
|---|---|
| Original tokens | ~546 |
| Gestalt tokens | ~883 |
| Token change | +62% expansion |

**Finding:** Applying Mandarin to code content inflates token count significantly despite reducing character count. This test inadvertently validated the design decision to exclude Mandarin from code compression.

### Test B — Correct Implementation (English semantic content)

| Metric | Value |
|---|---|
| Original tokens | ~546 |
| Gestalt tokens | ~384 |
| Token reduction | ~30% |

**Finding:** Correct v3.2 implementation produces meaningful compression at 78 lines. Syntax tax is visible but compression wins.

---

## Test 002 — NLP Compression, Single Paragraph
*(Character-based estimation — Generation 1)*

| Metric | Value |
|---|---|
| Original tokens | ~260 |
| Gestalt tokens | ~243 |
| Token reduction | ~7% |

**Finding:** Syntax tax nearly consumes compression gains at single paragraph length.

---

## Test 003 — NLP Compression, Identical Paragraphs
*(Character-based estimation — Generation 1)*

| Metric | Value |
|---|---|
| Original tokens | ~520 |
| Gestalt tokens | ~243 |
| Token reduction | ~53% |

**Finding:** Gestalt output remained flat while input doubled. Demonstrates nonlinear scaling behavior under maximum redundancy conditions.

---

## Test 004 — NLP Compression, Two Distinct Paragraphs
*(Character-based estimation — Generation 1)*

| Metric | Value |
|---|---|
| Original tokens | ~434 |
| Gestalt tokens | ~445 |
| Token change | ~-3% expansion |

**Finding:** Syntax tax dominates when content is distinct with minimal redundancy.

---

## Test 005 — NLP Compression, Executive Summary
*(Character-based estimation — Generation 1)*

| Metric | Value |
|---|---|
| Original tokens | ~1032 |
| Gestalt tokens | ~868 |
| Token reduction | ~16% |

**Finding:** Meaningful compression on domain specific multi-section document.

---

## Test 006 — Technical Specification Document (SYNTAX_SPEC.md)
*(Character-based estimation superseded by Test 009)*

| Metric | Value |
|---|---|
| Original tokens | ~3498 |
| Gestalt tokens | ~2246 |
| Estimated reduction | ~36% |

**Note:** This estimate has been superseded by deterministic tokenizer testing in Test 009. The actual cross-model average compression is ~20.4%.

---

## Test 007 — Code Compression, 566-Line Script
*(Character-based estimation — consistent with deterministic result)*

| Metric | Value |
|---|---|
| Original tokens | ~4817 |
| Gestalt tokens | ~1768 |
| Token reduction | ~63% |

**Finding:** Strongest Generation 1 result. Consistent with the deterministic cross-model average of ~55.9% measured in Test 010.

---

## Test 008 — Cross-Language Translation from Gestalt Source

**Purpose:** Demonstrate that Gestalt encoded source is sufficient for cross-language translation without reference to the original

**Method:** Go translation of CODE_EXAMPLE.md produced using only GST_CODE_EXAMPLE.md as input. Original Python was not referenced during translation.

| Source | Tokens (estimated) |
|---|---|
| Original Python (CODE_EXAMPLE.md) | ~4817 |
| Gestalt encoded (GST_CODE_EXAMPLE.md) | ~1768 |
| Input token reduction | ~63% |

**Output:** 418-line Go implementation — see GO_TRANSLATION_EXAMPLE.md

**Finding:** A ~63% reduction in input tokens produced a structurally complete, idiomatically correct Go translation. Structural correctness verified by inspection. Compilation pending Go runtime validation.

---

## Test 009 — Deterministic NLP Compression (SYNTAX_SPEC.md)
*(Generation 2 — actual tokenizer counts)*

**Script:** `tests/AUDIT_SYNTAX.md` — `gestalt_bench.py`
**Files:** `GST_SYNTAX_SPEC.md` vs `SYNTAX_SPEC.md`

| Tokenizer Family | GST Tokens | Control Tokens | Compression |
|---|---|---|---|
| OpenAI (o200k / GPT-5) | 2455 | 3029 | 19.0% |
| Meta (Llama 3.1) | 2436 | 3035 | 19.7% |
| Google (Gemma 3) | 2161 | 3034 | 28.8% |
| Mistral (Codestral) | 3252 | 3791 | 14.2% |
| **Cross-model average** | | | **~20.4%** |

**Finding:** Real NLP compression is ~20% cross-model average — lower than the character-based estimate of 36%. OpenAI and Meta show near-identical results suggesting compression is a content property. Gemma shows strongest compression. Mistral's code-optimized tokenizer produces the weakest NLP compression, consistent with the delimiter fertility finding.

---

## Test 010 — Deterministic Code Compression (CODE_EXAMPLE.md)
*(Generation 2 — actual tokenizer counts)*

**Script:** `tests/AUDIT_SYNTAX.md` — `gestalt_bench.py`
**Files:** `GST_CODE_EXAMPLE.md` vs `CODE_EXAMPLE.md`

| Tokenizer Family | GST Tokens | Control Tokens | Compression |
|---|---|---|---|
| OpenAI (o200k / GPT-5) | 2263 | 4808 | 52.9% |
| Meta (Llama 3.1) | 2253 | 4801 | 53.1% |
| Google (Gemma 3) | 2160 | 5890 | 63.3% |
| Mistral (Codestral) | 2860 | 6253 | 54.3% |
| **Cross-model average** | | | **~55.9%** |

**Finding:** Real code compression averages ~55.9% across four tokenizer families. Consistency across OpenAI, Meta, and Mistral (within ~1.4 points) suggests compression is a property of the content structure, not the tokenizer. Google Gemma shows strongest compression at 63.3%. The nonlinear scaling hypothesis is confirmed — code compression at scale significantly outperforms NLP compression.

---

## Test 011 — Delimiter Fertility
*(Generation 2 — actual tokenizer counts)*

**Script:** `tests/STRESS_TEST.md` — `stress_test.py`
**Tokenizer:** OpenAI o200k

| Metric | Value |
|---|---|
| Delimiters tested | `☩` `☸` `☥` |
| Tokens per character | 2.00 |
| Result | ⚠️ Attention Drift Warning |

**Finding:** Each Gestalt delimiter tokenizes as 2 sub-tokens rather than a single atomic unit under the OpenAI o200k tokenizer. The syntax tax is higher than character-based estimates assumed. Despite this cost, code compression achieves ~55.9% cross-model average — compression wins at sufficient document length. Delimiter selection is identified as a research direction for future versions.

---

## Observed Patterns

- Syntax tax dominates at short document lengths regardless of domain
- Compression improves nonlinearly as document length increases
- Code compression significantly outperforms NLP compression at scale (~55% vs ~20%)
- Compression behavior is consistent across tokenizer families for code — suggesting it is a content property
- Delimiter fertility (2.00 tokens/char) contributes to syntax tax but does not eliminate compression gains at scale
- Domain specific technical content outperforms general prose
- The content language split between code (English) and NLP domains has measurable token efficiency implications

---

## Language Layer Status

Modern Chinese was specified in earlier versions of Gestalt as the NLP semantic content language. This has not been validated under deterministic tokenization testing. Current testing uses English semantic content throughout. Cross-language transport remains theorized but unproven.

---

## Planned Tests

- Deterministic NLP compression at longer document lengths (5000+ tokens)
- RELATES overhead isolation — compression with and without relationship declarations
- Delimiter fertility across all four tokenizer families
- Alternative delimiter candidates with better tokenizer vocabulary coverage
- Cross-model comprehension accuracy testing
- Round-trip fidelity testing — compress then expand, compare to original
- Mandarin vs English NLP semantic content under deterministic tokenization

---

## Test Scripts

### Generation 1 — Character-based estimation

```python
def estimate_tokens(text, lang="english"):
    chars = len(text)
    if lang == "chinese":
        return round(chars / 1.5)
    return round(chars / 4)
```

### Generation 2 — Deterministic tokenizer testing

See `tests/STRESS_TEST.md` for the delimiter fertility script.
See `tests/AUDIT_SYNTAX.md` for the four-tokenizer comparative audit script.

Both scripts were contributed by a community member following the initial repo publication. The deterministic methodology supersedes the character-based estimates for any results where both exist.

---
