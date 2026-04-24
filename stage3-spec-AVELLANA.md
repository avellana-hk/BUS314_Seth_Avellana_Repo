# Technical Specification: Apple Inc. (AAPL) Financial Ratio Model
**BUS-314 | Stage 3 | FY2024 Ratio Analysis**

---

## 1. Problem Statement

This specification documents a quantitative ratio model built to evaluate the financial performance of **Apple Inc. (AAPL)** for fiscal year 2024 (ending September 28, 2024), with the prior-year period (FY2023, ending September 30, 2023) serving as the comparative base. The objective is to systematically compute and interpret a comprehensive set of financial ratios — spanning performance, profitability, efficiency, leverage, liquidity, and Du Pont decomposition — using data sourced directly from Apple's SEC-filed 10-K. The model is designed to support executive-level analysis and investment assessment, producing auditable, formula-driven outputs suitable for FP&A review and Stage 4 strategic interpretation.

---

## 2. Inputs (Known Variables)

All figures are in millions of USD unless noted. Sources: Apple 10-K FY2024, SEC EDGAR.

### Balance Sheet

| Named Range | Description | FY2024 | FY2023 |
|---|---|---|---|
| `BAL_cash_marketable_securities_2024` | Cash and marketable securities | 65,171 | 61,555 |
| `BAL_receivables_2024` / `_2023` | Receivables | 33,410 | 29,508 |
| `BAL_inventories_2024` / `_2023` | Inventories | 7,286 | 6,331 |
| `BAL_assets_current_2024` / `_2023` | Total current assets | 152,987 | 143,566 |
| `BAL_assets_total_2024` / `_2023` | Total assets | 364,980 | 352,583 |
| `BAL_liabilities_current_2024` | Total current liabilities | 176,392 | 145,308 |
| `BAL_liabilities_total_2024` | Total liabilities | 308,030 | 290,437 |
| `BAL_debt_long_term_2024` / `_2023` | Long-term debt | 85,750 | 95,281 |
| `BAL_equity_shareholders_2024` / `_2023` | Total shareholders' equity | 56,950 | 62,146 |

### Income Statement

| Named Range | Description | FY2024 |
|---|---|---|
| `INC_sales` | Net sales | 391,035 |
| `INC_cost_goods_sold` | Cost of goods sold | 210,352 |
| `INC_ebit` | Earnings before interest & taxes | 143,141 |
| `INC_interest_expense` | Interest expense | 3,931 |
| `INC_depreciation` | Depreciation | 11,445 |
| `INC_net` | Net income | 113,225 |

### Cash Flow Statement

| Named Range | Description | FY2024 |
|---|---|---|
| `CASH_operating` | Cash provided by operations | 131,033 |
| `CASH_capex` | Capital expenditures | (9,447) |
| `CASH_investing` | Cash from investing activities | 2,935 |
| `CASH_financing` | Cash from financing activities | (121,983) |

### Market / Analyst Inputs

| Named Range | Description | Value |
|---|---|---|
| `share_price` | End-of-fiscal-year closing price | $226.51 |
| `shares_outstanding` | Shares outstanding (millions) | 15,204 |
| `cost_capital` | Estimated WACC | 9.0% |
| `tax_rate` | Analyst assumption (effective rate) | 24.1% |

---

## 3. Named Range Conventions

Named ranges follow the pattern `[SOURCE]_[description]_[year]` for raw inputs and `[scope]_[description]` for derived values. Raw inputs use uppercase prefixes (`BAL_`, `INC_`, `CASH_`); derived/intermediate values use camelCase scope prefixes (`startYear_`, `currentYear_`, `avg_`); ratio outputs use `RATIO_`.

Examples: `BAL_equity_shareholders_2023`, `INC_sales`, `currentYear_after_tax_operating_income`, `avg_total_assets`, `RATIO_asset_turnover`.

---

## 4. Assumptions & Constraints

