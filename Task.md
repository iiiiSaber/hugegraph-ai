# OSPP: HugeGraph Graph Reasoning Based on Ontogeny

- **Project ID:** `26ec80107`
- **Official listing:** [基于 Ontogeny 的 HugeGraph 图推理方案](https://summer.ospp.ac.cn/org/prodetail/26ec80107?lang=zh&list=pro)
- **Duration:** Four months
- **Difficulty:** Advanced
- **Main deliverable:** A standalone Python package named `ontogeny-reasoning`

## Objective

Build a reusable graph-reasoning engine that Ontogeny and other applications can use to plan and execute graph queries and computations reliably.

Existing graph capabilities are scattered across modules. The project brings them behind consistent interfaces, execution controls, tracing, and evaluation.

```text
Ontogeny
  Business objects, permissions, and actions
                         ↓
ontogeny-reasoning
  Graph query and computation planning and execution
                         ↓
HugeGraph
  Graph storage and database operations
```

Ontogeny integration is one use case. The reasoning package must also work independently of the full platform.

## 1. Define shared contracts

The listing specifies three common structures:

| Structure | Meaning |
|---|---|
| `Task` | What the caller wants to find or compute |
| `Plan` | How the engine intends to execute the task |
| `Result` | What execution produced |

For illustration, imagine extending Tiny Library with authors and relationships between authors. A request could be “Find available books written by authors connected to this author.”
The engine would produce an executable plan and return matching graph identifiers. Ontogeny could then retrieve the business objects and apply its visibility rules.

The precise fields and semantics of these structures are part of the design work; the example above is explanatory rather than a prescribed API.

## 2. Implement interchangeable providers and routing

Providers implement graph capabilities behind the common contracts.

| Provider | Required capabilities |
|---|---|
| Text2Gremlin | Natural language → intermediate representation → Gremlin; schema constraints, static validation, and one repair attempt |
| Traversal | k-hop traversal, shortest path, weighted shortest path, subgraph, and reachability |
| Graph algorithms | PageRank, WCC, SSSP, BFS, CDLP, and LCC |

The named algorithms cover influence ranking, weakly connected components, single-source shortest paths, breadth-first search, community detection using label propagation, and local clustering coefficients.

Reuse and adapt existing HugeGraph capabilities where suitable. Establish which backend operations satisfy the required semantics instead of assuming similarly named APIs are equivalent.

Providers must declare capabilities and support registration through Python entry points. Routing selects providers by task type and cost and supports failure fallback.

## 3. Unify governance, tracing, and failure handling

All execution paths must share:

- **Budgets:** Constrain execution resources and account for their consumption.
- **Output and masking rules:** Follow the listing's ID-only and edge-endpoint contract, preserving the platform's governed object-retrieval boundary.
- **Tracing:** Connect natural-language input, intermediate representation, execution, and results through a shared trace schema and spans.
- **Auditing:** Check plan consistency, ID-only/masking compliance, and budget reconciliation.
- **Failure handling:** Support fallback, failure diagnosis, and replay.

Deliver a **conformance kit** that checks whether providers obey these shared contracts.

The listing identifies practical challenges including schema matching, invalid queries, asynchronous Job API timeouts and result parsing, variable-length path deduplication, GraphComputer capability boundaries, and large-graph timeouts.

## 4. Provide independent interfaces

Expose the package through:

- A Python SDK.
- HTTP endpoints.
- A command-line interface.

Accept schema information from Ontogeny snapshots or HugeGraph. The package must not depend on Ontogeny's Postgres database, DSL, or frontend.

Real graph execution requires a HugeGraph instance. Natural-language generation requires an LLM API or a local Ollama deployment.

## 5. Deliver reproducible evaluation

| Benchmark | Evaluation focus |
|---|---|
| CypherBench | Natural-language graph queries: execution accuracy (EX), PSJS, and invalid-query rate |
| LDBC SNB | Multi-hop correctness and performance: consistency, p95 latency, and engine overhead |
| LDBC Graphalytics | Graph algorithm correctness/pass count and runtime |
| HotpotQA/MuSiQue | Optional agent multi-hop question answering |

Deliver evaluation scripts and a report recording datasets, environment, baselines, applicable metrics, and comparative results. Validate effectiveness in common practical scenarios.

Agree on acceptance thresholds with the mentor early in the project. The listing does not prescribe numerical thresholds.

Update documentation with installation, configuration, usage, and typical examples.

## Existing code to build on

- [`hugegraph-llm`](./hugegraph-llm/): Text2Gremlin and GraphRAG application capabilities.
- [`hugegraph-llm-sdk`](./hugegraph-llm-sdk/): Standalone schema design and reviewed extraction contracts.
- [`hugegraph-python-client`](./hugegraph-python-client/): HugeGraph query, traversal, and OLAP APIs.
- [`ontogeny`](./ontogeny/): Ontology snapshots, schema definitions, permissions, actions, and agent governance on the `ontogeny` branch.

## Scope to clarify with the mentor

- **Agent integration:** The detailed description and remarks call the Agent path optional, while an output requirement mentions combining the providers with an Agent.
- **Supported backends:** Agree on HugeGraph/GraphComputer versions and verified algorithm capabilities.
- **Benchmark scope:** Agree on dataset sizes, subsets, baselines, and acceptance thresholds. CypherBench's Cypher workloads need a defined evaluation approach for Gremlin execution.
- **Result contract:** Clarify how algorithm scores, distances, and component assignments fit the ID-only output requirement.
- **Replay:** Define whether this means replaying recorded responses, reproducing a plan, or rerunning against live data.

The core engineering work is making several graph-execution paths behave like one dependable library. Agent integration consumes that library; shared contracts, providers, controls, and evaluations are the central deliverables.
