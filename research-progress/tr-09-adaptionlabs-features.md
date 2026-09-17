# tr-09: adaptionlabs feature deep dive

## feature inventory: adaptive data

adaptive data is adaptionlabs' automated data optimization and expansion pillar. it ingests raw structured or unstructured text, maps prompt and completion roles, executes recipe-based data cleaning and refinement, expands dataset volume via domain or general synthetic generation, and enforces quality and brand controls.

### 1. every datasets.* sdk method

the python sdk (`pip install "adaption>=0.7.0"`) exposes the following methods under `client.datasets`:

- `datasets.create(source={...})`: creates a dataset from a remote source url (hugging face dataset url or kaggle dataset url) or initiates local file upload instructions. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/create
- `datasets.upload_file(file_path)`: high-level sdk helper that creates a dataset record, uploads a local file to storage, completes the upload notification, and returns the dataset object. supported file extensions are `.csv`, `.json`, `.jsonl`, and `.parquet`. source: https://docs.adaptionlabs.ai/adaptive-data-quickstart
- `datasets.get(dataset_id)`: retrieves metadata, row count, schema, mapping configuration, and current status for a dataset. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/get
- `datasets.get_status(dataset_id)`: polls or returns the current processing status (`pending`, `running`, `succeeded`, `failed`, `cancelled`) and row count. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/get_status
- `datasets.list()`: returns a paginated list of all datasets in the user's workspace. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/list
- `datasets.run(dataset_id, column_mapping=..., recipe_specification=..., job_specification=..., brand_controls=..., estimate=false)`: validates configuration, reserves credits, and executes the core adaptive data optimization pipeline over the dataset. setting `estimate=true` prices the request and validates configuration without spending credits or starting execution. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
- `datasets.wait_for_completion(dataset_id, timeout=3600)`: sdk convenience helper that polls dataset status with exponential backoff (2 to 30 seconds) until status reaches `succeeded` or `failed`. source: https://docs.adaptionlabs.ai/adaptive-data/expand-data
- `datasets.download(dataset_id, format="jsonl")`: streams or downloads the processed rows of a dataset in `.csv`, `.jsonl`, or `.parquet` format. works on datasets with status `succeeded` or `ready`. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/download
- `datasets.augment(dataset_id, domain_rows=..., general_rows=..., estimate=false)`: retrieves additional domain-specific and general-purpose rows from curated pools, returning a brand new dataset id containing source plus added rows. source: https://docs.adaptionlabs.ai/adaptive-data/expand-data
- `datasets.translate(dataset_id, sample_rate=..., languages=[...], estimate=false)`: translates a fractional sample (`sample_rate` from 0.01 to 1.0) of source rows across target languages (supporting 242 iso language codes), creating a new expanded dataset. source: https://docs.adaptionlabs.ai/adaptive-data/expand-data
- `datasets.localize(dataset_id, sample_rate=..., pairs=[{"country": "...", "language": "..."}])`: converts source rows into regional/cultural language and country variations using iso 3166-1 alpha-2 country codes and iso 639-1 language codes. source: https://docs.adaptionlabs.ai/adaptive-data/expand-data
- `datasets.invent(dataset_prompt=..., domains=[...], subdomains=[...], rows=..., sample_rate=..., estimate=false)`: generates a synthetic dataset from a natural language task description without requiring an initial seed file ("invent a dataset"). source: https://docs.adaptionlabs.ai/adaptive-data/invent-a-dataset
- `datasets.invent_domains()`: lists supported domain and subdomain categories available for synthetic dataset creation. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/invent_domains
- `datasets.publish(dataset_id, target="huggingface"|"kaggle", target_spec={...})`: publishes the processed dataset directly to hugging face or kaggle repos. currently returns http 501 in active development. source: https://docs.adaptionlabs.ai/api/resources/datasets/methods/publish
- `datasets.delete(dataset_id)`: permanently deletes a dataset and its associated row storage. source: https://docs.adaptionlabs.ai/api/resources/datasets/methods/delete
- `datasets.get_evaluation(dataset_id)`: retrieves quantitative and qualitative quality comparison signals between the source dataset and the adapted output dataset. source: https://docs.adaptionlabs.ai/api/resources/datasets/methods/get_evaluation
- `datasets.upload.initiate`, `datasets.upload.complete`, `datasets.upload.complete_by_id`, `datasets.upload.initiate_batch`, `datasets.upload.complete_batch`: low-level file upload subresource methods used by `upload_file`. source: https://docs.adaptionlabs.ai/api/resources/datasets/subresources/upload/methods/complete_by_id
- `datasets.combine.create(dataset_ids=[...])` and `datasets.combine.validate(...)`: subresource methods that merge multiple completed datasets into a single consolidated dataset for training. source: https://docs.adaptionlabs.ai/tutorials/autoscientist-app-walkthrough

