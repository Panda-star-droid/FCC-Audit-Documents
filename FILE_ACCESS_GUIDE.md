# 🤖 AI Assistant File Access Guide
**For Claude and other AI assistants working on this project**

**Last Updated:** 2025-10-14  
**Purpose:** Solve file access errors and ensure reliable file operations

---

## 🎯 Quick Reference - File Paths That Work

### ✅ ALWAYS Use This Format
```
D:\Business Ideas\Excel Sheets\Personal Finance\filename.md
```

**Key Rules:**
1. Use backslashes (`\`) not forward slashes (`/`)
2. Start with drive letter (`D:`)
3. NO quotes around the path
4. NO `/mnt/` prefix
5. Exact spacing in "Business Ideas" and "Excel Sheets"

---

## 📂 Current File Structure

**All files are now in the root directory** (BUILD_PHASES folder removed as of 2025-10-14)

```
D:\Business Ideas\Excel Sheets\Personal Finance\
│
├── 00_BUILD_PROGRESS.md
├── 01_Master_Index.md
├── 02_Design_System.md
├── 03_Formula_Patterns.md
│
├── Phase_01_Foundation.md
├── Phase_02_Transaction_System.md
├── Phase_03_Utilities.md
├── Phase_04_Income_Sheet.md
├── Phase_05_Budget_System.md
├── Phase_06_Debt_System.md
├── Phase_07_NetWorth_System.md
├── Phase_08_Dashboard.md
│
├── PROJECT_SUMMARY.md
├── README.md
├── AUDIT_TRACKER.md
├── AUDIT_COMPLETE_SUMMARY.md
├── ISSUE_6_VALIDATION.md
│
└── AI_ASSISTANT_GUIDES\
    └── (helper documents)
```

---

## 🔧 File Access Tools - How to Use

### Tool 1: `Filesystem:read_file` (Single File)

**When to use:** Reading one file at a time

```xml
<invoke name="Filesystem:read_file">
<parameter name="path">D:\Business Ideas\Excel Sheets\Personal Finance\Phase_01_Foundation.md