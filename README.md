# n8n Showcase

A hand-picked collection of my n8n workflows, curated from 50+ complex workflows I have built.

## What's Inside

- [Music Theory Benchmark](benchmark/) - Tests LLM music theory knowledge with seventh chord calculations. Exposes how even advanced models fail at music fundamentals.
- [Manim AI Video Generator](manim/) - Turns text requests into polished infographic animations. Beats Google Veo 3.1 on text rendering, accuracy, and cost (77–85% cheaper).

## Coming Soon

- **Personal Deep Research AI Crawler** - Advanced web research automation for extracting and synthesizing information with higher accuracy than generic crawlers.
- **Devin Implementation** - GitHub agent that autonomously solves issues, creates pull requests, and handles complex development tasks.
- **Productivity Workflows Suite** - Email responder, calendar manager, advanced news aggregator, and other personal automation tools.
- ...and more

## Notes

- Workflows are provided as-is and requires credentials as well as environment-specific configuration to work.

## Prompting Insight

Forcing a model to output "Strict JSON and nothing else" is a trap that actually makes it fail more often because it can't show its "scratch work" before giving the final answer. When you let the AI talk freely and explain its logic first, you are giving it the space to think step-by-step, which leads to much better results for both math and creative tasks. Do not waste your prompt space on strict rules and "don't do this" constraints; that just makes the AI too narrow-minded and distracted from the actual problem. Instead, keep your system prompts as short and simple as possible, and just use your own code to find and clean up the data after the AI is done talking.
