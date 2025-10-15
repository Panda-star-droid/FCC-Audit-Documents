# 📘 Finance Control Center - Project Understanding
**Audit Phase 1 Output**  
**Date:** 2025-10-14  
**Status:** 🟡 In Progress

---

## 1. Executive Summary

### Product Definition
**Name:** Finance Control Center  
**Version:** 2.1 No-VBA Edition  
**Price:** $24.99 (one-time)  
**Core Value:** Professional personal finance tracking without subscriptions, macros, or complexity

### Key Differentiators
- **9-second transaction entry** (keyboard-optimized)
- **100% formula-driven** (works on Excel 2016+, Google Sheets, Mac, Windows, web)
- **Cross-platform** (desktop, web, mobile view-optimized)
- **Offline-capable** (no internet required for Excel desktop)

### Target Audience
- Budget-conscious professionals ($40K-$100K income)
- Side hustlers with multiple income streams
- Couples managing joint finances
- Users frustrated with subscription apps but want better than basic spreadsheets

### Market Positioning
- **vs. Mint/YNAB:** One-time $24.99 vs $10-15/month subscription = payback in 2-3 months
- **vs. Basic Spreadsheets:** Professional features with 5-minute setup vs hours of DIY
- **vs. Desktop Apps:** $24.99 vs $79.99+ (QuickBooks/Banktivity pricing)

---

## 2. Architecture Overview

### System Structure
**19 Total Sheets:**
- **9 User-Facing:** Instructions, Settings, Income, Budget, Dashboard, Quick Entry, Debt, Net Worth, CSV Import
- **10 Backend DBs (Hidden):** Transaction_DB, Debt_DB, Debt_Payments_DB, Net_Worth_Snapshots_DB, Budget_Allocations_DB, Goals_DB, Income_Sources_DB, Income_Plan_History, Categories_DB, Settings_DB, Keywords_DB

### Critical Dependencies
```
Phase 1: Foundation → Provides base data
    ↓
Phase 2: Transactions → Core transaction system
    ↓
Phase 3: Utilities → Provides config (Date format, Currency)
    ↓
Phase 4: Income → Provides income baseline
    ↓
Phase 5: Budget → Needs transactions, income, settings
    ↓
Phase 6: Debt → Writes to Transaction_DB
    ↓
Phase 7: Net Worth → Independent tracking
    ↓
Phase 8: Dashboard → Reads from ALL systems
```

### Named Range Strategy
**100+ critical named ranges** enable:
- Cross-sheet formula references
- Dynamic dropdown lists
- Period filtering (Week/Month/Quarter/Year)
- Data validation rules

---

## 3. Design Principles

### Visual Framework (WCAG AA Compliant)

**Core Philosophy:** Formula-driven, no animations, instant feedback

### Color System
**Primary:**
- Navy #1A2B4A (headers, save cells) - 14.1:1 contrast (AAA)
- Gold #D4AF37 (accents) - 2.1:1 contrast (FAIL on white - accents only)
- Lt Blue #E8F1F8 (backgrounds)

**Status:**
- Green #2E7D32 (success, income, on-budget)
- Orange #F57C00 (warning, 80-100% budget)
- Red #C62828 (alert, over-budget, expense)

**15 Category Colors:** Navy, Gold, Green, Red, Orange, Blue, Purple, Teal, Pink, Brown, Indigo, Cyan, Lime, Amber, Gray

### Typography
- **Fonts:** Calibri (Excel), Roboto (Sheets)
- **Scale:** 36pt Display (KPIs), 18pt H1, 14pt H2, 12pt H3, 11pt Body, 10pt Small, 9pt Micro
- **Formatting:** Right-align numbers, left-align text, center-align dates
- **Currency:** $#,##0.00 (red negative, no parentheses)

### Component Specs
**Save Cells (Valid):**
- Background: Navy #1A2B4A
- Text: White #FFFFFF, 11pt Bold
- Height: 44px minimum (touch target)
- Content: "💾 Tap to Save"

**Save Cells (Invalid):**
- Background: Lt Gray #F5F5F5
- Text: Md Gray #757575, 11pt Regular
- Content: "Fill required fields..."

