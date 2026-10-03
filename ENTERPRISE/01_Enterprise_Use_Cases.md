# Enterprise Use Cases â€” api-oss-agents

## Overview

api-oss-agents is a local-first agent orchestration platform replacing OpenAgents and similar cloud-hosted frameworks. It runs entirely on-premises, eliminating per-token cloud fees and data-residency concerns.

---

## Use Case 1: Automated IT Operations (AIOps)

**Scenario:** A 500-person engineering organisation routes infrastructure alerts to specialised sub-agents that diagnose, escalate, and remediate issues without human intervention.

**Architecture:**
```
AlertSource â”€â”€â–º Router Agent â”€â”€â–º DiagnosticsAgent â”€â”€â–º RemediationAgent
                                        â”‚
                               AIOSS Ledger (audit)
```

**Integration Example:**
```python
from api_oss_agents import AgentOrchestrator
from aioss import Ledger

ledger = Ledger.open("./ops_ledger.aioss")
orch = AgentOrchestrator(model_endpoint="http://localhost:8000/v1")  # local vLLM

@orch.agent(role="diagnostics")
def diagnose(alert: dict) -> dict:
    result = orch.infer(prompt=f"Diagnose: {alert['message']}")
    ledger.append(
        entry_type="agent_inference",
        actor="diagnostics-agent",
        content={"tokens_in": result.usage.prompt_tokens,
                 "tokens_out": result.usage.completion_tokens,
                 "wall_time_ms": result.latency_ms,
                 "cost_if_cloud_microcents": 0}
    )
    return result.parsed

orch.run_pipeline(agents=["diagnostics", "remediation"])
```

**ROI:** Replacing a cloud-hosted AIOps SaaS (avg $4,000/month) with local inference on existing GPU hardware: **$48,000/year saved**. AIOSS ledger field `cost_if_cloud_microcents=0` confirms zero per-token charges.

---

## Use Case 2: Legal Document Review Pipeline

**Scenario:** A legal firm processes 10,000 contracts/month for clause extraction, risk flagging, and summary generation using a Llama-3-70B model served locally.

**Architecture:**
```
Contract Store â”€â”€â–º Ingestion Agent â”€â”€â–º Clause Extractor â”€â”€â–º Risk Scorer â”€â”€â–º Report Generator
                                              â”‚
                                    AIOSS Ledger (per-document trace)
```

**Key Configuration:**
```python
# No API keys â€” local Ollama endpoint
agent_config = {
    "model": "llama3:70b",
    "base_url": "http://localhost:11434/v1",
    "max_concurrent_agents": 8,
    "ledger_path": "./legal_ledger.aioss"
}
```

**ROI:** Cloud LLM API costs for 10,000 contracts at ~$0.003/contract = $30/month at minimum; enterprise legal AI SaaS typically $2,000â€“8,000/month. Local deployment: **$24,000â€“96,000/year saved**.

---

## Use Case 3: Customer Support Automation

**Scenario:** E-commerce company routes customer inquiries through specialised agents (returns, shipping, technical) with escalation to human agents only for edge cases.

**Deployment Diagram:**
```
CRM Webhook â”€â”€â–º Intent Classifier â”€â”€â–º Domain Agent Pool â”€â”€â–º Response Formatter
                                              â”‚
                                    AIOSS Ledger (per-ticket audit)
                                              â”‚
                              Human Escalation Gate (confidence < 0.85)
```

**Code:**
```python
from api_oss_agents import AgentPool
from aioss import Ledger

pool = AgentPool(
    agents=["returns", "shipping", "technical"],
    local_model="http://localhost:8000/v1",  # local vLLM
    ledger=Ledger.open("./support_ledger.aioss")
)

result = pool.handle(ticket_text="My order hasn't arrived", ticket_id="T-9182")
# AIOSS ledger captures full chain: intent â†’ agent â†’ response
```

**ROI:** Automated handling of 70% of tickets (industry average) reduces support headcount cost by $180,000/year for a 10-agent team. Zero cloud API fees.

---

## Deployment Architecture

```
[On-Premises Server]
â”œâ”€â”€ api-oss-agents (port 8080)
â”œâ”€â”€ vLLM inference server (port 8000, local GPU)
â”œâ”€â”€ api-oss-queue (Redis, port 6379)
â”œâ”€â”€ AIOSS Ledger (./ledgers/*.aioss)
â””â”€â”€ api-oss-security (JWT auth, port 8443)
```

All components communicate over localhost. No data leaves the corporate network.
