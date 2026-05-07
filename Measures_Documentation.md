# Swiss Dashboard — Measures Documentation

> **Model:** Swiss_Dashboard  
> **Table:** `__MeasureTable`  
> **Total Measures:** 21  
> **Generated:** 2026-03-08

---

## Utility

### Species Image URL

| Property | Value |
|----------|-------|
| **Description** | Returns the embedded image for the current animal species from the Picture column. Used for image rendering in visuals. |
| **Format String** | *(text)* |
| **Data Type** | String |
| **Hidden** | No |

```dax
MAX(Animals[Picture])
```

---

## Core Totals

### Total Consumption (kg)

| Property | Value |
|----------|-------|
| **Description** | Total amount consumed in kilograms across all activities. |
| **Format String** | `#,##0.00` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
SUM(Fact[Quantity_Consumed_Kg])
```

### Total Prey Units

| Property | Value |
|----------|-------|
| **Description** | Total number of prey units across all activities. |
| **Format String** | `#,##0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
SUM(Fact[Quantity_Prey_Units])
```

### Total Activities

| Property | Value |
|----------|-------|
| **Description** | Total number of recorded activities (rows in Fact table). |
| **Format String** | `#,##0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
COUNTROWS(Fact)
```

### Successful Hunts

| Property | Value |
|----------|-------|
| **Description** | Count of activities where the hunt was successful (`Success_Flag = 1`). |
| **Format String** | `#,##0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
CALCULATE(COUNTROWS(Fact), Fact[Success_Flag] = 1)
```

### Distinct Animals

| Property | Value |
|----------|-------|
| **Description** | Count of distinct animal species observed in the filtered context. |
| **Format String** | `#,##0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
DISTINCTCOUNT(Fact[Animal_Key])
```

---

## Rates & Ratios

### Success Rate %

| Property | Value |
|----------|-------|
| **Description** | Percentage of activities that were successful (Successful Hunts / Total Activities). |
| **Format String** | `0.0%` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
DIVIDE([Successful Hunts], [Total Activities], 0)
```

### Avg Consumption per Activity

| Property | Value |
|----------|-------|
| **Description** | Average consumption in kg per activity (Total Consumption / Total Activities). |
| **Format String** | `#,##0.00` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
DIVIDE([Total Consumption (kg)], [Total Activities], 0)
```

### Avg Animal Weight (kg)

| Property | Value |
|----------|-------|
| **Description** | Average body weight (kg) of animal species in the current filter context. |
| **Format String** | `#,##0.0` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
AVERAGE(Animals[Avg_Weight_kg])
```

### Consumption per Prey Unit

| Property | Value |
|----------|-------|
| **Description** | Average consumption in kg per prey unit (Total Consumption / Total Prey Units). |
| **Format String** | `#,##0.00` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
DIVIDE([Total Consumption (kg)], [Total Prey Units], 0)
```

### Weight-Normalized Consumption Index

| Property | Value |
|----------|-------|
| **Description** | Average consumption per activity divided by species average body weight. |
| **Format String** | `0.000` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR TotalConsumption = [Total Consumption (kg)]
VAR AvgWeight = [Avg Animal Weight (kg)]
VAR TotalActs = [Total Activities]
RETURN
    DIVIDE(
        DIVIDE(TotalConsumption, TotalActs, BLANK()),
        AvgWeight,
        BLANK()
    )
```

### Weighted Success Rate (by Prey)

| Property | Value |
|----------|-------|
| **Description** | Proportion of total prey units from successful hunts. |
| **Format String** | `0.0%` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR SuccessfulPrey =
    CALCULATE(
        SUM(Fact[Quantity_Prey_Units]),
        Fact[Success_Flag] = 1
    )
VAR TotalPrey = [Total Prey Units]
RETURN
    DIVIDE(SuccessfulPrey, TotalPrey, BLANK())
