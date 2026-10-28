# Contributing to Smart CSV Toolkit

Thanks for considering a contribution! This is a Streamlit app for cleaning
CSVs, inferring column metadata, and generating plots, with an optional
LLM-assisted cleaning flow. This doc covers how to get it running locally,
how to test your change, and how to submit it.

## 1. Dev setup

**Requirements:** Python 3.11 (matches CI in `.github/workflows/ci-cd.yml`).

```bash
# Clone and enter the repo
git clone https://github.com/iamvisheshsrivastava/Smart_CSV_Toolkit.git
cd Smart_CSV_Toolkit

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

If you're only working on core cleaning logic (`pipeline_logic.py`,
`metadata_inference.py`) and don't need the heavier LLM/ML extras, the
production-trimmed `requirements-prod.txt` installs faster, but prefer
`requirements.txt` for full local development since the test suite exercises
LLM-related code paths too.

### Environment variables (optional)

LLM-powered features (AI cleaning suggestions, custom LLM cleaning, webpage
summarization) read API keys from a `.env` file in the project root (see
`python-dotenv` usage in `app.py`). These are optional — the core CSV
cleaning pipeline, metadata inference, and most of the UI work fine without
any keys configured. Only set up a `.env` if you're working on an
LLM-related feature.

## 2. Running the app locally

```bash
streamlit run app.py
```

This opens the app at `http://localhost:8501`. The sample file
`browser_sample.csv` is a convenient small CSV to upload while testing.

- **Tab 1 – CSV Cleaner:** pick steps from `table_steps.json`, run the
  pipeline (`pipeline_logic.py`), download the cleaned CSV.
- **Tab 2 – Metadata Inspector:** upload/merge CSVs, infer column types
  (`metadata_inference.py`), view auto-generated plots.

## 3. Running tests

Tests live under `tests/`. Run the full suite with:

```bash
pytest
```

Run a single file or test while iterating:

```bash
pytest tests/test_pipeline_logic.py -v
pytest tests/test_pipeline_logic.py::test_impute_missing_mean -v
```

### End-to-end tests

`tests/e2e/` contains Playwright-driven tests that exercise the running
Streamlit app in a browser (`test_streamlit_app.py`). These need a browser
binary installed once:

```bash
playwright install chromium
```

They're slower and more environment-sensitive than the unit tests in
`tests/`, so unit tests are the faster feedback loop for most changes —
run the e2e tests when your change touches UI flow, not just pipeline logic.

### CI

Every push/PR to `main` runs `pip install -r requirements.txt` followed by a
syntax check (`python -m py_compile`) on the core modules — see
`.github/workflows/ci-cd.yml`. Running `pytest` locally before opening a PR
is strongly encouraged even though CI doesn't currently run it, since it's
the fastest way to catch a regression before review.

## 4. Code style

- Match the existing style in the file you're editing (type hints on
  function signatures, docstrings on non-trivial functions — see
  `pipeline_logic.py` for the pattern this repo follows).
- Keep pipeline operations in `pipeline_logic.py` pure with respect to their
  `(data, params)` contract: raise a clear `ValueError`/`RuntimeError` with
  an actionable message on bad input rather than letting a raw
  pandas/sklearn exception bubble up to the Streamlit UI.
- If you add or change a pipeline step, keep `table_steps.json`'s
  `description` fields accurate — they're shown directly as UI help text.

## 5. Pull request process

1. **Open an issue first** for anything beyond a trivial fix, so we can
   agree on the approach before you invest time. Issues labeled
   [`good first issue`](https://github.com/iamvisheshsrivastava/Smart_CSV_Toolkit/labels/good%20first%20issue)
   are a good place to start if you're new to the repo.
2. **Branch from `main`** and keep the PR focused on one change — small,
   reviewable PRs get merged faster.
3. **Add/update tests** for any behavior change in `pipeline_logic.py` or
   `metadata_inference.py`. A bug fix should include a test that fails
   without the fix.
4. **Run `pytest`** locally and make sure it passes before opening the PR.
5. **Describe the change** in the PR: what it does, why, and how you
   verified it (test output, or a screenshot/GIF for UI changes).
6. A maintainer will review and may ask for changes — please keep the
   conversation in the PR thread so the history stays useful for others.

## 6. Reporting bugs / requesting features

Open a GitHub issue. For bugs, include: what you did, what you expected,
what happened instead, and (if relevant) a minimal CSV or recipe JSON that
reproduces it. For security issues, please see any security policy in the
repo before filing publicly, or contact the maintainer directly.
