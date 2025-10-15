# Finance Control Center - Formula Patterns
**Version:** 2.1 No-VBA | **Universal Templates**

---

## 📝 Overview

This document contains formula templates used across **multiple phases**. Each pattern includes:
- Purpose
- Implementation steps
- Formula code
- Helper cells needed
- Visual states (conditional formatting)

**Note:** Phase-specific formulas are documented inline in each phase document.

---

## 1️⃣ Save Cell Pattern (ALL Forms)

**Used in:** Quick Entry, Budget, Debt, Net Worth, Goals, CSV Import

### Purpose
Replace VBA buttons with formula-driven save cells that:
- Change color when form is valid
- Trigger append when clicked/edited
- Clear form after save
- Show status message

### Implementation Steps

#### Step 1: Display Formula (Save Cell)
```excel
=IF(AND(Required_Field_1<>"", 
        Required_Field_2<>"", 
        [Additional_Validation]),
   "💾 Tap to Save",
   "Fill required fields...")
```

**Customize per form:**
- Replace `Required_Field_1`, `Required_Field_2` with actual cell references
- Add validation (e.g., `Amount>0`, `ISNUMBER(Amount)`)
- Adjust error message ("Enter amount", "Select category", etc.)

#### Step 2: Conditional Formatting Rules (Save Cell)

**Rule 1: Valid State**
```
Applies to:  [Save_Cell_Address]
Formula:     =AND(Required_Field_1<>"", Required_Field_2<>"", [Validation])
Format:
  - Fill: #1A2B4A (Navy)
  - Font: White, Bold, 11pt
  - Border: 2px solid Navy
```

**Rule 2: Invalid State**
```
Applies to:  [Save_Cell_Address]
Formula:     =NOT(AND(Required_Field_1<>"", Required_Field_2<>"", [Validation]))
Format:
  - Fill: #F5F5F5 (Lt Gray)
  - Font: #757575 (Md Gray), Regular, 11pt
  - Border: 1px solid Lt Gray
```

**Rule 3: Focus State**
```
Applies to:  [Save_Cell_Address]
Trigger:     Cell selected (automatic)
Format:
  - Outline: 2px solid Gold #D4AF37, 2px offset
```

#### Step 3: Trigger Detection (Hidden Helper Cells)

**Cell Z1: Edit Counter**
```excel
=IF([Save_Cell]<>"", Z1+1, Z1)
```

**Cell Z2: Previous Counter**
```excel
=Z1
```
(Updates after each calculation cycle)

**Cell Z3: Trigger Detected**
```excel
=Z1<>Z2
```

#### Step 4: Append Formula (Target DB Row)

**Example: Transaction_DB!A2**
```excel
=IF(Dashboard!Z3,
   IF(Dashboard!QuickAdd_Amount<>"",
      {Dashboard!QuickAdd_Date,
       "Expense",
       Dashboard!QuickAdd_Description,
       Dashboard!QuickAdd_Category,
       -1 * Dashboard!QuickAdd_Amount,
       Dashboard!QuickAdd_Account,
       "",
       "",
       NOW(),
       FALSE,
       TEXT(RAND()*1E15,"0")},
      ""),
   "")
```

**Customize:**
- Replace `Dashboard!` with actual sheet name
- Map form fields to DB columns
- Handle type-specific logic (Income vs Expense, sign conventions)

#### Step 5: Auto-Clear Formula (Form Fields)

```excel
=IF(NOW() - [LastSave_Timestamp] > TIME(0,0,2), 
   "", 
   [Current_Value])
```

**Where:**
- `LastSave_Timestamp` = Hidden cell storing `NOW()` when save triggered
- `Current_Value` = Previous field value (circular reference safe)

**Alternative (simpler):**
```excel
=IF([Trigger_Detected], "", [Manual_Entry])
```
(Clears immediately on save, user re-enters for next transaction)

#### Step 6: Status Cell Formula

