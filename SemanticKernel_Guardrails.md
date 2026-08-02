
# Semantic Kernel + Guardrails

This document summarizes the `00-getting-started.ipynb` Semantic Kernel notebook and shows example code for using Semantic Kernel with a model service and simple client-side guardrail checks. It also explains how Microsoft Foundry guardrails work and how to combine them with application-level safety checks.

File referenced: [Week5/semantic-kernal/00-getting-started.ipynb](Week5/semantic-kernal/00-getting-started.ipynb)

## Summary of the notebook

- Installs the `semantic-kernel` Python package.
- Loads environment config from a `.env` file to select a LLM service (OpenAI, AzureOpenAI, or HuggingFace).
- Initializes a `Kernel()` instance and registers an LLM chat service (OpenAI or Azure OpenAI).
- Loads a plugin (e.g., `FunPlugin`) and invokes a semantic function (e.g., `Joke`).

## Prerequisites

- Python 3.8+ and pip
- `pip install -U semantic-kernel python-dotenv`
- A `.env` file in the notebook directory with service credentials (see the notebook for example `.env` contents). Example keys shown in the notebook for `AzureOpenAI` and `OpenAI`.

## Minimal setup (from the notebook)

Install packages:

```bash
pip install -U semantic-kernel python-dotenv
```

Create a `.env` in the same folder and include the relevant keys. Example (Azure OpenAI):

```ini
GLOBAL_LLM_SERVICE="AzureOpenAI"
AZURE_OPENAI_API_KEY="..."
AZURE_OPENAI_ENDPOINT="https://..."
AZURE_OPENAI_CHAT_DEPLOYMENT_NAME="..."
AZURE_OPENAI_API_VERSION="2023-10-01"
```

## Example: Kernel initialization and invoking a semantic function

This example adapts the notebook into a runnable Python script that shows how to initialize the kernel, add a chat completion service, load a plugin, and run a semantic function.

```python
import os
import asyncio
from pathlib import Path
from dotenv import load_dotenv
from semantic_kernel import Kernel
from semantic_kernel.functions import KernelArguments

async def main():
    load_dotenv()
    kernel = Kernel()

    # Example: register Azure chat completion (the notebook uses conditional logic)
    from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
    kernel.add_service(AzureChatCompletion(service_id="default"))

    # Load a plugin located in the same directory as the notebook
    notebook_dir = Path(__file__).parent
    plugin = kernel.add_plugin(parent_directory=str(notebook_dir), plugin_name="FunPlugin")

    joke_function = plugin["Joke"]

    result = await kernel.invoke(
        joke_function,
        KernelArguments(input="time travel to dinosaur age", style="super silly"),
    )

    text = str(result)
    # Run a simple safety check before printing
    if safety_check(text):
        print(text)

def safety_check(text: str) -> bool:
    # Simple example: block if dangerous keywords appear (customize for your app)
    blocked = ["bomb", "kill", "suicide", "hate"]
    lower = text.lower()
    for kw in blocked:
        if kw in lower:
            raise RuntimeError(f"Blocked content detected: {kw}")
    return True

if __name__ == "__main__":
    asyncio.run(main())
```

## How Guardrails work with Semantic Kernel

- Microsoft Foundry guardrails are configured and enforced at the model deployment level in the Foundry portal. They intercept prompts and completions for a deployment and apply blocking/allowing thresholds for risks (Hate, Violence, Sexual, Self-harm, etc.).
- Semantic Kernel is a client-side SDK that talks to a model service (OpenAI, Azure OpenAI). Guardrails applied in Foundry will affect responses returned to any client that calls the deployed model endpoint.
- You do not programmatically enable Foundry guardrails from the Semantic Kernel client; instead, apply guardrails in Foundry to the model deployment you use with Semantic Kernel.

## Recommended integration pattern

1. Apply Foundry guardrails to your model deployment (via the Foundry portal) to enforce server-side safety.
2. Add lightweight client-side checks in your application (as shown in `safety_check`) to:
   - Reduce latency (catch obviously disallowed inputs before calling the model).
   - Provide custom user-facing messaging or escalation workflows.
   - Maintain a secondary safety net if you need custom business logic beyond Foundry thresholds.
3. Log model interactions and guardrail triggers for auditing and tuning.

## Example: prompt filtering before invoking the kernel

```python
def prompt_allowed(prompt: str) -> bool:
    # Example: block short list of disallowed instructions
    banned_phrases = ["help me kill", "plan a robbery", "manufacture a bomb"]
    p = prompt.lower()
    for phrase in banned_phrases:
        if phrase in p:
            return False
    return True

# usage
if not prompt_allowed(user_input):
    raise ValueError("Prompt violates policy")
else:
    # call kernel.invoke(...)
    pass
```

## Operational notes

- Testing: use representative and adversarial prompts to validate both Foundry guardrail settings and your client-side checks.
- Tuning: track false positives and false negatives in logs; adjust thresholds in Foundry and rules in your app accordingly.
- Human-in-the-loop: escalate content flagged by guardrails to human reviewers for high-risk decisions.

## Links & Resources

- Notebook: [Week5/semantic-kernal/00-getting-started.ipynb](Week5/semantic-kernal/00-getting-started.ipynb)
- Foundry Responsible AI overview: https://learn.microsoft.com/azure/ai-foundry/responsible-use-of-ai-overview
- Foundry guardrails (portal): configure via the Guardrails page in the Foundry portal

---
_Prepared for Week5 — ready to upload to GitHub._
