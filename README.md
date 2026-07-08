# Awesome Agent Memory Benchmark

[中文](README.zh-CN.md) | English

Although general-purpose LLMs are becoming increasingly capable as agents, the quality of memory use remains a key factor in whether agents can succeed in concrete business scenarios. This naturally raises several questions: What makes a good Agent Memory System? How should such a system be evaluated? How are existing Agent Memory Benchmarks constructed? This repo is an attempt to answer those questions by curating recent benchmark work and summarizing their evaluation goals, capability evolution, and dataset construction patterns.

It provides a roadmap of benchmarks for evaluating long-term memory in LLM agents, from early session-continuity and personal recall tests to longitudinal state tracking, implicit memory use, multi-party memory, and memory-to-action evaluation.

## Overview

![Research roadmap of long-term agent memory benchmarks](./assets/memory_bench.png)

*Research roadmap of long-term agent memory benchmarks.*

## Project Scope

Long-term memory is becoming a core capability for LLM agents. A useful memory system should not only retrieve old facts, but also maintain the current state of a user, task, relationship, or group over time.

This repo organizes agent memory benchmarks around one central shift:

> Agent memory evaluation is moving from factual recall to longitudinal state tracking.

Early benchmarks ask whether an agent can remember a fact from previous sessions. Newer benchmarks ask whether it can integrate evidence across sessions, ground states in time, overwrite outdated facts, infer implicit constraints, keep speaker-specific states, and convert memory into actions.

## Benchmark Roadmap

From the perspective of research questions, the development of Agent Memory Benchmarks is not simply about making contexts longer. It progressively decomposes "remembering history" into more specific system capabilities: maintaining continuity across sessions, retrieving personal facts and events, reasoning across sessions, updating states, applying implicit constraints, grounding states to specific speakers, and using memory for actions.

| Stage | Core Question | Representative Benchmarks |
|---|---|---|
| Session continuity | Can the agent avoid forgetting across sessions? | MSC |
| Personal factual memory | Can the agent remember user facts, preferences, relationships, and events? | MemoryBank, PerLTQA |
| Long-context and cross-session reasoning | Can the agent reason over long histories with multi-hop, temporal, and unanswerable questions? | LoCoMo, LongMemEval |
| Dynamic state maintenance | Can the agent handle state changes, conflicts, overwrites, and forgetting? | LongMemEval, MemoryAgentBench, EvoEmo |
| Implicit and abstract state | Can the agent infer goals, preferences, emotions, or profiles from indirect cues? | LoCoMo-Plus, MemBench, EvoEmo |
| Context-conditioned state use | Can the agent use memory according to speaker, task, group context, or tool schema? | GroupMemBench, Mem2ActBench |

## Capability Taxonomy

If the roadmap is a timeline, the taxonomy below is a capability-level view. A single benchmark often tests multiple capabilities: LongMemEval involves evidence integration, temporal reasoning, state update, and abstention; GroupMemBench involves multi-party dialogue, speaker-specific state, and cross-evidence reasoning. This view helps identify which parts of an Agent Memory System are strong and where it is likely to fail.

| Capability | Problem It Tests | Benchmarks |
|---|---|---|
| Evidence integration | How to combine scattered evidence into one state | LongMemEval, LoCoMo, GroupMemBench |
| Temporal grounding | Which state is earlier, later, expired, or still valid | LongMemEval, LoCoMo, DialSim |
| State update / overwrite | What the current state is when old and new facts conflict | LongMemEval, MemoryAgentBench, EvoEmo |
| Implicit constraint application | Whether unstated goals or constraints can guide future responses | LoCoMo-Plus |
| State abstraction | How to infer preferences, emotions, and profiles from events | MemBench, EvoEmo, PerLTQA |
| Speaker-specific state | Whose state, belief, task, or preference should be used | GroupMemBench |
| Memory-to-action | Whether memory can be converted into tool parameters or actions | Mem2ActBench |

## Useful Benchmark Construction Patterns

These works are valuable not only because they introduce new evaluation sets, but also because they provide reusable dataset-construction patterns. Compared with simply asking an LLM to generate many QA pairs, a more reliable approach is to first design controllable latent states, event chains, or conflict chains; then generate conversations and questions around those states; and finally filter out samples that leak answers, are too easy, or are not truly memory-dependent.

For example, to evaluate a user's current valid address, we should not simply ask an LLM to generate a random conversation and question. A better construction first defines a state evolution chain: the user originally lived in Beijing, later planned to move to Shanghai, eventually moved to Shanghai, and may have temporarily traveled to Shenzhen. Multi-session conversations and the question "Where should we send the materials now?" can then be generated around this chain. Such a sample tests state update and temporal validity, rather than keyword matching over city names that appeared in history.

