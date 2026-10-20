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

Two PDFs matter:

- `TOML2026-27-LinAlgPortfolio.pdf` — the exercise sheet; extracted as
  `portfolio.txt`.
- `TOML2026_27__Review_of_linear_algebra_and_applications_to_data_science.pdf` —
  the lecture notes. Every "Example N.M" and "Figure N" cited by the sheet is
  defined there, not in the sheet: when a task says "the matrix in Example 4.1",
  search this one. Extracted as `notes.txt`:

  ```bash
  pixi run pdftotext -layout \
    TOML2026_27__Review_of_linear_algebra_and_applications_to_data_science.pdf notes.txt
  ```

  `pdftotext -layout` scatters worked matrices over many lines, so read a whole
  page around the hit rather than the grep line alone. Example 4.1 is
  A = [[2, 1], [3, 0], [1, 3]] with σ = 4.169, 2.572.

## Jupyter notebooks

Notebooks are run through the `jupyter` MCP server. opencode launches that server
over stdio; the server itself talks to a local JupyterLab over HTTP at
`JUPYTER_URL`.

1. Start JupyterLab first (the MCP server connects at startup), rooted at the
   project directory — the script passes no `--notebook-dir`, so run it from the
   repository root:

   ```bash
   ./jupyter_server.sh
   ```

   which is equivalent to:

   ```bash
   pixi run -e mcp jupyter lab --port 8888 --ip 127.0.0.1 --IdentityProvider.token portfolio-la-jupyter-dev-token
   ```

2. The MCP server is configured in `opencode.jsonc` and runs from
   `.pixi/envs/mcp/bin/jupyter-mcp-server`. Install/reinstall the environment
   with `pixi install -e mcp`.

3. Under Code Mode the tools are called as `tools.jupyter.<tool>(...)` through
   `execute`. Typical flow: `use_notebook` to activate one, then
   `read_notebook`, `insert_cell`, `execute_cell`.

4. All notebook changes go through the MCP tools; never write `.ipynb` JSON
   directly or commit an empty placeholder notebook. If the MCP connection
   fails, start JupyterLab with `./jupyter_server.sh` and retry; do not fall
   back to editing the file on disk.

`ALLOW_IMG_OUTPUT` is `false` because the model is not multimodal. The exercises
still require the plots the sheet asks for, but a figure is not evidence the
agent can inspect: print the numbers behind every figure and report results from
those numbers.

## Portfolio notebooks

The exercises come from `TOML2026-27-LinAlgPortfolio.pdf`; `portfolio.txt` is the
extracted text. The extraction mangles some formulas (split lines, missing
symbols, broken fractions), so reconstruct each formula from the surrounding
prose instead of trusting every line break in `portfolio.txt`.

`notebooks/Part1.ipynb` is the completed **reference implementation** for topic 1
and the model for structure, tone and code style. `notebooks/Part2.ipynb` and
`notebooks/Part3.ipynb` are also complete. `notebooks/Part4.ipynb` exists and
implements exercises 4.1–4.3; exercise 4.4 is added when the user asks for it.
Create the remaining Parts 5–8 only when the user asks for them.

The notebooks are organised **one notebook per topic**, not per exercise:

| notebook | topic | exercises |
|---|---|---|
| `notebooks/Part1.ipynb` | 1 — The four subspaces of a linear map | 1.1, 1.2, 1.3 |
| `notebooks/Part2.ipynb` | 2 — Positive definite matrices | 2.1, 2.2, 2.3 |
| `notebooks/Part3.ipynb` | 3 — Multivariate Gaussian | 3.1, 3.2, 3.3, 3.4 |
| `notebooks/Part4.ipynb` | 4 — Singular value decomposition | 4.1, 4.2, 4.3, 4.4 |
| `notebooks/Part5.ipynb` | 5 — Pseudoinverse | 5.1, 5.2, 5.3 |
| `notebooks/Part6.ipynb` | 6 — Least squares for overdetermined systems | 6.1, 6.2, 6.3, 6.4 |
| `notebooks/Part7.ipynb` | 7 — Minimum-norm solution of underdetermined systems | 7.1, 7.2, 7.3 |
| `notebooks/Part8.ipynb` | 8 — Fourier basis and Fourier matrix | 8.1, 8.2, 8.3, 8.4, 8.5 |

The naming is `notebooks/Part<N>.ipynb`, `N` being the topic number in the sheet.
A notebook opens with `# Part N — <topic>` listing its exercises, then a `## Setup`
cell holding the shared imports, then one `## <ex> — <title>` section per exercise.

Each notebook is standalone: imports and helpers belong in its own `## Setup`
cell, and notebooks must not import from one another.

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
- Static figures use Matplotlib. Interactive figures follow Part1's pattern:
  self-contained Plotly (`go.Frame`, Plotly sliders, `updatemenus`) rather than
  ipywidgets, so they render without a live widget manager. Do not switch to
  ipywidgets unless the user asks. Part4's 4.1 entry sliders are the one exception,
  added at the user's request: four matrix entries need four independent controls,
  which Plotly sliders cannot express, so that cell uses `ipywidgets`, redrawing by
  replacing its outputs in place (see *Way of solving exercises*), with the static
  Matplotlib four-panel figure in the preceding cell as the durable view.
  `pixi.toml` declares `ipywidgets` and `.vscode/settings.json` configures widget
  script sources for that case.

### Way of solving exercises

Every exercise stays as short as its question. `Part4.ipynb` §4.1 is the lean
model and the calibration: about 420 characters of Task, 320 of Explore, 1400 of
Explain.

- **Text.** Task and Explore state what to compute and what to plot, nothing more.
  Explain answers one short paragraph per question — 4.1 answers four in ~250
  words — and never re-narrates a run or restates an earlier paragraph.
- **Code.** One cell per Task or Explore step, with a one-line
  `# --- Task: ... ---` header. No banner comments, no implementation narration,
  no printing objects the figure already shows.
- **Format once.** Numbers that appear in two places (figure title and read-out,
  table and check line) are formatted once — a local `kap_s` or a small helper —
  and reused.
- **Checks in one block.** Put the identities in a dict and print them in a single
  loop instead of one `print` per identity; each one is still an `np.allclose`
  check.
- **Interactive cells update in place.** `display(obj, display_id=...)` once, then
  `DisplayHandle.update(...)` for every change, so the output is replaced rather
  than cleared and refilled. Do not drive redraws from `widgets.interactive_output`
  or a per-update cleared `Output`: the block blanks out on every slider move.

### Definition of done

A notebook is finished when:

- it runs from a fresh kernel, top to bottom, without errors;
- the saved file keeps its outputs and the `execution_count`s are sequential;
- every Explain question has a written answer;
- every identity or claim that should hold prints `True`; a deliberately negative
  control is clearly labelled as such;
- there is no prediction content anywhere; and
- the figures the sheet asks for are present and their key numbers are printed.

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

Create notebooks with `use_notebook(mode="create", ...)`, not by writing the file.
The MCP tools do not expose notebook metadata, so a newly created notebook has an
empty `kernelspec`; if JupyterLab makes you pick a kernel, choose the only entry it
offers, **Python 3 (ipykernel)** (kernel name `python3`, the ipykernel of the `mcp`
environment), once and save; never patch the `.ipynb` JSON by hand. `Part1.ipynb`
already has the correct kernel recorded — its metadata display name
`portfolio-LA:mcp (3.12.15)` is what pixi calls the environment and is *not* a
JupyterLab kernelspec label; `Part2.ipynb` and `Part3.ipynb` still have empty
`kernelspec` metadata and need the kernel selected once in JupyterLab and the
notebook saved.