### 2. required file format and schema

adaptive data supports structured files (`.csv`, `.json`, `.jsonl`, `.parquet`) as well as unstructured documents (`.pdf`, `.docx`, `.pptx`, `.xlsx`, `.html`, `.zip`, `.txt`) processed via adaptive data forge. source: https://docs.adaptionlabs.ai/adaptive-data/overview

the column mapping schema (`column_mapping` parameter on `datasets.run`) defines how input file columns map to training roles:

- `prompt` (string, optional/required in prompt mode): column containing input instructions or source text (for gloveterpreter, raw english sentence text).
- `completion` (string, optional): column containing expected output text (for gloveterpreter, target asl or bsl gloss string).
- `context` (list of strings, optional): columns providing background context or retrieved knowledge.
- `universal_prompt` (string, optional): static instruction applied across all rows when a per-row prompt column is absent.
- `chat` (string, optional): column containing full conversational/chat json structures. exclusive with `prompt`, `completion`, and `context`.
- `image` (string, optional): column containing image urls, paths, or bytes for multimodal context. note: mapping an image column disqualifies the dataset from autoscientist fine-tuning. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run

### 3. augmentation parameters and controls

adaptive data provides multiple levers for dataset cleaning, expansion, and alignment:

- `recipe_specification.recipes.deduplication` (boolean): identifies and removes semantic near-duplicate rows from the dataset. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
- `recipe_specification.recipes.prompt_rephrase` (boolean): rephrases prompt text to improve variation, vocabulary diversity, and instructional clarity. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
- `recipe_specification.recipes.reasoning_traces` (boolean): appends explicit chain-of-thought reasoning steps to completion outputs. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
- `brand_controls.length` (literal: `"minimal"`, `"concise"`, `"detailed"`, `"extensive"`): steers completion verbosity. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
- `brand_controls.hallucination_mitigation` (boolean): enforces grounding and factual consistency checking on generated completions. source: https://docs.adaptionlabs.ai/resources/faq
- `brand_controls.blueprint` (string): freeform system prompt specification layer that injects tone, formatting rules, persona, or structural constraints across all completion generation. source: https://docs.adaptionlabs.ai/resources/faq
- `datasets.augment(domain_rows=n, general_rows=m)`: expands dataset size up to the recommended 20,000 domain rows target by sampling domain-specific or general diversity examples. source: https://docs.adaptionlabs.ai/autoscientist/data-augmentation

### 4. step-by-step execution of run()

when `client.datasets.run(dataset_id, ...)` is called, the platform executes a 6-stage pipeline:

1. **import validation**: ingests and validates schema from upload, hugging face, or kaggle. source: https://docs.adaptionlabs.ai/adaptive-data/overview
2. **column mapping validation**: validates that mapped role keys (`prompt`, `completion`, `context`) exist in the dataset schema and meet exclusivity rules. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
3. **recipe execution**: applies toggled processing recipes including near-duplicate removal, prompt rephrasing, and chain-of-thought generation. source: https://docs.adaptionlabs.ai/adaptive-data/overview
4. **expansion and augmentation**: optionally performs row translation, localization, or domain expansion if specified in parameters. source: https://docs.adaptionlabs.ai/adaptive-data/overview
5. **brand control and blueprint enforcement**: applies response length limits, content safety filtering, web grounding, and blueprint system prompts to standardise completion structure. source: https://docs.adaptionlabs.ai/adaptive-data/overview
6. **run completion and evaluation**: reserves necessary account credits, finalizes the adapted dataset, and computes comparative quality evaluation metrics between source and adapted rows. source: https://docs.adaptionlabs.ai/adaptive-data/overview

### 5. credit estimation before spending

to preview credit cost and execution duration without spending account credits, pass `estimate=true` to `datasets.run()`, `datasets.augment()`, `datasets.translate()`, or `datasets.invent()`:

