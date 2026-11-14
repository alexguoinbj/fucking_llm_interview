# 12.8 Video and Audio Understanding — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 15 questions

**1. What is the Typical Architecture of Video Understanding Models?**

<details>
<summary>💡 Show Answer</summary>

The typical architecture of a video understanding model is **Video Encoder + Temporal Modeling + Language Model**:

**Video Encoder:**
- Frame-by-frame encoding: Uses an image encoder (e.g., ViT) for each frame, and then performs temporal modeling on the frame sequence (common solutions: Video-LLaMA, LLaVA-Video);
- 3D convolutional encoder (e.g., VideoSwin): Directly models spatiotemporal voxels to capture temporal dynamics;
- Spatiotemporal attention (TimeSformer, ViViT): Adds temporal Self-Attention alongside the spatial patch dimension.

**Temporal Modeling:** Temporal Pooling (average or max pooling across the temporal dimension of frame features); LSTM/Transformer Encoder (modeling temporal dependencies across frame sequences); Q-Former with Temporal Attention (a Q-Former incorporating inter-frame temporal attention).

**Integration with Language Models:** Concatenates (or compresses) the visual token sequences of individual frames before feeding them into the language model; uses temporal positional encodings to distinguish tokens from different frames; supports tasks such as video captioning, temporal grounding, and video QA.

</details>

**2. Compared to Static Images, What Additional Challenges Must VLMs Address When Processing Video and Other Multimodal Data?**

<details>
<summary>💡 Show Answer</summary>

Processing video data introduces the temporal dimension, bringing additional and unique challenges compared to static images:

1. **Temporal information representation**: Requires understanding temporal relationships between frames, such as object motion trajectories, action continuity, and event causality. Solutions include frame sampling combined with temporal fusion modules (temporal attention, 3D convolutions, concatenated pooling), as well as spatiotemporal modeling (3D CNNs, Video Transformers like ViViT) that simultaneously consider spatial and temporal dimensions during the feature extraction stage.
2. **Huge computational and storage overhead**: A short video may contain hundreds or even thousands of frames, resulting in data volumes far exceeding a single image. Solutions: Sparse sampling (processing only significantly changing or representative frames) and feature compression (compressing or pooling frame-by-frame features to reduce the token count).
3. **Long-range dependency modeling**: Key causal relationships may span long time windows. Solutions: Adopting Transformer-like architectures to model inter-frame relationships, leveraging their global receptive field advantages.
4. **Increased complexity of multimodal fusion**: Videos are typically accompanied by audio (speech, background sounds) and subtitles, requiring the design of more complex alignment and fusion modules to process multiple asynchronous or synchronous time-series data.

</details>

**3. How Do Multimodal Models Address the Token Explosion Problem Caused by "Excessive Frames" in Video Inputs?**

<details>
<summary>💡 Show Answer</summary>

Even a video of just a few tens of seconds, if processed by sampling several frames per second and encoding each frame into hundreds of visual tokens, can quickly accumulate tens of thousands or more tokens—far exceeding the context length that language models can efficiently handle. Therefore, specialized strategies are required to manage the total token count corresponding to the video:

- **Uniform / Keyframe Sampling**: Instead of feeding every single frame of a video into the model, representative frames are uniformly sampled at fixed time intervals (e.g., 1 frame per second), or going a step further, by detecting the degree of content change to select only "keyframes" where significant visual changes occur (avoiding redundant processing of large numbers of nearly identical adjacent frames);
- **Single-frame Visual Token Compression**: For each sampled frame, compression techniques such as Q-Former or Pixel Shuffle are applied to compress the per-frame representation—which originally might contain hundreds of tokens—into a smaller number of tokens, thereby controlling the total token count simultaneously along both dimensions ("number of frames" and "tokens per frame");
- **Cross-frame Temporal Compression Modules**: Specialized modules are designed (such as pooling and merging similar tokens across adjacent frames, or introducing attention mechanisms along the temporal dimension to aggregate multi-frame features) to further exploit the high visual similarity and information redundancy between adjacent frames. This aggregates and compresses multi-frame information into a more compact global representation, rather than simply stacking each frame's (even compressed) tokens linearly.
These strategies are typically used in combination to control the total video token count—which might otherwise reach tens of thousands—down to a reasonable range (e.g., several thousand tokens) that language models can handle efficiently.

</details>

**4. How Do Multimodal Models Support Long Contexts?**

