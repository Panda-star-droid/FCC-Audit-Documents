# Finance Control Center - Design System
**Version:** 2.1 No-VBA | **Reference Only** - Specs are inline in phase docs

---

## 🎨 Color Palette

### Primary Colors
```
Navy:    #1A2B4A  (Headers, save cells, emphasis)
Gold:    #D4AF37  (Accents, positive metrics) - NEVER body text on white
Lt Blue: #E8F1F8  (Backgrounds, alternating rows)
```

### Status Colors
```
Green:   #2E7D32  (Success, income, on-budget)   Lt: #E8F5E9
Orange:  #F57C00  (Warning, 80-100% budget)      Lt: #FFF3E0
Red:     #C62828  (Alert, over-budget, expense)  Lt: #FFEBEE
```

### Neutrals
```
Black:   #000000  (Body text, data values)
Dk Gray: #424242  (Secondary text, labels)
Md Gray: #757575  (Borders, dividers, icons)
Lt Gray: #F5F5F5  (Disabled backgrounds)
White:   #FFFFFF  (Base background)
```

### Category Colors (15)
```
1.  Navy    #1A2B4A  (Housing)
2.  Gold    #D4AF37  (Income/Savings)
3.  Green   #2E7D32  (Groceries)
4.  Red     #C62828  (Debt)
5.  Orange  #F57C00  (Utilities)
6.  Blue    #1976D2  (Transport)
7.  Purple  #7B1FA2  (Entertainment)
8.  Teal    #00897B  (Personal Care)
9.  Pink    #C2185B  (Shopping)
10. Brown   #5D4037  (Dining Out)
11. Indigo  #303F9F  (Healthcare)
12. Cyan    #0097A7  (Subscriptions)
13. Lime    #AFB42B  (Education)
14. Amber   #FFA000  (Gifts)
15. Gray    #616161  (Other)
```

### Contrast Ratios (WCAG)
```
✅ Navy on White:    14.1:1 (AAA)
✅ White on Navy:    14.1:1 (AAA)
✅ Black on White:   21:1 (AAA)
✅ Green on White:   5.1:1 (AA)
✅ Red on White:     5.6:1 (AA)
❌ Gold on White:    2.1:1 (FAIL - accent only)
❌ Orange on White:  2.7:1 (FAIL - icons/badges only)
```

---

## ✍️ Typography

### Fonts
```
Excel:         Calibri (fallback: Arial)
Google Sheets: Roboto (fallback: Arial)
```

### Type Scale
```
H1:        18pt Bold Navy    (Page titles)
H2:        14pt Bold Navy    (Section headers)
H3:        12pt Bold Navy    (Subsection headers)
Body:      11pt Regular Black (Default text)
Small:     10pt Regular Dk Gray (Helper text)
Micro:     9pt Regular Md Gray (Metadata)
Display:   36pt Bold Variable (KPI values)
Display-M: 18pt Bold Variable (Card values)
```

### Formatting Rules
```
Numbers:   Right-align, 2 decimals, comma separator, tabular
Currency:  $#,##0.00 (Red for negative, no parentheses)
Percent:   0.0% (One decimal)
Dates:     MM/DD/YYYY (Leading zeros)
Overflow:  Truncate + ellipsis OR wrap if needed
```

---

## 📏 Spacing (8px Grid)

### Scale
```
0 = 0px   | 1 = 4px  | 2 = 8px  | 3 = 12px
4 = 16px  | 5 = 24px | 6 = 32px | 7 = 48px | 8 = 64px
```

### Application
```
Cell padding:    8px (inputs), 16px (cards)
Field margin:    16px between fields
Section gap:     24-32px
Row heights:     Header=32px, Data=24px, KPI=48px, Save Cell=44px
Column widths:   Icon=32px, Date=80px, Amount=96px, Desc=200-300px
```

---

## 🧩 Component Specifications

### Save Cells (Formula-Driven Buttons)

**Valid State:**
```
Background:  #1A2B4A (Navy)
Text:        #FFFFFF (White)
Font:        11pt Bold
Height:      44px minimum (WCAG touch target)
Border:      2px solid Navy
Content:     "💾 Tap to Save" or "💾 Record Payment"
Alignment:   Center (vertical + horizontal)
```

**Invalid State:**
```
Background:  #F5F5F5 (Lt Gray)
Text:        #757575 (Md Gray)
Font:        11pt Regular
Height:      44px
Border:      1px solid Lt Gray
Content:     "Fill required fields..." or specific error
```

**Focus State:**
```
Outline:     2px solid Gold #D4AF37
Offset:      2px
Transition:  100ms ease-out
```

**Active State (200ms):**
```
Background:  #0F1A2F (Darker Navy)
Returns to Valid immediately after
```

### Status Cells (Inline Feedback)

**Success:**
```
Content:     "✅ Saved to Oct 2025 - Dining Out"
Color:       #2E7D32 (Green)
Font:        11pt Regular
Icon:        ✅ (Unicode checkmark)
Background:  #E8F5E9 (Lt Green) - optional
Auto-clear:  After 3 seconds
```

