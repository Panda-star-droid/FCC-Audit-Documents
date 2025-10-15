# Phase 5: Budget System
**Deliverables:** Budget_Allocations_DB, Goals_DB, Budget sheet  
**Estimated Time:** 4 hours  
**Dependencies:** Phases 1-4 (Foundation, Transactions, Utilities/Settings, Income)
**Critical:** Must be built as Phase 5 - requires Income baseline from Phase 4

---

## 🎯 Deliverables

1. **Budget_Allocations_DB** - Budget method and per-category allocations
2. **Goals_DB** - Savings goals with targets and deadlines
3. **Budget** - User-facing sheet with budget planning, goals, spending analysis

---

## 🔗 Dependencies

**Required from Phase 1 (Foundation):**
- `Categories_DB[Name]`, `Categories_DB[Type]` - For budget categories
- `Settings_DB!Budget_Method` - For budget method selection

**Required from Phase 2 (Transactions):**
- `Transaction_DB` - Calculate actual spending per category

**Required from Phase 3 (Utilities/Settings):**
- Settings sheet for category management
- Settings_DB for storing Budget_Method and Monthly_Income (UI is on Budget sheet, not Settings)

**Required from Phase 4 (Income):**
- `Settings_DB!Monthly_Income` or Income_Sources_DB - Income baseline for budget calculations (CRITICAL)
- Income tracking mode affects budget planning

**Provides to later phases (6-8):**
- Budget vs Actual data for Dashboard comparison charts
- Goal progress for Dashboard widget
- Budget allocations for spending analysis

---

## 📊 Database Schemas

### Budget_Allocations_DB (Hidden)

