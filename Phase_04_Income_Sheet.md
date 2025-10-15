# Phase 4: Income Sheet
**Deliverables:** Income sheet with Simple and Detailed tracking modes  
**Estimated Time:** 2 hours  
**Dependencies:** Phases 1-3 (Foundation, Transactions, Utilities/Settings)
**Critical:** Must be built as Phase 4 - provides income baseline for Budget phase

---

## 🎯 Deliverables

1. **Income Sheet** - Dedicated sheet for income planning and tracking
2. **Two-tier system** - Simple mode (single total) and Detailed mode (multiple sources)
3. **Income_Plan_History** - Historical tracking of income changes (promotions, raises)
4. **Income_Sources_DB** - Up to 8 income sources for Detailed mode

---

## 🔗 Dependencies

**Required from Phase 1 (Foundation):**
- `Settings_DB` - For storing Income_Tracking_Mode and Typical_Monthly_Income
- `Categories_DB` - For creating income categories ("Income: Salary", etc.)

**Required from Phase 2 (Transactions):**
- `Transaction_DB` - To calculate actual income received

**Required from Phase 3 (Utilities/Settings):**
- Settings sheet for category management
- Settings_DB for storing Income_Tracking_Mode (UI is on Income sheet, not Settings)

**Provides to later phases (5-8):**
- Income baseline for Budget calculations (Phase 5 needs this)
- Income data for Dashboard charts (Phase 8)
- Multi-source income tracking for detailed analysis

---

## 💾 Database Schemas

### Income_Plan_History (Hidden)

**Location:** Settings_DB sheet, Row 50-100

**Purpose:** Track income changes over time (promotions, raises, new sources)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Number | 1 | Auto-increment |
| B | Start_Date | Date | 2025-01-01 | When amount begins |
| C | Source_ID | Number | 1 | Links to Income_Sources_DB |
| D | Planned_Amount | Number | 4200.00 | Expected monthly amount |
| E | Note | Text | "Promotion" | User explanation |

**Sample Data:**
```
1 | 2025-01-01 | 1 | 3500.00 | Initial salary
2 | 2025-04-01 | 1 | 4200.00 | Promotion
3 | 2025-05-01 | 2 | 800.00  | Started freelancing
4 | 2026-03-01 | 1 | 4800.00 | Annual raise
```

**Key Behavior:**
- No end date (active until next entry for same source)
- Sorted by Start_Date ascending
- LOOKUP finds most recent entry ≤ target month
- Enables promotions to carry forward indefinitely

**Named Ranges:**
```
Income_Plan_History[Start_Date] = $B$50:$B$100
Income_Plan_History[Source_ID] = $C$50:$C$100
Income_Plan_History[Planned_Amount] = $D$50:$D$100
```

---

### Income_Sources_DB (Hidden)

**Location:** Settings_DB sheet, Row 10-20

**Purpose:** Define income sources for Detailed mode

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Number | 1 | Row number 1-8 |
| B | Source_Name | Text | "Salary" | Max 30 chars |
| C | Expected_Monthly | Number | 4200.00 | For display only |
| D | Active | Boolean | TRUE | TRUE if used |
| E | Category_Created | Text | "Income: Salary" | Auto-linked category |

**Sample Data:**
```
1 | Salary      | 4200.00 | TRUE  | Income: Salary
2 | Freelance   | 800.00  | TRUE  | Income: Freelance
3 | [Available] | 0.00    | FALSE | 
4 | [Available] | 0.00    | FALSE |
5 | [Available] | 0.00    | FALSE |
6 | [Available] | 0.00    | FALSE |
7 | [Available] | 0.00    | FALSE |
8 | [Available] | 0.00    | FALSE |
```

**Key Behavior:**
- 8 pre-allocated rows (max 8 sources)
- "[Available]" displayed for inactive rows
- Expected_Monthly shows most recent from Income_Plan_History

---

## 🎨 Income Sheet Layout

**Freeze:** Row 4, Column A

**Column Widths:**
```
A: 200px (Label) | B-D: 120px each (Planned/Actual/Variance) | E: 100px (Status)
```

### Row 1-3: Title and Controls

