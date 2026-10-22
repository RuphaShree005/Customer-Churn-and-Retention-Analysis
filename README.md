# Customer-Churn-and-Retention-Analysis
Analyzed IBM’s Telco Customer Churn data in Power BI, using Power Query and DAX to identify churn patterns across customer tenure, contracts, payment methods, and services.
A Power BI project analyzing which customer groups have higher churn rates and how churn is associated with customer characteristics.

## Business question

Which customer groups have the highest churn, and how is churn associated with tenure, contract type, payment method, internet service, and monthly charges?

## Tools

- Power BI Desktop
- Power Query
- DAX

## Dataset

This project uses IBM’s public [Telco Customer Churn dataset](https://github.com/IBM/telco-customer-churn-on-icp4d).

The dataset is a customer snapshot. The analysis compares customer groups; it does not present a monthly churn trend.

## Project workflow

### 1. Import and clean the data

1. Download `Telco-Customer-Churn.csv` from IBM’s repository.
2. In Power BI Desktop, select **Home → Get data → Text/CSV → Transform Data**.
3. Rename the query to `Telco`.
4. Set `customerID`, `Churn`, `Contract`, `PaymentMethod`, and service fields to **Text**.
5. Set `tenure` to **Whole Number** and `MonthlyCharges` and `TotalCharges` to **Decimal Number**.
6. Check `TotalCharges` for blanks or conversion errors. Replace blank values with null before setting the column type. Do not replace blanks with zero unless that choice is justified.
7. Select **Home → Close & Apply**.

### 2. Create tenure groups

In Power Query, select the `Telco` query and choose **Add Column → Conditional Column**. Name the new column `Tenure Group` and use these rules in order:

| Condition | Tenure Group |
|---|---|
| `tenure` is less than or equal to 12 | `0–12 months` |
| `tenure` is less than or equal to 24 | `13–24 months` |
| `tenure` is less than or equal to 48 | `25–48 months` |
| Otherwise | `49+ months` |

Select **Close & Apply**.

### 3. Create DAX measures

Select the `Telco` table in the Data pane. Choose **New measure** and add each measure separately:

```dax
Customers =
DISTINCTCOUNT(Telco[customerID])
```

```dax
Churned Customers =
CALCULATE(
    [Customers],
    Telco[Churn] = "Yes"
)
```

```dax
Churn Rate =
DIVIDE([Churned Customers], [Customers], 0)
```

```dax
Retained Customers =
CALCULATE(
    [Customers],
    Telco[Churn] = "No"
)
```

```dax
Retention Rate =
DIVIDE([Retained Customers], [Customers], 0)
```

```dax
Monthly Charges of Churned Customers =
CALCULATE(
    SUM(Telco[MonthlyCharges]),
    Telco[Churn] = "Yes"
)
```

```dax
Average Tenure of Churned Customers =
CALCULATE(
    AVERAGE(Telco[tenure]),
    Telco[Churn] = "Yes"
)
```

Format `Churn Rate` and `Retention Rate` as percentages. Format monthly charges as currency and average tenure as a number of months.

`Monthly Charges of Churned Customers` is the monthly charges associated with customers marked as churned. It should not be described as revenue lost.

### 4. Build the report pages

#### Page 1 — Churn Overview

- Cards: `Customers`, `Churned Customers`, `Churn Rate`, `Retention Rate`
- Bar chart: customer count by `Contract`, with `Churn` as the legend
- Bar chart: `Churn Rate` by `Tenure Group`
- Slicers: `Contract`, `Internet Service`, and `Payment Method`

#### Page 2 — Customer Segments

- Bar chart: `Churn Rate` by `Contract`
- Bar chart: `Churn Rate` by `Payment Method`
- Bar chart: `Churn Rate` by `Tenure Group`
- Cards: `Average Tenure of Churned Customers` and `Monthly Charges of Churned Customers`

#### Page 3 — Customer Detail

- Table columns: `customerID`, `tenure`, `Contract`, `MonthlyCharges`, `TotalCharges`, and `Churn`
- Slicers: `Contract`, `Tenure Group`, and `Internet Service`

Use **Format → Edit interactions** to confirm that slicers filter the visuals you intend.

## Interpretation

Describe differences between customer groups as associations. This snapshot dataset can show which groups have higher churn in the data, but it does not establish why customers left or prove that a customer characteristic caused churn.

## Suggested repository structure

```text
customer-churn-analysis/
├── data/
│   └── README.md
├── powerbi/
│   └── customer_churn_retention.pbix
├── screenshots/
│   └── dashboard-overview.png
└── README.md
```

If the dataset file is not included in the repository, explain in `data/README.md` where it can be downloaded.

## Resume summary

Analyzed IBM’s Telco Customer Churn dataset in Power BI to compare churn and retention across contract, tenure, payment, and service segments. Cleaned customer and billing fields in Power Query, created DAX measures, and designed report pages for churn overview, segment comparison, and customer-level detail.
