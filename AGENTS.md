# AGENTS.md

## What this project is

Solved portfolio for `TOML2026-27-LinAlgPortfolio.pdf`: 8 topics, 29 exercises.

Deliverable: one **executed** Jupyter notebook per topic, `notebooks/Part1…Part8`.
Each exercise keeps the sheet's own split — `Task`, `Explore` (only where the
sheet has one), `Explain` — and the `Explain` answers are part of the deliverable
and live in markdown cells.

Source material (read-only):

- `TOML2026-27-LinAlgPortfolio.pdf` — statements, extracted as `portfolio.txt`
- `TOML2026_27__Review_of_linear_algebra_and_applications_to_data_science.pdf` —
  lecture notes, extracted as `notes.txt`. Every "Example N.M" and "Figure N"
  cited by the sheet is defined there, not in the sheet.
- `Example 4.1` is `A = [[2, 1], [3, 0], [1, 3]]` with singular values
  4.1696 and 2.5719.

The PDF extractions are already in the repo. To regenerate one, use poppler
(a pixi dependency):

```sh
pixi run pdftotext -layout TOML2026-27-LinAlgPortfolio.pdf portfolio.txt
pixi run pdftotext -layout TOML2026_27__Review_of_linear_algebra_and_applications_to_data_science.pdf notes.txt
```

`pdftotext -layout` scatters worked matrices over many lines, so read a whole
page around a hit rather than the grep line alone. The extraction also mangles
some sheet formulas (split lines, missing symbols, broken fractions):
reconstruct each formula from the surrounding prose, not from the line breaks.

## Environment

Use pixi for everything:

```sh
pixi install
pixi run python ...
pixi run jupyter lab
```

Dependencies are pinned in `pixi.toml` (`python` 3.12, `numpy`, `scipy`,
`matplotlib`, `ipywidgets`, plus the notebook and MCP stack). Do not add packages
without asking. There are no `.py` notebook sources: edit `.ipynb` files through
the MCP tools, or with `nbformat` when a tool does not expose the field. Never
rewrite the JSON by hand.

## Jupyter MCP server

`opencode.jsonc` registers the project-local server `jupyter`
(`pixi run python -m jupyter_mcp.server`). Its code lives in `jupyter_mcp/`.
Tools appear as `jupyter_*`; in Code Mode call them through `tools.jupyter.<tool>`.

- Kernel: `start_kernel`, `execute`, `list_kernels`, `restart_kernel`, `stop_kernel`
- Notebook: `create_notebook`, `list_cells`, `read_cell`, `write_cell`,
  `insert_cell`, `delete_cell`, `move_cell`, `execute_notebook`,
  `validate_notebook`, `notebook_summary`

Prefer `execute_notebook` for the final artifact: it runs in a fresh kernel,
saves outputs in place, and reports unexecuted cells or error outputs. Use
`start_kernel` + `execute` only for quick exploration. Call `stop_kernel` when a
persistent kernel is no longer needed.

Notebook paths are relative to the project root; the server refuses paths
outside it. No JupyterLab process is needed by the server; `pixi run jupyter lab`
is only for browsing.

Things that bite:

- `execute_notebook` renumbers the cells 1…n, so it fixes any out-of-order
  `execution_count`, but it also rewrites `language_info`. Write the metadata
  block below **after** the last run.
- A VS Code save can drop `execution_count` from individual cells while keeping
  their outputs; `validate_notebook` then reports them as "not executed". Re-run
  `execute_notebook` instead of trusting the stored outputs.
- `execute_notebook` records the widget state in `metadata.widgets` (including
  the captured `Output`), so a saved panel renders from state. The static figure
  in the preceding cell is still the durable view. If the state goes stale — its
  model ids no longer match the widget-view outputs — delete `metadata.widgets`
  and re-execute.

## Notebook conventions

- File names: `notebooks/Part<N>.ipynb`, `N` being the topic number in the sheet.
- A notebook opens with `# Part N — <topic>` listing its exercises, then one
  `## Setup` code cell with the shared imports and helpers, then one
  `## <ex> — <title>` section per exercise.