```excel
=IF([LastSave_Timestamp]<>"",
   "✅ Saved to " & TEXT([Date_Field],"MMM YYYY") & " - " & [Category_Field],
   IF(AND([Required_Fields_Filled]), "Ready", ""))
```

**Conditional Formatting (Status Cell):**
```
Success:
  Formula: =[Status_Cell]<>""
  Format:  Font=#2E7D32 (Green), 11pt Regular

Auto-clear after 3 seconds:
  Formula: =IF(NOW()-[LastSave_Timestamp]>TIME(0,0,3), "", [Status_Text])
```

### Example: Quick Entry Transaction Form

**Layout:**
- C9: Date (input)
- F9: Type dropdown (Income/Expense)
- I9: Description (input)
- L9: Category dropdown
- O9: Amount (input)
- A12: Save Cell
- A13: Status Cell

**Formulas:**

**A12 (Save Cell):**
```excel
=IF(AND(C9<>"", L9<>"", O9<>"", ISNUMBER(O9), O9>0),
   "💾 Tap to Save",
   IF(C9="", "Enter date",
      IF(L9="", "Select category",
         IF(OR(O9="", NOT(ISNUMBER(O9)), O9<=0), "Enter valid amount", ""))))
```

**Z1 (Edit Counter):**
```excel
=IF(A12<>"", Z1+1, Z1)
```

**Z3 (Trigger):**
```excel
=Z1<>Z2
```

**Transaction_DB!A2 (Append):**
```excel
=IF('Quick Entry'!Z3,
   IF('Quick Entry'!O9<>"",
      {'Quick Entry'!C9,
       'Quick Entry'!F9,
       'Quick Entry'!I9,
       'Quick Entry'!L9,
       'Quick Entry'!O9,
       "",
       "",
       "",
       NOW(),
       FALSE,
       TEXT(RAND()*1E15,"0")},
      ""),
   "")
```

**A13 (Status):**
```excel
=IF(NOW()-Z4<TIME(0,0,3),
   "✅ Saved to " & TEXT(C9,"MMM YYYY") & " - " & L9,
   IF(AND(C9<>"", L9<>"", O9>0), "Ready", ""))
```

**C9-O9 (Auto-Clear):**
```excel
=IF(Z3, "", [User_Input])
```

### Row Height & Protection
- Set Save Cell row height to **44px** (WCAG touch target)
- Protect Save Cell (requires edit to trigger)
- Leave form fields unprotected

---

## 2️⃣ Status Cell Pattern

**Used in:** All forms for success/error feedback

### Success Status
```excel
=IF([LastSave_Timestamp]<>"",
   "✅ " & [Success_Message],
   "")

Where [Success_Message] might be:
- "Saved to Oct 2025 - Dining Out"
- "Payment recorded! Card A: $7,500 → $7,400"
- "Goal created: Emergency Fund"
```

**Conditional Formatting:**
```
Font: #2E7D32 (Green), 11pt Regular
Background: #E8F5E9 (Lt Green) - optional
```

**Auto-Clear:**
```excel
=IF(NOW() - [LastSave_Timestamp] > TIME(0,0,3), 
   "", 
   [Success_Message])
```

### Error Status
```excel
=IF([Error_Condition],
   "✕ " & [Error_Message],
   "")

Where [Error_Condition] examples:
- Amount="" : "Amount is required"
- NOT(ISNUMBER(Amount)) : "Enter valid number (e.g., 45.50)"
- Date>TODAY() : "Date in future. Proceed? [Yes] [Fix]"
- Income<0 : "Income is negative. Did you mean Expense?"
```

**Conditional Formatting:**
```
Font: #C62828 (Red), 11pt Regular
Background: #FFEBEE (Lt Red) - optional
```

**Persistent:** Error remains until condition fixed (no auto-clear)

### Info Status
```excel
=IF(AND([Form_Fields_Filled], NOT([Save_Triggered])),
   "Ready",
   "")
```

**Conditional Formatting:**
```
Font: #757575 (Md Gray), 10pt Regular
Background: None
```

---

## 3️⃣ Auto-Clear Pattern

