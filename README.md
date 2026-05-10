## EE522 – Soundscapes in Psychoacoustics

**Live Site:** [https://will943.github.io/EE522-Project/](https://will943.github.io/EE522-Project/) 

**Course:** EE 522 – Spatial Audio for Real and Virtual Spaces (Spring 2026)   
**Team:** Ali Ali, William Wong, Shamlan Alzenki  
**Institution:** University of Southern California, Viterbi School of Engineering

---

## Overview

This repository contains the full web-based experiment platform for our EE522 final project. The study investigates how room acoustics affect human cognitive performance and emotional response. Participants complete two cognitive tasks (Stroop and N-back) while listening to multi-talker babble convolved with real room impulse responses, and then listen to five emotionally designed soundscapes and report what they feel.

---

## Repository Structure

```
EE522-Project/
│
├── index.html               # Main experiment application
├── chatter.wav              # Base stimulus: multi-talker babble
│
├── ir_dry.wav               # Impulse response: Dry room (Auditorium)
├── ir_earlyrefl.wav         # Impulse response: Early reflections (Staircase)
├── ir_resonant.wav          # Impulse response: Resonant room (Tower)
├── ir_reverberant.wav       # Impulse response: Reverberant room (Chapel)
├── ir_spatial.wav           # Impulse response: Spatial room (Reactor Hall)
│
├── Happy1.wav               # Emotional soundscape: Happiness
├── Surprised2.wav           # Emotional soundscape: Surprise
├── angrynew3.wav            # Emotional soundscape: Anger
├── horror4.wav              # Emotional soundscape: Fear
├── sad5.wav                 # Emotional soundscape: Sadness
│
└── DataProcessDashboard     # Data processing and visualisation dashboard
```

---

## File Descriptions

### `index.html`
The entire experiment is contained in this single HTML file. It is built with vanilla JavaScript and the **Web Audio API**. It handles:
- Participant demographic collection
- Volume calibration
- Stroop color-word task (interference suppression)
- N-back working memory task (N = 2, 3, or 4, randomised per trial)
- Five emotional soundscape listening tests with open-ended and 6-AFC responses
- Automatic data submission to Google Sheets via Google Apps Script

---

### `chatter.wav`
The base auditory stimulus used across all six cognitive conditions. This is a **multi-talker speech babble** recording containing multiple speakers talking simultaneously at a natural conversational level. It was chosen because intelligible background speech engages the phonological loop, creating selective interference with the N-back working memory task. The same file is used for every room condition so that any performance difference between conditions is attributable solely to the room acoustics, not the stimulus content.

---

### Impulse Response Files

These five `.wav` files are **room impulse responses (IRs)** sourced from the [OpenAIR (Open Acoustic Impulse Response) library](https://www.openair.hosted.york.ac.uk/). Each IR captures the complete acoustic transfer function of a real physical space, including direct sound, early reflections, and late reverberation. In `index.html`, `chatter.wav` is convolved with each IR in real time via the Web Audio API `ConvolverNode`, making the babble sound as if it were playing inside that real space.

| File | Condition Label | Real Space | RT60 (s) | C50 (dB) | EDT (s) |
|---|---|---|---|---|---|
| `ir_dry.wav` | Dry | Arthur Sykes Rymer Auditorium, York | 0.39 | +6.26 | 0.45 |
| `ir_earlyrefl.wav` | Early Reflections | University of York Stairway | 1.16 | +4.11 | 0.73 |
| `ir_resonant.wav` | Resonant | Clifford Tower, York | 2.50 | −2.60 | 2.66 |
| `ir_reverberant.wav` | Reverberant | Lady Chapel, St Albans Cathedral | 2.42 | +1.83 | 1.50 |
| `ir_spatial.wav` | Spatial | R1 Nuclear Reactor Hall, Stockholm | 5.11 | −5.27 | 4.23 |

**RT60** = reverberation time (how long sound takes to decay 60 dB).
**C50** = clarity index (positive = speech is clear; negative = late reflections dominate and speech smears).
**EDT** = early decay time (perceived reverberance).

A sixth condition — **white noise** — has no IR file because it is synthesised directly in the Web Audio API using a `BiquadFilterNode` (bandpass, 1200 Hz, Q = 0.6) applied to a randomly generated buffer.

---

### Emotional Soundscape Files

Five original soundscapes designed to evoke specific universal emotions. Each is approximately 10 seconds long and was composed in FL Studio using MIDI and layered sound effects. Participants first describe the emotion in their own words (open-ended), then select from Happiness / Sadness / Fear / Anger / Surprise / Neutral (6-AFC) and rate their confidence on a 1–7 scale.

| File | Target Emotion | Design Notes |
|---|---|---|
| `Happy1.wav` | Happiness | Bright, fast-paced; bird chirps, metal chimes, upbeat MIDI inspired by Super Mario Galaxy's Gusty Garden Galaxy |
| `Surprised2.wav` | Surprise | Railroad ambience building in volume, Kitty MIDI, single gasp sound effect |
| `angrynew3.wav` | Anger | Repeated car honks targeting 2–5 kHz sensitivity range; car skid; slowed "I'm Walkin' Here" dialogue from Midnight Cowboy |
| `horror4.wav` | Fear | Thunder strike + wolf growl at peak amplitude; ominous steel pan MIDI with water-space effect; wolf howl with reverb at end |
| `sad5.wav` | Sadness | Rain normalised to −10 dB; slowed reverb bell; string MIDI with pitch fluctuation mimicking a crying voice |

---

### `DataProcessDashboard`
An HTML dashboard for processing and visualising the raw participant data collected from Google Sheets. Used internally by the team for statistical analysis, figure generation, and verifying data quality before running the formal analysis pipeline.

---

## How the Experiment Works

1. Participant visits the live site or opens `index.html`
2. Completes a short demographic survey
3. Sets volume using a stereo calibration tone
4. Completes practice trials for both cognitive tasks
5. **Cognitive test:** Performs Stroop (8 trials × 6 conditions) and N-back (4 trials × 6 conditions) tasks with background audio playing throughout each block
6. **Emotional test:** Listens to each of the 5 soundscapes and responds with an open-ended description and a 6-AFC forced choice
7. Data is automatically submitted to a Google Sheet on completion

---

## Audio Signal Chain (Cognitive Test)

```
chatter.wav
    └── BufferSourceNode (loop)
            └── ConvolverNode (room IR)  ← set per condition
                    ├── DryGainNode (0.20)
                    └── WetGainNode (0.80)
                            └── MasterGainNode
                                    └── AudioContext.destination
```

**`chatter.wav`** — the raw multi-talker babble audio file, loaded into memory as the sound source at startup.

**`BufferSourceNode (loop)`** — the Web Audio API node that plays the audio file. Set to loop so the babble repeats continuously throughout each condition with no gaps or cutouts.

**`ConvolverNode (room IR)`** — the core of the signal chain. This node convolves the babble in real time with the room impulse response file for the current condition, making the babble sound as if it were physically playing inside that real space. The IR is swapped out at the start of each condition. For the white noise baseline this node is bypassed entirely — a randomly generated buffer is passed through a `BiquadFilterNode` instead.

**`DryGainNode (0.20)`** — taps off 20% of the completely unprocessed signal before room treatment. Mixing in a small amount of dry signal prevents the output from sounding unrealistically distant or fully submerged in reverb.

**`WetGainNode (0.80)`** — carries the fully convolved (room-processed) signal at 80% volume. The dry and wet streams are summed together at this point, which is standard practice in reverb processing.

**`MasterGainNode`** — a final gain stage that applies per-condition volume adjustments to equalise perceived loudness across conditions, so differences in participant performance reflect acoustics rather than loudness differences.

**`AudioContext.destination`** — the endpoint of the Web Audio API graph, representing the participant's physical audio output device (headphones or speakers).

White noise baseline bypasses the ConvolverNode entirely and routes a filtered random buffer through the MasterGainNode directly.

---

## Data Collection

Response data (participant ID, condition, reaction time, correctness, N-value, emotion selections, confidence ratings) are submitted via `FormData` POST to a Google Apps Script web app endpoint, which appends each row to a Google Sheet for later analysis.

---

## Citation / Acknowledgements

- Room impulse responses from the [OpenAIR Library](https://www.openair.hosted.york.ac.uk/) (University of York)
- Acoustic parameters verified using [ODEON Room Acoustics Software](https://odeon.dk/)
- Happy soundscape MIDI inspired by *Gusty Garden Galaxy* — Super Mario Galaxy (Nintendo)
- Sad soundscape MIDI inspired by *Benedict's Path* — Triangle Strategy (Square Enix)
- Anger soundscape dialogue from *Midnight Cowboy* (1969, dir. John Schlesinger)
- Universal emotions framework: Ekman, P. (2019). *Universal Emotions*. Paul Ekman Group