**Status Cells:**
- Success: "✅ Saved to Oct 2025 - Dining Out" (Green)
- Error: "✕ Amount is required" (Red)
- Auto-clear: 3 seconds

**Forms:**
- Input height: 32px
- Padding: 8px horizontal
- Border: 1px Md Gray (normal), 2px Navy + Gold outline (focus)

**Tables:**
- Header: 32px, Navy bg, 12pt Bold White
- Data rows: 24px, alternating White/Lt Blue
- Alignment: Text left, numbers right, dates center

**Charts:**
- Line stroke: 2pt
- Markers: 6pt circles
- Colors: Income (Gold), Expenses (Red), Savings (Green)
- Gridlines: Lt Gray, horizontal only

### Icons (Unicode Only)
- Status: ✓ ✕ ⚠ ℹ ▲ ▼
- Actions: ➕ ➖ 🔍 ▶️ 💾
- **Critical:** NO emoji for categories (inconsistent rendering)

### Layout
- **Grid:** 12-column system
- **Spacing:** 8px base unit (0, 4, 8, 12, 16, 24, 32, 48, 64px)
- **Freeze panes:** Dashboard row 7, Quick Entry row 6, Budget row 4

### Accessibility
- Contrast: WCAG AA minimum (4.5:1 normal, 3:1 large)
- Touch targets: 44px minimum
- Focus: 2px Gold outline always visible
- Tab order: Top→bottom, left→right

---

## 4. Formula Library

**11 Universal Patterns** used across all phases (100% formula-driven, no VBA):

### 4.1 Save Cell Pattern
**Used in:** All forms (Quick Entry, Budget, Debt, Net Worth, Goals, CSV Import)
- **Display:** `IF(Valid, "💾 Tap to Save", "Fill required fields...")`
- **Trigger Detection:** Edit counter in Z1-Z3 helper cells (Z1≠Z2 = save triggered)
- **Append Logic:** DB formula checks trigger, appends form data if valid
- **Visual States:** Navy bg (valid), Gray bg (invalid), Gold outline (focus)
- **Touch Target:** 44px minimum height (WCAG compliance)

### 4.2 Status Cell Pattern
**Used in:** All forms for feedback
- **Success:** `"✅ Saved to Oct 2025 - Dining Out"` (Green, auto-clear 3s)
- **Error:** `"✕ Amount is required"` (Red, persists until fixed)
- **Info:** `"Ready"` (Gray, form valid but not saved)

### 4.3 Auto-Clear Pattern
**Used in:** Form field reset after save
- **Time-Based:** `IF(NOW()-SaveTime>2s, "", Value)` (smooth UX)
- **Immediate:** `IF(TriggerDetected, "", UserEntry)` (simpler)

### 4.4 Duplicate Detection Pattern
**Used in:** CSV Import validation
- **Logic:** `COUNTIFS(DB[Date]=Date, DB[Amount]=Amount, DB[Desc] contains First20Chars) > 0`
- **Result:** "Duplicate" or "New" flag
- **Visual:** Orange background for duplicates, green for ready

### 4.5 Period Filter Pattern
**Used in:** Dashboard, Budget, Charts
- **Period Types:** Week | Month | Quarter | Year
- **Start Date:** `CHOOSE(MATCH(Period), Week=Sunday, Month=1st, Quarter=3mo, Year=Jan1)`
- **End Date:** `CHOOSE(MATCH(Period), Week=Today, Month=LastDay, Quarter=LastDay, Year=Dec31)`
- **Usage:** `SUMIFS(DB[Amount], DB[Date]>=Start, DB[Date]<=End)`

### 4.6 Budget Calculation Pattern
**Used in:** Budget sheet, Dashboard KPIs
- **50/30/20 Method:** Needs=50%, Wants=30%, Savings=20% (split evenly per category type)
- **Zero-Based:** Manual entry per category (must sum to income)
- **Custom %:** Percentage per category (must sum to 100%)
- **Actual:** `SUMIFS(Transactions, Category=X, Period) * -1` (negate expense sign)
- **Status:** Over (red), Near Limit >90% (orange), On Track (green)

### 4.7 Debt Scenario Pattern
**Used in:** Debt sheet planning
- **Months to Debt-Free:** `NPER(APR/12, -Payment, Balance)` (accurate compound interest)
- **Total Interest:** `(Months * Payment) - Balance`
- **Payoff Order:** Avalanche (sort by APR desc), Snowball (sort by balance asc)