**Purpose:** Reset form fields after successful save

### Method 1: Time-Based (2-3 second delay)
```excel
=IF(NOW() - [LastSave_Timestamp] > TIME(0,0,2), 
   "", 
   [Field_Value])
```

**Pros:** Smooth UX (user sees confirmation before clear)  
**Cons:** Requires circular reference handling

### Method 2: Immediate Clear
```excel
=IF([Trigger_Detected], "", [User_Entry])
```

**Pros:** Simpler, no circular refs  
**Cons:** Instant clear (no visual confirmation)

### Recommended: Method 1 + Status Cell
- Form clears after 2 seconds
- Status cell shows success for 3 seconds
- User sees confirmation before fields clear

---

## 4️⃣ Duplicate Detection Pattern

**Used in:** CSV Import, Transaction validation

### Formula (CSV Import Preview)
```excel
=IF(COUNTIFS(Transaction_DB[Date], [Normalized_Date],
             Transaction_DB[Amount], [Normalized_Amount],
             Transaction_DB[Description], "*"&LEFT([Description],20)&"*") > 0,
   "Duplicate",
   "New")
```

**Explanation:**
- Matches on: Date + Amount + First 20 chars of description
- Wildcard match handles slight description variations
- Returns "Duplicate" or "New"

### Status Column Formula
```excel
=IF([Duplicate_Flag]="Duplicate", 
   "⚠️ Possible duplicate",
   IF([Category]="Uncategorized", 
      "⚠️ Needs category",
      "✅ Ready"))
```

### Conditional Formatting (Preview Table)
```
Duplicate rows:
  Formula: =[Status_Cell]="⚠️ Possible duplicate"
  Format:  Background=#FFF3E0 (Lt Orange)

Ready rows:
  Formula: =[Status_Cell]="✅ Ready"
  Format:  Background=#E8F5E9 (Lt Green)
```

---

## 5️⃣ Period Filter Pattern

**Used in:** Dashboard, Budget, charts (all period-aware views)

### Period Selector Dropdown (Settings_DB)
```
Values: Week | Month | Quarter | Year
Named Range: Period_Selected = Settings_DB!B2
```

### Start Date Calculation
```excel
=CHOOSE(MATCH(Period_Selected, {"Week","Month","Quarter","Year"}, 0),
   TODAY() - WEEKDAY(TODAY(), Settings_DB!Week_Start_Offset),  /* Week */
   DATE(YEAR(TODAY()), MONTH(TODAY()), 1),                      /* Month */
   DATE(YEAR(TODAY()), MONTH(TODAY())-2, 1),                    /* Quarter */
   DATE(YEAR(TODAY()), 1, 1))                                   /* Year */
```

### End Date Calculation
```excel
=CHOOSE(MATCH(Period_Selected, {"Week","Month","Quarter","Year"}, 0),
   TODAY(),                                                     /* Week */
   EOMONTH(TODAY(), 0),                                        /* Month */
   EOMONTH(TODAY(), 0),                                        /* Quarter */
   DATE(YEAR(TODAY()), 12, 31))                                /* Year */
```

### Usage in Calculations
```excel
Income = SUMIFS(Transaction_DB[Amount],
                Transaction_DB[Type], "Income",
                Transaction_DB[Date], ">="&Period_Start,
                Transaction_DB[Date], "<="&Period_End,
                Transaction_DB[Deleted?], FALSE)
```

### Week Start Offset
```
Sunday = 1 (US, Canada, Middle East)
Monday = 2 (Europe, Australia)
Saturday = 7 (Some Middle East countries)

Stored in: Settings_DB!Week_Start_Offset
```

---

## 6️⃣ Budget Calculation Pattern

**Used in:** Budget sheet, Dashboard KPIs

### 50/30/20 Method
```excel
Needs_Budget = Monthly_Income * 0.50 / COUNT(Needs_Categories)
Wants_Budget = Monthly_Income * 0.30 / COUNT(Wants_Categories)
Savings_Budget = Monthly_Income * 0.20 / COUNT(Savings_Categories)
```

