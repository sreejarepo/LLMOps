# LLMOps Assignment 2: API Cost, Latency and Prompt Version Management

This is my submission for the EduAssist AI scenario. I measured latency and token cost for five academic questions, then
registered three prompt versions in MLflow, compared them, and promoted the best one to the `production` alias.

**Report:** [report.pdf](report.pdf) (500 to 700 words)  |  **Notebook:** [llmops_assignment.ipynb](llmops_assignment.ipynb)
([PDF copy](llmops_assignment.pdf))

## Important: why I used a local model
My OpenAI calls failed with an `insufficient_quota` error. Section 7 of the brief allows a locally hosted open-source model when
API access is unavailable, so I used:

- Model: `llama3.1:8b`, served by Ollama on a Tesla T4 GPU
- API: Ollama's OpenAI-compatible chat completions endpoint, called with the `openai` Python library (temperature 0)
- Cost: Ollama is free, so I estimated cost from the published price of the same model: DeepInfra (`deepinfra/fp8`) on OpenRouter,
  listed price, checked on 2026-11-04, in USD: 0.02 per million input tokens and 0.04 per million output tokens
  ([pricing screenshot](screenshots/00_pricing_evidence.png))

## Results at a glance
| Prompt | Avg latency (s) | Avg input tokens | Avg output tokens | Avg cost (USD) | Valid JSON | Recommended |
|---|---|---|---|---|---|---|
| v1 | 16.91 | 30.6 | 500.4 | 0.00002063 | 0% | No |
| v2 | 13.42 | 52.6 | 358.6 | 0.00001540 | 0% | No |
| v3 | 4.48 | 86.6 | 115.2 | 0.00000634 | 100% | **Yes (production)** |

Task 1 (prompt v2, five questions): mean latency 10.52 s, median 10.63 s, min 7.72 s, max 13.26 s, standard deviation 2.45 s.
Task 2: average cost 0.0000154 USD per request, about 0.15 USD for 10,000 requests a month.

## Evidence (click to open)
Screenshots:
- [Screenshot 1: experiment runs with metrics](screenshots/01_experiment_runs_metrics.png)
- [Screenshot 2: metrics compared across runs](screenshots/02_metrics_compare.png)
- [Screenshot 3: Prompt Registry, v1 to v3](screenshots/03_prompt_registry.png)
- [Screenshot 4a: prompt diff, v1 vs v2](screenshots/04_prompt_diff_v1_v2.png)
- [Screenshot 4b: prompt diff, v2 vs v3](screenshots/04_prompt_diff_v2_v3.png)
- [Screenshot 5: production alias on v3](screenshots/05_production_alias.png)
- [Pricing evidence (DeepInfra on OpenRouter)](screenshots/00_pricing_evidence.png)

Files:
- [Notebook](llmops_assignment.ipynb)
- [Notebook as PDF](llmops_assignment.pdf)
- [latency_tokens_cost.csv](latency_tokens_cost.csv)
- [prompt_evaluation_results.csv](prompt_evaluation_results.csv)
- [promotion_justification.txt](promotion_justification.txt)
- [README](README.md)
- [requirements.txt](requirements.txt)
- [mlflow.db](mlflow.db) and [mlruns/](mlruns) (stored MLflow results)

## Environment
- Python: 3.13.16
- Libraries (same as [requirements.txt](requirements.txt)): mlflow==3.18.0, openai==2.54.0, pandas==2.2.3, reportlab==5.1.0
- Ollama, installed with `curl -fsSL https://ollama.com/install.sh | sh`
- Environment variables: none are needed for the local model. If you switch back to OpenAI, set `OPENAI_API_KEY` in the shell
  before starting Jupyter. I never put a key in the notebook.

## Install
```
pip install -r requirements.txt
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
ollama pull llama3.1:8b
```

## How to reproduce the experiment
1. Open `llmops_assignment.ipynb` on a machine with a GPU (I used a Tesla T4) where Ollama can run.
2. In the configuration cell, fill `PRICING_SOURCE`, `PRICING_DATE` and both prices, then set `PRICES_VERIFIED = True`.
3. Run all cells from top to bottom. The notebook asks the model five questions with prompt v2 (Tasks 1 and 2), then five
   questions with each of v1, v2 and v3 (Task 3), logs one MLflow run per version, and sets the `production` alias on the best version.
4. The notebook writes `latency_tokens_cost.csv`, `prompt_evaluation_results.csv`, `promotion_justification.txt`, `report.pdf`
   and `mlflow.db`.

Please do not run all cells a second time against the same `mlflow.db`. It would register prompt versions 4 to 6 and change the numbers.

## View the MLflow results
The results are already stored in `mlflow.db`, so there is no need to rerun the notebook. In the folder that contains `mlflow.db`, run:
```
mlflow server --host 127.0.0.1 --port 5000 --workers 1 --backend-store-uri sqlite:///mlflow.db
```
Then open http://127.0.0.1:5000. Runs and metrics are under the experiment `eduassist_llmops_assignment` (Model training view).
The prompts are under Prompts in the GenAI view, in `eduassist_qa_prompt`.

## Repository layout
```
llmops_assignment.ipynb / .pdf   notebook and its PDF copy
report.pdf, README.md            report and this file
requirements.txt
latency_tokens_cost.csv          Tasks 1 and 2 results
prompt_evaluation_results.csv    Task 3 results, including every response
promotion_justification.txt      why v3 was promoted
mlflow.db, mlruns/               MLflow tracking data
screenshots/                     pricing evidence and the five MLflow screenshots
```
