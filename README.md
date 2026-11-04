# LLMOps Assignment 2: API Cost, Latency and Prompt Version Management

Scenario: EduAssist AI academic Q&A assistant. Model: `llama3.1:8b` served by Ollama on Google Colab (Tesla T4).
OpenAI access was unavailable (insufficient_quota), so a locally hosted open-source model is used as allowed in section 7 of the brief.
Cost is estimated from DeepInfra (deepinfra/fp8) via OpenRouter, meta-llama/llama-3.1-8b-instruct, listed price, https://openrouter.ai/meta-llama/llama-3.1-8b-instruct (checked 2026-11-04, currency USD).

## Environment
- Python: 3.13.16
- Libraries (also in requirements.txt): mlflow==3.18.0, openai==2.54.0, pandas==2.2.3, reportlab==5.1.0
- Ollama (installed with `curl -fsSL https://ollama.com/install.sh | sh`), model `llama3.1:8b`
- Environment variables: none are required for the local model. If you switch back to OpenAI, set `OPENAI_API_KEY` in the
  shell that starts Jupyter and never put the key in the notebook.

## Install
```
pip install -r requirements.txt
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
ollama pull llama3.1:8b
```

## Reproduce the experiment
1. Open the notebook (Google Colab with a T4 GPU is recommended; the same code runs locally if Ollama is running).
2. Fill PRICING_SOURCE, PRICING_DATE and both prices in the configuration cell, then set PRICES_VERIFIED = True.
3. Run all cells top to bottom. This calls the model for 5 questions with v2 (Tasks 1-2) and for 5 questions with each of v1, v2, v3 (Task 3),
   logs one MLflow run per version, and sets the `production` alias.
4. Outputs: `latency_tokens_cost.csv`, `prompt_evaluation_results.csv`, `promotion_justification.txt`, `report.pdf`, `mlflow.db`.

## MLflow UI
Run this in the folder that contains `mlflow.db`, then open http://127.0.0.1:5000
```
mlflow server --host 127.0.0.1 --port 5000 --workers 1 --backend-store-uri sqlite:///mlflow.db
```
Screenshots required: experiment runs with metrics, Prompt Registry (v1-v3), prompt diff/compare view, `production` alias.

## Files
notebook (.ipynb) and its PDF/HTML export, requirements.txt, latency_tokens_cost.csv, prompt_evaluation_results.csv,
report.pdf, MLflow screenshots, this README.
