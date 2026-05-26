---
title: Cycling Retail Customer Analysis
---
<style>
h1:first-of-type {
  display: none;
}
</style>

# Cycling Retail Customer Analysis
<p align="center">
  <img src="images/cycling_front.jpg" width="800">
</p>

## Executive Summary

This project presents an end-to-end customer analysis pipeline for a cycling retail company, built on 56,046 transactions, 18,154 customers, and 293 products spanning 2020 to 2022.

On the product side, an **Isolation Forest** model flagged 7 anomalous products (~5% of the catalog), revealing two distinct behavioral profiles: high-volume demand outliers such as the Water Bottle - 30 oz. (7,967 units sold) and a high return rate outlier in the Road-650 Red, 52, which recorded an 11.76% return rate — the highest in the catalog by a significant margin.

On the customer side, an **RFM segmentation** across 17,416 customers identified 7 actionable segments. Champions (1,968 customers, $4,013 avg. spend) and Can't Lose Them (1,683 customers, $3,828 avg. spend) represent the highest-value groups and should be the primary focus of retention and reactivation strategies respectively. Loyal Customers form the largest segment (4,537 customers) and represent the strongest upsell opportunity. At the other end, Lost and At Risk customers show clear churn signals with low recency and minimal monetary value.

**Predictive modeling** revealed that demographic features alone are poor predictors of purchasing behavior — the Random Forest Classifier achieved only 24.3% accuracy on a 7-class problem, and R² values for the individual RFM regressors remained below 0.20. K-Means clustering confirmed this finding, with silhouette scores between 0.12 and 0.13 indicating no strong natural cluster structure in the data. Both results validate the rule-based RFM approach as the most appropriate segmentation framework for this customer base.

The analytical findings are complemented by an **interactive Power BI dashboard** with three pages — an executive overview, a product-level drill-down with performance vs. targets and a price adjustment simulator, and a customer-level view with individual metrics and income-based segmentation.


## 1. Project Overview

This project presents an end-to-end customer analysis pipeline for a cycling retail company, using three years of transactional data (2020–2022). The goal is to extract actionable business intelligence from raw sales, returns, and customer records through a combination of unsupervised machine learning, rule-based segmentation, and predictive modeling — complemented by an interactive Power BI dashboard.

The analysis is structured in five modules:

**1. Product Anomaly Detection** applies an Isolation Forest model to flag products whose sales volume and return behavior deviate significantly from the rest of the catalog. Given the heterogeneous nature of a cycling retail assortment — where accessories, components, and full bicycles have fundamentally different demand and return profiles — a model-based approach was preferred over static thresholds.

**2. Customer Segmentation (RFM)** combines a rule-based RFM framework (Recency, Frequency, Monetary value) with machine learning to profile the customer base into seven actionable segments.

**3. Predictive Modeling** uses Random Forest Classifier and Regressor models to test whether demographic features can explain or predict RFM-based segments, assessing the relationship between who the customer is and how they behave.

**4. K-Means Clustering Exploration** evaluates whether natural groupings emerge from the data using an unsupervised approach, and contrasts the results against the rule-based RFM segmentation.

**5. Business Intelligence Dashboard** translates the analytical findings into an interactive Power BI report with three pages: an executive view with global KPIs and revenue trends, a product drill-down with performance vs. targets and return tracking, and a customer detail view with individual-level metrics and income-based segmentation.


## 2. Data Sources

The analysis is built on seven relational tables covering sales transactions, customer demographics, product catalog, and return records. Sales data from three individual yearly files was consolidated into a single DataFrame of 56,046 records prior to analysis.

| Table | Description | Records |
|---|---|---|
| Sales 2020–2022 | Order-level transactional data | 56,046 |
| Customers | Demographics and customer profile | 18,154 |
| Products | Catalog with pricing and subcategory | 293 |
| Returns | Return records by product and territory | 1,809 |
| Product Categories | Top-level product hierarchy | 4 |
| Product Subcategories | Mid-level product hierarchy | 37 |
| Territories | Sales regions across 10 geographies | 10 |

All tables were joined on shared keys (`ProductKey`, `CustomerKey`, `TerritoryKey`) to build the analytical datasets for each module.

## 3. Module 1 — Product Anomaly Detection

### 3.1 Methodology

For each product, three behavioral metrics were computed from the consolidated sales and returns data:

- **total_sold**: total units sold across the 2020–2022 period
- **total_returned**: total units returned
- **return_rate**: proportion of sold units that were returned

An **Isolation Forest** model (`n_estimators=100`, `contamination=0.05`, `random_state=42`) was trained on these three features. The contamination parameter was set to 5%, meaning the model was configured to flag approximately 7 products out of the 293 in the catalog as anomalous.

