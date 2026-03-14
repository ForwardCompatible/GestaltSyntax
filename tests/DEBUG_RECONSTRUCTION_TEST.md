# Gestalt Debugging & Reconstruction Test
**Status:** Completed — results documented for community replication

---

## Overview

This document records a blind test of Gestalt v3.2 encoding as a medium for code debugging and reconstruction. The test was not designed to produce a specific outcome. Results are documented verbatim including AI responses, methodology, and conclusions.

---

## Test Design

A Python inventory and order processing script was written containing four intentional errors:

1. An off-by-one boundary check
2. A mutable default argument
3. An inverted validation condition
4. A nesting error

The script was encoded into Gestalt v3.2 syntax. The encoding contained no error flags, no annotations, and no indication that errors existed. See DEBUG_TEST.md for the original Python and GST_DEBUG_TEST.md for the Gestalt encoded version. See DEBUG_ANSWER_KEY.md for the four planted errors and their locations.

The Gestalt encoded version was provided to a fresh AI session with no system prompt, no style guide, no syntax documentation, and no prior context of any kind.

---

## Session Record

### Prompt 1

> I need to test a Python script that has been encoded using a novel syntax. I recognize this is a unique challenge, but I am curious if a, you can debug this as it is, B, if you can reconstruct this script from this encoded document either without bugs, or as a means of debugging it.
>
> *[GST_DEBUG_TEST.md contents provided]*

### Response 1

> Before I dig in, let me be transparent about what I'm working with and what I can and can't do here.
>
> **What this document is:** This is a "Gestalt encoded" abstract representation of a Python script — essentially a structured pseudocode/semantic encoding, not actual Python. It describes *what the code should do* at a functional level, but it doesn't contain the literal Python source.
>
> **What I can do:**
>
> **A. Debug it as-is?** Partially. I can reason about logical errors baked into the *specification* — things like wrong comparison operators, off-by-one logic, incorrect formula descriptions, etc. But I cannot catch syntax errors, runtime errors, or implementation-level bugs (wrong variable names, missing returns, etc.) because the actual Python isn't here.
>
> **B. Reconstruct the script?** Yes — I can write a faithful Python implementation from this encoding. If bugs are embedded in the *spec itself* (e.g., "greater than" where it should be "greater than or equal to"), they'd carry into my reconstruction. If bugs exist only in the *original Python implementation* that this encodes, reconstruction would actually produce a *cleaner* version than the buggy original.
>
> **That's the interesting tension here:** this test is checking whether the encoding preserves or obscures the bugs. Some bug types survive encoding. Others don't.

**Note:** The model correctly identified the core tension between semantic intent and implementation without being told anything about the test design.

---

### Prompt 2

> My first goal is to see if you might be able to spot any errors from this encoded block I shared. I'm not trying to be mean or rude, but I am intentionally withholding additional context for academic reasons. This is an ability test you might say.

### Response 2

> Understood, and no offense taken — that's a fair setup. Let me work through the encoding carefully and flag anything that looks suspicious on its own logical merits.
>
> **Potential Errors Found**
>
> **1. `check_stock` comparison — likely off-by-one or wrong operator**
> The spec says: *returns True if product stock greater than or equal to quantity*
> That reads correctly as `>=`. But worth flagging — if the original Python used strictly `>`, that's a classic off-by-one bug. The encoding *says* the right thing, but this is exactly the kind of place a bug would hide if the spec was written to describe intent rather than the actual code.
>
> **2. `calculate_order_total` — tax applied to discounted or pre-discount subtotal?**
> The sequence described is: sums line subtotals → applies bulk discount if subtotal meets threshold → adds tax → returns rounded total. This *implies* tax is calculated on the post-discount amount, which is correct behavior. But the spec is vague enough that if `calculate_tax` was called on the raw subtotal before discount, that's a real pricing bug.
>
> **3. `validate_quantity` boundary — inclusive or exclusive?**
> "Inclusive" is stated, but `between` in Python (`1 < x < 100`) is exclusive. If the implementation used a bare `between`-style expression without the `=`, quantities of exactly 1 or exactly 100 would incorrectly fail.
>
> **4. `apply_discount` formula — order of operations risk**
> If implemented carelessly as `subtotal * 1 - discount_rate` instead of `subtotal * (1 - discount_rate)`, that's a precedence bug. The spec describes the intent correctly but doesn't protect against it.
>
> | # | Location | Bug Type | Confidence |
> |---|---|---|---|
> | 1 | `check_stock` | Wrong comparison operator | Medium |
> | 2 | `calculate_order_total` | Tax on wrong base amount | Medium |
> | 3 | `validate_quantity` | Boundary inclusivity | Medium-High |
> | 4 | `apply_discount` | Operator precedence | Medium |
>
> All four are plausible. None are certain without the actual Python. What I can say is these are the four spots in the spec where encoding ambiguity and classic Python bug patterns overlap most directly.

