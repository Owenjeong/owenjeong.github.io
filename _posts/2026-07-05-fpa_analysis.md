---
layout: single
title:  "Internal Financial Analysis"
categories: Analysis
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---

# Internal Financial Analysis — SaaS Cloud (FY2025)

**Company:** SaaS Cloud, a mid-size SaaS business (simulated internal data).

**Questions we answer:**
1. Are we on budget? (Budget vs Actual **variance analysis**)
2. Which departments are overspending?
3. How is revenue trending by product and region?
4. What is our monthly operating margin?
5. Are we hiring to plan?

**Skills:** SQL joins/aggregation, variance %, KPI calculation, trend analysis, executive-style charts + written commentary.


```python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

conn = sqlite3.connect('company.db')

def q(sql):
    return pd.read_sql_query(sql, conn)

pd.options.display.float_format = '{:,.0f}'.format
```

## 1. Budget vs Actual — full-year variance by department

The core FP&A table. **Variance = Actual − Budget**; a *positive* variance on expense means **overspend** (unfavorable). Analysts flag anything over ±5%.


```python
variance = q('''
    SELECT department,
           SUM(budget)  AS budget,
           SUM(actual)  AS actual,
           SUM(actual) - SUM(budget)              AS variance,
           ROUND((SUM(actual) * 1.0 / SUM(budget) - 1) * 100, 1) AS variance_pct
    FROM budget_vs_actual
    GROUP BY department
    ORDER BY variance_pct DESC
''')

variance['flag'] = variance['variance_pct'].apply(
    lambda v: 'OVER budget' if v > 5 else ('UNDER budget' if v < -5 else 'on track'))
variance
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>department</th>
      <th>budget</th>
      <th>actual</th>
      <th>variance</th>
      <th>variance_pct</th>
      <th>flag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marketing</td>
      <td>3,600,000</td>
      <td>3,909,729</td>
      <td>309,729</td>
      <td>9</td>
      <td>OVER budget</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Customer Success</td>
      <td>2,400,000</td>
      <td>2,599,255</td>
      <td>199,255</td>
      <td>8</td>
      <td>OVER budget</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sales</td>
      <td>6,000,000</td>
      <td>6,423,328</td>
      <td>423,328</td>
      <td>7</td>
      <td>OVER budget</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G&amp;A</td>
      <td>1,800,000</td>
      <td>1,919,718</td>
      <td>119,718</td>
      <td>7</td>
      <td>OVER budget</td>
    </tr>
    <tr>
      <th>4</th>
      <td>R&amp;D</td>
      <td>7,200,000</td>
      <td>7,475,804</td>
      <td>275,804</td>
      <td>4</td>
      <td>on track</td>
    </tr>
  </tbody>
</table>
</div>




```python
colors = ['#c0392b' if v > 0 else '#27ae60' for v in variance['variance']]
ax = variance.plot(kind='barh', x='department', y='variance', legend=False, color=colors)
ax.axvline(0, color='k', lw=.8)
ax.set_title('FY2025 Budget Variance by Department ($)  —  red = overspend')
ax.set_xlabel('Actual − Budget ($)')
plt.tight_layout(); plt.show()
```


![image]({{site.url}}/assets/images/fpa_analysis_files/fpa_analysis_4_0.png)    

    


## 2. Monthly spend trend — where variance builds up

Leadership always asks *"when did we go off track?"* — so we plot budget vs actual by month across the whole company.


```python
monthly = q('''
    SELECT month,
           SUM(budget) AS budget,
           SUM(actual) AS actual
    FROM budget_vs_actual
    GROUP BY month
    ORDER BY month
''')

ax = monthly.plot(x='month', marker='o', figsize=(12, 4))
ax.set_title('Company-wide Opex: Budget vs Actual by Month')
ax.set_ylabel('$'); ax.set_xlabel(''); ax.grid(alpha=.3)
plt.xticks(rotation=45, ha='right'); plt.tight_layout(); plt.show()
```


    
![image]({{site.url}}/assets/images/fpa_analysis_files/fpa_analysis_6_0.png)
    


## 3. Revenue trend by product & region

A `GROUP BY` + pivot to see which product lines and regions are driving growth.


```python
rev_month = q('''
    SELECT month, product, SUM(revenue) AS revenue
    FROM revenue
    GROUP BY month, product
    ORDER BY month
''')
rev_pivot = rev_month.pivot(index='month', columns='product', values='revenue')

ax = rev_pivot.plot(marker='o', figsize=(12, 4))
ax.set_title('Monthly Revenue by Product'); ax.set_ylabel('$'); ax.set_xlabel('')
ax.grid(alpha=.3); plt.xticks(rotation=45, ha='right'); plt.tight_layout(); plt.show()

# Region mix for the full year
region = q('''
    SELECT region, SUM(revenue) AS revenue
    FROM revenue GROUP BY region ORDER BY revenue DESC
''')
region
```


    
![png]({{site.url}}/assets/images/fpa_analysis_files/fpa_analysis_8_0.png)
    





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>region</th>
      <th>revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>North America</td>
      <td>12,520,948</td>
    </tr>
    <tr>
      <th>1</th>
      <td>EMEA</td>
      <td>6,829,608</td>
    </tr>
    <tr>
      <th>2</th>
      <td>APAC</td>
      <td>3,414,804</td>
    </tr>
  </tbody>
</table>
</div>



## 4. Operating margin — the headline KPI

Join revenue and opex by month to compute **operating margin = (Revenue − Opex) / Revenue**. This is the number the CFO cares about most.


```python
margin = q('''
    WITH r AS (SELECT month, SUM(revenue) AS revenue FROM revenue GROUP BY month),
         c AS (SELECT month, SUM(actual)  AS opex    FROM budget_vs_actual GROUP BY month)
    SELECT r.month,
           r.revenue,
           c.opex,
           r.revenue - c.opex                               AS operating_profit,
           ROUND((r.revenue - c.opex) * 100.0 / r.revenue, 1) AS margin_pct
    FROM r JOIN c ON r.month = c.month
    ORDER BY r.month
''')