A model-based approach was chosen over static thresholds because the catalog is highly heterogeneous — accessories like water bottles sell thousands of units while specific bike models sell fewer than 100. A single threshold would either miss true anomalies in low-volume products or over-flag high-volume ones.

### 3.2 Flagged Products

The model identified 7 anomalous products, which fall into two distinct behavioral patterns:

| Product | Total Sold | Total Returned | Return Rate | Anomaly Score |
|---|---|---|---|---|
| Water Bottle - 30 oz. | 7,967 | 155 | 1.95% | -0.209 |
| Patch Kit/8 Patches | 5,898 | 95 | 1.61% | -0.068 |
| Mountain Tire Tube | 5,678 | 93 | 1.64% | -0.062 |
| Road-650 Red, 52 | 51 | 6 | 11.76% | -0.060 |
| Mountain Bottle Cage | 3,810 | 77 | 2.02% | -0.030 |
| AWC Logo Cap | 4,151 | 46 | 1.11% | -0.019 |
| Road Tire Tube | 4,327 | 67 | 1.55% | -0.004 |

<p align="center">
  <img src="images/Anomaly_chart.png" width="800">
</p>

### 3.3 Interpretation

Two distinct anomaly profiles emerge from the results:

**High-volume outliers** — Water Bottle, Patch Kit, Mountain Tire Tube, Mountain Bottle Cage, AWC Logo Cap, and Road Tire Tube are flagged primarily due to their exceptionally high sales volume relative to the rest of the catalog. Their return rates are within a normal range, so the anomaly signal is driven by demand concentration rather than quality issues. These products likely warrant dedicated supply chain monitoring.

**High return rate outlier** — Road-650 Red, 52 stands apart with a return rate of 11.76%, the highest in the catalog by a significant margin. With only 51 units sold, its volume is low, but 1 in 8 units was returned — a pattern that suggests a potential quality, sizing, or product description issue worth investigating.

## 4. Module 2 — Customer Segmentation

### 4.1 Methodology

Customer segmentation was built using the **RFM framework**, a widely used approach in retail analytics that characterizes each customer along three behavioral dimensions:

- **Recency**: number of days since the customer's last purchase (relative to December 31, 2022)
- **Frequency**: total number of orders placed over the period
- **Monetary**: total revenue generated by the customer

Each metric was divided into 5 quintiles and assigned a score from 1 to 5. Recency was scored inversely — a more recent purchase yields a higher score. The three scores were combined into a composite `RFM_Score`, which was then mapped to one of 7 business segments using a rule-based classification.

Prior to scoring, the Monetary distribution was log-transformed to reduce the influence of high-spending outliers on the quintile boundaries.

### 4.2 Segment Summary

The RFM model was applied to 17,416 customers with complete transactional records. The resulting segments, ordered by average spend, are as follows:

| Segment | Customers | Avg Recency (days) | Avg Frequency | Avg Monetary | Avg Age | Avg Income |
|---|---|---|---|---|---|---|
| Champions | 1,968 | 53.4 | 2.4 | $4,013 | 59.2 | $61,367 |
| Can't Lose Them | 1,683 | 289.1 | 1.9 | $3,828 | 58.1 | $59,346 |
| Loyal Customers | 4,537 | 94.7 | 1.7 | $1,193 | 59.9 | $57,679 |
| Potential Loyalists | 2,605 | 201.1 | 1.0 | $737 | 60.0 | $56,534 |
| New Customers | 2,574 | 53.7 | 1.0 | $692 | 60.9 | $58,726 |
| At Risk | 2,261 | 276.8 | 1.1 | $598 | 59.9 | $53,689 |
| Lost | 1,788 | 258.4 | 1.0 | $60 | 61.1 | $54,284 |

<p align="center">
  <img src="images/RFM graph.png" width="800">
</p>

### 4.3 Interpretation

**Champions** are the most valuable segment — recent, relatively frequent, and high-spending. With an average monetary value of $4,013 and a recency of just 53 days, they represent the core of the business and should be prioritized for retention and loyalty programs.

**Can't Lose Them** share a similar spending profile to Champions ($3,828 average) but have not purchased in an average of 289 days. This is the most urgent segment for reactivation — these customers have demonstrated high value but are showing signs of disengagement.

**Loyal Customers** form the largest segment with 4,537 customers. Their mid-range spending ($1,193) and consistent frequency make them a strong candidate for upsell and cross-sell campaigns aimed at moving them toward Champion status.

