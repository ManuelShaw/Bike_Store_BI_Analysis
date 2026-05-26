## Cycling Retail Customer Analysis

An end-to-end customer analysis project for a cycling retail company, combining machine learning, behavioral segmentation, and business intelligence to extract actionable insights from three years of transactional data (2020–2022).

### Contents

#### bike_store_analysis.ipynb
The main notebook, covering two analytical modules:
- **Product Anomaly Detection** — Isolation Forest model trained on sales volume and return behavior to flag anomalous products across the catalog
- **Customer Segmentation** — RFM scoring across 17,416 customers, followed by Random Forest Classifier and Regressor models to test whether demographics predict purchasing behavior, and K-Means clustering as an alternative unsupervised approach

#### Bike Store Dashboard.pbix
Power BI report with three pages: an executive overview, a product-level drill-down, and a customer detail view. Download the file to open it in Power BI Desktop — GitHub does not support in-browser preview of .pbix files. A full description of the dashboard and screenshots of each page are available in the project report.

#### docs/index.md
Full project report covering methodology, results, and interpretation for each module, including all visualizations and dashboard screenshots.

#### data/
Raw data files used in the analysis, organized in the following folders and files:
- `sales_data/` — Sales transactions for 2020, 2021, and 2022
- `product_data/` — Product catalog and subcategories
- `customers.csv` — Customer demographics
- `territories.csv` — Sales regions
- `calendar_lookup.csv` — Date reference table

### Tools & Libraries
Python · Pandas · NumPy · Scikit-learn · Matplotlib · Seaborn · Power BI