```python
from adaption import Adaption

client = Adaption()

# validate and estimate adaptive data run cost
estimate = client.datasets.run(
    dataset_id="dataset_seed123",
    column_mapping={
        "prompt": "english_text",
        "completion": "gloss_text",
    },
    estimate=True,
)

print(f"estimated credits: {estimate.estimatedCreditsConsumed}")
print(f"estimated duration: {estimate.estimatedMinutes} minutes")
```

source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run

---

## feature inventory: autoscientist

autoscientist is adaptionlabs' automated training and fine-tuning pillar. it co-optimizes training datasets and model training recipes (sft and dpo) over an iterative research loop until quality converges on a target win rate.

### 1. autoscientist.create() parameters

`client.autoscientist.create()` initiates an automated fine-tuning run:

- `dataset_id` (required string): the id of an adapted or raw dataset to train on. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `model` (optional string): base model id returned by `client.autoscientist.list_models()`. if omitted, autoscientist automatically selects an optimal base model for the dataset. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `max_iterations` (optional integer): maximum search loop iterations, from 1 through 5 (or up to 10 depending on plan/model). source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `target_win_rate` (optional float): target win rate ratio between 0.0 and 1.0 (e.g. `0.85`). stops the iterative loop early when reached. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `hyperparams` (optional dict): explicit recipe overrides for hyperparameters (`learning_rate`, `n_epochs`, `batch_size`, `lora_r`, `lora_alpha`, `lora_dropout`, `lora_trainable_modules`, `lr_scheduler_type`, `optimizer`, `warmup_ratio`, `grad_clip`). unset parameters retain platform defaults. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `training_type` (optional string): `"lora"` (default) or `"full"` (where supported by the base model). source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `augmentation_domain_rows` (optional integer): domain-specific synthetic rows added prior to training (targeting recommended 20,000 rows). source: https://docs.adaptionlabs.ai/autoscientist/data-augmentation
- `augmentation_general_rows` (optional integer): general-purpose diversity rows added prior to training. source: https://docs.adaptionlabs.ai/autoscientist/data-augmentation
- `column_mapping` (optional dict): specifies column roles if training directly on raw/non-adapted datasets. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
- `idempotency_key` (optional string): client-generated key preventing duplicate run creation on network retries. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist

### 2. experiment loop and win rate measurement

autoscientist automates the full fine-tuning research loop:

1. **recipe proposal**: autoscientist proposes an initial hyperparameter recipe based on model architecture and dataset characteristics. source: https://docs.adaptionlabs.ai/autoscientist/recommended-hyperparameters
2. **iterative candidate training**: across iterations (1 to `max_iterations`), it trains candidate models, adjusting hyperparameters and data subset selections. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist
3. **win rate measurement**: win rate is measured on held-out evaluation splits from the training dataset using automated llm judges and domain evaluation benchmarks. candidate model completions are evaluated side-by-side against baseline model completions. source: https://docs.adaptionlabs.ai/autoscientist/interpreting-results
4. **early convergence stopping**: if `best_win_rate` meets or exceeds `target_win_rate`, autoscientist halts training early with status `succeeded`. source: https://docs.adaptionlabs.ai/autoscientist/running-autoscientist

### 3. sft, dpo, and lora options

autoscientist supports two training objectives:

- **sft (instruction tuning)**: set via `training_method="instruction"`. requires a minimum of 1,000 dataset rows across all base models. source: https://docs.adaptionlabs.ai/autoscientist/supported-models
- **dpo (alignment / preference training)**: set via `training_method="alignment"`. requires preference pair datasets and a minimum of 12,000 dataset rows. source: https://docs.adaptionlabs.ai/autoscientist/supported-models
- **lora adapter configuration**:
  - `lora_r`: rank parameter from 1 through 64 across all supported base models. source: https://docs.adaptionlabs.ai/autoscientist/supported-models
  - `lora_alpha`: scaling factor, must be exactly 1x or 2x `lora_r`. source: https://docs.adaptionlabs.ai/autoscientist/supported-models
  - `lora_dropout`: dropout probability between 0.0 and 1.0. source: https://docs.adaptionlabs.ai/api/python/resources/autoscientist
  - `lora_trainable_modules`: string specified as `"all-linear"` or explicit comma-separated layer targets (e.g. `"q_proj,v_proj"`). source: https://docs.adaptionlabs.ai/api/python/resources/autoscientist

