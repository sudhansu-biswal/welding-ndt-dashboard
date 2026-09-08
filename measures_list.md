# Measures List — Welding & NDT Quality Dashboard

All measures live in the `_Measures` table. The `Column1` placeholder field in that table is hidden from report view.

Model: `Welding_Inspection_Fact` (2,100 rows) related 1:* from `DateTable[Date]` to `Welding_Inspection_Fact[Inspection_Date]`, single cross-filter direction. `DateTable` is marked as a date table.

---

## Base counts

### Total Joints

```dax
Total Joints = COUNTROWS(Welding_Inspection_Fact)
```

Counts every inspected weld joint in the current filter context. Format: `#,##0`. Unfiltered value: 2,100.

### Rejected Joints

```dax
Rejected Joints =
CALCULATE(
    COUNTROWS(Welding_Inspection_Fact),
    Welding_Inspection_Fact[Result] = "Reject"
)
```

Counts joints that failed inspection. `CALCULATE` applies the Result filter on top of whatever slicers are active. Format: `#,##0`. Unfiltered value: 349.

### Accepted Joints

```dax
Accepted Joints = [Total Joints] - [Rejected Joints]
```

Derived by subtraction rather than a second `CALCULATE`, so it can never disagree with the other two. Format: `#,##0`.

---

## Rate

### Rejection Rate %

```dax
Rejection Rate % = DIVIDE([Rejected Joints], [Total Joints], 0)
```

The headline KPI. `DIVIDE` is used instead of the `/` operator so that a filter combination returning no rows yields 0 rather than an error — this happens as soon as a user slices to, say, a contractor–material pair that does not exist. Format: `0.0%`. Unfiltered value: 16.6%.

---

## Cost and time

### Total Repair Cost

```dax
Total Repair Cost = SUM(Welding_Inspection_Fact[Repair_Cost_INR])
```

Sum of rework cost. The source column is typed as Fixed Decimal Number, which stores exactly four decimal places and avoids floating-point rounding drift across 2,100 rows. Format: `₹#,##0`. Unfiltered value: ₹17,52,558.

### Total Repair Hours

```dax
Total Repair Hours = SUM(Welding_Inspection_Fact[Repair_Hours])
```

Labour hours consumed by rework. Format: `#,##0`. Unfiltered value: 1,845.

### Avg Repair Cost per Reject

```dax
Avg Repair Cost per Reject = DIVIDE([Total Repair Cost], [Rejected Joints], 0)
```

Cost intensity per failure, as distinct from total spend. Distinguishes a contractor with many cheap repairs from one with few expensive ones. Format: `₹#,##0`. Unfiltered value: ₹5,036.

---

## Pareto analysis

These two measures drive the welder Pareto chart on page 2. They are the least self-explanatory DAX in the model, so read the notes before modifying them.

### Cumulative Repair Cost

```dax
Cumulative Repair Cost =
VAR CurrentCost = [Total Repair Cost]
VAR AllWelders =
    ALLSELECTED(Welding_Inspection_Fact[Welder_ID])
VAR RankedTable =
    ADDCOLUMNS(AllWelders, "@Cost", [Total Repair Cost])
RETURN
    SUMX(
        FILTER(RankedTable, [@Cost] >= CurrentCost),
        [@Cost]
    )
```

For the welder in the current row of a visual, this sums the repair cost of that welder **and every welder whose cost is greater than or equal to theirs**.

- `ALLSELECTED` removes the row-level welder filter but keeps any slicer or visual-level filter the user has applied, so the cumulative total respects a contractor slicer selection.
- `ADDCOLUMNS` materialises a virtual table of every welder with their cost attached.
- `FILTER` keeps only the rows at or above the current welder's cost; `SUMX` totals them.

The measure is order-independent — it ranks internally by cost, so the result is correct regardless of how the visual is sorted. Format: `₹#,##0`.

### Cumulative Cost %

```dax
Cumulative Cost % =
DIVIDE(
    [Cumulative Repair Cost],
    CALCULATE([Total Repair Cost], ALLSELECTED(Welding_Inspection_Fact[Welder_ID])),
    0
)
```

Expresses the running total as a share of the whole. The denominator uses the same `ALLSELECTED` scope as the numerator, so both respond identically to slicers — a mismatch here is the commonest way a Pareto measure silently breaks.

Format: `0.0%`. Validation: reaches exactly 100.0% on the final (lowest-cost) welder, confirming no rows are double-counted or dropped.

---

## Validation checks

Run these after any model change. All values are unfiltered.

| Check | Expected |
|---|---|
| `Total Joints` | 2,100 |
| `Rejected Joints` + `Accepted Joints` | 2,100 |
| `Rejection Rate %` | 16.6% |
| `Total Repair Cost` | ₹17,52,558 |
| `Cumulative Cost %` on the lowest-cost welder | 100.0% |
| `Cumulative Cost %` on the 3rd-ranked welder | 25.8% |

---

## Notes on formatting

All formats are set at the measure level (Measure tools → Format), not per visual. Visual-level overrides were deliberately avoided so that any new visual inherits the correct format automatically.

`Total Joints` required the custom format string `#,##0` to prevent the card visual abbreviating 2,100 to "2K".
