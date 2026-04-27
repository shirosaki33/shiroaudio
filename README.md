# ComfyUI-ShiroAudioTools

Small audio utility nodes for ComfyUI.

This pack helps video workflows choose between different audio sources and clean common silence problems from generated voice.

Main idea:

```text
Original audio / CozyVoice / MMAudio / other audio
        ↓
Choose which one goes into VideoCombine
```

---

## Installation

Put this folder here:

```text
ComfyUI/custom_nodes/ComfyUI-ShiroAudioTools/
```

Restart ComfyUI.

Search for:

```text
Shiro
```

or look under:

```text
audio/Shiro
```

---

## Nodes

### Trim Leading Silence (Shiro)

Removes silence only from the beginning of an audio clip.

Use it when a generated voice starts late because there is silence before the speech.

Example:

```text
Before:
[silence][silence][voice]

After:
[voice]
```

It does not speed up the voice.
It does not cut the video.
It does not remove pauses inside the sentence.

Recommended settings:

```text
threshold_db: -45
min_silence_ms: 250
padding_ms: 50
```

How to use:

```text
CozyVoice
  ↓
Trim Leading Silence (Shiro)
  ↓
Audio Selector or VideoCombine
```

If it does not remove enough silence:

```text
threshold_db: -40
```

If it cuts too aggressively:

```text
threshold_db: -50
padding_ms: 100
```

---

### Limit Long Silence (Shiro)

Shortens long pauses inside the audio.

Useful when CozyVoice pauses too long after punctuation, especially commas.

Example:

```text
Before:
"Hello, .......... this is a test."

After:
"Hello, ... this is a test."
```

It does not remove normal small pauses.
It only reduces pauses longer than the configured value.

Recommended settings:

```text
threshold_db: -45
min_silence_ms: 350
max_silence_ms: 180
```

How to use:

```text
CozyVoice
  ↓
Trim Leading Silence (Shiro)
  ↓
Limit Long Silence (Shiro)
  ↓
Audio Selector
```

If pauses are still too long:

```text
max_silence_ms: 120
```

If the voice feels too rushed:

```text
max_silence_ms: 250
```

---

### Audio Selector 8 Slots (Shiro)

Selects one audio input from up to 8 slots.

The selected slot is controlled by an INT value.

Slot map:

```text
1 = audio_1
2 = audio_2
3 = audio_3
4 = audio_4
5 = audio_5
6 = audio_6
7 = audio_7
8 = audio_8
```

Recommended setup:

```text
audio_1 = Original video audio
audio_2 = CozyVoice audio
audio_3 = MMAudio audio
```

Example:

```text
Original audio ───────────────► audio_1
CozyVoice processed audio ────► audio_2
MMAudio audio ────────────────► audio_3

Audio Selector 8 Slots (Shiro)
  ↓
VideoCombine audio
```

Use:

```text
selected_slot = 1 → Original
selected_slot = 2 → CozyVoice
selected_slot = 3 → MMAudio
```

There is also:

```text
fallback_to_audio_1
```

If enabled, and the selected slot is empty, it uses `audio_1`.

So if you select slot 4 but nothing is connected to audio_4, it falls back to original audio.

---

### Audio Slot Output (Shiro)

Outputs a fixed INT slot number.

Use this inside each audio group so the group can tell the central selector which audio slot it uses.

Example:

```text
CozyVoice group:
Audio Slot Output = 2

MMAudio group:
Audio Slot Output = 3
```

Then connect all slot outputs into an INT Any Switch:

```text
CozyVoice Slot Output = 2 ─┐
MMAudio Slot Output = 3 ───┼─► Any Switch INT ─► Audio Selector selected_slot
Other Slot Output = 4 ─────┘
```

This lets the active group automatically control the audio selector.

Simple full setup:

```text
Original audio ───────────────► Audio Selector audio_1

CozyVoice group:
CozyVoice
  ↓
Trim Leading Silence
  ↓
Limit Long Silence
  ────────────────────────────► Audio Selector audio_2

Audio Slot Output = 2 ─┐

MMAudio group:
MMAudio
  ────────────────────────────► Audio Selector audio_3

Audio Slot Output = 3 ─┘

Slot outputs ─► Any Switch INT ─► Audio Selector selected_slot

Audio Selector output ─► VideoCombine audio
```

Important rule:

```text
Only one audio group should be active at a time.
```

If more than one slot output is active, the Any Switch may choose the first one it finds.

---

### Audio Auto Selector (Shiro)

Simple legacy selector.

It chooses between:

```text
original_audio
cozyvoice_audio
mmaudio_audio
```

This is kept for older workflows.

For new workflows, use:

```text
Audio Selector 8 Slots (Shiro)
```

It is easier to expand and organize.

---

## Recommended Basic Workflow

Use this if you want manual slot control:

```text
Original audio ───────────────────────────────────────► audio_1

CozyVoice
  ↓
Trim Leading Silence (Shiro)
  ↓
Limit Long Silence (Shiro)
  ────────────────────────────────────────────────────► audio_2

MMAudio
  ────────────────────────────────────────────────────► audio_3

Audio Selector 8 Slots (Shiro)
  ↓
VideoCombine
```

Then select:

```text
1 = Original
2 = CozyVoice
3 = MMAudio
```

---

## Recommended Automatic Group Workflow

Use this if your groups already control which branch is active.

Inside each group:

```text
CozyVoice group → Audio Slot Output = 2
MMAudio group   → Audio Slot Output = 3
```

Outside the groups:

```text
Audio Slot Outputs
  ↓
Any Switch INT
  ↓
Audio Selector 8 Slots selected_slot
```

Now the active audio group sends the slot number automatically.

---

## Important Notes

These nodes work with ComfyUI `AUDIO` objects.

They do not edit video frames.

They do not speed up speech.

They do not stretch audio.

They do not magically fit long speech into a short video.

If the generated voice is longer than the video, you still need one of these:

```text
shorter text
longer video
different timing strategy
```

---

## Simple explanation

If you are confused, use this:

```text
Original audio goes into slot 1.
CozyVoice goes into slot 2.
MMAudio goes into slot 3.

Then choose 1, 2, or 3.
```

That is the whole idea.