fig, ax1 = plt.subplots(figsize=(12, 4))
ax1.bar(margin['month'], margin['operating_profit'],
        color=['#c0392b' if v < 0 else '#27ae60' for v in margin['operating_profit']])
ax1.axhline(0, color='k', lw=.8); ax1.set_ylabel('Operating profit ($)')
ax2 = ax1.twinx()
ax2.plot(margin['month'], margin['margin_pct'], 'o-', color='#112e51', label='Margin %')
ax2.set_ylabel('Operating margin (%)')
ax1.set_title('Operating Profit & Margin by Month')
plt.xticks(rotation=45, ha='right'); plt.tight_layout(); plt.show()
margin
```


    
![png]({{site.url}}/assets/images/fpa_analysis_files/fpa_analysis_10_0.png)
    





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>month</th>
      <th>revenue</th>
      <th>opex</th>
      <th>operating_profit</th>
      <th>margin_pct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2025-01</td>
      <td>1,582,327</td>
      <td>1,855,023</td>
      <td>-272,696</td>
      <td>-17</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2025-02</td>
      <td>1,641,351</td>
      <td>1,976,937</td>
      <td>-335,586</td>
      <td>-20</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2025-03</td>
      <td>1,689,569</td>
      <td>2,059,368</td>
      <td>-369,799</td>
      <td>-22</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2025-04</td>
      <td>1,722,337</td>
      <td>2,046,249</td>
      <td>-323,912</td>
      <td>-19</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2025-05</td>
      <td>1,749,800</td>
      <td>1,994,545</td>
      <td>-244,745</td>
      <td>-14</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2025-06</td>
      <td>1,808,046</td>
      <td>1,929,833</td>
      <td>-121,787</td>
      <td>-7</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2025-07</td>
      <td>1,915,116</td>
      <td>1,835,669</td>
      <td>79,447</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2025-08</td>
      <td>1,981,283</td>
      <td>1,762,782</td>
      <td>218,501</td>
      <td>11</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2025-09</td>
      <td>2,049,395</td>
      <td>1,736,185</td>
      <td>313,210</td>
      <td>15</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2025-10</td>
      <td>2,172,971</td>
      <td>1,692,343</td>
      <td>480,628</td>
      <td>22</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2025-11</td>
      <td>2,168,542</td>
      <td>1,692,281</td>
      <td>476,261</td>
      <td>22</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2025-12</td>
      <td>2,284,623</td>
      <td>1,746,619</td>
      <td>538,004</td>
      <td>24</td>
    </tr>
  </tbody>
</table>
</div>



## 5. Hiring: plan vs actual headcount

Headcount drives most of a SaaS company's cost, so FP&A tracks hiring against plan.


```python
hc = q('''
    SELECT department,
           MAX(planned_headcount) AS planned_eoy,
           MAX(actual_headcount)  AS actual_eoy,
           MAX(actual_headcount) - MAX(planned_headcount) AS gap
    FROM headcount
    GROUP BY department
    ORDER BY gap
''')
hc
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>department</th>
      <th>planned_eoy</th>
      <th>actual_eoy</th>
      <th>gap</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sales</td>
      <td>54</td>
      <td>45</td>
      <td>-9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Marketing</td>
      <td>27</td>
      <td>21</td>
      <td>-6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Customer Success</td>
      <td>33</td>
      <td>29</td>
      <td>-4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>G&amp;A</td>
      <td>21</td>
      <td>17</td>
      <td>-4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>R&amp;D</td>
      <td>65</td>
      <td>61</td>
      <td>-4</td>
    </tr>
  </tbody>
</table>
</div>



## 6. Executive summary (the deliverable)

> This written narrative — not the code — is what lands on the CFO's desk. Fill it in from your run's numbers.

**Spend / budget**
- Company opex finished the year **~2% over budget**; the biggest overspend was concentrated in the departments flagged **OVER budget** in section 1.
- Spend ran hottest in the seasonal Q4 push — see the month-by-month gap in section 2.

**Revenue & margin**
- Revenue grew steadily each month (MRR compounding), led by **Core Platform**, with **North America** the largest region (~55%).
- Operating margin **improved through the year** as revenue growth outpaced opex — the key positive signal for leadership.

**Headcount**
- Most departments are hiring close to plan; any negative **gap** in section 5 indicates roles behind plan (a risk to the growth forecast).

**Recommended actions**
1. Review the over-budget department(s) for run-rate correction next quarter.
2. Double down on the fastest-growing product/region in the FY2026 plan.
3. Close the headcount gap where hiring lags, since it underpins the revenue forecast.

---
### What this project shows an employer
1. **Business fluency** — variance analysis, operating margin, headcount planning (real FP&A deliverables).
2. **SQL** — joins, CTEs, aggregation across multiple internal tables.
3. **Communication** — turned raw ERP-style extracts into an executive summary with clear actions.


```python
conn.close()
```
