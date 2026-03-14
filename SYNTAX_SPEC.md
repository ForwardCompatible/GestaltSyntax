# Gestalt Syntax Specification
**Version 3.2**

---

## Overview

Gestalt is a novel, AI-focused syntax schema created as a project to explore context density in structured documents, as well as explore alternatives to vector store and RAG approaches for relational context. It is an AI-native format — designed to be written by AI, read by AI, and translated by AI. It is not intended for human authorship or direct human consumption.

Gestalt was designed with two intentions in mind:

- Reduce the token and reasoning overhead that AI models incur when processing long or complex documents
- Increase context density per token through an intentionally flexible, relational syntax structure

Whether the design achieves either of these goals in practice is the subject of ongoing testing.

---

## Core Principles

**AI native comprehension** — Designed for zero-shot understanding across AI models without prior training or examples.

**Semantic density optimization** — Prioritizes maximum meaning per token through deliberate structural design.

**Nonlinear compression scaling** — Framework overhead is a fixed cost that amortizes over longer content. Short documents may see little or no compression benefit.

**Unified syntax framework** — Code and NLP content share identical syntax patterns and delimiter conventions within a single document.

**Cross language semantic mining** — Semantic content may be expressed in the language that best represents the concept, independent of the source document language.

**Structured relationship preservation** — Semantic connections between blocks are explicit, never inferred.

**Deterministic expansion** — A correctly encoded Gestalt document should expand to a consistent and predictable meaning. If the same block could reasonably be interpreted two different ways, the encoding is incomplete.

---

## Delimiter System

Gestalt uses Unicode characters as delimiters, specifically chosen to avoid collision with code syntax. These characters must not be used for any other purpose within a document.

| Character | Role |
|---|---|
| `☩` | Separates block components |
| `☸` | Opens a block or section |
| `☥` | Closes a block or section |

---

## Document Header

Every Gestalt document must begin with a META block on the first line:

```
META☩document_identifier☩configuration_parameters
```

The META block declares the parsing context and content configuration for the document. It is required and must appear first.

### META Header Variants

**Code document:**
```
META☩code_syntax☩lang:python,lbrace_escape:☸,rbrace_escape:☥
```

**NLP document:**
```
META☩nlp_syntax☩lang:modern_chinese,content_type:technical,lbrace_escape:☸,rbrace_escape:☥
```

**Mixed content document:**
```
META☩mixed_content☩code_lang:python,text_lang:modern_chinese,lbrace_escape:☸,rbrace_escape:☥
```

### Configuration Parameters

| Parameter | Purpose |
|---|---|
| `lang` | Primary content language |
| `code_lang` | Code content language for mixed documents |
| `text_lang` | NLP content language for mixed documents |
| `content_type` | Domain optimization hint: technical, conversational, academic, creative |
| `lbrace_escape` | Unicode escape for open delimiter |
| `rbrace_escape` | Unicode escape for close delimiter |

---

## Block Format

Every Gestalt block follows the same base format:

```
BLOCK_TYPE☩identifier☩metadata☸semantic_content☥
```

**BLOCK_TYPE** — Semantic classification of the block. Reserved and recommended types are defined below.

**identifier** — A concise semantic label for the block.

**metadata** — Required key:value pairs providing expansion context. Multiple pairs separated by commas. Keys and values should be kept concise.

**semantic_content** — The compressed semantic core of the block.

A regular expression for block validation:

```
^(\w+)☩([^☩]*?)(?:☩([^☸]*?))?☸([^☥]+)☥$
```

---

## Reserved Block Types

These block types have fixed structural meaning and must be used exactly as defined. They may not be repurposed or redefined.

**META** — Document header. Required. Must appear as the first line of every Gestalt document.

**DOC** — Outermost document wrapper. Contains all other blocks.
```
DOC☩document_name☩metadata☸
  blocks here
☥
```

**SEC** — Groups related blocks into a named hierarchical section. Requires `level` metadata key.
```
SEC☩section_name☩level:1☸
  blocks here
☥
```

**DEFINITIONS** — Declares custom block types or abbreviations used in the document. Must appear at the top or bottom of the document.
```
DEFINITIONS☩document_key☩scope:document☸
  CUSTOM_TYPE☩description
  ABBREV☩full_form
☥
```

