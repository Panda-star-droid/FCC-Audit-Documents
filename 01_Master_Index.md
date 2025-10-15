# Finance Control Center - Master Index
**Version:** 2.1 No-VBA | **Product:** Personal Finance Tracker

---

## 🚨 CRITICAL - FOR AI ASSISTANTS (Claude, etc.)

**IF YOU ARE AN AI ASSISTANT WORKING ON THIS PROJECT:**

📚 **READ `FILE_ACCESS_GUIDE.md` FIRST** before accessing any files in this directory.

That guide contains:
- ✅ Correct file path formats for this Windows directory
- ✅ Tool usage examples (read, edit, create, move files)
- ✅ Common error solutions and troubleshooting
- ✅ Best practices for reliable file operations

**Quick Path Format:**
```
D:\Business Ideas\Excel Sheets\Personal Finance\filename.md
```

**All 8 phase files are in the ROOT directory** (not in BUILD_PHASES subdirectory):
- Phase_01_Foundation.md
- Phase_02_Transaction_System.md
- Phase_03_Utilities.md
- Phase_04_Income_Sheet.md
- Phase_05_Budget_System.md
- Phase_06_Debt_System.md
- Phase_07_NetWorth_System.md
- Phase_08_Dashboard.md

**⚠️ Using incorrect paths will cause file access errors. Read FILE_ACCESS_GUIDE.md to avoid issues.**

---

## 📋 Sheet Overview (19 Total)

### User-Facing Sheets (9)
1. **Instructions** - Video tour, guide, FAQ (first-time setup)
2. **Settings** - Preferences, categories, data management
3. **Income** - Income source tracking and planning (NEW in V11)
4. **Budget** - Budget planning, goals, spending analysis
5. **Dashboard** - Main hub with KPIs, charts (view-only)
6. **Quick Entry** - Extended transaction and debt forms
7. **Debt** - Debt tracking, strategy comparison, scenarios
8. **Net Worth** - Asset/liability tracking with projections
9. **CSV Import** - Paste-and-normalize bank transaction import

### Backend Database Sheets (10 - Hidden)
10. **Transaction_DB** - All income/expense transactions
11. **Debt_DB** - Active and historical debts
12. **Debt_Payments_DB** - Payment history linked to transactions
13. **Net_Worth_Snapshots_DB** - Monthly net worth snapshots
14. **Budget_Allocations_DB** - Budget method and category allocations
15. **Goals_DB** - Savings goals with targets and deadlines
16. **Income_Sources_DB** - Income source tracking (NEW in V11)
17. **Income_Plan_History** - Historical income changes (NEW in V11)
18. **Categories_DB** - Category list with types and colors
19. **Settings_DB** - Global settings and configuration
20. **Keywords_DB** - Keywords for auto-categorization

---

## 🔗 Dependency Graph

```
Phase 1: Foundation (No dependencies)
├── Categories_DB
├── Settings_DB
└── Keywords_DB

Phase 2: Transactions (Depends on Phase 1)
├── Transaction_DB ──┐
└── Quick Entry     ─┤ Reads: Categories_DB, Settings_DB
                     └ Writes: Transaction_DB

Phase 3: Utilities/Settings (Depends on Phase 1)
├── CSV Import ──┐
├── Settings     ┤ Reads: Categories_DB, Settings_DB
└── Instructions ┤ Provides: Budget_Method, Currency, Date_Format
                 └ Writes: Settings_DB, Transaction_DB (CSV import)

Phase 4: Income Sheet (Depends on Phases 1, 2, 3)
├── Income_Sources_DB ──┐
├── Income_Plan_History ┤ Reads: Transaction_DB, Settings_DB
└── Income Sheet        ┤ Provides: Income baseline for Budget
                        └ Writes: Income_Sources_DB, Income_Plan_History

Phase 5: Budget (Depends on Phases 1, 2, 3, 4)
├── Budget_Allocations_DB ──┐
├── Goals_DB               ─┤
└── Budget Sheet            ┤ Reads: Transaction_DB, Income_Sources_DB, Settings_DB
                            └ Writes: Budget_Allocations_DB, Goals_DB

Phase 6: Debt (Depends on Phases 1, 2, 3)
├── Debt_DB ──────────────┐
├── Debt_Payments_DB      ┤
└── Debt Sheet            ┤ Reads: Transaction_DB, Settings_DB
                          └ Writes: Debt_DB, Debt_Payments_DB, Transaction_DB

Phase 7: Net Worth (Depends on Phases 1, 3, 6)
├── Net_Worth_Snapshots_DB ──┐
└── Net Worth Sheet           ┤ Reads: Settings_DB, Debt_DB (for liabilities)
                              └ Writes: Net_Worth_Snapshots_DB

Phase 8: Dashboard (Depends on ALL - final integration)
└── Dashboard ──────┐ Reads: ALL DBs
                    └ Displays: 6-7 charts (Chart 7 conditional, view-only)
```

