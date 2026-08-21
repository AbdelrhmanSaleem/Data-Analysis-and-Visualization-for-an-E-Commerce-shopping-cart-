# E-commerce Shopping Cart — Analytics

Four raw CSV exports from an online store, taken through cleaning, forecasting and segmentation, and
published as dashboards in both Power BI and Tableau.

Built during the Digital Egypt Pioneers Initiative (DEPI).

---

## The problem

The store had four exports that did not agree with each other. Customers were duplicated, gender was
recorded three different ways, product names were typed by hand and drifted, and the sales sheet had
gaps in price, quantity and revenue. Nothing could be totalled with any confidence until that was
fixed.

---

## Cleaning

`E-commerce Shopping Cart Datasets Cleaning.ipynb` — pandas, seaborn, difflib

Each sheet is profiled before and after, so the effect of every step is visible in the notebook
rather than asserted.

| Sheet | What was wrong | What I did |
|---|---|---|
| Customers | Duplicate IDs, three spellings of each gender, missing ages and cities, inconsistent state casing | Deduplicated on `customer_id`, mapped gender to a fixed set, filled age with the median, marked unknown cities explicitly rather than dropping the row, normalised state casing |
| Orders | Duplicate order IDs, missing payments, dates stored as text with gaps | Deduplicated on `order_id`, filled payment with the mean, parsed dates properly and filled the remaining gaps with the median date |
| Sales | Missing values scattered across `price_per_unit`, `quantity` and `total_revenue` | See below |
| Products | Duplicate product IDs, hand-typed names that had drifted, missing prices | Deduplicated, repaired names by fuzzy matching against the valid set, filled price with the median |

Two steps are worth calling out.

**Recovering the sales figures instead of dropping them.** Price, quantity and revenue are
arithmetically related. Where exactly one of the three was missing, I calculated it from the other
two rather than discarding the row. Only rows missing enough to make that impossible were dropped.
That keeps revenue totals intact instead of quietly understating them.

**Repairing product names by similarity.** Hand-entered product types had drifted into near
misses that no exact join would catch. Each name is matched against the valid set with
`difflib.get_close_matches` at a 0.6 cutoff, which corrects genuine typos without collapsing
distinct products into each other.

---

## Forecasting

`E-commerce Shopping Cart Data Forecasting using Linear Regression.ipynb` — pandas, NumPy, scikit-learn

Sales are aggregated to monthly periods and two separate linear regressions are fitted:

- **Monthly revenue**, summed from the cleaned sales sheet
- **Monthly active customers**, counted as unique `customer_id` per month

Both are projected forward. Splitting revenue from customer count matters, because revenue rising
while the customer count is flat is a different business situation from both rising together, and
the two call for different responses.

---

## Segmentation

`RFM Model.xlsx` — customers scored on recency, frequency and monetary value, then grouped into
segments so marketing spend can be aimed at the customers likely to respond rather than the whole
list.

---

## Dashboards

- `E-Commerce Shopping Cart (Power Bi File).pbix`
- `E-commerce Shopping Cart Data Analysis (Tableau).twbx`

Both are built on the cleaned outputs, not the raw exports.

---

## Files

| File | What it is |
|---|---|
| `E-commerce Shopping Cart Datasets Cleaning.ipynb` | Cleaning and preparation, profiled before and after |
| `E-commerce Shopping Cart Data Forecasting using Linear Regression.ipynb` | Revenue and customer forecasts |
| `RFM Model.xlsx` | Recency, frequency and monetary segmentation |
| `E-Commerce Shopping Cart (Power Bi File).pbix` | Power BI dashboard |
| `E-commerce Shopping Cart Data Analysis (Tableau).twbx` | Tableau dashboard |
| `E-commerce Shopping Cart Data Analysis Documentation (DEPI).pdf` | Full written documentation |

---

## Running the notebooks

```
pip install pandas numpy scikit-learn matplotlib seaborn
```

The notebooks read `Customers.csv`, `Orders.csv`, `Sales.csv` and `Products.csv` from the working
directory. The export paths at the end of the cleaning notebook are commented out and point at the
original author's machine; set them to your own before running that cell.
