# Project instructions

## PDFs

The active model does not accept PDF input, so `read` on a `.pdf` fails. Convert
first, then read the text:

```bash
pixi run pdftotext -layout <file>.pdf <file>.txt
```

`poppler` is a pixi dependency, so `pdftotext` (and `pdftoppm` for page images)
are available through `pixi run`. Do not try to `read` a PDF directly or attach
one to a prompt (attachments only forward text and PNG/JPEG/GIF/WebP).

## Jupyter notebooks

Notebooks are run through the `jupyter` MCP server, which talks to a local
JupyterLab over stdio.

1. Start JupyterLab first (the MCP server connects at startup), rooted at the
   project directory:

   ```bash
   pixi run -e mcp jupyter lab --port 8888 --ip 127.0.0.1 --IdentityProvider.token portfolio-la-jupyter-dev-token
   ```

2. The MCP server is configured in `opencode.jsonc` and runs from
   `.pixi/envs/mcp/bin/jupyter-mcp-server`. Install/reinstall the environment
   with `pixi install -e mcp`.

3. Under Code Mode the tools are called as `tools.jupyter.<tool>(...)` through
   `execute`. Typical flow: `use_notebook` to activate one, then
   `read_notebook`, `insert_cell`, `execute_cell`.

`ALLOW_IMG_OUTPUT` is `false` because the model is not multimodal; prefer
numeric or textual output over plots when reporting results.

## Portfolio notebooks

The exercises come from `TOML2026-27-LinAlgPortfolio.pdf`; `portfolio.txt` is the
extracted text. The notebooks are organised **one notebook per topic**, not per
exercise:

| notebook | topic | exercises |
|---|---|---|
| `notebooks/Part1.ipynb` | 1 — The four subspaces of a linear map | 1.1, 1.2, 1.3 |
| `notebooks/Part2.ipynb` | 2 — Positive definite matrices | 2.1, 2.2, 2.3 |
| … | … | … |

The naming is `notebooks/Part<N>.ipynb`, `N` being the topic number in the sheet.
A notebook opens with `# Part N — <topic>` listing its exercises, then a `## Setup`
cell holding the shared imports, then one `## <ex> — <title>` section per exercise.

### Rules for exercise notebooks

- **Never write predictions.** Sheet rule 1 ("before running anything, write down
  what you expect to see. Then compare.") is done by hand by the user and never
  belongs in a notebook. Do not add any "what I expect to see", "prediction",
  "expected result" or "before running" section, sentence, bullet or code comment —
  not before a run, not after it, not even to say what turned out as expected. The
  notebook reports only what is computed, and the Explain answers say why that is
  so. If a task text mentions predicting, ignore that part.
- Keep the sheet's own three-part split inside each exercise: **Task** (compute and
  plot), **Explore** (sliders, extra cases), **Explain** (the written answer, in a
  final section of that exercise). Some exercises have no Explore section — do not
  invent one.
- Do write the **Explain** answers.
- Every identity gets a one-line `np.allclose` check (sheet rule 3), and each figure
  is paired with the numbers behind it.
- `scipy.linalg.orth` is itself SVD-based, so for a fixed matrix it returns the same
  frame as `U[:, :r]`. Only the *subspace* is canonical, not the frame — phrase it
  that way rather than claiming the two differ.

### MCP tooling notes

Two things bite when writing cells through `tools.jupyter.*` inside `execute`:

- **Backticks.** A markdown cell written as a JS template literal cannot contain a
  literal backtick. Write `@@` instead and substitute, e.g.
  `src.split("@@").join(String.fromCharCode(96))`.
- **Markdown arrays.** Building a markdown cell from a JS *array* of quoted strings
  has been observed to fail the Code Mode parser (`Unexpected token` at the
  `].join(...)` line). Use one template literal per markdown cell.

After editing a cell, re-execute it; and when several cells were touched, restart
and re-run the whole notebook top-to-bottom so `execution_count` stays in order.

### Known papercut

`notebooks/Part1.ipynb` has an empty `kernelspec` in its metadata (`name: ""`), so
JupyterLab may ask which kernel to use. Picking Python 3 from the kernel selector
once and saving repairs it. The MCP tools cannot set notebook metadata.
