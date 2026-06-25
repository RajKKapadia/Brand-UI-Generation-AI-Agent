# Brand UI Generation AI Agent

Notebook prototype for generating brand-consistent web UI mockups from visual
brand references. The workflow analyzes a brand image, creates multiple distinct
UI concepts, generates a mockup image, evaluates it, revises the prompt when it
fails, and finalizes the best attempt.

The main implementation lives in `agent.ipynb`. There is no packaged CLI or app
entrypoint yet.

## Current Workflow

The notebook implements this loop:

```text
analyze brand
generate concepts
for attempt in 1..max_attempts:
    generate image
    evaluate image
    if passing:
        finalize
        stop
    revise prompt using feedback
finalize best attempt
```

Main functions in `agent.ipynb`:

- `stitch_images_from_folder(...)`: combines multiple reference screenshots into
  one brand board.
- `brand_analysis_tool(...)`: extracts brand colors, typography, layout
  patterns, UI elements, content patterns, must-preserve details, and things to
  avoid.
- `generate_design_concepts(...)`: creates multiple differentiated UI concepts
  using the same brand system.
- `generate_idea_image(...)`: generates a landscape UI mockup image from a
  concept prompt.
- `evaluate_image(...)`: scores brand fidelity, visual quality, contrast,
  repetition, alignment, proximity, content specificity, and concept
  distinctiveness.
- `revise_image_prompt(...)`: rewrites the image prompt using evaluation
  feedback.
- `run_design_generation(...)`: runs the full end-to-end retry loop and writes
  artifacts to disk.

## Setup

This project is managed with `uv` and requires Python `>=3.12.11`.

```bash
uv sync
```

Create a local `.env` file:

```bash
OPENAI_API_KEY=your_api_key_here
```

The notebook uses OpenAI Responses API calls and GPT Image generation, so running
the generation/evaluation cells will make API requests.

### Notebook Environment

`pyproject.toml` currently pins the runtime libraries, but it does not pin a
Jupyter frontend. Open `agent.ipynb` from VS Code, JupyterLab, or another
Jupyter-capable environment attached to the project virtual environment.

If your environment does not already provide a notebook UI, install one in your
local environment before opening the notebook.

## Inputs

You can provide brand references in two ways.

### Multiple Screenshots

Put screenshot files in `inputs/`, then run the stitching cell:

```python
stitch_images_from_folder(
    folder_path="inputs",
    output_path="output/brand_image.jpg",
    layout="grid",
    padding=20,
)
```

Then use:

```python
brand_guidelines_image = "output/brand_image.jpg"
```

### Single Brand Image

Point directly to one brand reference image:

```python
brand_guidelines_image = "output/guide.png"
```

`inputs/`, `output/`, and `.env*` are intentionally ignored by Git because they
usually contain private screenshots, generated images, and credentials.

## Running The Full Loop

After running the setup/import cells in `agent.ipynb`, set the request and brand
image path:

```python
user_description = "Create a UI landing page for this cooking app"
brand_guidelines_image = "output/guide.png"
```

Run the end-to-end workflow:

```python
run_result = run_design_generation(
    user_description=user_description,
    brand_guidelines_image=brand_guidelines_image,
    run_dir="output/design_run",
    threshold=4.6,
    max_attempts=3,
    num_concepts=3,
    concept_index=0,
)
```

Preview the final image:

```python
display(Image(filename=run_result.final_image_path, width=600))
```

`num_concepts` controls how many unique UI directions are generated. Use
`concept_index` to choose which concept to run through the image/evaluation loop.

## Outputs

`run_design_generation(...)` writes these files under `run_dir`:

- `concept_<n>_attempt_<m>.png`: generated attempt images.
- `final_image.png`: best or first passing image.
- `brand_analysis.json`: structured brand analysis.
- `concepts.json`: generated concept options and prompts.
- `attempts.jsonl`: attempt records with prompts, evaluations, and revisions.
- `final_prompt.txt`: prompt used for the final selected image.
- `final_evaluation.json`: final evaluation scores and feedback.
- `summary.md`: compact run summary.

## Notes

- The reference image is treated as a brand guideline, not as a layout template.
- The evaluator includes `concept_distinctiveness` so an output can fail if it
  looks too similar to the source screenshots.
- `openai-agents` is installed, but the current notebook uses direct OpenAI
  client calls rather than a formal Agents SDK `Agent`/`Runner` setup.
- `flowchart.excalidraw` contains the original high-level workflow sketch.