- **Tax rate** is held constant at 24.1% (analyst assumption; can be toggled to effective rate).
- **Cost of capital** is set at 9.0% (estimated WACC; may be refined via CAPM).
- **Interest expense** is treated as a simple annual figure; no intra-period compounding.
- **Share price** is the end-of-fiscal-year closing price; no time-weighted averaging.
- **No off-balance-sheet items** are incorporated (e.g., operating leases are excluded from leverage metrics beyond what appears on-balance-sheet).
- **Retained earnings** for FY2024 and FY2023 are reported as $0 on the model balance sheet, reflecting Apple's accumulated other comprehensive loss netting; equity figures are used directly as reported.
- All ratios use **365-day year** for daily conversion.
- **Start-of-year denominators** (FY2023 figures) are used for turnover and return ratios to reflect beginning-of-period capital base; average-based alternatives are also computed.

---

## 5. Calculation Flow

### 5.1 Derived Inputs

```
market_capitalization        = share_price × shares_outstanding
startYear_equity             = BAL_equity_shareholders_2023
startYear_total_assets       = BAL_assets_total_2023
startYear_total_capitalization = BAL_debt_long_term_2023 + BAL_equity_shareholders_2023

currentYear_after_tax_operating_income = INC_net + (1 − tax_rate) × INC_interest_expense
currentYear_daily_sales_average        = INC_sales / 365
currentYear_cost_goods_sold_daily      = INC_cost_goods_sold / 365
currentYear_working_capital_net        = BAL_assets_current_2024 − BAL_liabilities_current_2024
currentYear_total_capitalization       = BAL_debt_long_term_2024 + BAL_equity_shareholders_2024

avg_equity                   = AVERAGE(startYear_equity, currentYear_equity)
avg_total_assets             = AVERAGE(startYear_total_assets, currentYear_assets_total)
avg_total_capitalization     = AVERAGE(startYear_total_capitalization, currentYear_total_capitalization)
```

### 5.2 Performance Ratios

```
MVA   = market_capitalization − currentYear_equity
M/B   = market_capitalization / currentYear_equity
EVA   = currentYear_after_tax_operating_income − (cost_capital × startYear_total_capitalization)
```

### 5.3 Profitability Ratios

```
ROA        = currentYear_after_tax_operating_income / startYear_total_assets
ROC        = currentYear_after_tax_operating_income / startYear_total_capitalization
ROE        = INC_net / startYear_equity
ROA [avg]  = currentYear_after_tax_operating_income / avg_total_assets
ROC [avg]  = currentYear_after_tax_operating_income / avg_total_capitalization
ROE [avg]  = INC_net / avg_equity
```

### 5.4 Efficiency Ratios

```
Asset turnover              = INC_sales / startYear_total_assets                    → RATIO_asset_turnover
Receivables turnover        = INC_sales / startYear_receivables
Avg. collection period      = startYear_receivables / currentYear_daily_sales_average
Inventory turnover          = INC_cost_goods_sold / startYear_inventory
Days in inventory           = startYear_inventory / currentYear_cost_goods_sold_daily
Profit margin               = INC_net / INC_sales
Operating profit margin     = currentYear_after_tax_operating_income / INC_sales     → RATIO_operating_profit_margin
```

### 5.5 Leverage Ratios

```
Long-term debt ratio        = currentYear_debt_long_term / (currentYear_debt_long_term + currentYear_equity)
Long-term debt-equity ratio = currentYear_debt_long_term / currentYear_equity
Total debt ratio            = currentYear_liabilities_total / currentYear_assets_total
Times interest earned       = INC_ebit / INC_interest_expense
Cash coverage ratio         = (INC_ebit + INC_depreciation) / INC_interest_expense
Debt burden                 = INC_net / currentYear_after_tax_operating_income        → RATIO_debt_burden
Leverage ratio              = currentYear_assets_total / currentYear_equity           → RATIO_leverage
```

### 5.6 Liquidity Ratios

