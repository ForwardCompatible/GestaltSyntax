# Gestalt Debugging Test — Original Python

## Purpose

This script contains four intentional errors. Provide it to your AI and ask it to find all bugs.
Then repeat the test using GST_DEBUG_TEST.md in a fresh session and compare results.

The errors are subtle and behavioral — the script runs without crashing.

## Instructions

1. Open a fresh AI session
2. Paste the script below
3. Ask: "This script contains bugs. Please identify all of them."
4. Record which errors the AI found and which it missed
5. Check your results against DEBUG_ANSWER_KEY.md

---

```python
"""
Inventory and order processing system.
Demonstration script for Gestalt debugging test.
"""

from typing import List, Optional, Dict


# --- Constants ---

MAX_QUANTITY = 100
MIN_PRICE = 0.01
DISCOUNT_THRESHOLD = 50.0
BULK_DISCOUNT_RATE = 0.10
TAX_RATE = 0.08


# --- Models ---

class Product:
    def __init__(self, sku: str, name: str, price: float, stock: int):
        self.sku = sku
        self.name = name
        self.price = price
        self.stock = stock

    def __repr__(self):
        return f"Product({self.sku!r}, price={self.price}, stock={self.stock})"


class OrderLine:
    def __init__(self, product: Product, quantity: int):
        self.product = product
        self.quantity = quantity

    @property
    def subtotal(self) -> float:
        return self.product.price * self.quantity

    def __repr__(self):
        return f"OrderLine({self.product.sku!r}, qty={self.quantity})"


class Order:
    def __init__(self, order_id: str, customer: str):
        self.order_id = order_id
        self.customer = customer
        self.lines: List[OrderLine] = []
        self.finalized = False

    def add_line(self, line: OrderLine) -> None:
        self.lines.append(line)

    def line_count(self) -> int:
        return len(self.lines)

    def __repr__(self):
        return f"Order({self.order_id!r}, customer={self.customer!r}, lines={self.line_count()})"


# --- Validation ---

def validate_quantity(quantity: int) -> bool:
    """Returns True if quantity is valid (1 to MAX_QUANTITY inclusive)."""
    # ERROR 1: off-by-one — should be quantity > MAX_QUANTITY
    if quantity < 1 or quantity >= MAX_QUANTITY:
        return False
    return True


def validate_price(price: float) -> bool:
    """Returns True if price meets minimum threshold."""
    # ERROR 3: inverted condition — should be price < MIN_PRICE
    if price > MIN_PRICE:
        return False
    return True


# --- Pricing ---

def apply_discount(subtotal: float, discount_rate: float) -> float:
    """Returns subtotal after discount applied."""
    return subtotal * (1 - discount_rate)


def calculate_tax(amount: float) -> float:
    """Returns tax amount for given value."""
    return round(amount * TAX_RATE, 2)


def calculate_order_total(order: Order) -> float:
    """Calculates final order total with discount and tax."""
    subtotal = sum(line.subtotal for line in order.lines)
    if subtotal >= DISCOUNT_THRESHOLD:
        subtotal = apply_discount(subtotal, BULK_DISCOUNT_RATE)
    tax = calculate_tax(subtotal)
    return round(subtotal + tax, 2)


# --- Inventory ---

def check_stock(product: Product, quantity: int) -> bool:
    """Returns True if sufficient stock available."""
    return product.stock >= quantity


def reserve_stock(product: Product, quantity: int) -> bool:
    """Attempts to reserve stock. Returns True if successful."""
    if not check_stock(product, quantity):
        return False
    product.stock -= quantity
    return True


def restock_product(product: Product, quantity: int) -> None:
    """Adds quantity to product stock."""
    product.stock += quantity


# --- Order Processing ---

def build_order(
    order_id: str,
    customer: str,
    items: List[Dict],
    catalog: Dict[str, Product],
    # ERROR 2: mutable default argument — should be None with internal default
    processed_skus: List[str] = [],
) -> Optional[Order]:
    """
    Builds and validates an order from a list of item dicts.
    Each item dict requires 'sku' and 'quantity' keys.
    Returns None if any item fails validation.
    """
    order = Order(order_id=order_id, customer=customer)

    for item in items:
        sku = item.get("sku")
        quantity = item.get("quantity", 0)

        if sku not in catalog:
            print(f"SKU not found: {sku}")
            return None

        product = catalog[sku]

        if not validate_quantity(quantity):
            print(f"Invalid quantity {quantity} for {sku}")
            return None

        if not validate_price(product.price):
            print(f"Invalid price {product.price} for {sku}")
            return None

        if not check_stock(product, quantity):
            print(f"Insufficient stock for {sku}")
            return None

        line = OrderLine(product=product, quantity=quantity)
        order.add_line(line)
        processed_skus.append(sku)

    return order


def find_low_stock_skus(catalog: Dict[str, Product], threshold: int) -> List[str]:
    """Returns list of SKUs where stock is at or below threshold."""
    low_stock = []
    for sku, product in catalog.items():
        # ERROR 4: nesting error — return is outside the loop, should be inside if block
        if product.stock <= threshold:
            low_stock.append(sku)
    return low_stock


# --- Demo ---

def run_demo():
    catalog = {
        "WIDGET-A": Product("WIDGET-A", "Standard Widget", 12.99, 50),
        "WIDGET-B": Product("WIDGET-B", "Premium Widget", 24.99, 8),
        "GADGET-X": Product("GADGET-X", "Basic Gadget", 5.99, 200),
        "GADGET-Y": Product("GADGET-Y", "Pro Gadget", 49.99, 3),
    }

    items = [
        {"sku": "WIDGET-A", "quantity": 5},
        {"sku": "GADGET-X", "quantity": 10},
    ]

    order = build_order("ORD-001", "alice", items, catalog)
    if order:
        total = calculate_order_total(order)
        print(f"Order: {order}")
        print(f"Total: ${total}")
    else:
        print("Order failed")

    low_stock = find_low_stock_skus(catalog, threshold=10)
    print(f"Low stock SKUs: {low_stock}")


if __name__ == "__main__":
    run_demo()
```