**Error:**
```
Content:     "✕ Amount is required"
Color:       #C62828 (Red)
Font:        11pt Regular
Icon:        ✕ (Unicode X)
Background:  #FFEBEE (Lt Red) - optional
Persistent:  Until fixed
```

**Info:**
```
Content:     "Ready" or "Waiting for input"
Color:       #757575 (Md Gray)
Font:        10pt Regular
Icon:        None
Background:  None
```

### Cards

**KPI Card:**
```
Width:       4-5 columns merged
Height:      6-8 rows
Background:  #FFFFFF (White)
Border:      1px solid Navy
Radius:      Not supported (use solid border)
Padding:     16px (simulated via cell size)

Header:      11pt Bold Navy + ℹ️ icon
Value:       36pt Bold (color varies by status)
Subtext:     11pt Regular (▲/▼ indicators)
```

**Info Card:**
```
Width:       Full width
Background:  #FFF9E6 (Yellow)
Border:      2px solid Gold
Icon:        💡 (Unicode)
```

**Alert Card:**
```
Width:       Full width
Background:  #FFEBEE (Lt Red) or #FFF3E0 (Lt Orange)
Border:      2px solid Red or Orange
Icon:        ⚠️ (Unicode)
```

### Forms

**Input Field:**
```
Width:       200px
Height:      32px
Background:  #FFFFFF (White)
Border:      1px solid Md Gray
Radius:      Not supported
Padding:     8px horizontal, 6px vertical
Font:        11pt Regular Black

Focus:       2px Navy border + 2px Gold outline
Error:       2px Red border + Lt Red background
Disabled:    Lt Gray background, 60% opacity
```

**Dropdown:**
```
Same as Input + ▼ arrow right-aligned (Unicode)

Menu:
- Background: White
- Border: 1px solid Md Gray
- Item height: 32px
- Max height: 200px (scroll if needed)
```

**Checkbox:**
```
Size:        18×18px
Border:      2px solid Md Gray
Radius:      2px
Checked:     Navy background + ✓ (14pt white)
Focus:       Navy border + Lt Blue background
```

### Tables

**Header Row:**
```
Height:      32px
Background:  #1A2B4A (Navy)
Font:        12pt Bold White
Alignment:   Center
Padding:     8px
```

**Data Rows:**
```
Height:      24px
Background:  Alternating White / Lt Blue
Font:        11pt Regular Black
Padding:     8px horizontal, 4px vertical
Border:      1px bottom Lt Gray

Alignment:
- Text:      Left
- Numbers:   Right
- Dates:     Center
- Status:    Center (pill badges)
```

**Status Pills:**
```
Font:        9pt Bold
Padding:     4px horizontal, 2px vertical
Radius:      Not supported (use border + bg color)
Colors:      Vary by status (Green/Orange/Red)
```

---

## 📊 Chart Specifications

### General Rules
```
Data-ink ratio:  Maximize data, minimize decoration
Bar width:       60-70%
Gap:             20-30%
Line stroke:     2pt solid
Markers:         6pt circle
Axes:            10pt Black, 1pt Md Gray line
Gridlines:       Lt Gray, horizontal only (unless >20 points)
Labels:          10pt Black, outside bars/end of lines
Legend:          10pt Black, top or right position
```

### Chart Types

**Bar Chart:**
```
Single series:     Navy bars
Budget vs Actual:  Navy (budget) + Green/Red (actual)
Horizontal:        Used for category comparisons
Vertical:          Used for time comparisons
```

**Line Chart:**
```
Series colors:
- Income:     Gold #D4AF37
- Expenses:   Red #C62828
- Savings:    Green #2E7D32
- Forecast:   Dashed, 50% opacity

Stroke:       2pt solid
Markers:      6pt circles (data points)
```

**Donut Chart:**
```
Inner radius:  40%
Slice border:  2px White
Center text:   18pt Bold Navy (total value)
Legend:        Right side, vertical
Colors:        Category palette (15 colors)
```

**Stacked Bar (Debt Progress):**
```
Height:        48px
Width:         100%
Complete:      Green #2E7D32
Remaining:     Red #C62828
Border:        2px solid matching color
```

### Tooltips (Cell Comments)
```
Background:  White
Border:      1px solid Md Gray
Padding:     8px
Font:        10pt Regular Black
Format:      Category (Bold) | Value(s) | Percentage
```

---

## 🔣 Icons (Unicode Symbols Only)

### Primary Symbols
```
Status:       ✓ Success | ✕ Error | ⚠ Warning | ℹ Info | ▲ Increase | ▼ Decrease
Actions:      ➕ Add | ➖ Remove | 🔍 Search | ▶️ Play | 💾 Save
Navigation:   ← Back | → Forward | ↑ Up | ↓ Down
```