### 4. supported base models

autoscientist supports sub-10b edge models as well as large frontier models:

- `Qwen/Qwen3.5-0.8B` (0.8B, sub-10B tiny autoscientist target for edge/browser deployment)
- `google/gemma-3-4b-it` (4B)
- `meta-llama/Llama-3.2-3B-Instruct` (3B)
- `mistralai/Mistral-7B-Instruct-v0.2` (7B)
- `google/gemma-4-31B-it` (31B)
- `openai/gpt-oss-20b` (20B)
- `openai/gpt-oss-120b` (120B)
- `meta-llama/Llama-3.3-70B-Instruct-Reference` (70B)
- `meta-llama/Llama-4-Scout-17B-16E-Instruct` (109B)
- `mistralai/Mixtral-8x7B-Instruct-v0.1` (46.7B)
- `nvidia/Nemotron-3-Nano-Omni-30B-A3B-Reasoning-BF16` (30B)
- `Qwen/Qwen3.5-122B-A10B` (122B)
- `Qwen/Qwen3.6-35B-A3B` (35B)

source: https://docs.adaptionlabs.ai/autoscientist/supported-models

### 5. downloadable model artifact formats

model artifacts are streamed using `client.autoscientist.with_streaming_response.download(run_id)`:

- **archive format**: streams a compressed `.tgz` tar archive containing the best iteration checkpoint. source: https://docs.adaptionlabs.ai/autoscientist/download-the-model
- **checkpoint content**: contains standard hugging face peft lora adapter files:
  - `adapter_config.json`
  - `adapter_model.safetensors`
  - `tokenizer.json`
  - `trainer_state.json`

  source: https://docs.adaptionlabs.ai/autoscientist/download-the-model

---

## pricing

adaptionlabs uses a credit-based pricing model for data adaptation and model fine-tuning:

1. **adaptive data pricing**:
   - text dataset adaptation and row generation are billed per output row (e.g. 40 credits per synthetic row batch). source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/augment
   - multimodal image context mapped alongside text columns is billed at 10 credits per 100 output rows. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
2. **credit estimation (`estimate=true`)**:
   - calling `datasets.run(..., estimate=True)`, `datasets.augment(..., estimate=True)`, `datasets.translate(..., estimate=True)`, or `datasets.invent(..., estimate=True)` returns `estimatedCreditsConsumed` (or `estimated_credits`) and `available_credits` without spending credits or starting jobs. source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/run
3. **free tier and competition allowances**:
   - standard accounts receive trial credits upon sign-up. competition participants in autoscientist / uncharted challenges receive 1,000 adaptive data credits plus free compute for autoscientist training. source: https://adaptionlabs.ai/autoscientist-challenge
4. **rate limits and retries**:
   - bursting requests can trigger http 429 rate limit responses. the python sdk automatically retries http 429 and >=500 status codes using exponential backoff. source: https://docs.adaptionlabs.ai/api/python

---

## deployment path

autoscientist produces trained lora weight artifacts that can be deployed across cloud servers or converted for browser edge execution:

### 1. what is downloaded

`client.autoscientist.with_streaming_response.download(run.id)` streams a `.tgz` archive containing:
- `adapter_model.safetensors` (lora adapter weights)
- `adapter_config.json` (peft adapter configuration)
- `tokenizer.json` (tokenization vocabulary and configs)

source: https://docs.adaptionlabs.ai/autoscientist/download-the-model

### 2. serving paths

- **path a: cloud inference server (vllm / hugging face endpoint)**:
  1. extract `best-checkpoint.tgz`.
  2. load base model (`Qwen/Qwen3.5-0.8B`) along with peft lora adapter using hugging face transformers / peft library.
  3. optionally merge lora weights into base weights via `model = model.merge_and_unload()`.
  4. serve via vllm REST endpoint or hugging face inference endpoints for real-time api inference.
- **path b: browser edge inference (gloveterpreter webgpu target)**:
  1. merge lora adapter weights into base `Qwen/Qwen3.5-0.8B` model using peft.
  2. export merged model to onnx format via `optimum-cli export onnx --model merged_qwen_0.8b onnx_qwen/` or convert to mlc-llm / webllm format via `mlc_llm convert_weight`.
  3. quantize weights to 4-bit / q4f16 for lightweight browser downloads (~500mb total weight footprint).
  4. run directly client-side in the gloveterpreter web application using webgpu and onnx runtime web or webllm for 0ms network latency translation.

