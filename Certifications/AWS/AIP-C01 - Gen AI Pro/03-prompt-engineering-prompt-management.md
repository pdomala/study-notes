[← Back to index](README.md)

# ✍️ 03 · Prompt engineering & Prompt Management

> Exam domain: D1

## Key points
- Prompt elements: **instruction, context, input data, output indicator**. **System prompt** overrides the user prompt and persists across turns.
- Long documents → **document first, instructions last**.
- Techniques: zero/few-shot (labels in examples don't have to be correct; too many examples → dynamic few-shot) · **CoT** · **self-consistency** = many reasoning paths + majority vote · **ToT** = branching/planning · **ReAct** = reason + act with external tools · **ART** = task library + tools.
- Doc bigger than the context window → **chunk + chain (map-reduce) summarisation**.
- **Prompt Management** = versioned templates with `{{vars}}`, invoked by ARN; reuse in Flows. Multi-turn state / checkpoint / fork (LangGraph) → **Session Management APIs**.
- **Prompt injection** = input overrides instructions · **prompt leaking** = output exposes system/sensitive info. A system prompt alone is bypassable → add Guardrails prompt-attack filter + output filtering.

## Notes

### Prompt engineering fundamentals & elements of a prompt
- Prompt eng vs fine-tuning: fine-tuning changes weights (expensive compute + time); prompt eng guides an already-trained model (**no weight changes**) → fastest/cheapest lever
- Benefits: boost abilities & safety, **add domain knowledge/tools without changing params**, better outputs from better inputs
- Key aspects: **design, augmentation** (examples/constraints), **tuning** (iterate), **ensembling**, **mining** (prompt libraries/search)
- **Elements of a prompt:** **Instruction** (task) · **Context** (guiding info) · **Input data** · **Output indicator** (type/format, e.g. "in two sentences")
- **Design tips:**
  - Clear & concise, specific, natural language (not keyword soup)
  - Include context ("summary *for a blog post*")
  - Directives for response type ("answer in a full sentence")
  - **Put the requested output at the end** of the prompt
  - Start with a question word (who/what/why/how)
  - Provide an example response (in brackets)
  - Break up complex tasks → subtasks / multiple prompts / "think step by step"
  - Watch context window/tokens; use **prompt templates**; experiment, iterate & test
- **Evaluate responses** — iterate; can use **one model copy to check another's output** (LLM-as-judge idea)
- **System prompt / role prompting** sets persona, rules and boundaries for the whole conversation; **role-based** prompting is a core technique
- **Structured output** → force JSON/XML via schema or tool definitions; validate with JSON Schema

### Basic techniques — zero-shot, few-shot, chain-of-thought
- **Zero-shot** — task, no examples. Works better on **larger LLMs**; **instruction tuning + RLHF** improve zero-shot
- **Few-shot** (in-context learning, typically 2–5 examples) — include examples (task + desired output)
  - Labels in examples **don't need to be correct** — random labels beat no labels; **label space + input distribution** matter most
  - Large example set → **dynamic prompt templates** with an **example selector based on semantic similarity** (stay within token limit)
- **Chain-of-thought (CoT)** — intermediate reasoning steps; trigger with **"Think step by step"**; works zero-shot or few-shot (few-shot CoT = more powerful). Use for multi-step reasoning/math

### Advanced techniques — self-consistency, ToT, ART, ReAct
- **Self-consistency** — like CoT but samples **multiple reasoning paths** (not the single greedy path) and **aggregates/majority-votes** the final answer. Fixes CoT arithmetic errors (sister-age example: single CoT = 25 ✗, self-consistency = 35 ✓)
- **Tree of thoughts (ToT)** — **branching** exploration with self-evaluation of choices; best for **planning, important initial decisions, exploring multiple solutions**. Game of 24: GPT-4 **4% with CoT → 74% with ToT**
- **RAG as a prompting technique** — retrieve relevant docs → add as context; **no weight changes** (details in [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md))
- **ART (Automatic Reasoning & Tool-use)** — builds on CoT for **multi-step reasoning**; picks few-shot demos from a **task library** + uses predefined **external tools** (search, code gen). Beats few-shot & auto-CoT on unseen tasks; update task library to fix errors
- **ReAct** — **reasoning traces + task-specific actions** with external tools (Wikipedia, SQL DB, calculator). Fixes CoT's inability to access external info → reduces **fact hallucination**. Thought → Action → Observation loop (calculator example: wrong without tool, right with)

### Inference parameters
- Adjust **one parameter at a time**; behaviour varies by FM; not all params on all models
- **Randomness & diversity:**
  - **Temperature** (`temperature`, 0–1, default 0) → randomness; low/0 ≈ deterministic, factual/extraction · high = creative/brainstorming
  - **Top K** (`top_k`) → only the K most probable tokens; lower = fewer unusual words, more predictable
  - **Top P** (`topP`, 0–1, default 1) → nucleus sampling: smallest token set whose cumulative probability sums to p; lower = more focused
- **Length:**
  - **Response length** (`maxTokenCount` / `max_tokens`, 0–8,000, default 512 on Titan; also MinTokens) → **hard limit**; cost control / truncated answers
  - **Length penalty** → **soft limit**, penalises longer responses
  - **Stop sequences** → strings that end generation (structured output, multi-turn templates)
- **numResults** → number of responses
- **Penalties** (Jurassic; presence/frequency model-dependent) → frequency / presence / count penalty to reduce repetition

### Model-specific prompting — Titan, Nova, Claude, Jurassic
- **Amazon Titan**
  - Specify length as **sentences/bullets/paragraphs, not word count**
  - Simple, clear, complete instructions; add context + put instruction/output indicator **at the end**
  - Provide **default output** when unsure ("if unsure, say 'I don't know'") → avoids plausible-but-wrong answers
  - Role in prompt for personalisation; can generate code
- **Amazon Nova**
  - Clear, structured, short, specific; specify output format/style (e.g. JSON only)
  - **Clear prompt sections with delimiters** (Markdown, XML, ##Instructions##) — also stops model revealing prompt parts
  - **System role** — persona, allowed/disallowed content; **supersedes user prompt instructions** and **carries across all turns**
  - CoT: "think step-by-step" / "DO NOT provide answer without thinking step by step" → accuracy + debuggability
  - Few-shot: **diverse** examples (common + edge cases), **matching complexity**, **relevant**
  - Provide supporting text (RAG); require **structured output** (JSON schema, no preamble)
  - Long context (Nova Premier 1M tokens): **long docs first, instructions at the end**, use \[Document Start\]/\[Document End\] markers
- **Anthropic Claude**
  - Legacy Text Completions format needs `\n\nHuman:` / `\n\nAssistant:` tags (trained with RLHF on that format) — Messages API uses user/assistant roles instead
  - Detailed task description "as to a new employee"; include rules, exceptions, examples
  - **Pre-fill the Assistant response** to limit verbosity / skip preamble (e.g. start with the opening XML tag)
  - **XML tags** to structure prompt and response; **instruction after long document**; tags for few-shot examples
  - Specify approx **words/paragraphs/items** (not characters)
  - Hallucinations → explain task more clearly, allow "I don't know"; break complex tasks up
- **AI21 Jurassic-2**
  - Good at summarisation, generation, Q&A; trained for **instruction-only (zero-shot)** prompts but also does few-shot
  - Specify number of sentences/items; say what to do if answer isn't in the docs
  - Use "Instruction:" label, separate sections with new lines
  - **Avoid negative formulations** ("no more than five sentences") → state the target ("in four sentences")
  - Long docs → **data first, then instructions, then output indicator**

### Multimodal & domain-specific prompting
- Multimodal models encode text/image/audio/video into a **shared representation space**
- Weak spots: **fine-grained detail, spatial reasoning, abstract visual concepts**; sensitive to image quality/lighting and prompt structure
- Multimodal techniques: **clear instructions referencing image + output format** · **contextual framing** (background info) · **sequential questioning** (broad → specific) · **role-based** ("As an interior designer…") · **iterative** refinement
- Domain principles: **contextual specificity** (human vs vet medicine; personal vs corporate finance) · **terminology precision** · **regulatory & ethical awareness** (remind of limits, human oversight) · **output formatting to domain conventions**
  - **Healthcare** — patient confidentiality, **HIPAA**, structured medical docs, accuracy/safety
  - **Finance** — precise numerical handling, time-sensitive data, regulations (**SEC, Basel**), risk frameworks
  - **Legal** — terminology, **citation formats, jurisdiction**-specific requirements

### Prompt misuse — injection, leaking, jailbreaks
- → See [11 · Security](11-security-identity-agent-security.md) (Prompt injection, prompt leaking & defences)

### Prompt Management, prompt history & Session Management APIs
- **Bedrock Prompt Management** → central catalog to create, test, **version**, share prompts
  - Structured prompts: system instructions + user message + optional **tools (function calling)**
  - **Variables** `{{customer_name}}` filled at runtime
  - **Compare variants** side by side (wording/model)
  - Versions are **immutable** → traceable history, rollback
  - Invoke from code via **Converse / InvokeModel passing the prompt ARN** + variables; works with **Prompt Flows**, LangChain, LlamaIndex
- **Prompt history store** → multi-turn beyond the **context window**; persists conversation state; also for **audit & compliance** and **debugging**; can avoid repeat FM requests
- **Bedrock Session Management APIs** → fully managed conversation state/context
  - Benefits: coherence, personalisation, **error recovery from last checkpoint**, debugging/traceability
  - Integrates with **LangGraph / LlamaIndex** via **checkpointing**; sessions can be **retrieved, replayed, forked**
  - Security: **IAM** access, encrypted by default (**KMS CMK** optional), **tenant isolation**
- **Prompt chaining** = output of one prompt feeds the next (Bedrock Flows, Step Functions)

### LangChain prompt templates, chains, memory & large-doc summarisation
- `PromptTemplate(input_variables=[...], template="...{var}...")` → `.format(...)`; include instructions, context, examples + dynamic variables → **reusable** prompt engineering
- **Chain** = set of components run together (LLM call, API, other chains); output of one → input of next
  - **LCEL (LangChain Expression Language)** = declarative, primary way to chain components · **legacy** subclassed `Chain` (e.g. **LLMChain** = LLM + prompt template + params → output, can parse to structured format)
  - **RouterChain** → **selects a prompt template based on user input**
  - Large data beyond context → chain **chunks the doc + multiple LLM calls** → single summary
- **Memory** → LLMs hold no state; memory stores & **summarises** prior conversation, re-injected into context
  - **ConversationBufferMemory** = most common, keeps past conversation; used with **ConversationChain** (follow-up "closest beach?" answered using earlier "Los Angeles")
  - LangChain adds **conversation chains** to text generation (orchestration layer between prompt and Bedrock FM)
- **Summarisation pattern** (depends on doc size vs context window): small → pass whole text directly · large → **map-reduce**: chunk (LangChain) → summarise chunk → **append next chunk to summary, re-summarise** → iterate (**chunking + prompt chaining**)

## Exam triggers
- "No examples given" → **zero-shot** · "Include a few labelled examples" → **few-shot** · "Show reasoning steps" → **CoT**
- "Multiple reasoning paths, pick most consistent answer" → **self-consistency**
- "Explore branches / planning / lookahead" → **Tree of thoughts**
- "Needs external tools/APIs to act, reduce hallucinated facts" → **ReAct**
- "Task library of demos + tools" → **ART**
- "Too many few-shot examples for context window" → **dynamic example selection by semantic similarity**
- "Factual, deterministic output" → **low temperature / top\_p / top\_k** · "Creative, diverse" → **high**
- "Stop generation at a marker" → **stop sequences** · "Reduce repetition" → **frequency/presence penalty**
- "Instructions must persist across all turns and override user" → **system prompt**
- "Skip Claude's preamble / force format" → **pre-fill assistant response** + **XML tags**
- "Long document + question" → **document first, instructions last**
- "Version, compare, reuse prompt templates with variables" → **Bedrock Prompt Management** (invoke by ARN)
- "Conversation exceeds context window / need audit trail of prompts" → **prompt history store**
- "Persist/resume multi-turn state, LangGraph checkpointing, fork session" → **Session Management APIs**
- "Summarise doc bigger than context window" → **chunk + chain (map-reduce)**
- "Store/summarise previous turns" → **memory** (ConversationBufferMemory) — not agents
- "Pick prompt template based on input" → **RouterChain**

## Flashcards
<details>
<summary><b>Q:</b> In the Clario clinical document case, what should be optimised before resorting to fine-tuning or continued pre-training, and which prompting technique improved accuracy?</summary>

Optimise the 4 pillars (parsing, retrieval, prompting, generation) first; zero-shot (and few-shot) prompting on Bedrock improved accuracy. Amazon MQ launched the async document-classification tasks.

*D1 · Clario accuracy*
</details>

<details>
<summary><b>Q:</b> How do you make Claude skip its preamble and follow a strict output format?</summary>

Pre-fill the Assistant response (e.g. with the opening XML tag) and use XML tags to structure prompt and response.

*D1 · Claude prompting*
</details>

<details>
<summary><b>Q:</b> There are too many few-shot examples to fit in the context window. What technique should be used?</summary>

Dynamic prompt templates with an example selector that picks examples by semantic similarity to the input.

*D1 · Dynamic few-shot*
</details>

<details>
<summary><b>Q:</b> In few-shot prompting, do example labels need to be correct?</summary>

No – even random labels beat no labels; the label space and input distribution matter most.

*D1 · Few-shot labels*
</details>

<details>
<summary><b>Q:</b> A use case needs factual, deterministic answers. How should temperature, top_p and top_k be set, and how should they be tuned?</summary>

Set them low (high = creative/diverse). Adjust one parameter at a time.

*D1 · Inference parameters*
</details>

<details>
<summary><b>Q:</b> A document is larger than the model's context window and must be summarised. What pattern?</summary>

Chunk + chain (map-reduce): chunk with LangChain, summarise a chunk, append the next chunk to the summary, re-summarise, iterate.

*D1 · Large doc summarisation*
</details>

<details>
<summary><b>Q:</b> When prompting with a long document plus a question, where should the instructions go?</summary>

Document first, instructions (and output indicator) at the end – recommended for Nova, Claude, Titan and Jurassic.

*D1 · Long-context ordering*
</details>

<details>
<summary><b>Q:</b> Instructions must persist across all turns and take precedence over user instructions in Nova. Where should they go?</summary>

In the system role/prompt, which supersedes user prompt instructions and carries across all turns.

*D1 · Nova system role*
</details>

<details>
<summary><b>Q:</b> What are the four elements of a prompt?</summary>

Instruction, context, input data, and output indicator (type/format of the output).

*D1 · Prompt elements*
</details>

<details>
<summary><b>Q:</b> A team needs a central catalog to version, compare and reuse prompt templates with runtime variables. What should it use?</summary>

Amazon Bedrock Prompt Management – immutable versions, `{{variables}}`, side-by-side variant comparison; invoke via Converse/InvokeModel with the prompt ARN.

*D2 · Prompt Management*
</details>

<details>
<summary><b>Q:</b> Which technique interleaves reasoning traces with actions using external tools to reduce fact hallucination, and which selects demos from a task library plus tools?</summary>

ReAct = Thought → Action → Observation with external tools (Wikipedia, SQL, calculator). ART = CoT + task library of demos + predefined tools.

*D1 · ReAct vs ART*
</details>

<details>
<summary><b>Q:</b> Which parameters stop generation at a marker and reduce repetitive output?</summary>

Stop sequences end generation at given strings; frequency/presence/count penalties (e.g. on Jurassic) reduce repetition.

*D1 · Repetition control*
</details>

<details>
<summary><b>Q:</b> Chain-of-thought gives inconsistent arithmetic answers. Which technique samples multiple reasoning paths and majority-votes the result?</summary>

Self-consistency.

*D1 · Self-consistency*
</details>

<details>
<summary><b>Q:</b> In the Snoonu case, what was the key strategic shift for recommendation precision and what role did Amazon MQ (RabbitMQ) play?</summary>

Moving from one global Amazon Personalize model to separate models per vertical; Amazon MQ logged recommendation requests to compare pre/post filtering.

*D1 · Snoonu personalization*
</details>

<details>
<summary><b>Q:</b> How should you specify output length and avoid plausible-but-wrong answers with Amazon Titan?</summary>

Specify length in sentences/bullets/paragraphs (not word count) and provide a default output such as 'if unsure, say I don't know'.

*D1 · Titan prompting*
</details>

<details>
<summary><b>Q:</b> Which prompting technique explores branching reasoning with self-evaluation and suits planning tasks?</summary>

Tree of Thoughts (ToT) – e.g. Game of 24 accuracy rose from 4% with CoT to 74% with ToT on GPT-4.

*D1 · Tree of thoughts*
</details>

---
[← Back to index](README.md)