**A1 (Title):**
```
"💰 INCOME TRACKING" - 18pt Bold Navy, Lt Blue bg, merge A1:P1
```

**A2-D2: Mode Selector (EDITABLE)**
```
A2: Income Tracking Mode: [Simple â–¼]
    Options: Simple | Detailed
    Default: Simple

D2: [Switch to Detailed →] (if Simple mode active)
    [Switch to Simple →] (if Detailed mode active)

DESIGN NOTE: Income Tracking Mode is configured HERE on the Income sheet,
             not in Settings. Settings_DB stores the value, but the UI lives
             where you use it. This follows "configure where you use it" philosophy.
```

**A3: Description (changes based on mode)**
```
If Simple:  "Track one total monthly income amount - perfect for fixed salary"
If Detailed: "Track income by source - see which sources are performing"
```

---

### Simple Mode Layout (Rows 5-40)

**Visible when:** `Settings_DB!Income_Tracking_Mode = "Simple"`

**A5: Year Selector**
```
Year: [2025 â–¼]
```

**A7-E19: Monthly Income Table**
```
Headers: Month | Planned | Actual | Variance | Status

Jan 25 | $3,500 | $3,500 | $0      | ✓
Feb 25 | $3,500 | $3,500 | $0      | ✓
Mar 25 | $3,500 | $3,450 | -$50    | ⚠️
Apr 25 | $4,200 | $4,200 | $0      | ✓ [Edit]
May 25 | $4,200 | $4,200 | $0      | ✓
Jun 25 | $4,200 | $4,200 | $0      | ✓
Jul 25 | $4,200 | $4,200 | $0      | ✓
Aug 25 | $4,200 | $4,180 | -$20    | ✓
Sep 25 | $4,200 | $4,200 | $0      | ✓
Oct 25 | $4,200 | $2,100 | -$2,100 | Partial ← Current
Nov 25 | $4,200 | -      | -       | Pending
Dec 25 | $4,200 | -      | -       | Pending
```

**Visual Specs:**
- Month: 11pt Regular Black
- Numbers: Right-aligned, $#,##0 format
- Status: Center-aligned
  - ✓ Green: Met target (Variance ≥ 0)
  - ⚠️ Orange: Missed by <15%
  - 🚨 Red: Missed by ≥15%
  - "Partial": Current month in progress
  - "Pending": Future month

**A21: Summary Card**
```
┌────────────────────────────────────────────────┐
│ 2025 SUMMARY                                   │
│ Average Planned: $4,033/month                  │
│ Average Actual:  $3,994/month (-1.0%)          │
│ Hit Target: 9 of 10 months ✓                   │
└────────────────────────────────────────────────┘
```

**A24: Income Change Log**
```
"💡 Your planned income changed in April (promotion)"
```

---

### Detailed Mode Layout (Rows 5-60)

**Visible when:** `Settings_DB!Income_Tracking_Mode = "Detailed"`

**A5: Year Selector**
```
Year: [2025 â–¼]
```

**A7-F19: Monthly Summary Table**
```
Headers: Month | Planned | Actual | Variance | Status | [+] Sources

Jan 25 | $3,500 | $3,500 | $0      | ✓      | [▶]
Feb 25 | $3,500 | $3,500 | $0      | ✓      | [▶]
Mar 25 | $3,500 | $3,450 | -$50    | ⚠️     | [▶]
Apr 25 | $4,200 | $4,200 | $0      | ✓ [Edit] | [▶]
May 25 | $5,000 | $4,850 | -$150   | ⚠️     | [▼] ← Expanded
...
```

**A21-E28: Source Breakdown (when expanded)**
```
[▼] May 2025 Breakdown:

┌─────────────────┬──────────┬──────────┬───────────┬─────────┐
│ Source          │ Expected │ Received │ Variance  │ Txns    │
├─────────────────┼──────────┼──────────┼───────────┼─────────┤
│ Salary          │ $4,200   │ $4,200   │ $0   ✓   │ 1       │
│ Freelance       │ $800     │ $650     │ -$150 ⚠️  │ 2       │
├─────────────────┼──────────┼──────────┼───────────┼─────────┤
│ TOTAL           │ $5,000   │ $4,850   │ -$150 ⚠️  │ 3       │
└─────────────────┴──────────┴──────────┴───────────┴─────────┘
```