### 4.8 Goal Progress Pattern
**Used in:** Budget goals, Dashboard widget
- **Current:** `SUMIFS(Transactions, Category="Goal: "+Name)` (goal contributions = income)
- **Progress:** `MIN(100, Current/Target*100)` (capped at 100%)
- **Monthly Needed:** `MAX(0, (Target-Current) / MonthsRemaining)`
- **Status:** Goal Reached | Past Deadline | Very Aggressive | On Track

### 4.9 Net Worth Projection Pattern
**Used in:** Net Worth forecasting
- **Future Value:** `CurrentNW * (1+Rate)^Years + Contribution * (((1+Rate)^Years-1)/Rate)`
- **Month-over-Month:** `Current - VLOOKUP(LastMonth, Snapshots[Date:NW], 5)`
- **Change %:** `(Current - Previous) / ABS(Previous)`

### 4.10 CSV Import Normalization Pattern
**Used in:** CSV Import cleanup
- **Date:** `DATEVALUE(Raw)` fallback to `DATE(YYYY,MM,DD)` parsing (handles 3 formats)
- **Amount:** `VALUE(SUBSTITUTE(Raw, "$", ""), ",", ""))` (strip currency symbols)
- **Type:** `IF(Amount>0, "Income", "Expense")`
- **Category:** `XLOOKUP(DESC, Keywords[Keyword], Keywords[Cat], "Uncategorized", wildcard)`

### 4.11 Income Tracking Pattern (V11)
**Used in:** Income sheet planning
- **Planned Lookup:** `LOOKUP(2, 1/(History[Date]<=Month), History[Planned])` (most recent plan)
- **Actual:** `SUMIFS(Transactions, Month_ID=X, Type="Income")`
- **Variance:** `Actual - Planned`
- **Status:** ✓ Met | ⚠️ Missed<15% | 🚨 Missed≥15% | Missing | Pending

### Quick Reference Matrix
| Pattern | Trigger | Primary Sheet | Key Named Range |
|---------|---------|---------------|-----------------|  
| Save Cell | User edit | All forms | [Form]_SaveCell |
| Period Filter | Dropdown | Settings | Period_Selected, Period_Start, Period_End |
| Budget 50/30/20 | Income change | Budget | Budget_Method, Monthly_Income |
| Debt NPER | Input change | Debt | Total_Balance, Total_Min_Payments |
| Goal Progress | Transaction | Budget | Goals_DB[Target_Amount] |
| CSV Normalize | Import | CSV Import | Keywords_DB[Keyword:Category] |
| Income Variance | Month end | Income | Income_Plan_History[Date:Planned] |

---

## 5. Sheet Inventory (19 Sheets)

### 5.1 User-Facing Sheets (9)

#### Sheet 1: Instructions
**Phase:** 3 (Utilities) | **Purpose:** Onboarding and help
- **Content:** Video tour, quick start guide, FAQ, troubleshooting
- **Layout:** Read-only, hyperlinked sections, minimal interaction
- **Dependencies:** None (standalone reference sheet)

#### Sheet 2: Settings
**Phase:** 3 (Utilities) | **Purpose:** System-wide configuration
- **Sections:** Currency/locale (3 settings), Category management (30 max), Data management (delete/reset/archive/export)
- **Key Features:** Save buttons (44px touch target), category CRUD, delete confirmation dialogs
- **Formulas:** Minimal (mostly UI), category count validation
- **Dependencies:** Reads/writes Settings_DB, Categories_DB
- **Named Ranges:** Currency_Symbol, Date_Format, Week_Start_Offset

#### Sheet 3: Income
**Phase:** 4 (Income Sheet) | **Purpose:** Income planning and tracking
- **Modes:** Simple (1 total) or Detailed (up to 8 sources)
- **Simple Layout:** 12-month table (Planned/Actual/Variance/Status), year selector, edit buttons
- **Detailed Layout:** Same + expandable source breakdown, source management interface
- **Key Formulas:** LOOKUP for planned (finds most recent ≤ month), SUMIFS for actual
- **Dependencies:** Income_Plan_History (plans), Income_Sources_DB (sources), Transaction_DB (actuals)
- **Critical:** Promotions carry forward indefinitely via Income_Plan_History

