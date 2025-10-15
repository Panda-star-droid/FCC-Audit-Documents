# Phase 2: Transaction System
**Deliverables:** Transaction_DB, Quick Entry sheet  
**Estimated Time:** 3 hours  
**Dependencies:** Phase 1 (Categories, Settings, Keywords)

---

## 🎯 Deliverables

1. **Transaction_DB** - Complete transaction history (income/expenses/transfers)
2. **Quick Entry** - User-facing sheet with extended transaction forms

---

## 🔗 Dependencies

**Required from Phase 1:**
- `Categories_DB[Name]` - Category dropdown lists
- `Settings_DB!Currency_Symbol` - Display formatting
- `Settings_DB!Date_Format` - Date formatting
- `Keywords_DB` - Auto-categorization (used in Phase 6 CSV Import)

**Provides to future phases:**
- `Transaction_DB` - All income/expense data for Dashboard, Budget, Debt calculations

---

## 📊 Database Schema

### Transaction_DB (Hidden Sheet)

**Purpose:** Master transaction log - single source of truth for all money movement

**Columns:** A-L (12 columns)

| Col | Name | Type | Example | Validation | Notes |
|-----|------|------|---------|------------|-------|
| A | Date | Date | 2025-10-11 | >=2000-01-01 | Transaction date |
| B | Month_ID | TEXT | "2025-10" | Auto-formula | YYYY-MM format for filtering |
| C | Type | Text | "Expense" | Income/Expense/Transfer | Transaction type |
| D | Description | Text (200 char) | "Starbucks Coffee" | Any text | Merchant/note |
| E | Category | Text | "Dining Out" | From Categories_DB | Must be active |
| F | Amount | Number | -45.75 | Required | Expenses negative, income positive |
| G | Account | Text | "Checking" | Optional | Source account |
| H | From | Text | "Checking" | Optional | Transfer source |
| I | To | Text | "Savings" | Optional | Transfer destination |
| J | Timestamp | DateTime | 2025-10-11 14:30 | Auto: NOW() | When created |
| K | Deleted? | Boolean | FALSE | TRUE/FALSE | Soft delete flag |
| L | ID | Text | "ab3f2c1d..." | Auto: GUID | Unique identifier |

**Sample Data (30 transactions - last 30 days):**

See Phase 1's "Sample Data Composition" section in original docs. Generate via formula:
```excel
A2: =TODAY()-RANDBETWEEN(1,30)
B2: =TEXT(A2,"YYYY-MM")
C2: =IF(RAND()>0.9, "Income", "Expense")
D2: =INDEX(Merchants, RANDBETWEEN(1, COUNT(Merchants)))
E2: =INDEX(Active_Categories, RANDBETWEEN(1, COUNT(Active_Categories)))
F2: =IF(C2="Income", RANDBETWEEN(500, 3000), -RANDBETWEEN(5, 200))
```

**Named Ranges:**
```
Transaction_DB[Date]        = Transaction_DB!$A$2:$A$10000
Transaction_DB[Month_ID]    = Transaction_DB!$B$2:$B$10000
Transaction_DB[Type]        = Transaction_DB!$C$2:$C$10000
Transaction_DB[Amount]      = Transaction_DB!$F$2:$F$10000
Transaction_DB[Category]    = Transaction_DB!$E$2:$E$10000
Transaction_DB[Deleted?]    = Transaction_DB!$K$2:$K$10000
Active_Transactions         = FILTER(Transaction_DB, Transaction_DB[Deleted?]=FALSE)
```

**Indexes:**
- Sort by: Date DESC (most recent first)
- Filter: Deleted? = FALSE (default view)
- Index on: Month_ID (for fast period filtering in Dashboard)

---

## 🎨 Layout & Visual Specs

### Transaction_DB (Hidden)

**Same formatting as Phase 1 DBs:**
- Headers: Navy bg, White text, 12pt Bold
- Data rows: Alternating White/Lt Blue, 11pt Regular
- Column widths: Date=80px, Type=80px, Desc=250px, Category=120px, Amount=96px, others 100px
- Sheet protection: Locked, no password
- Visibility: Hidden

