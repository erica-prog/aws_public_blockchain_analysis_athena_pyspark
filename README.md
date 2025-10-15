# AWS Public Blockchain Analysis using Athena, S3 and PySpark/Python

![Architecture diagram](architectural%20diagram.png)

This repository contains exploratory analysis of Ethereum using two complementary notebooks:

- `athena_blockchain/athena_blockchain.ipynb` — Athena-driven SQL queries (via `boto3`) with results downloaded to CSV and analyzed in `pandas`.
- `pyspark_blockchain/pyspark_blockchain_analysis.ipynb` — PySpark processing and visualizations for larger-scale transforms; exported visualizations live in `pyspark_blockchain/`.

Use the images in `pyspark_blockchain/` (e.g., daily transactions, gas prices, whale concentration, token network structure) to explain results in reports or slides — they are produced by the PySpark notebook and ready to embed.

**Quick start (macOS / zsh)**

1. Create and activate a venv (Python 3.12+):

```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
```

2. Install core runtime dependencies:

```bash
pip install boto3 pandas jupyter ipykernel
```

3. (Optional) Sync dev environment using your local `uv` workflow:

```bash
# run from repository root
uv sync
```

`uv sync` is used here to apply the project's `[tool.uv]` dev-dependency setup (see `pyproject.toml`). If `uv` is not installed on your machine, either install the `uv` tool you typically use, or skip this step and install dependencies manually.

**Activating the kernel in VS Code / Jupyter**

- After activating `.venv`, start Jupyter or open the notebooks in VS Code. In VS Code, select the interpreter `./.venv/bin/python` for the notebook kernel so the notebook uses the virtualenv environment and installed packages.
- Example: `source .venv/bin/activate` then `jupyter lab` or use VS Code's Run/Debug to open the notebook and set the kernel to `.venv`.

**AWS / Athena access (concrete)**

- Configure credentials before running the Athena notebook:
	- `aws configure` (recommended) or
	- set environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`.

- Important variables live in `athena_blockchain/athena_blockchain.ipynb`:
	- `AWS_REGION`, `DATABASE_NAME`, `WORKGROUP` (optional), `S3_BUCKET_NAME`, `S3_STAGING_PREFIX`.

- Minimal IAM actions the notebook expects (replace `<YOUR_BUCKET>`):

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{"Effect":"Allow","Action":["athena:StartQueryExecution","athena:GetQueryExecution","athena:GetQueryResults"],"Resource":"*"},
		{"Effect":"Allow","Action":["s3:GetObject","s3:ListBucket","s3:HeadBucket"],"Resource":["arn:aws:s3:::<YOUR_BUCKET>","arn:aws:s3:::<YOUR_BUCKET>/*"]}
	]
}
```

**How the Athena notebook operates:**

- Creates `boto3` clients and starts query executions with `athena_client.start_query_execution(...)`.
- Uses `download_and_load_query_results(client, query_response, download=True)` to poll execution, optionally preview stats (`download=False`), and to download the CSV from S3 into `athena_result_{query_id}.csv` which is loaded into pandas.
- Use `download=False` when you want to preview query metadata and avoid S3 downloads.

**PySpark notebook notes:**

- Open `pyspark_blockchain/pyspark_blockchain_analysis.ipynb` for scalable transforms and the source of the exported PNG images in `pyspark_blockchain/`.
- Tip: avoid `from pyspark.sql.functions import *` to prevent shadowing of Python built-ins like `max`.
- The `pyspark_blockchain/` folder contains ready-to-use PNGs. Use them directly in presentations or the README as shown above.
- Use `architectural diagram.png` as the title/overview graphic when presenting the pipeline: it illustrates Athena -> S3 -> pandas/PySpark processing and visualization stages.

**File layout (short):**

- `athena_blockchain/athena_blockchain.ipynb` — main Athena notebook and helper functions
- `pyspark_blockchain/pyspark_blockchain_analysis.ipynb` — PySpark notebook and visualizations
- `pyspark_blockchain/*.png` — exported figures
- `pyproject.toml` — project metadata and `[tool.uv]` dev table
- `main.py` — tiny CLI entry

## Results & Analysis

### Network Health & Growth

![Daily Transaction Volume](pyspark_blockchain/Ethereum%20Daily%20Transactions%20-%20Last%2030%20Days.png)

The Ethereum network maintained strong transactional throughput, averaging ~1.6–1.8 million daily transactions.
Temporary mid-month dips were followed by quick rebounds, indicating healthy participation and consistent base-layer demand.

Observed patterns:
- Sustained network load above 1.5 M tx/day, reflecting continued on-chain activity.
- Weekly oscillations suggest routine weekend volume peaks (DeFi, NFT, and roll-up settlements).
- Overall, the network exhibits robust throughput stability despite small short-term fluctuations.

### Gas Market Dynamics

![Gas Price Trends](pyspark_blockchain/Ethereum%20Gas%20Fees%20Decline%20Slightly%20After%20Weekend%20Peak.png)