**A30: [⚙️ Manage Income Sources] Button**
```
Opens source management interface (A32-E50)
```

**A32-E50: Income Sources Management (conditional visibility)**
```
INCOME SOURCES MANAGEMENT

Active Sources:
┌────┬────────────────┬──────────┬────────┬─────────┐
│ #  │ Source Name    │ Expected │ Active │ Actions │
├────┼────────────────┼──────────┼────────┼─────────┤
│ 1  │ Salary         │ $4,200   │ ☑      │ [Edit]  │
│ 2  │ Freelance      │ $800     │ ☑      │ [Edit]  │
│ 3  │ [Add Source]   │ $0       │ ☐      │         │
│ 4  │ [Add Source]   │ $0       │ ☐      │         │
│ 5  │ [Add Source]   │ $0       │ ☐      │         │
│ 6  │ [Add Source]   │ $0       │ ☐      │         │
│ 7  │ [Add Source]   │ $0       │ ☐      │         │
│ 8  │ [Add Source]   │ $0       │ ☐      │         │
└────┴────────────────┴──────────┴────────┴─────────┘

Maximum 8 sources
Expected shows most recent planned amount

[Back to Income Tracking]
```

---

### Edit Planned Income Form (Popup/Inline)

**Triggered by:** Clicking [Edit] button on any month

```
┌─────────────────────────────────────────────────┐
│ EDIT PLANNED INCOME - APRIL 2025                │
├─────────────────────────────────────────────────┤
│                                                 │
│ Current Expected: $3,500/month                  │
│ (based on entry from 2025-01-01)                │
│                                                 │
│ New Expected Amount: $[4,200]                   │
│                                                 │
│ Starting When?                                  │
│ ◉ April 2025 (this month)                       │
│ ○ Custom date: [____-__-__]                     │
│                                                 │
│ Apply to Future Months?                         │
│ ◉ Yes - All future months (raises/promotions)   │
│ ○ No - This month only (one-time bonuses)       │
│                                                 │
│ Reason (optional): [Got promotion]              │
│                                                 │
│ Preview Impact:                                 │
│ ┌──────────┬─────────────┬────────────┐        │
│ │ Period   │ Before Edit │ After Edit │        │
│ ├──────────┼─────────────┼────────────┤        │
│ │ Jan-Mar  │ $3,500      │ $3,500     │ (no change) │
│ │ Apr 2025+│ $3,500      │ $4,200     │ (updated)   │
│ └──────────┴─────────────┴────────────┘        │
│                                                 │
│ [💾 Save Change] [Cancel]                       │
└─────────────────────────────────────────────────┘
```

**On Save:**
```
Creates new row in Income_Plan_History:
  Start_Date: 2025-04-01
  Source_ID: 1 (or selected source)
  Planned_Amount: 4200
  Note: "Got promotion"

All formulas automatically use new value for months ≥ 2025-04
```

---

## 🔧 Key Formulas

### Get Planned Income for Any Month (Simple Mode)

```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Start_Date] <= DATEVALUE(Month_ID&"-01")), 
    Income_Plan_History[Planned_Amount]),
  Settings_DB!Typical_Monthly_Income)

Where:
  Month_ID = "2025-04" (format YYYY-MM)
  Returns: Most recent Planned_Amount where Start_Date ≤ 2025-04-01
```

### Get Planned Income for Source (Detailed Mode)

```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Source_ID] = SourceID) *
      (Income_Plan_History[Start_Date] <= DATEVALUE(Month_ID&"-01")), 
    Income_Plan_History[Planned_Amount]),
  0)
```

### Actual Income Received (Simple Mode)

```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], Month_ID,
        Transaction_DB[Type], "Income",
        Transaction_DB[Deleted?], FALSE)
```

### Actual Income by Source (Detailed Mode)

```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], Month_ID,
        Transaction_DB[Category], "Income: "&Source_Name,
        Transaction_DB[Deleted?], FALSE)
```

### Variance

```excel
=Actual - Planned
```

### Status Indicator