**Conditional Formatting:**
```
Rule 1: Deleted rows
  Formula: =$K2=TRUE
  Format: Font Gray, Strikethrough

Rule 2: Income (Amount > 0)
  Formula: =$F2>0
  Format: Font Green #2E7D32, Bold

Rule 3: Expense (Amount < 0)
  Formula: =$F2<0
  Format: Font Red #C62828
```

---

### Quick Entry Sheet (User-Facing)

**Freeze Panes:** Row 6 (keep header + title visible)

**Column Widths:**
```
A: 110px | B: 95px  | C: 120px | D: 80px  | E: 90px  | F: 160px
G: 120px | H: 120px | I: 60px  | J-W: 100px each
```

#### Row 1-6: Title & Instructions

**A1 (Title):**
```
Content: "QUICK ENTRY"
Format: 18pt Bold Navy, Lt Blue bg
Merge: A1:W1
Height: 40px
```

**A3 (Instructions):**
```
Content: "💡 Use TAB to move between fields | Press ENTER on last field to save"
Format: 11pt Regular Dk Gray
Merge: A3:W3
Height: 28px
```

#### Row 8-13: Regular Transaction Form

**Layout:**
```
Row 8 (Labels):
A8: Date | C8: Type | E8: Description | H8: Category | K8: Amount

Row 9 (Inputs):
A9: [Date picker]     Width: 95px
C9: [Type dropdown]    Width: 90px
E9: [Description]      Width: 250px (merge E9:G9)
H9: [Category dropdown] Width: 160px (merge H9:I9)
K9: [Amount]           Width: 96px

Row 11: [Space]

Row 12 (Save Button):
A12: "💾 Tap to Save" OR "Fill required fields..."
Merge: A12:K12
Height: 44px

Row 13 (Status):
A13: "✅ Saved to Oct 2025 - Dining Out" (conditional)
Merge: A13:K13
Height: 24px
```

**Visual Specs (Row 8 Labels):**
```
Font: 11pt Bold Dk Gray
Alignment: Left
Background: White
Height: 28px
```

**Visual Specs (Row 9 Inputs):**
```
Font: 11pt Regular Black
Background: White
Border: 1px solid Md Gray
Height: 32px
Padding: 8px (simulated via cell size)

Focus state:
  Border: 2px solid Navy
  Outline: 2px solid Gold (2px offset)
```

**Visual Specs (Row 12 Save Cell):**
```
Valid state:
  Background: Navy #1A2B4A
  Font: 11pt Bold White
  Border: 2px solid Navy
  Content: "💾 Tap to Save"

Invalid state:
  Background: Lt Gray #F5F5F5
  Font: 11pt Regular Md Gray
  Border: 1px solid Lt Gray
  Content: "Fill required fields..."
```

**Visual Specs (Row 13 Status Cell):**
```
Success:
  Font: 11pt Regular Green #2E7D32
  Content: "✅ Saved to Oct 2025 - Dining Out"
  Auto-clear after 3 seconds

Error:
  Font: 11pt Regular Red #C62828
  Content: "✕ Amount is required" (or specific error)
  Persistent until fixed
```

#### Row 22-35: Pay Debt Form (Always Visible)

**See Phase 4 (Debt System) for detailed specs**

Basic layout here:
```
Row 22: Title "💳 LOG DEBT PAYMENT"
Row 24-28: Date, Debt dropdown, Amount, From Account inputs
Row 29: Current Balance / After Payment display
Row 30: Save button (44px height)
Row 31: Status cell
```

#### Row 37-50: Add New Debt Form (Always Visible)

**See Phase 4 (Debt System) for detailed specs**

Basic layout:
```
Row 37: Title "➕ ADD NEW DEBT"
Row 39-45: Name, Original Balance, Current Balance, APR%, Min Payment, Start Date inputs
Row 46: Save button (44px height)
Row 47: Status cell
```

#### Row 52-100: Recent Transactions Table

**Headers (Row 52):**
```
A52: Date | B52: Description | C52: Amount | D52: Category | E52: Delete?
Format: Navy bg, White text, 12pt Bold, Center-aligned
Height: 32px
```

