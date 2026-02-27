# Music Theory Benchmark (Seventh Chords)

This benchmark measures how well a model can derive seventh chords from a given scale and chord degree. It generates a dataset of scale/mode/degree prompts, sends them to a model via n8n, and evaluates the model output against the expected chord tones.

## What It Does

- Builds a dataset of seventh-chord prompts across several modes and roots
- Calls a model to return chord notes in structured JSON
- Compares model output to the expected chord notes
- Stores results (success/failure counts and accuracy) in a data table

## Workflow File

- Evaluation workflow: [Evaluation.json](Evaluation.json)

## Inputs

The benchmark expects two inputs when executed as a sub-workflow:

- `modelName` (string): OpenRouter model ID, e.g., `google/gemini-2.5-flash`
- `totalRuns` (number): How many random prompts to evaluate (1-20 recommended)

You can also run it from the included form trigger, which asks for the same inputs.

## Output Format

The AI agent is instructed to return JSON only. Each item should match this schema:

```json
[
	{
		"scaleRoot": "F",
		"scaleMode": "Dorian",
		"chordDegree": 2,
		"chordNotes": ["G", "Bb", "D", "F"]
	}
]
```

## How Evaluation Works

- The dataset generator builds full scales for several modes and roots.
- For each prompt, it computes the correct seventh chord (root, third, fifth, seventh).
- The model returns `chordNotes` and the workflow compares them to the expected notes.
- Results are aggregated into `success` and `failed` counts.
- An overall accuracy percentage is computed and stored per model.

## Data Tables Used

Two n8n Data Tables are referenced:

- `seventhChordBenchmarkDataset` stores the generated prompt/answer pairs
- `seventhChordBenchmarkResults` stores per-model accuracy

If you import this workflow into a different n8n instance, recreate these tables and remap the nodes.

## Requirements

- OpenRouter credentials configured in n8n
- An n8n instance with Data Tables enabled

## Running The Benchmark

1. Run the dataset generation section once (manual trigger) to populate the dataset table.
2. Use the form trigger to select a model and number of runs.
3. The workflow reports the accuracy and stores it in the results table.

## Notes

- The workflow expects strict JSON output from the model.
- Some prompts include accidentals; the model must use correct spelling.
- Accuracy is based on exact matching of chord note arrays.

## Sample Dataset

The dataset includes seventh chords across various scales and modes. Here's a sample:

| Scale Root | Scale Mode | Chord Degree | Full Scale | Expected Chord Notes | Seventh Chord |
|---|---|---|---|---|---|
| G# | Phrygian | 3 | G#;A;B;C#;D#;E;F# | B;D#;F#;A | B7 |
| Eb | Lydian | 5 | Eb;F;G;A;Bb;C;D | Bb;D;F;A | Bbmaj7 |
| F# | Phrygian | 2 | F#;G;A;B;C#;D;E | G;B;D;F# | Gmaj7 |
| Bb | Locrian | 5 | Bb;Cb;Db;Eb;Fb;Gb;Ab | Fb;Ab;Cb;Eb | Fbmaj7 |
| A# | Dorian | 2 | A#;B#;C#;D#;E#;F##;G# | B#;D#;F##;A# | B#m7 |
| C# | Mixolydian | 7 | C#;D#;E#;F#;G#;A#;B | B;D#;F#;A# | Bmaj7 |
| Bb | Phrygian | 4 | Bb;Cb;Db;Eb;F;Gb;Ab | Eb;Gb;Bb;Db | Ebm7 |
| D# | Phrygian | 5 | D#;E;F#;G#;A#;B;C# | A#;C#;E;G# | A#m7b5 |

*Full dataset: [seventhChordBenchmarkDataset.csv](seventhChordBenchmarkDataset.csv)*

## Benchmark Results

Accuracy results across tested models (20 runs each):

| Model | Accuracy | Result | Total Runs |
|---|---|---|---|
| google/gemini-2.5-pro | 100% | 1.0 | 20 |
| google/gemini-3-pro-preview | 95% | 0.95 | 20 |
| openai/gpt-5.3-codex | 95% | 0.95 | 20 |
| qwen/qwen3.5-122b-a10b | 75% | 0.75 | 20 |
| google/gemini-3-flash-preview | 60% | 0.6 | 20 |
| openai/gpt-4o | 40% | 0.4 | 20 |
| google/gemini-2.5-flash | 35% | 0.35 | 20 |
| google/gemini-2.5-flash-lite | 15% | 0.15 | 20 |
| mistralai/devstral-2512 | 20% | 0.2 | 20 |

*Full results: [seventhChordBenchmarkResults.csv](seventhChordBenchmarkResults.csv)*

## Insights

Higher benchmark scores ≠ better music theory. I've noticed a trend of generational regression where Gemini 3 Pro fails on niche seventh chord logic that Gemini 2.5 Pro handles well. This suggests that while newer models are optimizing for popular benchmarks like MMLU, they may be suffering from overfitting or "reasoning dilution" in specialized domains like music theory math. This serves as a reminder that the "newest" model isn't always the most capable for a specialized task.
