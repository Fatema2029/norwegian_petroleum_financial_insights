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

