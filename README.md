# Norwegian Petroleum Financial Insights

A Power BI analysis of how Brent-price movements and production changes influenced the estimated gross value of Norwegian oil production from 2021–2025.

## Research Question

**When Norway's estimated oil value changes, how much is driven by Brent prices and how much by production?**

## Dashboard Preview

### Norwegian Petroleum Financial Overview

![Financial Overview](Financial%20Overview.png)

### Field Resilience 2023

![Field Resilience 2023](Field%20Resilience%202023.png)

## Project Objective

The purpose of this project was to combine Norwegian oil-production data with monthly Brent crude prices and analyse how changes in production volumes and market prices influenced estimated gross oil value.

The project also examines field-level resilience during periods of weaker Brent prices.

## Tools Used

- Power BI
- Power Query
- DAX
- Data Modelling
- Financial Analysis
- Data Visualisation
## Data Sources

The analysis uses public data from:

- **Norwegian Offshore Directorate (SODIR)** – monthly field-level petroleum production data
- **Norwegian Offshore Directorate (SODIR)** – field information and field identifiers
- **U.S. Energy Information Administration (EIA)** – monthly Europe Brent Spot Price FOB

The analysis period covers **January 2021 to December 2025**.

---

## Data Preparation

Data cleaning and transformation were completed in **Power Query**.

### Production Data

The monthly production dataset was cleaned by:

- filtering the data to 2021–2025
- keeping field-level oil production variables
- creating a monthly date field
- standardising field identifiers
- assigning appropriate numeric and date data types

### Brent Price Data

The Brent dataset was cleaned by:

- removing metadata rows
- keeping the monthly date and Brent-price columns
- converting Brent prices to decimal format
- filtering the period to 2021–2025

### Field Lookup Data

The field lookup table was cleaned by:

- retaining field name and field ID
- removing blank field IDs
- checking for duplicate field IDs
- trimming text fields
- removing unnecessary metadata columns

---

## Data Model

The Power BI model contains four main tables:

- `Production`
- `Brent_price`
- `Field_Lookup`
- `Date_Table`

### Relationships

- `Production[Date]` → `Date_Table[Date]`
- `Brent_price[Date]` → `Date_Table[Date]`
- `Production[Field_ID]` → `Field_Lookup[Field_ID]`

The Date Table supports consistent year-over-year and time-intelligence calculations.

---

## Key Measures

### Total Oil Production

```DAX
Total Oil Production (mill Sm3) =
SUM(Production[Oil_mill_Sm3])
```

### Total Oil Barrels

```DAX
Total Oil Barrels =
[Total Oil Production (mill Sm3)] * 1000000 * 6.2898
```

### Brent Price

```DAX
Brent Price =
AVERAGE(Brent_price[Brent_USD_per_bbl])
```

### Estimated Gross Oil Value

```DAX
Estimated Gross Oil Value USD =
SUMX(
    VALUES(Date_Table[Date]),
    CALCULATE([Total Oil Barrels]) *
    CALCULATE([Brent Price])
)
```

This measure estimates the market value of oil production by multiplying monthly oil volumes by the corresponding monthly Brent price.

It should not be interpreted as reported company revenue.

## Year-over-Year Measures

To evaluate how production, Brent prices, and estimated gross oil value changed over time, several year-over-year measures were created.

### Estimated Gross Oil Value – Previous Year

```DAX
Estimated Gross Oil Value USD PY =
CALCULATE(
    [Estimated Gross Oil Value USD],
    SAMEPERIODLASTYEAR(Date_Table[Date])
)
```

### Oil Value YoY Change USD

```DAX
Oil Value YoY Change USD =
VAR PY = [Estimated Gross Oil Value USD PY]
RETURN
IF(
    ISBLANK(PY),
    BLANK(),
    [Estimated Gross Oil Value USD] - PY
)
```

### Oil Value YoY Change %

```DAX
Oil Value YoY Change % =
DIVIDE(
    [Oil Value YoY Change USD],
    [Estimated Gross Oil Value USD PY]
)
```