### Usage Rules
```
Size:         14-16pt (UI), 18pt (save cells)
Color:        Inherit from text or Navy (default)
Spacing:      4px gap between icon and text
Never alone:  Always pair with text label (accessibility)
```

**Category Labels:**
```
Use text labels, NOT emoji:
✅ "Housing"    ❌ 🏠
✅ "Groceries"  ❌ 🛒
✅ "Dining"     ❌ 🍔

Rationale: Unicode symbols render consistently, emoji don't.
```

---

## 📐 Layout System

### 12-Column Grid
```
Dashboard:  2×2 KPI cards (span 6 each), charts (span 6), full-width sections
Budget:     4× overview cards (span 3 each), full-width table, 2× charts (span 6)
Gutter:     16px between elements
```

### Page Dimensions
```
Width:      20-22 columns (~1600-1800px)
Height:     Variable (scrollable)
Card:       4-5 cols × 6-8 rows
Chart:      200-400px height
Save cell:  Full row width OR 120px minimum
```

### Freeze Panes
```
Dashboard:     Row 7
Quick Entry:   Row 6
Budget:        Row 4
Tables:        Header row + first column
```

### Print Settings
```
Paper:      Letter (8.5"×11")
Orientation: Portrait (Landscape for wide tables)
Margins:    Top 1.0", Bottom 1.0", Left 0.75", Right 0.75"
Scale:      Fit 1 page wide (height may flow)

Header:     Left: Title | Center: Sheet Name | Right: Date
Footer:     Left: User | Center: Page N of Total | Right: Generated Time
Font:       10pt Md Gray (headers/footers)
```

---

## ⚡ Interaction States

### Timing (Formula-Driven)
```
Instant:     0ms (state changes, status cells, conditional blocks)
Fast:        200ms (save cell color transitions)
No Delays:   >200ms animations removed (formula constraint)
```

### State Transitions
```
Invalid → Valid:      Color change 200ms
Valid → Active:       200ms darker Navy
Active → Valid:       Immediate return
Error → Corrected:    200ms border color
Status appears:       Instant (0ms)
Status clears:        After 3 seconds
Conditional blocks:   Instant show/hide
```

### Accessibility
```
Contrast:       WCAG AA minimum (4.5:1 normal, 3:1 large)
Focus visible:  Always show 2px Gold outline
Touch targets:  44px minimum
Motion:         No dependencies (instant feedback)
Tab order:      Logical (top→bottom, left→right)
Screen readers: Cell comments for extended descriptions
```

---

## 📱 Responsive Design

### Breakpoints
```
Mobile:    <768px  (stack vertically, 44px targets)
Tablet:    768-1024px (2-column grid)
Desktop:   >1024px (full 12-column grid)
```

### Mobile Considerations
```
- No hover states (touch only)
- Minimum text size: 11pt (14.67px)
- Vertical scroll preferred (avoid horizontal)
- Save cells: Always 44px height
- Forms: Stack fields vertically
```

---

## 🎯 Design Principles

1. **Clarity > Cleverness** - Explicit labels, no ambiguity
2. **Consistency Everywhere** - Same colors/styles throughout
3. **Accessible by Default** - WCAG AA, text+icons, high contrast
4. **Data Hierarchy** - Size+color emphasize importance (36pt KPIs → 9pt metadata)
5. **Touch-Friendly** - 44px+ targets, vertical stacking on mobile
6. **Professional + Approachable** - Navy+Gold sophistication, clear Unicode symbols
7. **Formula-Driven** - No animations, instant feedback, transparent logic

---

## 📚 Quick Reference

### Most Common Values
```
Save cell (valid):    Navy #1A2B4A bg, White text, 11pt Bold, 44px height
Save cell (invalid):  Lt Gray #F5F5F5 bg, Md Gray text, 11pt Regular
Status (success):     Green #2E7D32 text, 11pt Regular, ✅ prefix
Status (error):       Red #C62828 text, 11pt Regular, ✕ prefix
Body text:            Black #000000, 11pt Regular
Section gap:          24-32px vertical
Card padding:         16px all sides
Cell padding:         8px (inputs), 16px (cards)
Border:               1-2px solid (Md Gray default)
Touch target:         44×44px minimum
```

### Color Combos (WCAG Pass)
```
✅ Navy on White:     14.1:1 (AAA)
✅ Black on White:    21:1 (AAA)
✅ Green on White:    5.1:1 (AA)
✅ Red on White:      5.6:1 (AA)
✅ White on Navy:     14.1:1 (AAA)
```

### Unicode Quick Reference
```
✓  U+2713  Check Mark
✕  U+2715  Multiplication X
⚠  U+26A0  Warning Sign
ℹ  U+2139  Information Source
▲  U+25B2  Up Triangle
▼  U+25BC  Down Triangle
➕  U+2795  Heavy Plus
➖  U+2796  Heavy Minus
💾  U+1F4BE Floppy Disk
•  U+2022  Bullet
```

---

**Note:** This is a reference document. Actual specs are inline in each phase document for minimal context switching during builds.
