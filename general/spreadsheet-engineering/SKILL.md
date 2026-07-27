---
name: spreadsheet-engineering
description: 
category: general
tags: [spreadsheet-engineering]
---

## When to Use
Build complex spreadsheets: formulas, pivot tables, data validation, automation.

## Power Formulas
```
# VLOOKUP (legacy)
=VLOOKUP(A2, Sheet2!A:B, 2, FALSE)

# INDEX/MATCH (better)
=INDEX(B:B, MATCH(A2, A:A, 0))

# XLOOKUP (modern)
=XLOOKUP(A2, A:A, B:B, "Not found")

# Conditional aggregation
=SUMIFS(C:C, A:A, ">=2024-01-01", B:B, "Sales")

# Array formulas
=FILTER(A2:C100, B2:B100="Active")

# UNIQUE + SORT
=SORT(UNIQUE(A2:A100))
```

## Data Validation
```excel
# Dropdown list
Data > Validation > List: "Option1,Option2,Option3"

# Custom formula
=AND(A1>0, A1<1000)

# Error alert
Data > Validation > Error Alert: "Value must be 1-999"
```

## Pitfalls
- **Circular references**: Break with helper columns
- **Volatile functions**: NOW(), TODAY() recalculate on every change
- **Performance**: Avoid entire-column references in large sheets
- **Named ranges**: Use for readability

## Verification
- Test with sample data
- Check formulas handle edge cases
- Verify pivot tables update correctly