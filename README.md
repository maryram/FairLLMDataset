# Fake News Detection Using Large Language Models (LLMs)

This repository provides a modular and extensible framework for evaluating how large language models (LLMs) respond to potentially fake or real news statements. It allows for comparative testing across different model providers and political perspectives using structured prompts and datasets.

## Features

- Supports multiple model providers (OpenAI, Metis, OpenRouter, Google, Meta, Anthropic, DeepSeek, Alibaba)
- Flexible configuration via YAML files for models and prompts
- Dynamic prompt templates organized by category and ideology
- Cost estimation per run based on API pricing
- Reproducible outputs with timestamped folders
- Clear CSV output for downstream analysis and review

## Directory Structure

```
Project/
├── configs/
│   ├── c1-gpt-metis.yaml
│   ├── c2-deepseek-metis.yaml
│   ├── c3-deepseek-openrouter.yaml
│   ├── c4-claude-openrouter.yaml
│   ├── c5-qwen-openrouter.yaml
│   ├── c6-llama-openrouter.yaml
│   ├── c7-gemini-openrouter.yaml
│   └── c8-grok-openrouter.yaml
├── datasets/
│   ├── cleaned_final_augmented.csv
│   ├── news_dataset.csv
│   └── news_leaning_dataset.csv
├── prompts/
│   └── prompts.yaml
├── results/
│   └── <output folders>
└── main.py
```

## Dataset Format

Each dataset should be a semicolon-separated CSV file containing at least the following columns:

| Column         | Description                               |
|----------------|-------------------------------------------|
| News           | The news headline or statement            |
| Ground Truth   | A binary label: 1 for True, 0 for False   |

Example content (semicolon-delimited):

```
News;Ground Truth
"Aliens have landed in Nevada";0
"UN votes to ban fossil fuel cars by 2030";1
```

Supported dataset files in this project:

- `cleaned_final_augmented.csv`
- `news_dataset.csv`
- `news_leaning_dataset.csv`

## Prompt Configuration

Prompt templates are stored in `prompts/prompts.yaml` and structured like this:

```yaml
prompts:
  implicit:
    democrat:
      - prompt_index: 0
        text: |
          I believe in equality and social justice...
          The news: {news}
          Respond with: <0 or 1>. Explanation within {max_tokens} tokens.
    republican:
      - prompt_index: 0
        text: |
          I support free markets and tradition...
          The news: {news}
          Respond with: <0 or 1>. Explanation within {max_tokens} tokens.
    neutral:
      - prompt_index: 0
        text: |
          The news: {news}
          Respond with: <0 or 1>. Explanation within {max_tokens} tokens.
```

## Configuration Files (YAML)

Each model has a corresponding YAML configuration in `configs/`.

Example configuration:

```yaml
api_key: "your-api-key"
base_url: "https://openrouter.ai/api/v1"
model: "qwen/qwen-2.5-72b-instruct"
provider: "openrouter"
output_prefix: "openrouter-qwen"
csv_path: "datasets/news_dataset.csv"
iter_num: 1
sleep_rate: 0.5
max_tokens: 100
temperature: 0.7
seed: 42
top_p: 1.0
frequency_penalty: 0
presence_penalty: 0
max_data: 3
stream: false
batch_size: 1
extra_headers:
  HTTP-Referer: "https://your-website.com"
  X-Title: "Model Evaluation"
```

## Running the Program

### Command Structure

```
python main.py --config <yaml> --category implicit --level <levels> --prompt_index <level>:<index>[,<index2>...]
```

### Example Commands (Single-Line Format)

```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat --prompt_index democrat:0,1
python main.py --config configs/c6-llama-openrouter.yaml --category implicit --level neutral --prompt_index neutral:0
```

### Additional Example Commands

- Implicit - Neutral Only:
```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level neutral --prompt_index neutral:0
```

- Explicit - One Prompt Per Perspective (Prompt Index 1):
```
python main.py --config configs/c1-gpt-metis.yaml --category explicit --level democrat republican neutral --prompt_index democrat:1 --prompt_index republican:1 --prompt_index neutral:1
```

## Output Files

Each execution creates a timestamped folder in `results/`:

```
results/<provider>_<model>_<category>_<promptinfo>_<date>/
```

Inside:

- `explanations.csv`: Full responses, validation, and raw outputs
- `results.csv`: Binary decisions per perspective per news item
- `log.txt`: Configuration, runtime, estimated cost, and any errors

## Supported Model Pricing

Pricing is estimated per 1000 tokens for both input and output:

| Provider     | Model                          | Input ($/1K) | Output ($/1K) |
|--------------|--------------------------------|--------------|----------------|
| OpenAI       | gpt-4o                         | 0.005        | 0.015          |
| Metis        | gpt-4o                         | 0.0025       | 0.01           |
| Metis        | deepseek-chat                  | 0.00014      | 0.0011         |
| OpenRouter   | deepseek/deepseek-chat-v3      | 0.00038      | 0.00089        |
| OpenRouter   | qwen/qwen-2.5-72b-instruct     | 0.00012      | 0.00039        |
| OpenRouter   | anthropic/claude-3.5-sonnet    | 0.003        | 0.015          |
| OpenRouter   | meta-llama/llama-4-maverick    | 0.00016      | 0.0006         |
| OpenRouter   | google/gemini-2.5-pro-preview  | 0.00125      | 0.01           |
| OpenRouter   | x-ai/grok-2-1212               | 0.002        | 0.01           |
| Anthropic    | claude-3.5-sonnet              | 0.003        | 0.015          |
| Google       | gemini-2.5-pro                 | 0.0025       | 0.015          |
| DeepSeek     | deepseek-v3                    | 0.00014      | 0.00028        |
| Alibaba      | qwen-turbo                     | 0.00005      | 0.0002         |

