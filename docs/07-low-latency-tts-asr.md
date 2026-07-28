---
title: "Engineering Low-Latency Voice AI: How to Bring E2E Latency below 1.5 Seconds"
date: "2026-03-03"
tags: ["Technical Exploration"]
slug: "low-latency-tts-asr"
---

# Engineering Low-Latency Voice AI: How to Bring E2E Latency below 1.5 Seconds

Phone calls are a real-time interaction. When people talk on the phone, the other person typically begins responding within 300–500 ms after the first person finishes speaking. After two seconds, people start to wonder whether the line has disconnected.

This psychological window is the fundamental engineering constraint for voice AI products. **End-to-end latency must stay below 1.5 seconds, with an ideal P95 below 1.2 seconds.**

Reaching this goal requires careful design of every stage in the voice processing pipeline.

## 1. Breaking Down the Full Pipeline

A complete AI voice interaction, from the moment the user finishes the last word to the moment AI starts playing its response, passes through the following stages:

```
User finishes speaking
    ↓
VAD silence detection (End-of-Turn Detection)    ~80ms
    ↓
STT speech recognition (Speech-to-Text)          ~200-250ms
    ↓
LLM inference (first token generation)           ~300-450ms
    ↓
TTS first-packet synthesis (Text-to-Speech, TTFP) ~80-120ms
    ↓
Network transmission + buffering                 ~80-120ms
    ↓
User hears the first syllable                     Total ~740-1,020ms
```

On the ideal path, E2E latency is around 890 ms, leaving a healthy margin. In production, however, network jitter, model cold starts, long inputs, and slow tool calls can push latency above two or three seconds.

The engineering work behind low-latency voice AI is therefore fundamentally about **controlling the worst case at every stage**, not simply optimizing the average case.

## 2. Voice Activity Detection: Get the Turn Boundary Right Before Making It Fast

VAD (Voice Activity Detection) is the first stage in the pipeline. It detects whether the user has finished speaking. This stage has an interesting tension:

- **Cut too early:** The user has not finished, so AI begins processing incomplete input.
- **Cut too late:** The system waits too long before processing, increasing perceived latency.

The common industry solution is a combined strategy:

**Energy-threshold silence detection** acts as the fast path. When it detects 300–400 ms of continuous low-energy audio, it treats the turn as complete and starts the pipeline. This handles around 85% of normal scenarios.

**Semantic completeness detection** acts as a safety net. It performs a quick semantic check on the detected end of the input. If the recognized text starts with a question word but does not yet form a complete question, the system may be looking at a truncated turn and should delay processing slightly. This reduces comprehension errors caused by incorrect VAD cuts.

**Interruption handling:** When the user interrupts while AI is speaking, VAD must immediately stop TTS playback and restart the STT → LLM flow. This is essential for a natural conversational experience, but it requires the system to interrupt an active TTS stream as soon as VAD detects speech.

## 3. Choosing STT: Measured Data from Deepgram Nova-3

The three key metrics for speech recognition are time to first word (TTFW), word error rate (WER), and multilingual support.

During evaluation, iMorph.ai compared several leading STT services:

| Service | Time to first word | WER, conversational English | Languages | Real-time streaming |
|---------|--------------------|-----------------------------|-----------|---------------------|
| Deepgram Nova-3 | ~220ms | ~6% | 30+ | Yes |
| OpenAI Whisper (API) | ~800ms+ | ~5% | 99 | Primarily batch |
| Google Speech-to-Text | ~350ms | ~7% | 125 | Yes |
| AssemblyAI | ~300ms | ~7% | 17 | Yes |

**The core advantages of Deepgram Nova-3:**

The first is its streaming architecture. Deepgram uses true streaming STT: it generates a partial transcript while the user is speaking instead of waiting for the user to finish. By the time VAD detects silence, most STT processing is already complete and only the final transcript confirmation remains.

The second is optimization for phone conversations. Nova-3 is specifically trained for telephone audio at 8 kHz narrowband quality, conversational language, and accented English. These are the primary input types in enterprise receptionist scenarios.

The third is the `endpointing` parameter. Deepgram allows the sensitivity of end-of-turn detection to be configured. When VAD silence detection fires, the system can tell Deepgram to force transcription completion, reducing wait time further.

**Special handling for multilingual scenarios:**