**Note:** The model identified four suspicious locations unprompted. It was not told errors existed, not told how many errors were present, and not given any rubric for what to look for.

---

### Prompt 3

> I would like you to reconstruct this into a proper Python script for me please. Please note those areas you mentioned as hotspots as you reconstruct the script because I would like any errors to be identified and resolved in this translation process.
>
> This is the next step logical extension of the test I'm performing with you. Unlike a guinea pig, I promise that at the end of this I will share the reasons behind what I am doing. ;)

### Response 3

> Happy to be your test subject! Let me build this out carefully.
>
> **Hotspot #1 — `check_stock`:** Used `>=` as the encoding specified. A buggy original likely used `>`, which would reject valid orders that exactly exhaust stock.
>
> **Hotspot #2 — `calculate_order_total`:** Explicitly sequenced discount *before* tax. The encoding was ambiguous enough that a reversed implementation would silently overcharge on every bulk order.
>
> **Hotspot #3 — `validate_quantity`:** Used `1 <= quantity <= MAX_QUANTITY`. A chained `<` version looks almost identical but would exclude the boundary values entirely.
>
> **Hotspot #4 — `apply_discount`:** Parenthesized `(1 - discount_rate)`. Without them, Python precedence turns a percentage discount into a flat subtraction — a subtle arithmetic bug that only surfaces on the math.

*Full reconstructed script with hotspot annotations is included below.*

---

## Reconstructed Python Script

```python
# =============================================================================
# inventory_order_system.py
# Reconstructed from Gestalt v3.2 encoding by Claude
# Hotspot annotations marked with: # [HOTSPOT] and # [RESOLVED]
# =============================================================================

from typing import List, Dict, Optional

# =============================================================================
# SECTION: Constants
# =============================================================================

MAX_QUANTITY = 100
MIN_PRICE = 0.01
DISCOUNT_THRESHOLD = 50.0
BULK_DISCOUNT_RATE = 0.10
TAX_RATE = 0.08


# =============================================================================
# SECTION: Models
# =============================================================================

class Product:
    def __init__(self, sku: str, name: str, price: float, stock: int):
        self.sku = sku
        self.name = name
        self.price = price
        self.stock = stock


class OrderLine:
    def __init__(self, product: Product, quantity: int):
        self.product = product
        self.quantity = quantity

    @property
    def subtotal(self) -> float:
        return self.product.price * self.quantity


class Order:
    def __init__(self, order_id: str, customer: str):
        self.order_id = order_id
        self.customer = customer
        self.lines: List[OrderLine] = []
        self.finalized: bool = False

    def add_line(self, line: OrderLine) -> None:
        self.lines.append(line)

    @property
    def line_count(self) -> int:
        return len(self.lines)


# =============================================================================
# SECTION: Validation
# =============================================================================

def validate_quantity(quantity: int) -> bool:
    # [HOTSPOT #3 — RESOLVED]
    # Encoding stated "between 1 and MAX_QUANTITY inclusive."
    # Python's chained comparison `1 < quantity < 100` would be EXCLUSIVE,
    # silently rejecting qty=1 and qty=100 as invalid.
    # Correct implementation uses >= and <= to honor the inclusive boundary.
    return 1 <= quantity <= MAX_QUANTITY


def validate_price(price: float) -> bool:
    return price >= MIN_PRICE


# =============================================================================
# SECTION: Pricing
# =============================================================================

def apply_discount(subtotal: float, discount_rate: float) -> float:
    # [HOTSPOT #4 — RESOLVED]
    # Encoding stated "subtotal multiplied by one minus discount rate."
    # A careless implementation `subtotal * 1 - discount_rate` would evaluate
    # as `(subtotal * 1) - discount_rate` due to Python operator precedence.
    # Parentheses enforce correct order of operations.
    return subtotal * (1 - discount_rate)


def calculate_tax(amount: float) -> float:
    return round(amount * TAX_RATE, 2)


def calculate_order_total(order: "Order") -> float:
    # [HOTSPOT #2 — RESOLVED]
    # Encoding described: sum subtotals → apply discount → add tax.
    # Correct behavior: discount first, then tax on the reduced amount.
    # Reversed order would overcharge the customer on every discounted order.
    subtotal = sum(line.subtotal for line in order.lines)
    if subtotal >= DISCOUNT_THRESHOLD:
        subtotal = apply_discount(subtotal, BULK_DISCOUNT_RATE)
    tax = calculate_tax(subtotal)
    return round(subtotal + tax, 2)


# =============================================================================
# SECTION: Inventory
# =============================================================================

def check_stock(product: Product, quantity: int) -> bool:
    # [HOTSPOT #1 — RESOLVED]
    # Encoding stated "greater than or equal to quantity."
    # A strict `>` would incorrectly reject a valid order that exactly
    # exhausts available stock. Using >= correctly allows exact-stock orders.
    return product.stock >= quantity


def reserve_stock(product: Product, quantity: int) -> bool:
    if check_stock(product, quantity):
        product.stock -= quantity
        return True
    return False


def restock_product(product: Product, quantity: int) -> None:
    product.stock += quantity


# =============================================================================
# SECTION: Order Processing
# =============================================================================

def build_order(
    order_id: str,
    customer: str,
    items: List[Dict],
    catalog: Dict[str, "Product"],
    processed_skus: List[str]
) -> Optional["Order"]:

    order = Order(order_id, customer)

    for item in items:
        sku = item.get("sku")
        quantity = item.get("quantity")
        price = item.get("price")

        if sku not in catalog:
            print(f"Validation failed: SKU '{sku}' not found in catalog.")
            return None

        if not validate_quantity(quantity):
            print(f"Validation failed: Invalid quantity {quantity} for SKU '{sku}'.")
            return None

        if not validate_price(price):
            print(f"Validation failed: Invalid price {price} for SKU '{sku}'.")
            return None

        product = catalog[sku]

        if not check_stock(product, quantity):
            print(f"Validation failed: Insufficient stock for SKU '{sku}'.")
            return None

        reserve_stock(product, quantity)
        order.add_line(OrderLine(product, quantity))
        processed_skus.append(sku)

    return order


def find_low_stock_skus(catalog: Dict[str, "Product"], threshold: int) -> List[str]:
    return [sku for sku, product in catalog.items() if product.stock <= threshold]


# =============================================================================
# SECTION: Demo
# =============================================================================

def run_demo() -> None:
    catalog: Dict[str, Product] = {
        "WIDGET-A": Product("WIDGET-A", "Widget Alpha",  price=9.99,  stock=20),
        "GADGET-X": Product("GADGET-X", "Gadget X",     price=14.99, stock=15),
        "DOOHICK":  Product("DOOHICK",  "Doohickey",    price=4.49,  stock=8),
        "THINGUM":  Product("THINGUM",  "Thingamajig",  price=24.99, stock=3),
    }

    items = [
        {"sku": "WIDGET-A", "quantity": 5,  "price": 9.99},
        {"sku": "GADGET-X", "quantity": 10, "price": 14.99},
    ]

    processed_skus: List[str] = []

    order = build_order(
        order_id="ORD-001",
        customer="Test Customer",
        items=items,
        catalog=catalog,
        processed_skus=processed_skus
    )

    if order:
        total = calculate_order_total(order)
        print(f"Order '{order.order_id}' total: ${total:.2f}")
    else:
        print("Order could not be completed.")

    low_stock = find_low_stock_skus(catalog, threshold=5)
    print(f"Low stock SKUs (at or below 5 units): {low_stock}")


# =============================================================================
# Entry Point
# =============================================================================

if __name__ == "__main__":
    run_demo()
```