- Each notebook is standalone: imports and helpers belong in its own `## Setup`
  cell, and notebooks must not import from one another.
- Keep the sheet's three-part split inside each exercise. Some exercises have no
  `Explore` section — do not invent one.
- **Never write predictions.** Sheet rule 1 ("before running anything, write down
  what you expect to see. Then compare.") is done by hand by the user and never
  belongs in a notebook. Do not add any "what I expect to see", "prediction",
  "expected result" or "before running" section, sentence, bullet or code comment
  — not before a run, not after it, not even to say what turned out as expected.
  The notebook reports only what is computed, and the Explain answers say why
  that is so. If a task text mentions predicting, ignore that part.
- Write the `Explain` answers. Keep them conceptual: one short paragraph per
  exercise, typically 400–600 characters, answering the question directly and
  not restating it.
- Every identity gets a one-line `np.allclose` check (sheet rule 3). Put the
  identities in a dict and print them in a single loop instead of one `print`
  per identity. A deliberately negative control is labelled as such.
- Every figure is paired with the numbers behind it: print them, because a figure
  is not evidence the reader can check. `ALLOW_IMG_OUTPUT` no longer exists; the
  active model still cannot inspect images, so report results from the numbers.
- `scipy.linalg.orth` is itself SVD-based, so for a fixed matrix it returns the
  same frame as `U[:, :r]`. Only the *subspace* is canonical, not the frame —
  phrase it that way rather than claiming the two differ.
- Keep notebooks under a few MB. The `.ipynb` files keep their outputs, so diffs
  are unreadable — review with `list_cells` / `notebook_summary` and re-read a
  cell before deleting or rewriting it.
- Do not quote computed values in `Explain`; they belong in the code and its
  printed output, and in the reference numbers below.

### Length and tone

Every exercise stays as short as its question. `Part4.ipynb` §4.1 is the
calibration: about 420 characters of Task and 320 of Explore. `Explain` is a few
sentences, typically 400–600 characters: answer the question, do not restate the
task, and do not quote computed numbers — they belong in the code and its
printed output. One code cell per Task or Explore step, with a one-line
`# --- Task: ... ---` header. No banner comments, no implementation narration,
no printing objects the figure already shows.

Numbers that appear in two places (figure title and read-out, table and check
line) are formatted once — a local `kap_s` or a small helper — and reused.

### Widgets

Static figures use Matplotlib. Any interactive control uses `ipywidgets`:

- one `widgets.Output()` per panel, redrawn with `clear_output(wait=True)` inside
  `with out:`, then `plt.show()`;
- `continuous_update=False` on every slider, so the panel redraws once per
  release instead of once per pixel;
- `display(widgets.VBox([*controls, out]))`, then call the redraw once;
- unique names per panel (`out11`/`redraw11`, `out53`/`redraw53`), because all
  cells share the notebook globals and a reused name would redirect an earlier
  widget's output;
- a static Matplotlib figure in the preceding cell as the durable view, since a
  widget cell saves only its widget state.

`Part4.ipynb` §4.1 and `Part5.ipynb` §5.3 are the reference implementations;
§5.3 has two independent sliders for the direction of a 3-D vector, and
`Part1.ipynb` §1.1 / §1.2 drive static 3-D views from `ToggleButtons` and a
`FloatSlider`. Plotly is not used anywhere: no `go.Frame` sliders, no
`updatemenus`, and the dependency has been dropped.

### Metadata

`kernelspec` stays the ipykernel default, `python3`, which resolves to the pixi
interpreter (3.12.x). Write this block last, after the final
`execute_notebook`, with `nbformat`:

```python
nb.metadata["kernelspec"] = {
    "display_name": "Python 3 (ipykernel)", "language": "python", "name": "python3"}
nb.metadata["language_info"] = {
    "codemirror_mode": {"name": "ipython", "version": 3},
    "file_extension": ".py", "mimetype": "text/x-python", "name": "python",
    "nbconvert_exporter": "python", "pygments_lexer": "ipython3",
    "version": "3.12.16"}
```

Re-check it after any MCP re-save. A system Python run silently rewrites
`language_info`.

## Topic map

| Part | Title | Exercises | Notebook | Status |
| --- | --- | --- | --- | --- |
| 1 | The four subspaces of a linear map | 1.1–1.3 | `Part1.ipynb` | done, executed, validated |
| 2 | Positive definite matrices | 2.1–2.3 | `Part2.ipynb` | done, executed, validated |
| 3 | Multivariate Gaussian | 3.1–3.4 | `Part3.ipynb` | done, executed, validated |
| 4 | Singular value decomposition | 4.1–4.4 | `Part4.ipynb` | done, executed, validated |
| 5 | Pseudoinverse | 5.1–5.3 | `Part5.ipynb` | done, executed, validated |
| 6 | Least squares for overdetermined systems | 6.1–6.4 | `Part6.ipynb` | done, executed, validated |
| 7 | Minimum-norm solution of underdetermined systems | 7.1–7.3 | `Part7.ipynb` | done, executed, validated |
| 8 | Fourier basis and Fourier matrix | 8.1–8.5 | `Part8.ipynb` | done, executed, validated |

All eight topics of the sheet are covered; create a further part only when the
user asks for it.

## Current phase

All eight notebooks are complete: one executed notebook per topic, sequential
`execution_count`, zero error outputs, `language_info` 3.12.16. Sizes: Part1
1.0 MB, Part2 0.9, Part3 1.4, Part4 1.4, Part5 0.2, Part6 0.4, Part7 0.3, Part8
1.0.

The part counts are 30/12 cells (Part1), 31/13 (Part2), 35/15 (Part3), 26/10
(Part4), 19/6 (Part5), 21/6 (Part6), 17/5 (Part7), 31/13 (Part8), markdown/code.

Widget panels: 1.1 rank buttons and 1.2 slide slider (Part1), 2.1 `a,b,c` and
2.2 eigenvalues (Part2), 3.1 geometry (Part3), 4.1 matrix entries (Part4), 5.3
two-angle 3-D direction (Part5), 7.1 radius (Part7), 8.5 k/sigma/white-noise
(Part8). Part6 has no interactive cell.

The `Explain` answers are short conceptual paragraphs, one per exercise,
typically 400–600 characters. They are the user's to hand-edit from here; keep
their wording where it is already correct and re-read a markdown cell before
changing it.

## Reference numbers

For checking a re-run, not for quoting in `Explain`.

**1** random rank-2 matrix: `sigma = 1.6044, 0.1485, 0`; dims `2+1=3` both
spaces. Rank sweep: 1 gives `1+2`, 2 gives `2+1`, 3 gives `3+0`. Rank-1 example
`A = u v^T` with `v·x = 1.000` for every `c` in `-2…2`. Example 1.12: rank 3,
`sigma = 7.0409, 4.0323, 0.4077`, null vector proportional to `(6,-4,1,-9)`,
entries scaled `(1, -0.6667, 0.1667, -1.5)`; all four 3-column subsets are bases
with dets `-9, -1, -4, -6`.

**2** `S = [[1, 0.5], [0.5, 2]]`: eigenvalues `0.7929, 2.2071`, det `1.75`. The
four cases are PD `(2,0.5,1)`, PSD `(1,1,1)`, ND `(-1,0.5,-2)`, saddle
`(1,2,1)`. Ellipse stages at `theta=30`, `lambda=(2,0.6)`: area `3.7699`; with
`lambda2 = 0` the image collapses to the `q1` axis. 1000 matrices: 500 full-rank
with `lambda_min` in `[1.704e-06, 9.096]`, 500 rank-deficient with `|lambda_min|
<= 3.910e-14`.

**3** `theta=30`, `lambda=(4,1)` gives `Sigma = [[3.25,1.299],[1.299,1.75]]`;
fractions inside `c=1,2,3` are `0.396, 0.864, 0.989` against chi2 `0.3935,
0.8647, 0.9889` at `N=1000`. `lambda1 = lambda2 = 2` leaves the axis
unidentified: mean `|delta| = 51.18` deg. Precision example `Sigma = [[1.5,-0.5],
[-0.5,1.5]]`: eigenvalues `1,2` become `0.5,1`, det `2` becomes `0.5`, peak
density `0.112540`.

**4** Example 4.3: `sigma = 2.9208, 0.6847`, `kappa = 4.2656`, det `2`. Explore:
singular `kappa = inf`, orthogonal rotation `kappa = 1`. Example 4.1:
`sigma = 4.1696, 2.5719`, `lambda(A^T A) = 17.3852, 6.6148`, `AA^T` adds a zero
eigenvalue; `V`/`U` match `eigh` up to signs `(1,-1)` and `(-1,-1,-1)`. Photo
`600x512`, k = 1,5,20,50,100 relative Frobenius `0.5184, 0.3465, 0.1919, 0.1090,
0.0624`; at k=20 photo/checkerboard/noise `0.192/0.000/0.933`, energy kept
`0.963/1.000/0.129`. Example 4.6: `sigma = 6.9092, 2.2380, 1.5015`, norms
`6.9092` (2), `7.4162` (F), `10.6486` (nuclear).

**5** Tall/wide/rank-deficient: left and right roads are `not defined` outside
their case; SVD road matches `pinv` to `2.78e-17`. Example 2.2: `t = (2,1,2)`,
`t_hat = (1.6667,1.6667,0.3333)`, `e = (0.3333,-0.6667,1.6667)`, `||e|| =
1.8257`, `A^T e = 0`.

**6** Line fit: `x_hat = (1.9880, 0.5235)` against true `(2, 0.5)`, `J = 0.1198`,
`cond(A^T A) = 18.0741`. Example 1.14: all four solvers give `(0.9, 0.5)`,
residual `3.834058`. Degree-10 fit: normal equations relative coefficient error
`4.095e-04` against `lstsq` `1.539e-10`; `cond(A) = 2.067e7`,
`cond(A^T A) = 4.268e14`, ratio `0.998701`.

**7** `x_r = (1.2, 0.6)`, `||x_r|| = 1.341641` = distance to the line. Example
1.13: `x_r = (1.5333,-0.0667,-0.3333)`, `x_ns = (1,-2,5)`,
`||x_r||^2 = 37/15`. Gradient descent from `x0 = 0` reaches `8.9e-16` in 400
steps on the slow contraction `0.858857` (`eta = 0.065071`).

**8** `||x|| = ||F x|| = 7.138895` while `||fft(x)|| = 40.383688`; Parseval and
inverse errors `1.6e-14`. Timings over the four largest `N`: slopes `2.062` for
the dense product and `1.004` for the FFT, ratio slope `1.057`. Circulant:
`F C F^H` off-diagonal `2.718e-14`, `N^2 = 1024` multiply-adds against
`3 N log2 N = 480`. Example 6.2: kept bins `[50, 120, 880, 950]`, denoising
error `0.0542`, energy kept `0.7884`.

## Numerical notes

- **Timings need one BLAS thread.** `Part8`'s `## Setup` sets
  `OPENBLAS_NUM_THREADS`, `OMP_NUM_THREADS` and `MKL_NUM_THREADS` to `"1"`
  *before* the first `import numpy` — the thread count is read when the library
  loads. With the default thread pool the spin-waiting threads make small
  timings jitter by an order of magnitude and the log–log slope becomes
  unreadable.
- **Batched products.** A helper that returns one row per parameter gives an
  array of shape `(n, d)`. Multiply it as `points @ A.T`; `A @ points` raises
  `ValueError: matmul ... size n is different from d`.

## Definition of done

A notebook is finished when:

- `execute_notebook` runs it from a fresh kernel, top to bottom, without errors;
- the saved file keeps its outputs and the `execution_count`s are `1…n`;
- every `Explain` question has a written answer;
- every identity or claim that should hold prints `True`; a deliberately negative
  control is clearly labelled as such;
- there is no prediction content anywhere;
- the figures the sheet asks for are present and their key numbers are printed;
  and
- `validate_notebook` reports `ok`, followed by the metadata write above.

One topic per session. Finish with `execute_notebook`, then `validate_notebook`,
then update the Status column and the reference numbers here: this file is the
only handoff between sessions.