```
Current ratio               = currentYear_assets_current / currentYear_liabilities_current
Quick ratio                 = (currentYear_cash_marketable_securities + BAL_receivables_2024) / currentYear_liabilities_current
Cash ratio                  = currentYear_cash_marketable_securities / currentYear_liabilities_current
NWC to assets               = currentYear_working_capital_net / currentYear_assets_total
```

### 5.7 Du Pont Decomposition

```
ROA (Du Pont) = RATIO_asset_turnover × RATIO_operating_profit_margin

ROE (Du Pont) = RATIO_leverage × RATIO_asset_turnover × RATIO_operating_profit_margin × RATIO_debt_burden
```

---

## 6. Outputs

The model produces the following outputs in the **Ratios** worksheet:

- **Scalar metrics:** MVA ($3,386,908M), M/B (60.5×), EVA ($102,040M)
- **Profitability table:** ROA, ROC, ROE — both start-of-year and average variants (six values)
- **Efficiency table:** Seven ratios including turnover rates, collection period, days in inventory, and margin metrics
- **Leverage table:** Seven ratios including debt ratios, coverage ratios, debt burden, and leverage multiple
- **Liquidity table:** Four ratios (current, quick, cash, NWC-to-assets)
- **Du Pont decomposition:** Two decomposed return metrics (ROA and ROE) with labeled component drill-down
- **Supporting inputs summary:** All named ranges with formula references for auditability

---

## 7. Model Review — What Worked & What to Improve

**What worked well:**
The named range framework is the model's strongest feature. Mapping every input to a structured identifier (e.g., `BAL_debt_long_term_2024`) makes formulas self-documenting and dramatically reduces reference errors. The separation of raw inputs, derived intermediates, and ratio outputs into distinct blocks within the Ratios tab keeps the calculation flow readable. The dual profitability treatment (start-of-year vs. average denominators) adds analytical depth that a single-method approach would miss. The Du Pont decomposition cross-validates ratio outputs by confirming that the product of component ratios reconciles to the headline return figures.

**What should be improved:**
- **Quick ratio formula inconsistency:** The quick ratio pulls `BAL_receivables_YEAR` rather than the properly named `BAL_receivables_2024`, suggesting a copy-paste naming gap that should be standardized.
- **Negative equity and retained earnings:** Apple's accumulated deficit (-$26,326M) is zeroed out in the model balance sheet. A rigorous version should reflect this correctly, as it affects equity-based ratios and signals share repurchase scale.
- **Static tax rate:** The 24.1% assumption is reasonable but should be a clearly labeled, yellow-highlighted assumption cell to signal to any reviewer that it is adjustable — not buried in a formula.
- **No scenario or sensitivity layer:** The current model is a single-point estimate. Adding a scenario toggle (e.g., varying cost of capital from 8%–11%) would enable richer Stage 4 analysis.
- **Missing cash flow ratios:** Operating cash flow relative to total debt or capital expenditures coverage would strengthen the leverage and liquidity picture.
- **Layout:** The Ratios tab mixes input scaffolding, derived intermediates, and output ratios in a long vertical list. A cleaner build would separate these into named sections or sub-tables with clear visual demarcation.

---

## 8. Limitations & Next Steps

This model is limited to a **single fiscal year** with no multi-year trend analysis, no peer benchmarking, and no forward projections. It excludes off-balance-sheet obligations, stock-based compensation adjustments, and segment-level breakdown. The WACC is estimated rather than formally derived.

These constraints are appropriate for Stage 2–3 scope. In **Stage 4**, the computed ratios will be interpreted in the context of Apple's competitive position, capital return program, and sector norms. The Du Pont decomposition and EVA will anchor the strategic recommendation. The "Model Review" findings above — particularly the scenario gap and cash flow ratio omissions — inform the AI prompt design and the qualitative analysis framing of the executive memo.

---

*Data source: Apple 10-K FY2024, SEC EDGAR. All figures in millions of USD unless otherwise noted.*