**EXAMPLE** — A raw content container that nests under a parent block. Content inside an EXAMPLE block is never compressed — it is preserved exactly as written. Use `ref:parent` to link it to its parent block.
```
CONCEPT☩function_name☩domain:python☸compressed semantic description☥
EXAMPLE☩function_name☩ref:parent☸
def function_name():
    pass
☥
```

---

## Recommended Block Types

The following is a non-exhaustive starting vocabulary. Custom block types are permitted provided a `DEFINITIONS` block is supplied.

| Block Type | Purpose |
|---|---|
| STATEMENT | Facts, assertions, declarations, opinions |
| QUESTION | Information seeking, clarification, exploration |
| DESCRIPTION | Sensory, visual, or environmental information |
| INTENT | Objectives, goals, desired outcomes |
| EMOTION | Emotional states, sentiment, mood |
| INSTRUCTION | Commands, procedures, directions |
| NARRATIVE | Temporal sequences, cause and effect chains |
| CONCEPT | Abstract ideas, definitions, theoretical constructs |
| PROTOCOL | Systematic procedures, workflows |
| RULE | Constraints, requirements, limitations |

---

## Code Domain Block Types

The following block types are defined specifically for code compression:

**FUNC** — Defines a function.
```
FUNC☩function_name☩params:param_types,return:type,async:bool,complexity:O_notation☸semantic description of function purpose☥
```

**CLASS** — Defines a class.
```
CLASS☩class_name☩inherits:parent,access:visibility,namespace:scope☸semantic description of class purpose☥
```

**Control structures** follow inline shorthand patterns:

```
IF☩condition☸action☥ELSE☸alternative☥
LOOP☩type☩condition☸body☥
TRY☸attempt☥CATCH☩exception☸handler☥
SWITCH☩variable☸cases☥
```

---

## Content Language

Gestalt distinguishes content language by domain:

**NLP content** — Semantic content is written in Modern Chinese by design. Character-based languages carry higher semantic density per token than grammatically complex languages, reducing overhead while preserving meaning.

**Code content** — Semantic content uses English or language-agnostic identifiers. Code structure is already compressed through function names, types, and signatures.

**Mixed content** — Each domain follows its own content language convention within the same document.

---

## Hierarchical Organization

Gestalt documents can be organized into nested sections using the `SEC` reserved block type. Sections group related blocks and provide inherited context to everything they contain.

```
SEC☩section_name☩level:1☸
  BLOCK_TYPE☩identifier☩metadata☸semantic_content☥

  SEC☩subsection_name☩level:2☸
    BLOCK_TYPE☩identifier☩metadata☸semantic_content☥
  ☥
☥
```

**Rules:**
- The `level` metadata key is required on every `SEC` block
- Blocks inside a section inherit the context of their parent section
- Sections are optional — a flat document structure is valid
- Indentation is permitted and may improve readability but carries no syntactic meaning

---

## Metadata

Metadata is required on every block. It provides the contextual information that compression removes and is the primary mechanism by which deterministic expansion is achieved.

```
key:value
key:value,key:value,key:value
```

### Reserved Metadata Keys

| Key | Used On | Purpose |
|---|---|---|
| `level` | SEC | Indicates hierarchical nesting depth |
| `ref:parent` | EXAMPLE | Links the example to its parent block |
| `scope` | DEFINITIONS | Declares the scope of the definitions block |
| `async` | FUNC | Indicates asynchronous function |
| `complexity` | FUNC | Big O notation for algorithmic complexity |

### Common Metadata Categories

Custom keys are permitted without a `DEFINITIONS` entry as descriptive key names are considered self-documenting.

- **Certainty** — `certainty:level`
- **Domain** — `domain:field`
- **Agent** — `agent:actor`
- **Temporal** — `temporal:timeframe`
- **Spatial** — `spatial:location`
- **Emotional** — `intensity:level`, `valence:positive_negative_neutral`
- **Causal** — `causality:type`
- **Code** — `async:bool`, `complexity:O_notation`, `access:visibility`, `safety:level`

---

## Relationship Syntax

