---
name: attemory-code
description: "Use this skill for natural-language semantic search in an Attemory-indexed code repository: implementation lookup, definitions, entry points, call sites, architecture, config flow, tests, errors, or cross-file behavior."
---

# Attemory Code Search

Attemory Code provides semantic code search over repositories already indexed
with the `atcode` CLI. Use the MCP `search` tool to locate compact file/range
evidence before broad grep/read exploration, before editing unfamiliar code, and
before answering repository understanding questions.

Attemory search is not keyword search. Natural-language questions work best.

## When To Search

Call `search` for repository questions such as:

- where a behavior is implemented
- which method owns an architectural responsibility
- where a symbol is defined, exported, configured, validated, or dispatched
- which code path handles an error, request, CLI command, config option, or test
- how data or dependencies flow across files
- which files should be read before making a code change

When the user asks a semantic code question, call Attemory search first instead
of starting with broad grep or file reads. Then read the returned file ranges.

## Search Tool

Call the MCP `search` tool with:

- `query`: the final natural-language repository question.
- `repo_root`: the repository root, or any path inside it. Prefer passing this
  explicitly when the target repo/path is known.
- `query_context`: optional request-time facts or retrieval guidance.
- `display_top_k`: optional number of fused file/range results to return.
- `candidate_chunk_top_k`: optional candidate pool size before reranking.
- `include_snippets`: false by default. Set true only when file/range evidence
  is not enough.

The tool returns file/range evidence and a compact
`<semantic_search_results>` context block. After search, read the returned file
ranges before giving a final answer or editing code.

## Writing Good Queries

Use a sentence that describes the behavior or responsibility to find. Include
known symbols, APIs, files, error messages, command names, or user-visible
effects when available.

Good queries:

```text
Where does session restore load persisted segment KV state and update session status?
Find the method that collects, validates, and filters dependencies from multiple build contexts for pkg-config generation.
Which route handles session search requests and maps backend errors to HTTP responses?
Where is the resident KV RAM budget enforced during multi-segment search?
What code decides when a repository file starts a new segment during indexing?
```

Poor queries:

```text
restore
session
index bug
dependencies
```

Poor queries are too keyword-like. Rewrite them as repository questions with
the behavior, responsibility, or failure mode you need to understand.

## Query Context

Use `query_context` when the user gives extra request-time facts, constraints,
or retrieval guidance that should shape ranking but should not itself be
retrieved as evidence.

Attemory places `query_context` between the indexed repository memories and the
final query. It helps the model interpret the question and rank memories.
`query_context` is not a candidate memory and is not returned as a search
result.

Good uses for `query_context`:

- narrow the search to a subsystem, feature, protocol, or runtime path
- explain domain vocabulary from the user's question
- include a concise error message, stack-frame clue, or observed behavior
- state what evidence should be preferred, such as implementation over tests
- disambiguate a common term or symbol

Do not put large source files, long logs, or the desired final answer into
`query_context`. Summarize only the facts needed to guide retrieval.

Example:

```text
query:
Find where repository indexing decides to start a new segment before adding a file.

query_context:
Prefer code in the Attemory Code CLI or project indexing layer. Look for logic
that compares remaining token budget with context_window and
file_boundary_reserve_percent. Return implementation code before docs or tests.
```

Example:

```text
query:
Where is the pkg-config dependency collection responsibility implemented?

query_context:
The user is asking about code that gathers dependencies from host/build
contexts, validates them, filters unusable entries, and emits pkg-config file
content. Prefer generator implementation paths and direct helper methods.
```

If the first search is broad, run a second search with a more specific query or
add `query_context` with the missing constraints. If results are too sparse,
increase `candidate_chunk_top_k` before increasing `display_top_k`.

## Result Handling

Use the returned file/range list as evidence pointers, not as a final answer by
itself.

Recommended flow:

1. Call `search` with a specific natural-language query.
2. Read the most relevant returned file ranges.
3. If the evidence is incomplete, search again with a refined query or
   `query_context`.
4. Answer or edit only after reading the relevant code.

Set `include_snippets=true` only when the agent cannot read files directly or
when the user specifically wants snippets in the tool result. Otherwise keep it
false to save context.

## Index Lifecycle

Do not trigger heavy indexing implicitly through MCP. The MCP tool is read-only
repository search.

If search reports that the repo is not initialized or indexed, tell the user to
run:

```bash
atcode init
atcode index
```

If indexing was interrupted after memories were added, tell the user to run:

```bash
atcode index --resume
```

Use `atcode reindex` or `atcode reset` only when the user explicitly asks for a
clean rebuild or removal. Do not initialize, index, reindex, reset, or modify
repositories unless the user requests that action.