**Data (Rows 53-102):**
```
Formula shows last 50 transactions (non-deleted, sorted by date DESC)

Conditional Formatting:
- Income rows: Lt Green bg
- Expense rows: White/Lt Blue alternating
- Recently added (last 2s): Yellow flash (via formula)

Delete? column: Checkbox (data validation: TRUE/FALSE)
```

---

## 🔧 Formulas

### Named Ranges to Create

```
QE_Date             = 'Quick Entry'!$A$9
QE_Type             = 'Quick Entry'!$C$9
QE_Description      = 'Quick Entry'!$E$9
QE_Category         = 'Quick Entry'!$H$9
QE_Amount           = 'Quick Entry'!$K$9
QE_SaveBtn          = 'Quick Entry'!$A$12
QE_Status           = 'Quick Entry'!$A$13
QE_Trigger          = 'Quick Entry'!$Z$3 (hidden helper)
```

### Form Validation (Save Cell A12)

```excel
=IF(AND(QE_Date<>"", 
        QE_Category<>"", 
        QE_Amount<>"", 
        ISNUMBER(QE_Amount), 
        QE_Amount<>0),
   "💾 Tap to Save",
   IF(QE_Date="", "Enter date",
      IF(QE_Category="", "Select category",
         IF(OR(QE_Amount="", NOT(ISNUMBER(QE_Amount))), "Enter valid amount",
            IF(QE_Amount=0, "Amount cannot be $0.00", "")))))
```

### Conditional Formatting (Save Cell A12)

**Rule 1: Valid**
```
Formula: =AND(QE_Date<>"", QE_Category<>"", ISNUMBER(QE_Amount), QE_Amount<>0)
Format: Fill #1A2B4A, Font White Bold 11pt
```

**Rule 2: Invalid**
```
Formula: =NOT(AND(QE_Date<>"", QE_Category<>"", ISNUMBER(QE_Amount), QE_Amount<>0))
Format: Fill #F5F5F5, Font #757575 Regular 11pt
```

### Trigger Detection (Hidden Column Z)

**Z1: Edit Counter**
```excel
=IF(QE_SaveBtn<>"", Z1+1, Z1)
```

**Z2: Previous Counter**
```excel
=Z1
```

**Z3: Trigger Detected**
```excel
=Z1<>Z2
```

**Z4: Last Save Timestamp**
```excel
=IF(Z3, NOW(), Z4)
```

### Append to Transaction_DB (Transaction_DB!A2)

```excel
=IF('Quick Entry'!Z3,
   IF('Quick Entry'!QE_Amount<>"",
      {'Quick Entry'!QE_Date,
       TEXT('Quick Entry'!QE_Date,"YYYY-MM"),  /* Month_ID */
       IF('Quick Entry'!QE_Type="", "Expense", 'Quick Entry'!QE_Type),
       'Quick Entry'!QE_Description,
       'Quick Entry'!QE_Category,
       IF('Quick Entry'!QE_Type="Income", 
          ABS('Quick Entry'!QE_Amount), 
          -1*ABS('Quick Entry'!QE_Amount)),
       "",  /* Account */
       "",  /* From */
       "",  /* To */
       NOW(),
       FALSE,
       TEXT(RAND()*1E15,"0")},
      ""),
   "")
```

**Note:** This formula appends to row 2. Subsequent rows use similar formula, checking if previous row is filled.

**Better Pattern (Phase 7):** Use array formula or manual row insertion. For MVP, pre-allocate 10,000 rows with IF cascade.

### Auto-Clear Form Fields (A9, C9, E9, H9, K9)

**Option 1: Time-based (2 second delay)**
```excel
=IF(NOW() - Z4 > TIME(0,0,2), "", [User_Input])
```

**Option 2: Immediate clear**
```excel
=IF(Z3, "", [User_Input])
```

**Recommended:** Option 2 for simplicity (immediate clear).

### Status Cell (A13)

```excel
=IF(NOW()-Z4 < TIME(0,0,3),
   "✅ Saved to " & TEXT(QE_Date,"MMM YYYY") & " - " & QE_Category,
   IF(AND(QE_Date<>"", QE_Category<>"", ISNUMBER(QE_Amount)), "Ready", ""))
```