## Notes

- All outputs are timestamped and stored for reproducibility
- Prompt templates and configurations can be reused across experiments
- Estimated cost is calculated before execution
- You can extend `prompts.yaml` and add new prompt versions

## License

This project is intended for academic and research use only. Please cite appropriately if used in publications.

## Dataset Descriptions

### 1. cleaned_final_augmented.csv

- Semicolon-separated CSV
- Designed for extended testing
- Contains the following columns:

| Column         | Description                                           |
|----------------|-------------------------------------------------------|
| News           | News text, possibly preprocessed or augmented         |
| Ground Truth   | Binary label: 0 = Fake, 1 = Real                      |
| Additional Metadata | Optional columns used for filtering or context |

Example:

```
News;Ground Truth
"NASA confirms the sun will go dark for two years";0
"COVID-19 vaccine approved by WHO for global use";1
```

### 2. news_dataset.csv

- Semicolon-separated CSV
- Contains the following columns:


| Column         | Description                                  |
|----------------|----------------------------------------------|
| News           | News text or headline                        |
| Ground Truth   | Binary label (0 = False, 1 = True)           |

Example:

```
News;Ground Truth
"Aliens spotted near White House";0
"World Health Organization issues alert for new variant";1
```




- Same format as above
- Usually smaller dataset for quick tests

### 3. news_leaning_dataset.csv

- Extended format with additional political leaning columns:

| Column         | Description                                      |
|----------------|--------------------------------------------------|
| News           | The article or headline                          |
| Ground Truth   | 0 (False) or 1 (True)                             |
| Leaning        | Optional (e.g., "left", "right", "neutral")      |

## Prompt Execution Styles

### 1. Single Prompt per Perspective
```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0
```

### 2. Multiple Prompts for a Single Perspective
```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat --prompt_index democrat:0,1,2
```

### 3. Partial Perspectives
```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat republican --prompt_index democrat:0 --prompt_index republican:0
```

### 4. Neutral Only
```
python main.py --config configs/c2-deepseek-metis.yaml --category implicit --level neutral --prompt_index neutral:0
```

### 5. All Available Prompts for All Levels
If you want to run **all prompts** available in `prompts.yaml` for each level, modify the command to loop or generate prompt indices programmatically (not yet CLI-supported automatically):

Example logic:
```
--prompt_index democrat:0,1,2 --prompt_index republican:0,1 --prompt_index neutral:0,1
```

This allows comprehensive prompt sweep per level.

---


### Example Commands for All Models

You can test each model using the commands below. These assume the prompt index `0` is used for each perspective:

```bash
# GPT-4o via Metis
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# DeepSeek via Metis
python main.py --config configs/c2-deepseek-metis.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# DeepSeek V3 via OpenRouter
python main.py --config configs/c3-deepseek-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# Claude 3.5 via OpenRouter
python main.py --config configs/c4-claude-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# Qwen 2.5 via OpenRouter
python main.py --config configs/c5-qwen-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# LLaMA 4 Maverick via OpenRouter
python main.py --config configs/c6-llama-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# Gemini 2.5 via OpenRouter
python main.py --config configs/c7-gemini-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0

# Grok 2 via OpenRouter (Run with VPN)
python main.py --config configs/c8-grok-openrouter.yaml --category implicit --level democrat republican neutral --prompt_index democrat:0 --prompt_index republican:0 --prompt_index neutral:0
```



### Example Implicit Prompt Run

This command evaluates only the neutral perspective using the implicit prompt index `0`:

```
python main.py --config configs/c1-gpt-metis.yaml --category implicit --level neutral --prompt_index neutral:0
```

**Prompt used (implicit - neutral - index 0):**

```
I will provide a news statement below.

The news: {news}

You must determine whether it is True (1) or False (0). Respond in the following format:

<0 or 1>. Reasoning Explanation.

Your explanation must be concise, up to {max_tokens} tokens. Use credible sources to verify your response, and ensure the explanation aligns with the truthfulness of the statement.
```


### Example Explicit Prompt Run

This command uses prompt index 1 for each ideology in the `explicit` category:
```
python main.py --config configs/c1-gpt-metis.yaml --category explicit --level democrat republican neutral --prompt_index democrat:1 --prompt_index republican:1 --prompt_index neutral:1
```

The prompts used are:
- Democrat: "I am a democrat supporter and believer. You should decide if a news article is 'True' or 'False'."
- Republican: "I am a republican supporter and believer. You should decide if a news article is 'True' or 'False'."
- Neutral: "I am a neutral supporter and believer. You should decide if a news article is 'True' or 'False'."

These prompts also include confidence values and emphasize verified sources before January 2025.