| Pattern | Used By | Why It Is Useful |
|---|---|---|
| Latent state / event graph before dialogue generation | LoCoMo, GroupMemBench | Keeps gold states and evidence controllable |
| Cue-trigger pair construction | LoCoMo-Plus | Tests implicit memory activation under low lexical similarity |
| Solve-Judge-Refine filtering | GroupMemBench | Keeps questions solvable but challenging |
| No-memory discriminator | Mem2ActBench | Removes samples where the query leaks the answer |
| Conflict / update chains | LongMemEval, MemoryAgentBench, EvoEmo, Mem2ActBench | Tests current valid state rather than historical mention |
| Real or semi-real data conversion | LongMemEval, MemBench, Mem2ActBench | Improves realism while keeping controlled evidence |
| Human-authored or human-reviewed core questions | LongMemEval, EvoEmo | Balances quality and scale |
| Semantic filtering against surface matching | LoCoMo-Plus | Prevents benchmarks from becoming keyword retrieval tests |
| Retrieval baseline reverse filtering | LoCoMo-Plus, GroupMemBench, Mem2ActBench | Removes samples that are too easy for BM25 or embedding retrieval |
| Hard negatives and unanswerable questions | LoCoMo, LongMemEval, DialSim, GroupMemBench | Tests abstention and reduces hallucinated answers |
| Structured metadata retention | GroupMemBench, DialSim, LoCoMo | Enables speaker-aware, thread-aware, and time-aware evaluation |
| Event-level and state-level evaluation | LoCoMo, EvoEmo | Evaluates whether summaries or answers preserve key state changes |
| Multi-granularity task suites | MemBench, MemoryAgentBench | Avoids overfitting to one QA format |

## Recommended Design Recipe

If you are building a new agent memory benchmark, the most reusable recipe is:

1. Define a latent state graph with users, speakers, events, time, tasks, preferences, conflicts, and validity intervals.
2. Generate multi-session conversations from that state graph.
3. Add cue-trigger pairs for implicit memory use.
4. Add conflict chains to test current-state tracking.
5. Add unanswerable and hard-negative samples.
6. Filter with no-memory, BM25, embedding, and strong LLM baselines.
7. Keep structured metadata such as speaker, timestamp, session id, topic, reply-to, state id, and evidence id.
8. Evaluate answer accuracy, evidence recall, state accuracy, abstention accuracy, tool-parameter accuracy, and state-level summary quality.

Returning to the questions raised at the beginning: What makes a good Agent Memory System? The answer from these benchmarks is not simply "storing more history" or "retrieving from a longer context." A strong Agent Memory System should maintain session continuity over long-term interaction, retrieve personal facts and events accurately, understand temporal order and state overwrite, recognize implicit constraints and abstract user states, distinguish state ownership in multi-party settings, and convert memory into responses or actions when needed. 

## Benchmarks
The list below summarizes the evaluation goal, construction style, and main value of each benchmark along this research trajectory.

### MSC: Beyond Goldfish Memory

- Year: 2022
- Venue: ACL
- Paper: https://aclanthology.org/2022.acl-long.356/
- Focus: session continuity in long-term open-domain conversation

MSC studies whether dialogue models can maintain continuity across sessions. It builds multi-session conversations from PersonaChat-style interactions and evaluates whether models can naturally reuse information from previous sessions.

Why it matters: MSC is an early benchmark that makes long-term conversational memory a concrete evaluation problem. It mainly measures whether models can avoid forgetting previous-session context.

### MemoryBank

- Year: 2024
- Venue: AAAI
- Paper: https://arxiv.org/abs/2305.10250
- Code: https://github.com/zhongwanjun/MemoryBank-SiliconFriend
- Focus: personal recall and AI companion memory

MemoryBank evaluates long-term memory in simulated AI companion scenarios. It synthesizes user histories and probing questions to test whether a system can retrieve relevant memories, answer user-specific questions, and generate personalized responses.

Why it matters: MemoryBank moves from long-context conversation to an explicit memory system with extraction, storage, retrieval, update, and forgetting.

### PerLTQA

- Year: 2024
- Venue: ACL
- Paper: https://arxiv.org/abs/2402.16288
- Code: https://github.com/Elvin-Yiming-Du/PerLTQA
- Focus: personal long-term QA over semantic and episodic memory

PerLTQA builds synthetic long-term memories for fictional characters, including profiles, relationships, events, and dialogues. It evaluates memory classification, memory retrieval, and memory-grounded answer synthesis.

Why it matters: PerLTQA separates semantic memory from episodic memory, making it useful for studying personal profiles, relationships, and event memories.

### DialSim

