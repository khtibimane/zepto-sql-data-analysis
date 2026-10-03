# Zepto Inventory Analysis (SQL)

SQL analysis of 7,462 Zepto product listings covering discounts, stock availability, pricing and inventory. Includes data cleaning, eight business queries, and a data quality issue that affects category-level reporting.

**Tools:** PostgreSQL 18, pgAdmin 4

---

## Key Findings

| Area | Finding |
|---|---|
| Stock | 906 of 7,462 listings (12.1%) are out of stock. Four have an MRP above ₹300, led by Patanjali Cow's Ghee (₹565) |
| Discounts | Highest discount is 51%. Fruits & Vegetables have the highest average discount (15.46%), then Meats, Fish & Eggs (11.03%) |
| Pricing | Oil jars with an MRP of ₹1,050 to ₹1,250 carry discounts of 0% to 8% (Q4 output) |
| Data quality | Several categories return identical totals in three separate queries; the 14 categories reduce to 9 distinct profiles |

## Dataset

`zepto_v2.csv`: 7,462 rows, 14 categories, one row per SKU. Columns: `category`, `name`, `mrp`, `discountPercent`, `availableQuantity`, `discountedSellingPrice`, `weightInGms`, `outOfStock`, `quantity`. Prices are stored in paise and converted to rupees during cleaning. `sku_id` is generated on import.

## Approach

1. **Explore:** row count, null check, categories, stock split, repeated names. Result: no nulls; 6,556 in stock and 906 out of stock; 1,680 product names appear more than once.
2. **Clean:** checked for zero prices (none found) and converted `mrp` and `discountedSellingPrice` from paise to rupees.
3. **Analyse:** eight queries using aggregates, `GROUP BY` / `HAVING`, `CASE WHEN`, `DISTINCT`, sorting and limits.

## Business Questions and Results

| # | Question | Result |
|---|---|---|
| Q1 | Top 10 products by discount | Three Dukes Waffy wafers at 51%; seven products at 50% |
| Q2 | Out-of-stock products with MRP above ₹300 | Cow's Ghee ₹565, MamyPoko Pants XL ₹399, Aashirvaad Multigrain Atta ₹315, Everest Kashmiri Lal Chilli Powder ₹310 |
| Q3 | Estimated inventory value by category | Cooking Essentials and Munchies tied at ₹674,738; Fruits & Vegetables lowest at ₹21,692 |
| Q4 | MRP above ₹500 and discount below 10% | Rows shown are oil jars with 0% to 8% discounts |
| Q5 | Top 5 categories by average discount | Fruits & Vegetables 15.46%, Meats, Fish & Eggs 11.03%, three categories tied at 8.32% |
| Q6 | Price per gram (products of 100 g or more) | 1,395 products; lowest are Vicks Cough Drops (₹0.0172/g), iodised salt and onions (about ₹0.019/g) |
| Q7 | Weight bands: Low (under 1 kg), Medium (1 to under 5 kg), Bulk (5 kg and above) | 1,782 distinct product and weight combinations classified |
| Q8 | Total inventory weight by category | Munchies and Cooking Essentials tied at 2,809,308 g; Meats, Fish & Eggs lowest at 96,032 g |

Q3 is selling price × units in stock, so it measures estimated inventory value, not sales.

## Data Quality Issue

Categories in these groups return identical value, weight and (where shown) average discount:

| Categories | Est. inventory value | Total weight |
|---|---|---|
| Cooking Essentials, Munchies | ₹674,738 | 2,809,308 g |
| Personal Care, Paan Corner | ₹541,698 | 696,374 g |
| Ice Cream & Desserts, Chocolates & Candies, Packaged Food | ₹448,770 | 981,594 g |
| Dairy, Bread & Batter, Beverages | ₹110,102 | 287,470 g |

The same groups appear in Q3 and Q8, and the three-way tie in Q5 (8.32%) matches the third group, so coincidence is very unlikely. The likely cause is the same products listed under several categories; this is inferred from the totals and not yet tested row by row. If correct, summing all 14 categories (₹4,486,161.20) overstates the 9 distinct values (₹2,262,083.20), so category rankings should be used with caution. Product-level results (Q1, Q2, Q4, Q6) are less affected.

Other checks: stock counts sum to the row count (6,556 + 906 = 7,462), and Q6 price-per-gram values were recomputed and match. One value looks wrong: Vicks Cough Drops at 1,160 g for ₹20.

## Recommendations

- Check and restock the four high-MRP out-of-stock items first.
- Test a small promotion on premium cooking oils.
- Review whether the deep Fruits & Vegetables discounts are justified; they have the highest average discount but the lowest estimated inventory value.
- Fix category mapping before any category-level reporting.

These are hypotheses: the data has stock levels but no sales.

## Repository

```
Zepto_SQL_data_analysis.sql   schema, exploration, cleaning, analysis (Q1 to Q8)
zepto_v2.csv                  dataset
README.md
```

**To run:** create a database, run the schema section, import `zepto_v2.csv` (leave out `sku_id`), then run exploration, cleaning and analysis. Run the cleaning step once only. Re-running the schema section deletes the data, and re-running the paise-to-rupee `UPDATE` divides prices by 100 again.

## Limitations

- Inventory value and weight reflect current stock, not sales.
- The `quantity` column is undocumented and was not used.
- Q4, Q6 and Q7 conclusions are based on the rows shown in the output.

---

**Imane Khtib** | khtibimane23@gmail.com