**New Customers** and **Potential Loyalists** represent growth opportunities. Both have low frequency and monetary value, but New Customers are recent (53.7 days) while Potential Loyalists have been inactive for over 200 days, suggesting different engagement strategies are needed for each.

**At Risk** and **Lost** customers show the clearest signals of churn. The Lost segment in particular — with an average monetary value of just $60 — likely represents one-time or heavily lapsed buyers, where reactivation cost may outweigh the expected return.


## 5. Module 3 — Predictive Modeling

### 5.1 Objective

The goal of this module is to assess whether customer demographics can explain or predict RFM-based segments. If demographics were strong predictors of purchasing behavior, they could be used to anticipate segment membership for new customers before any transaction data is available.

Two approaches were tested:

- **Random Forest Classifier**: predicts the RFM segment label from demographic features
- **Random Forest Regressor**: predicts each RFM metric (Recency, Frequency, Monetary) individually

### 5.2 Feature Engineering

Customer age was derived from birth date using December 31, 2022 as the reference date. Education level was ordinally encoded on a 1–5 scale. The remaining categorical variables — Occupation, Gender, Marital Status, and Home Ownership — were one-hot encoded, resulting in a final feature set of 11 demographic variables:

`Age`, `AnnualIncome`, `TotalChildren`, `EducationLevel`, `Occupation_Management`, `Occupation_Manual`, `Occupation_Professional`, `Occupation_Skilled Manual`, `Gender_M`, `MaritalStatus_S`, `HomeOwner_Y`

The RFM table was merged with the encoded demographic features on `CustomerKey`, yielding a complete dataset of 17,416 records with no missing values.

### 5.3 Random Forest Classifier

A Random Forest Classifier was trained to predict the RFM segment from demographic features alone, using an 80/20 train-test split.

**Test set accuracy: 0.243**

An accuracy of 24.3% on a 7-class problem is only marginally better than random chance (14.3%), indicating that demographic features have very limited power to distinguish between RFM segments. The model struggles particularly with segments that share similar demographic profiles — which, as the segment summary shows, is the case across almost all groups.

<p align="center">
  <img src="images/Feature importance RFC.png" width="800">
</p>

### 5.4 Random Forest Regressors

Three separate Random Forest Regressors were trained to predict each RFM metric individually from the same demographic feature set.

| Target | R² Score |
|---|---|
| Recency | -0.204 |
| Frequency | -0.339 |
| Monetary | 0.185 |

All three models show poor predictive performance. Negative R² values for Recency and Frequency indicate that the models perform worse than a simple mean baseline. Monetary achieves a modest positive R² of 0.185, suggesting that income and occupation carry some signal for spending level — but not enough to be actionable on its own.

<p align="center">
  <img src="images/Feature importance RFR.png" width="800">
</p>

### 5.5 Interpretation

The consistent underperformance across both the classifier and the regressors points to a clear conclusion: **purchasing behavior in this customer base is driven more by individual preferences and timing than by demographic profile alone**. Knowing that a customer is 59 years old, has a bachelor's degree, and earns $60,000 per year tells us very little about when they last bought, how often they buy, or how much they spend.

This finding reinforces the value of the RFM approach — behavioral data, not demographic data, is the right lens for segmenting this customer base.

## 6. Module 4 - K-Means Clustering Exploration

### 6.1 Objective

As an alternative to the rule-based RFM segmentation, K-Means clustering was explored to assess whether the customer data contains natural groupings that emerge purely from geometric proximity in the RFM feature space — without relying on predefined business rules.

### 6.2 Methodology

The three RFM metrics (Recency, Frequency, Monetary) were standardized using `StandardScaler` prior to clustering. K-Means was run for values of K ranging from 2 to 11, and two metrics were computed for each:

- **Inertia (Elbow Method)**: measures within-cluster sum of squared distances; a pronounced elbow indicates a natural optimal K
- **Silhouette Score**: measures how well-separated the clusters are; values range from -1 to 1, with higher being better

### 6.3 Results

The Elbow Method showed a gradual, continuous decline in inertia with no pronounced inflection point, making it difficult to identify a clear optimal K. Silhouette scores across all tested values remained consistently low, ranging between **0.12 and 0.13**, with no meaningful improvement as K increased.

<p align="center">
  <img src="images/Elbow & silhouette.png" width="800">
</p>

### 6.4 Interpretation

Silhouette scores in the 0.12–0.13 range indicate that the customer data does not have a strong natural cluster structure in RFM space. Forcing K-Means under these conditions produces groupings that are more a mathematical artifact than meaningful customer groups — customers assigned to different clusters are not substantially more similar to their cluster peers than to customers in other clusters.

