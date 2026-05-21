---
title: Cycling Retail Customer Analysis
---
<style>
h1:first-of-type {
  display: none;
}
</style>

# Cycling Retail Customer Analysis

## 1. Project Overview

This project presents an end-to-end customer analysis pipeline for a cycling retail company, using three years of transactional data (2020–2022). The goal is to extract actionable business intelligence from raw sales, returns, and customer records through a combination of unsupervised machine learning, rule-based segmentation, and predictive modeling — complemented by an interactive Power BI dashboard.

The analysis is structured in three modules:

**1. Product Anomaly Detection** applies an Isolation Forest model to flag products whose sales volume and return behavior deviate significantly from the rest of the catalog. Given the heterogeneous nature of a cycling retail assortment — where accessories, components, and full bicycles have fundamentally different demand and return profiles — a model-based approach was preferred over static thresholds.

**2. Customer Segmentation** combines a rule-based RFM framework (Recency, Frequency, Monetary value) with machine learning to profile the customer base. Random Forest models are then used to test whether demographic features can explain the resulting segments, and K-Means clustering is explored as an alternative unsupervised approach.

**3. Business Intelligence Dashboard** translates the analytical findings into an interactive Power BI report with three pages: an executive view with global KPIs and revenue trends, a product drill-down with performance vs. targets and return tracking, and a customer detail view with individual-level metrics and income-based segmentation.
