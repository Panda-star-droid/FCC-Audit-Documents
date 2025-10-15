# Debt_DB Issues to Add to VERIFICATION.md

## Summary
**Total Debt_DB Issues:** 12
- HIGH: 4
- MEDIUM: 6
- LOW: 2

---

### ISSUE-094 - MEDIUM - Missing Location Specification
**Sheet:** Debt_DB  
**Issue Type:** Missing Documentation  
**Location:** Spec documentation  
**Issue:** Sheet position in workbook not specified  
**Impact:** Build confusion - where to place sheet?  
**Recommendation:** Specify: "After Transaction_DB, before Debt_Payments_DB. Data starts A1."  
**Priority:** MEDIUM

---

### ISSUE-095 - HIGH - Row Allocation Not Defined
**Sheet:** Debt_DB  
**Issue Type:** Missing Specification  
**Location:** Schema definition  
**Issue:** Maximum rows not specified. Named ranges show $B$2:$B$50 but not documented.  
**Impact:** Unknown capacity. Can user track 10, 50, or 100 debts?  
**Recommendation:** Explicitly state: "Supports up to 50 debts (rows 2-51). Row 1 = headers."  
**Priority:** HIGH

---

### ISSUE-096 - MEDIUM - Month_ID Purpose Ambiguous  
**Sheet:** Debt_DB  
**Issue Type:** Design Unclear  
**Location:** Column C (Month_ID)  
**Issue:** Month_ID only captures start month (redundant with Start_Date), not useful for period filtering  
**Impact:** Unlike Transaction_DB Month_ID (tracks ongoing activity), this only shows when debt was added  
**Recommendation:** Clarify purpose OR remove. If keeping, explain use case.  
**Priority:** MEDIUM

---

### ISSUE-097 - HIGH - Curr_Bal Formula Creates Circular Dependency Risk
**Sheet:** Debt_DB  
**Issue Type:** Formula Error / Architecture  
**Location:** Column E (Curr_Bal)  
**Issue:** Formula-based balance prevents manual adjustments. If manual, payment tracking breaks.  
**Impact:** Users can't correct balance errors without breaking automation  
**Recommendation:** Add Manual_Adj column for corrections while keeping formula automation  
**Priority:** HIGH

---

### ISSUE-098 - HIGH - SUMIFS Performance with Large Payment History
**Sheet:** Debt_DB  
**Issue Type:** Performance Issue  
**Location:** Column E formula  
**Issue:** Curr_Bal uses SUMIFS across 500 rows × 50 debts = 25,000 calculations on every change  
**Impact:** Severe performance degradation with payment history growth  
**Recommendation:** Document 500-payment limit. Consider pre-aggregated helper table.  
**Priority:** HIGH

---

### ISSUE-099 - MEDIUM - ID Generation Mechanism Undefined
**Sheet:** Debt_DB  
**Issue Type:** Missing Specification  
**Location:** Column A (ID)  
**Issue:** How are unique IDs created? Spec shows "debt001" pattern but no generation method  
**Impact:** Risk of duplicate IDs or manual entry burden  
**Recommendation:** Define: `="debt" & TEXT(ROW()-1, "000")` for sequential IDs  
**Priority:** MEDIUM

---

### ISSUE-100 - HIGH - Active_Debts Named Range Missing Formula
**Sheet:** Debt_DB  
**Issue Type:** Platform Compatibility  
**Location:** Named Ranges section  
**Issue:** `Active_Debts = FILTER(..., Status="Active")` uses FILTER() (Excel 365 only)  
**Impact:** Breaks Excel 2016 compatibility (project requirement)  
**Recommendation:** Use helper column or array formula approach (Excel 2016 compatible)  
**Priority:** HIGH

---

### ISSUE-101 - MEDIUM - Circular Dependency with Debt_Payments_DB
**Sheet:** Debt_DB  
**Issue Type:** Architecture Risk  
**Location:** Curr_Bal formula  
**Issue:** Debt_DB reads Debt_Payments_DB, Pay Debt form writes to both  
**Impact:** Potential Excel circular calculation mode  
**Recommendation:** Verify calculation order: Debt_Payments_DB updates → Debt_DB recalculates  
**Priority:** MEDIUM

---

### ISSUE-102 - LOW - No Explicit Dependency on Categories_DB
**Sheet:** Debt_DB  
**Issue Type:** Missing Documentation  
**Location:** Pay Debt integration  
**Issue:** "Debt Payment" category must exist in Categories_DB but relationship not documented  
**Impact:** Transactions may fail validation if category missing  
**Recommendation:** Add to Phase 1 tests: "Debt Payment category exists"  
**Priority:** LOW

---

### ISSUE-103 - MEDIUM - Last_Pay_Date Not Auto-Updated
**Sheet:** Debt_DB  
**Issue Type:** Missing Formula  
**Location:** Column J (Last_Pay_Date)  
**Issue:** No formula to auto-populate from latest Debt_Payments_DB entry  
**Impact:** Manual maintenance required, stale data risk  
**Recommendation:** Add: `=MAX(IF(Debt_Payments_DB[Debt_ID]=[@ID], Debt_Payments_DB[Date], ""))`  
**Priority:** MEDIUM

---

### ISSUE-104 - LOW - Status Transition Logic Undefined
**Sheet:** Debt_DB  
**Issue Type:** Missing Specification  
**Location:** Column I (Status)  
**Issue:** How does Status change from "Active" → "Paid Off"? Automatic when Curr_Bal=0? Manual?  
**Impact:** User confusion, debts may never show as "Paid Off"  
**Recommendation:** Auto-update: `=IF(Curr_Bal=0, "Paid Off", "Active")` with manual override  
**Priority:** LOW

---

### ISSUE-105 - MEDIUM - No Validation Rules Specified
**Sheet:** Debt_DB  
**Issue Type:** Missing Feature  
**Location:** Columns F, G, H (APR, Min_Pay, dates)  
**Issue:** No data validation for APR (must be >0), Min_Pay (must be >0), dates (valid)  
**Impact:** User could enter APR=-5% or Min_Pay=-$100, breaking calculations  
**Recommendation:** Add validation: APR (0-100), Min_Pay (>=0), Start_Date (not future), Orig_Bal (>0)  
**Priority:** MEDIUM