This result directly supports the decision to rely on the **rule-based RFM segmentation** as the primary framework. Business-logic-driven segments may not be geometrically tight, but they are interpretable, stable, and grounded in domain knowledge — qualities that matter far more in a retail context than cluster compactness alone.

## 7. Power BI Dashboard

The analytical findings from both modules were translated into an interactive Power BI report designed for business users. The report consists of three pages, each addressing a different level of analysis, and includes a collapsible slicer panel for filtering by year and country.

### 7.1 Module 5 - Executive Dashboard

The main page provides a high-level overview of business performance across the full period. Four KPI cards display the most critical metrics at a glance: **$25M in total revenue**, **$10M in total profit**, **25,160 total orders**, and a **2.17% overall return rate**.

A weekly revenue chart with a trend line illustrates the consistent growth trajectory from 2020 through mid-2022. Three monthly KPI cards track the most recent period against the previous month, providing an at-a-glance view of short-term momentum across revenue, orders, and returns.

On the right side, an orders-by-category bar chart shows that Accessories lead with 17,000 orders, followed by Bikes (13,900) and Clothing (7,000). A top 10 products table displays orders, revenue, and return rate per product — with return rates highlighted in red for values above a defined threshold. Dynamic cards surface the most ordered product type (**Tires and Tubes**) and the most returned product type (**Shorts**).

<p align="center">
  <img src="images/Dashboard 1.1 - Excec Dashboard.jpg" width="800">
</p>

The dashboard includes a collapsible slicer panel, accessible via the filter icon on the left sidebar, allowing users to filter all visuals by year (2020, 2021, 2022) and country (Australia, Canada, France, Germany, United Kingdom, United States).

<p align="center">
  <img src="images/Dashboard 1.2 - Slicer Panel.png" width="400">
</p>

Hovering over a category bar triggers a custom tooltip showing a **Weekly Orders** area chart alongside detailed KPIs for that category, including total revenue, profit, orders, returns, and return rate.

<p align="center">
  <img src="images/Dashboard 1.3 - Category Tooltip.png" width="500">
</p>

### 7.2 Product Detail

The second page provides a drill-down view for individual products. Gauge charts compare current orders, profit, and revenue against predefined targets, giving an immediate visual signal of whether a product is on track. An interactive **price adjustment slider** allows the user to simulate the impact of a price change on adjusted profit, displayed alongside the actual profit trend in a dual-line chart. A weekly returns area chart completes the view, enabling return pattern analysis over time for the selected product.

<p align="center">
  <img src="images/Dashboard 2 - Product Detail.jpg" width="800">
</p>

### 7.3 Customer Detail

The third page shifts the focus to the customer level. A toggle between **Total Customers** and **Avg Revenue per Customer** controls the weekly customers chart, allowing the user to switch between volume and value perspectives. A searchable customer table lists individual records with total orders and total revenue, with high-value customers highlighted. Two donut charts break down orders by income level bracket. Prominent KPI cards surface the **top revenue customer** (Mr. Maurice Shan, $12M) and the **customer with the most orders** (Mr. Dalton Perez, 26 orders).

<p align="center">
  <img src="images/Dashboard 3 - Customer Detail.jpg" width="800">
</p>


## 8. Conclusions

This project demonstrates how a combination of unsupervised machine learning, rule-based segmentation, and business intelligence tooling can extract actionable insights from retail transactional data.

On the product side, the Isolation Forest model successfully identified two distinct types of anomalous behavior within the catalog. High-volume products like the Water Bottle and Patch Kit stand out due to demand concentration rather than quality issues, while the Road-650 Red, 52 — with an 11.76% return rate — represents a genuine quality signal that warrants direct investigation. The ability to distinguish between these two anomaly profiles is precisely what makes a model-based approach more valuable than a simple threshold rule.

On the customer side, the RFM segmentation produced seven interpretable and actionable segments across 17,416 customers. The Champions and Can't Lose Them segments, despite their similar spending profiles, require fundamentally different strategies: the former needs retention and reward, the latter urgently needs reactivation. The predictive modeling results reinforce an important finding — demographic features alone are poor predictors of purchasing behavior, with classifier accuracy at 24.3% and R² values below 0.20 across all three regressors. This is not a modeling failure; it is a meaningful result that confirms behavioral data is the right foundation for segmentation in this context. The K-Means exploration corroborates this conclusion, with silhouette scores between 0.12 and 0.13 indicating no strong natural cluster structure in the data.

Together, these findings point to a clear priority order for business action: protect Champions, reactivate Can't Lose Them, upsell Loyal Customers, and investigate the Road-650 Red before the return pattern erodes its margin further.
