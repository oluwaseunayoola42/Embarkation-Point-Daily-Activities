# Analysis Notes: Checks and Definitions

Working notes behind the [README](README.md). Figures come from the two dashboard screenshots. Anything marked *confirm* should be checked in the Power BI model before it is quoted as final.

## 1. Partial months (now using real totals)

The window is 16 July to 17 November 2021, which is 125 days. July and November are partial months.

| Month | Embarked | Days | Per day | Share of September's daily rate |
|---|---|---|---|---|
| July (16 to 31) | 17,114 | 16 | 1,070 | 56% |
| August | 45,036 | 31 | 1,453 | 76% |
| September | 57,584 | 30 | 1,919 | 100% |
| October | 48,098 | 31 | 1,552 | 81% |
| November (1 to 17) | 26,039 | 17 | 1,532 | 80% |

The five months add up to 193,871, which matches the headline total. The raw comparison "September is 3.4x July" becomes about 1.8x per day.

## 2. Swipe-to-embark reconciliation by point

| Point | Embarked | Outbound | Inbound | Swiped | Gap | Gap % |
|---|---|---|---|---|---|---|
| BRPH | 114,166 | 66,169 | 47,329 | 113,498 | 668 | 0.6% |
| ESBL | 26,429 | 13,656 | 11,969 | 25,625 | 804 | 3.0% |
| QIEK | 23,253 | 11,763 | 11,690 | 23,453 | -200 | -0.9% |
| Everything else | 30,023 | | | 28,211 | 1,812 | 6.0% |

"Everything else" is the total minus the three rows above (193,871 - 163,848 embarked; 190,787 - 162,576 swiped). Several outbound labels for the smaller points are cut off in the screenshot, so they are not listed individually.

**Reading it:** at the three biggest points, swipes in both directions come within about 3% of embarked movements, which supports the assumption that outbound plus inbound should equal embarked. On that basis, 59% of the total gap sits in points that carry 15.5% of the volume.

**To check:**
1. What "embarked" counts (both directions, or outbound only).
2. The gap for each of the smaller points.
3. Whether the gap clusters on certain dates, shifts or card types.
4. Why QIEK has more swipes than embarked movements (duplicate swipes, re-entries, or a timing difference).

## 3. Compliance flags

2,175 + 841 + 351 = 3,367, about 1.74% of 193,871 movements. The donut legend is truncated in the screenshot ("Personnel Wit...", "Count of Pers..."), so which count belongs to which category follows the chart title's order and needs confirming. The blue measure is named differently from the other two, which is worth checking: make sure all three are calculated the same way (rows versus distinct people).

**To check:**
1. Whether flags are mutually exclusive.
2. Flag rate by point and by month.
3. Whether flagged movements were allowed through or stopped.

## 4. "Personnel" versus "movements"

193,871 counts movements. If the model has a person ID, add a distinct count of personnel and report both.

## 5. Clearing sources

| Source | Movements | Share of all movements | Share of the five shown |
|---|---|---|---|
| Lagos Office | 22,771 | 11.7% | 63.5% |
| Officer D | 7,640 | 3.9% | 21.3% |
| Officer C | 4,736 | 2.4% | 13.2% |
| Officer B | 487 | 0.3% | 1.4% |
| Officer A | 213 | 0.1% | 0.6% |
| **Total shown** | **35,847** | **18.5%** | 100% |

Officers are anonymised here. The five sources cover only 18.5% of movements, so ask what clears the rest. "Lagos Office" is an office and the others are individuals, so compare offices with offices if more data is available, and find out how many people work in the Lagos Office.

## 6. Government agency view

The treemap is titled "Total Personnel Embarked by Government Agency", and the table below it has columns named GA_DEPARTURE DATE and GA_OFFSHORE LOCATION. That suggests it counts **agency personnel travelling offshore**, which is different from "clearances granted by an agency". *Confirm which one it is.*

Listed tiles add up to about 44,700 movements (23%): NIG Customs 19,376, NIS 13,489, PHS 4,923, a Federal Ministry body 2,298, Nigerian Navy 1,891, NOSDRA 1,341, NIG... 775, NPF 628, plus small unlabelled tiles.

## 7. Open question: "Total Captured" = 6,278

This card appears on the overview page next to Total Embarked (193,871) and Total Swiped (190,787) with no label explaining it. It is not equal to the 3,084 gap. *Confirm what it measures*, then add a definition or remove it.

## 8. Dashboard design notes

- Several data labels and legends are truncated (outbound values for smaller points, donut legend).
- The embarkation-point chart scrolls, so smaller points are hidden. A table or a combined chart would show all of them.
- Treemap tiles for the smaller agencies are unreadable. A sorted bar chart would work better.
- KPI numbers have no thousands separators (193871).
- Section headers use many different colours. One or two colours would look more consistent.
- Adding a daily-rate card would stop the partial-month problem misleading readers.

## DAX starting points

Rename tables and columns to match your model. These are templates, not tested against your file.

```dax
Movements = COUNTROWS ( Movements )

Days In Selection = DISTINCTCOUNT ( 'Date'[Date] )

Movements Per Day = DIVIDE ( [Movements], [Days In Selection] )

Compliance Flagged =
CALCULATE ( [Movements], Movements[ComplianceFlag] <> "OK" )

Compliance Flag Rate = DIVIDE ( [Compliance Flagged], [Movements] )

Swipes Total = [Outbound Swipes] + [Inbound Swipes]

Swipe Gap = [Movements] - [Swipes Total]

Swipe Gap % = DIVIDE ( [Swipe Gap], [Movements] )

Share Of Attributed Clearances =
DIVIDE (
    [Movements],
    CALCULATE ( [Movements], ALLSELECTED ( Movements[ClearedBy] ) )
)
```
