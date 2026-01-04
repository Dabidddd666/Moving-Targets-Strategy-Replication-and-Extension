# Moving Targets Strategy Implementation

This repository implements the "Moving Targets" trading strategy based on the paper:

**Cohen, L., & Nguyen, Q. (2024). Moving Targets. SSRN 4736129.**

## Overview

The strategy identifies when managers strategically shift performance targets in earnings conference calls. When firms "move" targets (stop mentioning previously discussed metrics), it predicts negative future returns.

**Key Finding**: Firms with moving targets underperform by ~78-99 basis points per month (9-12% annually) in the following quarter.

## Implementation Steps

The implementation is organized into 11 sequential notebooks:

### Step 0: Setup (`00_setup.ipynb`)
- Environment setup and configuration
- Directory structure creation
- Russell 3000 universe loading

### Step 1: Ingest Transcripts (`01_ingest_transcripts.ipynb`)
- Fetches earnings call transcripts from Financial Modeling Prep API
- Cleans and normalizes text
- Separates presentation vs Q&A sections

### Step 2: Identifier Mapping (`02_identifier_mapping.ipynb`)
- Maps tickers to firm identifiers (permno/gvkey)
- Aligns calls to fiscal quarters
- Handles multiple calls per quarter

### Step 3: Target Extraction (`03_target_extraction.ipynb`)
- Uses spaCy NLP to extract "targets" from transcripts
- Extracts PRODUCT entities and MONEY/PERCENT entities with referents
- Normalizes target strings

### Step 4: Build Target Sets (`04_build_target_sets.ipynb`)
- Aggregates targets to firm-quarter level
- Creates target sets T_t for each firm-quarter

### Step 5: Compute MT Signal (`05_compute_mt_signal.ipynb`)
- Computes Moving Targets signal: MT_t = len(T_{t-4} - T_t) / len(T_{t-4})
- Also computes persistent MT and complexity measures

### Step 6: Monthly Signal (`06_monthly_signal.ipynb`)
- Converts quarterly signal to monthly "as-of" signal
- Forward-fills until next call updates

### Step 7: Universe Construction (`07_universe_construction.ipynb`)
- Filters to Russell 3000 universe
- Applies tradable filters (price, market cap, etc.)

### Step 8: Backtest (`08_backtest.ipynb`)
- Ranks firms into quantiles based on MT signal
- Forms long-short portfolios (low MT vs high MT)
- Computes portfolio returns


## Requirements

- Python 3.10+ (3.11 ideal)
- Core libraries: pandas, numpy, pyarrow, scipy, statsmodels, matplotlib
- NLP: spacy + en_core_web_sm model
- Optional: tqdm, joblib

## Setup Instructions

1. Install dependencies:
```bash
pip install pandas numpy pyarrow scipy statsmodels matplotlib seaborn
pip install spacy tqdm joblib certifi requests
python -m spacy download en_core_web_sm
```

2. Update API key in `00_setup.ipynb`:
   - Replace the API key with your Financial Modeling Prep API key

3. Run notebooks sequentially (00 → 01 → 02 → ... → 10)

## Data Requirements

For full implementation, you'll need:
- **CRSP**: Stock returns, prices, market caps, permno mappings
- **Compustat**: Fiscal calendar, gvkey mappings
- **Fama-French Factors**: For alpha calculations

The current implementation provides the framework and can work with sample data.

## Key Parameters

- **Lookback**: 4 quarters (t-4) for MT calculation
- **Quantiles**: 10 (deciles) for portfolio formation
- **Universe**: Russell 3000
- **Date Range**: 2006-2020 (configurable)

## Expected Results

Based on the paper:
- **Long-Short Return**: ~78-99 bps/month (9-12% annually)
- **T-statistic**: ~4.38 (highly significant)
- **Persistent Targets**: Stronger signal (99 bps/month)
- **Complex Targets**: Stronger signal

## Notes

- The implementation uses ticker symbols as firm identifiers. In production, use permno (CRSP) or gvkey (Compustat).
- Return calculations require CRSP data. The framework is provided but needs actual return data.
- API rate limits may require adjusting delays in Step 1.

## Citation

If you use this implementation, please cite:
```
Cohen, L., & Nguyen, Q. (2024). Moving Targets. SSRN 4736129.
```
