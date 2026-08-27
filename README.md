# Candidate focal new hires and their presence in USPTO

This directory is a self-contained publication snapshot of three marimo reports:

1. `C01_UniverseCandidateFNH.py` describes the universe of candidate focal new hires.
2. `C02_MatchRatesToInventorData.py` reports inventor-linkage rates.
3. `C03_MatchedCandidateFNH.py` describes inventor-matched candidate focal new hires.

The notebooks read only aggregate Parquet tables under `public/data/`. The `public/`
location follows marimo's WebAssembly convention: marimo copies this directory into
the exported site, and `mo.notebook_location()` resolves the same paths locally and
in the browser.

## Bundle layout

```text
.
|-- C01_UniverseCandidateFNH.py
|-- C02_MatchRatesToInventorData.py
|-- C03_MatchedCandidateFNH.py
|-- layouts/
|-- public/data/
|-- .github/workflows/pages.yml
|-- index.html
`-- requirements.txt
```

The three data directories contain 148 Parquet files totaling 46,697,400 bytes.
They are publication inputs, not the individual-level Revelio or inventor data.
The source data-preparation scripts remain in the TalentDiscovery repository.

## Run locally

Run each notebook through the project's `Talent` conda environment from this directory:

```powershell
conda run -s -n Talent marimo edit C01_UniverseCandidateFNH.py
conda run -s -n Talent marimo edit C02_MatchRatesToInventorData.py
conda run -s -n Talent marimo edit C03_MatchedCandidateFNH.py
```

Do not regenerate the aggregate tables from this publication directory. Regenerate them
with the corresponding `C01_`, `C02_`, and `C03_DataPrep_*_AggTables.py` scripts in
`codes/B01_ConstructAnalysisSample`, then refresh the publication snapshot.

## Publish to GitHub Pages with GitHub Actions

The included workflow exports each notebook as a read-only, interactive WebAssembly page.
No Python server is needed after deployment.

1. Confirm that all aggregate tables and category labels are cleared for public release.
   GitHub Pages sites are public even in some private-repository configurations.
2. Clone `https://github.com/wenzhi-econ/candidate-new-hires` and copy the *contents* of
   this directory to the repository root. In particular, `.github/workflows/pages.yml`
   must be at the repository root, not below another folder.
3. Check `git status` before committing. Confirm that all 148 files under `public/data/`
   and all three JSON files under `layouts/` are tracked. The TalentDiscovery repository
   has a narrow ignore-rule exception for this bundle; the publication repository must
   likewise retain these files.
4. Commit and push the bundle to the publication repository's `main` branch. If its default
   branch has another name, change the branch under `on.push.branches` in
   `.github/workflows/pages.yml`.
5. On GitHub, open **Settings > Pages** and select **GitHub Actions** as the source under
   **Build and deployment**.
6. Open the **Actions** tab and inspect the `Build and deploy marimo reports` run. The
   workflow installs the pinned Python environment and `uv` import resolver, exports the
   three notebooks, uploads `_site`, and deploys it with GitHub's official Pages actions.
7. When the run succeeds, open
   `https://wenzhi-econ.github.io/candidate-new-hires/`. Test all three links, change each
   main selector at least once, and inspect the browser console for missing packages or data.

The workflow can also be run manually from **Actions > Build and deploy marimo reports >
Run workflow**.

If export fails with `uv must be installed to resolve local imports`, verify that the
`Install uv for marimo's import resolver` step appears before the export step. The bundled
workflow already includes this fix.

If a deployed notebook reports `BadGzipFile: Not a gzipped file (b'PA')`, do not pass the
HTTP URL directly to `pandas.read_parquet`. In WebAssembly, the browser can decompress a
GitHub Pages response while preserving its `Content-Encoding: gzip` header. Pandas then
tries to decompress the already-decoded Parquet bytes a second time. The publication
notebooks avoid this by fetching HTTP data as bytes and passing an in-memory `BytesIO`
stream to `pandas.read_parquet`; local paths still go directly to pandas.

If an Altair output reports that an out-of-range `nan` value is not JSON compliant, pass
only the columns used by the chart and convert any intentionally missing plotted values to
`None`. WebAssembly serializes the full DataFrame supplied to Altair, including unused
columns.

## Preview the exported site before publishing

From this directory, export the notebooks with the same marimo version used by the workflow.
For example, in PowerShell:

```powershell
New-Item -ItemType Directory -Force -Path _site | Out-Null
$notebooks = @(
    "C01_UniverseCandidateFNH.py",
    "C02_MatchRatesToInventorData.py",
    "C03_MatchedCandidateFNH.py"
)
foreach ($notebook in $notebooks) {
    $stem = [System.IO.Path]::GetFileNameWithoutExtension($notebook)
    conda run -s -n Talent marimo export html-wasm $notebook `
        --output "_site/$stem.html" --mode run --no-show-code --force
}
Copy-Item -LiteralPath index.html -Destination _site/index.html -Force
conda run -s -n Talent python -m http.server 8000 --directory _site
```

Then open `http://localhost:8000`. WebAssembly exports must be served over HTTP; opening
the HTML files directly from disk is not a valid test.

## Publication cautions

- Publishing makes every file under `public/` downloadable. Absence of person identifiers
  should be rechecked after every data refresh, and data-license clearance remains a human
  decision.
- Browser execution uses Pyodide's WebAssembly-compatible package builds, which can differ
  from the pinned local package versions. Revalidate displayed totals, charts, and controls
  after dependency or marimo upgrades.
- The site carries roughly 46.7 MB of aggregate data plus the Python runtime and plotting
  packages. First load can therefore be noticeably slower than a conventional static page.

References: [marimo's GitHub publishing guide](https://docs.marimo.io/guides/publishing/github/),
[marimo's WebAssembly export guide](https://docs.marimo.io/guides/exporting/webassembly_html/),
and [GitHub's custom Pages workflow guide](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).