### Oil Production – Previous Year

```DAX
Oil Production PY =
CALCULATE(
    [Total Oil Production (mill Sm3)],
    SAMEPERIODLASTYEAR(Date_Table[Date])
)
```

### Oil Production YoY %

```DAX
Oil Production YoY % =
DIVIDE(
    [Total Oil Production (mill Sm3)] - [Oil Production PY],
    [Oil Production PY]
)
```

### Brent Price – Previous Year

```DAX
Brent Price PY =
CALCULATE(
    [Brent Price],
    SAMEPERIODLASTYEAR(Date_Table[Date])
)
```

### Brent Price YoY %

```DAX
Brent Price YoY % =
DIVIDE(
    [Brent Price] - [Brent Price PY],
    [Brent Price PY]
)
```

These measures were used to compare changes in estimated oil value with changes in physical production and Brent crude prices.

Because the analysis begins in 2021, year-over-year measures are blank for 2021 because 2020 data are not included in the model.

## Price–Volume Decomposition

To understand what caused year-over-year changes in estimated gross oil value, the total change was decomposed into three components:

- **Volume Effect**
- **Price Effect**
- **Interaction Effect**

The decomposition follows the identity:

**Change in Oil Value = Volume Effect + Price Effect + Interaction Effect**

### Volume Effect

This measures the impact of changes in production volume while holding price at the previous-year level.

Conceptually:

**Volume Effect = Change in Volume × Previous-Year Price**

```DAX
Volume Effect USD =
SUMX(
    VALUES(Date_Table[Date]),
    VAR CurrentVolume =
        CALCULATE([Total Oil Barrels])
    VAR PreviousVolume =
        CALCULATE(
            [Total Oil Barrels],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    VAR PreviousPrice =
        CALCULATE(
            [Brent Price],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    RETURN
        IF(
            ISBLANK(PreviousVolume) || ISBLANK(PreviousPrice),
            BLANK(),
            (CurrentVolume - PreviousVolume) * PreviousPrice
        )
)
```

### Price Effect

This measures the impact of Brent-price changes while holding production at the previous-year level.

Conceptually:

**Price Effect = Previous-Year Volume × Change in Price**

```DAX
Price Effect USD =
SUMX(
    VALUES(Date_Table[Date]),
    VAR PreviousVolume =
        CALCULATE(
            [Total Oil Barrels],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    VAR CurrentPrice =
        CALCULATE([Brent Price])
    VAR PreviousPrice =
        CALCULATE(
            [Brent Price],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    RETURN
        IF(
            ISBLANK(PreviousVolume) || ISBLANK(PreviousPrice),
            BLANK(),
            PreviousVolume * (CurrentPrice - PreviousPrice)
        )
)
```

### Interaction Effect

This captures the additional effect created when both production volume and Brent price change simultaneously.

Conceptually:

**Interaction Effect = Change in Volume × Change in Price**

```DAX
Interaction Effect USD =
SUMX(
    VALUES(Date_Table[Date]),
    VAR CurrentVolume =
        CALCULATE([Total Oil Barrels])
    VAR PreviousVolume =
        CALCULATE(
            [Total Oil Barrels],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    VAR CurrentPrice =
        CALCULATE([Brent Price])
    VAR PreviousPrice =
        CALCULATE(
            [Brent Price],
            SAMEPERIODLASTYEAR(Date_Table[Date])
        )
    RETURN
        IF(
            ISBLANK(PreviousVolume) || ISBLANK(PreviousPrice),
            BLANK(),
            (CurrentVolume - PreviousVolume) *
            (CurrentPrice - PreviousPrice)
        )
)
```

### Decomposition Validation

A reconciliation measure was used to verify that the three effects fully explain the year-over-year change in estimated gross oil value.

```DAX
Decomposition Difference =
[Oil Value YoY Change USD]
-
(
    [Volume Effect USD]
    + [Price Effect USD]
    + [Interaction Effect USD]
)
```

