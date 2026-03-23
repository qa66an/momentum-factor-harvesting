# momentum-factor-harvesting

Implementation of the classic 12-1 price momentum factor (Jegadeesh & Titman, 1993) on a fully investable US equity universe using point-in-time data. Built to document that the momentum premium is real and reproducible — and to honestly assess its statistical properties on an investable mid/large cap universe.

---

## Finding

On an investable mid/large cap universe (2000–2025), the 12-1 momentum factor delivers **8.9% annualized return** with a **win rate of 59.6%** on the long leg. The long-short factor spread is positive but **not statistically significant** on this universe and period — consistent with the known finding that momentum is strongest in smaller, less efficient stocks that are excluded from an investable universe.

The value factor (see [value-factor-harvesting](https://github.com/qa66an/value-factor-harvesting)) achieves statistical significance on the same universe and pipeline, confirming the infrastructure is sound. The momentum result is an honest reflection of the factor's behavior under investability constraints.

---

## Methodology

### Signal: 12-1 Momentum

Returns are computed over a 12-month formation window, skipping the most recent month to avoid short-term reversal:

```
Formation window:  t-13 → t-2  (12 months)
Skip:              t-1          (reversal avoidance)
Holding period:    t → t+1      (1 month)
Rebalance:         Monthly
```

### Universe

- Domestic common stock only (primary, secondary, and standard class)
- Inflation-adjusted market cap filter: $200M base (year 2000), adjusted at 3% annually
- Equivalent to approximately $200M in 2000 → $440M by 2025 — maintains consistent investability standard through time
- Winsorized at 99th percentile of momentum signal to remove extreme outliers

$$\text{Limit}_{\text{min}} = \$200,000,000 \times (1.03)^{(\text{year} - 2000)}$$

### Portfolio Construction

- **Long leg**: top decile by momentum signal (~250 stocks/month), equal weighted
- **Short leg**: bottom decile by momentum signal (~250 stocks/month), equal weighted
- **Long-short**: long return minus short return each month

### Data

- **Price data**: Sharadar SEP — point-in-time adjusted prices, no survivorship bias
- **Fundamentals**: Sharadar SF1 — point-in-time shares, equity, net income, assets
- **Beta**: 126-day rolling OLS beta vs SPY, computed at formation end date
- **Trading calendar**: NYSE calendar via `pandas_market_calendars`

No lookahead bias. All signals use data available as of the ranking date.

---

## Results

### Overall Performance (2000–2025)

| Strategy                   | Annual Return | Sharpe | T-Stat | P-Value | Win Rate | Max Drawdown |
| -------------------------- | ------------- | ------ | ------ | ------- | -------- | ------------ |
| Long only (top decile)     | 8.9%          | 0.34   | 1.75   | 0.081   | 59.6%    | -62.2%       |
| Short only (bottom decile) | -4.7%         | -0.13  | -0.66  | 0.511   | 48.7%    | -97.8%       |
| Long-Short (factor)        | 3.9%          | 0.14   | 0.69   | 0.490   | 59.9%    | -78.9%       |
| SPY (benchmark)            | 9.6%          | ~0.45  | —      | —       | —        | -51.0%       |

### Year-by-Year Long-Short Returns

| Year | Annual Return | Sharpe | Win Rate |
| ---- | ------------- | ------ | -------- |
| 2000 | +31.6%        | 0.45   | 50.0%    |
| 2001 | -26.8%        | -0.46  | 58.3%    |
| 2002 | +64.9%        | 1.18   | 66.7%    |
| 2003 | -30.0%        | -1.83  | 41.7%    |
| 2004 | -5.5%         | -0.45  | 33.3%    |
| 2005 | +17.4%        | 1.26   | 75.0%    |
| 2006 | -9.0%         | -1.05  | 50.0%    |
| 2007 | +34.4%        | 2.97   | 83.3%    |
| 2008 | +11.0%        | 0.25   | 66.7%    |
| 2009 | -58.4%        | -1.86  | 41.7%    |
| 2010 | +1.4%         | 0.10   | 50.0%    |
| 2011 | +18.7%        | 2.43   | 83.3%    |
| 2012 | +18.2%        | 1.26   | 66.7%    |
| 2013 | +6.4%         | 1.10   | 66.7%    |
| 2014 | +7.4%         | 0.73   | 66.7%    |
| 2015 | +29.8%        | 0.99   | 66.7%    |
| 2016 | -21.1%        | -1.21  | 33.3%    |
| 2017 | +11.6%        | 1.01   | 58.3%    |
| 2018 | +11.7%        | 1.02   | 58.3%    |
| 2019 | -4.3%         | -0.18  | 58.3%    |
| 2020 | -11.9%        | -0.34  | 58.3%    |
| 2021 | +2.7%         | 0.11   | 66.7%    |
| 2022 | +85.2%        | 3.25   | 91.7%    |
| 2023 | -20.1%        | -0.98  | 50.0%    |
| 2024 | +20.0%        | 1.20   | 58.3%    |
| 2025 | +10.7%        | 0.84   | 58.3%    |

---

## Discussion

### Statistical significance

```
Long-Short T-Stat:  0.69  → not statistically significant
Long-Short P-Value: 0.49  → cannot reject null hypothesis
Long only T-Stat:   1.75  → borderline (significant at 90%, not 95%)
```

The 12-1 momentum factor does not achieve statistical significance on this investable universe over the 2000–2025 period. This is an honest result, not a pipeline failure — the same pipeline produces a statistically significant result for the value factor (T-Stat 2.82, P-Value 0.005), confirming the infrastructure is correctly implemented.

### Why momentum is not significant here but value is

Two structural reasons:

**1. Universe constraint.** The momentum premium is documented to be strongest in small and micro-cap stocks (Israel & Moskowitz, 2013). The inflation-adjusted $200M filter excludes precisely the stocks where the signal is strongest. O'Shaughnessy (2011) documents 17% annualized for 6-1 momentum on an all-stocks universe — the premium compresses significantly on a mid/large cap investable universe.

**2. Time period.** The 2000–2025 sample is dominated by two extended bull markets (2003–2007, 2009–2021) interrupted by severe momentum crashes (2009, 2003). Value is more stable across regimes; momentum is regime-dependent and its crash risk (Daniel & Moskowitz, 2016) reduces statistical power over shorter samples.

### The short leg

The short leg exhibits near-zero negative returns with high volatility — a known empirical result. Bottom decile stocks tend to mean-revert over 1-month holding periods (loser reversal effect), and are expensive/difficult to short in practice. Consistent with AQR (2014), approximately 70%+ of the momentum premium is driven by the long leg.

### Momentum crashes

2009 and 2003 show severe drawdowns on the long-short factor, consistent with the documented momentum crash phenomenon (Daniel & Moskowitz, 2016). These crashes occur when prior losers snap back violently after a market selloff, hurting the short leg disproportionately.

### Comparison with O'Shaughnessy

On an all-stocks universe including micro-caps, O'Shaughnessy documents ~31% excess return spread for 6-1 momentum over the benchmark. This implementation on an investable universe with 12-1 signal delivers a positive but statistically insignificant spread — confirming the premium exists in the data directionally but is not reliably harvestable at institutional scale without accessing smaller, less liquid names.

---

## References

- Jegadeesh, N. & Titman, S. (1993). _Returns to Buying Winners and Selling Losers_. Journal of Finance.
- Israel, R. & Moskowitz, T. (2013). _The Role of Shorting, Firm Size, and Time on Market Anomalies_. Journal of Financial Economics.
- Daniel, K. & Moskowitz, T. (2016). _Momentum Crashes_. Journal of Financial Economics.
- O'Shaughnessy, J. (2011). _What Works on Wall Street_. McGraw-Hill.
- AQR Capital Management (2014). _Fact, Fiction and Momentum Investing_.

---

## Repository Structure

```
momentum-factor-harvesting/
├── functions.py               # Core pipeline functions
├── prepare_yearly_data.py     # Generates raw yearly CSVs from Sharadar
├── create_deciles.py          # Top/bottom decile construction
├── eval.py                    # Performance stats and results
├── requirements.txt
└── README.md
```

---

## Requirements

```
duckdb
pandas
numpy
scipy
pandas_market_calendars
python-dateutil
```

---

## Data

Requires a Sharadar subscription via Nasdaq Data Link:

- `SHARADAR/SEP` — daily equity prices (adjusted and unadjusted)
- `SHARADAR/SF1` — point-in-time fundamental data
