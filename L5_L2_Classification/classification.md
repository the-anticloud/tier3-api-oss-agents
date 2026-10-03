# L5 Narrow / L2 General Classification — api-oss-agents
**Platform:** Anticloud | **PAX:** 27B | **IP:** USPTO pending 2026, Anticloud FZ LLE
**Domain:** Agent orchestration layer: multi-agent task decomposition and coordination for Anticloud API

## L5 Narrow
api-oss-agents specializes in decomposing complex API requests into multi-step agent tasks, assigning each to the appropriate PAX module, collecting results, and synthesizing final responses. Narrow scope: Anticloud agent topology only.

## L2 General
L2 General: any Anticloud project that needs multi-step reasoning — clinical diagnosis workflow, robotics mission planning, security audit pipeline — uses api-oss-agents as the orchestration layer.

## PAX Integration
PAX 27B acts as both the task decomposer (breaking requests into subtasks) and the result synthesizer (combining subtask outputs into coherent responses). Each agent step is AIOSS-chained.

## AIOSS Audit Relevance
Every agent task event (task decomposition hash + subtask assignments + completion statuses + final synthesis hash) is AIOSS-chained: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n).
Tamper-evident, offline-verifiable, zero cloud dependency.

## Regulatory / Compliance
NIST AI RMF 1.0 (multi-agent accountability), ISO/IEC 42001 (AI system governance)