The decomposition difference was **0 for each analysed year**, confirming that the price, volume, and interaction effects reconciled with the total year-over-year change.

## Key Findings

### 1. Brent-price movements were the main driver of estimated oil-value changes

Across the analysed period, changes in Brent prices generally had a larger financial impact than changes in physical oil production.

Production changes still mattered because they either partially offset or amplified the price effect.

### 2. 2022 was strongly price-driven

Estimated gross oil value increased substantially in 2022.

The decomposition showed that the positive price effect was much larger than the production effect.

Although production declined, stronger Brent prices more than compensated for the lower production volume.

### 3. Higher production did not prevent the 2023 value decline

In 2023, overall oil production increased modestly, but estimated gross oil value declined.

The positive production effect was not large enough to offset the negative Brent-price effect.

This illustrates that higher production does not necessarily result in higher estimated value when market prices weaken.

### 4. 2024 showed a weaker contribution from both drivers

In 2024, both production and price effects were slightly negative.

As a result, estimated gross oil value declined modestly compared with the previous year.

### 5. Production helped cushion the 2025 decline

In 2025, production made a positive contribution to estimated gross oil value.

However, the negative Brent-price effect was substantially larger, leading to an overall decline in estimated value.

### 6. Johan Sverdrup was important during the weaker-price environment

The field-level analysis for 2023 showed that Johan Sverdrup combined:

- strong production growth
- positive estimated value growth
- a much larger estimated gross value than most other high-growth fields

This made Johan Sverdrup especially important in cushioning the effect of weaker Brent prices.

### 7. Percentage growth alone can be misleading

Some smaller fields recorded very high percentage increases in production.

However, their overall financial contribution remained relatively small compared with larger producing fields.

For this reason, the field-resilience analysis considered both:

- percentage growth
- estimated gross oil value

This provides a more meaningful view of financial materiality.

## Dashboard Pages

### 1. Norwegian Petroleum Financial Overview

This page provides the overall financial and operational view of Norwegian oil production from 2021 to 2025.

It includes:

- Total Oil Production
- Total Oil Barrels
- Average Brent Price
- Estimated Gross Oil Value
- Oil Value YoY Change %
- Oil Production YoY %
- Brent Price YoY %
- Price vs. Production Contribution waterfall
- Decomposition Matrix
- Top fields by estimated gross oil value

The purpose of this page is to show how market-price movements and production changes influenced estimated gross oil value over time.

### 2. Field Resilience 2023

This page focuses on field-level performance during the weaker Brent-price environment in 2023.

The scatter plot compares:

- **Oil Production YoY %** on the X-axis
- **Field Oil Value YoY %** on the Y-axis
- **Estimated Gross Oil Value** as bubble size

The chart helps identify which fields:

- increased production and value
- increased production but still lost value
- experienced declines in both production and value
- remained financially important despite weaker prices

The accompanying field-level table provides detailed performance values for individual fields.

This page highlights how large producers such as Johan Sverdrup could materially cushion weaker price conditions through stronger production growth.

## Limitations

The estimated gross oil-value measure used in this project is a simplified market-value estimate.

It is calculated using:

**Oil Production × Brent Price**

The measure does not represent reported company revenue or profit.

It does not account for:

- company ownership shares
- realised selling prices
- crude-quality differences
- hedging
- transportation costs
- operating expenses
- taxes
- royalties
- other commercial adjustments

For this reason, the results should be interpreted as an estimate of gross market value rather than accounting revenue or profitability.

The analysis also uses Brent crude as a common benchmark price for all fields, even though actual realised prices may differ across crude grades and commercial arrangements.

## Skills Demonstrated

This project demonstrates practical experience in:

- Power BI
- Power Query
- DAX
- Data Cleaning
- Data Transformation
- Data Modelling
- Time-Intelligence Analysis
- Financial Analysis
- Price–Volume Decomposition
- Data Visualisation
- Dashboard Design
- Data Storytelling
- Energy-Sector Analytics
