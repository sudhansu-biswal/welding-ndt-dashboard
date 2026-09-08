
# Welding & NDT Quality Dashboard

**Identifying where rework cost concentrates in a pipe fabrication programme**

Power BI · Power Query · DAX · 2,100 weld joints · 12 months

---

## The problem

A fabrication contractor inspecting thousands of weld joints a year knows its overall rejection rate. What it usually cannot answer is *where the rejection rate comes from* — which welders, which weld positions, which materials, and what that costs.

Inspection data sits in spreadsheets and NDT report logs, recorded joint by joint. It is complete, but it is not analysed. The result is that rework budget gets treated as a fixed cost of doing business rather than something with identifiable, addressable causes.

This project takes a 12-month inspection log — 2,100 joints, 28 welders, 3 contractors, 7 plant areas — and turns it into a four-page decision tool.

---

## What the data showed

### Rework cost is concentrated in three people

Three welders performed 214 joints, **10.2% of total volume**, but generated **₹4,52,469 in repair cost — 25.8% of all rework spend**.

Their individual rejection rates were 49.3%, 39.1% and 38.2%, against a site average of 16.6%. Critically, their workloads were normal: 69, 69 and 76 joints against a 75-joint average. The rates are not a small-sample artefact.

**Action indicated:** requalification testing for three named welders before further deployment. Not a site-wide retraining programme.

### Weld position drives failure; inspection method does not

A cross-tabulation of rejection rate by weld position and NDT method produced a clear gradient:

| Position | Rejection rate |
|---|---|
| 1G | 14.0% |
| 2G | 14.3% |
| 3G | 15.6% |
| 5G | 19.1% |
| **6G** | **25.5%** |

Column totals by NDT method ranged only from 15.0% to 17.8% — a two-point spread. Method is not a driver. Position is, and the pattern is mechanically consistent: 6G is the fixed all-position pipe weld with the hardest root access, and it also shows the largest share of Incomplete Penetration defects.

**Action indicated:** position-specific procedure review and welder qualification weighting, rather than changes to the inspection regime.

### Porosity leads the defect profile

Across 349 rejections, Porosity was the most frequent defect type, ahead of Lack of Fusion and Undercut. Porosity points to gas shielding, consumable storage or surface contamination — process controls, not individual skill.

**Action indicated:** consumable handling audit and shielding gas verification.

### Contractor selection is not the problem

| Contractor | Joints | Rejection rate | Repair cost | Cost per reject |
|---|---|---|---|---|
| Anchor Piping Works | 661 | 15.9% | ₹5,31,118 | ₹5,058 |
| Meridian Fabricators | 748 | 16.8% | ₹6,52,006 | ₹5,175 |
| Delta Mech Services | 691 | 16.9% | ₹5,69,434 | ₹4,867 |

A one-point spread in rejection rate and a ₹308 spread in cost per reject, across roughly 700 joints each. Monthly trend lines cross repeatedly with no persistent leader.

**Ruling something out is a finding.** No contractor-level intervention is warranted, and any effort spent renegotiating or reallocating between these three would not move the rework number.

### Shift is marginal

Night shift ran at 17.6% against day shift at 16.1%. A 1.5-point difference is worth monitoring but does not justify a staffing change on its own.

---

## What was built

**Four pages, each answering one question:**

1. **Overview** — headline KPIs, monthly rejection trend against an average reference line, rejection rate by weld position
2. **Welder Performance** — Pareto chart of repair cost by welder with a cumulative percentage curve, top-3 cost driver table with conditional formatting, defect type breakdown
3. **Defect Analysis** — weld position × NDT method heat map, material and shift comparison, defect composition by position
4. **Contractor Comparison** — contractor scorecard, rejection rate comparison, monthly trend by contractor, with shift/material/area slicers

**Technical approach:**

- Source CSV loaded through a parameterised path so the report survives folder moves and data refreshes
- All 16 columns explicitly typed in Power Query; auto-detection disabled to avoid sampling errors on a 2,100-row file
- Currency stored as Fixed Decimal to prevent floating-point drift when summed
- A DAX-generated date table, dynamically bounded by the data range and marked as a date table, so time intelligence resolves correctly
- Nine measures in a dedicated `_Measures` table, including a `SUMX`/`FILTER` Pareto calculation that respects slicer context via `ALLSELECTED`
- Measure-level formatting throughout, so new visuals inherit correct display automatically

---

## Why it matters commercially

The dashboard converts a rework budget into a set of specific, costed decisions:

- **₹4,52,469** is attributable to three named individuals — a targeted intervention, not a programme
- **1,845 labour hours** of rework is quantified, which converts the cost figure into schedule impact
- **Contractor renegotiation is ruled out**, saving effort that would have produced nothing
- **6G procedure review** is identified as the highest-leverage process change

For a QA manager reporting upward, these are the four lines that go in the monthly report.

---

## Domain note

This analysis was built by an engineer with twelve years of site-based QA/QC experience across refinery and steel plant projects, holding ASNT Level II certification in RT, UT, MT and PT. The interpretation of weld position difficulty, defect aetiology and NDT method selection reflects field practice, not just the numbers.

---

*Dataset is synthetic, modelled on real fabrication inspection log structure. Full PBIX, data dictionary and measure documentation available on request.*