**Columns:** A-H (8 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Text | "ba001" | Unique |
| B | Month_ID | TEXT | "2025-10" | YYYY-MM format for filtering |
| C | Method | Text | "50/30/20 Auto" | 50/30/20/Zero-Based/Custom |
| D | Category | Text | "Housing" | From Categories_DB |
| E | Percentage | Number | 35.0 | % of income (Custom method) |
| F | Allocated_Amt | Number | 1400.00 | Dollar amount |
| G | Effective_Date | Date | 2025-01-01 | When allocation starts |
| H | Status | Text | "Active" | Active/Archived |

**Sample Data (12 categories, 50/30/20 method):**
```
Housing: 35%, $1400 (Needs)
Groceries: 10%, $400 (Needs)
Transport: 5%, $200 (Needs)
Dining Out: 10%, $400 (Wants)
Entertainment: 8%, $320 (Wants)
Shopping: 8%, $320 (Wants)
Personal: 4%, $160 (Wants)
Savings: 20%, $800 (Savings)
```

**Month_ID Formula:**
```excel
B2: =TEXT(G2,"YYYY-MM")  /* Auto-generate from Effective_Date */
```

**Named Ranges:**
```
Budget_Allocations_DB[Month_ID] = $B$2:$B$100
Budget_Allocations_DB[Category] = $D$2:$D$100
Budget_Allocations_DB[Allocated_Amt] = $F$2:$F$100
Active_Allocations = FILTER(..., Status="Active")
```

**Indexes:**
- Index on: Month_ID (for fast period filtering)

---

### Goals_DB (Hidden)

**Columns:** A-J (10 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Text | "goal001" | Unique |
| B | GoalName | Text | "Emergency Fund" | Max 50 chars |
| C | TargetAmt | Number | 10000.00 | Target amount |
| D | CurrentAmt | Number | 6500.00 | Calculated from transactions |
| E | Deadline | Date | 2025-12-31 | Goal deadline |
| F | Created | Date | 2025-01-01 | When goal created |
| G | Status | Text | "Active" | Active/Complete/Paused |
| H | CategoryLinked | Text | "Goal: Emergency Fund" | Auto-created category |
| I | Priority | Number | 1 | 1=High, 5=Medium, 10=Low |
| J | Notes | Text | "6 months expenses" | Optional |

**Sample Data (2 goals):**
```
1. Emergency Fund: $10,000 target, $6,500 current, 12/31/2025
2. Vacation: $3,000 target, $1,200 current, 06/01/2026
```

**Named Ranges:**
```
Goals_DB[GoalName] = $B$2:$B$20
Goals_DB[TargetAmt] = $C$2:$C$20
Active_Goals = FILTER(..., Status="Active")
```

---

## 🎨 Budget Sheet Layout

**Freeze:** Row 4, Column A

**Column Widths:**
```
A: 200px (Category) | B-H: 110px each (data columns) | I: 120px (Status)
```

### Row 1-4: Period Controls + Overview

**A1 (Title):**
```
"BUDGET" - 18pt Bold Navy, Lt Blue bg, merge A1:P1
```

**B2-H2: Budget Configuration**
```
B2: Period dropdown (Week/Month/Quarter/Year) - links to Settings_DB!Period_Selected

D2-E2: Monthly Income Input
    D2 Label: "Monthly Income:" (Bold)
    E2 Input: $[____] (editable, saves to Settings_DB!Monthly_Income)

F2-H2: Budget Method Selection
    F2 Label: "Budget Method:" (Bold, 12pt)
    G2 Dropdown: [50/30/20 Auto ▼]
        Width: 180px
        Options:
          - 50/30/20 Auto (Needs 50% / Wants 30% / Savings 20%)
          - Zero-Based (Allocate every dollar manually)
          - Custom % (Set your own category percentages)
        Saves to: Settings_DB!Budget_Method
    
    H2: [💾 Save] button
        36px height, Navy bg, White text
        Saves both Monthly Income and Budget Method to Settings_DB
        On click: Updates Settings_DB, shows "✓ Saved" for 2 seconds

DESIGN NOTE: Budget Method and Monthly Income are configured HERE on the Budget sheet.
             This "configure where you use it" approach provides better workflow.
             Settings sheet stores these values in Settings_DB but has no UI for them.
```

**A3-D3: Overview Cards (4-up)**
```
Card 1: Monthly Income  | $4,000
Card 2: Total Budgeted  | $4,000 | Status: ✅ Balanced
Card 3: Total Spent     | $3,120 | 78% of budget
Card 4: Remaining       | $880   | 22% available
```

### Row 6-30: Budget Detail Table

**Headers (Row 6):**
```
A6: Category | B6: Type | C6: Budgeted | D6: Actual | E6: Remaining | F6: % Used | G6: Status | H6: Trend
```

**Visual Specs:**
- Headers: Navy bg, White Bold 12pt, Center-aligned, 32px height
- Data rows: 24px height, alternating White/Lt Blue
- Numbers: Right-aligned, $#,##0.00 format
- Status: Center-aligned pills (Green/Orange/Red)

**Conditional Formatting (Remaining column E):**
```
< $0: Red bold
$0-20% of budget: Orange
> 20%: Green
```

**Conditional Formatting (% Used column F):**
```
Color scale: Green (0%) → Yellow (80%) → Red (100%+)
```

**Status Pills (Column G):**
```
Formula: =IF(E7<0, "⚠️ Over", IF(F7>0.9, "⚠️ Near Limit", "✅ On Track"))
```

### Row 28-50: Charts

**Chart 1: Budget vs Actual (A28:K50)**
```
Type: Column chart
X-axis: Categories (top 10 by spending)
Y-axis: Dollars ($0-$2000)
Series 1: Budgeted (Navy bars, 60% width)
Series 2: Actual (Green if under, Red if over, 60% width)
Gap: 20%
Legend: Top center
```

**Chart 2: Spending Mix (M28:V50)**
```
Type: Donut chart
Data: Actual spending by category (Top 8 + "Other")
Colors: Category palette (15 colors)
Inner radius: 40%
Center text: "Total: $3,120" (18pt Bold Navy)
Legend: Right side
```

### Row 52-60: Overspending Alerts

**A52 Title:**
```
"⚠️ TOP 3 OVER-BUDGET" - 14pt Bold Orange
```

**A53-A55: Alert Cards**
```
1. Dining Out: $320/$255 (+25% over) 💡 Try meal prepping Sundays
2. Entertainment: $180/$170 (+6% over) 💡 Free community events
3. Transport: $215/$213 (+1% over) ✅ Nearly on track
```

**Formula (Top 3 categories where Actual > Budgeted, sorted by % over):**
```excel
=FILTER(Categories, Actual > Budgeted, 
        SORT by (Actual-Budgeted)/Budgeted DESC, 
        LIMIT 3)
```

### Row 62-120: Goals Section

**A62 Title:**
```
"🎯 SAVINGS GOALS" - 14pt Bold Navy
```

**A64: [➕ Add New Goal] Button**
```
Hyperlink that expands form below (or navigates to form)
Format: Navy bg, White text, 11pt Bold, 44px height
```

**A66-A100: Active Goals (3 visible, expandable)**

**Goal Card Template:**
```
Row 1: 🎯 Goal Name                                    [Edit] [Delete]
Row 2: Target: $10,000 | Saved: $6,500 | Deadline: 12/31/2025 | Progress: 65%
Row 3: ████████████░░░░░░░░ 65%  (Progress bar - 20 chars wide)
Row 4: 💰 Save $583/month to reach goal
Row 5: [Log $____ Contribution] (44px save button)
Row 6: [Blank spacing]
```

**Add Goal Form (A102-A115, conditional visibility):**
```
Visible when [Add New Goal] clicked

Goal Name:   [____________________] (30 char max)
Target:      $[______]
Deadline:    [MM/DD/YYYY] (must be future date)
Priority:    [High â–¼] (High/Medium/Low)

[💾 Save Goal] [Cancel]
```

**Log Contribution Button Formula:**
```excel
Creates Income transaction with Category="Goal: Emergency Fund"
Amount: [User input]
Date: TODAY()
```

### Row 122-150: Custom Budget Allocation (if Method="Custom")

**Visible only when Settings_DB!Budget_Method = "Custom %"**

**A122 Title:**
```
"CUSTOM BUDGET ALLOCATION" - 14pt Bold Navy
```

**A124-A145: Allocation Table**
```
Headers: Category | Allocation % | Budgeted Amount | Status

Row formulas:
Budgeted Amount = (Allocation% / 100) * Monthly_Income
Total row: SUM(Allocation%) must = 100%
```

**A146: Validation Status**
```
Formula: =IF(SUM(Allocation_Range)=100, "✅ Valid - Ready to save", "⚠️ Total must equal 100% (currently "&SUM(Allocation_Range)&"%)")

Conditional Format:
- If =100%: Green text
- If ≠100%: Red text
```

**A148: [💾 Save Custom Allocations] Button**
```
44px height, Navy bg when valid (total=100%), Lt Gray when invalid
```

---

## 🔧 Key Formulas

### Budget Calculation (50/30/20 Method)

**Per-Category Budgeted Amount:**
```excel
=IF(Category_Type="Needs", 
   Monthly_Income * 0.50 / COUNTIF(Categories_DB[Type], "Needs"),
   IF(Category_Type="Wants",
      Monthly_Income * 0.30 / COUNTIF(Categories_DB[Type], "Wants"),
      Monthly_Income * 0.20 / COUNTIF(Categories_DB[Type], "Savings")))
```

### Actual Spending

**Using Month_ID for filtering (preferred method for monthly periods):**
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Category], Category_Name,
        Transaction_DB[Type], "Expense",
        Transaction_DB[Month_ID], Current_Month_ID,
        Transaction_DB[Deleted?], FALSE) * -1