#### Sheet 4: Budget
**Phase:** 5 (Budget System) | **Purpose:** Budget planning and goal tracking
- **Budget Section:** Method selector (50/30/20, Zero-Based, Custom%), 12-category table, charts (Budget vs Actual, Spending Mix)
- **Goals Section:** Up to 10 active goals, progress bars, contribution logging, monthly savings calculator
- **Custom Allocation:** Conditional visibility (Custom % only), validation (must sum to 100%)
- **Key Formulas:** Budget calc (50/30/20 split), SUMIFS actual, goal progress (SUMIFS contributions)
- **Dependencies:** Budget_Allocations_DB, Goals_DB, Transaction_DB, Categories_DB
- **Named Ranges:** Budget_Method, Monthly_Income, Active_Goals

#### Sheet 5: Dashboard
**Phase:** 8 (Final Integration) | **Purpose:** View-only financial overview
- **Layout:** Title + period selector, 4 KPI cards, recent transactions, 6-7 charts (16 rows each)
- **Charts:**
  1. Financial Overview (dual-axis line): Income/Expenses/Savings + Savings Rate %
  2. Budget Spending (horizontal bars): Top 8 categories with variance indicators
  3. Spending Composition (treemap/donut): Proportional spending breakdown
  4. Income Performance (grouped columns): Planned vs Actual last 6 months
  5. Debt Payoff (stacked area): Historical + projected with strategy selector
  6. Net Worth Projection (area + cone): Historical + 5-year forecast
  7. Income by Source (stacked columns): *Conditional - Detailed mode only*
- **Key Features:** Period filter (Week/Month/Quarter/Year), Month_ID filtering, chart 7 conditional display
- **Dependencies:** ALL phases (1-7), reads from all DBs
- **Critical:** View-only (no data entry), must be built last

#### Sheet 6: Quick Entry
**Phase:** 2 (Transaction System) | **Purpose:** All transaction data entry
- **Forms:** Transaction entry (row 9), Pay Debt (rows 22-35), Add Debt (rows 37-50)
- **Save Pattern:** Navy save cell (valid), gray (invalid), Z1-Z4 trigger detection
- **Recent Transactions:** Last 50 non-deleted, soft delete checkboxes, highlight flash on new
- **Key Formulas:** Save cell validation, auto-clear (2-3s delay), status feedback, dual-write (Pay Debt → both Transaction_DB + Debt_Payments_DB)
- **Dependencies:** Transaction_DB (append), Debt_DB (for Pay Debt dropdown), Categories_DB (dropdowns)
- **Named Ranges:** QuickAdd_[Field] for each form input

#### Sheet 7: Debt
**Phase:** 6 (Debt System) | **Purpose:** Debt tracking and payoff planning
- **Sections:** Snapshot cards (4), debt list table, strategy comparison (Snowball vs Avalanche), scenario planner
- **Snapshot Cards:** Total Balance, Weighted APR, Debt-Free ETA, Total Paid
- **Strategy Cards:** Side-by-side comparison with pros/cons, results (months, interest, first payoff)
- **Scenario Planner:** Extra payment slider (unlimited), strategy selector, charts (months to debt-free, interest paid)
- **Key Formulas:** Weighted APR (SUMPRODUCT), NPER for payoff, interest saved calculator
- **Dependencies:** Debt_DB, Debt_Payments_DB, Transaction_DB (dual-write integration)

#### Sheet 8: Net Worth
**Phase:** 7 (Net Worth System) | **Purpose:** Asset/liability tracking and wealth projection
- **Hero Section:** Current net worth (36pt), MoM change with arrow, log snapshot button
- **Account Categories:** Cash & Checking, Investments, Property (assets) | Credit Cards, Loans (liabilities from Debt_DB)
- **Projection:** Settings (monthly contrib, growth rate, period), FV formula, results display
- **Charts:** Net worth trend (line, historical + projected), asset allocation (donut), category summary table
- **Key Formulas:** Net worth = Assets - Liabilities, MoM change (current vs VLOOKUP previous), projection (FV with contribution)
- **Dependencies:** Net_Worth_Snapshots_DB (historical), Debt_DB (liabilities auto-sync)
- **Critical:** Liabilities link to Debt_DB (auto-update on debt payment)

