# Phase 1: Foundation
**Deliverables:** Categories_DB, Settings_DB, Keywords_DB  
**Estimated Time:** 2 hours  
**Dependencies:** None (starting point)

---

## 🎯 Deliverables

1. **Categories_DB** - Category list with types, colors, icons
2. **Settings_DB** - Global configuration and preferences
3. **Keywords_DB** - Keywords for auto-categorization

---

## 🔗 Dependencies

**None** - This is the foundation layer. All other phases depend on this.

**Provides to future phases:**
- `Categories_DB[Name]` - Category dropdown lists
- `Settings_DB!Currency_Symbol` - Currency formatting
- `Settings_DB!Date_Format` - Date display format
- `Settings_DB!Period_Selected` - Period filter (Week/Month/Quarter/Year)
- `Settings_DB!Monthly_Income` - Budget calculations
- `Keywords_DB[Keyword]` → `Keywords_DB[Category]` - Auto-categorization

---

## 📊 Database Schemas

### Categories_DB (Hidden Sheet)

**Purpose:** Master list of income/expense categories

**Columns:** A-G (7 columns)

| Col | Name | Type | Example | Validation | Notes |
|-----|------|------|---------|------------|-------|
| A | Name | Text (30 char max) | "Housing" | Unique, no special chars | Display name |
| B | Type | Text | "Expense" | List: Income/Expense/Savings/Transfer | Category type |
| C | Color | Hex | "#1A2B4A" | List: 15 colors | Chart/visual color |
| D | Icon | Text | "Housing" | Text label, not emoji | Display label |
| E | Default? | Boolean | TRUE | TRUE/FALSE | Pre-loaded category? |
| F | Status | Text | "Active" | List: Active/Archived | Soft delete |
| G | Created | Date | 2025-01-15 | Auto: TODAY() | Audit trail |

**Indexes:**
- Primary: Name (unique)
- Filter: Status="Active"

**Sample Data (12 default categories):**

```
A        | B        | C        | D          | E    | F      | G
---------|----------|----------|------------|------|--------|------------
Housing  | Expense  | #1A2B4A  | Housing    | TRUE | Active | 2025-01-01
Groceries| Expense  | #AFB42B  | Groceries  | TRUE | Active | 2025-01-01
Dining Out|Expense  | #5D4037  | Dining     | TRUE | Active | 2025-01-01
Transport| Expense  | #0097A7  | Transport  | TRUE | Active | 2025-01-01
Utilities| Expense  | #FFA000  | Utilities  | TRUE | Active | 2025-01-01
Healthcare|Expense  | #303F9F  | Healthcare | TRUE | Active | 2025-01-01
Entertainment|Expense|#7B1FA2 | Entertainment|TRUE| Active | 2025-01-01
Shopping | Expense  | #C2185B  | Shopping   | TRUE | Active | 2025-01-01
Personal | Expense  | #00897B  | Personal   | TRUE | Active | 2025-01-01
Salary   | Income   | #D4AF37  | Income     | TRUE | Active | 2025-01-01
Savings  | Savings  | #2E7D32  | Savings    | TRUE | Active | 2025-01-01
Transfer | Transfer | #757575  | Transfer   | TRUE | Active | 2025-01-01
```

**Named Ranges:**
```
Categories_DB[Name]   = Categories_DB!$A$2:$A$1000
Categories_DB[Type]   = Categories_DB!$B$2:$B$1000
Categories_DB[Color]  = Categories_DB!$C$2:$C$1000
Categories_List       = Categories_DB!$A$2:$A$31 (max 30 custom)
Active_Categories     = FILTER(Categories_DB[Name], Categories_DB[Status]="Active")
```

**Constraints:**
- Max 30 categories total
- Cannot delete last category of each Type
- Name must be unique (case-insensitive)

---

### Settings_DB (Hidden Sheet)

**Purpose:** Global configuration key-value store

