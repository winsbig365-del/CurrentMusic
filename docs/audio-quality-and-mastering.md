# Audio quality and mastering: implementation boundaries

## What the app can and cannot promise

CurrentMusic streams audio from its configured music provider and plays the returned media URL. The player can request the highest quality tier that the provider makes available to that account and track. The existing Web Audio equalizer is playback processing; it does not create a new mastered audio file. The current AI assistant handles music questions and metadata, not audio mastering.

A conversion to FLAC can be lossless relative to the decoded input, but it cannot restore information discarded by a lossy source (for example, an MP3 or AAC stream). Upsampling, increasing bit depth, or saving a stream as FLAC does not make it equivalent to a native lossless or high-resolution master. “Apple Music quality” is not a single format or mastering target; Apple Music lossless uses ALAC, while FLAC is a different lossless container/codec.

For those reasons, the app must not label an upscaled or transcoded lossy source as lossless, hi-res, or an Apple Music master, and must not silently download or permanently transform every streamed track. A larger file is not evidence of better audio.

## Safe, honest feature scope

A real mastering feature should initially be an explicit, opt-in workflow for audio files the user owns or is authorized to process—not a background operation on third-party streaming URLs. Keep the original file unchanged and write a separate output. Before processing, show the detected source format, sample rate, bit depth, channel count, approximate output size, and the proposed processing steps. Allow cancellation and let the user choose a destination and output format.

An implementation should:

1. Analyze only after the user selects files or explicitly enables a clearly described batch job. Never upload audio to an AI service without a separate, informed consent step.
2. Report measurable properties such as integrated loudness, loudness range, and true peak. Treat any AI-generated EQ or dynamics suggestions as recommendations that can be previewed, adjusted, and bypassed—not as an objectively “best” master.
3. Preserve the source sample rate and channel layout by default. Make resampling, loudness normalization, limiting, and format conversion explicit options; do not claim these recover source information.
4. Offer FLAC as a lossless output of the processed PCM, and offer Apple-compatible output only when the chosen encoder and target are explicitly identified (for example, ALAC). Do not advertise one as the other.
5. Estimate output size before processing, enforce configurable disk-space limits, expose progress, and support cancellation. Keep originals and provide a way to remove generated files.
6. Record provenance in a sidecar or UI details: source hash, source codec/metadata, processing settings, tool/model version, and output codec. Clearly mark whether the source itself was lossless.
7. Respect provider terms and copyright. Do not circumvent DRM, extract protected streams, or store provider playback URLs as mastered downloads.

## Current implementation status

This repository does not currently have a user-owned audio-file import and mastering pipeline, an audio-analysis AI integration, or a job system for large media conversions. Its AI assistant is text/chat oriented, and its player consumes provider playback URLs. Implementing automatic mastering of all app audio therefore requires a separately scoped product and architecture change; it cannot be enabled safely by changing the existing quality selector or EQ.

A practical first release would be a local-file-only, user-triggered analysis and optional processing tool. Provider-aware “highest available quality” selection can remain a separate playback feature and should report the quality actually returned by the provider without claiming it has been mastered.