#### Sheet 9: CSV Import
**Phase:** 3 (Utilities) | **Purpose:** Bulk transaction import from bank CSV
- **Workflow:** Paste raw CSV → Normalize (date/amount) → Auto-categorize (Keywords_DB) → Preview with duplicate detection → Import selected
- **Normalization:** Date (handles 3 formats), Amount (strip $, commas), Type (Income vs Expense)
- **Auto-Categorize:** Wildcard match description against Keywords_DB[Keyword:Category]
- **Preview Table:** Status column (Ready/Duplicate/Needs Category), color-coded rows
- **Key Formulas:** DATEVALUE fallback, SUBSTITUTE for cleanup, COUNTIFS for duplicate detection, XLOOKUP/INDEX-MATCH for keywords
- **Dependencies:** Keywords_DB (auto-cat), Transaction_DB (import target), Categories_DB (validation)

---

### 5.2 Backend Database Sheets (10 - Hidden)

#### DB 1: Transaction_DB
**Phase:** 2 | **Columns:** 12 (A-L)
- **Schema:** ID, Date, Month_ID (auto), Type, Description, Category, Amount, Account, From/To, Timestamp, Deleted?, ID
- **Month_ID Formula:** `=TEXT(Date,"YYYY-MM")` (enables fast period filtering)
- **Key Behavior:** Soft delete (Deleted?=TRUE), append-only (formulas in row 2+)
- **Named Ranges:** Transaction_DB[Amount], Transaction_DB[Month_ID], Transaction_DB[Category], Transaction_DB[Deleted?]
- **Indexes:** Month_ID (critical for Dashboard performance)
- **Sample:** 50+ sample transactions across 6 months

#### DB 2: Debt_DB
**Phase:** 6 | **Columns:** 11 (A-K)
- **Schema:** ID, Name, Month_ID (auto from Start_Date), Orig_Bal, Curr_Bal (calc), APR, Min_Pay, Start_Date, Status, Last_Pay_Date, Created
- **Curr_Bal Formula:** `Orig_Bal - SUMIFS(Debt_Payments_DB[Amount], Debt_ID=This_ID)`
- **Named Ranges:** Debt_DB[Curr_Bal], Debt_DB[APR], Active_Debts (FILTER where Status="Active")
- **Sample:** 2 debts (Credit Card A $7,500 @ 22%, Credit Card B $2,500 @ 11%)

#### DB 3: Debt_Payments_DB
**Phase:** 6 | **Columns:** 7 (A-G)
- **Schema:** ID, Debt_ID (FK), Date, Month_ID (auto), Amount, From_Acct, Transaction_ID (links Transaction_DB)
- **Purpose:** Payment history log, dual-write partner of Transaction_DB
- **Named Ranges:** Debt_Payments_DB[Amount], Debt_Payments_DB[Debt_ID], Debt_Payments_DB[Month_ID]
- **Sample:** 8 payments across 2 debts

#### DB 4: Net_Worth_Snapshots_DB
**Phase:** 7 | **Columns:** 9 (A-I)
- **Schema:** ID, Date, Month_ID (auto), Total_Assets, Total_Liab, Net_Worth, Assets_JSON (serialized breakdown), Liab_JSON, Notes
- **Month_ID Formula:** `=TEXT(Date,"YYYY-MM")`
- **Purpose:** Monthly wealth tracking, powers Dashboard Chart 6 + Net Worth sheet trend
- **Named Ranges:** Net_Worth_Snapshots_DB[Net_Worth], Net_Worth_Snapshots_DB[Date], Latest_Snapshot
- **Sample:** 2 snapshots (current + previous month)

#### DB 5: Budget_Allocations_DB
**Phase:** 5 | **Columns:** 8 (A-H)
- **Schema:** ID, Month_ID, Method (50/30/20/Zero/Custom), Category, Percentage, Allocated_Amt, Effective_Date, Status
- **Month_ID Formula:** `=TEXT(Effective_Date,"YYYY-MM")`
- **Purpose:** Store budget allocations per month per category
- **Named Ranges:** Budget_Allocations_DB[Category], Budget_Allocations_DB[Allocated_Amt], Active_Allocations
- **Sample:** 12 categories with 50/30/20 splits