**Per-Category:**
```excel
=IF(Category_Type="Needs", 
   Monthly_Income * 0.50 / COUNTIF(Categories_DB[Type], "Needs"),
   IF(Category_Type="Wants",
      Monthly_Income * 0.30 / COUNTIF(Categories_DB[Type], "Wants"),
      Monthly_Income * 0.20 / COUNTIF(Categories_DB[Type], "Savings")))
```

### Zero-Based Method
```excel
Budgeted_Amount = [Manual_Entry_Per_Category]

Validation: SUM(All_Categories) must = Monthly_Income
```

### Custom % Method
```excel
Budgeted_Amount = (Allocation_Percentage / 100) * Monthly_Income

Validation: SUM(Allocation_Percentages) must = 100%
```

### Actual Spending
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Category], Category_Name,
        Transaction_DB[Type], "Expense",
        Transaction_DB[Date], ">="&Period_Start,
        Transaction_DB[Date], "<="&Period_End,
        Transaction_DB[Deleted?], FALSE) * -1
```
(Multiply by -1 because expenses are stored negative)

### Remaining & % Used
```excel
Remaining = Budgeted - Actual
Percent_Used = IF(Budgeted>0, Actual/Budgeted, 0)
```

### Status Indicator
```excel
=IF(Remaining < 0, 
   "⚠️ Over",
   IF(Percent_Used > 0.9, 
      "⚠️ Near Limit",
      "✅ On Track"))
```

---

## 7️⃣ Debt Scenario Pattern

**Used in:** Debt sheet scenario planner

### Months to Debt-Free (Simplified)
```excel
=ROUNDUP(Total_Balance / 
         (Total_Min_Payments + Extra_Payment - Monthly_Interest), 0)

Where:
Monthly_Interest = (Total_Balance * Weighted_APR) / 12
Weighted_APR = SUMPRODUCT(Debt_DB[Curr_Bal], Debt_DB[APR]) / SUM(Debt_DB[Curr_Bal])
```

### Months to Debt-Free (Accurate - NPER)
```excel
=NPER(Weighted_APR / 12,
      -(Total_Min_Payments + Extra_Payment),
      Total_Balance)
```

### Total Interest Paid
```excel
=(Months_To_DebtFree * (Total_Min_Payments + Extra_Payment)) - Total_Balance
```

### Interest Saved (vs Current Plan)
```excel
=Interest_Paid_Current - Interest_Paid_Strategy
```

### Payoff Order (Avalanche - Highest APR First)
```excel
=SORT(Debt_DB, MATCH("APR", Debt_DB_Headers, 0), -1)
```

### Payoff Order (Snowball - Smallest Balance First)
```excel
=SORT(Debt_DB, MATCH("Curr_Bal", Debt_DB_Headers, 0), 1)
```

---

## 8️⃣ Goal Progress Pattern

**Used in:** Budget sheet goals section, Dashboard widget

### Current Amount (Real-Time)
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Category], "Goal: " & Goal_Name,
        Transaction_DB[Deleted?], FALSE)
```

**Note:** Goal contributions are Income transactions with special category

### Progress Percentage
```excel
=MIN(100, (Current_Amount / Target_Amount) * 100)
```
(Cap at 100% even if over-funded)

### Progress Bar (Text-Based)
```excel
=REPT("█", INT((Current_Amount / Target_Amount) * 20)) &
 REPT("░", 20 - INT((Current_Amount / Target_Amount) * 20)) &
 " " & TEXT(MIN(100, (Current_Amount/Target_Amount)*100), "0") & "%"
```

**Result:** `████████████░░░░░░░░ 65%`

### Monthly Savings Needed
```excel
=MAX(0, (Target_Amount - Current_Amount) / 
        MAX(1, DATEDIF(TODAY(), Deadline, "M")))
```

