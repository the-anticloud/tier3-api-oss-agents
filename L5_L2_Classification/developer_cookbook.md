# Developer Cookbook — api-oss-agents
**Stack:** Python 3.11, asyncio, PAX 27B, AIOSS_FORMAT, ZeroMQ
**Domain:** Agent orchestration layer: multi-agent task decomposition and coordination for Anticloud API
**License:** Apache-2.0 | **IP:** USPTO pending 2026, Anticloud FZ LLE

## Core Usage

```python
from api_oss_agents import AgentOrchestrator

orch = AgentOrchestrator(pax_model="./pax-27b-q4.gguf", aioss_chain="./agents.aioss")

# Decompose and execute
result = await orch.execute(
    goal="Audit this Python codebase for OWASP vulnerabilities and generate a remediation plan",
    context={"codebase_path": "./TIER_4_INFERENCE_AGENTS/K_SGLANG"},
    max_steps=10
)
print(result.synthesis, result.chain_hash)
```

```python
# Register custom agent
@orch.register_agent("SECURITY_AUDITOR")
async def security_audit_agent(task, context):
    # Uses K_VLMEVAL under the hood
    return audit_result

# Multi-agent parallel execution
results = await orch.parallel([task_a, task_b, task_c])
```

## AIOSS Chain Append

```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

# After every api-oss-agents output:
chain_hash = aioss_append("./api_oss_agents.aioss",
                           result_bytes, "api-oss-agents")
```

## Performance & Integration

Parallel subtask execution via asyncio.gather(). Max 8 concurrent agent tasks per orchestrator. PAX 27B task decomposer cached between calls. ZeroMQ DEALER/ROUTER for agent communication. Integration: dispatches to PAX_PLANNING (T2), PAX_REASONING (T2), domain-specific tier modules.