#### DB 6: Goals_DB
**Phase:** 5 | **Columns:** 10 (A-J)
- **Schema:** ID, GoalName, TargetAmt, CurrentAmt (calc from TX), Deadline, Created, Status, CategoryLinked, Priority, Notes
- **CurrentAmt Formula:** `SUMIFS(Transaction_DB[Amount], Category="Goal: "+GoalName)`
- **Purpose:** Track savings goals with progress
- **Named Ranges:** Goals_DB[GoalName], Goals_DB[TargetAmt], Active_Goals (FILTER Status="Active")
- **Sample:** 2 goals (Emergency Fund $10K, Vacation $3K)

#### DB 7: Income_Sources_DB
**Phase:** 4 | **Location:** Settings_DB sheet, rows 10-20
- **Schema:** ID (1-8 fixed), Source_Name, Expected_Monthly, Active, Category_Created
- **Purpose:** Define income sources for Detailed mode (8 pre-allocated rows)
- **Key Behavior:** Inactive rows show "[Available]", no deletion (just toggle Active)
- **Sample:** Row 1: Salary $4200 Active, Row 2: Freelance $800 Active, Rows 3-8: [Available]

#### DB 8: Income_Plan_History
**Phase:** 4 | **Location:** Settings_DB sheet, rows 50-100
- **Schema:** ID, Start_Date, Source_ID (FK), Planned_Amount, Note
- **Purpose:** Track income changes over time (promotions, raises) with indefinite carry-forward
- **Key Behavior:** No end date, LOOKUP finds most recent entry ≤ target month, sorted by Start_Date (CRITICAL)
- **Indexes:** Composite on (Source_ID, Start_Date) for fast LOOKUP
- **Sample:** {2025-01-01, Source 1, $3500, "Initial"}, {2025-04-01, Source 1, $4200, "Promotion"}

#### DB 9: Categories_DB
**Phase:** 1 | **Location:** Hidden sheet or Settings_DB rows 120-150
- **Schema:** ID, Name, Type (Income/Expense/Savings/Transfer), Color, Icon, Active
- **Purpose:** Master category list (max 30)
- **Default:** 12 categories (Housing, Groceries, Dining, Transport, Entertainment, Shopping, Healthcare, Personal, Utilities, Savings, Debt, Income)
- **Named Ranges:** Categories_DB[Name], Categories_DB[Type], Active_Categories

#### DB 10: Settings_DB
**Phase:** 1 | **Location:** Hidden sheet or Settings sheet metadata rows
- **Schema:** Key-value pairs (Setting_Name, Setting_Value)
- **Key Settings:** Currency_Symbol, Date_Format, Week_Start_Offset, Period_Selected, Budget_Method, Monthly_Income, Income_Tracking_Mode, Typical_Monthly_Income
- **Purpose:** Store all global configuration
- **Named Ranges:** Individual named ranges per setting (e.g., `Currency_Symbol = Settings_DB!B2`)

#### DB 11: Keywords_DB
**Phase:** 1 | **Location:** Hidden sheet or Settings_DB rows 200-230
- **Schema:** ID, Keyword (lowercase), Category (FK)
- **Purpose:** Auto-categorization for CSV Import (wildcard matching)
- **Sample:** 30 keywords ("starbucks" → Dining, "shell" → Transport, "netflix" → Entertainment)
- **Named Ranges:** Keywords_DB[Keyword], Keywords_DB[Category]

---

## 6. Multi-Sheet Relationships

### 6.1 Data Flow Architecture

