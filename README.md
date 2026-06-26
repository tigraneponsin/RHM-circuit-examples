# Circuit Tracing Library

A static, browsable library of circuit-tracing (attribution-graph) HTML files for transformers trained on the Random Hierarchy Model. Organized by RHM config and stream. Served by GitHub Pages — no build server, no dependencies.

## How it's organized

The **folder tree is the source of truth.** Each circuit lives at:

```
circuits/v{v}_m{m}_s{s}_L{L}/{residual|nores}/<name>.html
```

Example:

```
circuits/
  v16_m4_s2_L3/
    nores/
      sweep_circuit_nores_0.html
    residual/
      ...
  v8_m4_s2_L4/
    ...
```

- `v` vocabulary size (`n = v`), `m` multiplicity, `s` branching (fixed 2), `L` levels.
- Stream folder: `residual` or `nores` (rendered as "non-residual").

`build_index.py` scans this tree, parses the params from the folder names, and writes
`circuits.js`. `index.html` reads `circuits.js` and renders a list grouped by config with
filters for v / m / L / stream. You never hand-edit the index.

## Files

```
index.html        # the gallery UI (static template — don't regenerate)
circuits.js       # GENERATED data: window.CIRCUITS = [...]  — commit this
build_index.py    # scans circuits/ and rewrites circuits.js
.nojekyll         # serve files as-is (don't delete)
circuits/         # the library
README.md
```

## Adding a circuit

1. Put the `.html` in the matching folder, creating it if needed:
   `circuits/v{v}_m{m}_s{s}_L{L}/{residual|nores}/`
2. Commit the new file and push.

That's it — a **GitHub Action** (`.github/workflows/build-index.yml`) runs `build_index.py`
on every push that touches `circuits/`, regenerates `circuits.js`, and commits it back
automatically. You only run the script by hand if you want to preview locally first:

```bash
python3 build_index.py
```

If you drop a file into a folder whose name doesn't match `v{v}_m{m}_s{s}_L{L}`, the script
prints it under "Skipped" so you can fix the name.

## One-time setup (enable GitHub Pages)

1. Push these files to `main`.
2. **Settings → Pages → Source: Deploy from a branch → `main` / `(root)`** → Save.
3. ~1 min later the site is at `https://<username>.github.io/<repo>/`.

## Local preview

```bash
python3 -m http.server 8000     # then open http://localhost:8000
```
Use a server (not `file://`) so the browser loads `circuits.js` reliably.

## Notes / gotchas

- **Rename the example folder.** `v16_m4_s2_L3` was inferred from the file's tree (v=16, s=2, L=3);
  `m=4` is a **placeholder** — set it to the real multiplicity.
- **`.nojekyll` matters.** Without it GitHub Pages runs Jekyll and ignores files/folders starting
  with `_` or `.`.
- **An `index.html` at the root is required** — Pages 404s on a bare directory; it won't auto-list.
- **Plotly CDN is fine.** The circuit files pull `cdn.plot.ly` over HTTPS, which works on Pages.
  Each is ~1.6 MB, well under the 100 MB per-file limit.
- **The GitHub Action needs write access.** It commits `circuits.js` back to the repo. If the push
  step fails with a permissions error, go to **Settings → Actions → General → Workflow permissions**
  and select **Read and write permissions**.
