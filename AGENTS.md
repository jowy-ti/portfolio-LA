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
`notebooks/Part3.ipynb` are also complete. `notebooks/Part4.ipynb` implements
exercises 4.1–4.4, `notebooks/Part5.ipynb` implements 5.1–5.3,
`notebooks/Part6.ipynb` implements 6.1–6.4, `notebooks/Part7.ipynb` implements
7.1–7.3 and `notebooks/Part8.ipynb` implements 8.1–8.5. All eight topics of the
sheet are covered; create a further part only when the user asks for it.

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
- Static figures use Matplotlib. Any interactive control (sliders, buttons that
  change a parameter) uses `ipywidgets`, redrawing by replacing its outputs in
  place (see *Way of solving exercises*), with a static Matplotlib figure in the
  preceding cell as the durable view. Do not build sliders with Plotly
  (`go.Frame` sliders or `updatemenus`). Part4 §4.1 and Part5 §5.3 are the
  reference implementations of this pattern; §5.3 has two independent sliders
  for the direction of a 3-D vector.
  `pixi.toml` declares `ipywidgets` and `.vscode/settings.json` configures widget
  script sources for interactive cells.

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

### Numerical notes

- **Timings need one BLAS thread.** Set `OPENBLAS_NUM_THREADS`, `OMP_NUM_THREADS`
  and `MKL_NUM_THREADS` to `"1"` in the `## Setup` cell, *before* the first
  `import numpy` — the thread count is read when the library loads. With the
  default thread pool the spin-waiting threads make small timings jitter by an
  order of magnitude (the same product measured 1.6e-6 s and 1.2e-4 s on
  different runs), and the log–log slope of a cost curve becomes unreadable.
- **Batched products.** A helper that returns one row per parameter gives an
  array of shape `(n, d)`. Multiply it as `points @ A.T`; `A @ points` raises
  `ValueError: matmul ... size n is different from d`.

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
That call inserts a placeholder cell 0 ("New Notebook Created by Jupyter MCP
Server") — overwrite it with the `# Part N — <topic>` title, or delete it.

The MCP tools do not expose notebook metadata, so a newly created notebook has an
empty `kernelspec`. Fix it through the Jupyter server's own contents API — a
server-side save, not a hand-edit of the `.ipynb` JSON:

```python
import json, urllib.request

BASE = "http://127.0.0.1:8888/api/contents/notebooks/"
AUTH = {"Authorization": "token portfolio-la-jupyter-dev-token"}

def get(name):
    return json.loads(urllib.request.urlopen(
        urllib.request.Request(BASE + name, headers=AUTH)).read())

def put(name, model):
    body = json.dumps({"content": model["content"], "format": "json",
                       "type": "notebook", "metadata": {}}).encode()
    req = urllib.request.Request(BASE + name, data=body, method="PUT",
                                 headers={**AUTH, "Content-Type": "application/json"})
    return json.loads(urllib.request.urlopen(req).read())

src, dst = get("Part6.ipynb"), get("Part8.ipynb")
for key in ("kernelspec", "language_info"):
    dst["content"]["metadata"][key] = src["content"]["metadata"][key]
put("Part8.ipynb", dst)
```

Copy `kernelspec` and `language_info` from `Part1.ipynb` or `Part6.ipynb`; their
metadata display name `portfolio-LA:mcp (3.12.15)` is what pixi calls the
environment and is *not* a JupyterLab kernelspec label. **This metadata write
must be the last write to the notebook**: the MCP server holds its own in-memory
model and re-saves it with the empty `kernelspec` on the next edit or re-run, so
re-run the notebook first and write the metadata afterwards. `Part2.ipynb` and
`Part3.ipynb` still have empty `kernelspec` metadata and need the same fix.