```
LEVEL 1 (Foundation):
Categories_DB → Provides category lists to ALL sheets
Settings_DB → Provides config to ALL sheets
Keywords_DB → Powers CSV Import auto-categorization

LEVEL 2 (Core Data Entry):
Quick Entry (UI) → Appends to → Transaction_DB
                 → Dual-writes to → Debt_Payments_DB (Pay Debt form)
                 → Creates entries in → Debt_DB (Add Debt form)

LEVEL 3 (Income System):
Income_Plan_History ← Written by ← Income sheet (Edit Planned Income)
Income_Sources_DB ← Managed by ← Income sheet (Source Management)
Income sheet → Reads from → Transaction_DB (Actual income)
            → Reads from → Income_Plan_History (Planned income)

LEVEL 4 (Budget System):
Budget sheet → Reads from → Transaction_DB (Actual spending)
           → Reads from → Budget_Allocations_DB (Budget amounts)
           → Reads from → Goals_DB (Goal progress)
           → Writes to → Goals_DB (Goal contributions as Income TX)
           → Writes to → Transaction_DB (via Quick Entry link)

LEVEL 5 (Debt System):
Debt sheet → Reads from → Debt_DB (Current balances)
         → Reads from → Debt_Payments_DB (Payment history)
         ← Debt_DB.Curr_Bal calculated from ← Debt_Payments_DB

LEVEL 6 (Wealth Tracking):
Net Worth sheet → Reads from → Debt_DB (Total liabilities)
              → Writes to → Net_Worth_Snapshots_DB (Monthly snapshots)
              → Reads from → Net_Worth_Snapshots_DB (Historical trend)

LEVEL 7 (Dashboard Integration):
Dashboard → Reads from → ALL databases (view-only)
        → Period filter applies to → ALL chart queries
        → Chart 7 conditional on → Settings_DB!Income_Tracking_Mode
```

### 6.2 Critical Dependencies

**Phase Build Order (MUST follow this sequence):**
1. Phase 1 (Foundation) → No dependencies
2. Phase 2 (Transactions) → Requires Phase 1
3. Phase 3 (Utilities) → Requires Phases 1-2
4. Phase 4 (Income) → Requires Phases 1-3
5. Phase 5 (Budget) → Requires Phases 1-4 (CRITICAL: needs income baseline)
6. Phase 6 (Debt) → Requires Phases 1-2
7. Phase 7 (Net Worth) → Requires Phases 1, 3, 6
8. Phase 8 (Dashboard) → Requires ALL phases 1-7

**Why Phase 5 needs Phase 4:**
- Budget methods (50/30/20, Zero-Based) calculate from Monthly_Income
- Monthly_Income comes from Income sheet configuration
- Without Income baseline, Budget allocations cannot be calculated

### 6.3 Named Range Network

**Global Named Ranges (used across multiple sheets):**
- `Period_Selected` (Settings_DB) → Read by: Dashboard, Budget, Debt, Net Worth
- `Period_Start` (Settings_DB, calculated) → Read by: All sheets with period filtering
- `Period_End` (Settings_DB, calculated) → Read by: All sheets with period filtering
- `Currency_Symbol` (Settings_DB) → Read by: ALL sheets displaying money
- `Date_Format` (Settings_DB) → Read by: ALL sheets displaying dates
- `Budget_Method` (Settings_DB) → Read by: Budget sheet, Dashboard Chart 2
- `Monthly_Income` (Settings_DB) → Read by: Budget sheet, Dashboard KPIs
- `Income_Tracking_Mode` (Settings_DB) → Read by: Income sheet, Dashboard Chart 7 (conditional)

**Sheet-Specific Named Ranges:**
- Transaction_DB[Amount] → Referenced by: Budget, Income, Dashboard, Debt (via Debt_Payments)
- Transaction_DB[Month_ID] → Referenced by: ALL sheets with period filtering (INDEX)
- Categories_DB[Name] → Referenced by: Quick Entry dropdowns, Budget table, Dashboard charts
- Debt_DB[Curr_Bal] → Referenced by: Debt sheet KPIs, Net Worth liabilities, Dashboard Chart 5
- Active_Goals → Referenced by: Budget sheet goals section, Dashboard goals widget
- Active_Debts → Referenced by: Debt sheet calculations, Net Worth liabilities

### 6.4 Integration Points

**Dual-Write Operations (maintain data integrity):**
1. **Pay Debt (Quick Entry)**
   - Writes to Transaction_DB: Expense, Category="Debt Payment", negative amount
   - Writes to Debt_Payments_DB: Links Debt_ID, positive amount, Transaction_ID
   - Triggers: Debt_DB.Curr_Bal recalculation
   - Triggers: Net Worth liabilities update (if Debt_DB linked)

