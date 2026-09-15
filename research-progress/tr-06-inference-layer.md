# tr-06: inference layer

## open questions
1. what exactly do adaptionlabs adaptive data and autoscientist support, and can they produce trained models for edge or web deployment?
2. which small instruct model family (1b to 7b parameter class) is best suited for fine-tuning on english to asl and bsl gloss translation?
3. can a 1b to 3b parameter model realistically execute in-browser via webgpu or transformers.js or webllm, or is a hosted inference api required for the hackathon demo?
4. what is the latency budget for interpretation given speech pause detection boundaries and human sign language interpreter ear-voice span (evs)?
5. how can we strictly enforce vocabulary-constrained decoding so the model only outputs valid sign gloss tokens?

## findings

### 1. adaptionlabs capabilities, sdk, and boundaries
- adaptive data analyzes source text structure, ingests seed datasets, deduplicates examples, optimizes prompt and completion pairs, and performs synthetic data expansion up to 20,000 domain rows (source: https://docs.adaptionlabs.ai/guides/autoscientist). it also supports dataset translation and expansion across iso language codes (source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/translate).
- autoscientist automates hyperparameter optimization and training recipe search (sft and dpo) over an iterative loop of up to 10 iterations targeting a specified win rate, yielding up to 35% relative performance improvements over human-configured recipes (source: https://adaptionlabs.ai/blog/autoscientist).
- base models offered by autoscientist include `qwen/qwen3.5-0.8b` (0.8b), `google/gemma-3-4b-it` (4b), `google/gemma-3-27b-it` (27b), `google/gemma-4-26b-a4b-it` (26b), `google/gemma-4-31b-it` (31b), `openai/gpt-oss-20b` (20b), `openai/gpt-oss-120b` (120b), `meta-llama/llama-3.3-70b-instruct-reference` (70b), `meta-llama/llama-4-scout-17b-16e-instruct` (17b), `mistralai/mixtral-8x7b-instruct-v0.1` (8x7b), `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning-bf16` (30b), and `nvidia/nvidia-nemotron-3-super-120b-a12b-bf16` (120b) (source: https://docs.adaptionlabs.ai/autoscientist/supported-models).
- tiny autoscientist restricts training selection to sub-10b models (such as `qwen/qwen3.5-0.8b` or `google/gemma-3-4b-it`) specifically for edge and low-latency deployments (source: https://docs.adaptionlabs.ai/tutorials/autoscientist-app-walkthrough).
- python sdk interface uses `pip install adaption`, `client.datasets.upload_file()`, `client.datasets.run()`, `client.autoscientist.create(dataset_id=..., model=...)`, and `client.autoscientist.download(experiment_id)` (source: https://docs.adaptionlabs.ai/api/python/resources/autoscientist).
- pricing model is credit-based: credit costs are charged based on newly added synthetic rows and training iterations, with live credit estimation provided via the sdk (source: https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/translate).
- what adaptionlabs can do: generate synthetic english to gloss pairs from seed sets, auto-tune lora and sft hyperparameters, train sub-10b models, and export trained weight artifacts via `client.autoscientist.download()` (source: https://docs.adaptionlabs.ai/api/python/resources/autoscientist).
- what adaptionlabs cannot do: it does not serve real-time api endpoints for inference, run browser-side engines, or enforce runtime vocabulary decoding constraints: weight artifacts must be downloaded and hosted or converted separately (source: https://docs.adaptionlabs.ai/autoscientist/overview).

### 2. small instruct models for constrained gloss generation
- small models (1b to 3b parameter range) fine-tuned on paired translation data consistently outperform large general-purpose llms (gpt-4, claude) on domain-specific sign gloss grammar, topic-comment order, and spatial or temporal markers (source: https://qwenlm.github.io/blog/qwen2.5).
- qwen 2.5 instruct series (0.5b, 1.5b, 3b) demonstrates exceptional structured output generation, table understanding, and strict instruction following (source: https://qwenlm.github.io/blog/qwen2.5). qwen 2.5 1.5b and 3b are open-weight models available under apache 2.0 (0.5b and 1.5b) or qwen license (3b) (source: https://ollama.com/library/qwen2.5:1.5b).
- ibm granite 3.1 instruct series offers dense models (2b, 8b) and moe models (1b-a400m, 3b-a800m with 800m active parameters) with 128k context windows, exhibiting strong tool-use, structured chat, and low compute footprints suitable for edge environments (source: https://huggingface.co/ibm-granite/granite-3.1-3b-a800m-instruct, https://www.rivista.ai/wp-content/uploads/2024/10/paper-1.pdf).
- llama 3.2 (1b and 3b instruct) provides compact instruction-tuned models optimized for edge devices and native compatibility with browser webgpu runtimes (source: https://www.buildmvpfast.com/blog/webgpu-browser-ai-inference-cost-savings-2026).

### 3. in-browser inference reality check
- running 1b to 3b models in browser tabs via webgpu (using transformers.js v3 with onnx runtime web or webllm with mlc-llm) is technically feasible in 2026 (source: https://arxiv.org/html/2605.20706v1, https://maddevs.io/writeups/running-ai-models-locally-in-the-browser).
- download sizes: int4 quantized 1b models require ~500mb to 800mb download, while 3b int4 models require ~1.5gb to 2.0gb (source: https://maddevs.io/writeups/running-ai-models-locally-in-the-browser, https://www.buildmvpfast.com/blog/webgpu-browser-ai-inference-cost-savings-2026).
- load time and startup: cold downloads over typical broadband take 10 to 30 seconds. once cached in indexeddb or cachestorage, wllama initialises in ~800ms while transformers.js initialises in ~2.5s (source: https://mikeesto.com/posts/wllama). webgpu shader compilation introduces a cold-start delay of 1 to 2 seconds unless pre-warmed with dummy inference (source: https://www.sitepoint.com/webgpu-vs-webasm-transformers-js).
- inference speed: 1b to 1.5b int4 models achieve 25 to 60+ tokens/second on webgpu (source: https://mikeesto.com/posts/wllama, https://www.sitepoint.com/webgpu-vs-webasm-transformers-js). generating a short gloss sequence (5 to 10 tokens) takes under 150 to 250ms of generation time (source: https://mikeesto.com/posts/wllama).
- client-side risks: high memory usage (~1.2gb to 2.5gb ram/vram) causes out-of-memory tab crashes on mobile devices or lower-end laptops, and cold asset downloads create heavy initial friction for demo users (source: https://maddevs.io/writeups/running-ai-models-locally-in-the-browser).
- practical demo conclusion: hosting the fine-tuned model on a dedicated inference api (vllm, hugging face inference endpoints, or modal) provides instant load, low latency (<200ms), and 100% cross-device reliability for the hackathon demo, while webgpu local execution serves as an optional stretch goal (source: https://arxiv.org/html/2605.20706v1).

### 4. latency budget and human interpreter lag
- human sign language simultaneous interpreters operate with a strategic time delay known as ear-voice span (evs) or ear-hand span, typically ranging between 2.0 and 5.0 seconds (average 2.5 to 3.5s) (source: https://www.linkedin.com/posts/ilhem-bezzaoucha-a4227312b_in-simultaneous-interpreting-the-terms-d%C3%A9calage-activity-7384209991928549376-Y5mS, https://repository.ubn.ru.nl/bitstream/handle/2066/227577/227577.pdf?sequence=1&isAllowed=y). audience comprehension remains natural as long as total lag stays under 5 seconds (source: https://www.linkedin.com/posts/ilhem-bezzaoucha-a4227312b_in-simultaneous-interpreting-the-terms-d%C3%A9calage-activity-7384209991928549376-Y5mS).
- real-time speech capture apis (speechmatics, assemblyai) fire utterance boundary events within 300ms to 700ms of speaker silence or prosodic pause (source: https://docs.adaptionlabs.ai).
- inference latency budget: given a ~500ms speech boundary detection latency and ~50ms 3d keyframe preparation time, the english to gloss inference layer has a target latency budget of under 300ms (500ms maximum) to keep total end-to-end delay under 1.2 to 1.5 seconds, well within human interpreter evs standards (source: https://www.linkedin.com/posts/ilhem-bezzaoucha-a4227312b_in-simultaneous-interpreting-the-terms-d%C3%A9calage-activity-7384209991928549376-Y5mS).

### 5. structured output and vocabulary-constrained decoding
- gloss generation requires output restricted strictly to valid gloss vocabulary tokens (uppercase gloss words, structural markers, spatial locations) (source: https://www.kunwar.page/chapter/043-structured-generation-guided-decoding-json-mode-regex-constraints-fsm-masking).
- server-side constrained decoding: serving engines like vllm and sglang utilize grammar backends like xgrammar or outlines to perform finite state machine (fsm) logit masking, ensuring 100% compliance with regular expressions or json schemas (source: https://www.kunwar.page/chapter/043-structured-generation-guided-decoding-json-mode-regex-constraints-fsm-masking, https://blog.squeezebits.com/guided-decoding-performance-vllm-sglang).
- regex constraint formulation: a regex pattern such as `^([A-Z0-9_-]+)( [A-Z0-9_-]+)*$` or explicit vocabulary enum regex `^(BOOK|BUY|YESTERDAY|ME|STORE|GO)( (BOOK|BUY|YESTERDAY|ME|STORE|GO))*$` forces the sampler to mask out non-gloss tokens (source: https://www.kunwar.page/chapter/043-structured-generation-guided-decoding-json-mode-regex-constraints-fsm-masking).
- client-side constrained decoding: in transformers.js or onnx runtime web, a custom logits processor sets the logits of out-of-vocabulary tokens to negative infinity at each decoding step (source: https://www.kunwar.page/chapter/043-structured-generation-guided-decoding-json-mode-regex-constraints-fsm-masking).

## recommended setup

- **model family**: qwen 2.5 1.5b instruct (or granite 3.1 2b instruct) fine-tuned via adaptionlabs autoscientist on augmented english to gloss parallel data.
- **serving route**:
  - **primary (hackathon build)**: hosted serverless api via vllm or hugging face inference endpoints running on a low-cost gpu (e.g. t4 or a10g). this setup provides <150ms inference latency, zero initial client asset download, 100% cross-device compatibility, and native xgrammar or outlines guided decoding.
  - **fallback / edge (experimental local route)**: transformers.js v3 with webgpu using int4 onnx quantized weights (~550mb download), utilizing a custom client-side logits processor for vocabulary masking.
- **expected latency budget**:
  - speech boundary detection: ~500ms
  - hosted gloss inference: ~100-150ms
  - joint angle lookup and keyframe interpolation: ~50ms
  - total end-to-end delay: ~650-700ms (substantially faster than natural human sign interpreter lag of 2.0 to 5.0 seconds).
- **fallback plan**:
  - if fine-tuning or weight export encounters issues during hackathon build, fall back to a hosted fast general-purpose llm (e.g. groq with llama 3.3 70b or gpt-4o-mini) using strict json regex schema constrained output.

## open items
1. benchmark fine-tuned qwen 2.5 1.5b versus granite 3.1 2b once seed dataset is prepared and adaptionlabs training runs complete.
2. verify xgrammar regex guided decoding parameter performance on the chosen vllm serving stack.
3. prototype client-side onnx conversion and quantization of fine-tuned weights using optimum for optional offline webgpu execution.