```excel
=IF(Actual = 0,
   IF(DATEVALUE(Month_ID&"-01") <= TODAY(), "Missing", "Pending"),
   IF(Variance >= 0, "✓",
      IF(Variance/Planned > -0.15, "⚠️", "🚨")))
```

### Year Summary - Average Planned

```excel
=AVERAGE(Planned_Jan:Planned_Dec)
```

### Year Summary - Average Actual

```excel
=AVERAGE(Actual_Jan:Actual_Dec)
```

### Months Hit Target

```excel
=COUNTIFS(Status_Range, "✓") & " of " & 
 COUNTIFS(Actual_Range, "<>0") & " months"
```

---

## ✅ Acceptance Tests (20 tests)

### Simple Mode Tests (8)

1. **Initial Setup - Simple Mode**
   - Setup wizard: Choose Simple, enter $4,000
   - ✅ Settings_DB!Income_Tracking_Mode = "Simple"
   - ✅ Settings_DB!Typical_Monthly_Income = 4000
   - ✅ Income_Plan_History entry created (Start_Date = Today)
   - ✅ Income sheet displays Simple mode layout

2. **Planned Amount Display**
   - ✅ All 12 months show $4,000 in Planned column
   - ✅ Formula references Income_Plan_History

3. **Add Income Transaction**
   - Add transaction: Category = "Income", Amount = $4,000
   - ✅ Actual column updates for that month
   - ✅ Variance = $0
   - ✅ Status = "✓"

4. **Partial Income (Under Target)**
   - Current month shows $2,000 actual (vs $4,000 planned)
   - ✅ Variance = -$2,000
   - ✅ Status = "Partial"

5. **Edit Planned Income - Promotion**
   - Click [Edit] on April
   - Enter $4,500
   - Select "Yes - All future months"
   - Save
   - ✅ Jan-Mar remain $4,000
   - ✅ Apr-Dec update to $4,500
   - ✅ Next year (2026) also shows $4,500

6. **Edit Planned Income - One-Time Bonus**
   - Click [Edit] on December
   - Enter $5,000
   - Select "No - This month only"
   - Save
   - ✅ Only December shows $5,000
   - ✅ January 2026 reverts to previous amount

7. **Year Selector**
   - Change Year dropdown to 2024
   - ✅ Table shows 2024 months
   - ✅ Planned amounts reflect 2024 Income_Plan_History
   - Change to 2026
   - ✅ Table shows 2026 months
   - ✅ Future months show "Pending"

8. **Summary Card Calculations**
   - ✅ Average Planned = SUM(Planned)/12
   - ✅ Average Actual = SUM(Actual where Actual>0) / COUNT(Actual>0)
   - ✅ Hit Target count = months where Variance ≥ 0

### Detailed Mode Tests (8)

9. **Initial Setup - Detailed Mode**
   - Setup wizard: Choose Detailed
   - Select: Salary ($4,000), Freelance ($800)
   - ✅ 2 entries in Income_Sources_DB (Active = TRUE)
   - ✅ 2 categories created: "Income: Salary", "Income: Freelance"
   - ✅ 2 Income_Plan_History entries
   - ✅ Income sheet displays Detailed mode layout

10. **Monthly Summary Table**
    - ✅ Planned column = SUM of all source plans
    - ✅ January shows $4,800 ($4,000 + $800)

11. **Add Income by Source**
    - Add transaction: Category = "Income: Salary", Amount = $4,000
    - Add transaction: Category = "Income: Freelance", Amount = $750
    - ✅ Actual column = $4,750
    - ✅ Variance = $4,750 - $4,800 = -$50

12. **Expand Source Breakdown**
    - Click [▼] on May
    - ✅ Breakdown section displays
    - ✅ Salary row: Expected $4,000, Received $4,000, Variance $0
    - ✅ Freelance row: Expected $800, Received $750, Variance -$50
    - ✅ Total matches monthly summary

13. **Collapse Source Breakdown**
    - Click [▶] on expanded May
    - ✅ Breakdown section hides
    - ✅ Summary row remains