2. **Log Goal Contribution (Budget sheet)**
   - Writes to Transaction_DB: Income, Category="Goal: [Name]", positive amount
   - Triggers: Goals_DB.CurrentAmt recalculation (via SUMIFS)
   - Triggers: Dashboard Chart 1 (income increases)

3. **Add Debt (Quick Entry)**
   - Writes to Debt_DB: New debt entry
   - Triggers: Net Worth liabilities increase
   - Triggers: Debt sheet snapshot card updates
   - Triggers: Dashboard Chart 5 new debt appears

**Calculated Dependencies (formulas that depend on other sheets):**
1. **Debt_DB.Curr_Bal** ← `Orig_Bal - SUMIFS(Debt_Payments_DB[Amount])`
2. **Goals_DB.CurrentAmt** ← `SUMIFS(Transaction_DB[Amount], Category="Goal: "+Name)`
3. **Net Worth Total Liabilities** ← `SUM(Debt_DB[Curr_Bal where Active])`
4. **Budget Actual Spending** ← `SUMIFS(Transaction_DB[Amount], Month_ID, Category)`
5. **Income Actual** ← `SUMIFS(Transaction_DB[Amount], Month_ID, Type="Income")`
6. **Dashboard KPIs** ← All calculated from Transaction_DB, Debt_DB, Net_Worth_Snapshots_DB

### 6.5 Performance Considerations

**Index Strategy:**
- **Transaction_DB.Month_ID** → CRITICAL: Most-queried column (every Dashboard chart)
- **Income_Plan_History (Source_ID, Start_Date)** → CRITICAL: LOOKUP performance
- **Debt_Payments_DB.Debt_ID** → For Curr_Bal calculation
- **Net_Worth_Snapshots_DB.Month_ID** → For trend chart queries

**Query Optimization:**
- Use Month_ID filtering instead of Date range (faster index lookup)
- FILTER function preferred over SUMIFS for multiple conditions (array formula optimization)
- Named ranges cached for fast reference
- Avoid volatile functions (NOW, RAND) in calculated columns (use only in triggers)

**Bottleneck Identification:**
- Dashboard initial load: Queries all DBs simultaneously
- Period selector change: Recalculates all charts (target <1s)
- Save button: Appends 1 row (minimal impact)
- CSV Import: Bulk append (N rows, test with 100+)

### 6.6 Data Integrity Rules

**Referential Integrity (manual enforcement via formulas):**
1. Transaction_DB.Category → Must exist in Categories_DB (VLOOKUP validation)
2. Debt_Payments_DB.Debt_ID → Must exist in Debt_DB (VLOOKUP validation)
3. Goals_DB.CategoryLinked → Auto-created in Categories_DB on goal save
4. Income_Sources_DB.Category_Created → Auto-created in Categories_DB on source activate

**Business Rules:**
1. Categories_DB: Max 30 active (validation on add)
2. Goals_DB: Max 10 active (validation on add)
3. Income_Sources_DB: Max 8 sources (pre-allocated, no add beyond 8)
4. Budget Custom %: Must sum to exactly 100% (validation before save)
5. Debt_DB.Curr_Bal: Cannot go negative (payment cannot exceed balance)

**Soft Delete Pattern:**
- Transaction_DB: Deleted? = TRUE (preserves for audit trail)
- Categories_DB: Active = FALSE (preserves for historical transactions)
- Debt_DB: Status = "Archived" (preserves for payment history)
- Goals_DB: Status = "Complete" (preserves for historical tracking)

### 6.7 Multi-Year Support

**Design Principle:** Month_ID enables unlimited year tracking

**Implementation:**
- All tables with dates include auto-populated Month_ID column
- Format: "YYYY-MM" (TEXT) for reliable sorting and filtering
- Year selector dropdowns dynamically generate from `UNIQUE(TEXT(Date,"YYYY"))`
- Income_Plan_History carries forward indefinitely (no end date)
- Dashboard charts adapt X-axis scale based on data range

**Migration Path:**
- Jan 2025 → Dec 2025: Year 1 operations
- Jan 2026+: Seamless continuation (Month_ID handles year boundary)
- Promotion in Dec 2025: Carries to Jan 2026+ automatically
- Historical data preserved for all years (no archival needed unless performance issue)

---

**End of Document** | Last Update: 2025-10-14