```

**Using Date ranges (for Week/Quarter/Year periods):**
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Category], Category_Name,
        Transaction_DB[Type], "Expense",
        Transaction_DB[Date], ">="&Period_Start,
        Transaction_DB[Date], "<="&Period_End,
        Transaction_DB[Deleted?], FALSE) * -1
```

**Note:** Month_ID filtering is faster and recommended for monthly budget tracking

### Remaining & % Used

```excel
Remaining = Budgeted - Actual
Percent_Used = IF(Budgeted>0, Actual/Budgeted, 0)
```

### Goal Current Amount (Real-Time)

```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Category], "Goal: "&Goal_Name,
        Transaction_DB[Deleted?], FALSE)
```

### Goal Progress Percentage

```excel
=MIN(100, (Current_Amount / Target_Amount) * 100)
```

### Monthly Savings Needed

```excel
=MAX(0, (Target_Amount - Current_Amount) / 
        MAX(1, DATEDIF(TODAY(), Deadline, "M")))
```

### Progress Bar (Text)

```excel
=REPT("█", INT((Current_Amount / Target_Amount) * 20)) &
 REPT("░", 20 - INT((Current_Amount / Target_Amount) * 20)) &
 " " & TEXT(MIN(100, (Current_Amount/Target_Amount)*100), "0") & "%"
```