**Handles edge cases:**
- Past deadline: Returns 0 (can't save backwards in time)
- Zero months remaining: Returns full remaining amount
- Already funded: Returns 0

### Status Indicator
```excel
=IF(Current_Amount >= Target_Amount,
   "✅ Goal Reached!",
   IF(Deadline < TODAY(),
      "⚠️ Past Deadline",
      IF(Monthly_Needed > Monthly_Income * 0.5,
         "⚠️ Very Aggressive",
         "📈 On Track")))
```

---

## 9️⃣ Net Worth Projection Pattern

**Used in:** Net Worth sheet projection section

### Future Value (With Contributions)
```excel
=Current_Net_Worth * (1 + Growth_Rate)^Years + 
 Monthly_Contribution * (((1 + Growth_Rate)^Years - 1) / Growth_Rate)

Where:
Growth_Rate = Annual_Return / 12 (convert to monthly)
Years = Projection_Period (in years)
```

**Simplified (Annual Contributions):**
```excel
=Current_Net_Worth * (1 + Annual_Growth_Rate)^Years +
 (Annual_Contribution * 12) * (((1 + Annual_Growth_Rate)^Years - 1) / Annual_Growth_Rate)
```

### Month-over-Month Change
```excel
=Current_Net_Worth - VLOOKUP(EOMONTH(TODAY(),-1),
                              Net_Worth_Snapshots_DB[[Date]:[Net_Worth]],
                              5, FALSE)
```

### Change Percentage
```excel
=IF(Previous_Month_NW<>0,
   (Current_Net_Worth - Previous_Month_NW) / ABS(Previous_Month_NW),
   0)
```

---

## 🔟 CSV Import Normalization Pattern

**Used in:** CSV Import sheet

### Date Normalization (Handles Multiple Formats)
```excel
=IFERROR(DATEVALUE(Raw_Date),
  IFERROR(DATE(RIGHT(Raw_Date,4), LEFT(Raw_Date,2), MID(Raw_Date,4,2)),
    IFERROR(DATE(LEFT(Raw_Date,4), MID(Raw_Date,6,2), MID(Raw_Date,9,2)),
      "ERROR: Invalid Date")))
```

**Handles:**
- MM/DD/YYYY
- DD/MM/YYYY (if DATEVALUE fails, tries this)
- YYYY-MM-DD (ISO format)

### Amount Normalization (Clean Currency)
```excel
=VALUE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(Raw_Amount, "$", ""), ",", ""), " ", ""))
```

**Removes:** $, commas, spaces

### Type Detection (Income vs Expense)
```excel
=IF(Normalized_Amount > 0, "Income", "Expense")
```

### Auto-Categorize (Keyword Matching)
```excel
=IFERROR(XLOOKUP(LOWER(Description),
                 Keywords_DB[Keyword],
                 Keywords_DB[Category],
                 "Uncategorized",
                 2),  /* Wildcard match */
         "Uncategorized")
```

**Fallback (Excel 2016):**
```excel
=IFERROR(INDEX(Keywords_DB[Category],
               MATCH("*"&LOWER(Description)&"*", 
                     Keywords_DB[Keyword], 0)),
         "Uncategorized")
```

### Status Summary
```excel
=CONCATENATE(
  COUNTIF(Status_Range, "✅ Ready"), " ready | ",
  COUNTIF(Status_Range, "⚠️ Duplicate"), " duplicates | ",
  COUNTIF(Status_Range, "⚠️ Needs Category"), " need categories"
)
```

---

## 🎯 Quick Reference Table

| Pattern | Primary Use | Key Formula |
|---------|-------------|-------------|
| Save Cell | All forms | `IF(Valid, "💾 Save", "Fill fields")` |
| Status Cell | Feedback | `IF(Saved, "✅ Success", "")` |
| Auto-Clear | Form reset | `IF(NOW()-Save>2s, "", Value)` |
| Duplicate | Import | `COUNTIFS(DB, Criteria) > 0` |
| Period Filter | Dashboard/Budget | `SUMIFS(..., Date>=Start, Date<=End)` |
| Budget 50/30/20 | Budget calc | `Income * 0.50 / COUNT(Needs)` |
| Debt Scenario | Debt planning | `NPER(APR/12, -Payment, Balance)` |
| Goal Progress | Goal tracking | `Current / Target * 100` |
| Net Worth Projection | Forecasting | `NW * (1+Rate)^Years + Contributions` |
| CSV Normalize | Import cleanup | `VALUE(SUBSTITUTE(Amount, "$", ""))` |
| Income Tracking | Income planning (V11) | `LOOKUP(2, 1/(History[Date]<=Month), History[Amount])` |

---

## 💡 Tips for Using These Patterns

1. **Copy-Paste Template** - Start with formula above, customize cell refs
2. **Test with Sample Data** - Verify formula before applying to full DB
3. **Name Your Ranges** - Replace cell refs with names for readability
4. **Error Handling** - Always wrap in `IFERROR()` for production
5. **Performance** - Use named ranges for large datasets (faster than INDIRECT)
6. **Documentation** - Add cell comments explaining complex formulas

---

**Next:** Apply these patterns in each phase document. Phase docs include phase-specific formulas inline.

---

## 1️⃣1️⃣ Income Tracking Pattern (NEW in V11)

**Used in:** Income sheet (Phase 8)

### Planned Income Lookup (Simple Mode)
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

**How it works:**
- LOOKUP finds the largest value that is ≤ target month
- Returns corresponding Planned_Amount
- Fallback to Typical_Monthly_Income if no history exists

### Planned Income by Source (Detailed Mode)
```excel
=IFERROR(
  LOOKUP(2, 
    1/(Income_Plan_History[Source_ID] = SourceID) *
      (Income_Plan_History[Start_Date] <= DATEVALUE(Month_ID&"-01")), 
    Income_Plan_History[Planned_Amount]),
  0)
```

**Filters by:**
- Source_ID (which income source)
- Start_Date ≤ target month (most recent entry)

### Actual Income Received (Simple Mode)
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], Month_ID,
        Transaction_DB[Type], "Income",
        Transaction_DB[Deleted?], FALSE)