Relationships between blocks are declared explicitly using the `RELATES` keyword. Place the declaration immediately after the originating block. No semantic connection should be left to inference.

```
RELATES☩target_identifier☩relationship_type
```

### Logical
`supports`, `contradicts`, `builds_on`, `evidences`, `derives_from`, `exemplifies`

### Causal
`causes`, `results_in`, `enables`, `prevents`, `triggered_by`, `influences`

### Temporal
`precedes`, `follows`, `concurrent`, `interrupts`, `resumes`, `cyclical`

### Semantic
`defines`, `clarifies`, `contextualizes`, `generalizes`, `specifies`, `analogizes`

### Code Specific
`calls`, `implements`, `contains`, `throws`, `returns`, `inherits`, `depends_on`

### Cross Domain
`explains`, `documents`, `tests`, `validates`, `implements_concept`

---

## Encoding Guidelines

Gestalt encoding is the process of reducing natural language or code to its semantic core. The goal is to preserve meaning completely while removing everything that does not contribute to it.

### What to Preserve

- **Nouns and proper nouns** — the subjects and objects of meaning
- **Verbs** — the actions and states that connect meaning
- **Adjectives** — qualifiers that change or specify meaning
- **Negations** — "not", "never", "no" — removing these inverts meaning
- **Quantifiers** — "all", "some", "none", "every" — these define scope
- **Domain specific terminology** — technical terms that cannot be substituted without loss of precision
- **Semantic logic and architectural relationships** — the reasoning structure of code

### What to Omit

- **Articles** — "the", "a", "an"
- **Most prepositions** — unless their removal alters the relationship between elements
- **Conjunctions** — unless they carry logical weight such as "but", "however", or "except"
- **Boilerplate and repetitive patterns** — structural overhead that adds no semantic value
- **Redundant restatements** — information already captured in metadata

### The Guiding Test

If removing something changes what the block means, keep it. If the meaning survives without it, omit it.

---

## Content Type Considerations

Different content types compress differently. This is a design consideration, not a guarantee:

**Technical documentation** — High redundancy patterns make this a strong candidate for compression.

**Conversational text** — Filler words and informal patterns offer moderate compression opportunity. Emotional tone and social context should be preserved.

**Academic literature** — Already structurally optimized. Primary benefit may be improved relationship clarity rather than token reduction.

**Creative and literary content** — Deliberately word-choice optimized. Compression potential is minimal. Primary benefit is structural analysis support.

**Short content of any type** — Framework overhead dominates at low token counts. Compression benefits emerge at longer document lengths.

---

## Validation Requirements

A correctly encoded Gestalt document should satisfy the following requirements. These serve both as properties of well-formed Gestalt and as a checklist for encoding:

**Semantic fidelity** — The compressed form must preserve the complete meaning of the original. Nothing that changes interpretation should be omitted.

**Deterministic expansion** — A Gestalt block must expand to a consistent and predictable meaning. If the same block could reasonably be interpreted two different ways, the encoding is incomplete.

**Relationship integrity** — All logical connections between blocks must be explicitly declared using `RELATES`.

**Syntactic correctness** — Encoded content should be verifiable against the target language or domain after expansion.

---

## Mixed Content Documents

Gestalt handles mixed code and NLP content within a single document using the same delimiter system throughout. Each domain follows its own synergistic content language convention. Cross-domain relationships between code blocks and NLP blocks are expressed using the cross-domain `RELATES` types.

This unified approach enables complete system documentation where code blocks reference NLP concepts and NLP blocks reference code implementations within a single traversable semantic graph.

---

## Future Research Directions

The following areas are identified for further investigation:

- Optimal language selection for NLP semantic content — comparison of character-based languages
- Domain specific vocabulary optimization for technical fields
- Compression behavior across different document lengths and content types
- Probabilistic and deterministic parsing approaches
- Semantic graph storage and traversal strategies
- Real time streaming document compression
- Applications to legacy codebase analysis and cross-language translation
- Suitability as a storage and transport format versus a format for actively edited documents — whether FIM style editing introduces structural artifacts or requires document-wide restructuring after changes
- Potential applications as a compressed instruction format for AI skills and cross-session context transfer
- Viability as a long-running rules or context format when source material is first encoded from natural language

---