### Custom Budget Validation

**Save Button Display:**
```excel
=IF(SUM(Custom_Allocation_Range)=100,
   "💾 Save Custom Allocations",
   "Total must = 100% (currently " & SUM(Custom_Allocation_Range) & "%)")
```

**Save Trigger (when clicked):**
```excel
Updates Budget_Allocations_DB with:
- Method = "Custom %"
- Category, Percentage, Allocated_Amt for each row
- Effective_Date = TODAY()
```

---

## ✅ Acceptance Tests (35 tests)

### Budget Calculation Tests (10)

1. **50/30/20 Auto-Calculation**
   - Set Monthly_Income = $4000, Method = "50/30/20 Auto"
   - ✅ Needs total = $2000 (50%)
   - ✅ Wants total = $1200 (30%)
   - ✅ Savings total = $800 (20%)
   - ✅ Total budgeted = $4000

2. **Per-Category Allocation (50/30/20)**
   - Housing (Needs): ✅ $2000 / 4 Needs categories ≈ $500
   - Dining (Wants): ✅ $1200 / 4 Wants categories = $300

3. **Actual Spending Calculation**
   - Add transactions: Dining Out $50, $75, $100
   - ✅ Budget table shows Dining Out Actual = $225

3a. **Month_ID Filtering**
   - Add transactions in Oct 2025 and Nov 2025
   - Set period to "2025-10"
   - ✅ Budget shows only Oct transactions
   - ✅ Nov transactions excluded
   - ✅ Formula uses Transaction_DB[Month_ID] filtering

4. **Remaining Calculation**
   - Budgeted $300, Actual $225
   - ✅ Remaining = $75
   - ✅ Green color (>20%)

5. **% Used Calculation**
   - ✅ % Used = 225/300 = 75%
   - ✅ Color scale shows yellow-green

6. **Status Indicator**
   - Remaining $75 (>20%): ✅ "✅ On Track"
   - Remaining -$20 (<0): ✅ "⚠️ Over"
   - % Used 95%: ✅ "⚠️ Near Limit"

7. **Zero-Based Method**
   - Switch to Zero-Based
   - ✅ Budgeted amounts become editable (manual entry)
   - ✅ Validation: SUM(all categories) = Monthly_Income

8. **Custom % Method**
   - Switch to Custom %
   - ✅ Custom allocation table appears (row 122+)
   - ✅ Can edit percentages per category
   - ✅ Validation enforces total = 100%

9. **Custom % Save**
   - Set allocations: Housing 35%, Groceries 10%, etc.
   - Total = 100%
   - Click [Save Custom Allocations]
   - ✅ Budget_Allocations_DB updates
   - ✅ Method = "Custom %"
   - ✅ Budgeted amounts reflect custom %

10. **Custom % Validation Error**
    - Set allocations totaling 95%
    - ✅ Save button shows error "Total must = 100% (currently 95%)"
    - ✅ Button disabled (Lt Gray bg)

### Budget Display Tests (8)

11. **Budget Detail Table**
    - ✅ All active categories listed
    - ✅ Columns: Category, Type, Budgeted, Actual, Remaining, % Used, Status
    - ✅ Numbers right-aligned, formatted $#,##0.00