Where:
  Month_ID = YEAR(Date)&"-"&TEXT(MONTH(Date),"00")
```

### Actual Income by Source (Detailed Mode)
```excel
=SUMIFS(Transaction_DB[Amount],
        Transaction_DB[Month_ID], Month_ID,
        Transaction_DB[Category], "Income: "&Source_Name,
        Transaction_DB[Deleted?], FALSE)
```

### Income Variance
```excel
=Actual_Income - Planned_Income
```

### Income Status Indicator
```excel
=IF(Actual_Income = 0,
   IF(DATEVALUE(Month_ID&"-01") <= TODAY(), "Missing", "Pending"),
   IF(Variance >= 0, "✓",
      IF(Variance/Planned_Income > -0.15, "⚠️", "🚨")))
```

**Status meanings:**
- "✓" = Met or exceeded target
- "⚠️" = Missed by <15% (minor shortfall)
- "🚨" = Missed by ≥15% (significant shortfall)
- "Missing" = Past month with no income recorded
- "Pending" = Future month

### Year-to-Date Income Summary
```excel
Average_Planned = AVERAGE(Planned_Jan:Planned_Dec)
Average_Actual = AVERAGE(FILTER(Actual_Range, Actual_Range<>0))
Months_Hit_Target = COUNTIFS(Status_Range, "✓") & " of " & COUNTIFS(Actual_Range, "<>0") & " months"
```

### Income Source Breakdown (Detailed Mode)
```excel
Source_Total = SUMIFS(Transaction_DB[Amount],
                       Transaction_DB[Month_ID], Month_ID,
                       Transaction_DB[Category], "Income: "&Source_Name,
                       Transaction_DB[Deleted?], FALSE)

Transaction_Count = COUNTIFS(Transaction_DB[Month_ID], Month_ID,
                              Transaction_DB[Category], "Income: "&Source_Name,
                              Transaction_DB[Deleted?], FALSE)
```