**Columns:** A-D (4 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | Setting_Name | Text | "Currency_Symbol" | Unique key |
| B | Setting_Value | Text/Number | "$" | Actual value |
| C | Data_Type | Text | "String" | String/Number/Boolean/Date |
| D | Last_Updated | Date | 2025-10-12 | Auto: NOW() |

**Sample Data (20 core settings):**

```
A                        | B              | C       | D
-------------------------|----------------|---------|------------
Currency_Symbol          | $              | String  | 2025-01-01
Currency_Code            | USD            | String  | 2025-01-01
Date_Format              | MM/DD/YYYY     | String  | 2025-01-01
Week_Start               | Sunday         | String  | 2025-01-01
Week_Start_Offset        | 1              | Number  | 2025-01-01
Monthly_Income           | 4000           | Number  | 2025-01-15
Budget_Method            | 50/30/20 Auto  | String  | 2025-01-15
Period_Selected          | Month          | String  | 2025-10-12
Setup_Complete           | FALSE          | Boolean | 2025-01-01
Setup_Step_1             | FALSE          | Boolean | 2025-01-01
Setup_Step_2             | FALSE          | Boolean | 2025-01-01
Setup_Step_3             | FALSE          | Boolean | 2025-01-01
Onboarding_State         | S0             | String  | 2025-01-01
Banner_Dismissed         | FALSE          | Boolean | 2025-01-01
Dismiss_Forever          | FALSE          | Boolean | 2025-01-01
Sample_Data_Active       | TRUE           | Boolean | 2025-01-01
Last_Transaction_Date    | 2025-10-10     | Date    | 2025-10-10
Last_Dashboard_Open      | 2025-10-12     | Date    | 2025-10-12
Default_Debt_Strategy    | Avalanche      | String  | 2025-01-01
Assumed_Investment_Return| 7.0            | Number  | 2025-01-01
```

**Named Ranges (Direct Cell References):**
```
Currency_Symbol     = Settings_DB!$B$2
Date_Format         = Settings_DB!$B$3
Period_Selected     = Settings_DB!$B$8
Monthly_Income      = Settings_DB!$B$6
Budget_Method       = Settings_DB!$B$7
Setup_Complete      = Settings_DB!$B$9
```

**Helper Function (VLOOKUP wrapper):**
```excel
=VLOOKUP("Currency_Symbol", Settings_DB!$A:$B, 2, FALSE)
```

**Update Pattern:**
```excel
When user changes setting:
- Update Setting_Value (column B)
- Update Last_Updated to NOW() (column D)
```

---

### Keywords_DB (Hidden Sheet)

**Purpose:** Keyword-to-category mapping for auto-categorization

**Columns:** A-C (3 columns)

| Col | Name | Type | Example | Notes |
|-----|------|------|---------|-------|
| A | Keyword | Text | "starbucks" | Lowercase, partial match |
| B | Category | Text | "Dining Out" | Must exist in Categories_DB |
| C | Confidence | Number | 90 | 0-100, higher = better match |

**Sample Data (30 common keywords):**

```
A           | B             | C
------------|---------------|----
starbucks   | Dining Out    | 95
dunkin      | Dining Out    | 95
mcdonalds   | Dining Out    | 95
chipotle    | Dining Out    | 95
restaurant  | Dining Out    | 80
cafe        | Dining Out    | 80
safeway     | Groceries     | 95
walmart     | Groceries     | 90
target      | Shopping      | 85
grocery     | Groceries     | 90
whole foods | Groceries     | 95
shell       | Transport     | 95
chevron     | Transport     | 95
gas         | Transport     | 85
uber        | Transport     | 90
lyft        | Transport     | 90
netflix     | Entertainment | 95
spotify     | Entertainment | 95
hulu        | Entertainment | 95
amazon prime| Entertainment | 90
rent        | Housing       | 95
mortgage    | Housing       | 95
electric    | Utilities     | 90
water       | Utilities     | 90
internet    | Utilities     | 90
cvs         | Healthcare    | 90
walgreens   | Healthcare    | 90
pharmacy    | Healthcare    | 85
salary      | Salary        | 95
paycheck    | Salary        | 95
```

**Named Ranges:**
```
Keywords_DB[Keyword]   = Keywords_DB!$A$2:$A$500
Keywords_DB[Category]  = Keywords_DB!$B$2:$B$500
Keywords_DB[Confidence]= Keywords_DB!$C$2:$C$500
```

**Matching Formula (used in CSV Import):**
```excel
=IFERROR(INDEX(Keywords_DB[Category],
               MATCH("*"&LOWER(Description)&"*", 
                     Keywords_DB[Keyword], 0)),
         "Uncategorized")
```

**Adding Keywords (Future - Settings Sheet):**
User can add custom keywords via Settings > Category Management.

---

## 🎨 Layout & Visual Specs

### All Three Sheets: Hidden by Default

**Sheet Protection:**
- Lock all cells
- Allow: Filter, Sort (for dev/debug only)
- Password: None (user can unhide if needed)

**Visibility:**
- Hidden (not VeryHidden)
- User can unhide via Format > Sheet > Unhide if needed

**Column Widths:**
```
Categories_DB:
  A: 150px (Name)
  B: 80px  (Type)
  C: 80px  (Color)
  D: 120px (Icon)
  E: 60px  (Default?)
  F: 70px  (Status)
  G: 90px  (Created)

Settings_DB:
  A: 200px (Setting_Name)
  B: 150px (Setting_Value)
  C: 80px  (Data_Type)
  D: 100px (Last_Updated)

Keywords_DB:
  A: 150px (Keyword)
  B: 120px (Category)
  C: 80px  (Confidence)
```

**Header Rows (Row 1):**
```
Background: #1A2B4A (Navy)
Font: 12pt Bold White
Height: 32px
Alignment: Center
```

**Data Rows:**
```
Background: Alternating White / #E8F1F8 (Lt Blue)
Font: 11pt Regular Black
Height: 24px
Border: 1px bottom Lt Gray
```

**Conditional Formatting (Categories_DB):**
```
Rule 1: Status="Archived"
  Formula: =$F2="Archived"
  Format: Font Gray, Strikethrough
```

---

## 🔧 Formulas & Named Ranges

### Named Ranges to Create

**Categories_DB:**
```
Name: Categories_DB[Name]
Refers to: =Categories_DB!$A$2:$A$1000

Name: Active_Categories
Refers to: =FILTER(Categories_DB!$A$2:$A$1000, Categories_DB!$F$2:$F$1000="Active")
(Note: For Excel 2016, use helper column instead of FILTER)
```

**Settings_DB:**
```
Name: Currency_Symbol
Refers to: =Settings_DB!$B$2

Name: Period_Selected
Refers to: =Settings_DB!$B$8

(Create similar for all 20 settings)
```

**Keywords_DB:**
```
Name: Keywords_DB[Keyword]
Refers to: =Keywords_DB!$A$2:$A$500
```

### Formulas (Minimal - mostly static data)

**Categories_DB!G2 (Created Date - if blank):**
```excel
=IF(A2<>"", IF(G2="", TODAY(), G2), "")
```
(Auto-fills today's date when category added, then persists)

**Settings_DB!D2 (Last_Updated - if Value changes):**
```excel
=IF(B2<>B2_Previous, NOW(), D2)
```
(Note: Requires helper column or manual update for MVP)

**No other formulas needed** - these are data tables.

---

## ✅ Acceptance Tests (15 tests)

### Categories_DB Tests (5)

1. **Load Default Categories**
   - ✅ 12 default categories present (Housing through Transfer)
   - ✅ All have Status="Active"
   - ✅ All have unique names
   - ✅ All have assigned colors from palette

2. **Add New Category**
   - Create new row: "Subscriptions", "Expense", "#0097A7", "Subscriptions", FALSE, "Active", TODAY()
   - ✅ Category appears in Active_Categories range
   - ✅ Name is unique (validation blocks duplicates)
   - ✅ Max 30 categories enforced

3. **Archive Category**
   - Change Status of "Entertainment" to "Archived"
   - ✅ Category removed from Active_Categories
   - ✅ Strikethrough formatting applied
   - ✅ Still exists in full Categories_DB[Name] range

4. **Color Validation**
   - ✅ All 12 defaults use colors from 15-color palette
   - ✅ Each color used once (no duplicates initially)

5. **Type Validation**
   - ✅ Only valid types: Income, Expense, Savings, Transfer
   - ✅ Cannot enter invalid type (data validation)

### Settings_DB Tests (6)

6. **Load Default Settings**
   - ✅ 20 core settings present
   - ✅ Currency_Symbol = "$" (or detected from locale)
   - ✅ Date_Format = "MM/DD/YYYY" (or detected)
   - ✅ Monthly_Income = 4000 (sample data)

7. **Update Setting Value**
   - Change Currency_Symbol from "$" to "€"
   - ✅ Value updates in cell B2
   - ✅ Last_Updated changes to NOW()
   - ✅ Named range Currency_Symbol reflects new value

8. **Period Selector**
   - Change Period_Selected to "Week"
   - ✅ Value updates
   - ✅ Future phases can read this value for filtering

9. **Setup Flags**
   - ✅ Setup_Complete defaults to FALSE
   - ✅ All Setup_Step_X default to FALSE
   - ✅ Onboarding_State defaults to "S0"

10. **Numeric Settings**
    - ✅ Monthly_Income accepts numbers only
    - ✅ Assumed_Investment_Return accepts decimals (7.0)
    - ✅ Week_Start_Offset accepts 1-7

11. **Boolean Settings**
    - ✅ Setup_Complete, Banner_Dismissed, etc. accept TRUE/FALSE only
    - ✅ No mixed case (true/false)

### Keywords_DB Tests (4)

12. **Load Default Keywords**
    - ✅ 30 common keywords present
    - ✅ All keywords lowercase
    - ✅ All categories exist in Categories_DB

13. **Keyword Matching (Formula Test)**
    - Test description: "Starbucks Coffee Shop"
    - ✅ Match formula returns "Dining Out"
    - ✅ Case-insensitive match works

14. **No Match Returns Uncategorized**
    - Test description: "Unknown Merchant XYZ"
    - ✅ Returns "Uncategorized"
    - ✅ No error

15. **Confidence Scoring**
    - ✅ All confidence values between 0-100
    - ✅ Higher confidence keywords (95+) are exact merchant names
    - ✅ Lower confidence keywords (80-85) are generic terms

---

## 🚧 Known Issues / Blockers

*None at start of phase. Document any issues encountered here.*

**Example entries:**
```
- Excel 2016 doesn't support FILTER() in Active_Categories range
  → Workaround: Use helper column with IF formula

- Named ranges not updating after data entry
  → Solution: Press Ctrl+Alt+F9 to force recalculation
```

---

## 📝 Implementation Checklist

### Step 1: Create Sheets
- [ ] Create Categories_DB sheet
- [ ] Create Settings_DB sheet
- [ ] Create Keywords_DB sheet
- [ ] Hide all three sheets

### Step 2: Add Headers
- [ ] Categories_DB headers (A1:G1)
- [ ] Settings_DB headers (A1:D1)
- [ ] Keywords_DB headers (A1:C1)
- [ ] Apply header formatting (Navy bg, White text, Bold)

### Step 3: Load Sample Data
- [ ] Paste 12 default categories into Categories_DB
- [ ] Paste 20 default settings into Settings_DB
- [ ] Paste 30 keywords into Keywords_DB

### Step 4: Create Named Ranges
- [ ] Categories_DB[Name]
- [ ] Active_Categories
- [ ] Currency_Symbol
- [ ] Period_Selected
- [ ] Monthly_Income
- [ ] Budget_Method
- [ ] Keywords_DB[Keyword]
- [ ] Keywords_DB[Category]

### Step 5: Apply Formatting
- [ ] Column widths (see specs above)
- [ ] Alternating row colors (White / Lt Blue)
- [ ] Header row formatting
- [ ] Conditional formatting (Categories_DB archived rows)

### Step 6: Data Validation
- [ ] Categories_DB: Type dropdown (Income/Expense/Savings/Transfer)
- [ ] Categories_DB: Color dropdown (15 colors)
- [ ] Categories_DB: Status dropdown (Active/Archived)
- [ ] Settings_DB: Data type validation per setting
- [ ] Keywords_DB: Category dropdown (from Categories_DB[Name])

### Step 7: Sheet Protection
- [ ] Protect all three sheets
- [ ] Allow: Filter, Sort
- [ ] No password (user-accessible)

### Step 8: Testing
- [ ] Run all 15 acceptance tests
- [ ] Verify named ranges resolve correctly
- [ ] Test adding new category
- [ ] Test archiving category
- [ ] Test updating settings

---

## 🎯 Success Criteria

**Phase 1 is complete when:**
1. All 3 database sheets exist and are hidden
2. All sample data loaded (12 categories, 20 settings, 30 keywords)
3. All named ranges created and functional
4. All 15 acceptance tests pass
5. No #REF! or #VALUE! errors anywhere
6. Settings can be updated and named ranges reflect changes
7. Categories can be added/archived
8. Keywords match descriptions correctly

---

## 📞 Next Phase

**Phase 2: Transaction System**
- Depends on: Categories_DB[Name], Currency_Symbol, Date_Format
- Builds: Transaction_DB + Quick Entry sheet
- Estimated: 3 hours

Ready to proceed when all Phase 1 tests pass! ✅
