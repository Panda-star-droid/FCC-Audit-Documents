# Phase 3: Utilities/Settings
**Deliverables:** Settings, CSV Import, Instructions sheets  
**Estimated Time:** 4 hours  
**Dependencies:** Phase 1 (Foundation) only
**Critical:** Must be built as Phase 3 - provides Currency, Date_Format for later phases

---

## 🎯 Deliverables

1. **Settings** - Preferences, category management, data management
2. **CSV Import** - Paste-and-normalize workflow for bank data
3. **Instructions** - Video tour, quick start, FAQ, troubleshooting

---

## 🔗 Dependencies

**Uses from Phase 1 (Foundation):**
- `Settings_DB` (for storing preferences)
- `Categories_DB` (for category management UI)
- `Keywords_DB` (for CSV auto-categorization)

**Uses from Phase 2 (Transactions):**
- `Transaction_DB` (for CSV import target)

**Provides to ALL later phases (4-8):**
- Global settings (currency, date format)
- Category management interface
- Data reset/archive capabilities
- CSV import for bulk transaction entry

**DESIGN NOTE:**  
Settings sheet handles system-wide preferences (currency, date format, categories, data management).  
Budget-specific settings (Budget Method, Monthly Income) are configured on the Budget sheet (Phase 5).  
Income-specific settings (Tracking Mode, Sources) are configured on the Income sheet (Phase 4).  
This "configure where you use it" approach provides better workflow and context.

---

## 📊 Settings Sheet Layout

**Freeze:** Row 2

**Column Widths:**
```
A: 200px (Label) | B: 150px (Input) | C: 100px (Action/Help)
```

### Row 1: Title

**A1:**
```
"SETTINGS" - 18pt Bold Navy, Lt Blue bg, merge A1:P1
```

### Row 3-15: Currency & Locale

**A3: Section Title**
```
"📍 CURRENCY & LOCALE" - 14pt Bold Navy
```

**A5-B7: Inputs**
```
A5: Currency Symbol:    [$ ▼] (USD, EUR, GBP, JPY, etc. - 50+ options)
A6: Date Format:        [MM/DD/YYYY ▼] (MM/DD/YYYY, DD/MM/YYYY, YYYY-MM-DD)
A7: First Day of Week:  [Sunday ▼] ℹ️

ℹ️ Tooltip: "Week view calculates from this day.
            Sunday (US) | Monday (Europe) | Saturday (Middle East)"
```

**A9: [💾 Save Locale Settings] Button**
```
44px height, Navy bg when inputs changed, Lt Gray when unchanged
Saves: Currency, Date Format, First Day of Week
```

### Row 17-75: Category Management

**A17: Section Title**
```
"🏷️ CATEGORY MANAGEMENT (Max 30 categories)" - 14pt Bold Navy
```

**A19: Current Count**
```
"Current Categories: 12/30"
```

**A21-C45: Category List**
```
Headers: Category Name | Type | Actions

Row 37: Housing        | Needs    | [Rename] [Delete]
Row 38: Groceries      | Needs    | [Rename] [Delete]
Row 39: Dining Out     | Wants    | [Rename] [Delete]
...
```

**Visual Specs:**
- Category names: 11pt Regular Black
- Type: 10pt Regular Dk Gray
- Actions: Hyperlinks, Navy text
- Each category has color indicator dot (left of name)

**A47: [➕ Add New Category] Button**
```
44px height, Navy bg
Expands form below (A49-B55) or opens in sidebar
```

**Add Category Form (A51-B57):**
```
A51: Name:  [____________________] (30 char max, must be unique)
A52: Type:  [Expense ▼] (Income/Expense/Savings/Transfer)
A53: Color: [Navy ▼] (15 color options)
A54: Icon:  [🏠 ▼] (emoji picker - optional)

A56: [💾 Save Category] [Cancel]
```

**Delete Warning (conditional block):**
```
When [Delete] clicked on category with transactions:

⚠️ Delete 'Dining Out'?
This will reassign 47 existing transactions to 'Uncategorized'.

Type DELETE below to confirm: [____]

[Cancel] [Yes, Delete]
```

### Row 77-95: Data Management

**A77: Section Title**
```
"🗂️ DATA MANAGEMENT" - 14pt Bold Navy
```

**A79: Delete Sample Data**
```
┌────────────────────────────────────────────────┐
│ [🗑️ Delete Sample Data]                       │
│ Removes all sample transactions, debts, and   │
│ snapshots. Keeps: Categories, settings, your  │
│ custom data (if any)                           │
└────────────────────────────────────────────────┘
```

**Confirmation (A81-B83):**
```
A81: Type DELETE to confirm: [____]
A82: [Cancel] [Yes, Delete Sample Data]
```

**A85: Reset & Start Fresh**
```
┌────────────────────────────────────────────────┐
│ [🔄 Reset & Start Fresh]                      │
│ Deletes ALL data (sample + yours) and         │
│ restarts setup. Keeps: Settings (currency,    │
│ date, categories)                              │
└────────────────────────────────────────────────┘
```

**Confirmation (A87-B93):**
```
A87: Type RESET to confirm: [____]
A88: [Cancel] [Yes, Reset Everything]

Confirmation expands:
"⚠️ Reset Everything?
This deletes:
• All sample data and test entries
• All your real transactions (if any)
• Budget history, debt records, net worth snapshots

This keeps:
• Currency, date format, week start settings
• Your category list and colors

After reset, a 3-step setup guide will walk you through.

Type 'RESET' above to confirm."
```

**A90: Archive Old Transactions**
```
┌────────────────────────────────────────────────┐
│ [📦 Archive Old Transactions]                 │
│ Moves transactions older than 1 year to       │
│ Archive_DB. Reduces file size and improves    │
│ performance.                                   │
└────────────────────────────────────────────────┘
```

**A92: Export to CSV**
```
┌────────────────────────────────────────────────┐
│ [📥 Export to CSV]                            │
│ Downloads all transactions as CSV file for    │
│ backup/analysis.                               │
└────────────────────────────────────────────────┘
```

**Settings sheet ends at Row 95.**

---

## 📊 CSV Import Sheet Layout

*(CSV Import and Instructions sections remain unchanged...)*

---

## 🔧 Key Formulas

*(Formulas remain unchanged...)*

---

## ✅ Acceptance Tests (40 tests)

### Settings Tests (10)

1. **Currency Change**
   - Change currency from USD to EUR
   - Click [Save Locale Settings]
   - ✅ Settings_DB!Currency_Symbol = "€"
   - ✅ All amounts display with € symbol

2. **Date Format Change**
   - Change from MM/DD/YYYY to DD/MM/YYYY
   - ✅ All dates display in new format
   - ✅ CSV Import handles both formats

3. **Week Start Change**
   - Change from Sunday to Monday
   - ✅ Period selector "Week" recalculates
   - ✅ Charts update for week view

4. **Add Category**
   - Click [➕ Add New Category]
   - Fill: "Subscriptions", Expense, Orange, 📺
   - Click [Save]
   - ✅ New category in Categories_DB
   - ✅ Appears in dropdowns (Transaction, Budget)
   - ✅ Category count updates (13/30)

*(Remaining acceptance tests unchanged...)*

---

## 🎯 Success Criteria

Phase 3 complete when all 25 tests pass and:
- Settings changes work globally
- CSV import workflow functional
- Data management actions work safely
- Instructions clear and helpful

---

## 📞 Next Phase

**Phase 4: Income Sheet**  
Depends on: Foundation, Transactions, Settings  
Estimated: 2 hours
