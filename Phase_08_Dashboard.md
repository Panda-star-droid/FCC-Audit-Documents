# 📊 PHASE 08: DASHBOARD - COMPLETE IMPLEMENTATION V6.0

**Project:** Finance Control Center | **Version:** 6.0 - Production Ready | **Date:** October 13, 2025 | **Status:** ✅ Ready for Implementation  
**Deliverables:** Dashboard sheet (view-only integration)  
**Estimated Time:** 5 hours  
**Dependencies:** ALL previous phases (1-7)  
**Critical:** Must be built LAST - integrates all systems

---

## 📋 TABLE OF CONTENTS

1. [Overview](#overview)
2. [Critical Foundations](#critical-foundations)
3. [Income Tracking System](#income-tracking-system)
4. [Dashboard Layout](#dashboard-layout)
5. [Chart Specifications](#chart-specifications)
6. [Universal Chart Standards](#universal-chart-standards)
7. [Data Structures](#data-structures)
8. [Formula Pattern Library](#formula-pattern-library)
9. [Implementation Checklist](#implementation-checklist)

---

## 🎯 OVERVIEW

### Phase Objectives
- ✅ Create comprehensive financial dashboard with 6-7 charts
- ✅ Implement flexible income tracking (Simple + Detailed modes)
- ✅ Enable multi-year tracking across all data
- ✅ Maintain 5-minute setup promise

### Key Features
1. **View-Only Dashboard:** No data entry forms - all transactions via Quick Entry sheet
2. **Progressive Income Tracking:** Simple mode (95% users) + Detailed mode (5% users)
3. **Year-Aware Data:** All tables track Month_ID (YYYY-MM format)
4. **Promotion Handling:** Income changes carry forward indefinitely
5. **6 Core Charts:** Financial health, budget, spending, income, debt, net worth
6. **Chart 7 (Conditional):** Income by source (Detailed mode only)

---

## 🔗 DEPENDENCIES

**Dashboard is view-only (V11):**
- NO Quick Add forms (removed in V11)
- NO data entry
- All entry via Quick Entry sheet (Phase 2)

**Required from Phase 1 (Foundation):**
- `Categories_DB`, `Settings_DB` - For all configurations

**Required from Phase 2 (Transactions):**
- `Transaction_DB` - For recent transactions, spending charts

**Required from Phase 3 (Utilities/Settings):**
- Settings for period selector, currency display

**Required from Phase 4 (Income):**
- `Income_Sources_DB` - For income tracking (Chart 7 if Detailed mode)

**Required from Phase 5 (Budget):**
- `Budget_Allocations_DB`, `Goals_DB` - For budget vs actual, goals widget

**Required from Phase 6 (Debt):**
- `Debt_DB`, `Debt_Payments_DB` - For debt progress chart

**Required from Phase 7 (Net Worth):**
- `Net_Worth_Snapshots_DB` - For net worth trend (if implemented)

---

## 🔧 CRITICAL FOUNDATIONS

### Foundation 1: Month_ID Tracking

**Problem Solved:** Historical data across multiple years

**Implementation:**
- Format: "YYYY-MM" (TEXT)
- Examples: "2025-01", "2025-12", "2026-01"
- All tables include Month_ID column (auto-populated from Date)
- Indexed for fast filtering

**Formula:**
```excel
=TEXT(Date,"YYYY-MM")
Example: 2025-10-13 → "2025-10"
```

**Display Formula:**
```excel
=TEXT(DATEVALUE(Month_ID&"-01"),"MMM YY")
Example: "2025-10" → "Oct 25"
```

---

### Foundation 2: Income Plan History

**Problem Solved:** Promotions/raises carrying forward across years

**New Table: Income_Plan_History** (Settings_DB sheet, Row 50-100)

**Structure:**
| Column | Type | Description |
|--------|------|-------------|
| ID | INT | Unique identifier (auto-increment) |
| Start_Date | DATE | When this amount begins |
| Source_ID | INT | Links to Income_Sources_DB |
| Planned_Amount | DECIMAL | Expected monthly amount |
| Note | TEXT | User explanation (optional) |

**Sample:**
```
{1, 2025-01-01, 1, 3500, "Initial salary"}
{2, 2025-04-01, 1, 4200, "Promotion"}
{3, 2026-03-01, 1, 4800, "Annual raise"}
```

**Key Behavior:**
- No end date (active until next entry)
- Sorted by Start_Date ascending (CRITICAL for LOOKUP)
- LOOKUP finds most recent entry ≤ target month
- Works across unlimited years

**Formula to Get Planned Amount:**
```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Start_Date] <= MonthStartDate), 
    Income_Plan_History[Planned_Amount]),
  Default_Amount)

Where: MonthStartDate = DATEVALUE(Month_ID & "-01")
```

---

## 💰 INCOME TRACKING SYSTEM

### Two-Track Design

**Track 1: SIMPLE MODE (Default - 95% users)**
- Single total income amount
- Setup: 30 seconds
- Budget tab: 4-column table
- Dashboard: 6 charts (no Chart 7)

**Track 2: DETAILED MODE (Opt-in - 5% users)**
- Multiple income sources (up to 8)
- Setup: 2 minutes
- Budget tab: 4-column table + expandable breakdown
- Dashboard: 7 charts (includes Chart 7)

---

### Setup Wizard Integration

**Location:** Settings Tab, Setup Wizard Step 3

**Step 3: Income Setup**

**Radio Options:**
- ◉ **SIMPLE (Recommended)**: Track one total monthly income
  - Input field: Typical monthly income ($)
  - Benefits: 30-sec setup, fixed salary, upgradeable later
  
- ○ **DETAILED**: Track by source (salary, freelance, rental, etc.)
  - Action button: [Configure Sources →]
  - Benefits: Performance tracking, variable income, adds Chart 7

**Navigation:** [< Back to Categories] [Continue to Goals >]

**Data Saved to Settings_DB:**
```
Income_Tracking_Mode: "Simple" or "Detailed"
Typical_Monthly_Income: User entered amount (if Simple)
```

---

### Simple Mode Configuration

**When User Selects Simple:**

1. **Saves to Settings_DB:**
   - `Income_Tracking_Mode = "Simple"`
   - `Typical_Monthly_Income = [User entered amount]`

2. **Creates Single Category:**
   - Name: "Income" | Type: "Income" | Color: Gold | Icon: 💰

3. **Creates Initial History Entry:**
   - Start_Date: Today | Source_ID: 1 | Planned_Amount: Typical_Monthly_Income | Note: "Initial setup"

---

### Detailed Mode Configuration

**When User Clicks "Configure Sources":**

**Income Sources Setup Screen**

**Instructions:** Select your income sources (check all that apply)

**Predefined Sources (8 maximum):**
| Source | Expected Monthly |
|--------|-----------------|
| ☑ Salary/Wages | $[____] |
| ☑ Freelance/Consulting | $[____] |
| ☐ Business Income | $[____] |
| ☐ Rental Income | $[____] |
| ☐ Investment Income | $[____] |
| ☐ Side Hustle | $[____] |
| ☐ Commission | $[____] |
| ☐ Other: [_____] | $[____] |

**Total Expected:** $[Auto-calculated] /month

**Note:** Don't worry - you can change these amounts anytime

**Navigation:** [< Back] [Save & Continue]

**On Save Creates:**

1. **Income_Sources_DB entries** (8 pre-allocated rows)
   - Active sources: TRUE, Expected_Monthly set
   - Inactive sources: FALSE, Expected_Monthly = 0, Name = "[Available]"

2. **Categories for each active source:**
   - Name: "Income: [Source]" | Type: "Income" | Color: Gold | Icon: varies

3. **Income_Plan_History entries:**
   - Start_Date: Today | Source_ID: [each active] | Planned_Amount: [as entered] | Note: "Initial - [Source]"

4. **Sets mode:**
   - `Settings_DB!Income_Tracking_Mode = "Detailed"`

---

### Budget Tab - Simple Mode

**Location:** Budget Tab, Row 10-40

**Header:** INCOME TRACKING - [Year] | View: [2025 ▼] | Mode: Simple [Switch to Detailed →]

**Monthly Income Table:**
| Month | Planned | Actual | Variance | Status |
|-------|---------|--------|----------|--------|
| Jan 25 | Formula: Plan History | Formula: Tx Sum | =C-B | Conditional |
| ... | (12 rows) | | | |

**Formulas:**

**Planned (Column B):**
```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Start_Date] <= DATEVALUE(A5&"-01")), 
    Income_Plan_History[Planned_Amount]),
  Settings_DB!Typical_Monthly_Income)
```

**Actual (Column C):**
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], A5,
        Transaction_DB[Type], "Income",
        Transaction_DB[Deleted?], FALSE)
```

**Variance (Column D):** `=C5-B5`

**Status (Column E):**
```excel
=IF(C5=0, 
   IF(DATEVALUE(A5&"-01")<=TODAY(), "Missing", "Pending"),
   IF(D5>=0, "✓",
      IF(D5/B5>-0.15, "⚠️", "🚨")))
```

**UI Note Below Table:** 💡 Your planned income changed in [Month] (promotion)

---

### Budget Tab - Detailed Mode

**Location:** Budget Tab, Row 10-60

**Header:** INCOME TRACKING - [Year] | View: [2025 ▼] | Mode: Detailed [Switch to Simple →]

**Summary Table (Same as Simple):**
| Month | Planned | Actual | Variance | Status | [+] Sources |
|-------|---------|--------|----------|--------|-------------|
| (12 rows with expandable breakdown) | | | | | [▶]/[▼] toggle |

**Expandable Source Breakdown (when [▼] clicked):**
| Source | Expected | Received | Variance | Transactions |
|--------|----------|----------|----------|--------------|
| (Per active source) | Formula | Formula | Calc | Count |
| TOTAL | Sum | Sum | Sum | Total |

**Source Breakdown Formulas:**

**Expected (per source):**
```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Source_ID]=SourceID)*
      (Income_Plan_History[Start_Date]<=DATEVALUE(Month_ID&"-01")), 
    Income_Plan_History[Planned_Amount]),
  0)
```

**Received (per source):**
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], Month_ID,
        Transaction_DB[Category], "Income: "&SourceName,
        Transaction_DB[Deleted?], FALSE)
```

**Action Button:** [⚙️ Manage Income Sources]

---

### Edit Planned Income Form

**Triggered by:** [Edit] button click

**Form Fields:**
- Current Expected: $X,XXX/month (based on entry from [date])
- New Expected Amount: $[____]
- Starting When: ◉ [This month] ○ Custom date: [____]
- Apply to Future: ◉ Yes (raises/promotions) ○ No (one-time bonuses)
- Reason (optional): [____]

**Preview Impact Table:**
| Period | Before Edit | After Edit | Note |
|--------|-------------|------------|------|
| [Past months] | $X | $X | (no change) |
| [This month+] | $X | $Y | (updated) |

**Actions:** [💾 Save Change] [Cancel]

**On Save:** Creates new Income_Plan_History row with Start_Date, Source_ID, Planned_Amount, Note

---

### Managing Income Sources (Detailed Mode)

**Location:** Settings Tab, Row 60-80

**Active Sources Table:**
| # | Source Name | Expected | Active | Actions |
|---|-------------|----------|--------|---------|
| 1 | Salary | $4,200 | ☑ | [Edit] |
| 2 | Freelance | $800 | ☑ | [Edit] |
| 3 | [Add Source] | $0 | ☐ | |
| ... | (8 rows total) | | | |

**Behavior:**
- Max 8 sources (pre-allocated rows)
- Expected shows most recent from Income_Plan_History
- Add: Type name, amount, check Active → auto-creates category
- Edit: Modify amount → creates new Income_Plan_History entry
- Remove: Uncheck Active → grays out, preserves history

**Navigation:** [Back to Budget]

---

## 📊 DASHBOARD LAYOUT

### Complete Dashboard Structure

**Location:** Dashboard Tab, Rows 1-120 (conditional Chart 7 adds to Row 116)

**Layout:**
| Rows | Section | Content |
|------|---------|---------|
| 1 | Navigation | Sheet links |
| 2 | Header | Title + Period Selector [Week/Month/Quarter/Year ▼] |
| 4-11 | KPI Grid | 2×2: Net Worth, Monthly Savings, Savings Rate, Budget Used |
| 13-20 | Transactions | Recent 5 entries (view-only) |
| 22-26 | Budget Progress | Scrollable category tracker |
| 28 | Section Header | "FINANCIAL HEALTH" |
| 29-44 | Chart 1 | Financial Overview (full width A-T) |
| 46 | Section Header | "BUDGET ANALYSIS" |
| 47-62 | Charts 2-3 | Budget Spending (A-K), Spending Mix (L-T) |
| 64 | Section Header | "INCOME & DEBT PROGRESS" |
| 65-80 | Charts 4-5 | Income Performance (A-K), Debt Payoff (L-T) |
| 82 | Section Header | "WEALTH TRAJECTORY" |
| 83-98 | Chart 6 | Net Worth Projection (full width A-T) |
| 100* | Section Header | "INCOME SOURCE PERFORMANCE" (*Detailed mode only) |
| 101-116* | Chart 7* | Income by Source (full width A-T) |
| 118-120 | Quick Links | Action buttons |

**All charts:** 16 rows tall (balanced visual weight)

---

## 📈 CHART SPECIFICATIONS

### Universal Chart Standards

**Use these standards across all charts (reference this section instead of repeating):**

**Axis Formatting:**
- Currency: "$X,XXX" or "$XX,XXX"
- Dates: "MMM YY" (e.g., "Oct 25")
- Percentages: "XX%" or "XX.X%"
- Integers: "#,##0"

**Data Labels:**
- Position: Above/right of data point
- Font: 9pt
- Format: Matches axis format
- Show only on key points (last, max, min)

**Legend:**
- Position: Bottom-center (horizontal) or Right-side (vertical)
- Font: 10pt
- Use series names, not colors alone

**Status Icons (Consistent across all charts):**
- ✓ Green: Good (<80% budget used, met targets)
- ⚠️ Orange: Warning (80-100% budget used, close miss)
- 🚨 Red: Alert (>100% budget used, significant shortfall)

**Gridlines:**
- Horizontal: Light gray (#E5E7EB), 1pt
- Vertical: Usually hidden
- Major only (no minor)

**Colors:**
- Income: Gold (#D4AF37)
- Expenses: Red (#EF4444)
- Savings: Green (#10B981)
- Budget: Navy (#1E40AF)
- Secondary: Teal (#14B8A6)

**Hover Tooltips:**
- Show: Category/Series name, Value, Percentage (if applicable)
- Format: "Category: $X,XXX (XX%)"

---

### Chart 1: Financial Overview (Priority 1)

**Problem:** "What's my overall financial health?"

**Type:** Dual-axis line chart  
**Location:** Dashboard, Rows 29-44 (Full Width, A-T)  
**Period:** Last 6 periods (responsive to selector)

**Title:** "Financial Health Overview"

**Left Y-Axis:** Dollar Amount ($0 - auto-scale)
- Series 1: Total Income (Gold line, 2pt, ● markers)
- Series 2: Total Expenses (Red line, 2pt, ■ markers)
- Series 3: Net Savings (Green line, 2pt, ♦ markers)

**Right Y-Axis:** Percentage (0% - 100%)
- Series 4: Savings Rate % (Blue dashed, 2pt, ▲ markers)
- Benchmark Lines: 20% (Good), 30% (Excellent) - Gray dashed, 1pt

**X-Axis:** Last 6 periods (format adapts to Period selector)

**Uses:** Standard axis formatting, data labels (last point only), horizontal gridlines

**Key Stats Box (Top right):**
```
This Month:
Income: $5,200
Expenses: $3,120
Saved: $2,080 (40.0%) ✓
```

**Data Formulas:** (See Formula Pattern Library - Period-Filtered Sum)

---

### Chart 2: Budget Spending Tracker (Priority 2)

**Problem:** "Am I on track with my budget by category?"

**Type:** Horizontal bar chart with variance indicators  
**Location:** Dashboard, Rows 47-62 (Left, A-K)  
**Period:** Current Month only

**Title:** "Budget Performance by Category"

**Y-Axis:** Top 8 categories by budget + "Other" (sorted by variance, worst at top)

**X-Axis:** Percentage (0% - 150%) with markers at 80%, 100%, 120%

**Bars (Stacked):**
- Base layer (gray 30% opacity): Full budget (100%)
- Actual layer (conditional color): Amount spent
  - Green: 0-80% (safe)
  - Yellow: 80-90% (approaching)
  - Orange: 90-100% (at limit)
  - Red: 100%+ (over)

**Reference Lines:** 80% threshold (dashed), 100% limit (solid 2pt)

**Data Labels:** Inside: "$X spent" | Outside (if over): "+$X over"

**Status Icons:** (See Universal Chart Standards)

**Annotation Box (Below chart):** Dynamic message
```
Example: "🚨 2 categories over: Dining (+$65), Shopping (+$35)
         ⚠️ 1 near limit: Transport (91%)
         💡 Tip: Review Dining transactions to find cuts"
```

**Companion Table (Rows 63-72):**
| Category | Budget | Spent | Remaining | Status |
|----------|--------|-------|-----------|--------|
| (Top 8 + Other + TOTAL) | | | | |

---

### Chart 3: Spending Composition (Priority 2)

**Problem:** "Where did my money actually go?"

**Type:** Treemap (preferred) or Donut (fallback)  
**Location:** Dashboard, Rows 47-62 (Right, L-T)  
**Period:** Period-aware (responsive to selector)

**Title:** "Spending Breakdown ([Period Name])"

**Treemap Version:**
- Rectangles sized proportionally to spending
- Colors from Categories_DB
- Labels (inside): Category name, Amount, Percentage (varies by size)
- Center summary: "Total Spent | $X,XXX | X categories"

**Donut Version:**
- Slices: Top 8 + "Other" (clockwise from 12:00, largest to smallest)
- Min slice: 2% (combine smaller into "Other")
- Labels (outside): "[Category] $X (Y%)"
- Center text: Period label, Total amount (16pt), Category count

**Uses:** Standard formatting, category colors, hover tooltips

---

### Chart 4: Income Performance (Priority 4)

**Problem:** "Am I hitting my income targets?"

**Type:** Grouped column chart  
**Location:** Dashboard, Rows 65-80 (Left, A-K)  
**Period:** Last 6 months

**Title:** "Income: Plan vs Actual (Last 6 Months)"

**X-Axis:** Last 6 months (MMM format)

**Y-Axis:** Dollar amount (auto-scale)

**Columns (Grouped):**
- Series 1: Planned (Navy, 60% width, solid)
- Series 2: Actual (Conditional color, 60% width)
  - Green: Actual ≥ Planned
  - Orange: Actual 85-99% of Planned
  - Red: Actual < 85% of Planned

**Benchmark Line:** Average planned (Navy dashed, 2pt)

**Data Labels:** Above actual bars: "$X,XXX" | Below (if variance): "+/-$X"

**Summary Badge (Top right):**
```
Last 6 months avg:
Planned: $5,200
Actual: $5,100 (-1.9%)
Hit target: 4 of 6 ✓
```

**Uses:** Standard formatting, conditional colors, gridlines

---

### Chart 5: Debt Payoff Tracker (Priority 5)

**Problem:** "How fast am I paying down debt? When debt-free?"

**Type:** Stacked area chart with projection  
**Location:** Dashboard, Rows 65-80 (Right, L-T)  
**Period:** 12 months historical + projection to debt-free

**Title:** "Debt Payoff Timeline"

**X-Axis:** Months (historical solid, future with "Future" label + dashed grid)

**Y-Axis:** Total debt balance ($0 - auto-scale, starts at $0)

**Stacked Areas (Historical):**
- Each debt = colored area (70% opacity)
- Stack order: Highest APR at bottom
- Colors: Darkest red = highest APR

**Projected Line (Future):**
- Style: Dashed (5pt dash, 3pt gap), 2pt width
- Based on: Current payment rate + selected strategy

**Key Annotations:**
- "Now" marker: Vertical solid gray line, 2pt
- Milestones: Horizontal dashed lines at $7.5K, $5K, $2.5K
- Debt-free point: Vertical green dashed, 🎉 icon, "Debt-Free: [Month]"

**Data Labels:** Start point, Current point, End point

**Strategy Selector (Below):** [Avalanche ▼] dropdown (Avalanche/Snowball/Custom)

**Summary Box (Rows 81-82):**
```
Progress: $2,500 paid (20%) | Remaining: $10,000 (80%)
At current rate: 18 months to debt-free
Interest saved: $2,100 (vs minimum payments)
💡 Increase monthly payment by $100 → Debt-free 3 mo sooner
```

---

### Chart 6: Net Worth Projection (Priority 6)

**Problem:** "Is my wealth growing? Where will I be in 5 years?"

**Type:** Area chart with projection cone  
**Location:** Dashboard, Rows 83-98 (Full Width, A-T)  
**Period:** All-time + 5 years projected

**Title:** "Net Worth Growth & 5-Year Projection"

**X-Axis:** Time (format adapts: Monthly <2yr, Quarterly 2-5yr, Yearly >5yr)

**Y-Axis:** Net worth (auto-scale, include $0 line if starting negative)

**Historical Area:**
- Line: Green, 2pt, ● markers
- Fill: Light green, 30% opacity
- Data: Actual snapshots from Net_Worth_Snapshots_DB

**Projection Cone:**
- Center Line: Expected growth (Green dashed, 2pt)
- Upper Bound: Optimistic +2% (Light green dashed, 1pt)
- Lower Bound: Conservative -2% (Light green dashed, 1pt)
- Cone Fill: Green gradient, 20% opacity

**Milestone Markers:**
- $0 Crossing (if applicable): "Break Even"
- $50K: "Emergency Fund Secured"
- $100K: 🎯 "Six Figures!"
- Custom goals from user

**"Today" Marker:** Vertical solid gray, 2pt, "Now: $45,000"

**Assumption Inputs (Rows 99-100):**
| Field | Value |
|-------|-------|
| Monthly Savings | $[850] |
| Expected Return | [7.0]% annual |
| Debt-Free Date | [Jul 2026] (from Chart 5) |

**Action:** [🔄 Recalculate Projection]

**Projection Results (Below inputs):**
```
In 5 years (October 2030):
• Expected: $125,000
• Range: $110,000 - $145,000
• Growth: 2.8× current net worth
• Yearly increase: ~$16,000/year average
```

---

### Chart 7: Income by Source (Conditional - Detailed Mode Only)

**Conditional Display:** Only if `Settings_DB!Income_Tracking_Mode = "Detailed"`

**Type:** Stacked column chart  
**Location:** Dashboard, Rows 101-116 (Full Width, A-T)  
**Period:** Last 6 months

**Title:** "Income Performance by Source (Last 6 Months)"

**X-Axis:** Last 6 months (MMM format)

**Y-Axis:** Dollar amount (auto-scale)

**Columns (Stacked):**
- Each source = separate colored series
- Stack order: Largest source at bottom
- Colors: Assign from palette (Navy, Teal, Green, etc.)

**Data Labels:** Top of stack only: Total "$X,XXX"

**Hover Tooltips:** Month, Source name, Amount, % of total

**Legend:** Right side, vertical, all active sources

**Reference Line:** Average total income (dashed gray)

**Summary Box (Below):**
```
Most Consistent: Salary (100% of expected, 6/6 months)
Most Variable: Freelance (±35% variance)
Highest Month: June ($5,900)
Lowest Month: April ($4,200)
6-mo Total: $30,900
```

**Conditional Formula:**
```excel
=IF(Settings_DB!Income_Tracking_Mode="Detailed", [CHART RANGE], "")
```

---

## 💾 DATA STRUCTURES

### Updated Tables with Month_ID

**1. Transaction_DB** (Transactions sheet, Row 2-10000)

Columns: ID (INT) | Date (DATE) | Month_ID (TEXT "YYYY-MM", auto) | Description (TEXT) | Amount (DECIMAL) | Category_ID (INT FK) | Type (TEXT) | Deleted? (BOOL)

Index: Month_ID

Auto-formula (Col C): `=TEXT(B2,"YYYY-MM")`

Sample: {1, 2025-01-15, "2025-01", "Groceries", -85.50, 1, "Expense", FALSE}

---

**2. Budget_Allocations_DB** (Budget_Data sheet, Row 2-1000)

Columns: ID (INT) | Month_ID (TEXT "YYYY-MM") | Category_ID (INT FK) | Allocated_Amt (DECIMAL) | Spent_Amt (DECIMAL, calculated) | Notes (TEXT)

Index: Month_ID

Sample: {1, "2025-01", 1, 425.00, 380.25, ""}

---

**3. Net_Worth_Snapshots_DB** (Net_Worth sheet, Row 2-1000)

Columns: ID (INT) | Snapshot_Date (DATE, EOM) | Month_ID (TEXT "YYYY-MM", auto) | Net_Worth (DECIMAL) | Assets (DECIMAL) | Liabilities (DECIMAL) | Note (TEXT)

Index: Month_ID

Auto-formula: `Month_ID = TEXT(Snapshot_Date,"YYYY-MM")`

Sample: {1, 2025-01-31, "2025-01", 45000, 62000, 17000, ""}

---

**4. Debt_DB** (Debt sheet, Row 2-100)

Columns: ID (INT) | Debt_Name (TEXT) | Start_Date (DATE) | Month_ID (TEXT "YYYY-MM") | Current_Balance (DECIMAL) | APR (DECIMAL) | Monthly_Payment (DECIMAL)

Sample: {1, "CC #1", 2024-03-15, "2024-03", 5200.00, 18.99, 150.00}

---

**5. Debt_History_DB** (Debt_History sheet, Row 2-1000) **[NEW]**

Columns: ID (INT) | Debt_ID (INT FK) | Month_ID (TEXT "YYYY-MM") | Balance_EOM (DECIMAL) | Payment (DECIMAL) | Interest_Accrued (DECIMAL)

Purpose: Track monthly debt balances for Chart 5

Sample: {1, 1, "2024-03", 5200.00, 0.00, 0.00}

---

**6. Income_Sources_DB** (Settings_DB sheet, Row 10-20) **[NEW]**

Columns: ID (INT, 1-8 fixed) | Source_Name (TEXT) | Expected_Monthly (DECIMAL) | Active (BOOL) | Row_Number (INT, 1-8) | Category_Created (TEXT)

Structure: 8 pre-allocated rows (no VBA needed)

Inactive display: "[Available]" in Source_Name

Sample: {1, "Salary", 4200.00, TRUE, 1, "Income: Salary"}

---

**7. Income_Plan_History** (Settings_DB sheet, Row 50-100) **[NEW - CRITICAL]**

Columns: ID (INT) | Start_Date (DATE) | Source_ID (INT FK) | Planned_Amount (DECIMAL) | Note (TEXT)

Index: Composite on (Source_ID, Start_Date) - CRITICAL for LOOKUP performance

Sorted: By Start_Date ascending (MANDATORY)

Sample: {1, 2025-01-01, 1, 3500.00, "Initial salary"}

Behavior: No end date, active until next entry for same source, enables indefinite carry-forward

---

**8. Settings_DB Updates** (Settings sheet, Row 2-50)

New Settings:
- Income_Tracking_Mode: "Simple" or "Detailed"
- Typical_Monthly_Income: DECIMAL (Simple mode only)
- Current_Month_ID: TEXT "YYYY-MM"
- Current_Year: INT

---

## 📐 FORMULA PATTERN LIBRARY

**Use these patterns throughout the dashboard - reference by name instead of repeating full formula**

### Pattern 1: Period-Filtered Sum
**Purpose:** Sum transactions for a specific period

**Base:**
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], TargetMonth,
        Transaction_DB[Type], TypeFilter,
        Transaction_DB[Deleted?], FALSE)
```

**Variations:**
- **Income:** TypeFilter="Income"
- **Expenses:** TypeFilter="Expense", multiply result by -1
- **By Category:** Add `Transaction_DB[Category_ID], CategoryID`
- **By Source:** Add `Transaction_DB[Category], "Income: "&SourceName`
- **Multiple Months:** Replace TargetMonth with range array

**Used in:** Chart 1, Chart 2, Chart 3, Chart 4, Budget tables, Dashboard KPIs

---

### Pattern 2: Income Plan Lookup
**Purpose:** Get planned income for any month (handles promotions)

**Formula:**
```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Source_ID]=TargetSourceID)*
      (Income_Plan_History[Start_Date]<=DATEVALUE(TargetMonthID&"-01")), 
    Income_Plan_History[Planned_Amount]),
  DefaultAmount)
```

**Variations:**
- **Simple mode:** Omit Source_ID filter (or use Source_ID=1)
- **All sources:** Sum across all active Source_IDs
- **No default:** Remove DefaultAmount parameter (returns blank if not found)

**Used in:** Chart 4, Budget Simple table, Budget Detailed table, Income Sources management

---

### Pattern 3: Month_ID Generation
**Purpose:** Auto-populate Month_ID from Date

**Formula:**
```excel
=TEXT(Date_Column, "YYYY-MM")
```

**Used in:** Transaction_DB, Budget_Allocations_DB, Net_Worth_Snapshots_DB, Debt_DB

---

### Pattern 4: Budget Percentage Used
**Purpose:** Calculate budget utilization for color-coding

**Formula:**
```excel
=Actual_Spending / Budget_Allocation
```

**Conditional Colors:**
```excel
Color = IF(Percent<0.8, GREEN,
          IF(Percent<0.9, YELLOW,
             IF(Percent<=1.0, ORANGE, RED)))

Status = IF(Percent<0.8, "✓",
           IF(Percent<=1.0, "⚠️", "🚨"))
```

**Used in:** Chart 2, Budget Progress tracker, Budget table

---

### Pattern 5: Debt Projection (Avalanche Strategy)
**Purpose:** Calculate months to debt-free

**Algorithm:**
```
Monthly_Payment = SUM(Debt_DB[Monthly_Payment])
Sort debts by APR descending

For each month M:
  Apply minimum payments to all debts
  Extra_Payment = Monthly_Payment - SUM(Minimums)
  Apply Extra_Payment to highest APR debt
  When debt balance = 0:
    Remove from list
    Redirect its minimum to Extra_Payment
  Continue until all debts = 0
  
Debt_Free_Month = M when Total_Balance = 0
```

**Snowball Variation:** Sort debts by balance ascending instead of APR descending

**Used in:** Chart 5, Debt sheet scenarios

---

### Pattern 6: Net Worth Projection
**Purpose:** Forecast future net worth with investment growth

**Formula:**
```excel
Starting_NW = Current_Net_Worth
Monthly_Return = (1 + Annual_Return/100)^(1/12) - 1

For each future month M:
  Debt_Payment[M] = IF(M < Debt_Free_Month, 
                       Total_Monthly_Debt_Payment, 0)
  
  Net_Savings[M] = Monthly_Savings + 
                   (IF(M >= Debt_Free_Month, Debt_Payment, 0))
  
  Growth[M] = Previous_NW × Monthly_Return
  
  NW[M] = Previous_NW + Net_Savings[M] + Growth[M]
  Previous_NW = NW[M]
```

**Projection Bounds:**
- Center: Expected_Return (e.g., 7%)
- Upper: Optimistic_Return (e.g., 9%)
- Lower: Conservative_Return (e.g., 5%)

**Used in:** Chart 6, Net Worth sheet

---

### Pattern 7: Conditional Chart Display
**Purpose:** Show/hide Chart 7 based on income mode

**Formula:**
```excel
=IF(Settings_DB!Income_Tracking_Mode="Detailed",
    ChartRange,
    "")
```

**Used in:** Chart 7 display logic

---

### Pattern 8: Year Selector Dropdown
**Purpose:** Generate dynamic year list

**Formula:**
```excel
=UNIQUE(TEXT(Transaction_DB[Date],"YYYY"))
```

**Returns:** Array of years with data (e.g., {2024, 2025, 2026})

**Used in:** Budget tab View selector

---

## ✅ IMPLEMENTATION CHECKLIST

### Phase 1: Data Structure Updates (Foundation)

**Schema Changes:**
- [ ] Add Month_ID column to Transaction_DB with auto-formula `=TEXT(Date,"YYYY-MM")`
- [ ] Add Month_ID column to Budget_Allocations_DB
- [ ] Add Month_ID column to Net_Worth_Snapshots_DB with auto-formula
- [ ] Add Month_ID column to Debt_DB
- [ ] Create Debt_History_DB table (structure per Data Structures section)
- [ ] Create Income_Plan_History table (structure per Data Structures section)
- [ ] Create Income_Sources_DB table (8 pre-allocated rows)
- [ ] Add to Settings_DB: Income_Tracking_Mode, Typical_Monthly_Income, Current_Month_ID

**Data Migration:**
- [ ] Backfill Month_ID in existing Transaction_DB records
- [ ] Backfill Month_ID in existing Budget_Allocations_DB records
- [ ] Backfill Month_ID in existing Net_Worth_Snapshots_DB records
- [ ] Create initial Income_Plan_History entry (Start_Date=Today, Source_ID=1, Planned_Amount=0)

**Indexes:**
- [ ] Create index on Transaction_DB[Month_ID]
- [ ] Create index on Budget_Allocations_DB[Month_ID]
- [ ] Create composite index on Income_Plan_History(Source_ID, Start_Date) - CRITICAL

---

### Phase 2: Setup Wizard Enhancement

- [ ] Add Step 3: Income Setup screen to Settings Tab
- [ ] Implement Simple/Detailed mode radio selector
- [ ] Simple mode: Single input field for Typical_Monthly_Income
- [ ] Detailed mode: [Configure Sources] button opens source selection screen
- [ ] Source selection: 8 checkbox rows with Expected Monthly inputs
- [ ] Save to Settings_DB: Income_Tracking_Mode value
- [ ] Create categories based on selected mode
- [ ] Create initial Income_Plan_History entries for active sources
- [ ] Test wizard: Simple mode start-to-finish
- [ ] Test wizard: Detailed mode start-to-finish

---

### Phase 3: Budget Tab - Income Tracking

**Simple Mode Table:**
- [ ] Create table structure (Rows 10-40, columns: Month | Planned | Actual | Variance | Status)
- [ ] Add View selector dropdown (Year selection)
- [ ] Formula: Planned column uses Pattern 2 (Income Plan Lookup)
- [ ] Formula: Actual column uses Pattern 1 (Period-Filtered Sum, Income variation)
- [ ] Formula: Variance = Actual - Planned
- [ ] Formula: Status with conditional icons (✓/⚠️/🚨)
- [ ] Add [Edit] button with click trigger
- [ ] Create Edit Income form (inline or popup)
- [ ] Form saves to Income_Plan_History with Start_Date, Planned_Amount, Note
- [ ] Add mode switch link: [Switch to Detailed →]
- [ ] Test: Initial setup, monthly tracking, promotion entry, carry-forward

**Detailed Mode Table:**
- [ ] Create summary table (same columns as Simple + [+] Sources toggle)
- [ ] Implement expandable breakdown rows per month
- [ ] Breakdown columns: Source | Expected | Received | Variance | Transactions
- [ ] Formula: Total planned (sum all active sources)
- [ ] Formula: Total actual (sum all income categories)
- [ ] Formula: Source-level expected uses Pattern 2 per source
- [ ] Formula: Source-level received uses Pattern 1 with category filter
- [ ] [▶]/[▼] toggle to show/hide breakdown
- [ ] Add [⚙️ Manage Income Sources] button
- [ ] Test: Multi-source tracking, variable income, source performance

**Income Sources Management:**
- [ ] Create management table (Settings Tab, 8 rows)
- [ ] Display "[Available]" for inactive sources
- [ ] Add source: Type name, amount, check Active
- [ ] Auto-create category on activate: "Income: [Name]"
- [ ] Edit source: Modify amount creates new Income_Plan_History entry
- [ ] Deactivate: Uncheck Active, preserve history
- [ ] Link back to Budget tab
- [ ] Test: Add 3 sources, edit amounts, deactivate 1, reactivate

---

### Phase 4: Dashboard Layout

- [ ] Row 1: Navigation bar with sheet hyperlinks
- [ ] Row 2: Title + Period selector dropdown (stores to Settings_DB)
- [ ] Rows 4-11: KPI cards 2×2 grid (Net Worth, Monthly Savings, Savings Rate, Budget Used)
- [ ] Rows 13-20: Recent transactions list (last 5, read-only)
- [ ] Rows 22-26: Budget progress tracker (scrollable category bars)
- [ ] Insert section headers at Rows 28, 46, 64, 82, 100 (conditional)
- [ ] Rows 118-120: Quick action links
- [ ] Test: Period selector updates, KPIs calculate correctly

---

### Phase 5: Chart Implementation

**Chart 1: Financial Overview**
- [ ] Create dual-axis line chart (Rows 29-44, A-T)
- [ ] Left axis: Income, Expenses, Savings (lines per spec)
- [ ] Right axis: Savings Rate % line
- [ ] Benchmark lines: 20%, 30%
- [ ] Data source: Last 6 periods using Pattern 1
- [ ] Data labels: Last point only
- [ ] Key stats box (top right)
- [ ] Test: Different periods, verify calculations

**Chart 2: Budget Spending Tracker**
- [ ] Create horizontal bar chart (Rows 47-62, A-K)
- [ ] Y-axis: Top 8 categories + Other (sorted by variance)
- [ ] Stacked bars: Gray base + conditional color actual
- [ ] Implement color logic using Pattern 4
- [ ] Reference lines: 80%, 100%
- [ ] Status icons per category
- [ ] Dynamic annotation box below
- [ ] Create companion table (Rows 63-72)
- [ ] Test: Over-budget scenarios, color changes

**Chart 3: Spending Composition**
- [ ] Create treemap or donut (Rows 47-62, L-T)
- [ ] Data: Top 8 + Other from Pattern 1
- [ ] Colors from Categories_DB
- [ ] Labels: Category, Amount, Percentage (size-dependent)
- [ ] Center summary (if donut)
- [ ] Period-responsive
- [ ] Test: Different periods, verify totals sum to 100%

**Chart 4: Income Performance**
- [ ] Create grouped column chart (Rows 65-80, A-K)
- [ ] Grouped: Planned (navy) vs Actual (conditional)
- [ ] Planned uses Pattern 2, Actual uses Pattern 1
- [ ] Conditional colors: Green/Orange/Red based on performance
- [ ] Variance labels below bars
- [ ] Benchmark line: Average planned
- [ ] Summary badge
- [ ] Test: Simple mode, Detailed mode, mixed performance

**Chart 5: Debt Payoff Tracker**
- [ ] Create stacked area chart (Rows 65-80, L-T)
- [ ] Historical: 12 months from Debt_History_DB
- [ ] Stack by debt (highest APR at bottom)
- [ ] Projected: Dashed line using Pattern 5
- [ ] Strategy selector dropdown (Avalanche/Snowball)
- [ ] Annotations: Now, Milestones, Debt-free point
- [ ] Summary box (Rows 81-82)
- [ ] Test: Multiple debts, strategy switching, projection accuracy

**Chart 6: Net Worth Projection**
- [ ] Create area chart with cone (Rows 83-98, A-T)
- [ ] Historical: Snapshots from Net_Worth_Snapshots_DB
- [ ] Projected: Center + bounds using Pattern 6
- [ ] Milestone markers ($0, $50K, $100K, custom)
- [ ] "Today" marker
- [ ] Assumption inputs (Rows 99-100)
- [ ] [Recalculate] button functionality
- [ ] Results display below inputs
- [ ] Test: Different assumptions, verify future value math

**Chart 7: Income by Source (Conditional)**
- [ ] Create stacked column chart (Rows 101-116, A-T)
- [ ] Conditional display using Pattern 7
- [ ] Stack by source (colors from palette)
- [ ] Data: Last 6 months per source using Pattern 1
- [ ] Total labels on top
- [ ] Legend: All active sources
- [ ] Summary box below
- [ ] Test: Mode switching (chart appears/disappears correctly)

---

### Phase 6: Testing & Validation

**Data Accuracy:**
- [ ] Verify Month_ID auto-population in all tables
- [ ] Verify Pattern 2 (Income Plan Lookup) for past months
- [ ] Verify Pattern 2 for future months
- [ ] Verify promotion carries forward to next year
- [ ] Verify source-level tracking matches total in Detailed mode
- [ ] Manually verify all chart calculations vs spreadsheet formulas

**Mode Switching:**
- [ ] Test Simple → Detailed: Data preserved, Chart 7 appears
- [ ] Test Detailed → Simple: Data preserved, Chart 7 disappears
- [ ] Verify Categories_DB updated correctly
- [ ] Verify Income_Plan_History intact both directions

**Year Boundaries:**
- [ ] Add transactions in Dec 2025 and Jan 2026
- [ ] Verify Month_ID correct: "2025-12", "2026-01"
- [ ] Verify charts display both years correctly
- [ ] Verify Budget tab shows both years in dropdown
- [ ] Verify promotion from 2025 carries to 2026

**Edge Cases:**
- [ ] Test with $0 income user
- [ ] Test with negative net worth
- [ ] Test with no debt (Chart 5 shows appropriate message)
- [ ] Test with 8 income sources (maximum)
- [ ] Test promotion on first day of month
- [ ] Test promotion on last day of month
- [ ] Test income source with $0 expected (valid for variable income)

**Performance:**
- [ ] Test with 1,000 transactions: Chart load time
- [ ] Test with 5,000 transactions: Calculation speed
- [ ] Test with 10,000 transactions: Period selector response
- [ ] Measure Period selector change: Target <1 second
- [ ] Measure Dashboard initial load: Target <3 seconds
- [ ] Verify no lag on Month_ID filtered queries

---

### Phase 7: Documentation & Polish

**User Documentation:**
- [ ] Setup wizard tooltips and help text
- [ ] Income tracking modes explanation (Simple vs Detailed)
- [ ] How to record promotion/raise guide
- [ ] How to add/remove income source guide
- [ ] Dashboard chart interpretation guide
- [ ] Troubleshooting: Common issues FAQ

**Visual Polish:**
- [ ] Consistent color palette across all charts (use Universal Standards)
- [ ] Font sizes standardized (10pt body, 14pt headers, 9pt labels)
- [ ] Icons consistent (✓ ⚠️ 🚨 across all sheets)
- [ ] Tooltips on complex fields
- [ ] Loading states for calculations >1 second
- [ ] Print preview: All charts fit on pages correctly

**Final QA:**
- [ ] Test on Excel 2019 (Windows & Mac)
- [ ] Test on Excel 365 (Windows & Mac)
- [ ] Test on Google Sheets (web)
- [ ] Test on Excel Mobile (iOS)
- [ ] Test on Excel Mobile (Android)
- [ ] Cross-browser if web-based (Chrome, Firefox, Safari, Edge)

---

## 🎯 SUCCESS CRITERIA

### Must Haves (V6.0)
- ✅ Two-track income system fully functional (Simple + Detailed)
- ✅ Month_ID tracking implemented across all 8 tables
- ✅ Income_Plan_History enables promotion carry-forward
- ✅ All 6 core charts working with accurate data
- ✅ Chart 7 conditional display (Detailed mode only)
- ✅ Multi-year support (2025, 2026, 2027+)
- ✅ Setup wizard completes in 5 minutes (Simple) or 2 minutes (Detailed)

### Performance Targets
- Dashboard loads: <3 seconds with 1,000 transactions
- Chart refresh: <1 second on period change
- Edit income form: Opens instantly
- Month_ID filtering: No perceptible lag

### Quality Targets
- Zero formula errors in any cell
- No hardcoded dates (all dynamic)
- Mobile-responsive (view-optimized)
- Cross-platform: Excel 2016+ & Google Sheets
- Data integrity: All relationships maintained

---

## 📚 APPENDIX

### A. Example Scenarios (Testing Reference)

**Scenario 1:** New user, fixed salary
- Setup: Simple mode, $4,500 → Monthly paycheck → Auto-tracks → 6 charts working

**Scenario 2:** Promotion in April
- Edit planned income: $4,500 → $5,200, "Apply to future" → Jan-Mar shows $4,500, Apr+ shows $5,200 → 2026 inherits $5,200 ✓

**Scenario 3:** Freelancer, variable income
- Setup: Detailed mode, Salary + Freelance → Categorized transactions → Source breakdown visible → Chart 7 appears

### B. Migration Guide (V5 → V6)

**Required:**
1. Add Month_ID to 4 tables (Transaction, Budget, NetWorth, Debt)
2. Backfill Month_ID from Date columns
3. Create 3 new tables (Income_Plan_History, Income_Sources_DB, Debt_History_DB)
4. Add Settings fields
5. Update formulas to use Month_ID filters

**Time:** ~8 hours (2h schema, 1h migration, 3h formulas, 2h testing)

**Data Preserved:** All transactions, budgets, categories, snapshots intact

### C. Future Enhancements (V7.0+)
- Year-over-year comparison charts
- AI-powered budget recommendations
- Income source ROI analysis
- Tax withholding calculator
- Multiple currency support
- Automated savings rules
- Bill reminders integration

---

**END OF PHASE 07 SPECIFICATION**

Version 6.0 - Optimized (~50KB vs 71KB original)  
100% Information Retained | Implementation-Ready  
October 14, 2025
