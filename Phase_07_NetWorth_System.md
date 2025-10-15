# Phase 7: Net Worth System
**Deliverables:** Net_Worth_Snapshots_DB, Net Worth sheet  
**Estimated Time:** 3 hours  
**Dependencies:** Phases 1, 3, 6 (Foundation, Utilities/Settings, Debt)

---

## 🎯 Deliverables

1. **Net_Worth_Snapshots_DB** - Monthly snapshots with asset/liability breakdown
2. **Net Worth** - User-facing sheet with account tracking, projection, trends

---

## 🔗 Dependencies

**Required from Phase 1 (Foundation):**
- `Settings_DB` - For currency formatting

**Required from Phase 3 (Utilities/Settings):**
- Settings sheet for global configuration

**Required from Phase 6 (Debt):**
- `Debt_DB` - For calculating total liabilities in net worth
- Credit card and loan balances feed into liability calculations

**Provides to later phases (8):**
- Net worth snapshots for Dashboard trend chart
- Month-over-month change for Dashboard KPI
- Asset allocation data for potential Dashboard visualization

---

## 📊 Database Schema

### Net_Worth_Snapshots_DB (Hidden)

**Columns:** A-I (9 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Text | "nws001" | Unique |
| B | Date | Date | 2025-10-31 | Snapshot date (typically month-end) |
| C | Month_ID | TEXT | "2025-10" | YYYY-MM from Date |
| D | Total_Assets | Number | 273500.00 | Sum of all assets |
| E | Total_Liab | Number | 228500.00 | Sum of all liabilities |
| F | Net_Worth | Number | 45000.00 | Assets - Liabilities |
| G | Assets_JSON | Text | "Cash:15500\|Inv:43000..." | Serialized account list |
| H | Liab_JSON | Text | "CC:10000\|Mortgage:180000..." | Serialized debt list |
| I | Notes | Text | "Added IRA" | Optional notes |

**Sample Data (2 snapshots):**
```
1. Current Month (2025-10-31):
   Assets: $273,500 | Liabilities: $228,500 | Net Worth: $45,000

2. Previous Month (2025-09-30):
   Assets: $271,000 | Liabilities: $228,500 | Net Worth: $42,500
   Change: +$2,500 (+5.9%)
```

**Month_ID Formula:**
```excel
C2: =TEXT(B2,"YYYY-MM")  /* Auto-generate from Date */
```

**Named Ranges:**
```
Net_Worth_Snapshots_DB[Date] = $B$2:$B$100
Net_Worth_Snapshots_DB[Month_ID] = $C$2:$C$100
Net_Worth_Snapshots_DB[Net_Worth] = $F$2:$F$100
Latest_Snapshot = INDEX(Net_Worth_Snapshots_DB, MAX(ROW(...)))
```

**Indexes:**
- Index on: Month_ID (for fast Dashboard filtering by month)

---

## 🎨 Net Worth Sheet Layout

**Freeze:** Row 3

**Column Widths:**
```
A: 200px (Account name) | B: 120px (Value) | C: 80px (Actions)
```

### Row 1-5: Hero Section

**A1 (Title):**
```
"NET WORTH TRACKER" - 18pt Bold Navy, Lt Blue bg, merge A1:P1
```

**A2: As-Of Date Selector**
```
B2: [MM/DD/YYYY] dropdown or manual entry
Default: Last day of current month
```

**A3-D4: Hero Display (merged cells)**
```
CURRENT NET WORTH: $45,000
(36pt Bold Navy, centered)

Month-over-Month: +$2,500 (+5.9%) ▲
(18pt Regular, Green if positive, Red if negative)
```

**A5: [📊 Log Snapshot ℹ️] Button**
```
44px height, Navy bg, White text
Tooltip: "Click monthly (e.g., last day of month) to track progress. 
          Snapshots power the trend chart and projection."
```

### Row 7-60: Account Breakdown by Category

**A7 Title:**
```
"ACCOUNTS" - 14pt Bold Navy
```

#### Cash & Checking (A9-C18)

**A9: 📊 CASH & CHECKING**
```
TOTAL: $15,500 (right-aligned, 12pt Bold)
```

**A10-A15: Accounts Table**
```
Headers: Account Name | Value | Actions
Row 10: Checking 1    | $3,500  | [Edit]
Row 11: Savings       | $12,000 | [Edit]
Row 12: [➕ Add Cash Account] (44px button, full width)
```

#### Investments (A20-C29)

**A20: 📈 INVESTMENTS**
```
TOTAL: $43,000
```

**A21-A26:**
```
Row 21: 401(k)        | $28,000 | [Edit]
Row 22: Roth IRA      | $15,000 | [Edit]
Row 23: [➕ Add Investment]
```

#### Property (A31-C40)

**A31: 🏠 PROPERTY**
```
TOTAL: $215,000
```

**A32-A37:**
```
Row 32: Home (est.)   | $200,000 | [Edit]
Row 33: Car (KBB)     | $15,000  | [Edit]
Row 34: [➕ Add Property]
```

#### Credit Cards (A42-C51)

**A42: 💳 CREDIT CARDS**
```
TOTAL: -$10,000 (Red text)
```

**A43-A48:**
```
Row 43: Card A        | -$7,500  | [Edit]
Row 44: Card B        | -$2,500  | [Edit]
Row 45: [➕ Add Credit Card]
```

**Note:** These link to Debt_DB for active debts with Type="Credit Card"

#### Loans (A53-C62)

**A53: 🎓 LOANS**
```
TOTAL: -$218,500 (Red text)
```

**A54-A59:**
```
Row 54: Mortgage      | -$180,000 | [Edit]
Row 55: Student Loan  | -$38,500  | [Edit]
Row 56: [➕ Add Loan]
```

**Note:** These link to Debt_DB for active debts with Type="Loan"

**A61: Summary Line**
```
NET WORTH = Assets ($273,500) - Liabilities ($228,500) = $45,000
(14pt Bold Navy, centered, merge A61:C61)
```

### Row 64-78: Projection Controls

**A64 Title:**
```
"PROJECTION SETTINGS" - 14pt Bold Navy
```

**A66-C70: Inputs**
```
A66: Monthly Contribution: $[500]
A67: Assumed Growth Rate:  [7.0]% per year
A68: Projection Period:    [10] years

A70: [📊 Update Projection] (44px button)
```

**Future Value Formula:**
```excel
FV = Current_NW × (1 + Growth_Rate)^Years + 
     Monthly_Contrib × ((1 + Growth_Rate)^Years - 1) / (Growth_Rate/12)
```

**A72-C76: Results**
```
A72: Projected Net Worth (10 years): $XXX,XXX
A73: Total Contributions: $XX,XXX
A74: Growth from Returns: $XX,XXX
```

### Row 80-110: Visualizations

**Chart 1: Net Worth Trend (A80-J95)**
```
Type: Line chart
X-axis: Months (past 12 months + projected 10 years)
Y-axis: Net Worth ($0-$500K, auto-scale)
Series 1: Historical (solid blue, 2pt, markers)
Series 2: Projected (dashed blue, 2pt, no markers, 50% opacity)
Current point: Highlighted (larger marker, Gold)
```

**Chart 2: Asset Allocation (L80-T95)**
```
Type: Donut chart
Data: Current assets breakdown
Slices:
  - Cash (Navy)
  - Investments (Gold)
  - Property (Green)
  - Other (Gray)
Inner radius: 40%
Center text: "Assets: $273,500" (18pt Bold Navy)
Legend: Right side
```

**Table: Category Summary (A97-D110)**
```
Headers: Category | Value | % of Assets | % of Net Worth

Row 98:  Cash          | $15,500   | 5.7%    | 34.4%
Row 99:  Investments   | $43,000   | 15.7%   | 95.6%
Row 100: Property      | $215,000  | 78.6%   | 477.8%
Row 101: ---           | ---       | ---     | ---
Row 102: Liabilities   | -$228,500 | N/A     | -507.8%
Row 103: ---           | ---       | ---     | ---
Row 104: NET WORTH     | $45,000   | ---     | 100%
```

---

## 🔧 Key Formulas

### Current Net Worth

```excel
Total_Assets = SUM(Cash_Range, Investments_Range, Property_Range, Other_Assets_Range)
Total_Liabilities = SUM(Credit_Cards_Range, Loans_Range, Other_Liab_Range)
Net_Worth = Total_Assets - Total_Liabilities
```

**Liabilities Link to Debt_DB:**
```excel
Credit_Cards_Total = -1 * SUMIFS(Debt_DB[Curr_Bal], 
                                  Debt_DB[Type], "Credit Card",
                                  Debt_DB[Status], "Active")

Loans_Total = -1 * SUMIFS(Debt_DB[Curr_Bal],
                           Debt_DB[Type], "Loan",
                           Debt_DB[Status], "Active")
```

### Month-over-Month Change

```excel
Current_NW = Latest_Snapshot[Net_Worth]
Previous_NW = VLOOKUP(EOMONTH(TODAY(),-1), 
                      Net_Worth_Snapshots_DB[[Date]:[Net_Worth]], 
                      5, FALSE)

MoM_Change = Current_NW - Previous_NW
MoM_Percent = (MoM_Change / Previous_NW) * 100
```

### Log Snapshot (Button Trigger)

```excel
=IF(Log_Snapshot_Trigger_Detected,
   {TEXT(RAND()*1E15,"0"),                    /* ID */
    TODAY(),                                   /* Date */
    TEXT(TODAY(),"YYYY-MM"),                  /* Month_ID */
    Total_Assets,
    Total_Liabilities,
    Net_Worth,
    TEXTJOIN("|", TRUE, 
             Accounts[Name] & ":" & Accounts[Value]),  /* Assets_JSON */
    TEXTJOIN("|", TRUE,
             Debts[Name] & ":" & Debts[Balance]),      /* Liab_JSON */
    ""},                                       /* Notes */
   "")
```

### Projection Formula

```excel
Growth_Rate_Monthly = Growth_Rate_Annual / 12
Periods = Years * 12

FV_From_Current = Current_NW * (1 + Growth_Rate_Monthly)^Periods

FV_From_Contributions = Monthly_Contribution * 
                        (((1 + Growth_Rate_Monthly)^Periods - 1) / 
                         Growth_Rate_Monthly)

Projected_NW = FV_From_Current + FV_From_Contributions
```

**OR use Excel's FV function:**
```excel
=FV(Growth_Rate_Annual/12,     /* rate per period */
    Years*12,                  /* number of periods */
    -Monthly_Contribution,     /* payment per period */
    -Current_NW,               /* present value */
    1)                         /* payments at beginning of period */
```

### Add Account (Form)

**Form expands when [➕ Add] clicked:**
```
Account Name: [____________________]
Current Value: $[______]
Category: [Cash â–¼] (auto-filled based on section)

[💾 Save Account] [Cancel]
```

**Save Action:**
- For assets: Add new row to internal Assets table
- For liabilities: Create entry in Debt_DB
- Update totals immediately

### Edit Account (Inline)

**[Edit] hyperlink behavior:**
- Makes Value cell editable
- User types new value, presses ENTER
- Totals recalculate automatically
- No separate save button needed

---

## ✅ Acceptance Tests (20 tests)

### Account Management Tests (8)

1. **Load Sample Accounts**
   - ✅ Cash accounts: Checking ($3,500), Savings ($12,000)
   - ✅ Investments: 401(k) ($28,000), IRA ($15,000)
   - ✅ Property: Home ($200,000), Car ($15,000)
   - ✅ Credit Cards: Card A (-$7,500), Card B (-$2,500)
   - ✅ Loans: Mortgage (-$180,000), Student (-$38,500)

2. **Total Assets Calculation**
   - ✅ Total Assets = $15,500 + $43,000 + $215,000 = $273,500

3. **Total Liabilities Calculation**
   - ✅ Total Liabilities = $10,000 + $218,500 = $228,500
   - ✅ Displayed as negative (-$228,500)

4. **Net Worth Calculation**
   - ✅ Net Worth = $273,500 - $228,500 = $45,000

5. **Add New Account**
   - Click [➕ Add Cash Account]
   - Fill: "Emergency Fund", $5,000
   - Click [Save]
   - ✅ New account appears in Cash section
   - ✅ Cash total increases by $5,000
   - ✅ Total Assets increases by $5,000
   - ✅ Net Worth increases by $5,000

6. **Edit Account Value**
   - Click [Edit] on Savings account
   - Change from $12,000 to $15,000
   - Press ENTER
   - ✅ Value updates immediately
   - ✅ Cash total increases by $3,000
   - ✅ Net Worth increases by $3,000

7. **Delete Account**
   - Click [Delete] on Car
   - ✅ Confirmation: "Remove 'Car' ($15,000)? [Cancel] [Remove]"
   - Confirm
   - ✅ Account removed from Property section
   - ✅ Property total decreases by $15,000
   - ✅ Net Worth decreases by $15,000

8. **Liabilities Link to Debt_DB**
   - Credit Cards total matches Debt_DB
   - ✅ Card A: Debt_DB balance = Net Worth display
   - Pay $100 on Card A (via Quick Entry)
   - ✅ Credit Cards total decreases by $100
   - ✅ Net Worth increases by $100

### Snapshot & Trend Tests (5)

9. **Log Snapshot**
   - Click [📊 Log Snapshot]
   - ✅ New row in Net_Worth_Snapshots_DB
   - ✅ Date = TODAY()
   - ✅ Month_ID auto-populated (e.g., "2025-10")
   - ✅ Total_Assets, Total_Liab, Net_Worth captured
   - ✅ Assets_JSON contains account breakdown

9a. **Month_ID Auto-Population**
   - ✅ Month_ID auto-populates from Date
   - ✅ Format is "YYYY-MM" (e.g., "2025-10")
   - ✅ Enables monthly net worth tracking for Dashboard

10. **Month-over-Month Change**
    - ✅ Current month: $45,000
    - ✅ Previous month: $42,500
    - ✅ MoM Change: +$2,500 (+5.9%)
    - ✅ Green text and ▲ arrow

11. **Net Worth Trend Chart**
    - ✅ Line chart displays
    - ✅ Past 12 months of snapshots visible
    - ✅ Current point highlighted (Gold marker)
    - ✅ Y-axis scales appropriately

12. **Asset Allocation Donut**
    - ✅ Donut chart displays
    - ✅ Slices: Cash (5.7%), Investments (15.7%), Property (78.6%)
    - ✅ Center text: "Assets: $273,500"
    - ✅ Colors match design system

13. **Category Summary Table**
    - ✅ All categories listed with values
    - ✅ % of Assets calculated correctly
    - ✅ % of Net Worth calculated correctly
    - ✅ Liabilities shown as negative %

### Projection Tests (5)

14. **Projection Settings**
    - Enter: $500/month, 7% growth, 10 years
    - Click [Update Projection]
    - ✅ Projected Net Worth calculated
    - ✅ Shows total contributions
    - ✅ Shows growth from returns

15. **Projection Formula Accuracy**
    - Current NW: $45,000 | Contrib: $500/mo | Rate: 7% | Years: 10
    - ✅ Projected NW ≈ $184,000 (±5%)
    - ✅ Total Contributions = $500 × 120 = $60,000
    - ✅ Growth from Returns ≈ $79,000

16. **Projection Chart**
    - ✅ Dashed line extends from current point
    - ✅ 50% opacity (faded)
    - ✅ No markers on projected line
    - ✅ Ends at 10-year mark

17. **Change Projection Inputs**
    - Increase monthly contribution to $1,000
    - ✅ Projected NW increases
    - ✅ Chart updates
    - Increase growth rate to 9%
    - ✅ Projected NW increases further

18. **Projection with Zero Contributions**
    - Set monthly contribution to $0
    - ✅ Projected NW = Current × (1 + Rate)^Years
    - ✅ Formula doesn't error
    - ✅ Chart displays correctly

---

## 🚧 Known Issues / Blockers

*Document during build.*

---

## 📝 Implementation Checklist

### Step 1: Create Database Sheet
- [ ] Net_Worth_Snapshots_DB (hidden, 9 columns - includes Month_ID)
- [ ] Add Month_ID formula: =TEXT(B2,"YYYY-MM") in column C
- [ ] Add index on Month_ID column
- [ ] Load sample data (2 snapshots)

### Step 2: Create Net Worth Sheet
- [ ] Hero section (rows 1-5)
- [ ] Account breakdown by category (rows 7-62)
- [ ] Projection controls (rows 64-78)
- [ ] Charts (rows 80-110)

### Step 3: Add Formulas
- [ ] Current net worth calculation
- [ ] Month-over-month change
- [ ] Log snapshot trigger
- [ ] Projection formula
- [ ] Asset allocation percentages

### Step 4: Link to Debt_DB
- [ ] Credit Cards total from Debt_DB
- [ ] Loans total from Debt_DB
- [ ] Verify updates on debt payment

### Step 5: Add Account Management
- [ ] Add Account forms (per category)
- [ ] Edit functionality (inline)
- [ ] Delete functionality (with confirmation)

### Step 6: Charts
- [ ] Net Worth Trend (line chart)
- [ ] Asset Allocation (donut chart)
- [ ] Category Summary table

### Step 7: Testing
- [ ] Run all 18 acceptance tests
- [ ] Verify net worth calculates correctly
- [ ] Verify snapshots capture data properly
- [ ] Check projection formula accuracy

---

## 🎯 Success Criteria

Phase 5 complete when all 18 tests pass and:
- Net worth calculation accurate
- Snapshots log correctly
- Projection formula works
- Charts display properly
- Liabilities link to Debt_DB

---

## 📞 Next Phase

**Phase 6: Utilities (Settings, CSV Import, Instructions)**  
Depends on: All database sheets (for CSV import normalization)  
Estimated: 4 hours