14. **Add New Income Source**
    - Click [⚙️ Manage Income Sources]
    - Click row 3, enter "Rental Income", $1,200
    - Check Active box
    - ✅ Income_Sources_DB row 3 updates
    - ✅ Category "Income: Rental Income" created
    - ✅ Income_Plan_History entry added
    - ✅ Future months show $6,000 planned ($4,000 + $800 + $1,200)

15. **Edit Source Amount**
    - Click [Edit] on Freelance source
    - Change from $800 to $1,000
    - ✅ New Income_Plan_History entry (Start_Date = Today)
    - ✅ Current and future months update to $1,000

16. **Deactivate Source**
    - Uncheck Active on Rental Income
    - ✅ Source grays out
    - ✅ Historical data preserved
    - ✅ Future planned amounts exclude this source
    - ✅ Category "Income: Rental Income" remains (not deleted)

### Mode Switching Tests (4)

17. **Switch Simple → Detailed**
    - Start in Simple mode with $5,000/month
    - Click [Switch to Detailed]
    - ✅ Prompt: "Convert to Detailed mode?"
    - Confirm
    - ✅ Creates 1 source: "Primary Income", $5,000
    - ✅ Income_Sources_DB row 1 populated
    - ✅ Layout changes to Detailed mode
    - ✅ Historical data preserved

18. **Switch Detailed → Simple**
    - Start in Detailed with 2 sources ($4,000 + $800)
    - Click [Switch to Simple]
    - ✅ Prompt: "Combine sources into single income?"
    - Confirm
    - ✅ Settings_DB!Typical_Monthly_Income = $4,800 (sum)
    - ✅ Income_Sources_DB marked inactive
    - ✅ Layout changes to Simple mode
    - ✅ Historical data preserved

19. **Switch Preserves Transactions**
    - Have 10 income transactions (various categories)
    - Switch Simple ↔ Detailed
    - ✅ All 10 transactions still in Transaction_DB
    - ✅ Categories unchanged
    - ✅ Amounts calculated correctly in both modes

20. **Switch Preserves History**
    - Have 3 Income_Plan_History entries (Jan, Apr, Jul)
    - Switch modes
    - ✅ All 3 entries remain in Income_Plan_History
    - ✅ Can still see April promotion in either mode
    - ✅ Can edit/add new entries in either mode

---

## 🚧 Known Issues / Blockers

*Document during build.*

---

## 📝 Implementation Checklist

### Step 1: Create Database Tables
- [ ] Income_Plan_History table (Settings_DB, rows 50-100)
- [ ] Income_Sources_DB table (Settings_DB, rows 10-20)
- [ ] Load initial sample data (1 simple source or 2 detailed sources)

### Step 2: Create Income Sheet Structure
- [ ] Title and mode selector (rows 1-3)
- [ ] Simple mode layout (rows 5-40)
- [ ] Detailed mode layout (rows 5-60)
- [ ] Edit income form (popup or inline)
- [ ] Manage sources interface

### Step 3: Add Formulas
- [ ] Planned income lookup (Income_Plan_History)
- [ ] Actual income calculation (SUMIFS Transaction_DB)
- [ ] Variance and status formulas
- [ ] Summary card calculations
- [ ] Source breakdown calculations (Detailed mode)

### Step 4: Mode Switching Logic
- [ ] Switch to Detailed button/logic
- [ ] Switch to Simple button/logic
- [ ] Data conversion routines
- [ ] Layout show/hide based on mode

### Step 5: Integration with Other Sheets
- [ ] Setup wizard: Income selection step
- [ ] Budget tab: Reference Income sheet for method
- [ ] Dashboard: Pull income data for Chart 4
- [ ] Settings: Store Income_Tracking_Mode

### Step 6: Testing
- [ ] Run all 20 acceptance tests
- [ ] Verify mode switching preserves data
- [ ] Test promotion/raise scenarios
- [ ] Test multi-source tracking

---

## 🎯 Success Criteria

Phase 8 complete when all 20 tests pass and:
- Both Simple and Detailed modes functional
- Mode switching works correctly
- Income history tracks promotions/raises
- Integration with Dashboard and Budget confirmed

---

## 📞 Next Phase

**Phase 7: Dashboard** (already completed)  
Uses: Income data for Chart 4 (Income Performance)  
Already specified: Chart 7 (Income by Source) conditional on Detailed mode
