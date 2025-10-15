# Copilot / AI assistant instructions — athena-blockchain

This repository contains a small data-analysis project that queries AWS Athena and downloads results from S3 into pandas (the primary work lives in `athena_blockchain/athena_blockchain.ipynb`). Use these instructions to make edits that fit repository conventions and to run/debug locally.

Key points (short):
- Main entry points: `athena_blockchain/athena_blockchain.ipynb` (notebook analysis) and `main.py` (tiny CLI entry).
- Runtime: Python >= 3.12 (see `pyproject.toml`). Primary deps: `boto3`, `pandas`, `jupyter`.
- External integrations: AWS Athena (via `boto3`) and S3. Queries are executed by the notebook and results are downloaded from S3 using `s3_client.download_file`.

Architecture & why:
- This repo is a lightweight analysis workspace, not a packaged service. The notebook drives the workflow: configure AWS settings (region, database, S3 bucket/prefix), start Athena queries, wait for completion, then download CSV output into pandas for analysis and visualization.
- Decisions visible in code:
  - Config is file-local constants in the notebook (e.g., `AWS_REGION`, `DATABASE_NAME`, `S3_BUCKET_NAME`, `S3_STAGING_PREFIX`). Edits should preserve these names to avoid breaking code that references them.
  - The helper `download_and_load_query_results(client, query_response, download=True)` implements polling and two modes: preview (download=False) and full download (default). Keep changes backward-compatible.

Developer workflows (how to run / debug):
- Install Python 3.12, create a venv, then install project dependencies from `pyproject.toml` (pip/hatch/hatchling as preferred). A minimal pip workflow:
  1. python -m venv .venv
  2. source .venv/bin/activate
  3. pip install -U pip
  4. pip install -r <(python - <<PY
import tomllib,sys
print('\n'.join([d for d in tomllib.loads(open('athena_blockchain/pyproject.toml','rb').read())['project']['dependencies']]))
PY)
  (Or inspect `pyproject.toml` and `pip install boto3 pandas jupyter`)
- Notebook run: open `athena_blockchain/athena_blockchain.ipynb` in Jupyter / VS Code. The notebook expects AWS credentials configured in the environment (~/.aws/credentials or environment variables). Use `aws configure` or set `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`/`AWS_REGION` when running.
- Quick local test without Athena access: mock AWS calls or set `S3_BUCKET_NAME` to a local-test bucket and use `download=False` for preview mode (the notebook will show query metadata without attempting to download CSV).

Project-specific conventions & patterns:
- Keep notebook-first workflow. Prefer lightweight helpers in the notebook rather than moving everything into modules unless adding tests or reusability.
- Configuration names are global constants in the notebook (uppercase). If you add a module that reads config, maintain these variable names or provide a single mapping (dict) to avoid rename churn.
- Use `download_and_load_query_results(...)` for all Athena query polling and downloading. It:
  - Polls the Athena query execution status until terminal state
  - Raises an exception if the query fails
  - When `download=False`, logs stats and returns None (preview mode)
  - When `download=True`, downloads CSV from `s3://{S3_BUCKET_NAME}/{S3_STAGING_PREFIX}/{query_id}.csv` into `athena_result_{query_id}.csv` and loads with pandas

Integration and security notes:
- AWS credentials are required to use the notebook end-to-end. Do NOT hardcode secrets in repository files. Use environment variables or the shared AWS credentials file.
- The notebook uses `s3_client.head_bucket(...)` and `s3_client.download_file(...)` — be mindful of permissions (s3:GetObject / s3:ListBucket, athena:StartQueryExecution, athena:GetQueryExecution).

Files to inspect for patterns/examples:
- `athena_blockchain/athena_blockchain.ipynb` — main analysis notebook and examples of queries and helper functions
- `athena_blockchain/pyproject.toml` — Python version and dependency list
- `athena_blockchain/main.py` — simple CLI entry (keep stable)

When editing code, prefer small, testable changes:
- For new helper modules, add them under `athena_blockchain/` and import from the notebook. Keep names short and document usage in the notebook cells.
- If you change polling timeouts or S3 key format, update the notebook cell that constructs `s3_key = f"{S3_STAGING_PREFIX}/{query_id}.csv"` and document compatibility.

Examples to include in PR descriptions (recommended):
- What config variables were touched (exact names), e.g., `S3_STAGING_PREFIX` or `DATABASE_NAME`.
- If adding a function that interacts with AWS, show the minimal IAM actions required.

Edge cases discovered:
- Queries that return large result sets: currently results are downloaded as CSV into the working directory — avoid storing secrets or large rows in commits.
- Notebook uses polling at 0.5s intervals; long-running queries may cause many API calls — prefer increasing interval for heavy queries.

If something is unclear or you need credentials or a reproducible test harness, ask the repository owner for an AWS test account or provide a small mock harness (we can add a simple `mocks/` module and example cell to run without AWS).

Please review and tell me which sections to expand (examples, IAM policy snippets, mock harness, or a unit-test skeleton using pytest + moto). 