12. **Overspending Alert**
    - Create over-budget scenario: Dining Out $320 vs $255 budget
    - ✅ "TOP 3 OVER-BUDGET" section displays
    - ✅ Dining Out listed first
    - ✅ Shows "+25% over"
    - ✅ Displays helpful tip

13. **Period Filter**
    - Change Period to "Week"
    - ✅ Actual spending recalculates for last 7 days only
    - ✅ Charts update
    - ✅ Overspending alerts update

14. **Budget vs Actual Chart**
    - ✅ Chart displays top 10 categories by spending
    - ✅ Navy bars (Budgeted)
    - ✅ Green bars (Actual, if under budget)
    - ✅ Red bars (Actual, if over budget)

15. **Spending Mix Donut**
    - ✅ Shows actual spending breakdown
    - ✅ Top 8 categories + "Other"
    - ✅ Center text shows "Total: $X,XXX"
    - ✅ Legend on right

16. **Overview Cards**
    - ✅ Monthly Income displays correctly
    - ✅ Total Budgeted = sum of all allocations
    - ✅ Total Spent = sum of actual expenses
    - ✅ Remaining = Budgeted - Spent
    - ✅ Status shows ✅ if balanced

17. **Empty Budget (No Spending)**
    - Reset transactions (all deleted)
    - ✅ Actual column shows $0
    - ✅ Remaining = Budgeted
    - ✅ % Used = 0%
    - ✅ No errors

18. **Over-Budget Indicators**
    - ✅ Remaining column: Red text for negative values
    - ✅ % Used: Red background for >100%
    - ✅ Status: "⚠️ Over" label

### Goals Tests (17)

19. **Load Sample Goals**
    - ✅ 2 sample goals present (Emergency Fund, Vacation)
    - ✅ Target amounts set
    - ✅ Current amounts calculated from transactions
    - ✅ Deadlines in future

20. **Goal Display**
    - ✅ Goal name, target, current, deadline, progress % visible
    - ✅ Progress bar displays (20 chars, █ filled, ░ remaining)
    - ✅ Monthly savings needed calculated correctly

21. **Goal Current Amount Calculation**
    - Add transaction: Category="Goal: Emergency Fund", Amount=$250
    - ✅ Emergency Fund current amount increases by $250
    - ✅ Progress % updates
    - ✅ Monthly savings needed recalculates

22. **Add New Goal**
    - Click [➕ Add New Goal]
    - ✅ Form expands/displays
    - ✅ Fields: Name, Target, Deadline, Priority

23. **Goal Validation**
    - Try to create goal with past deadline
    - ✅ Error: "⚠️ Deadline is in the past. Choose future date."
    - Try zero target
    - ✅ Error: "⚠️ Target must be greater than $0"

24. **Save New Goal**
    - Fill form: "Car Down Payment", $5000 target, 12/31/2026 deadline
    - Click [Save Goal]
    - ✅ New row appears in Goals_DB
    - ✅ Status = "Active"
    - ✅ Auto-creates category "Goal: Car Down Payment"
    - ✅ Goal card appears in Goals section

25. **Log Contribution**
    - Enter $100 in [Log Contribution] field for Emergency Fund
    - Click/press ENTER
    - ✅ Creates Income transaction
    - ✅ Category = "Goal: Emergency Fund"
    - ✅ Amount = $100
    - ✅ Date = TODAY()
    - ✅ Goal current amount increases by $100

26. **Contribution Form (Save Button)**
    - Contribution field empty
    - ✅ Button shows "Enter amount"
    - ✅ Lt Gray background (invalid)
    - Enter $250
    - ✅ Button shows "💾 Log Contribution"
    - ✅ Navy background (valid)

27. **Goal Progress 100%+**
    - Add contributions exceeding target ($10,500 for $10,000 goal)
    - ✅ Progress % caps at 100%
    - ✅ Status shows "✅ Goal Reached!"
    - ✅ Monthly savings needed = $0

