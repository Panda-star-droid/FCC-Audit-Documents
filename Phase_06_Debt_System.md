# Phase 6: Debt System
**Deliverables:** Debt_DB, Debt_Payments_DB, Debt sheet  
**Estimated Time:** 3 hours  
**Dependencies:** Phases 1-3 (Foundation, Transactions, Utilities/Settings)

---

## 🎯 Deliverables

1. **Debt_DB** - Active debts with balances, APR, minimum payments
2. **Debt_Payments_DB** - Payment history log
3. **Debt** - User-facing sheet with debt tracking, strategy comparison, scenario planner

---

## 🔗 Dependencies

**Required from Phase 1 (Foundation):**
- `Settings_DB` - For currency and date formatting, debt strategy preference

**Required from Phase 2 (Transactions):**
- `Transaction_DB` - Debt payments create transaction entries (dual-write)
- Quick Entry forms include Pay Debt functionality

**Required from Phase 3 (Utilities/Settings):**
- Settings sheet for global configuration

**Provides to later phases (7-8):**
- Debt data for Net Worth liabilities calculation (Phase 7)
- Debt progress for Dashboard charts (Phase 8)
- Interest saved calculations for financial insights

---

## 📊 Database Schemas

### Debt_DB (Hidden)

**Columns:** A-K (11 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Text | "debt001" | Unique |
| B | Name | Text | "Credit Card A" | Max 50 chars |
| C | Month_ID | TEXT | "2024-06" | YYYY-MM from Start_Date |
| D | Orig_Bal | Number | 8000.00 | Original balance |
| E | Curr_Bal | Number | 7500.00 | Current balance (calculated) |
| F | APR | Number | 22.0 | Annual %, e.g., 22.0 = 22% |
| G | Min_Pay | Number | 225.00 | Minimum monthly payment |
| H | Start_Date | Date | 2024-06-01 | When debt added |
| I | Status | Text | "Active" | Active/Paid Off/Archived |
| J | Last_Pay_Date | Date | 2025-10-10 | Last payment date |
| K | Created | Date | 2024-06-01 | When record created |

**Sample Data (2 debts):**
```
1. Credit Card A: $7,500 @ 22% APR, $225 min, 5 payments logged
2. Credit Card B: $2,500 @ 11% APR, $75 min, 3 payments logged
```

**Month_ID Formula:**
```excel
C2: =TEXT(H2,"YYYY-MM")  /* Auto-generate from Start_Date */
```

**Named Ranges:**
```
Debt_DB[Name] = $B$2:$B$50
Debt_DB[Month_ID] = $C$2:$C$50
Debt_DB[Curr_Bal] = $E$2:$E$50
Active_Debts = FILTER(..., Status="Active")
```

**Current Balance Calculation:**
```excel
Curr_Bal = Orig_Bal - SUMIFS(Debt_Payments_DB[Amount], 
                              Debt_Payments_DB[Debt_ID], This_Debt_ID)
```

**Indexes:**
- Index on: Month_ID (for tracking debt history by month)

---

### Debt_Payments_DB (Hidden)

**Columns:** A-G (7 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | ID | Text | "dp001" | Unique |
| B | Debt_ID | Text | "debt001" | Links to Debt_DB |
| C | Date | Date | 2025-10-11 | Payment date |
| D | Month_ID | TEXT | "2025-10" | YYYY-MM from Date |
| E | Amount | Number | 100.00 | Payment amount |
| F | From_Acct | Text | "Checking" | Optional account |
| G | Transaction_ID | Text | "t12345" | Links to Transaction_DB |

**Sample Data (8 payments):**
```
5 payments to Credit Card A ($100-250 each)
3 payments to Credit Card B ($75 each)
```

**Month_ID Formula:**
```excel
D2: =TEXT(C2,"YYYY-MM")  /* Auto-generate from Date */
```

**Named Ranges:**
```
Debt_Payments_DB[Month_ID] = $D$2:$D$500
Debt_Payments_DB[Amount] = $E$2:$E$500
Debt_Payments_DB[Debt_ID] = $B$2:$B$500
```

**Indexes:**
- Index on: Month_ID (for monthly payment tracking in Dashboard)

---

## 🎨 Debt Sheet Layout

**Freeze:** Row 3

**Column Widths:**
```
A: 200px (Debt name) | B-H: 110px each (data columns) | I: 120px (Actions)
```

### Row 1-10: Snapshot Cards + Controls

**A1 (Title):**
```
"DEBT TRACKER" - 18pt Bold Navy, Lt Blue bg, merge A1:P1
```

**A3-D3: Snapshot Cards (4-up)**
```
Card 1: Total Balance   | $10,000
Card 2: Weighted APR ℹ️  | 18.2%
Card 3: Debt-Free ETA ℹ️ | Jul 2027 (32 months)
Card 4: Total Paid      | $800 (this month)
```

**APR ℹ️ Tooltip:**
```
Weighted average: (Bal₁×APR₁ + Bal₂×APR₂) ÷ Total Balance
Higher APR = more interest paid. Avalanche method targets highest APR first.
Example: 22% = $22 per $100 per year in interest.
```

**ETA ℹ️ Tooltip (Expandable):**
```
Debt-Free Date Calculation:
• Total Debt: $10,000
• Monthly Payments: $350 (minimums)
• Avg APR: 18.2% → Monthly Interest: ~$150
• Net Progress: $200/month
• Estimated Payoff: Jul 2027 (32 months)

Note: Assumes minimums only. Add extra payment to accelerate.
```

### Row 12-25: Debt List Table

**Headers (Row 12):**
```
A12: Debt Name | B12: Balance | C12: APR | D12: Min Pay | E12: Priority | F12: Actions
```

**Visual Specs:**
- Headers: Navy bg, White Bold 12pt, 32px height
- Data rows: 24px height, alternating White/Lt Blue
- Numbers: Right-aligned, $#,##0.00 (balance/payment), 0.0% (APR)

**Priority Column (E):**
```
Formula: =IF(APR=MAX(APR_Range), "🔴 Highest APR", 
            IF(Balance=MIN(Balance_Range), "🟡 Smallest", ""))
```

**Actions Column (F):**
```
[Pay] [Edit] [Delete]
Each is a hyperlink/protected cell
```

### Row 27-50: Strategy Comparison

**A27 Title:**
```
"DEBT PAYOFF STRATEGIES" - 14pt Bold Navy
```

**A29-J50: Strategy Cards (side-by-side)**

**Left Side (A29-E50): Snowball**
```
┌───────────────────────────────────────┐
│ 🗻 SNOWBALL (Smallest Balance First)  │
├───────────────────────────────────────┤
│ ✅ Quick wins boost motivation        │
│ ✅ First debt paid in 3-6 months      │
│ ❌ Pays more interest over time       │
│ 💡 Best for: Need momentum            │
├───────────────────────────────────────┤
│ Results (Extra $0/month):             │
│ • Months to Debt-Free: 34             │
│ • Total Interest: $5,800              │
│ • First Payoff: Credit Card B (11mo) │
└───────────────────────────────────────┘
```

**Right Side (G29-K50): Avalanche**
```
┌───────────────────────────────────────┐
│ 📊 AVALANCHE (Highest APR First)      │
├───────────────────────────────────────┤
│ ✅ Mathematically optimal             │
│ ✅ Minimizes total interest           │
│ ❌ First payoff may take 6-12mo       │
│ 💡 Best for: Math-driven savers       │
├───────────────────────────────────────┤
│ Results (Extra $0/month):             │
│ • Months to Debt-Free: 32             │
│ • Total Interest: $5,200              │
│ • First Payoff: Credit Card A (18mo) │
└───────────────────────────────────────┘
```

**A51: Recommendation**
```
"💡 OUR RECOMMENDATION: Start Avalanche (saves $600 in interest)"
Or: "Snowball for 3-6mo → Switch to Avalanche"
```

### Row 52-85: Scenario Planner

**A52 Title:**
```
"DEBT PAYOFF SCENARIOS" - 14pt Bold Navy
```

**A54-D55: Controls**
```
A54: Strategy:       ◉ Avalanche  ◯ Snowball  ◯ Hybrid
C54: Extra Payment:  [+$0] [+$50] [+$100] [+$250] [+$500] [+$1K]
D54: OR Custom: $[____] (unlimited)
```

**A56: [📊 Calculate Scenarios] Button**
```
44px height, Navy bg, White text
Triggers formula recalculation
```

**A58-J85: Results**

**Chart 1: Months to Debt-Free (A60-J70)**
```
Horizontal bar chart:
Current (minimums):     ████████████████████████████████ 32mo
Snowball + $100:        ████████████████████ 20mo
Avalanche + $100:       ██████████████████ 18mo ✅ FASTEST
```

**Chart 2: Total Interest Paid (A72-J82)**
```
Horizontal bar chart:
Current (minimums):     ████████████████ $5,200
Snowball + $100:        ████████ $2,800
Avalanche + $100:       ██████ $2,100 ✅ SAVES $3,100
```

**Table: Payoff Order (A84-J85)**
```
Avalanche + $100:
1. Credit Card A ($7,500 @ 22%) → Paid Month 18
2. Credit Card B ($2,500 @ 11%) → Paid Month 18 (freed funds)
```

**A87: Disclaimer**
```
"⚠️ Educational estimates only. Actual results vary. Not financial advice."
```

### Row 90-120: Debt Progress Chart (Optional)

**A90: Debt Progress Over Time (Line Chart)**
```
X-axis: Months (past 6 months)
Y-axis: Total Debt Balance ($0-$12,000)
Line: Red dashed, 2pt
Trend: Downward (if payments made)
```

---

## 🔧 Key Formulas

### Current Balance (Debt_DB Column D)

```excel
=Orig_Balance - SUMIFS(Debt_Payments_DB[Amount],
                       Debt_Payments_DB[Debt_ID],
                       This_Debt_ID)
```

### Weighted Average APR

```excel
=SUMPRODUCT(Debt_DB[Curr_Bal], Debt_DB[APR]) / SUM(Debt_DB[Curr_Bal])
```

### Debt-Free ETA (Simplified)

```excel
Total_Balance = SUM(Active_Debts[Curr_Bal])
Monthly_Payments = SUM(Active_Debts[Min_Pay])
Monthly_Interest = (Total_Balance * Weighted_APR) / 12
Net_Progress = Monthly_Payments - Monthly_Interest

Months_To_Debt_Free = ROUNDUP(Total_Balance / Net_Progress, 0)
```

**Accurate (using financial formulas):**
```excel
=NPER(Weighted_APR / 12,
      -(Total_Min_Payments + Extra_Payment),
      Total_Balance)
```

### Interest Saved (Strategy Comparison)

```excel
Interest_Current = (Months_Current * Avg_Interest_Per_Month)
Interest_Strategy = (Months_Strategy * Avg_Interest_Per_Month_Strategy)
Interest_Saved = Interest_Current - Interest_Strategy
```

### Payoff Order (Avalanche)

```excel
=SORT(Active_Debts, 
      MATCH("APR", Debt_DB_Headers, 0), 
      -1)  /* Sort by APR descending */
```

### Payoff Order (Snowball)

```excel
=SORT(Active_Debts,
      MATCH("Curr_Bal", Debt_DB_Headers, 0),
      1)  /* Sort by balance ascending */
```

### Pay Debt Dual-Write (from Phase 2 Quick Entry)

**Transaction_DB Append:**
```excel
=IF(PayDebt_Trigger_Detected,
   {PayDebt_Date,
    "Expense",
    "Debt Payment - " & PayDebt_Debt_Name,
    "Debt Payment",
    -1 * PayDebt_Amount,
    PayDebt_From_Account,
    "",
    "",
    NOW(),
    FALSE,
    TEXT(RAND()*1E15,"0")},
   "")
```

**Debt_Payments_DB Append:**
```excel
=IF(PayDebt_Trigger_Detected,
   {TEXT(RAND()*1E15,"0"),                           /* ID */
    VLOOKUP(PayDebt_Debt, Debt_DB, 1, FALSE),       /* Debt_ID */
    PayDebt_Date,
    PayDebt_Amount,
    PayDebt_From_Account,
    Transaction_ID_From_Above},                      /* Links */
   "")
```

---

## ✅ Acceptance Tests (30 tests)

### Debt Management Tests (10)

1. **Load Sample Debts**
   - ✅ 2 sample debts present (Credit Card A, B)
   - ✅ Balances, APR, min payments set
   - ✅ Status = "Active"

2. **Current Balance Calculation**
   - Credit Card A: Orig $8,000, Payments total $500
   - ✅ Current Balance = $7,500

3. **Add New Debt (Quick Entry)**
   - Go to Quick Entry → Add Debt form
   - Fill: "Student Loan", $20,000 balance, 5.0% APR, $200 min
   - Press ENTER
   - ✅ New row in Debt_DB
   - ✅ Appears in Debt sheet table
   - ✅ Status = "Active"

4. **Pay Debt (Quick Entry)**
   - Fill Pay Debt form: Credit Card A, $100 payment
   - Press ENTER
   - ✅ Creates Transaction_DB entry (Expense, -$100)
   - ✅ Creates Debt_Payments_DB entry
   - ✅ Credit Card A balance decreases by $100

5. **Pay Debt (Dashboard Mini-Form)**
   - Same as above but from Dashboard
   - ✅ Same dual-write behavior
   - ✅ Status cell shows "✅ Payment recorded! Card A: $7,500 → $7,400"

6. **Weighted APR Calculation**
   - Card A: $7,500 @ 22% | Card B: $2,500 @ 11%
   - ✅ Weighted APR = (7500×22 + 2500×11) / 10000 = 18.2%

7. **Debt-Free ETA**
   - Total balance $10,000, min payments $300/mo
   - ✅ ETA displayed (e.g., "Jul 2027, 32 months")
   - ✅ Tooltip expands with calculation details

8. **Edit Debt**
   - Click [Edit] on Credit Card A
   - Change APR from 22% to 19%
   - Save
   - ✅ Debt_DB updates
   - ✅ Weighted APR recalculates
   - ✅ ETA recalculates

9. **Delete Debt**
   - Click [Delete] on paid-off debt
   - ✅ Confirmation: "Archive 'Credit Card B'? Payment history preserved. [Cancel] [Archive]"
   - Confirm
   - ✅ Status = "Archived" (not deleted)
   - ✅ Removed from active debt table
   - ✅ Payment history remains in Debt_Payments_DB

10. **Payment History**
    - ✅ Debt_Payments_DB shows all payments
    - ✅ Linked to Transaction_DB entries
    - ✅ Date, amount, debt name visible

10a. **Month_ID Auto-Population**
    - ✅ Month_ID in Debt_DB auto-populates from Start_Date
    - ✅ Month_ID in Debt_Payments_DB auto-populates from Date
    - ✅ Format is "YYYY-MM" (e.g., "2025-10")
    - ✅ Enables monthly debt tracking for Dashboard

### Strategy & Scenario Tests (10)

11. **Strategy Comparison Display**
    - ✅ Snowball card displays with pros/cons
    - ✅ Avalanche card displays with pros/cons
    - ✅ Results show months, interest, first payoff

12. **Scenario Planner - No Extra Payment**
    - Extra payment = $0
    - Click [Calculate]
    - ✅ Results match minimums-only scenario
    - ✅ Months to debt-free calculated
    - ✅ Total interest calculated

13. **Scenario Planner - Quick Buttons**
    - Click [+$100] quick button
    - ✅ Extra payment field auto-fills to $100
    - ✅ Results recalculate instantly
    - ✅ Months decrease, interest saved increases

14. **Scenario Planner - Custom Amount**
    - Enter custom amount: $250
    - Click [Calculate]
    - ✅ Results recalculate
    - ✅ Custom amount used in formulas

15. **Scenario Planner - Unlimited Extra Payment**
    - Enter $5,000 extra payment
    - ✅ Accepts unlimited amount (no cap)
    - ✅ Results show very fast payoff (e.g., 3 months)

16. **Avalanche Payoff Order**
    - Extra $100, Avalanche strategy
    - ✅ Payoff order: Credit Card A first (22% APR)
    - ✅ Then Credit Card B (11% APR)

17. **Snowball Payoff Order**
    - Extra $100, Snowball strategy
    - ✅ Payoff order: Credit Card B first ($2,500 balance)
    - ✅ Then Credit Card A ($7,500 balance)

18. **Interest Saved Calculation**
    - Compare minimums vs Avalanche +$100
    - ✅ Interest saved = (34mo × $153/mo) - (18mo × $117/mo)
    - ✅ Shows realistic savings (e.g., $3,100)

19. **Charts Update**
    - Change extra payment amount
    - ✅ Months chart updates
    - ✅ Interest chart updates
    - ✅ Payoff order table updates

20. **Debt Progress Chart (Optional)**
    - If implemented:
    - ✅ Shows total debt balance over past 6 months
    - ✅ Downward trend if payments made
    - ✅ Updates when new payments logged

---

## 🚧 Known Issues / Blockers

*Document during build.*

---

## 📝 Implementation Checklist

### Step 1: Create Database Sheets
- [ ] Debt_DB (hidden, 11 columns - includes Month_ID)
- [ ] Add Month_ID formula: =TEXT(H2,"YYYY-MM") in column C
- [ ] Debt_Payments_DB (hidden, 7 columns - includes Month_ID)
- [ ] Add Month_ID formula: =TEXT(C2,"YYYY-MM") in column D
- [ ] Add indexes on Month_ID columns
- [ ] Load sample data (2 debts, 8 payments)

### Step 2: Create Debt Sheet
- [ ] Title and snapshot cards (rows 1-10)
- [ ] Debt list table (rows 12-25)
- [ ] Strategy comparison cards (rows 27-50)
- [ ] Scenario planner (rows 52-85)
- [ ] Optional: Debt progress chart (rows 90-120)

### Step 3: Add Formulas
- [ ] Current balance calculation
- [ ] Weighted APR
- [ ] Debt-free ETA (NPER formula)
- [ ] Interest saved
- [ ] Payoff order (Avalanche, Snowball)
- [ ] Scenario calculations

### Step 4: Link to Quick Entry
- [ ] Verify Pay Debt form creates dual entries
- [ ] Verify Add Debt form populates Debt_DB
- [ ] Test balance updates on payment

### Step 5: Charts
- [ ] Months to debt-free (horizontal bars)
- [ ] Total interest paid (horizontal bars)
- [ ] Optional: Debt progress over time (line chart)

### Step 6: Testing
- [ ] Run all 20 acceptance tests
- [ ] Verify strategy comparison accurate
- [ ] Verify scenario planner calculations
- [ ] Check charts update correctly

---

## 🎯 Success Criteria

Phase 4 complete when all 20 tests pass and:
- Debt tracking accurate
- Payments update balances correctly
- Strategy comparison shows realistic results
- Scenario planner works with unlimited extra payments

---

## 📞 Next Phase

**Phase 5: Net Worth System**  
Depends on: Transaction_DB (optional), Debt_DB (liabilities)  
Estimated: 3 hours