### Recent Transactions Table (A53)

**Date Column (A53):**
```excel
=IFERROR(INDEX(SORT(FILTER(Transaction_DB,
                            Transaction_DB[Deleted?]=FALSE),
                    1, -1),  /* Sort by column 1 (Date) DESC */
                ROW()-52,   /* Row offset */
                1),         /* Date column */
         "")
```

**Description (B53):**
```excel
=IFERROR(INDEX([SortedFilteredData], ROW()-52, 3), "")
```

**Amount (C53):**
```excel
=IFERROR(TEXT(INDEX([SortedFilteredData], ROW()-52, 5), "$#,##0.00"), "")
```

**Conditional Formatting (C53):**
```
Income (Amount > 0):
  Formula: =VALUE(SUBSTITUTE(C53, "$", "")) > 0
  Format: Font Green #2E7D32, Bold

Expense (Amount < 0):
  Formula: =VALUE(SUBSTITUTE(C53, "$", "")) < 0
  Format: Font Red #C62828
```

**Category (D53):**
```excel
=IFERROR(INDEX([SortedFilteredData], ROW()-52, 4), "")
```

**Delete? (E53):**
```
Data Validation: List = TRUE, FALSE
Default: FALSE
```

**Conditional Formatting (Row 53-102):**
```
Recently added (last 2 seconds):
  Formula: =(NOW() - INDEX([TimestampColumn], ROW()-52)) < TIME(0,0,2)
  Format: Background Yellow #FFF9E6, Bold
```

---

## ✅ Acceptance Tests (25 tests)

### Transaction_DB Tests (8)

1. **Sample Data Loaded**
   - ✅ 30 transactions present (or generated)
   - ✅ Date range: Last 30 days from TODAY()
   - ✅ Mix of Income (2-3) and Expense (27-28)
   - ✅ All categories from Categories_DB

2. **Data Types**
   - ✅ Date column: Valid dates only
   - ✅ Amount column: Numbers only
   - ✅ Type column: Income/Expense/Transfer only
   - ✅ Deleted? column: TRUE/FALSE only

3. **ID Uniqueness**
   - ✅ All IDs in column K are unique
   - ✅ IDs are 15-digit strings (GUIDs)

4. **Sign Convention**
   - ✅ Income amounts are positive
   - ✅ Expense amounts are negative
   - ✅ No zero amounts

5. **Category Validation**
   - ✅ All categories exist in Categories_DB[Name]
   - ✅ No orphaned categories
   - ✅ Categories are active (not archived)

6. **Timestamp Accuracy**
   - ✅ Timestamp column populated for all rows
   - ✅ Timestamps are recent (within test period)

6a. **Month_ID Auto-Population**
   - ✅ Month_ID column (B) populated for all rows
   - ✅ Format is "YYYY-MM" (e.g., "2025-10")
   - ✅ Month_ID matches Date column (e.g., Oct 11, 2025 → "2025-10")
   - ✅ Multi-year support: Dec 2025 → "2025-12", Jan 2026 → "2026-01"
   - ✅ Formula: =TEXT(A2,"YYYY-MM") in column B

7. **Soft Delete**
   - Change Deleted? to TRUE for one transaction
   - ✅ Transaction still in DB
   - ✅ Strikethrough formatting applied
   - ✅ Excluded from Active_Transactions range

8. **Named Ranges**
   - ✅ Transaction_DB[Date] resolves to A2:A10000
   - ✅ Transaction_DB[Month_ID] resolves to B2:B10000
   - ✅ Transaction_DB[Type] resolves to C2:C10000
   - ✅ Transaction_DB[Amount] resolves to F2:F10000
   - ✅ Transaction_DB[Category] resolves to E2:E10000
   - ✅ Transaction_DB[Deleted?] resolves to K2:K10000
   - ✅ Active_Transactions excludes deleted rows

### Quick Entry Form Tests (17)

