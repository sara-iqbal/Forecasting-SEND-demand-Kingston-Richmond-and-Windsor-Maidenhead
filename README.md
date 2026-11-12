# Forecasting SEND demand: Kingston, Richmond and Windsor & Maidenhead

**A three-year forecast of Education, Health and Care Plans using DfE statutory data, Python, SQL and Power BI.**

[Live dashboard](https://<your-username>.github.io/<repository-name>/) · [Notebook](send_ehcp_forecast.ipynb) · [Insight brief](insight_brief.md)

![Forecast](images/forecast.png)

## The problem

An Education, Health and Care Plan (EHCP) is a legal document that sets out the support a child or young person with special educational needs and disabilities must receive. Councils have to assess, issue, fund and review every plan, every year.

Demand is rising quickly. England had 718,838 plans in January 2026, 12.5% more than a year before, and fewer than half of new plans were issued within the statutory 20 weeks in 2025. For a children's services organisation the practical questions are simple to ask and hard to answer: **how many plans will we be responsible for in three years, where will those children be educated, and are we keeping up?**

I built this project to answer those questions for the three boroughs served by Achieving for Children.

## How I approached it

**1. Data.** I used the DfE's *Education, health and care plans* release (reporting year 2026), which comes from the SEN2 statutory return every council submits. The notebook recognises each file by its columns rather than its name, keeps only the totals, and treats suppressed values as missing rather than zero.

**2. SQL.** I loaded the cleaned tables into SQLite and did the analysis in SQL: window functions for year-on-year growth and indexing, a CTE with `RANK()` to place each borough among all 157 councils, joins across requests, new plans and timeliness to follow the pipeline, and a join to the DfE statistical neighbour list for fair comparisons.

**3. Forecasting.** With only eight January snapshots and a change in DfE's collection method in 2023, a complex model would just memorise noise. I compared six simple, explainable approaches and tested them fairly with **rolling-origin backtesting** on every council in England: train on the past, forecast the next three years, and compare with what actually happened. The model with the lowest typical error across all councils was chosen, and the forecast range comes from its real past errors rather than an assumption.

**4. So what.** I translated the forecast into planning numbers a manager can use: the extra annual reviews each year, and the extra special school and independent special school places needed if the placement mix stays the same.

## Results

### Which model?

Typical (median) error across 152 councils:

| Model | 1 year ahead | 2 years ahead | 3 years ahead | Average |
| :- | -: | -: | -: | -: |
| Compound growth (last 3 years) **(chosen)** | 3.3% | 6.4% | 9.7% | 6.5% |
| Ensemble | 3.5% | 7.1% | 10.6% | 7.1% |
| Drift | 3.8% | 7.4% | 11.4% | 7.5% |
| Linear (last 4 years) | 4.1% | 7.8% | 11.5% | 7.8% |
| Damped trend (Holt) | 4.0% | 8.3% | 11.5% | 7.9% |
| Naive | 9.6% | 18.8% | 26.6% | 18.3% |

![Model backtest](images/model_backtest.png)

### The forecast

| Borough | January 2026 | January 2029 forecast | 80% range | Change |
| :- | -: | -: | -: | -: |
| Kingston | 2,159 | 2,860 | 2,502 to 3,466 | +701 (32%) |
| Richmond | 2,166 | 2,753 | 2,409 to 3,337 | +587 (27%) |
| Windsor & Maidenhead | 1,363 | 1,686 | 1,475 to 2,043 | +323 (24%) |
| England | 718,838 | 999,381 | 874,360 to 1,211,204 | +280,543 (39%) |

### What it means

#### Kingston

* **Demand:** 2,159 EHC plans in January 2026, forecast to reach **2,860** by January 2029 (80% range 2,502 to 3,466), an increase of 32%. England is forecast to grow by 39% over the same period.
* **Growth:** plans grew by 9.3% a year on average since 2019, ranking 96 of 149 councils for growth (1 = fastest). Over the last three years: 9.8% a year.
* **Placements:** 29% of plans are in special schools (England 29%), including 5% in independent special schools (England 5%).
* **Timeliness:** 97% of new plans issued within 20 weeks in 2025 (England 46%).
* **So what:** about **701 more annual reviews a year** by 2029. If the placement mix stays the same, around 205 more children will need a special school place, including 33 in independent special schools, which are typically the most expensive placements.

#### Richmond

* **Demand:** 2,166 EHC plans in January 2026, forecast to reach **2,753** by January 2029 (80% range 2,409 to 3,337), an increase of 27%. England is forecast to grow by 39% over the same period.
* **Growth:** plans grew by 6.8% a year on average since 2019, ranking 132 of 149 councils for growth (1 = fastest). Over the last three years: 8.3% a year.
* **Placements:** 24% of plans are in special schools (England 29%), including 7% in independent special schools (England 5%).
* **Timeliness:** 96% of new plans issued within 20 weeks in 2025 (England 46%).
* **So what:** about **587 more annual reviews a year** by 2029. If the placement mix stays the same, around 138 more children will need a special school place, including 39 in independent special schools, which are typically the most expensive placements.

#### Windsor & Maidenhead

* **Demand:** 1,363 EHC plans in January 2026, forecast to reach **1,686** by January 2029 (80% range 1,475 to 2,043), an increase of 24%. England is forecast to grow by 39% over the same period.
* **Growth:** plans grew by 5.6% a year on average since 2019, ranking 137 of 149 councils for growth (1 = fastest). Over the last three years: 7.3% a year.
* **Placements:** 27% of plans are in special schools (England 29%), including 6% in independent special schools (England 5%).
* **Timeliness:** 100% of new plans issued within 20 weeks in 2025 (England 46%).
* **So what:** about **323 more annual reviews a year** by 2029. If the placement mix stays the same, around 87 more children will need a special school place, including 21 in independent special schools, which are typically the most expensive placements.

![Growth and timeliness](images/growth_and_timeliness.png)

## What I learned

**Simple, tested models beat clever, untested ones.** The most useful thing in this project is not the forecast itself but the evidence that the chosen method has worked in the past, across many councils.

**The range matters as much as the number.** A single figure gives false confidence. Showing an 80% range, built from real errors, lets a manager plan for a realistic high and low case.

**Bias is a finding.** The backtest shows whether demand has tended to come in above forecast. If it has, planning should lean towards the upper end of the range.

**A forecast has to lead to a decision.** Turning plans into annual reviews and school places is what makes the analysis useful to the people running the service.

## Limitations

* Eight years of data, with a collection change in 2023.
* The placement scenarios assume today's mix continues.
* The forecast is for plan numbers. Costs, staffing and sufficiency of places would need local data.

## Run it yourself

1. Download the *Education, health and care plans* "Download all data" zip from Explore Education Statistics (and optionally the statistical neighbour workbook from GOV.UK).
2. Open the notebook in Google Colab and select **Runtime > Run all**.

## Tools

Python (pandas, NumPy, statsmodels, Matplotlib), SQL (SQLite), Power BI, Chart.js, Google Colab, GitHub Pages.

*Built by Sara Iqbal (MSc Data Science). Contains public sector information licensed under the Open Government Licence v3.0.*