---

## 📂 Build Order (Sequential)

| Order | Phase | Deliverables | Why This Order |
|-------|-------|--------------|----------------|
| 1 | Foundation | 3 DBs | No dependencies - base for everything |
| 2 | Transactions | 1 DB + 1 sheet | Core system - needed by Budget & Debt |
| 3 | Utilities/Settings | 3 sheets | Provides Budget_Method, Currency, Date_Format configs |
| 4 | Income | 2 DBs + 1 sheet | Provides income baseline needed by Budget system |
| 5 | Budget | 2 DBs + 1 sheet | Needs Transaction_DB, Income data, and Settings |
| 6 | Debt | 2 DBs + 1 sheet | Writes to Transaction_DB (payments) |
| 7 | Net Worth | 1 DB + 1 sheet | Independent tracking - only needs Settings |
| 8 | Dashboard | 1 sheet | View-only integration - pulls from ALL DBs |

---

## 🎯 Critical Named Ranges (100+)

### Global Settings (Used by ALL sheets)
- `Settings_DB!Currency_Symbol` - e.g., "$"
- `Settings_DB!Date_Format` - e.g., "MM/DD/YYYY"
- `Settings_DB!Period_Selected` - "Week" / "Month" / "Quarter" / "Year"
- `Settings_DB!Monthly_Income` - Budget base amount

### Transaction System
- `Transaction_DB[Date]` - Array of all transaction dates
- `Transaction_DB[Amount]` - Array of all amounts
- `Transaction_DB[Category]` - Array of categories
- `Transaction_DB[Deleted?]` - Boolean flags for soft delete
- `Categories_DB[Name]` - List of active categories

### Budget System
- `Budget_Allocations_DB[Category]` - Budget per category
- `Goals_DB[Target]` - Goal target amounts
- `Goals_DB[Current]` - Goal progress (calculated)

### Debt System
- `Debt_DB[Curr_Bal]` - Current debt balances
- `Debt_DB[APR]` - Annual percentage rates
- `Debt_Payments_DB[Amount]` - Payment amounts

### Net Worth
- `Net_Worth_Snapshots_DB[Net_Worth]` - Historical net worth values

### Dashboard
- `Dash_Period` - Cell B2 (period dropdown)
- `Dash_QuickAdd_Amount` - Quick entry amount field
- `Dash_QuickAdd_Category` - Quick entry category dropdown

**Note:** Full named range list documented in each phase document.

---

## 📊 Sheet Coordinates Quick Reference

### Dashboard (Main View - VIEW ONLY)
- A1:D2 - Title + Period selector
- A3:I10 - KPI cards (4-up grid)
- A15:K40 - Charts (6-7 total, Chart 7 conditional on Detailed Income mode)
- A42:I55 - Recent transactions (last 10, read-only)

Note: Dashboard is view-only in V11. No Quick Add forms. All data entry via Quick Entry sheet.

### Quick Entry (Extended Forms)
- A8:W9 - Full transaction form
- A22:S35 - Pay Debt form (always visible)
- A37:S50 - Add New Debt form (always visible)
- A52:I100 - Recent transactions (last 50)

### Budget (Planning)
- B1 - Period selector
- E1 - Budget method dropdown
- A6:S26 - Budget detail table
- A28:K50 - Budget vs Actual chart
- M28:V50 - Spending Mix donut
- A62:S120 - Goals section
- A122:P150 - Custom allocation (if method="Custom")

### Debt (Payoff Planning)
- A12:J25 - Debt list table
- A27:T50 - Strategy comparison + scenarios
- E28 - Strategy dropdown
- E29 - Extra payment input

### Net Worth (Wealth Tracking)
- A3:T60 - Account breakdown by category
- A62:H75 - Projection controls
- A77:H100 - Charts (trend + allocation)

### CSV Import (Bulk Entry)
- A12:H500 - Paste area (blue background)
- I12:N500 - Hidden normalization formulas
- A502:H510 - Column mapping
- A512:H520 - Preview (first 50 rows)
- A524 - Confirm Import button

