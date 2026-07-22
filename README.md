# Self-Refine AI

Self-Refine AI uses Google Gemini to generate an
initial answer, critique it, and iteratively improve it. It also includes tools
for comparing baseline and refined answers with LLM-based judging and standard
SQuAD evaluation metrics.

This project is inspired by the paper
[**Self-Refine: Iterative Refinement with Self-Feedback**](https://arxiv.org/abs/2303.17651).
It implements a simplified version of the paper's central idea: a language
model generates an initial response, provides feedback on that response, and
uses the feedback to produce an improved answer. This repository is an
educational experiment rather than an official implementation of the paper.

## Features

- Generates baseline answers with Gemini
- Produces structured feedback on clarity, completeness, and usefulness
- Improves answers through one or more refinement rounds
- Processes samples from the Databricks Dolly 15K dataset
- Saves results in JSON Lines format for easy analysis
- Filters answers that changed during refinement
- Compares candidate answers using Gemini as an evaluator
- Scores answers from 1 to 10 with response validation
- Computes exact-match and F1 scores using the SQuAD metric
- Provides reusable, documented Python functions and a command-line interface

## How It Works

```text
Instruction and context
        |
        v
Generate baseline answer
        |
        v
Critique the current answer
        |
        v
Refine using the feedback
        |
        v
Repeat for the requested number of rounds
```

The original baseline is retained so it can be compared with the final refined
answer.

## Requirements

- Python 3.10 or later
- A Google Gemini API key

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

## Configuration

Set your Gemini API key as an environment variable.

macOS or Linux:

```bash
export GEMINI_API_KEY="your-api-key"
```

Windows PowerShell:

```powershell
$env:GEMINI_API_KEY = "your-api-key"
```

Do not commit API keys or local environment files to version control.

## Usage

### Generate and refine answers

Run the default experiment against the selected Databricks Dolly 15K samples:

```bash
python self_refine_ai.py generate
```

By default, the command creates:

- `results.jsonl`, containing every generated result
- `changed_results.jsonl`, containing only results where refinement changed the
  baseline answer

Customize the number of refinement rounds and output paths:

```bash
python self_refine_ai.py generate \
  --rounds 2 \
  --output results.jsonl \
  --changed-output changed_results.jsonl
```

Each additional round requires two more model calls per sample: one critique
and one refinement.

### Evaluate saved answers

Evaluate HotpotQA-style results with SQuAD exact-match and F1 metrics:

```bash
python self_refine_ai.py evaluate hotpot_results.jsonl
```

Evaluation records must contain these fields:

```json
{
  "question": "Example question",
  "ground_truth": "Expected answer",
  "baseline": "Initial model answer",
  "refined": "Refined model answer"
}
```

## Generated Result Format

The `generate` command writes one JSON object per line:

```json
{
  "question": "The dataset instruction",
  "context": "Optional supporting context",
  "answer": "The dataset reference answer",
  "original answer": "The Gemini baseline answer",
  "refined_answer": "The final refined answer"
}
```

JSON Lines is used so results can be processed incrementally and loaded easily
with Python, pandas, or command-line data tools.

## Use as a Python Module

The refinement service can also be used directly:

```python
from self_refine_ai import GeminiAnswerService

service = GeminiAnswerService()
baseline, refined = service.self_refine(
    instruction="Explain why the sky appears blue.",
    context="",
    rounds=1,
)

print("Baseline:", baseline)
print("Refined:", refined)
```

## Project Structure

```text
.
├── README.md
├── requirements.txt
└── self_refine_ai.py
```

## Notes

- Model calls may incur API usage costs.
- Results can vary between runs because generative models are nondeterministic.
- LLM-based evaluation is useful for experimentation but should not be treated
  as an objective replacement for human review.
- Dataset downloads and SQuAD metric loading require an internet connection.

## License

No license has been selected yet. Add a `LICENSE` file before distributing or
accepting external contributions.