<details>
<summary>💡 Show Answer</summary>

Multimodal long contexts face two major challenges: **a large number of visual tokens** and **text context window limitations**.

**Visual Token Compression:** Reduces the number of image tokens (larger patch sizes, Token Pooling); dynamic compression (attention-weighted compression of visual tokens based on text queries); Q-Former-style structures (fixed number of query tokens, independent of image resolution).

**Text Long-Context Extension:** Positional encoding extrapolation (RoPE + YaRN/LongRoPE); sliding window + sparse attention; retrieval augmentation (storing historical multimodal information in external memory and retrieving as needed).

**Video Long Context:** Keyframe sampling (uniform or scene-change-based adaptive sampling); cross-frame token compression (merging similar tokens from adjacent frames); temporal positional encoding (assigning different temporal positional encodings to tokens from different frames).

</details>

**5. How Is Modality Alignment Achieved Between Audio and Text?**

<details>
<summary>💡 Show Answer</summary>

**Feature Extraction Level:** Uses pre-trained models such as Wav2Vec 2.0, HuBERT, and Whisper Encoder to extract audio features, which are then mapped to the token space of the language model via a projection layer (MLP or Q-Former).

**Contrastive Learning Alignment:** Similar to CLIP, performs contrastive learning on audio-text pairs (e.g., CLAP: Contrastive Language-Audio Pretraining) to learn consistent representations of audio and text in a shared semantic space.

**Sequence Alignment:** CTC (Connectionist Temporal Classification) trains sequence-to-sequence alignment without requiring frame-level annotations; Attention-based Encoder-Decoder uses Cross-Attention to dynamically align audio frames when decoding text (the approach used by Whisper).