Chinese, Spanish, and Korean calls are not unusual among North American small and midsize businesses. iMorph.ai uses a runtime language-switching mechanism: the system prompt remains in English, and when STT detects non-English input in the first two utterances, it automatically switches the TTS language and tells the LLM to respond in the corresponding language without restarting the session.

## 4. Choosing TTS: Streaming Synthesis and First-Packet Latency

The key TTS metric is **time to first packet (TTFP)**: the time from when the LLM starts outputting text to when TTS synthesizes the first playable audio segment.

Traditional TTS APIs use a "submit the full text → return the full result" model. They must wait for the LLM to generate a complete sentence before synthesis can begin. This adds at least 500–800 ms of unnecessary waiting.

**Streaming TTS** changes this model. As the LLM generates tokens, TTS starts synthesizing from the first sentence fragment, allowing LLM output and TTS synthesis to run in parallel.

iMorph.ai uses **Cartesia Sonic** as its primary TTS engine:

- TTFP of ~80–100 ms, among the best in the industry.
- Natural delivery that avoids an obvious mechanical TTS sound.
- Emotional control that can switch between calm, concerned, and enthusiastic tones.
- Streaming output compatible with streaming LLM input.

**Key engineering detail: text chunking strategy**

TTS needs to receive text in sentence or short-phrase units. Chunks that are too short, such as individual words, sound unnatural; chunks that are too long, such as full paragraphs, increase wait time. iMorph.ai uses punctuation as a chunking signal. Whenever the LLM outputs punctuation such as `,`, `.`, or `?`, the accumulated fragment is immediately pushed to TTS for parallel synthesis.

```
LLM stream: "Hello," → Push to TTS ①
             "Thank you for calling today," → Push to TTS ②
             "I'm AI assistant Aya," → Push to TTS ③
             "How can I help you?" → Push to TTS ④
```

TTS ① finishes synthesizing while the LLM is still generating ②, ③, and ④, so the user's first words and the following sentences connect with almost no gap.

## 5. Telephony Layer: Telnyx and Real-Time WebSocket Streaming

The telephony layer for voice AI must handle codec differences in the PSTN (Public Switched Telephone Network). Phone networks typically use G.711 μ-law at 8 kHz and 64 kbps, while modern STT/TTS engines prefer 16 kHz or 24 kHz PCM audio.

iMorph.ai uses **Telnyx** as its telephony layer because it provides:

- WebSocket media streams for bidirectional real-time audio transmission.
- `TeXML`, a TwiML-like call control interface.
- Native call transfer and three-way calling, which are required for handoff scenarios.
- Strong North American coverage and stable latency.

The key configuration is to enable `g711_ulaw_8000` encoded input and perform real-time resampling from 8 kHz to 16 kHz on the Telnyx → STT path. This is the best practice for minimizing audio quality loss.

**Bidirectional WebSocket stream design:**

```
Telnyx ←→ WebSocket Server ←→ STT (Deepgram)
                              ←→ LLM (Claude)
                              ←→ TTS (Cartesia)
```

The WebSocket Server coordinates the entire pipeline: it receives the real-time audio stream from Telnyx and forwards it to Deepgram; sends Deepgram's text to the LLM; chunks the LLM token stream and sends it to Cartesia; then receives Cartesia's audio packets and returns them to Telnyx for playback.

Nothing is written to disk in the process. The entire pipeline runs in memory and across network streams.

## 6. Silence Padding and Perceived-Latency Optimization

There is still a gap between engineering "low latency" and an experience that feels smooth and natural. Several perception-level optimizations are equally important:

**Thinking indicators:** When LLM inference is expected to take more than one second, such as during a tool call, insert a natural acknowledgment such as "Okay, let me check that for you..." This does not reduce actual latency, but it substantially reduces the user's anxiety while waiting.

**Prioritize speech completeness over speed:** When TTS is playing a sentence, do not interrupt immediately for slight noise detected by VAD. Trigger interruption only when clear human speech is detected through energy and voice features. This avoids accidental interruptions caused by background noise.

**Silence padding:** Insert 50–100 ms of silence after each TTS segment to give the user natural space to respond and avoid the feeling that AI starts listening again too quickly after finishing a sentence.

---

Engineering low-latency voice AI is not a single-point optimization. It is a systems design problem across the entire pipeline. Every 50 ms improvement at every stage matters, but the real key is **making P95 latency predictable, not merely making the average look good.** In production, a system with a three-second P95 does not feel like a "one-second average." It feels like "sometimes it makes me wait three seconds." That is what sets the experience ceiling for a voice AI product.

---