```

---

## Time Intelligence

### YoY Consumption Growth %

| Property | Value |
|----------|-------|
| **Description** | Year-over-year growth in total consumption (kg), expressed as a percentage. |
| **Format String** | `0.0%` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR CurrentYearValue = [Total Consumption (kg)]
VAR PriorYearValue = CALCULATE([Total Consumption (kg)], SAMEPERIODLASTYEAR('Date'[FullDate]))
RETURN DIVIDE(CurrentYearValue - PriorYearValue, PriorYearValue, 0)
```

### Running Total Consumption (kg)

| Property | Value |
|----------|-------|
| **Description** | Cumulative total of consumption (kg) from the earliest date up to the current date in context. |
| **Format String** | `#,##0.00` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR CurrentDate = MAX('Date'[FullDate])
RETURN
    CALCULATE(
        [Total Consumption (kg)],
        'Date'[FullDate] <= CurrentDate,
        ALL('Date')
    )
```

### 30-Day Moving Avg Consumption (kg)

| Property | Value |
|----------|-------|
| **Description** | Average daily consumption (kg) over the trailing 30-day window ending on the current date. |
| **Format String** | `#,##0.00` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR CurrentDate = MAX('Date'[FullDate])
VAR WindowStart = CurrentDate - 29
RETURN
    CALCULATE(
        DIVIDE(
            [Total Consumption (kg)],
            DISTINCTCOUNT('Date'[FullDate])
        ),
        FILTER(
            ALL('Date'),
            'Date'[FullDate] >= WindowStart && 'Date'[FullDate] <= CurrentDate
        )
    )
```

### Seasonal Activity Index

| Property | Value |
|----------|-------|
| **Description** | Index comparing current activity count to average monthly activity (100 = average). |
| **Format String** | `#,##0.0` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR CurrentMonthActivities = [Total Activities]
VAR OverallMonthlyAvg =
    AVERAGEX(
        SUMMARIZE(
            ALL(Fact),
            'Date'[Year],
            'Date'[Month_Number]
        ),
        CALCULATE([Total Activities])
    )
RETURN
    DIVIDE(CurrentMonthActivities, OverallMonthlyAvg, BLANK()) * 100
```

---

## Ranking & Distribution

### Species Consumption Rank

| Property | Value |
|----------|-------|
| **Description** | Dense rank of the current species by total consumption (kg), where rank 1 = highest consumer. |
| **Format String** | `0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
IF(
    HASONEVALUE(Animals[Species_Name]),
    RANKX(
        ALL(Animals[Species_Name]),
        [Total Consumption (kg)],
        ,
        DESC,
        Dense
    )
)
```

### Prey Diversity Index (Shannon)

| Property | Value |
|----------|-------|
| **Description** | Shannon-Wiener diversity index (H') of prey species. |
| **Format String** | `0.000` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR TotalEvents = [Total Activities]
VAR PreyTable =
    ADDCOLUMNS(
        VALUES(Fact[Prey_Species_Hunted]),
        "@Count", CALCULATE(COUNTROWS(Fact)),
        "@Proportion", DIVIDE(CALCULATE(COUNTROWS(Fact)), TotalEvents)
    )
VAR ShannonIndex =
    -SUMX(
        FILTER(PreyTable, [@Proportion] > 0),
        [@Proportion] * LN([@Proportion])
    )
RETURN
    IF(TotalEvents > 0, ShannonIndex, BLANK())
```

### Pareto Cumulative Consumption %

