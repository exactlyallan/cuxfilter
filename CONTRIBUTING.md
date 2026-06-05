# Contributing

This repository now houses the `accelerated-visual-analytics` skill and
historical cuxfilter notebook examples. It does not build or publish the old
cuxfilter Python package.

## What to Change

- Skill behavior: edit `skills/accelerated-visual-analytics/SKILL.md`.
- Deep reference guidance: edit files under
  `skills/accelerated-visual-analytics/references/`.
- Generated artifact templates: edit files under
  `skills/accelerated-visual-analytics/templates/`.
- Evals and fixtures: edit files under
  `skills/accelerated-visual-analytics/evals/`.
- Historical notebook migration examples: edit or add files under `examples/`.

## Validation

Run lightweight checks before opening a PR:

```bash
python3 -m json.tool skills/accelerated-visual-analytics/evals/evals.json >/tmp/evals.json.checked
python3 -m json.tool skills/accelerated-visual-analytics/evals/files/rapids-viz-guide-base.ipynb >/tmp/base.ipynb.checked
python3 -m json.tool skills/accelerated-visual-analytics/examples/rapids-viz-guide-successful.ipynb >/tmp/success.ipynb.checked
git diff --check
```

If the skill eval harness is available, also run the repository eval suite
against `skills/accelerated-visual-analytics/`.

## Guidelines

- Do not reintroduce `cuxfilter` as a runtime dependency.
- Keep historical `cuxfilter` references only when they are clearly migration
  input or sunset context.
- Keep generated dashboard guidance backend-aware: CPU/GPU boundaries, data
  movement, and table/display caps should be explicit.
- Keep examples and eval fixtures output-free unless an output is intentionally
  part of the fixture.
