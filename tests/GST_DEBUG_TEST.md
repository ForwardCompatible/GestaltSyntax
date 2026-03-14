# Gestalt Debugging Test — Gestalt Encoded Version

## Purpose

This is the Gestalt v3.2 encoded version of DEBUG_TEST.md.
It contains the same four intentional errors. No errors are flagged or indicated in the encoding.

## Instructions

1. Open a fresh AI session — do not use the same session as the Python test
2. Provide your AI with STYLE_GUIDE.md first
3. Then paste the Gestalt encoded script below
4. Ask: "This script contains bugs. Please identify all of them."
5. Record which errors the AI found and which it missed
6. Check your results against DEBUG_ANSWER_KEY.md
7. Compare results between both sessions

---

```
META☩code_syntax☩lang:python,lbrace_escape:☸,rbrace_escape:☥

DOC☩inventory_order_system☩domain:python,type:demo☸

SEC☩constants☩level:1☸
RULE☩config☩domain:inventory☸MAX_QUANTITY=100,MIN_PRICE=0.01,DISCOUNT_THRESHOLD=50.0,BULK_DISCOUNT_RATE=0.10,TAX_RATE=0.08☥
☥

SEC☩models☩level:1☸

CLASS☩Product☩access:public☸
holds sku,name,price,stock☥

CLASS☩OrderLine☩access:public☸
holds product reference and quantity,subtotal property returns price times quantity☥

CLASS☩Order☩access:public☸
holds order_id,customer,lines list,finalized flag,add_line appends OrderLine,line_count returns len of lines☥
☥

SEC☩validation☩level:1☸
FUNC☩validate_quantity☩params:quantity:int,return:bool☸
returns True if quantity between 1 and MAX_QUANTITY inclusive,False otherwise☥

FUNC☩validate_price☩params:price:float,return:bool☸
returns True if price meets minimum threshold,False otherwise☥
☥

SEC☩pricing☩level:1☸
FUNC☩apply_discount☩params:subtotal:float,discount_rate:float,return:float☸
returns subtotal multiplied by one minus discount rate☥

FUNC☩calculate_tax☩params:amount:float,return:float☸
returns amount multiplied by TAX_RATE rounded to 2 decimal places☥

FUNC☩calculate_order_total☩params:order:Order,return:float☸
sums line subtotals,applies bulk discount if subtotal meets threshold,adds tax,returns rounded total☥
☥

SEC☩inventory☩level:1☸
FUNC☩check_stock☩params:product:Product,quantity:int,return:bool☸
returns True if product stock greater than or equal to quantity☥

FUNC☩reserve_stock☩params:product:Product,quantity:int,return:bool☸
checks stock,deducts quantity from product stock if available,returns True if successful☥

FUNC☩restock_product☩params:product:Product,quantity:int,return:None☸
adds quantity to product stock☥
☥

SEC☩order_processing☩level:1☸
FUNC☩build_order☩params:order_id:str,customer:str,items:List[Dict],catalog:Dict[str,Product],processed_skus:List[str],return:Optional[Order]☸
builds Order from item dicts,validates sku exists in catalog,validates quantity,validates price,checks stock,appends OrderLine,records sku in processed_skus,returns None if any validation fails☥

FUNC☩find_low_stock_skus☩params:catalog:Dict[str,Product],threshold:int,return:List[str]☸
iterates catalog,collects skus where stock at or below threshold,returns list☥
☥

SEC☩demo☩level:1☸
FUNC☩run_demo☩params:none,return:None☸
builds catalog of 4 products,creates order for WIDGET-A qty 5 and GADGET-X qty 10,prints total if successful,prints low stock skus☥

PROTOCOL☩entry_point☩sequence:final☸
calls run_demo☥
RELATES☩run_demo☩calls
☥

☥
```