| Property | Value |
|----------|-------|
| **Description** | Cumulative % of total consumption when species are ranked highest to lowest. |
| **Format String** | `0.0%` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
IF(
    HASONEVALUE(Animals[Species_Name]),
    VAR CurrentRank =
        RANKX(
            ALL(Animals[Species_Name]),
            [Total Consumption (kg)],
            ,
            DESC,
            Dense
        )
    VAR CumulativeConsumption =
        CALCULATE(
            [Total Consumption (kg)],
            FILTER(
                ALL(Animals[Species_Name]),
                RANKX(
                    ALL(Animals[Species_Name]),
                    [Total Consumption (kg)],
                    ,
                    DESC,
                    Dense
                ) <= CurrentRank
            )
        )
    VAR GrandTotal =
        CALCULATE(
            [Total Consumption (kg)],
            ALL(Animals[Species_Name])
        )
    RETURN
        DIVIDE(CumulativeConsumption, GrandTotal, BLANK())
)
```

---

## Composite Scores

### Habitat Efficiency Score

| Property | Value |
|----------|-------|
| **Description** | Composite score (0–100) rating each location's efficiency. Weighted: 40% success rate, 35% avg consumption, 25% prey diversity — each normalized against the max across all locations. |
| **Format String** | `#,##0.0` |
| **Data Type** | Double |
| **Hidden** | No |

```dax
VAR LocSuccessRate = [Success Rate %]
VAR LocAvgConsumption = [Avg Consumption per Activity]
VAR LocActivities = [Total Activities]
VAR LocPreyDiversity =
    VAR TotalEvents = [Total Activities]
    VAR PreyTable =
        ADDCOLUMNS(
            VALUES(Fact[Prey_Species_Hunted]),
            "@Prop", DIVIDE(CALCULATE(COUNTROWS(Fact)), TotalEvents)
        )
    RETURN
        -SUMX(FILTER(PreyTable, [@Prop] > 0), [@Prop] * LN([@Prop]))
VAR MaxSuccessRate =
    MAXX(ALL(Location), [Success Rate %])
VAR MaxConsumption =
    MAXX(ALL(Location), [Avg Consumption per Activity])
VAR MaxDiversity =
    MAXX(
        ALL(Location),
        VAR TE = CALCULATE([Total Activities])
        VAR PT =
            ADDCOLUMNS(
                CALCULATETABLE(VALUES(Fact[Prey_Species_Hunted])),
                "@P", DIVIDE(CALCULATE(COUNTROWS(Fact)), TE)
            )
        RETURN -SUMX(FILTER(PT, [@P] > 0), [@P] * LN([@P]))
    )
VAR NormSuccess = DIVIDE(LocSuccessRate, MaxSuccessRate, 0) * 40
VAR NormConsumption = DIVIDE(LocAvgConsumption, MaxConsumption, 0) * 35
VAR NormDiversity = DIVIDE(LocPreyDiversity, MaxDiversity, 0) * 25
RETURN
    IF(
        LocActivities > 0,
        NormSuccess + NormConsumption + NormDiversity
    )
```

### Max Consecutive Hunt Streak

| Property | Value |
|----------|-------|
| **Description** | Longest streak of consecutive successful hunts for the current species. |
| **Format String** | `#,##0` |
| **Data Type** | Int64 |
| **Hidden** | No |

```dax
IF(
    HASONEVALUE(Animals[Species_Name]),
    VAR _SuccessDates =
        CALCULATETABLE(
            DISTINCT(Fact[Date_Key]),
            Fact[Success_Flag] = 1
        )
    VAR _FailDates =
        CALCULATETABLE(
            DISTINCT(Fact[Date_Key]),
            Fact[Success_Flag] = 0
        )
    VAR _MinDate = CALCULATE(MIN(Fact[Date_Key])) - 1
    VAR _StreakEnds =
        ADDCOLUMNS(
            _SuccessDates,
            "@PrevFail",
            VAR _CurrDate = [Date_Key]
            VAR _LastFail = MAXX(FILTER(_FailDates, [Date_Key] < _CurrDate), [Date_Key])
            RETURN IF(ISBLANK(_LastFail), _MinDate, _LastFail)
        )
    VAR _Streaks =
        ADDCOLUMNS(
            _StreakEnds,
            "@Length",
            VAR _Start = [@PrevFail]
            VAR _End = [Date_Key]
            RETURN
                COUNTROWS(FILTER(_SuccessDates, [Date_Key] > _Start && [Date_Key] <= _End))
        )
    RETURN
        MAXX(_Streaks, [@Length])
)
```
