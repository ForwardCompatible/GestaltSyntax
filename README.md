# Gestalt Syntax

**An AI-native semantic compression schema for natural language and code.**

---

## What is Gestalt?

Gestalt is a structured syntax for encoding documents into a compressed, semantically dense format that any AI can read, write, and reason over without prior training or examples.

It was created as a personal project to explore a simple question: what happens when you design a document format for AI first, and humans second?

Gestalt is not a programming language. It is not a data serialization format. It is a way of representing meaning — stripped of grammatical overhead, organized by explicit relationships, and structured so that an AI can reason about the content rather than just retrieve it.

---

## What's in this repo?

**Specification and reference:**
- `SYNTAX_SPEC.md` — the full human-readable specification
- `GST_SYNTAX_SPEC.md` — the specification encoded in Gestalt (hand this to your AI and ask it questions)
- `STYLE_GUIDE.md` — a minimal drop-in that teaches any AI to encode documents in Gestalt

**Examples:**
- `CODE_EXAMPLE.md` — a Python task manager (~566 lines)
- `GST_CODE_EXAMPLE.md` — the same script encoded in Gestalt
- `GO_TRANSLATION_EXAMPLE.md` — a Go translation produced from the Gestalt version alone
- `TEST_RESULTS.md` — full test documentation including both estimated and deterministic results

**Tests folder — reproduce these yourself:**
- `tests/DEBUG_TEST.md` — a buggy Python script
- `tests/GST_DEBUG_TEST.md` — the same script encoded in Gestalt, no error flags
- `tests/DEBUG_ANSWER_KEY.md` — the four planted errors and where they are
- `tests/DEBUG_RECONSTRUCTION_TEST.md` — full session record of a blind debugging and reconstruction test
- `tests/STRESS_TEST.md` — delimiter fertility test script
- `tests/AUDIT_SYNTAX.md` — four-tokenizer comparative audit script

---

## What do the test results show?

Token counts were validated using actual tokenizer libraries across four major model families. Initial character-based estimates are preserved in `TEST_RESULTS.md` for reference — the deterministic results supersede them where both exist.

*Thank you to the community member who contributed the deterministic test scripts within hours of this repo going live.*

### NLP Compression — SYNTAX_SPEC.md vs GST_SYNTAX_SPEC.md

| Tokenizer | Compression |
|---|---|
| OpenAI (o200k) | 19.0% |
| Meta (Llama 3.1) | 19.7% |
| Google (Gemma 3) | 28.8% |
| Mistral (Codestral) | 14.2% |
| **Cross-model average** | **~20%** |

### Code Compression — CODE_EXAMPLE.md vs GST_CODE_EXAMPLE.md

| Tokenizer | Compression |
|---|---|
| OpenAI (o200k) | 52.9% |
| Meta (Llama 3.1) | 53.1% |
| Google (Gemma 3) | 63.3% |
| Mistral (Codestral) | 54.3% |
| **Cross-model average** | **~56%** |

The consistency across tokenizer families — particularly for code — suggests compression is a property of the content structure, not the tokenizer. Code compression significantly outperforms NLP compression at scale.

**On delimiter cost:** The Gestalt Unicode delimiters tokenize at 2.00 tokens per character under OpenAI o200k — twice the cost of standard English text. The syntax tax is real. At sufficient document length, compression wins anyway.

**On the debugging test:**

A Gestalt encoded script containing four intentional errors was provided to a fresh AI session with no system prompt, no syntax guide, and no indication that errors existed. The model identified four suspicious locations through semantic reasoning alone — unprompted — before being asked to reconstruct the script. During reconstruction it resolved all four hotspots correctly and produced cleaner code than the buggy original.

The identification of hotspots was unprompted emergent behavior. This suggests Gestalt may have applications to codebase analysis beyond line-by-line reasoning — though more testing is required.

**On cross-language translation:**

A Go translation was produced from the Gestalt encoded Python script alone, without reference to the original source. Input token cost was ~2263 tokens (Gestalt) versus ~4808 tokens (original Python) — a 52.9% reduction in translation source size under OpenAI tokenization.

---

## How to use it

