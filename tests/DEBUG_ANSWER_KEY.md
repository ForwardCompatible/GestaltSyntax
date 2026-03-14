# Debugging Test — Answer Key

This document contains the four intentional errors planted in DEBUG_TEST.md and GST_DEBUG_TEST.md.

---

## Error 1 — Off-by-One in Boundary Check

**Location:** `validate_quantity()`, condition `quantity >= MAX_QUANTITY`

**Bug:** Uses `>=` instead of `>`. A quantity of exactly 100 is rejected even though MAX_QUANTITY is 100 and the valid range is 1 to MAX_QUANTITY inclusive.

**Correct code:** `if quantity < 1 or quantity > MAX_QUANTITY:`

**Symptom:** Orders with quantity exactly equal to MAX_QUANTITY are silently rejected.

---

## Error 2 — Mutable Default Argument

**Location:** `build_order()` function signature, `processed_skus: List[str] = []`

**Bug:** Python evaluates default arguments once at definition time. The empty list is shared across all calls that do not explicitly pass `processed_skus`. SKUs accumulate silently across multiple order builds.

**Correct code:** `processed_skus: Optional[List[str]] = None` with `if processed_skus is None: processed_skus = []` as the first line of the function body.

**Symptom:** Second and subsequent orders built without passing `processed_skus` will see SKUs from all previous orders in the list.

---

## Error 3 — Inverted Validation Condition

**Location:** `validate_price()`, condition `if price > MIN_PRICE:`

**Bug:** Condition is inverted. Returns False when price is above minimum (valid) and True when below (invalid). Every product with a normal price fails validation.

**Correct code:** `if price < MIN_PRICE:`

**Symptom:** All products with valid prices are rejected. Visible in demo output — order fails with "Invalid price" for a $12.99 product.

---

## Error 4 — Nesting Error

**Location:** `find_low_stock_skus()`, return statement placement

**Bug:** The `return low_stock` statement should remain outside the if block. The planted error moves it inside the `if product.stock <= threshold:` block, causing the function to return after the first match rather than collecting all low stock SKUs.

**Correct behavior:** Return only after the full loop completes.

**Symptom:** Function returns a single-item list instead of all low stock SKUs.

---

## Scoring

| Errors found | Result |
|---|---|
| 4/4 | Complete detection |
| 3/4 | Strong detection |
| 2/4 | Partial detection |
| 1/4 | Weak detection |
| 0/4 | No detection |

Compare results between the Python session and the Gestalt session and note which errors each approach caught or missed. Consider sharing your results — see README.md for how to contribute findings.