### Settings (Configuration)
- A5:P15 - Currency & Locale
- A17:P30 - Budget Method
- A32:P90 - Category Management
- A97:P140 - Data Management
- A142:P165 - Setup Wizard

### Instructions (Help)
- A7:D15 - Video tour embed
- A17:D100 - Quick start guide
- A100:D200 - FAQ (15 questions)

---

## 🔧 Formula Patterns Index

See `03_Formula_Patterns.md` for universal templates:

1. **Save Cell Pattern** - All forms use this
2. **Status Cell Pattern** - Success/error feedback
3. **Auto-Clear Pattern** - Form field reset after save
4. **Duplicate Detection** - CSV import de-duplication
5. **Period Filter** - Week/Month/Quarter/Year calculations
6. **Budget Calculation** - 50/30/20, Zero-Based, Custom methods
7. **Debt Scenario** - Snowball/Avalanche payoff calculations
8. **Goal Progress** - Current amount + monthly savings needed

---

## 🎨 Visual Design Reference

See `02_Design_System.md` for:
- Color palette (Navy, Gold, Green, Red, Orange + 15 category colors)
- Typography (fonts, sizes, alignment rules)
- Spacing (8px grid system)
- Component specs (Save Cells, Status Cells, Cards, Forms, Tables, Charts)
- Icons (Unicode symbols only: ✓ ✕ ⚠ ℹ ▲ ▼ ➕)

**Key Visual Rules:**
- Save cells: 44px height (touch target)
- Navy #1A2B4A on White #FFFFFF = 14.1:1 contrast (WCAG AAA)
- Right-align numbers, left-align text, center-align dates
- Currency format: $#,##0.00 (red for negative, no parentheses)

---

## ✅ Acceptance Testing Map

Total: **190 tests** across all phases

| Phase | Tests | Focus Area |
|-------|-------|------------|
| Phase 1 | 15 | Data integrity, settings load/save |
| Phase 2 | 25 | Transaction CRUD, form validation |
| Phase 3 | 40 | CSV import, settings, category management |
| Phase 4 | 20 | Income source tracking, plan history |
| Phase 5 | 35 | Budget calculations, goal tracking |
| Phase 6 | 30 | Debt scenarios, payment logging |
| Phase 7 | 20 | Net worth calculations, projections |
| Phase 8 | 25 | Dashboard integration, KPI accuracy |

**Test execution:** Run after each phase completes. All tests must pass before moving to next phase.

---

## 📱 Platform Compatibility

| Feature | Excel 2016 | Excel 365 | Google Sheets | Mobile |
|---------|------------|-----------|---------------|--------|
| Formulas | ✅ INDEX-MATCH | ✅ XLOOKUP | ✅ Both | ✅ View |
| Charts | ✅ | ✅ | ✅ | ✅ View |
| Conditional Formatting | ✅ | ✅ | ✅ | ✅ |
| Data Validation | ✅ | ✅ | ✅ | ⚠️ Limited |
| Named Ranges | ✅ | ✅ | ✅ | ✅ |
| Cell Comments | ✅ | ✅ | ✅ Notes | ✅ |
| Print Ranges | ✅ | ✅ | ✅ | ❌ |
| **Entry Mode** | ✅ Full | ✅ Full | ✅ Full | ⚠️ View + Basic |

**Target:** 100% parity Excel 2016+ and Google Sheets for desktop/web. Mobile is view-optimized with limited editing.

---

## 🚀 Quick Start for Builders

1. **Read:** 00_BUILD_PROGRESS.md (this session's goals)
2. **Build:** Follow phase order 1→8 sequentially
3. **Reference:** Check this index for dependencies and coordinates
4. **Test:** Run acceptance tests after each phase
5. **Track:** Update BUILD_PROGRESS.md after each session

**Need formulas?** → 03_Formula_Patterns.md  
**Need colors/specs?** → 02_Design_System.md  
**Stuck?** → Check phase document's "Known Issues" section

---

## 📞 Documentation Locations

- **Executive Summary:** Revised Build/Doc_0_Executive_Summary_NO_VBA.md
- **Customer Journey:** Revised Build/Doc_1_Customer_Journey_NO_VBA.md
- **Technical Specs:** Revised Build/Doc_2_Technical_Specs_NO_VBA.md + Annex B
- **Visual Design:** Revised Build/Doc_3_Visual_Design_System_NO_VBA.md + Annex A

**This index is your navigation hub.** Bookmark it!
