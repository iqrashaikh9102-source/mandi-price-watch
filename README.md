# 🌾 Mandi Price Watch: Indian Agricultural Market Intelligence

An interactive **Power BI** dashboard that explains how Indian crop prices behave: which crops are expensive, which are risky, and when prices usually move. It uses 19,680 monthly price records for 175 crops (Feb 2002 to Nov 2026) and turns them into KPIs, seasonal analysis, volatility analysis and procurement recommendations.

![Page 1 – Mandi Price Analysis]

## Business problem

Agricultural prices swing with weather, arrivals and festivals. A buyer who only looks at the latest price misses two things that matter for planning: **how far prices typically swing** and **when in the year they tend to be high or low**. This project answers four questions for a procurement or market-analysis team:

1. Which crops are expensive and which are affordable?
2. Which crops carry the most price risk?
3. How do prices behave by season, and how does a season compare with last year?
4. What should a buyer do with this information?

## Dashboard pages

| Page | What it shows |
|---|---|
| **1. Mandi Price Analysis** | KPI cards (Total Crops, Total Category, Median Market Price, Highest Recorded Price, Price Volatility %), crops-by-category donut, Top 10 most expensive and most affordable crops, selected-crop price trend, crop-wise price comparison |
| **2. Seasonality and Volatility** | Seasonal selected vs previous year price, Price Volatility % by crop, Seasonal YoY % by season, seasonal price heatmap, volatility treemap, seasonal comparison |
| **3. Executive Insights** | Highest Risk Crop, Market Leader Crop, Seasonal YoY % and Best Performing Season cards; risk vs price scatter; seasonal performance by category; dynamic market highlights; business recommendations |

![Page 2 – Seasonality and Volatility]
![Page 3 – Executive Insights]

## Dataset

- **File:** `data/mandi-india-monthly.csv`: 19,680 rows, 175 crops, Feb 2002 to Nov 2026.
- **One row** = one crop in one month. The price is the median across markets of each market's monthly average modal price, in **₹ per quintal** (1 quintal = 100 kg).
- **Source:** Agmarknet (Directorate of Marketing & Inspection) via data.gov.in, compiled by Today Market Rates. Please check the original source's terms before reusing the data.
- **Category field:** the 17 crop categories used in the dashboard (for example Vegetables with 51 crops) are added in the Power BI model and are not in the CSV. Include your mapping file in `data/` so others can reproduce the dashboard.
- **Quality checks:** no blank values and no duplicate crop-month pairs. Coverage is uneven: about 81 crops report in 2015 versus 175 in 2026, 25 crops have fewer than 12 months of history, and November 2026 is a partial month.

## Key findings

- **Volatility is concentrated in cheap perishables.** Among crops with at least 12 months of data, the most volatile are Carrot (82.1%), Kinnow (81.4%), Radish (77.9%), Drumstick (74.7%) and Onion (73.7%).
- **Higher-priced crops tend to be steadier.** The rank correlation between median price and volatility is −0.44 across 150 crops. This is an association, not a proven cause.
- **Seasonality is strong.** The median peak-to-trough seasonal swing is about 29% across 81 long-history crops, and Post-monsoon is the peak season for 41 of them.
- **Headline seasonal YoY can mislead.** In Winter 2025 the all-crops average rose 56.3%, but only about 12% when comparing the 79 crops present in both years. Most of the headline came from new crops entering the data.

## Technical highlights (DAX)

**Price volatility** as a coefficient of variation, so crops of different price levels are comparable:

```dax
Price Volatility % =
DIVIDE(
    STDEV.P( 'mandi'[Median Price] ),
    [Average Price]
)
```

**Seasonal year-over-year** using a custom `Season Year` (December is counted in the following year so each winter stays in one block):

```dax
Season Year =
YEAR( 'mandi'[Period] ) + IF( MONTH( 'mandi'[Period] ) = 12, 1, 0 )

Seasonal Avg Selected Year =
VAR SelectedYear = SELECTEDVALUE( 'mandi'[Season Year] )
RETURN
    IF( ISBLANK( SelectedYear ), BLANK(),
        CALCULATE( [Average Price],
            REMOVEFILTERS( 'mandi'[Season Year] ),
            'mandi'[Season Year] = SelectedYear ) )

Seasonal YoY % =
DIVIDE( [Seasonal Avg Selected Year] - [Seasonal Avg Prior Year], [Seasonal Avg Prior Year] )
```

**Dynamic insight cards** built with `VAR`, `TOPN`, `CONCATENATEX` and `FORMAT`, so the sentences change with the slicers (for example *"Garlic records the highest price volatility at 58.67%."* with Season Year 2025 selected, or Carrot at 82.11% across all years).

Concepts used: calculated columns vs measures, filter context, `CALCULATE`, context transition, `SELECTEDVALUE`, `TOPN`, `VAR`/`RETURN`, `DIVIDE`.

## Business recommendations

1. **Procurement strategy:** use phased buying or price-locking for high-volatility, high-spend crops; spot-buy steadier crops.
2. **Price risk management:** monitor volatility with a minimum-history rule and read YoY on a like-for-like crop set.
3. **Seasonal planning:** plan purchases around the low season for strongly seasonal crops and avoid last-minute peak buying.

## Limitations and next steps

- Prices are national medians of market averages, so regional and quality differences are hidden.
- The volatility measure covers the whole period, so it includes long-run price growth as well as short-term swings.
- Ranking cards should ignore crops with fewer than 12 months of data (for example, Mentha oil has one record).
- Planned improvements: minimum-history rule in the insight measures, a like-for-like YoY measure, rolling 12-month volatility, a proper calendar table, and arrival-volume and weather data.

## Repository structure

```
mandi-price-watch/
├── Mandi_Price_Watch.pbix
├── data/
│   ├── mandi-india-monthly.csv
│   └── category-mapping.csv   (add yours)
├── docs/
│   ├── Mandi_Price_Watch_Analytics_Report.pdf
│   ├── Mandi_Price_Watch_DAX_Learning_Guide_Hinglish.pdf
│   └── Mandi_Price_Watch_Presentation.pptx
├── images/            (page-1.png, page-2.png, page-3.png)
└── README.md
```

## How to run

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `Mandi_Price_Watch.pbix`.
3. If Power BI asks for the data location, point it to `data/mandi-india-monthly.csv` (Home → Transform data → Data source settings).

## Tools

Power BI Desktop, DAX, Power Query.

## Author

**[Your Name]**: [LinkedIn](https://www.linkedin.com/in/your-profile) · [GitHub](https://github.com/iqrashaikh9102-source)