**End-to-End Unified Modeling:** After audio is tokenized (e.g., EnCodec, SoundStream), audio tokens and text tokens are unified and modeled together using a language model (the concept behind GPT-4o's voice mode), achieving truly unified audio-text generation and understanding.

</details>

**6. What Is Audio Tokenization? How Do Speech Multimodal Models Convert Speech into Token Representations?**

<details>
<summary>💡 Show Answer</summary>

Audio Tokenization is the process of converting continuous audio waveforms into discrete or continuous token sequence representations, primarily via three approaches:

1. **Continuous Representation Approach**: Extracts acoustic features of speech (e.g., Mel-spectrograms, Wav2Vec features) and converts them via an encoder into a sequence of continuous vectors, which are fed directly into the Transformer.
2. **Discrete Representation Approach**: Uses neural codecs (e.g., Encodec, SoundStream) to compress speech into discrete codec tokens, forming discrete sequences similar to text tokens.
3. **Hybrid Approach**: First extracts continuous features and then converts them into discrete tokens through a quantization module, balancing information richness with unified processing alongside text tokens.



![语音大模型中波形到离散神经Token的转换处理流程图 (Waveform to discrete neural token transformation in speech models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig28_audio_tokenization.png)

</details>

**7. What Is the Architectural Design Concept of Any-to-Any Multimodal Models (e.g., Supporting Simultaneous Image, Audio, and Text Inputs and Outputs)?**

<details>
<summary>💡 Show Answer</summary>

The goal of an Any-to-Any model is to build a single system capable of flexibly accepting inputs in **any** combination of modalities (e.g., simultaneous image and speech inputs) and generating outputs in **any** combination of modalities (e.g., simultaneously outputting a text answer, a synthesized audio clip, or even a newly generated image), achieving true free conversion and combination across modalities.

Its typical architectural design concept is "Multimodal Encoders/Decoders + Unified Central Representation Space":

- **Input Side**: Specialized encoders are designed for each supported modality (e.g., ViT-style encoders for images, Wav2Vec-style encoders for audio, video encoders for videos) to convert raw inputs of each modality into their corresponding feature representations, which are then mapped via their respective connector modules into a **shared central representation space** (typically the token embedding space of the language model);
- **Central Processing**: Information from all modalities is fused and reasoned over within this unified central representation space, typically by a Transformer-based language model backbone, consistent with the unified multimodal architecture concept described earlier;
- **Output Side**: Specialized decoders/generators are likewise designed for each modality that needs to be generated (e.g., the language model's own decoding head for text, diffusion models or autoregressive image generation modules for images, speech synthesis modules for speech). Information output from the central representation space is routed to the corresponding decoder/generator according to the target output modality requested by the user, converting it into the final content in the target modality.
This architectural design of "modality-specialized encoding/decoding modules + a unified shared central reasoning backbone" balances targeted processing capabilities for modality-specific data characteristics with the unification demands of cross-modal information fusion and joint reasoning. It represents the mainstream architectural paradigm currently explored for "all-in-one" multimodal models.

</details>

**8. What Is the "Missing Modality" Problem in Multimodal Models, and How Is It Handled?**

<details>
<summary>💡 Show Answer</summary>

Missing modality refers to the scenario where a model learns from complete data containing multiple modalities (e.g., both images and text) during the training phase, but during actual inference and deployment, certain modal data relied upon during training may be **unavailable** or missing in specific scenarios (for example, a user prompt provides only a text question without an accompanying image, or in video understanding, a video loses its audio track leaving only visual frames). If the model was never exposed to such "incomplete modality" scenarios during training, its behavior may become unpredictable or fail completely when encountering missing modalities in practice (e.g., the model might forcibly "assume" the presence of a non-existent image input, leading to severe errors in generated content).

Common approaches to handling this issue include:

- **Introducing Modality Dropout During Training**: During the training process, certain existing modal inputs are actively and artificially dropped with a certain probability (e.g., replacing image inputs with empty inputs at a given rate). This forces the model to learn during training a more robust behavioral pattern: "even if a modality is missing, make maximum use of the remaining available modal information to provide a reasonable response," rather than excessively relying on the ideal assumption that all modalities are complete;
- **Designing Default Placeholder Representations for Missing Modalities**: When a modality is missing, instead of simply leaving it blank or raising an error, a pre-designed default "placeholder" representation (such as a special fixed vector representing "no image") is used to fill the position where that modality's data would normally be. This keeps the input format structurally complete and consistent, preventing anomalous model behavior caused by sudden changes in input format.

</details>

**9. What Is the "Temporal Grounding" Task in Video Understanding, and How Is It Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Temporal Grounding (also known as Video Moment Retrieval) refers to the task where, given a (typically long) video and a natural language query describing a specific event or action (e.g., "the moment the athlete jumps" or "after the two people start shaking hands"), the model must accurately predict the specific time interval (represented by start and end timestamps) along the complete video timeline corresponding to that description, rather than merely performing coarse-grained overall judgments like video classification or video QA (e.g., judging "whether the entire video is relevant to the description"). This task imposes high demands on the model's fine-grained understanding of event temporal boundaries within videos.

**Evaluation Methods**: Similar to visual grounding in the spatial dimension (measured by bounding box IoU), temporal grounding uses **Temporal Intersection over Union** (Temporal IoU) to measure prediction accuracy: the predicted time interval and the human-annotated Ground Truth time interval are treated as two intervals on the timeline, and the duration of their overlap divided by the duration of their union yields an IoU value between 0 and 1. Typically, several different IoU thresholds are set (e.g., 0.3, 0.5, 0.7), and the proportion of predictions with IoU exceeding the threshold is recorded as the Accuracy@IoU metric (higher thresholds represent stricter requirements on localization precision). Evaluating performance across multiple thresholds provides a comprehensive reflection of the model's temporal grounding capabilities.

</details>

**10. What Is the Fundamental Difference Between Streaming Video Understanding and Offline Video Understanding?**

<details>
<summary>💡 Show Answer</summary>

**Offline Video Understanding** assumes that the complete video file is fully available before processing begins. The model can freely access any timestamp of the video (including "future" frames) and feeds all (or sampled subset of) frames into the model at once to generate an answer. This is the default setting adopted by the vast majority of current video QA and video captioning research and benchmarks.

**Streaming Video Understanding** (Online) simulates application scenarios closer to the real world—video content arrives in real time frame-by-frame or chunk-by-chunk over time (e.g., real-time camera surveillance streams, live broadcasts). When processing content at the "current timestamp," the model has zero knowledge of and no access to "future" frames that have not yet occurred. It must process data as it arrives and produce real-time outputs (e.g., detecting anomalous behavior in surveillance footage in real time and immediately issuing an alert).

This fundamental distinction imposes different requirements on model capabilities: offline understanding can rely on global Self-Attention computations over the entire video sequence to make full use of bidirectional context; streaming understanding, however, requires the model to possess **incremental** state-updating capabilities—every time a small segment of newly arrived content is processed, the model must efficiently update its understanding of the "current situation" without re-processing all historical frames (e.g., maintaining a continuously updated, compressed summary of historical state). This places far higher demands on online inference efficiency and state maintenance mechanisms (such as the aforementioned memory bank methods) compared to offline scenarios, explaining why streaming understanding remains a relatively more challenging research direction that currently lags behind offline understanding.



![在线滑动窗口缓存与离线全局缓存视频处理机制对比图 (Online sliding-window buffer versus offline cached video processing.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig29_streaming_vs_offline.png)

</details>

**11. How Do Speech Multimodal Models Achieve "Full-Duplex" (Interruptible at Any Time) Conversational Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Traditional speech dialogue systems typically employ a "half-duplex" interaction mode: the system must wait for the user to finish an entire utterance (determining "it is the system's turn to speak" via speech pause detection, etc.) before it begins generating and playing an audio response, and it usually cannot respond to user interruptions while speaking. This mode differs significantly from natural human conversation, where speakers can interrupt, supplement, or switch topics at any time.

Achieving **full-duplex** interaction—where users can interrupt at any moment while the system is speaking, and the system can immediately cease its current output and respond appropriately—typically requires technological support in the following areas:

- **Continuous Streaming Listening**: Audio input cannot wait for an "utterance to complete" before being processed as a whole; instead, it must be fed into the model continuously and in real time at a fine temporal granularity (e.g., audio chunks of tens to hundreds of milliseconds), enabling the model to detect "new incoming audio input" within a very short time after the user starts speaking;
- **Parallel Input Monitoring and Output Generation**: The model architecture must support monitoring and processing newly arriving user speech input in parallel while continuously generating speech output. Once a valid intent to interrupt is identified (rather than background noise or simple conversational backchanneling), it must be able to immediately halt the ongoing output generation process;
- **Understanding Interruption Semantics**: The model must not only "detect" an interruption but also understand the intent behind it—whether the user wants to supplement missing information, correct a previous statement, or switch to an entirely new topic. This requires training data containing abundant, diverse dialogue samples from real interruption scenarios, enabling the model to learn appropriate response strategies for different interruption cases (whether to continue the original topic, adjust the direction of the answer, or switch topics entirely).
These capabilities impose special requirements on training data construction and underlying architectural design (such as supporting bidirectional streaming processing) distinct from traditional voice assistants, representing one of the key technologies determining whether speech interaction can reach a "natural conversation" standard.

</details>

**12. What Is Lip Sync and Its Application in Audio-Visual Multimodal Generation?**

<details>
<summary>💡 Show Answer</summary>

Lip synchronization (Lip Sync) refers to ensuring that when generating or editing videos featuring speaking characters, the character's mouth movements (lip shape, opening amplitude, oral shape during pronunciation) correspond precisely in both **temporal** and **morphological** dimensions to the accompanying audio. This ensures that the viewer's perceived lip movements match the heard audio seamlessly without any sense of misalignment. This technology holds significant application value in digital human video generation (virtual anchors, virtual customer service agents) and cross-lingual lip re-dubbing for film and television (dubbing foreign films into another language while modifying actor lip movements to match the new language).

In terms of technical implementation, a strong mapping relationship must be established between audio features (e.g., speech sequences segmented by phonemes or finer acoustic feature units) and the generation or modification of facial regions (especially the mouth area) in the video: a common approach is to train a specialized lip generation model that takes the input audio feature sequence as conditioning to drive frame-by-frame generation of corresponding mouth shapes (which can involve directly generating pixel content for the mouth region or first generating an intermediate representation of mouth landmarks/shapes before rendering and compositing onto the original face). Simultaneously, additional constraints are required to ensure that the generated mouth dynamics remain naturally consistent with the rest of the video frame (e.g., facial expressions, head pose, lighting conditions) without producing visible stitching artifacts.

</details>

**13. What Are the Different Modeling Focuses Between Music/Environmental Sound Understanding and Speech Understanding?**

<details>
<summary>💡 Show Answer</summary>

Although speech, music, and environmental sounds all belong to the broader category of the "audio modality," the information types and structural characteristics they carry differ significantly, leading to distinct focuses during modeling:

- **Speech Understanding**: The core objective is to extract **linguistic information** (phoneme sequences, vocabulary, syntactic semantics) from acoustic signals. Such information relies heavily on fine-grained local temporal patterns (transitions between different phonemes typically occur on the scale of tens of milliseconds). Thus, speech understanding model design focuses more on capturing fine-grained phoneme-level acoustic feature variations over short time scales;
- **Music Understanding**: The core objective is to capture **musicological** features such as melodic contours, rhythmic patterns, and harmonic structures. These features often exhibit strong periodicity and structural patterns over long time scales (e.g., repetition of a melody or musical phrase transitions spanning seconds to tens of seconds). Consequently, music understanding models typically need to model periodic and structural patterns over longer time windows than speech models, with relatively weaker requirements for fine-grained phoneme-level modeling;
- **Environmental Sound / Audio Event Understanding**: The core objective is to identify sound **categories** (e.g., determining whether a sound is a "dog bark" or a "car horn"). Such tasks depend more on timbre features and spectral shapes (different sound sources have distinct spectral "fingerprints"), with lower demands for fine temporal modeling, making them closer to traditional audio classification tasks. Model design often focuses on spectral feature extraction and classification capabilities rather than sequence-to-sequence temporal modeling.
Because their modeling focuses differ, general multimodal audio models in practice typically need to cover all three signal types in pre-training data and objectives, or adopt specialized feature extraction branches for different downstream tasks, to maintain competency across all three domains.

</details>

**14. How Do Multimodal Models Handle Subtitles as a Special Video-Accompanying Modality?**

<details>
<summary>💡 Show Answer</summary>

Subtitles are essentially a text stream precisely aligned with the video timeline. Their uniqueness lies in being both "text" and naturally carrying temporal annotation information indicating "which specific time interval in the video this text corresponds to." Consequently, they offer unique utility during both training and inference of multimodal models:

**In the Training Phase:** Subtitles (especially professional subtitles with accurate timestamps written by humans) are often treated as a **weakly labeled source of video/image captions**. The corresponding visual frames can be automatically paired into "image-caption" training samples, drastically reducing the cost of manual video captioning and serving as a key data source for building large-scale video-text training data. Simultaneously, subtitle timestamps can be used to construct training data with time interval annotations required for temporal grounding tasks.

**In the Inference Phase:** If the input video includes native subtitles (such as news broadcasts, instructional videos, or subtitled movies), the model can directly read the subtitle text to assist in understanding dialogue and plot development. This approach is more accurate and computationally cheaper than relying on end-to-end speech recognition modules to extract spoken content. However, two potential issues must be managed: first, subtitle timestamps may suffer from slight time offsets relative to actual visual frames (especially with user-added subtitles in web videos); second, subtitle text typically reflects only "what was said," failing to capture visual details that are unmentioned yet equally important (e.g., background environments, non-verbal body language). Therefore, subtitle information should serve as a **supplement** to visual understanding rather than a replacement.



![处理嵌入式视觉字幕与音频文本转写的双规路径结构图 (Parallel pipelines for parsing embedded visual subtitles and audio transcripts.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig30_subtitle_modality.png)

</details>

**15. How Does the "Memory Bank" Method Work in Long Video Understanding?**

<details>
<summary>💡 Show Answer</summary>

When facing long videos of several minutes or even hours, retaining visual tokens for all frames and feeding them directly into the model results in a sequence length far exceeding the context window of language models. The core idea of memory bank methods is: rather than attempting to "remember all details of every frame," dynamically compress and maintain a limited-scale "memory" representation summarizing processed content, mimicking how humans recall video content.

A typical workflow includes:

1. **Segment-by-segment Processing**: Long videos are split into multiple shorter segments and processed sequentially, rather than loading the entire video at once.
2. **Memory Compression and Updating**: After processing each segment, key information from that segment (either compressed visual features or textual summary descriptions) is merged and updated with accumulated memory bank contents. Common practices include: weighted fusion of old and new memories, using attention mechanisms to filter and retain information most relevant to current memories, or directly discarding highly redundant, low-information historical memory entries (analogous to a "forgetting mechanism").
3. **Retrieval on Demand**: When answering questions about a specific timestamp or overall content of the video, the model retrieves memory entries from the memory bank that are most relevant to the prompt, combining them with current segment information to generate an answer without re-processing the entire video.

This design of "streaming processing + dynamic memory compression + retrieval on demand" enables models to process videos of arbitrary length with nearly constant (rather than linearly scaling with video length) computational and memory overhead. It represents one of the mainstream technical directions in long video understanding, where the core tradeoff lies between memory compression ratio and information preservation accuracy.

</details>