- Year: 2024
- Paper: https://arxiv.org/abs/2406.13144
- Code: https://github.com/jiho283/DialSim
- Focus: role knowability in long-term multi-party dialogue

DialSim evaluates agents in long-running multi-party dialogue simulation. Agents answer questions from the perspective of a role, using only information that the role should know at that point in time.

Why it matters: DialSim introduces role knowability and temporal access constraints. The agent must know not only what happened, but also who could know it and when.

### LoCoMo

- Year: 2024
- Venue: ACL
- Paper: https://arxiv.org/abs/2402.17753
- Code: https://github.com/snap-research/LoCoMo
- Focus: event state tracking in very long-term conversation

LoCoMo builds very long, multi-session, multimodal conversations from personas and temporal event graphs. It evaluates single-hop, multi-hop, temporal, open-domain, and adversarial unanswerable QA, as well as event summarization and multimodal dialogue generation.

Why it matters: LoCoMo is valuable because it generates conversations from latent event graphs. This makes gold states, evidence, temporal order, and event evolution more controllable.

### LongMemEval

- Year: 2025
- Venue: ICLR
- Paper: https://arxiv.org/abs/2410.10813
- Code: https://github.com/xiaowu0162/LongMemEval
- Focus: assistant state update across long-term interactive memory

LongMemEval evaluates long-term memory in chat assistants. It covers information extraction, multi-session reasoning, temporal reasoning, knowledge update, and abstention.

Why it matters: LongMemEval is a key transition point from factual recall to longitudinal state tracking. It asks whether an assistant can maintain the current valid state across long histories.

### MemBench

- Year: 2025
- Paper: https://arxiv.org/abs/2506.21605
- Code: https://github.com/import-myself/Membench
- Focus: comprehensive memory evaluation, including reflective abstraction

MemBench evaluates LLM-agent memory in participation and observation scenarios. It distinguishes factual memory from reflective memory and measures answer accuracy, retrieval recall, memory capacity, and temporal efficiency.

Why it matters: MemBench expands memory evaluation beyond QA accuracy. It tests whether memory systems can abstract high-level preferences and remain efficient at scale.

### MemoryAgentBench

- Year: 2026
- Paper: https://arxiv.org/abs/2507.05257
- Code: https://github.com/HUST-AI-HYZ/MemoryAgentBench
- Focus: incremental multi-turn memory, state overwrite, and forgetting

MemoryAgentBench injects memory incrementally through multi-turn interactions, then evaluates retrieval, test-time learning, long-range understanding, and selective forgetting.

Why it matters: It treats forgetting and overwrite as first-class memory abilities. A strong memory agent should not keep every old fact equally; it should know which facts are still valid.

### LoCoMo-Plus

- Year: 2026
- Venue: ACL
- Paper: https://aclanthology.org/2026.acl-long.1150/
- Code: https://github.com/xjtuleeyf/Locomo-Plus
- Focus: implicit application of cognitive memory

LoCoMo-Plus inserts implicit cues into long conversations and later asks low-similarity trigger queries. The task is to test whether an agent can apply previously implied user goals, constraints, or preferences.

Why it matters: It avoids shallow keyword retrieval and tests whether implicit memory can be activated by future situations.

### EvoEmo / ES-MemEval

- Year: 2026
- Venue: WWW
- Paper: https://arxiv.org/abs/2602.01885
- Focus: affective evolution and personalized long-term emotional support

EvoEmo evaluates whether conversational agents can track evolving emotional states, stressors, relationships, and support needs over long-term interactions.

Why it matters: It extends memory evaluation from facts and preferences to emotional state, causal event timelines, and personalized support.

### Mem2ActBench

- Year: 2026
- Venue: ACL
- Paper: https://aclanthology.org/2026.acl-long.370/
- Code: https://github.com/Cantaloupe-M/Mem2ActBench
- Focus: memory-to-action in task-oriented autonomous agents

Mem2ActBench asks agents to perform tool calls where key parameters are omitted from the current query and must be recovered from long-term memory.

Why it matters: It moves memory evaluation from answering to acting. The benchmark tests whether user states, preferences, and constraints can become correct tool-call parameters.

### GroupMemBench

- Year: 2026
- Paper: https://arxiv.org/abs/2605.14498
- Code: https://github.com/UCSB-NLP-Chang/GroupMemBench
- Focus: speaker-specific state in multi-party conversations

GroupMemBench evaluates memory in group conversations. The agent must retrieve and reason over states that belong to specific speakers, tasks, threads, and group contexts.

Why it matters: It shows that group memory is not just more dialogue. The central challenge is speaker-grounded belief tracking and avoiding state misattribution.

## Citation Note

This repo is a curated research list. Please cite the original papers when using their datasets, code, or benchmark designs.