**Drop the style guide into any AI session:**
Paste `STYLE_GUIDE.md` into your AI and ask it to encode a document. No further explanation should be necessary.

**Use it as a Claude skill:**
See the companion repo [GestaltAnalysis](https://github.com/ForwardCompatible/GestaltAnalysis) for Claude Code skills that encode and analyze codebases using Gestalt syntax.

**Use GST_SYNTAX_SPEC.md as a reference:**
Drop it into a project or session and ask your AI questions about the syntax. It will answer from the encoded document.

**Run the debugging test:**
Follow the instructions in `tests/DEBUG_RECONSTRUCTION_TEST.md` to reproduce the blind debugging experiment with your own AI. Record your results and share them.

**Run the compression audit:**
Follow the instructions in `tests/AUDIT_SYNTAX.md` to run deterministic token counts against any pair of documents.

---

## A note on claims

Nothing in this repo should be read as a finished conclusion. The test results are real and deterministic — the interpretations are still evolving. If you run these tests and get different results, that is valuable. If you find a use case that works better than expected, that is also valuable.

---

## Exploration roadmap

**Language layer optimization**
The v3.2 specification was designed with Modern Chinese as the NLP semantic content layer, theorized to provide higher semantic density per token. This has not been validated under deterministic tokenization. Comparing character-based languages against English for NLP compression is a logical next experiment.

**Delimiter optimization**
Current delimiters tokenize at 2.00 tokens per character. Identifying Unicode characters with better tokenizer vocabulary coverage across model families could reduce the syntax tax meaningfully.

**Hotspot analysis via prompt engineering**
The debugging test revealed spontaneous semantic ambiguity identification from Gestalt encoded code. Whether this behavior can be reliably triggered and directed through prompt engineering — and whether it scales to large codebases — has not been tested. See [GestaltAnalysis](https://github.com/ForwardCompatible/GestaltAnalysis) for an initial implementation.

**Legacy codebase analysis**
The ~56% code compression suggests Gestalt may be a viable strategy for fitting large codebases into constrained context windows. Whether a reasoning model can perform meaningful architectural analysis from Gestalt compressed source without reconstructing the original is an open question.

**Neutral Semantic Ground**
Community observation: Gestalt encodes intent rather than implementation, acting as a language-neutral intermediate representation. Cross-language translation from Gestalt source produces idiomatic target-language code rather than literal translation. Implications for legacy code migration and polyglot codebases warrant further investigation.

**RAG and vector store alternatives**
Each Gestalt block is a pre-chunked semantic unit with explicit relationship declarations. This structure maps naturally onto graph-based retrieval. Whether Gestalt encoded documents improve retrieval quality or reduce overhead compared to standard RAG pipelines has not been tested.

**FIM editing suitability**
Gestalt is likely better suited as a storage and transport format than for actively edited documents. Whether fill-in-the-middle style editing introduces structural artifacts has not been assessed.

**Compressed instruction sets for AI skills and cross-session transfer**
If Gestalt carries more meaning per token than plain prose, it may be viable for encoding long-running rules or context that needs to survive session boundaries. Untested but potentially high value.

**KV cache pre-computation**
Community observation: Gestalt may function as a form of pre-computed KV cache — by encoding relational structure explicitly, the model encounters tokens whose relationships are already declared rather than inferred. If accurate, the efficiency gains extend beyond token count to inference cost. Requires architectural investigation.

**Cross-model comprehension**
All tests conducted with a single model. Whether zero-shot comprehension holds consistently across architectures and sizes is an open question.

---

## Contributing

If you run any of the tests in the `tests/` folder, please consider sharing your results. What model did you use? How many errors did it find? Did the reconstruction succeed? Did you observe hotspot behavior?

This project advances through community testing. Your results — positive, negative, or inconclusive — are all useful.

---

*Gestalt v3.2 — published as an open exploration. Not a finished product. Not a proven system. A starting point.*

---

## Author

**Ryan Connelly**

---

## License

This work is licensed under the **Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0)**.

You are free to share and adapt this material for personal and academic purposes with attribution. Commercial use requires a separate license agreement.

For commercial licensing inquiries, contact the author via GitHub.

Full license terms: https://creativecommons.org/licenses/by-nc/4.0/

---
