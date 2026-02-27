# Manim AI Video Generator

Manim AI turns a text request into a polished infographic style animation. It generates Manim code, renders a first pass, fixes any render errors, and then runs a two stage video review loop. If the video meets the guidelines, it renders a final high resolution version.

![Manim AI Workflow Running in n8n](../assets/workflow-manim-ai.gif)

The goal is simple: AI generated infographic videos are still very bad even with state of the art models like Google Veo 3.1. This workflow shows how to make the results usable by combining code generation, deterministic rendering, and strict review gates.

## Why This Beats Google Veo 3.1

### Example: Flynn Effect Infographic

**Prompt:** Make a graph that British children's average scores on the Raven's Progressive Matrices test rose by 14 IQ points from 1942 to 2008. The graph visualizes the Flynn effect, it's the substantial and long-sustained increase in both fluid and crystallized intelligence test scores that were measured in many parts of the world over the 20th century, named after researcher James Flynn (1934–2020). Use animations and make it look fancy.

#### n8n Workflow Result

![n8n Manim AI Generated Animation](../assets/manim-ai-iq-animation.gif)

#### Google Veo 3.1 Result

![Google Veo 3.1 Generated Animation](../assets/veo3-1-iq-animation.gif)

### Results Comparison

| Aspect | n8n Workflow | Veo 3.1 |
|--------|--------------|--------|
| **Text Rendering** | Excellent—crisp, readable text perfect for infographics | Poor—text is often gibberish |
| **Factual Accuracy** | Highly reliable—deterministic rendering ensures accuracy | Variable—may hallucinate or distort facts |
| **Cost per Second** | **$0.023/sec** (11 seconds = $0.25) | $0.10–$0.15/sec |
| **Cost Advantage** | — | **77–85% cheaper** |
| **Quality** | Superior text and composition for data visualization | Lower quality for technical content |

**Verdict:** While state-of-the-art AI video generators like Veo 3.1 represent impressive technology, they fail at infographics due to gibberish text rendering. This workflow leverages Manim's deterministic rendering combined with AI-powered iteration to deliver professional results at a fraction of the cost. My workflow cost $0.2530065 to generate 11 seconds of video which is $0.023000591 per second. Veo on the other hand uses $0.10–$0.15 per second of video. That's 77% to 85% cheaper with better results.

## What It Does

- Takes a user prompt and expands it with curated Manim examples
- Produces structured Manim code plus the class name
- Renders the scene at a low resolution for fast iteration
- Fixes render errors by feeding the stack trace back to the code generator
- Runs a two model video audit to decide if the output is acceptable
- Rewrites the scene if it fails the guidelines
- Renders a final pass at higher resolution and frame rate when accepted

## Workflow File

- Main workflow: [Manim AI.json](Manim%20AI.json)

## Inputs

This workflow expects one input when executed as a sub workflow:

- `query` (string): the animation request

## Output Schema

The generator returns JSON only. Each response matches this schema:

```json
{
  "code": "from manim import *\n\nclass Example(Scene):\n    def construct(self):\n        ...\n",
  "class_name": "Example",
  "explanation": "Short description of the animation."
}
```

## Video Guidelines Enforced

- White background with black text and strong contrast
- 16:9 aspect ratio with content kept in frame
- Text is readable and does not overlap other text
- Arial font
- Minimal text, simple composition
- Color used sparingly (prefer blues)

## How It Works

1. Prompt expansion
   - The user prompt is combined with Manim example snippets to prime the generator.
2. Code generation
   - A model returns `code`, `class_name`, and `explanation` using a structured output parser.
3. Render loop for correctness
   - The render workflow runs Manim at 854x480, 30 fps.
   - If Manim throws an error, the stack trace, prompt, and last code are sent back to the generator.
   - The generator fixes the script and retries until it renders cleanly.
4. Two stage review loop for quality
   - A multimodal model describes what is actually visible in the video and lists guideline violations.
   - A second LLM formats that report into strict JSON so the system can reliably parse it.
   - This split avoids context overload and prevents the video focused model from also juggling JSON constraints.
   - If rejected, the fix request is sent back to the code generator.
5. Final render
   - Once accepted, the workflow renders again at 1280x720, 30 fps.

## Why Two LLMs For Review

One model is best at visual understanding. Another is best at turning that understanding into strict structured output. Keeping those roles separate avoids context garbage and reduces forced formatting mistakes. In practice, this improves quality and acceptance consistency.

## Technical Details

- Trigger: Execute Workflow Trigger with a single `query` input
- Prompting: A Set node injects Manim example snippets before the user prompt
- Generation model: OpenRouter Chat Model with a structured output parser
- Render pipeline: Child workflow named "Create Manim Animation"
  - Draft render: 854x480, 30 fps
  - Final render: 1280x720, 30 fps
- Error handling: On render error, the workflow loops with the stack trace and previous code
- Review pipeline:
  - Video analysis: Google Gemini 2.5 Flash via `Analyze video`
  - Report formatting: Second LLM with a JSON schema parser
- Decision gate: If accepted, proceed to final render; otherwise loop back to the generator

## Requirements

- OpenRouter credentials configured in n8n
- Google Gemini credentials configured in n8n
- A render workflow named "Create Manim Animation"
- Manim available in the render environment

## Running The Workflow

1. Execute the workflow or call it as a sub workflow.
2. Provide a `query` describing the animation.
3. The workflow renders and evaluates the video.
4. If rejected, it auto corrects and re renders.

## Notes

- The review step is intentionally strict and may reject valid renders for readability.
- To change aspect ratio or resolution, edit the render workflow call nodes.