9. **Layout Rendering**
   - ✅ Title visible in row 1
   - ✅ Instructions visible in row 3
   - ✅ Form fields in row 9
   - ✅ Save button in row 12 (44px height)
   - ✅ Status cell in row 13

10. **Form Field Validation**
    - ✅ Date field: Date picker (or manual entry)
    - ✅ Type dropdown: Income/Expense/Transfer
    - ✅ Category dropdown: Shows active categories only
    - ✅ Amount field: Numbers only
    - ✅ Description field: Accepts text (200 char max)

11. **Save Button States**
    - Fill all fields with valid data
    - ✅ Button shows "💾 Tap to Save"
    - ✅ Navy background, White text
    - Clear Amount field
    - ✅ Button shows "Enter valid amount"
    - ✅ Lt Gray background, Md Gray text

12. **Trigger Detection**
    - Fill form with valid data
    - Click Save button (edit cell A12)
    - ✅ Z3 (trigger) changes from FALSE to TRUE
    - ✅ Transaction appends to Transaction_DB
    - ✅ Z4 (timestamp) updates to NOW()

13. **Transaction Append**
    - Add transaction: Date=Today, Category=Dining Out, Amount=4.75
    - ✅ New row appears in Transaction_DB row 2 (or next empty)
    - ✅ Date, Category, Amount match input
    - ✅ Month_ID auto-populated (e.g., "2025-10")
    - ✅ Type defaults to "Expense"
    - ✅ Amount stored as -4.75 (negative)
    - ✅ Deleted? = FALSE
    - ✅ ID generated (15 digits)
    - ✅ Timestamp = NOW()

14. **Status Cell Feedback**
    - After save
    - ✅ Status shows "✅ Saved to Oct 2025 - Dining Out"
    - ✅ Green text, 11pt Regular
    - Wait 3 seconds
    - ✅ Status clears automatically

15. **Form Auto-Clear**
    - After save
    - ✅ All form fields clear (Date, Category, Amount, Description)
    - ✅ User can immediately enter next transaction

16. **Income Transaction**
    - Add transaction: Type=Income, Amount=2500
    - ✅ Amount stored as positive 2500
    - ✅ Type = "Income"

17. **Error Handling: Missing Amount**
    - Fill Date and Category, leave Amount blank
    - ✅ Save button shows "Enter valid amount"
    - ✅ Lt Gray background (invalid state)
    - ✅ Cannot trigger save

18. **Error Handling: Invalid Amount**
    - Enter "forty-five" in Amount field
    - ✅ Save button shows "Enter valid amount"
    - ✅ OR field rejects non-numeric input (data validation)

19. **Error Handling: Future Date**
    - Enter date 1 month in future
    - ✅ Save button shows "⚠️ Date in future. Proceed?"
    - ✅ OR blocking validation prevents future dates

20. **Recent Transactions Table**
    - ✅ Table displays last 50 transactions
    - ✅ Sorted by date DESC (newest first)
    - ✅ Columns: Date, Description, Amount, Category, Delete?
    - ✅ Income rows: Lt Green background
    - ✅ Expense rows: Alternating White/Lt Blue

21. **Delete Checkbox**
    - Check Delete? box for one transaction
    - ✅ Transaction disappears from Recent Transactions table
    - ✅ Transaction still exists in Transaction_DB (Deleted?=TRUE)
    - ✅ Dashboard/Budget exclude this transaction

22. **Recent Transaction Highlight**
    - Add new transaction via form
    - ✅ New row appears at top of Recent Transactions
    - ✅ Yellow background flash (first 2 seconds)
    - ✅ Returns to normal color after 2 seconds

23. **TAB Navigation**
    - Press TAB in Date field
    - ✅ Cursor moves to Type dropdown
    - Press TAB again
    - ✅ Moves to Description
    - Continue TAB through all fields
    - ✅ Ends at Amount field
    - Press ENTER on Amount field
    - ✅ Triggers save (same as clicking Save button)

24. **Multiple Transactions**
    - Add 5 transactions in sequence
    - ✅ All 5 appear in Transaction_DB
    - ✅ All 5 appear in Recent Transactions table
    - ✅ All have unique IDs
    - ✅ All have sequential timestamps

