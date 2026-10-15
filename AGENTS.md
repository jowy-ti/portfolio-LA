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
