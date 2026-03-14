# Gestalt Syntax

**An AI-native semantic compression schema for natural language and code.**

---

## What is Gestalt?

Gestalt is a structured syntax for encoding documents into a compressed, semantically dense format that any AI can read, write, and reason over without prior training or examples.

It was created as a personal project to explore a simple question: what happens when you design a document format for AI first, and humans second?

Gestalt is not a programming language. It is not a data serialization format. It is a way of representing meaning — stripped of grammatical overhead, organized by explicit relationships, and structured so that an AI can reason about the content rather than just retrieve it.

---

## What's in this repo?

This repository contains the complete v3.2 specification, a style guide you can drop into any AI session or Claude skill, example documents, and a tests folder with reproducible experiments anyone can run with their own AI.

---
## How to use it (QRG)

**Drop the style guide into any AI session:**
Paste `STYLE_GUIDE.md` into your AI and ask it to encode a document. No further explanation should be necessary.

**Use it as a Claude skill:**
Package `STYLE_GUIDE.md` as a skill and your AI will encode documents in Gestalt on request without needing the guide in every session. Or you can try using the skill provided  at https://github.com/ForwardCompatible/GestaltAnalysis

**Use GST_SYNTAX_SPEC.md as a reference:**
Drop it into a project or session and ask your AI questions about the syntax. It will answer from the encoded document.

**Run the debugging test:**
Follow the instructions in `tests/DEBUG_RECONSTRUCTION_TEST.md` to reproduce the blind debugging experiment with your own AI. Record your results and share them.

---


**Specification and reference:**
Specification/
- `SYNTAX_SPEC.md` — the full human-readable specification
- `GST_SYNTAX_SPEC.md` — the specification encoded in Gestalt (hand this to your AI and ask it questions)
- `STYLE_GUIDE.md` — a minimal drop-in that teaches any AI to encode documents in Gestalt

**Examples:**
examples/
- `CODE_EXAMPLE.md` — a Python task manager (~566 lines)
- `GST_CODE_EXAMPLE.md` — the same script encoded in Gestalt
- `GO_TRANSLATION_EXAMPLE.md` — a Go translation produced from the Gestalt version alone

**Tests folder — reproduce these yourself:**
- `tests/DEBUG_TEST.md` — a buggy Python script
- `tests/GST_DEBUG_TEST.md` — the same script encoded in Gestalt, no error flags
- `tests/DEBUG_ANSWER_KEY.md` — the four planted errors and where they are
- `tests/DEBUG_RECONSTRUCTION_TEST.md` — full session record of a blind debugging and reconstruction test

---

## What do the test results suggest?

All results are approximations based on character-to-token ratio estimation. Tests and their results have been included in `TEST_RESULTS.md` so you can verify and extend these findings yourself.

| Content | Input tokens | Compressed tokens | Change |
|---|---|---|---|
| Single NLP paragraph | ~260 | ~243 | -7% |
| Two distinct NLP paragraphs | ~434 | ~445 | +3% |
| Executive summary (553 words) | ~1032 | ~868 | -16% |
| Technical specification (~3500 tokens) | ~3498 | ~2246 | -36% |
| 78-line Python script | ~546 | ~384 | -30% |
| 566-line Python script | ~4817 | ~1768 | -63% |

The results suggest compression is not linear — framework overhead dominates at short document lengths and amortizes significantly at longer ones. The most consistent gains appear in domain-specific technical content and larger codebases.

**On the debugging test:**

A Gestalt encoded script containing four intentional errors was provided to a fresh AI session with no system prompt, no syntax guide, and no indication that errors existed. The model identified four suspicious locations through semantic reasoning alone before being asked to reconstruct the script. During reconstruction it resolved all four hotspots correctly and produced cleaner code than the buggy original.

The identification of hotspots was unprompted emergent behavior. The model was not told errors existed. It reasoned about where semantic descriptions were ambiguous enough that implementation could diverge from intent. This suggests Gestalt may have applications to codebase analysis beyond line-by-line reasoning — though more testing is required.

**On cross-language translation:**

A Go translation was produced from the Gestalt encoded Python script alone, without reference to the original source. The translation is structurally complete and idiomatically correct. Input token cost was ~1768 tokens versus ~4817 for the original Python — a 63% reduction in translation source size.

---

## A note on claims

Nothing in this repo should be read as a finished conclusion. The test results suggest interesting behavior. They do not prove it generalizes. The token estimates are approximations. The debugging test was run once, on one model, with one script.

The point of publishing this is to invite rigorous testing by people who have no stake in the outcome. If you run these tests and get different results, that is valuable. If you find a use case that works better than expected, that is also valuable. Both outcomes move the conversation forward.

---

## Exploration roadmap

The following areas are open questions. Testing is required — and contributions are welcome.

**Language layer optimization**
The v3.2 specification was designed with Modern Chinese as the NLP semantic content layer, theorized to provide higher semantic density per token due to character-based encoding. This has not been validated under proper tokenization testing. Comparing Chinese, Japanese, Korean, and other character-based languages against English for NLP compression is a logical next experiment.

**Hotspot analysis via prompt engineering**
The debugging test revealed that an AI presented with Gestalt encoded code will spontaneously identify locations where semantic descriptions are ambiguous enough that implementation could diverge from intent. Whether this behavior can be reliably triggered and directed through prompt engineering — and whether it scales to large codebases — has not been tested.

**Legacy codebase analysis**
The 63% token reduction on a 566-line script suggests Gestalt may be a viable strategy for fitting large codebases into constrained context windows. Whether a reasoning model can perform meaningful architectural analysis, dependency mapping, or refactoring recommendations from Gestalt compressed source — without reconstructing the original — is an open and interesting question.

**RAG and vector store alternatives**
Each Gestalt block is a pre-chunked semantic unit with explicit relationship declarations. This structure maps naturally onto graph-based retrieval without requiring probabilistic chunking strategies. Whether Gestalt encoded documents improve retrieval quality or reduce retrieval overhead compared to standard RAG pipelines has not been tested.

**FIM editing suitability**
Gestalt is likely better suited as a storage and transport format than as a format for actively edited documents. Whether fill-in-the-middle style editing introduces structural artifacts or requires document-wide restructuring after changes has not been assessed.

**Compressed instruction sets for AI skills and cross-session transfer**
If Gestalt can carry more meaning per token than plain prose, it may be a viable format for encoding long-running rules, behavioral instructions, or context that needs to survive session boundaries. This has not been tested but represents a potentially high-value application.

**Cross-model comprehension**
All tests in this repo were conducted with a single model. Whether the zero-shot comprehension property holds consistently across different model architectures and sizes is an open question.

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