28. **Goal Completion**
    - Reach 100% of goal
    - ✅ Option to mark as "Complete"
    - Mark complete
    - ✅ Status changes to "Complete"
    - ✅ Goal moves to "Completed Goals" section (or archives)

29. **Edit Goal**
    - Click [Edit] on existing goal
    - ✅ Fields become editable
    - Change target from $10,000 to $12,000
    - Save
    - ✅ Goals_DB updates
    - ✅ Progress % recalculates
    - ✅ Monthly savings needed recalculates

30. **Delete Goal**
    - Click [Delete] on goal
    - ✅ Confirmation prompt: "Delete 'Vacation'? Goal category will be archived. [Cancel] [Delete]"
    - Confirm
    - ✅ Goal removed from Goals section
    - ✅ Goals_DB Status = "Archived"
    - ✅ Category "Goal: Vacation" archived in Categories_DB

31. **Max Goals Limit**
    - Create 10 goals
    - Try to create 11th goal
    - ✅ Error: "⚠️ Maximum 10 goals. Complete or delete existing goals."

32. **Goal Priority Sorting**
    - Set goals with different priorities (High, Medium, Low)
    - ✅ Goals display in priority order (High first)

33. **Multiple Contributions (Same Day)**
    - Log $100 contribution to Emergency Fund
    - Log $50 contribution to Emergency Fund (same day)
    - ✅ Both transactions created
    - ✅ Current amount increases by $150 total

34. **Goal Without Deadline**
    - ✅ System blocks saving (deadline required)
    - OR: Allows saving with no deadline, monthly savings = "At your pace"

35. **Goal Category Auto-Link**
    - Create goal "Vacation"
    - ✅ Category "Goal: Vacation" auto-created in Categories_DB
    - ✅ Type = "Savings"
    - ✅ Appears in category dropdowns (Transaction, Quick Entry)

---

## 🚧 Known Issues / Blockers

*Document during build.*

---

## 📝 Implementation Checklist

### Step 1: Create Database Sheets
- [ ] Budget_Allocations_DB (hidden, 8 columns - includes Month_ID)
- [ ] Add Month_ID formula: =TEXT(G2,"YYYY-MM") in column B
- [ ] Add index on Month_ID column
- [ ] Goals_DB (hidden, 10 columns)
- [ ] Load sample data (12 budget allocations, 2 goals)

### Step 2: Create Budget Sheet
- [ ] Title and controls (rows 1-4)
- [ ] Overview cards
- [ ] Budget detail table (rows 6-30)
- [ ] Charts (rows 28-50)
- [ ] Overspending alerts (rows 52-60)
- [ ] Goals section (rows 62-120)
- [ ] Custom allocation table (rows 122-150, conditional)

### Step 3: Add Formulas
- [ ] Budget calculations (50/30/20, Zero-Based, Custom)
- [ ] Actual spending (SUMIFS from Transaction_DB using Month_ID)
- [ ] Remaining and % Used
- [ ] Status indicators
- [ ] Goal current amount
- [ ] Goal progress calculations
- [ ] Custom allocation validation
- [ ] Month_ID auto-population formula

### Step 4: Add Save Buttons
- [ ] Save Custom Allocations button (row 148)
- [ ] Add Goal button + form
- [ ] Log Contribution buttons (per goal)

### Step 5: Charts
- [ ] Budget vs Actual (column chart)
- [ ] Spending Mix (donut chart)
- [ ] Format axes, legends, colors

### Step 6: Testing
- [ ] Run all 35 acceptance tests
- [ ] Verify all budget methods work
- [ ] Verify goals system fully functional
- [ ] Check charts update correctly

---

## 🎯 Success Criteria

Phase 3 complete when all 35 tests pass and:
- Budget calculations accurate for all methods
- Charts display correctly
- Goals system fully functional
- No errors in any formula

---

## 📞 Next Phase

**Phase 4: Debt System**  
Depends on: Transaction_DB (for payment logging)  
Estimated: 3 hours