25. **Category Dropdown Population**
    - ✅ Dropdown shows all active categories from Categories_DB
    - Archive one category in Categories_DB
    - ✅ Archived category disappears from dropdown
    - ✅ Existing transactions with archived category still display

---

## 🚧 Known Issues / Blockers

*Document issues here during build.*

**Common issues:**
```
- Array formulas not working in Excel 2016
  → Use helper columns instead of FILTER/SORT

- Circular reference warnings
  → Ensure trigger detection uses proper cascade (Z1→Z2→Z3)

- Named ranges not updating
  → Press Ctrl+Alt+F9 to force full recalculation
```

---

## 📝 Implementation Checklist

### Step 1: Create Transaction_DB
- [ ] Create sheet, hide it
- [ ] Add headers (A1:L1) - includes Month_ID column
- [ ] Load 30 sample transactions
- [ ] Add Month_ID auto-formula in column B: =TEXT(A2,"YYYY-MM")
- [ ] Apply conditional formatting (deleted, income, expense)
- [ ] Set column widths
- [ ] Protect sheet
- [ ] Create index on Month_ID column

### Step 2: Create Named Ranges (Transaction_DB)
- [ ] Transaction_DB[Date]
- [ ] Transaction_DB[Month_ID]
- [ ] Transaction_DB[Type]
- [ ] Transaction_DB[Amount]
- [ ] Transaction_DB[Category]
- [ ] Transaction_DB[Deleted?]
- [ ] Active_Transactions (filtered)

### Step 3: Create Quick Entry Sheet
- [ ] Add title and instructions (rows 1-6)
- [ ] Create form fields (row 8-9)
- [ ] Add Save button (row 12, 44px height)
- [ ] Add Status cell (row 13)
- [ ] Set freeze panes (row 6)
- [ ] Set column widths

### Step 4: Add Form Validation
- [ ] Type dropdown (Income/Expense/Transfer)
- [ ] Category dropdown (from Categories_DB, active only)
- [ ] Amount: Number validation
- [ ] Date: Date validation

### Step 5: Create Formulas
- [ ] Save button display formula (A12)
- [ ] Conditional formatting (A12 - valid/invalid states)
- [ ] Trigger detection (Z1, Z2, Z3)
- [ ] Timestamp tracker (Z4)
- [ ] Append formula (Transaction_DB!A2)
- [ ] Auto-clear formulas (form fields)
- [ ] Status cell formula (A13)

### Step 6: Recent Transactions Table
- [ ] Add headers (row 52)
- [ ] Add FILTER/SORT formula for last 50 transactions
- [ ] Format columns (Date, Description, Amount, Category)
- [ ] Add Delete? checkbox column
- [ ] Conditional formatting (income/expense colors, recent highlight)

### Step 7: Create Named Ranges (Quick Entry)
- [ ] QE_Date, QE_Type, QE_Description, QE_Category, QE_Amount
- [ ] QE_SaveBtn, QE_Status, QE_Trigger

### Step 8: Testing
- [ ] Run all 25 acceptance tests
- [ ] Verify save button changes color
- [ ] Verify TAB navigation works
- [ ] Verify ENTER on Amount triggers save
- [ ] Verify transactions append to DB
- [ ] Verify form clears after save
- [ ] Verify status cell shows success message
- [ ] Verify recent transactions update
- [ ] Verify delete checkbox excludes transactions

---

## 🎯 Success Criteria

**Phase 2 is complete when:**
1. Transaction_DB exists with 30 sample transactions
2. Quick Entry sheet fully functional
3. Save button changes color based on validation
4. Transactions append to Transaction_DB on save
5. Form auto-clears after save
6. Status cell shows success message
7. Recent transactions table displays last 50
8. Delete checkbox soft-deletes transactions
9. All 25 acceptance tests pass
10. No #REF! or #VALUE! errors

---

## 📞 Next Phase

**Phase 3: Budget System**
- Depends on: Transaction_DB (for spending calculations)
- Builds: Budget_Allocations_DB, Goals_DB, Budget sheet
- Estimated: 4 hours

Ready to proceed when all Phase 2 tests pass! ✅