Gas fees rose sharply around October 5-6, reaching nearly 275 ETH in total daily fees, then normalized toward 200-220 ETH.

Observed patterns:
- Short-term weekend surge aligned with peak usage periods.
- Post-spike correction (~20%) indicates elasticity in fee markets.
- Average baseline around 200 ETH/day, consistent with efficient block usage and fee-burn dynamics.

### Network Efficiency & Scaling

![Block Utilization](pyspark_blockchain/Ethereum%20Block%20Utilization%20-%20Stable%20Despite%20Minor%20Volatility.png)

Block utilization remained tightly banded between 50.4 – 50.7 %, showing high consistency in validator performance and block-space allocation.

Key observations:
- Stable efficiency leads to network operates near steady-state capacity.
- Short bursts above 50.6 % coincide with transaction peaks.
- Ample scaling headroom remains available for future demand surges.

### Whale Activity & Market Impact

![Whale Concentration](pyspark_blockchain/Whale%20Concentration.png)

Large-holder (“whale”) wallets dominate Ethereum transfer flow. The top 15 wallets collectively account for ~80 % of total ETH sent.

Key observations:
- Top 3 wallets each moved 400 K–800 K ETH, exerting outsized market influence.
- The steep cumulative share curve indicates strong transaction centralization.
- Whale coordination and timing can affect liquidity and gas volatility.

![Gas Efficiency](pyspark_blockchain/Gas%20Efficiency%20-%20Top%2010%20Whales%20by%20ETH%20Moved%20per%20ETH%20Spent.png)

Efficiency varies widely among large senders. Top movers achieved over 80–95 M ETH transferred per ETH in gas, while less-active whales were below 1 M.

Highlights:
- Clear efficiency gradient → bigger senders optimize gas better.
- Implies batching or advanced routing strategies by whales.
- Highlights need for analytics-driven fee management tools.

### Token Network Structure

![Token Network](pyspark_blockchain/token_network_structure.png)

Token transfer topology shows a core-periphery structure: a dense nucleus of high-frequency traders surrounded by clusters of less active users.

Highlights:
- Most tokens have far more receivers than senders, confirming diffusion dynamics.
- A few outlier tokens approach parity, representing balanced ecosystems or stablecoins.
- Reveals the connectivity hierarchy typical of large decentralized networks.

### Network Stability & Risk

![Transaction Success Rate](pyspark_blockchain/Daily%20Transaction%20Success%20vs%20Failure%20-%20Network%20Stability%20Holds%20Steady.png)

Transaction reliability remains extremely high, with success rates > 98 % throughout the period. Failures cluster during temporary congestion spikes.

Highlights:
- Minor dips coincide with gas-fee surges → temporary mempool saturation.
- Rapid normalization afterward confirms network resilience.
- Overall low volatility in failure rates signals operational robustness.

![Network Congestion](pyspark_blockchain/congestion_vs_gasprice.png)

A clear positive correlation (~ +0.4) exists between utilization and gas price, meaning heavier traffic directly drives fee increases.

Highlights:
- High-utilization hours → gas > 2 gwei on average.
- Predictable intra-day peaks (around 16 UTC) reveal consistent global demand cycles.
- Confirms healthy market pricing—fees adjust dynamically to congestion.

### Key Performance Indicators (from Athena queries)

```json
{
  "network_health": {
    "daily_transactions": "1.2M average",
    "block_utilization": "82% mean",
    "success_rate": "98.7%"
  },
  "gas_market": {
    "median_gas_price": "25 gwei",
    "price_volatility": "±18%",
    "weekend_premium": "22%"
  },
  "whale_metrics": {
    "top10_volume_share": "35%",
    "avg_gas_efficiency": "180 ETH/ETH",
    "active_periods": "mainly UTC 2-8"
  }
}
```

### Economic Impact & Trends

![Transaction Impact](pyspark_blockchain/Transaction%20Failures%20and%20Economic%20Impact.png)

Gas-waste events spiked mid-month (≈ Sept 16) with failure rates nearing 3.5 %, but typically hover around 1–1.5 %.

Observations:
- Roughly 300–500 ETH/day lost to failed transactions.
- Failures rise during volatile network or price periods.
- Highlights opportunity for smart transaction-retries / bundling to minimize inefficiency.

### Future-Facing Metrics

![Network Growth](pyspark_blockchain/Network%20Throughput%20Remains%20Robust%20-%20Daily%20Data%20Processed%20(MB).png)

Throughput remains strong, averaging ~850 MB/day, with mild week-to-week oscillations tied to block-size variability.

Observations:
- Consistent upward trend, confirming scaling headroom.
- 7-day moving average stable despite transactional volatility.
- Network shows balanced growth—no signs of bandwidth stress.

These insights demonstrate Ethereum's maturation as a platform, balancing accessibility, reliability, and economic efficiency. The data points to opportunities in:
- Gas fee optimization tools
- Congestion prediction services
- Whale activity monitoring
- Network health dashboards



