META☩style_guide☩lang:english,content_type:technical,lbrace_escape:☸,rbrace_escape:☥

DOC☩gestalt_style_guide☩version:3.2,purpose:encoding_reference☸

SEC☩syntax☩level:1☸

RULE☩block_format☩domain:syntax☸
every block follows: BLOCK_TYPE☩identifier☩metadata☸semantic_content☥☥

RULE☩delimiters☩domain:syntax☸
☩ separates components, ☸ opens block, ☥ closes block — Unicode only, never substitute ASCII equivalents☥

RULE☩metadata☩domain:syntax☸
required on every block, key:value pairs separated by commas, self-documenting keys permitted without DEFINITIONS☥

RULE☩indentation☩domain:syntax☸
optional, carries no syntactic meaning, used for readability only☥

☥

SEC☩reserved_blocks☩level:1☸

RULE☩META☩domain:reserved☸
required first line of every document, declares parsing context☥

EXAMPLE☩META☩ref:parent☸
META☩document_name☩lang:english,content_type:technical,lbrace_escape:☸,rbrace_escape:☥
☥

RULE☩DOC☩domain:reserved☸
outermost wrapper, contains all blocks☥

RULE☩SEC☩domain:reserved☸
groups related blocks, level metadata key required☥

RULE☩DEFINITIONS☩domain:reserved☸
declares custom block types or abbreviations, must appear at top or bottom of document☥

EXAMPLE☩DEFINITIONS☩ref:parent☸
DEFINITIONS☩custom_types☩scope:document☸
  MYTYPE☩description of custom block type
☥
☥

RULE☩EXAMPLE☩domain:reserved☸
raw content container, content never compressed, preserved exactly as written, use ref:parent to link to parent block☥

☥

SEC☩encoding☩level:1☸

RULE☩preserve☩domain:encoding☸
nouns, verbs, adjectives, negations, quantifiers, domain terminology, semantic logic☥

RULE☩omit☩domain:encoding☸
articles, most prepositions, conjunctions without logical weight, redundant restatements already in metadata☥

RULE☩guiding_test☩domain:encoding☸
if removal changes block meaning keep it, if meaning survives without it omit it☥

RULE☩content_language☩domain:encoding☸
NLP content: English or language of highest semantic density for concept, Code content: English or language-agnostic identifiers☥

☥

SEC☩relationships☩level:1☸

RULE☩relates_syntax☩domain:syntax☸
RELATES☩target_identifier☩relationship_type — place immediately after originating block, never leave connections to inference☥

CONCEPT☩relationship_types☩domain:reference☸
logical: supports, contradicts, builds_on, evidences, derives_from, exemplifies
causal: causes, results_in, enables, prevents, triggered_by, influences
temporal: precedes, follows, concurrent, interrupts, resumes, cyclical
semantic: defines, clarifies, contextualizes, generalizes, specifies, analogizes
code: calls, implements, contains, throws, returns, inherits, depends_on
cross-domain: explains, documents, tests, validates, implements_concept☥

☥

SEC☩vocabulary☩level:1☸

CONCEPT☩recommended_block_types☩domain:reference☸
non-exhaustive starting vocabulary, custom types permitted with DEFINITIONS block:
STATEMENT: facts assertions declarations
QUESTION: inquiry clarification exploration
DESCRIPTION: sensory environmental information
INTENT: objectives goals outcomes
EMOTION: emotional states sentiment
INSTRUCTION: commands procedures directions
NARRATIVE: temporal sequences cause and effect
CONCEPT: abstract ideas definitions
PROTOCOL: workflows methodologies
RULE: constraints requirements limitations☥

☥

SEC☩anti_patterns☩level:1☸

CONCEPT☩anti_pattern_missing_metadata☩domain:anti_pattern☸
metadata omitted — block cannot deterministically expand, encoding is incomplete☥

EXAMPLE☩anti_pattern_missing_metadata☩ref:parent☸
ANTI-PATTERN — do not encode this way:
STATEMENT☩project_goal☸reduce token overhead☥

CORRECT:
STATEMENT☩project_goal☩domain:AI,certainty:stated☸reduce token overhead☥
☥

CONCEPT☩anti_pattern_compressed_code☩domain:anti_pattern☸
code content compressed instead of preserved — loses implementation fidelity, defeats reconstruction☥

EXAMPLE☩anti_pattern_compressed_code☩ref:parent☸
ANTI-PATTERN — do not encode this way:
FUNC☩calculate_tax☩params:amount:float,return:float☸multiplies amount by rate and rounds☥

CORRECT:
FUNC☩calculate_tax☩params:amount:float,return:float☸multiplies amount by TAX_RATE, rounds to 2 decimal places☥
EXAMPLE☩calculate_tax☩ref:parent☸
def calculate_tax(amount: float) -> float:
    return round(amount * TAX_RATE, 2)
☥
☥

CONCEPT☩anti_pattern_ascii_delimiters☩domain:anti_pattern☸
ASCII delimiters used instead of Unicode — collides with code syntax, breaks mixed content documents☥

EXAMPLE☩anti_pattern_ascii_delimiters☩ref:parent☸
ANTI-PATTERN — do not encode this way:
FUNC|calculate_tax|params:amount:float,return:float{multiplies amount by TAX_RATE}

CORRECT:
FUNC☩calculate_tax☩params:amount:float,return:float☸multiplies amount by TAX_RATE☥
☥

☥

☥