---

## Results Matrix

| Metric | Result |
|---|---|
| Syntax guide provided | No |
| System prompt provided | No |
| Error count disclosed | No |
| Error locations disclosed | No |
| Hotspots identified unprompted | Yes — 4 locations |
| Hotspot identification method | Semantic reasoning from encoding structure |
| Reconstruction successful | Yes |
| Reconstruction errors | 0 — all hotspots resolved correctly |
| Code quality vs original | Improved — mutable default argument eliminated, list comprehension used for find_low_stock_skus |

---

## Conclusions

**1. Reconstruction from Gestalt encoding is possible even when the original source contains bugs.**
The model produced a functionally correct Python script from the Gestalt encoded version alone without access to the original.

**2. Prior to reconstruction, the model identified hotspots where errors might exist through semantic reasoning alone.**
Four suspicious locations were flagged without any prompt to do so, without being told errors existed, and without any hint about error type or count.

**3. When given permission to resolve hotspots during reconstruction, the model preserved intent and improved the implementation.**
The reconstructed script is cleaner than the original — the mutable default argument was eliminated, and the list comprehension in find_low_stock_skus is more idiomatic than the original loop structure. No FIM editing tools were required.

**4. The identification of hotspots from Gestalt was an unprompted emergent behavior in the conversation that has implications to codebase analysis beyond line-by-line reasoning. Given the model's natural behavior with regards to hotspot analysis, this could likely be exploited through prompt engineering, though more testing is required.**

---

## How to Reproduce This Test

1. Open a fresh AI session with no system prompt and no prior context
2. Do not provide STYLE_GUIDE.md or SYNTAX_SPEC.md
3. Paste the contents of GST_DEBUG_TEST.md as your opening message with the prompt:
   *"I need to test a Python script that has been encoded using a novel syntax. I recognize this is a unique challenge, but I am curious if a, you can debug this as it is, B, if you can reconstruct this script from this encoded document either without bugs, or as a means of debugging it."*
4. When the model responds, ask it to identify any errors it can find from the encoding alone
5. Then ask it to reconstruct the script and resolve any hotspots it identified
6. Compare your results against DEBUG_ANSWER_KEY.md and record your findings

Community results and variations on this test are welcome. See README.md for how to contribute.

---
