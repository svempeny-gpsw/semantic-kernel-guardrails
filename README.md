
# semantic-kernel-guardrails

Examples showing Semantic Kernel usage with model guardrail guidance.

This folder contains the preparation material that demonstrates how to use the Semantic Kernel Python SDK with model deployments and how to reason about safety by combining Microsoft Foundry guardrails with lightweight application-level checks.

Contents
- `Guardrails.md` — Lab-style walkthrough for applying Foundry guardrails.  
- `SemanticKernel_Guardrails.md` — Summary of the `00-getting-started.ipynb` notebook, runnable Semantic Kernel examples, and suggested client-side safety checks.  
- `semantic-kernal/00-getting-started.ipynb` — Notebook used as the source for examples and setup instructions.

Quick start

1. Install dependencies:

```bash
pip install -U semantic-kernel python-dotenv
```

2. Copy the example environment file and populate credentials (`.env`) in the `Week5/semantic-kernal` directory. See the notebook for example keys for `OpenAI` and `AzureOpenAI`.

3. Run the notebook or follow examples in `SemanticKernel_Guardrails.md` to initialize a `Kernel()`, add a chat service, load a plugin, and invoke semantic functions.

Safety notes

- Apply Foundry guardrails via the Foundry portal to your model deployment to enforce server-side content filtering and blocking.  
- Add client-side prompt and completion checks (examples in `SemanticKernel_Guardrails.md`) for business-specific rules, faster validation, and improved user messaging.  
- Log guardrail/triggers and implement human review for high-risk items.

Next steps

- Tune guardrail thresholds in Foundry and iterate with logs and test prompts.  
- Expand examples to show logging, metrics, and escalation flows.  


