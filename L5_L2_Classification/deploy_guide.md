# Deploy Guide — api-oss-agents
**Platform:** Anticloud sovereign infrastructure | Air-gap capable
**Stack:** Python 3.11, asyncio, PAX 27B, AIOSS_FORMAT, ZeroMQ

## Prerequisites
Python 3.11+, asyncio (stdlib), pyzmq 25.0+, PAX 27B weights

## AIOSS Integration
```bash
aioss init --module api-oss-agents --output ./api_oss_agents.aioss
aioss append --chain ./api_oss_agents.aioss --payload ./output.bin --module api-oss-agents
aioss verify --chain ./api_oss_agents.aioss
```

## Air-Gap Deployment
```bash
# On networked machine:
pip download -r requirements.txt -d ./wheels/
# Transfer wheels/ to air-gap host, then:
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX 27B Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(
    model_path="./pax-27b-q4.gguf",
    module="api-oss-agents",
    aioss_chain="./api_oss_agents.aioss",
    classification="L5_NARROW_L2_GENERAL"
)
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./api_oss_agents.aioss --verbose
python -m api_oss_agents.tests.smoke
```
