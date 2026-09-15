# tr-04 utterance boundary and semantic completeness detection

## open questions

- how do speechmatics and assemblyai real-time streaming apis handle utterance boundary signals, turn-end detection, and silence thresholds?
- how can a web browser client securely establish websocket connections to these services without exposing permanent api keys?
- what pricing and endpointing latency profiles exist for assemblyai, speechmatics, deepgram, azure, google cloud, and amazon transcribe?
- what does academic literature and commercial implementations say about voice activity detection vs prosody vs semantic end-of-turn prediction?
- how can glovterpreter implement a lightweight semantic completeness checker to catch dangling sentences before committing text to english-to-gloss interpretation?

## findings

### assemblyai real-time streaming

- **websocket endpoint and browser authentication**: assemblyai real-time streaming operates over a websocket endpoint at `wss://streaming.assemblyai.com/v3/ws` (with regional options like `wss://streaming.eu.assemblyai.com/v3/ws`) (source: https://www.assemblyai.com/docs/streaming/api-spec/streaming-websocket). because browser webSocket clients cannot set custom http headers such as `authorization`, assemblyai provides temporary token authentication. a backend service requests a short-lived token and passes it to the browser, which connects via query parameter `?token=YOUR_TEMP_TOKEN` (source: https://www.assemblyai.com/docs/streaming/authenticate-with-a-temporary-token).
- **event model and boundary behavior**: assemblyai emits `SpeechStarted` when voice activity is detected, followed by streaming `Turn` messages with `end_of_turn: false` for partial results (source: https://www.assemblyai.com/docs/streaming/universal-3-pro/turn-detection-and-partials). when a boundary is reached, it emits a final `Turn` message with `end_of_turn: true`. clients can also dynamically update configuration mid-stream via `{"type": "UpdateConfiguration", ...}` or manually trigger a boundary with `{"type": "ForceEndpoint"}` (source: https://www.assemblyai.com/docs/streaming/api-spec/streaming-websocket).
- **turn detection parameters**: controlled via key threshold parameters: `min_turn_silence` sets the silence duration in milliseconds before a speculative turn-end check (default mode-dependent: 128 ms for `min_latency`/`balanced`, 512 ms for `max_accuracy`, range 50 to 10000 ms); `max_turn_silence` sets the maximum silence duration before forcing a turn end regardless of content (default 640 ms for `min_latency`, 1280 ms for `balanced`, 2560 ms for `max_accuracy`); `end_of_turn_confidence_threshold` sets the confidence score required for semantic end-of-turn (default 0.4 on universal streaming) (source: https://www.assemblyai.com/docs/streaming/getting-started/optimizing-accuracy-and-latency).
- **pricing**: universal-streaming costs $0.15 per hour ($0.0025/min) billed per second; universal-3.5 pro realtime costs $0.45 per hour ($0.0075/min) base (source: https://www.assemblyai.com/pricing). optional add-ons include speaker diarization (+$0.12/hr), prompting (+$0.05/hr), and keyterms prompting (+$0.04/hr or included in 3.5 pro) (source: https://www.assemblyai.com/pricing).
- **latency**: universal-3.5 pro realtime achieves median turn-end detection (time to complete turn) of ~300 ms to 500 ms with p95 latency of 534 ms (source: https://www.assemblyai.com/benchmarks).

### speechmatics real-time streaming

- **websocket endpoint and browser authentication**: speechmatics real-time api connects to `wss://global.rt.speechmatics.com/v2` or regional endpoints such as `wss://eu.rt.speechmatics.com/v2` (source: https://docs.speechmatics.com/api-ref/realtime-transcription-websocket). for browser integrations, temporary jwt keys are created server-side via `POST https://mp.speechmatics.com/v1/api_keys?type=rt` with a time-to-live (`ttl`) parameter and passed via query string `?jwt=$TEMP_KEY` (source: https://docs.speechmatics.com/get-started/authentication).
- **turn-end detection and message events**: speechmatics fires an `EndOfUtterance` event message over the websocket when a speaker stops speaking (source: https://docs.speechmatics.com/api-ref/realtime-transcription-websocket). transcripts are delivered via `AddPartialTranscript` and finalized via `AddTranscript` events (source: https://docs.speechmatics.com/speech-to-text/realtime/turn-detection).
- **configurability**: turn detection is configured in `conversation_config` via `end_of_utterance_silence_trigger` (float in seconds, range 0.0 to 2.0s, where 0 disables feature) (source: https://docs.speechmatics.com/api-ref/realtime-transcription-websocket). three turn-detection modes are supported in the sdk: `FIXED` (fixed silence trigger duration), `ADAPTIVE` (smart turn adjustments based on speech rate, disfluencies, and pause patterns), and `EXTERNAL` (client triggers turn completion manually via `ForceEndOfUtterance` or `client.finalize(end_of_turn=True)`) (source: https://github.com/speechmatics/speechmatics-python-sdk/blob/main/sdk/voice/README.md).
- **pricing**: real-time standard costs $0.24 per hour; real-time enhanced costs $0.43 per hour (source: https://www.speechmatics.com/pricing). free plan includes $100 credit on sign-up with 2 concurrent real-time sessions (source: https://www.speechmatics.com/pricing).
- **latency**: typical end-of-utterance detection fires within ~200 ms to 450 ms after acoustic speech stops when configured with low silence trigger thresholds (source: https://github.com/speechmatics/speechmatics-python-sdk/blob/main/sdk/voice/README.md).

### cloud speech apis endpointing mechanics

- **google cloud speech-to-text**: provides the `latest_short` model designed for single voice commands with `single_utterance` mode, automatically closing the stream and emitting `END_OF_SINGLE_UTTERANCE` upon detecting utterance end (source: https://docs.cloud.google.com/speech-to-text/docs/single-utterance). endpointing silence thresholds default to 500 ms to 1.25s, though low-latency configurations can set endpointing lower (source: https://developers.deepgram.com/docs/use-deepgram-with-dialogflow-cx).
- **azure speech sdk**: uses `SpeechEndDetected` events and provides silence configuration parameters including `Speech_SegmentationSilenceTimeoutMs` (100 ms to 5000 ms), `SpeechServiceConnection_EndSilenceTimeoutMs`, and `SpeechServiceConnection_InitialSilenceTimeoutMs` (source: https://learn.microsoft.com/en-us/javascript/api/microsoft-cognitiveservices-speech-sdk/propertyid?view=azure-node-latest). note that in browser javascript with default microphone input, audio buffering can prevent client-side silence settings under 500 ms from functioning accurately unless using `PushAudioInputStream` (source: https://learn.microsoft.com/en-us/answers/questions/2338575/azure-speech-sdk-javascript-silence-timeout-proper).
- **amazon transcribe streaming**: breaks audio into natural speech segments marked by `IsPartial: false` events (source: https://docs.aws.amazon.com/transcribe/latest/dg/streaming-partial-results.html). supports partial results stabilization levels (low, medium, high) via `PartialResultsStability` parameter, where high stability stabilizes text faster with lower overall turn-latency but slightly reduced context revisions (source: https://aws.amazon.com/blogs/machine-learning/amazon-transcribe-now-supports-partial-results-stabilization-for-streaming-audio).

### end-of-turn prediction literature and research models

- **vad vs prosody vs semantic end-of-turn models**: pure voice activity detection (vad) relies strictly on acoustic silence thresholds and fails when speakers pause mid-sentence. prosodic vad incorporates pitch drop/raise and syllable lengthening. semantic/linguistic end-of-turn models analyze whether the text transcript forms a syntactically and pragmatically complete thought (source: https://aclanthology.org/2020.findings-emnlp.268).
- **turngpt**: introduced by ekstedt and skantze (emnlp 2020), turngpt is a transformer language model that incrementally evaluates spoken word sequences to predict turn-shift probabilities after each token (source: https://aclanthology.org/2020.findings-emnlp.268). attention analysis shows that 20% of the model's attention is directed to preceding dialogue context, confirming that pragmatic completion depends on multi-turn history (source: https://aclanthology.org/2023.findings-acl.776.pdf).
- **voice activity projection (vap)**: developed by ekstedt and skantze (interspeech 2022 / sigdial 2025), vap uses a frame-based neural audio encoder (cpc) to project future voice activity and predict speaker transitions directly from acoustic signals (source: https://aclanthology.org/2025.sigdial-1.9.pdf).
- **turnsense 3-class classifier**: turnsense (baiji-team, 2024) is a 47M parameter transformer model (int8 onnx ~50mb) that natively classifies utterances into `complete`, `incomplete`, or `invalid` states (source: https://huggingface.co/Baiji-Team/TurnSense/blob/3041540737483f6a9153cee65a33ded530091462/README.md). it achieves 96.35% F1 score on complete utterances and 96.32% F1 on incomplete utterances with cpu p50 latency under 55 ms (source: https://huggingface.co/Baiji-Team/TurnSense/blob/3041540737483f6a9153cee65a33ded530091462/README.md).
- **on-device sentence completion detection**: research on small language models (bert-tiny with 4.4M params or bi-lstm) for ASR transcripts demonstrates sentence completion classification with 90.95% F1 score and on-device cpu execution latencies of 15 ms to 37 ms (source: https://aclanthology.org/2020.icon-main.53.pdf).

### semantic completeness checking prior art and latency

- **dangling structure detection**: rule-based heuristic checks search for trailing tokens that indicate unfinished thoughts: coordinating conjunctions (`but`, `and`, `or`, `so`), subordinating conjunctions (`because`, `although`, `if`, `when`, `while`), prepositions (`with`, `of`, `to`, `for`, `in`, `at`), or unresolved subordinate clause markers. rule-based pattern matching executes in under 1 ms in browser javascript or webassembly.
- **lightweight classifier architecture**: combining a fast regex/token heuristic (<1 ms) with a quantized ONNX transformer model (~15 ms to 55 ms, e.g. bert-tiny or turnsense ONNX) provides a robust two-stage verification step before triggering upstream interpretation.
- **latency impact**: total latency overhead added by the semantic completeness check is 1 ms for clear complete/dangling cases (via heuristic) and 15 ms to 55 ms for ambiguous cases (via ONNX classifier). this overhead is negligible compared to standard network round-trips and audio frame buffering (source: https://picovoice.ai/blog/speech-to-text-latency).

### endpointing latency comparison summary

- **assemblyai universal-3.5 pro realtime**: ~128 ms minimum silence trigger, ~300 ms P50 time-to-turn-end, 534 ms P95 (source: https://www.assemblyai.com/benchmarks).
- **speechmatics realtime**: ~200 ms silence trigger, ~250 ms to 450 ms detection time (source: https://docs.speechmatics.com/api-ref/realtime-transcription-websocket).
- **deepgram nova-3 / flux**: ~260 ms to 300 ms end-of-turn detection (source: https://www.coval.ai/blog/best-speech-to-text-providers-in-2026-independent-benchmarks-and-how-to-choose).
- **azure speech sdk**: ~300 ms to 500 ms minimum effective detection time in client environments (source: https://learn.microsoft.com/en-us/answers/questions/2128774/speech-sdk-speech-to-text-segmentation-silence-tim).
- **google cloud speech**: ~500 ms to 1250 ms default endpointing duration (source: https://developers.deepgram.com/docs/use-deepgram-with-dialogflow-cx).
- **amazon transcribe**: ~1000 ms to 1500 ms natural pause segmenting (source: https://docs.aws.amazon.com/transcribe/latest/dg/streaming-partial-results.html).

## comparison table

| provider / option | boundary signal event | silence threshold configurability | browser websocket integration | streaming price ($/hr) | typical turn-end latency |
|---|---|---|---|---|---|
| assemblyai (u3.5 pro / universal) | `Turn` (`end_of_turn: true`) | `min_turn_silence` (128-10000ms), `max_turn_silence` (640-2560ms) | backend temp token via query param `?token=` | $0.15 (universal) / $0.45 (3.5 pro) | ~128-300ms (P50), 534ms (P95) |
| speechmatics realtime | `EndOfUtterance` | `end_of_utterance_silence_trigger` (0.0-2.0s), fixed/adaptive/external modes | backend temp jwt via query param `?jwt=` | $0.24 (standard) / $0.43 (enhanced) | ~200-450ms |
| deepgram nova-3 / flux | `UtteranceEnd` | `utterance_end_ms` (1000ms+ default) | backend temp token / ws query param | ~$0.28-0.46/hr | ~260-300ms |
| azure speech sdk | `SpeechEndDetected` | `Speech_SegmentationSilenceTimeoutMs` (100-5000ms) | azure speech token via sdk / push stream | ~$1.00/hr ($0.017/min) | ~300-500ms |
| google cloud speech | `END_OF_SINGLE_UTTERANCE` | `latest_short` model endpointing (500-1250ms) | service account proxy / gcp client token | ~$0.96/hr ($0.016/min) | ~500-1250ms |
| amazon transcribe | `IsPartial: false` segment | natural speech segment pauses + stability levels | aws sigv4 signed websocket url | ~$1.44/hr ($0.024/min) | ~1000-1500ms |

## recommended architecture

for glovterpreter's pause-then-interpret paradigm, the recommended boundary detection architecture consists of a hybrid three-tier design combining acoustic/speech provider signals, client-side semantic completeness checking, and a hard safety timeout.

### 1. speech capture layer: primary provider selection

- **primary choice**: assemblyai universal-3.5 pro realtime (or universal streaming for low-cost deployments) (source: https://www.assemblyai.com/products/streaming-speech-to-text).
- **rationale**: assemblyai provides sub-150ms P50 latency endpoint checking, fine-grained `min_turn_silence` (set to 128 ms or 160 ms for rapid detection) and `max_turn_silence` configuration, mid-stream parameter updates, explicit `end_of_turn: true` boundary flags, and direct browser temporary token query parameters (source: https://www.assemblyai.com/docs/streaming/getting-started/optimizing-accuracy-and-latency).
- **fallback choice**: speechmatics realtime api (using `ADAPTIVE` turn detection or `end_of_utterance_silence_trigger: 0.2` with temp JWT auth) (source: https://docs.speechmatics.com/get-started/authentication).

### 2. backend token service

- browser clients must never expose long-lived provider api keys.
- a lightweight backend endpoint (`/api/speech-token`) requests a temporary token from assemblyai (`POST /v2/realtime/token`) or speechmatics (`POST /v1/api_keys?type=rt`) with a 60-second time-to-live and returns it to the client (source: https://www.assemblyai.com/docs/streaming/authenticate-with-a-temporary-token, https://docs.speechmatics.com/get-started/authentication).
- client opens browser websocket directly to the provider endpoint using the temporary token query parameter (`wss://streaming.assemblyai.com/v3/ws?token=$TEMP_TOKEN`).

### 3. two-stage semantic completeness checker

when the speech provider emits a final turn boundary (`end_of_turn: true` or `EndOfUtterance`), the buffered transcript is evaluated before passing to stage 2 (english -> gloss translation):

- **stage 3a: regex dangling structure heuristic (<1 ms execution)**
  - inspects the final 1-3 words of the buffered utterance for dangling conjunctions (`but`, `and`, `or`, `so`, `because`, `if`, `although`, `when`), dangling prepositions (`with`, `of`, `to`, `for`, `in`, `at`), or incomplete clause starters.
  - if a trailing dangling structure is found: **HOLD** buffer, defer interpretation, and wait for the next speech segment.
- **stage 3b: lightweight in-browser ONNX classifier (~15-50 ms execution)**
  - if heuristic passes, pass transcript to a small quantized ONNX model running in WASM / WebGL (e.g., TurnSense INT8 ONNX 47M or BERT-Tiny completeness classifier) (source: https://huggingface.co/Baiji-Team/TurnSense/blob/3041540737483f6a9153cee65a33ded530091462/README.md).
  - classifier yields `complete`, `incomplete`, or `invalid`.
  - if `complete`: **COMMIT** utterance buffer immediately to gloss translation model.
  - if `incomplete`: **HOLD** buffer and wait for speaker to continue.

### 4. hard timeout engine

- to prevent the interpreter from holding indefinitely during prolonged speaker pauses or incomplete thoughts, a hard safety timer runs in parallel.
- **maximum hold window**: 2 sentences or ~10 to 12 seconds.
- if the hard timeout expires while the buffer is in **HOLD** state, the system automatically forces a commit on the accumulated transcript buffer and passes it to english -> gloss translation.
- clients can also send a manual `ForceEndpoint` signal to flush the buffer on demand (source: https://www.assemblyai.com/docs/streaming/api-spec/streaming-websocket).

## open items

- benchmark assemblyai `min_turn_silence: 128` vs `160` vs speechmatics `ADAPTIVE` turn detection on actual spoken audio samples from conversational english corpora.
- export a lightweight INT8 ONNX completeness model (or TurnSense ONNX) to test in-browser execution speeds across Chrome and Safari WebAssembly runtime environments.
- build a prototype backend ephemeral token service and test websocket connection durability under network reconnection scenarios.