---

## recommended call sequence for gloveterpreter

below is the exact end-to-end python sdk code sequence for gloveterpreter to process seed data, estimate credits, expand gloss datasets, train a fine-tuned gloss model, and download trained weights:

```python
import os
import time
from adaption import Adaption, DatasetTimeout, TrainingTimeout

# 1. initialize client using ADAPTION_API_KEY env var
client = Adaption()

# 2. upload seed english -> gloss dataset (e.g. seed-asl.jsonl from tr-10)
print("uploading seed dataset...")
dataset = client.datasets.upload_file("seed-asl.jsonl")
dataset_id = dataset.dataset_id
print(f"uploaded dataset id: {dataset_id}")

# 3. preview credit cost with estimate=True before spending
estimate = client.datasets.run(
    dataset_id,
    column_mapping={
        "prompt": "english_text",
        "completion": "asl_gloss",
    },
    recipe_specification={
        "recipes": {
            "deduplication": True,
            "prompt_rephrase": True,
        }
    },
    brand_controls={
        "length": "concise",
        "blueprint": "you are an expert english to sign language gloss translator. output uppercase gloss tokens with proper grammatical ordering.",
    },
    estimate=True,
)
print(f"estimated credits required: {estimate.estimatedCreditsConsumed}")
print(f"estimated execution duration: {estimate.estimatedMinutes} minutes")

# 4. run adaptive data optimization pipeline
run_response = client.datasets.run(
    dataset_id,
    column_mapping={
        "prompt": "english_text",
        "completion": "asl_gloss",
    },
    recipe_specification={
        "recipes": {
            "deduplication": True,
            "prompt_rephrase": True,
        }
    },
    brand_controls={
        "length": "concise",
        "blueprint": "you are an expert english to sign language gloss translator. output uppercase gloss tokens with proper grammatical ordering.",
    },
)

# wait for adaptation run completion
adapted_dataset = client.datasets.wait_for_completion(dataset_id)
print(f"adapted dataset ready. rows: {adapted_dataset.row_count}")

# 5. expand dataset to domain target (e.g. 2,000 domain rows)
augmented = client.datasets.augment(
    dataset_id,
    domain_rows=2000,
    general_rows=500,
)
augmented_dataset = client.datasets.wait_for_completion(augmented.dataset_id)
print(f"augmented dataset ready. total rows: {augmented_dataset.row_count}")

# 6. initiate autoscientist fine-tuning on tiny edge model (Qwen 0.8B)
training_run = client.autoscientist.create(
    dataset_id=augmented_dataset.dataset_id,
    model="Qwen/Qwen3.5-0.8B",
    max_iterations=5,
    target_win_rate=0.85,
    hyperparams={
        "learning_rate": 2e-4,
        "n_epochs": 3,
        "lora_r": 16,
        "lora_alpha": 32,
    },
)
print(f"autoscientist training run launched: {training_run.id}")

# poll autoscientist until completion
completed_run = client.autoscientist.wait_for_completion(training_run.id)
print(f"training completed. status: {completed_run.status}")
print(f"best win rate: {completed_run.best_win_rate}")

# 7. download trained model checkpoint tarball
checkpoint_filename = "qwen_asl_gloss_checkpoint.tgz"
with client.autoscientist.with_streaming_response.download(completed_run.id) as response:
    response.stream_to_file(checkpoint_filename)

print(f"saved model checkpoint to {checkpoint_filename}")
```

### rough cost calculation for gloveterpreter seed v0:
- seed dataset size: 135 rows
- adaptive data run: ~15-20 credits
- synthetic domain expansion to 2,000 rows: ~800 credits
- autoscientist training (5 iterations on sub-10b Qwen 0.8B): free under competition tier or ~200 compute credits
- **total rough cost**: ~1,000 credits (completely covered by competition / free trial credit allowance).

---

## open items

1. **divergence pass on seed dataset**: as noted in status.md, 81 out of 135 seed pairs currently share identical ASL and BSL gloss strings. run a revision pass prior to upload.
2. **webllm / onnx conversion pipeline script**: write a post-processing python script to extract `adapter_model.safetensors` from `checkpoint.tgz`, merge with `Qwen/Qwen3.5-0.8B`, and run `optimum-cli` to produce ONNX WebGPU weights for in-browser deployment.
