---
layout: single
title:  "DCF Valuation & Analysis (TNK)"
categories: Analysis
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---

# Stock Valuation using Free Cash Flow to the Firm with Python

**[Info]** [**DCF modeling guideline**]({{site.url}}/analysis/valuation_fcff/)
{: .notice--info}
*The detail of the calculations will be skipped, all explaination is in the AAPL DCF modeling.😊* 

### DCF Modeling

1. Pull out quarter financial statments<br>

**Quarter Balance Sheet**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
import yfinance as yf
import pandas as pd
import datetime

ticker = yf.Ticker("TNK")
# - income statement
# pd.set_option('display.max_rows', None)
balance_sheet_df = ticker.quarterly_balance_sheet
balance_sheet_df
{% endhighlight %}

</details>


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
      <th>2023-09-30</th>
      <th>2023-06-30</th>
      <th>2023-03-31</th>
      <th>2022-12-31</th>
      <th>2022-09-30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Ordinary Shares Number</th>
      <td>34079721.0</td>
      <td>34000000.0</td>
      <td>33900000.0</td>
      <td>33938942.0</td>
      <td>33900000.0</td>
    </tr>
    <tr>
      <th>Share Issued</th>
      <td>34079721.0</td>
      <td>34000000.0</td>
      <td>33900000.0</td>
      <td>33938942.0</td>
      <td>33900000.0</td>
    </tr>
    <tr>
      <th>Total Debt</th>
      <td>231344000.0</td>
      <td>298953000.0</td>
      <td>451496000.0</td>
      <td>576203000.0</td>
      <td>586884000.0</td>
    </tr>
    <tr>
      <th>Tangible Book Value</th>
      <td>1418537000.0</td>
      <td>1345514000.0</td>
      <td>1236901000.0</td>
      <td>1066529000.0</td>
      <td>919113000.0</td>
    </tr>
    <tr>
      <th>Invested Capital</th>
      <td>NaN</td>
      <td>1348789000.0</td>
      <td>1240275000.0</td>
      <td>1070006000.0</td>
      <td>945073000.0</td>
    </tr>
    <tr>
      <th>Working Capital</th>
      <td>334945000.0</td>
      <td>305165000.0</td>
      <td>313175000.0</td>
      <td>289335000.0</td>
      <td>144205000.0</td>
    </tr>
    <tr>
      <th>Net Tangible Assets</th>
      <td>1418537000.0</td>
      <td>1345514000.0</td>
      <td>1236901000.0</td>
      <td>1066529000.0</td>
      <td>919113000.0</td>
    </tr>
    <tr>
      <th>Capital Lease Obligations</th>
      <td>231344000.0</td>
      <td>298953000.0</td>
      <td>451496000.0</td>
      <td>576203000.0</td>
      <td>564507000.0</td>
    </tr>
    <tr>
      <th>Common Stock Equity</th>
      <td>1421715000.0</td>
      <td>1348789000.0</td>
      <td>1240275000.0</td>
      <td>1070006000.0</td>
      <td>922696000.0</td>
    </tr>
    <tr>
      <th>Total Capitalization</th>
      <td>1421715000.0</td>
      <td>1348789000.0</td>
      <td>1240275000.0</td>
      <td>1070006000.0</td>
      <td>922696000.0</td>
    </tr>
    <tr>
      <th>Total Equity Gross Minority Interest</th>
      <td>1421715000.0</td>
      <td>1348789000.0</td>
      <td>1240275000.0</td>
      <td>1070006000.0</td>
      <td>922696000.0</td>
    </tr>
    <tr>
      <th>Stockholders Equity</th>
      <td>1421715000.0</td>
      <td>1348789000.0</td>
      <td>1240275000.0</td>
      <td>1070006000.0</td>
      <td>922696000.0</td>
    </tr>
    <tr>
      <th>Retained Earnings</th>
      <td>NaN</td>
      <td>44117000.0</td>
      <td>-64236000.0</td>
      <td>-233604000.0</td>
      <td>-380031000.0</td>
    </tr>
    <tr>
      <th>Capital Stock</th>
      <td>NaN</td>
      <td>1304672000.0</td>
      <td>1304511000.0</td>
      <td>1303610000.0</td>
      <td>1302727000.0</td>
    </tr>
    <tr>
      <th>Common Stock</th>
      <td>NaN</td>
      <td>1304672000.0</td>
      <td>1304511000.0</td>
      <td>1303610000.0</td>
      <td>1302727000.0</td>
    </tr>
    <tr>
      <th>Total Liabilities Net Minority Interest</th>
      <td>348920000.0</td>
      <td>431028000.0</td>
      <td>598605000.0</td>
      <td>713648000.0</td>
      <td>709886000.0</td>
    </tr>
    <tr>
      <th>Total Non Current Liabilities Net Minority Interest</th>
      <td>218957000.0</td>
      <td>284787000.0</td>
      <td>423600000.0</td>
      <td>543474000.0</td>
      <td>534668000.0</td>
    </tr>
    <tr>
      <th>Other Non Current Liabilities</th>
      <td>47501000.0</td>
      <td>51681000.0</td>
      <td>46179000.0</td>
      <td>44017000.0</td>
      <td>42485000.0</td>
    </tr>
    <tr>
      <th>Long Term Debt And Capital Lease Obligation</th>
      <td>171456000.0</td>
      <td>233106000.0</td>
      <td>377421000.0</td>
      <td>499457000.0</td>
      <td>492183000.0</td>
    </tr>
    <tr>
      <th>Long Term Capital Lease Obligation</th>
      <td>171456000.0</td>
      <td>233106000.0</td>
      <td>377421000.0</td>
      <td>499457000.0</td>
      <td>492183000.0</td>
    </tr>
    <tr>
      <th>Current Liabilities</th>
      <td>129963000.0</td>
      <td>146241000.0</td>
      <td>175005000.0</td>
      <td>170174000.0</td>
      <td>175218000.0</td>
    </tr>
    <tr>
      <th>Other Current Liabilities</th>
      <td>2890000.0</td>
      <td>7034000.0</td>
      <td>9135000.0</td>
      <td>2468000.0</td>
      <td>1310000.0</td>
    </tr>
    <tr>
      <th>Current Debt And Capital Lease Obligation</th>
      <td>59888000.0</td>
      <td>65847000.0</td>
      <td>74075000.0</td>
      <td>76746000.0</td>
      <td>94701000.0</td>
    </tr>
    <tr>
      <th>Current Capital Lease Obligation</th>
      <td>59888000.0</td>
      <td>65847000.0</td>
      <td>74075000.0</td>
      <td>76746000.0</td>
      <td>72324000.0</td>
    </tr>
    <tr>
      <th>Current Debt</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22377000.0</td>
    </tr>
    <tr>
      <th>Other Current Borrowings</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>22377000.0</td>
    </tr>
    <tr>
      <th>Line Of Credit</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Payables And Accrued Expenses</th>
      <td>67185000.0</td>
      <td>73360000.0</td>
      <td>91795000.0</td>
      <td>90960000.0</td>
      <td>79207000.0</td>
    </tr>
    <tr>
      <th>Current Accrued Expenses</th>
      <td>NaN</td>
      <td>32695000.0</td>
      <td>45946000.0</td>
      <td>47469000.0</td>
      <td>42864000.0</td>
    </tr>
    <tr>
      <th>Interest Payable</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1976000.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Payables</th>
      <td>67185000.0</td>
      <td>40665000.0</td>
      <td>45849000.0</td>
      <td>90960000.0</td>
      <td>36343000.0</td>
    </tr>
    <tr>
      <th>Dueto Related Parties Current</th>
      <td>4257000.0</td>
      <td>3244000.0</td>
      <td>1915000.0</td>
      <td>1141000.0</td>
      <td>1376000.0</td>
    </tr>
    <tr>
      <th>Accounts Payable</th>
      <td>62928000.0</td>
      <td>37421000.0</td>
      <td>43934000.0</td>
      <td>89819000.0</td>
      <td>34967000.0</td>
    </tr>
    <tr>
      <th>Total Assets</th>
      <td>1770635000.0</td>
      <td>1779817000.0</td>
      <td>1838880000.0</td>
      <td>1783654000.0</td>
      <td>1632582000.0</td>
    </tr>
    <tr>
      <th>Total Non Current Assets</th>
      <td>1305727000.0</td>
      <td>1328411000.0</td>
      <td>1350700000.0</td>
      <td>1324145000.0</td>
      <td>1313159000.0</td>
    </tr>
    <tr>
      <th>Other Non Current Assets</th>
      <td>6096000.0</td>
      <td>7139000.0</td>
      <td>6268000.0</td>
      <td>8208000.0</td>
      <td>6927000.0</td>
    </tr>
    <tr>
      <th>Financial Assets</th>
      <td>NaN</td>
      <td>0.0</td>
      <td>1092000.0</td>
      <td>1622000.0</td>
      <td>2111000.0</td>
    </tr>
    <tr>
      <th>Investments And Advances</th>
      <td>15215000.0</td>
      <td>16549000.0</td>
      <td>17328000.0</td>
      <td>16198000.0</td>
      <td>14490000.0</td>
    </tr>
    <tr>
      <th>Long Term Equity Investment</th>
      <td>15215000.0</td>
      <td>16549000.0</td>
      <td>17328000.0</td>
      <td>16198000.0</td>
      <td>14490000.0</td>
    </tr>
    <tr>
      <th>Investmentsin Joint Venturesat Cost</th>
      <td>15215000.0</td>
      <td>16549000.0</td>
      <td>17328000.0</td>
      <td>16198000.0</td>
      <td>14490000.0</td>
    </tr>
    <tr>
      <th>Goodwill And Other Intangible Assets</th>
      <td>3178000.0</td>
      <td>3275000.0</td>
      <td>3374000.0</td>
      <td>3477000.0</td>
      <td>3583000.0</td>
    </tr>
    <tr>
      <th>Other Intangible Assets</th>
      <td>752000.0</td>
      <td>849000.0</td>
      <td>948000.0</td>
      <td>1051000.0</td>
      <td>1157000.0</td>
    </tr>
    <tr>
      <th>Goodwill</th>
      <td>2426000.0</td>
      <td>2426000.0</td>
      <td>2426000.0</td>
      <td>2426000.0</td>
      <td>2426000.0</td>
    </tr>
    <tr>
      <th>Net PPE</th>
      <td>1281238000.0</td>
      <td>1301448000.0</td>
      <td>1322638000.0</td>
      <td>1296262000.0</td>
      <td>1286048000.0</td>
    </tr>
    <tr>
      <th>Accumulated Depreciation</th>
      <td>NaN</td>
      <td>-507700000.0</td>
      <td>-485600000.0</td>
      <td>-461800000.0</td>
      <td>-441400000.0</td>
    </tr>
    <tr>
      <th>Gross PPE</th>
      <td>1281238000.0</td>
      <td>1809148000.0</td>
      <td>1808238000.0</td>
      <td>1296262000.0</td>
      <td>1727448000.0</td>
    </tr>
    <tr>
      <th>Other Properties</th>
      <td>86624000.0</td>
      <td>90831000.0</td>
      <td>92691000.0</td>
      <td>42894000.0</td>
      <td>16063000.0</td>
    </tr>
    <tr>
      <th>Current Assets</th>
      <td>464908000.0</td>
      <td>451406000.0</td>
      <td>488180000.0</td>
      <td>459509000.0</td>
      <td>319423000.0</td>
    </tr>
    <tr>
      <th>Hedging Assets Current</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>2033000.0</td>
      <td>2087000.0</td>
      <td>2377000.0</td>
    </tr>
    <tr>
      <th>Assets Held For Sale Current</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Restricted Cash</th>
      <td>691000.0</td>
      <td>703000.0</td>
      <td>3703000.0</td>
      <td>3714000.0</td>
      <td>2730000.0</td>
    </tr>
    <tr>
      <th>Prepaid Assets</th>
      <td>10341000.0</td>
      <td>12833000.0</td>
      <td>12933000.0</td>
      <td>10248000.0</td>
      <td>10506000.0</td>
    </tr>
    <tr>
      <th>Inventory</th>
      <td>51678000.0</td>
      <td>56803000.0</td>
      <td>64421000.0</td>
      <td>60832000.0</td>
      <td>66173000.0</td>
    </tr>
    <tr>
      <th>Receivables</th>
      <td>174906000.0</td>
      <td>202217000.0</td>
      <td>235339000.0</td>
      <td>202116000.0</td>
      <td>159629000.0</td>
    </tr>
    <tr>
      <th>Other Receivables</th>
      <td>58406000.0</td>
      <td>77444000.0</td>
      <td>80579000.0</td>
      <td>82923000.0</td>
      <td>79425000.0</td>
    </tr>
    <tr>
      <th>Duefrom Related Parties Current</th>
      <td>66000.0</td>
      <td>70000.0</td>
      <td>211000.0</td>
      <td>2486000.0</td>
      <td>2519000.0</td>
    </tr>
    <tr>
      <th>Accounts Receivable</th>
      <td>116434000.0</td>
      <td>124703000.0</td>
      <td>154549000.0</td>
      <td>116707000.0</td>
      <td>77685000.0</td>
    </tr>
    <tr>
      <th>Allowance For Doubtful Accounts Receivable</th>
      <td>NaN</td>
      <td>-8200000.0</td>
      <td>-8600000.0</td>
      <td>-6400000.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Gross Accounts Receivable</th>
      <td>NaN</td>
      <td>132903000.0</td>
      <td>163149000.0</td>
      <td>123107000.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Cash Cash Equivalents And Short Term Investments</th>
      <td>227292000.0</td>
      <td>178850000.0</td>
      <td>169751000.0</td>
      <td>180512000.0</td>
      <td>78008000.0</td>
    </tr>
    <tr>
      <th>Cash And Cash Equivalents</th>
      <td>227292000.0</td>
      <td>178850000.0</td>
      <td>169751000.0</td>
      <td>180512000.0</td>
      <td>78008000.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

**Quarter Cash Flow**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
cash_flow_df = ticker.quarterly_cashflow
cash_flow_df
{% endhighlight %}

</details>


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
      <th>2023-09-30</th>
      <th>2023-06-30</th>
      <th>2023-03-31</th>
      <th>2022-12-31</th>
      <th>2022-09-30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Free Cash Flow</th>
      <td>120229000.0</td>
      <td>200634000.0</td>
      <td>166877000.0</td>
      <td>141174000.0</td>
      <td>42223000.0</td>
    </tr>
    <tr>
      <th>Repayment Of Debt</th>
      <td>-64417000.0</td>
      <td>-177035000.0</td>
      <td>-202649000.0</td>
      <td>-37646000.0</td>
      <td>-115369000.0</td>
    </tr>
    <tr>
      <th>Issuance Of Debt</th>
      <td>0.0</td>
      <td>26000000.0</td>
      <td>25000000.0</td>
      <td>0.0</td>
      <td>59989000.0</td>
    </tr>
    <tr>
      <th>Capital Expenditure</th>
      <td>-3275000.0</td>
      <td>-2258000.0</td>
      <td>-442000.0</td>
      <td>-3919000.0</td>
      <td>-4097000.0</td>
    </tr>
    <tr>
      <th>End Cash Position</th>
      <td>227983000.0</td>
      <td>179553000.0</td>
      <td>176589000.0</td>
      <td>187361000.0</td>
      <td>83873000.0</td>
    </tr>
    <tr>
      <th>Beginning Cash Position</th>
      <td>179553000.0</td>
      <td>176589000.0</td>
      <td>187361000.0</td>
      <td>83873000.0</td>
      <td>71904000.0</td>
    </tr>
    <tr>
      <th>Changes In Cash</th>
      <td>48430000.0</td>
      <td>2964000.0</td>
      <td>-10772000.0</td>
      <td>103488000.0</td>
      <td>11969000.0</td>
    </tr>
    <tr>
      <th>Financing Cash Flow</th>
      <td>-73799000.0</td>
      <td>-199570000.0</td>
      <td>-177649000.0</td>
      <td>-37686000.0</td>
      <td>-55380000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Financing Activities</th>
      <td>-73799000.0</td>
      <td>-199570000.0</td>
      <td>-177649000.0</td>
      <td>-37686000.0</td>
      <td>-55380000.0</td>
    </tr>
    <tr>
      <th>Net Other Financing Charges</th>
      <td>-866000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-40000.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Cash Dividends Paid</th>
      <td>-8516000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Net Issuance Payments Of Debt</th>
      <td>-64417000.0</td>
      <td>-151035000.0</td>
      <td>-177649000.0</td>
      <td>-37646000.0</td>
      <td>-55380000.0</td>
    </tr>
    <tr>
      <th>Net Short Term Debt Issuance</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-15000000.0</td>
    </tr>
    <tr>
      <th>Short Term Debt Payments</th>
      <td>0.0</td>
      <td>-25000000.0</td>
      <td>-25000000.0</td>
      <td>0.0</td>
      <td>-75000000.0</td>
    </tr>
    <tr>
      <th>Short Term Debt Issuance</th>
      <td>0.0</td>
      <td>25000000.0</td>
      <td>25000000.0</td>
      <td>0.0</td>
      <td>60000000.0</td>
    </tr>
    <tr>
      <th>Net Long Term Debt Issuance</th>
      <td>-64417000.0</td>
      <td>-151035000.0</td>
      <td>-177649000.0</td>
      <td>-37646000.0</td>
      <td>-40380000.0</td>
    </tr>
    <tr>
      <th>Long Term Debt Payments</th>
      <td>-64417000.0</td>
      <td>-152035000.0</td>
      <td>-177649000.0</td>
      <td>-37646000.0</td>
      <td>-40369000.0</td>
    </tr>
    <tr>
      <th>Long Term Debt Issuance</th>
      <td>0.0</td>
      <td>1000000.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>-11000.0</td>
    </tr>
    <tr>
      <th>Investing Cash Flow</th>
      <td>-1275000.0</td>
      <td>-358000.0</td>
      <td>-442000.0</td>
      <td>-3919000.0</td>
      <td>21029000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Investing Activities</th>
      <td>-1275000.0</td>
      <td>-358000.0</td>
      <td>-442000.0</td>
      <td>-3919000.0</td>
      <td>21029000.0</td>
    </tr>
    <tr>
      <th>Net Other Investing Changes</th>
      <td>2000000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>69646000.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Net PPE Purchase And Sale</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>25126000.0</td>
    </tr>
    <tr>
      <th>Sale Of PPE</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>25126000.0</td>
    </tr>
    <tr>
      <th>Capital Expenditure Reported</th>
      <td>-3275000.0</td>
      <td>-2258000.0</td>
      <td>-442000.0</td>
      <td>-3919000.0</td>
      <td>-4097000.0</td>
    </tr>
    <tr>
      <th>Operating Cash Flow</th>
      <td>123504000.0</td>
      <td>202892000.0</td>
      <td>167319000.0</td>
      <td>145093000.0</td>
      <td>46320000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Operating Activities</th>
      <td>123504000.0</td>
      <td>202892000.0</td>
      <td>167319000.0</td>
      <td>145093000.0</td>
      <td>46320000.0</td>
    </tr>
    <tr>
      <th>Change In Working Capital</th>
      <td>23596000.0</td>
      <td>19001000.0</td>
      <td>-28946000.0</td>
      <td>-25152000.0</td>
      <td>-29036000.0</td>
    </tr>
    <tr>
      <th>Other Non Cash Items</th>
      <td>-2279000.0</td>
      <td>-1604000.0</td>
      <td>1687000.0</td>
      <td>-539000.0</td>
      <td>-8348000.0</td>
    </tr>
    <tr>
      <th>Asset Impairment Charge</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Deferred Tax</th>
      <td>-3078000.0</td>
      <td>7863000.0</td>
      <td>1781000.0</td>
      <td>985000.0</td>
      <td>903000.0</td>
    </tr>
    <tr>
      <th>Deferred Income Tax</th>
      <td>-3078000.0</td>
      <td>7863000.0</td>
      <td>1781000.0</td>
      <td>985000.0</td>
      <td>903000.0</td>
    </tr>
    <tr>
      <th>Depreciation Amortization Depletion</th>
      <td>24565000.0</td>
      <td>24384000.0</td>
      <td>23975000.0</td>
      <td>24459000.0</td>
      <td>24251000.0</td>
    </tr>
    <tr>
      <th>Depreciation And Amortization</th>
      <td>24565000.0</td>
      <td>24384000.0</td>
      <td>23975000.0</td>
      <td>24459000.0</td>
      <td>24251000.0</td>
    </tr>
    <tr>
      <th>Operating Gains Losses</th>
      <td>-666000.0</td>
      <td>2005000.0</td>
      <td>-546000.0</td>
      <td>-1087000.0</td>
      <td>-9503000.0</td>
    </tr>
    <tr>
      <th>Earnings Losses From Equity Investments</th>
      <td>-666000.0</td>
      <td>-1120000.0</td>
      <td>-1130000.0</td>
      <td>-1708000.0</td>
      <td>-221000.0</td>
    </tr>
    <tr>
      <th>Gain Loss On Investment Securities</th>
      <td>0.0</td>
      <td>3125000.0</td>
      <td>584000.0</td>
      <td>621000.0</td>
      <td>-1126000.0</td>
    </tr>
    <tr>
      <th>Net Income From Continuing Operations</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
  </tbody>
</table>
</div>


<br>

**Quarter Income Statement**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
income_statement_df = ticker.quarterly_income_stmt
income_statement_df
{% endhighlight %}

</details>


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
      <th>2023-09-30</th>
      <th>2023-06-30</th>
      <th>2023-03-31</th>
      <th>2022-12-31</th>
      <th>2022-09-30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Tax Effect Of Unusual Items</th>
      <td>0.0</td>
      <td>138353.172611</td>
      <td>-17894.389747</td>
      <td>-5063.721764</td>
      <td>219180.93562</td>
    </tr>
    <tr>
      <th>Tax Rate For Calcs</th>
      <td>0.21</td>
      <td>0.050959</td>
      <td>0.013294</td>
      <td>0.006116</td>
      <td>0.01832</td>
    </tr>
    <tr>
      <th>Normalized EBITDA</th>
      <td>109843000.0</td>
      <td>186940000.0</td>
      <td>208189000.0</td>
      <td>157822000.0</td>
      <td>90634000.0</td>
    </tr>
    <tr>
      <th>Total Unusual Items</th>
      <td>0.0</td>
      <td>2715000.0</td>
      <td>-1346000.0</td>
      <td>-828000.0</td>
      <td>11964000.0</td>
    </tr>
    <tr>
      <th>Total Unusual Items Excluding Goodwill</th>
      <td>0.0</td>
      <td>2715000.0</td>
      <td>-1346000.0</td>
      <td>-828000.0</td>
      <td>11964000.0</td>
    </tr>
    <tr>
      <th>Net Income From Continuing Operation Net Minority Interest</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Reconciled Depreciation</th>
      <td>24565000.0</td>
      <td>24384000.0</td>
      <td>23975000.0</td>
      <td>24459000.0</td>
      <td>24251000.0</td>
    </tr>
    <tr>
      <th>Reconciled Cost Of Revenue</th>
      <td>193583000.0</td>
      <td>198957000.0</td>
      <td>199289000.0</td>
      <td>200692000.0</td>
      <td>202483000.0</td>
    </tr>
    <tr>
      <th>EBITDA</th>
      <td>109843000.0</td>
      <td>189655000.0</td>
      <td>206843000.0</td>
      <td>156994000.0</td>
      <td>102598000.0</td>
    </tr>
    <tr>
      <th>EBIT</th>
      <td>85278000.0</td>
      <td>165271000.0</td>
      <td>182868000.0</td>
      <td>156994000.0</td>
      <td>78347000.0</td>
    </tr>
    <tr>
      <th>Net Interest Income</th>
      <td>-3321000.0</td>
      <td>-4136000.0</td>
      <td>-8988000.0</td>
      <td>-8746000.0</td>
      <td>-8808000.0</td>
    </tr>
    <tr>
      <th>Interest Expense</th>
      <td>6440000.0</td>
      <td>5907000.0</td>
      <td>11218000.0</td>
      <td>9666000.0</td>
      <td>9024000.0</td>
    </tr>
    <tr>
      <th>Interest Income</th>
      <td>3119000.0</td>
      <td>1771000.0</td>
      <td>2230000.0</td>
      <td>920000.0</td>
      <td>216000.0</td>
    </tr>
    <tr>
      <th>Normalized Income</th>
      <td>81366000.0</td>
      <td>148666353.172611</td>
      <td>170696105.610253</td>
      <td>147249936.278236</td>
      <td>56308180.93562</td>
    </tr>
    <tr>
      <th>Net Income From Continuing And Discontinued Operation</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Total Expenses</th>
      <td>204283000.0</td>
      <td>211075000.0</td>
      <td>211558000.0</td>
      <td>211634000.0</td>
      <td>212170000.0</td>
    </tr>
    <tr>
      <th>Rent Expense Supplemental</th>
      <td>19378000.0</td>
      <td>18691000.0</td>
      <td>12945000.0</td>
      <td>8035000.0</td>
      <td>7236000.0</td>
    </tr>
    <tr>
      <th>Total Operating Income As Reported</th>
      <td>81575000.0</td>
      <td>159571000.0</td>
      <td>181851000.0</td>
      <td>154275000.0</td>
      <td>75372000.0</td>
    </tr>
    <tr>
      <th>Diluted Average Shares</th>
      <td>34571439.0</td>
      <td>34546749.0</td>
      <td>34540269.0</td>
      <td>34491486.0</td>
      <td>34374752.0</td>
    </tr>
    <tr>
      <th>Basic Average Shares</th>
      <td>34201415.0</td>
      <td>34140540.0</td>
      <td>34092504.0</td>
      <td>34081220.0</td>
      <td>34039501.0</td>
    </tr>
    <tr>
      <th>Diluted EPS</th>
      <td>2.35</td>
      <td>4.38</td>
      <td>4.9</td>
      <td>4.25</td>
      <td>1.98</td>
    </tr>
    <tr>
      <th>Basic EPS</th>
      <td>2.38</td>
      <td>4.43</td>
      <td>4.97</td>
      <td>4.3</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>Diluted NI Availto Com Stockholders</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Net Income Common Stockholders</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Net Income</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Net Income Including Noncontrolling Interests</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Net Income Continuous Operations</th>
      <td>81366000.0</td>
      <td>151243000.0</td>
      <td>169368000.0</td>
      <td>146427000.0</td>
      <td>68053000.0</td>
    </tr>
    <tr>
      <th>Tax Provision</th>
      <td>-2528000.0</td>
      <td>8121000.0</td>
      <td>2282000.0</td>
      <td>901000.0</td>
      <td>1270000.0</td>
    </tr>
    <tr>
      <th>Pretax Income</th>
      <td>78838000.0</td>
      <td>159364000.0</td>
      <td>171650000.0</td>
      <td>147328000.0</td>
      <td>69323000.0</td>
    </tr>
    <tr>
      <th>Other Income Expense</th>
      <td>584000.0</td>
      <td>3929000.0</td>
      <td>-2461000.0</td>
      <td>390000.0</td>
      <td>10915000.0</td>
    </tr>
    <tr>
      <th>Other Non Operating Income Expenses</th>
      <td>-82000.0</td>
      <td>94000.0</td>
      <td>-2245000.0</td>
      <td>-490000.0</td>
      <td>-1270000.0</td>
    </tr>
    <tr>
      <th>Special Income Charges</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>-1248000.0</td>
      <td>-1409000.0</td>
      <td>8156000.0</td>
    </tr>
    <tr>
      <th>Gain On Sale Of Ppe</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>8156000.0</td>
    </tr>
    <tr>
      <th>Impairment Of Capital Assets</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Restructuring And Mergern Acquisition</th>
      <td>0.0</td>
      <td>0.0</td>
      <td>1248000.0</td>
      <td>1409000.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Earnings From Equity Interest</th>
      <td>666000.0</td>
      <td>1120000.0</td>
      <td>1130000.0</td>
      <td>1708000.0</td>
      <td>221000.0</td>
    </tr>
    <tr>
      <th>Gain On Sale Of Security</th>
      <td>NaN</td>
      <td>2715000.0</td>
      <td>-98000.0</td>
      <td>581000.0</td>
      <td>3808000.0</td>
    </tr>
    <tr>
      <th>Net Non Operating Interest Income Expense</th>
      <td>-3321000.0</td>
      <td>-4136000.0</td>
      <td>-8988000.0</td>
      <td>-8746000.0</td>
      <td>-8808000.0</td>
    </tr>
    <tr>
      <th>Interest Expense Non Operating</th>
      <td>6440000.0</td>
      <td>5907000.0</td>
      <td>11218000.0</td>
      <td>9666000.0</td>
      <td>9024000.0</td>
    </tr>
    <tr>
      <th>Interest Income Non Operating</th>
      <td>3119000.0</td>
      <td>1771000.0</td>
      <td>2230000.0</td>
      <td>920000.0</td>
      <td>216000.0</td>
    </tr>
    <tr>
      <th>Operating Income</th>
      <td>81575000.0</td>
      <td>159571000.0</td>
      <td>183099000.0</td>
      <td>155684000.0</td>
      <td>67216000.0</td>
    </tr>
    <tr>
      <th>Operating Expense</th>
      <td>10700000.0</td>
      <td>12118000.0</td>
      <td>12269000.0</td>
      <td>10942000.0</td>
      <td>9687000.0</td>
    </tr>
    <tr>
      <th>Selling General And Administration</th>
      <td>10700000.0</td>
      <td>12118000.0</td>
      <td>12269000.0</td>
      <td>10942000.0</td>
      <td>9687000.0</td>
    </tr>
    <tr>
      <th>General And Administrative Expense</th>
      <td>10700000.0</td>
      <td>12118000.0</td>
      <td>12269000.0</td>
      <td>10942000.0</td>
      <td>9687000.0</td>
    </tr>
    <tr>
      <th>Other Gand A</th>
      <td>10700000.0</td>
      <td>12118000.0</td>
      <td>12269000.0</td>
      <td>10942000.0</td>
      <td>9687000.0</td>
    </tr>
    <tr>
      <th>Gross Profit</th>
      <td>92275000.0</td>
      <td>171689000.0</td>
      <td>195368000.0</td>
      <td>166626000.0</td>
      <td>76903000.0</td>
    </tr>
    <tr>
      <th>Cost Of Revenue</th>
      <td>193583000.0</td>
      <td>198957000.0</td>
      <td>199289000.0</td>
      <td>200692000.0</td>
      <td>202483000.0</td>
    </tr>
    <tr>
      <th>Total Revenue</th>
      <td>285858000.0</td>
      <td>370646000.0</td>
      <td>394657000.0</td>
      <td>367318000.0</td>
      <td>279386000.0</td>
    </tr>
    <tr>
      <th>Operating Revenue</th>
      <td>282893000.0</td>
      <td>367961000.0</td>
      <td>391754000.0</td>
      <td>365055000.0</td>
      <td>277737000.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

**Annual Income Statement**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
annual_income = ticker.income_stmt
annual_income
{% endhighlight %}

</details>



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
      <th>2022-12-31</th>
      <th>2021-12-31</th>
      <th>2020-12-31</th>
      <th>2019-12-31</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Tax Effect Of Unusual Items</th>
      <td>28210.722296</td>
      <td>-725629.746667</td>
      <td>-5625001.183932</td>
      <td>-1757970.0</td>
    </tr>
    <tr>
      <th>Tax Rate For Calcs</th>
      <td>0.002304</td>
      <td>0.007904</td>
      <td>0.076987</td>
      <td>0.27</td>
    </tr>
    <tr>
      <th>Normalized EBITDA</th>
      <td>352143000.0</td>
      <td>-11384000.0</td>
      <td>336401000.0</td>
      <td>237237000.0</td>
    </tr>
    <tr>
      <th>Total Unusual Items</th>
      <td>12245000.0</td>
      <td>-91804000.0</td>
      <td>-73064000.0</td>
      <td>-6511000.0</td>
    </tr>
    <tr>
      <th>Total Unusual Items Excluding Goodwill</th>
      <td>12245000.0</td>
      <td>-91804000.0</td>
      <td>-73064000.0</td>
      <td>-6511000.0</td>
    </tr>
    <tr>
      <th>Net Income From Continuing Operation Net Minority Interest</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Reconciled Depreciation</th>
      <td>99033000.0</td>
      <td>106084000.0</td>
      <td>117212000.0</td>
      <td>124002000.0</td>
    </tr>
    <tr>
      <th>Reconciled Cost Of Revenue</th>
      <td>772459000.0</td>
      <td>600379000.0</td>
      <td>635011000.0</td>
      <td>756356000.0</td>
    </tr>
    <tr>
      <th>EBITDA</th>
      <td>364388000.0</td>
      <td>-103188000.0</td>
      <td>263337000.0</td>
      <td>230726000.0</td>
    </tr>
    <tr>
      <th>EBIT</th>
      <td>265355000.0</td>
      <td>-209272000.0</td>
      <td>146125000.0</td>
      <td>106724000.0</td>
    </tr>
    <tr>
      <th>Net Interest Income</th>
      <td>-34402000.0</td>
      <td>-34909000.0</td>
      <td>-50326000.0</td>
      <td>-64491000.0</td>
    </tr>
    <tr>
      <th>Interest Expense</th>
      <td>35740000.0</td>
      <td>35031000.0</td>
      <td>51525000.0</td>
      <td>65362000.0</td>
    </tr>
    <tr>
      <th>Interest Income</th>
      <td>1338000.0</td>
      <td>122000.0</td>
      <td>1199000.0</td>
      <td>871000.0</td>
    </tr>
    <tr>
      <th>Normalized Income</th>
      <td>216869210.722296</td>
      <td>-151293629.746667</td>
      <td>154755998.816068</td>
      <td>46115030.0</td>
    </tr>
    <tr>
      <th>Net Income From Continuing And Discontinued Operation</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Total Expenses</th>
      <td>814228000.0</td>
      <td>644094000.0</td>
      <td>674017000.0</td>
      <td>811643000.0</td>
    </tr>
    <tr>
      <th>Rent Expense Supplemental</th>
      <td>27374000.0</td>
      <td>13799000.0</td>
      <td>36341000.0</td>
      <td>43189000.0</td>
    </tr>
    <tr>
      <th>Total Operating Income As Reported</th>
      <td>255949000.0</td>
      <td>-194095000.0</td>
      <td>141573000.0</td>
      <td>122663000.0</td>
    </tr>
    <tr>
      <th>Diluted Average Shares</th>
      <td>34287075.0</td>
      <td>33859306.0</td>
      <td>33921621.0</td>
      <td>33731171.0</td>
    </tr>
    <tr>
      <th>Basic Average Shares</th>
      <td>33997579.0</td>
      <td>33859306.0</td>
      <td>33718665.0</td>
      <td>33617635.0</td>
    </tr>
    <tr>
      <th>Diluted EPS</th>
      <td>6.68</td>
      <td>-7.16</td>
      <td>2.57</td>
      <td>1.23</td>
    </tr>
    <tr>
      <th>Basic EPS</th>
      <td>6.74</td>
      <td>-7.16</td>
      <td>2.59</td>
      <td>1.23</td>
    </tr>
    <tr>
      <th>Diluted NI Availto Com Stockholders</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Net Income Common Stockholders</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Net Income</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Net Income Including Noncontrolling Interests</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Net Income Continuous Operations</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Tax Provision</th>
      <td>529000.0</td>
      <td>-1931000.0</td>
      <td>7283000.0</td>
      <td>20103000.0</td>
    </tr>
    <tr>
      <th>Pretax Income</th>
      <td>229615000.0</td>
      <td>-244303000.0</td>
      <td>94600000.0</td>
      <td>41362000.0</td>
    </tr>
    <tr>
      <th>Other Income Expense</th>
      <td>15134000.0</td>
      <td>-107667000.0</td>
      <td>-67491000.0</td>
      <td>-3471000.0</td>
    </tr>
    <tr>
      <th>Other Non Operating Income Expenses</th>
      <td>2645000.0</td>
      <td>-1756000.0</td>
      <td>473000.0</td>
      <td>695000.0</td>
    </tr>
    <tr>
      <th>Special Income Charges</th>
      <td>7066000.0</td>
      <td>-92368000.0</td>
      <td>-70844000.0</td>
      <td>-5544000.0</td>
    </tr>
    <tr>
      <th>Impairment Of Capital Assets</th>
      <td>-8888000.0</td>
      <td>92368000.0</td>
      <td>69446000.0</td>
      <td>5544000.0</td>
    </tr>
    <tr>
      <th>Restructuring And Mergern Acquisition</th>
      <td>1822000.0</td>
      <td>0.0</td>
      <td>1398000.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Earnings From Equity Interest</th>
      <td>244000.0</td>
      <td>-14107000.0</td>
      <td>5100000.0</td>
      <td>2345000.0</td>
    </tr>
    <tr>
      <th>Gain On Sale Of Security</th>
      <td>5179000.0</td>
      <td>564000.0</td>
      <td>-2220000.0</td>
      <td>-967000.0</td>
    </tr>
    <tr>
      <th>Net Non Operating Interest Income Expense</th>
      <td>-34402000.0</td>
      <td>-34909000.0</td>
      <td>-50326000.0</td>
      <td>-64491000.0</td>
    </tr>
    <tr>
      <th>Interest Expense Non Operating</th>
      <td>35740000.0</td>
      <td>35031000.0</td>
      <td>51525000.0</td>
      <td>65362000.0</td>
    </tr>
    <tr>
      <th>Interest Income Non Operating</th>
      <td>1338000.0</td>
      <td>122000.0</td>
      <td>1199000.0</td>
      <td>871000.0</td>
    </tr>
    <tr>
      <th>Operating Income</th>
      <td>248883000.0</td>
      <td>-101727000.0</td>
      <td>212417000.0</td>
      <td>109324000.0</td>
    </tr>
    <tr>
      <th>Operating Expense</th>
      <td>41769000.0</td>
      <td>43715000.0</td>
      <td>39006000.0</td>
      <td>55287000.0</td>
    </tr>
    <tr>
      <th>Other Taxes</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>18883000.0</td>
    </tr>
    <tr>
      <th>Selling General And Administration</th>
      <td>41769000.0</td>
      <td>43715000.0</td>
      <td>39006000.0</td>
      <td>36404000.0</td>
    </tr>
    <tr>
      <th>General And Administrative Expense</th>
      <td>41769000.0</td>
      <td>43715000.0</td>
      <td>39006000.0</td>
      <td>36404000.0</td>
    </tr>
    <tr>
      <th>Other Gand A</th>
      <td>41769000.0</td>
      <td>43715000.0</td>
      <td>39006000.0</td>
      <td>36404000.0</td>
    </tr>
    <tr>
      <th>Gross Profit</th>
      <td>290652000.0</td>
      <td>-58012000.0</td>
      <td>251423000.0</td>
      <td>164611000.0</td>
    </tr>
    <tr>
      <th>Cost Of Revenue</th>
      <td>772459000.0</td>
      <td>600379000.0</td>
      <td>635011000.0</td>
      <td>756356000.0</td>
    </tr>
    <tr>
      <th>Total Revenue</th>
      <td>1063111000.0</td>
      <td>542367000.0</td>
      <td>886434000.0</td>
      <td>920967000.0</td>
    </tr>
    <tr>
      <th>Operating Revenue</th>
      <td>1054000000.0</td>
      <td>532055000.0</td>
      <td>869402000.0</td>
      <td>877368000.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

**Annual Cash Flow**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
annual_cash_flow = ticker.cashflow
annual_cash_flow
{% endhighlight %}

</details>


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
      <th>2022-12-31</th>
      <th>2021-12-31</th>
      <th>2020-12-31</th>
      <th>2019-12-31</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Free Cash Flow</th>
      <td>177835000.0</td>
      <td>-130889000.0</td>
      <td>331918000.0</td>
      <td>106033000.0</td>
    </tr>
    <tr>
      <th>Repayment Of Debt</th>
      <td>-534142000.0</td>
      <td>-387087000.0</td>
      <td>-1195818000.0</td>
      <td>-410438000.0</td>
    </tr>
    <tr>
      <th>Issuance Of Debt</th>
      <td>422108000.0</td>
      <td>411393000.0</td>
      <td>809872000.0</td>
      <td>320806000.0</td>
    </tr>
    <tr>
      <th>Issuance Of Capital Stock</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Capital Expenditure</th>
      <td>-15430000.0</td>
      <td>-21447000.0</td>
      <td>-16025000.0</td>
      <td>-11628000.0</td>
    </tr>
    <tr>
      <th>End Cash Position</th>
      <td>187361000.0</td>
      <td>55928000.0</td>
      <td>103146000.0</td>
      <td>96790000.0</td>
    </tr>
    <tr>
      <th>Beginning Cash Position</th>
      <td>55928000.0</td>
      <td>103146000.0</td>
      <td>96790000.0</td>
      <td>60507000.0</td>
    </tr>
    <tr>
      <th>Changes In Cash</th>
      <td>131433000.0</td>
      <td>-47218000.0</td>
      <td>6356000.0</td>
      <td>36283000.0</td>
    </tr>
    <tr>
      <th>Financing Cash Flow</th>
      <td>-113048000.0</td>
      <td>24081000.0</td>
      <td>-416104000.0</td>
      <td>-89758000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Financing Activities</th>
      <td>-113048000.0</td>
      <td>24081000.0</td>
      <td>-416104000.0</td>
      <td>-89758000.0</td>
    </tr>
    <tr>
      <th>Net Other Financing Charges</th>
      <td>-1014000.0</td>
      <td>-225000.0</td>
      <td>-30158000.0</td>
      <td>-126000.0</td>
    </tr>
    <tr>
      <th>Cash Dividends Paid</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Common Stock Dividend Paid</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Net Common Stock Issuance</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Common Stock Issuance</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Net Issuance Payments Of Debt</th>
      <td>-112034000.0</td>
      <td>24306000.0</td>
      <td>-385946000.0</td>
      <td>-89632000.0</td>
    </tr>
    <tr>
      <th>Net Short Term Debt Issuance</th>
      <td>-25000000.0</td>
      <td>15000000.0</td>
      <td>-40000000.0</td>
      <td>50000000.0</td>
    </tr>
    <tr>
      <th>Short Term Debt Payments</th>
      <td>-159000000.0</td>
      <td>-35000000.0</td>
      <td>-275000000.0</td>
      <td>-150000000.0</td>
    </tr>
    <tr>
      <th>Short Term Debt Issuance</th>
      <td>134000000.0</td>
      <td>50000000.0</td>
      <td>235000000.0</td>
      <td>200000000.0</td>
    </tr>
    <tr>
      <th>Net Long Term Debt Issuance</th>
      <td>-87034000.0</td>
      <td>9306000.0</td>
      <td>-345946000.0</td>
      <td>-139632000.0</td>
    </tr>
    <tr>
      <th>Long Term Debt Payments</th>
      <td>-375142000.0</td>
      <td>-352087000.0</td>
      <td>-920818000.0</td>
      <td>-260438000.0</td>
    </tr>
    <tr>
      <th>Long Term Debt Issuance</th>
      <td>288108000.0</td>
      <td>361393000.0</td>
      <td>574872000.0</td>
      <td>120806000.0</td>
    </tr>
    <tr>
      <th>Investing Cash Flow</th>
      <td>51216000.0</td>
      <td>38143000.0</td>
      <td>74517000.0</td>
      <td>8380000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Investing Activities</th>
      <td>51216000.0</td>
      <td>38143000.0</td>
      <td>74517000.0</td>
      <td>8380000.0</td>
    </tr>
    <tr>
      <th>Net Other Investing Changes</th>
      <td>66646000.0</td>
      <td>59590000.0</td>
      <td>4650000.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Net Business Purchase And Sale</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Sale Of Business</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>Net PPE Purchase And Sale</th>
      <td>69646000.0</td>
      <td>-21447000.0</td>
      <td>85892000.0</td>
      <td>20008000.0</td>
    </tr>
    <tr>
      <th>Sale Of PPE</th>
      <td>69646000.0</td>
      <td>58090000.0</td>
      <td>85892000.0</td>
      <td>20008000.0</td>
    </tr>
    <tr>
      <th>Purchase Of PPE</th>
      <td>NaN</td>
      <td>-21447000.0</td>
      <td>-16025000.0</td>
      <td>-11628000.0</td>
    </tr>
    <tr>
      <th>Capital Expenditure Reported</th>
      <td>-15430000.0</td>
      <td>-21447000.0</td>
      <td>-16025000.0</td>
      <td>-11628000.0</td>
    </tr>
    <tr>
      <th>Operating Cash Flow</th>
      <td>193265000.0</td>
      <td>-109442000.0</td>
      <td>347943000.0</td>
      <td>117661000.0</td>
    </tr>
    <tr>
      <th>Cash Flow From Continuing Operating Activities</th>
      <td>193265000.0</td>
      <td>-109442000.0</td>
      <td>347943000.0</td>
      <td>117661000.0</td>
    </tr>
    <tr>
      <th>Change In Working Capital</th>
      <td>-112224000.0</td>
      <td>-50116000.0</td>
      <td>89920000.0</td>
      <td>-30038000.0</td>
    </tr>
    <tr>
      <th>Change In Other Working Capital</th>
      <td>651000.0</td>
      <td>-2039000.0</td>
      <td>-10941000.0</td>
      <td>7091000.0</td>
    </tr>
    <tr>
      <th>Change In Payables And Accrued Expense</th>
      <td>11262000.0</td>
      <td>-6113000.0</td>
      <td>-49056000.0</td>
      <td>66813000.0</td>
    </tr>
    <tr>
      <th>Change In Payable</th>
      <td>11262000.0</td>
      <td>-6113000.0</td>
      <td>-49056000.0</td>
      <td>66813000.0</td>
    </tr>
    <tr>
      <th>Change In Account Payable</th>
      <td>21065000.0</td>
      <td>-13934000.0</td>
      <td>-50040000.0</td>
      <td>83244000.0</td>
    </tr>
    <tr>
      <th>Change In Prepaid Assets</th>
      <td>-25000.0</td>
      <td>-484000.0</td>
      <td>532000.0</td>
      <td>119000.0</td>
    </tr>
    <tr>
      <th>Change In Inventory</th>
      <td>-11804000.0</td>
      <td>-17619000.0</td>
      <td>13634000.0</td>
      <td>-28628000.0</td>
    </tr>
    <tr>
      <th>Change In Receivables</th>
      <td>-112308000.0</td>
      <td>-21730000.0</td>
      <td>135751000.0</td>
      <td>-75827000.0</td>
    </tr>
    <tr>
      <th>Changes In Account Receivables</th>
      <td>-114042000.0</td>
      <td>-22746000.0</td>
      <td>140290000.0</td>
      <td>-114793000.0</td>
    </tr>
    <tr>
      <th>Other Non Cash Items</th>
      <td>-10206000.0</td>
      <td>-24972000.0</td>
      <td>-19423000.0</td>
      <td>-26111000.0</td>
    </tr>
    <tr>
      <th>Asset Impairment Charge</th>
      <td>-8888000.0</td>
      <td>92368000.0</td>
      <td>69446000.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Deferred Tax</th>
      <td>-129000.0</td>
      <td>-3109000.0</td>
      <td>7113000.0</td>
      <td>18489000.0</td>
    </tr>
    <tr>
      <th>Deferred Income Tax</th>
      <td>-129000.0</td>
      <td>-3109000.0</td>
      <td>7113000.0</td>
      <td>18489000.0</td>
    </tr>
    <tr>
      <th>Depreciation Amortization Depletion</th>
      <td>99033000.0</td>
      <td>106084000.0</td>
      <td>117212000.0</td>
      <td>124002000.0</td>
    </tr>
    <tr>
      <th>Depreciation And Amortization</th>
      <td>99033000.0</td>
      <td>106084000.0</td>
      <td>117212000.0</td>
      <td>124002000.0</td>
    </tr>
    <tr>
      <th>Operating Gains Losses</th>
      <td>-12295000.0</td>
      <td>12675000.0</td>
      <td>-3642000.0</td>
      <td>8446000.0</td>
    </tr>
    <tr>
      <th>Earnings Losses From Equity Investments</th>
      <td>-244000.0</td>
      <td>14107000.0</td>
      <td>-5100000.0</td>
      <td>-2345000.0</td>
    </tr>
    <tr>
      <th>Gain Loss On Investment Securities</th>
      <td>-3163000.0</td>
      <td>-1432000.0</td>
      <td>1458000.0</td>
      <td>5247000.0</td>
    </tr>
    <tr>
      <th>Gain Loss On Sale Of PPE</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5544000.0</td>
    </tr>
    <tr>
      <th>Net Income From Continuing Operations</th>
      <td>229086000.0</td>
      <td>-242372000.0</td>
      <td>87317000.0</td>
      <td>41362000.0</td>
    </tr>
  </tbody>
</table>
</div>

<br>

**Annual Balance Sheet**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
annual_balance_sheet = ticker.balance_sheet
annual_balance_sheet = annual_balance_sheet.fillna(0)
annual_balance_sheet
{% endhighlight %}

</details>



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
      <th>2022-12-31</th>
      <th>2021-12-31</th>
      <th>2020-12-31</th>
      <th>2019-12-31</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Ordinary Shares Number</th>
      <td>3.393894e+07</td>
      <td>3.378851e+07</td>
      <td>3.373814e+07</td>
      <td>3.365458e+07</td>
    </tr>
    <tr>
      <th>Share Issued</th>
      <td>3.393894e+07</td>
      <td>3.378851e+07</td>
      <td>3.373814e+07</td>
      <td>3.365458e+07</td>
    </tr>
    <tr>
      <th>Net Debt</th>
      <td>0.000000e+00</td>
      <td>2.947190e+08</td>
      <td>1.557290e+08</td>
      <td>5.208550e+08</td>
    </tr>
    <tr>
      <th>Total Debt</th>
      <td>5.762030e+08</td>
      <td>6.540290e+08</td>
      <td>6.170040e+08</td>
      <td>1.044027e+09</td>
    </tr>
    <tr>
      <th>Tangible Book Value</th>
      <td>1.066529e+09</td>
      <td>8.344920e+08</td>
      <td>1.074487e+09</td>
      <td>9.849490e+08</td>
    </tr>
    <tr>
      <th>Invested Capital</th>
      <td>1.070006e+09</td>
      <td>3.452910e+08</td>
      <td>2.529610e+08</td>
      <td>6.096790e+08</td>
    </tr>
    <tr>
      <th>Working Capital</th>
      <td>2.893350e+08</td>
      <td>8.830700e+07</td>
      <td>4.828900e+07</td>
      <td>1.415200e+08</td>
    </tr>
    <tr>
      <th>Net Tangible Assets</th>
      <td>1.066529e+09</td>
      <td>8.344920e+08</td>
      <td>1.074487e+09</td>
      <td>9.849490e+08</td>
    </tr>
    <tr>
      <th>Capital Lease Obligations</th>
      <td>5.762030e+08</td>
      <td>3.087380e+08</td>
      <td>3.640430e+08</td>
      <td>4.343480e+08</td>
    </tr>
    <tr>
      <th>Common Stock Equity</th>
      <td>1.070006e+09</td>
      <td>8.384120e+08</td>
      <td>1.078902e+09</td>
      <td>9.899200e+08</td>
    </tr>
    <tr>
      <th>Total Capitalization</th>
      <td>1.070006e+09</td>
      <td>1.143203e+09</td>
      <td>1.311005e+09</td>
      <td>1.506026e+09</td>
    </tr>
    <tr>
      <th>Total Equity Gross Minority Interest</th>
      <td>1.070006e+09</td>
      <td>8.384120e+08</td>
      <td>1.078902e+09</td>
      <td>9.899200e+08</td>
    </tr>
    <tr>
      <th>Stockholders Equity</th>
      <td>1.070006e+09</td>
      <td>8.384120e+08</td>
      <td>1.078902e+09</td>
      <td>9.899200e+08</td>
    </tr>
    <tr>
      <th>Retained Earnings</th>
      <td>-2.336040e+08</td>
      <td>-4.626900e+08</td>
      <td>-2.203180e+08</td>
      <td>-3.076350e+08</td>
    </tr>
    <tr>
      <th>Capital Stock</th>
      <td>1.303610e+09</td>
      <td>1.301102e+09</td>
      <td>1.299220e+09</td>
      <td>1.297555e+09</td>
    </tr>
    <tr>
      <th>Common Stock</th>
      <td>1.303610e+09</td>
      <td>1.301102e+09</td>
      <td>1.299220e+09</td>
      <td>1.297555e+09</td>
    </tr>
    <tr>
      <th>Total Liabilities Net Minority Interest</th>
      <td>7.136480e+08</td>
      <td>7.803370e+08</td>
      <td>7.613430e+08</td>
      <td>1.248136e+09</td>
    </tr>
    <tr>
      <th>Total Non Current Liabilities Net Minority Interest</th>
      <td>5.434740e+08</td>
      <td>6.232490e+08</td>
      <td>5.642240e+08</td>
      <td>9.598510e+08</td>
    </tr>
    <tr>
      <th>Other Non Current Liabilities</th>
      <td>4.401700e+07</td>
      <td>4.614100e+07</td>
      <td>4.964200e+07</td>
      <td>5.104400e+07</td>
    </tr>
    <tr>
      <th>Derivative Product Liabilities</th>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>5.970000e+05</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>Long Term Debt And Capital Lease Obligation</th>
      <td>4.994570e+08</td>
      <td>5.771080e+08</td>
      <td>5.139850e+08</td>
      <td>9.088070e+08</td>
    </tr>
    <tr>
      <th>Long Term Capital Lease Obligation</th>
      <td>4.994570e+08</td>
      <td>2.723170e+08</td>
      <td>2.818820e+08</td>
      <td>3.927010e+08</td>
    </tr>
    <tr>
      <th>Long Term Debt</th>
      <td>0.000000e+00</td>
      <td>3.047910e+08</td>
      <td>2.321030e+08</td>
      <td>5.161060e+08</td>
    </tr>
    <tr>
      <th>Current Liabilities</th>
      <td>1.701740e+08</td>
      <td>1.570880e+08</td>
      <td>1.971190e+08</td>
      <td>2.882850e+08</td>
    </tr>
    <tr>
      <th>Other Current Liabilities</th>
      <td>2.468000e+06</td>
      <td>1.808000e+06</td>
      <td>4.863000e+06</td>
      <td>1.163300e+07</td>
    </tr>
    <tr>
      <th>Current Debt And Capital Lease Obligation</th>
      <td>7.674600e+07</td>
      <td>7.692100e+07</td>
      <td>1.030190e+08</td>
      <td>1.352200e+08</td>
    </tr>
    <tr>
      <th>Current Capital Lease Obligation</th>
      <td>7.674600e+07</td>
      <td>3.642100e+07</td>
      <td>8.216100e+07</td>
      <td>4.164700e+07</td>
    </tr>
    <tr>
      <th>Current Debt</th>
      <td>0.000000e+00</td>
      <td>4.050000e+07</td>
      <td>2.085800e+07</td>
      <td>9.357300e+07</td>
    </tr>
    <tr>
      <th>Other Current Borrowings</th>
      <td>0.000000e+00</td>
      <td>4.050000e+07</td>
      <td>2.085800e+07</td>
      <td>9.357300e+07</td>
    </tr>
    <tr>
      <th>Line Of Credit</th>
      <td>0.000000e+00</td>
      <td>2.500000e+07</td>
      <td>0.000000e+00</td>
      <td>5.000000e+07</td>
    </tr>
    <tr>
      <th>Payables And Accrued Expenses</th>
      <td>9.096000e+07</td>
      <td>7.835900e+07</td>
      <td>8.923700e+07</td>
      <td>1.414320e+08</td>
    </tr>
    <tr>
      <th>Current Accrued Expenses</th>
      <td>4.746900e+07</td>
      <td>3.258300e+07</td>
      <td>5.505500e+07</td>
      <td>5.973500e+07</td>
    </tr>
    <tr>
      <th>Interest Payable</th>
      <td>1.976000e+06</td>
      <td>1.951000e+06</td>
      <td>2.814000e+06</td>
      <td>2.610000e+06</td>
    </tr>
    <tr>
      <th>Payables</th>
      <td>9.096000e+07</td>
      <td>7.835900e+07</td>
      <td>8.923700e+07</td>
      <td>1.414320e+08</td>
    </tr>
    <tr>
      <th>Dueto Related Parties Current</th>
      <td>1.141000e+06</td>
      <td>1.094400e+07</td>
      <td>3.123000e+06</td>
      <td>2.139000e+06</td>
    </tr>
    <tr>
      <th>Accounts Payable</th>
      <td>8.981900e+07</td>
      <td>6.741500e+07</td>
      <td>8.611400e+07</td>
      <td>1.392930e+08</td>
    </tr>
    <tr>
      <th>Total Assets</th>
      <td>1.783654e+09</td>
      <td>1.618749e+09</td>
      <td>1.840245e+09</td>
      <td>2.238056e+09</td>
    </tr>
    <tr>
      <th>Total Non Current Assets</th>
      <td>1.324145e+09</td>
      <td>1.373354e+09</td>
      <td>1.594837e+09</td>
      <td>1.808251e+09</td>
    </tr>
    <tr>
      <th>Other Non Current Assets</th>
      <td>8.208000e+06</td>
      <td>5.225000e+06</td>
      <td>4.032000e+06</td>
      <td>5.360000e+06</td>
    </tr>
    <tr>
      <th>Financial Assets</th>
      <td>1.622000e+06</td>
      <td>6.680000e+05</td>
      <td>0.000000e+00</td>
      <td>8.200000e+04</td>
    </tr>
    <tr>
      <th>Investments And Advances</th>
      <td>1.619800e+07</td>
      <td>1.295400e+07</td>
      <td>2.856100e+07</td>
      <td>2.811200e+07</td>
    </tr>
    <tr>
      <th>Long Term Equity Investment</th>
      <td>1.619800e+07</td>
      <td>1.295400e+07</td>
      <td>2.856100e+07</td>
      <td>2.811200e+07</td>
    </tr>
    <tr>
      <th>Investmentsin Joint Venturesat Cost</th>
      <td>1.619800e+07</td>
      <td>1.295400e+07</td>
      <td>2.856100e+07</td>
      <td>2.811200e+07</td>
    </tr>
    <tr>
      <th>Goodwill And Other Intangible Assets</th>
      <td>3.477000e+06</td>
      <td>3.920000e+06</td>
      <td>4.415000e+06</td>
      <td>4.971000e+06</td>
    </tr>
    <tr>
      <th>Other Intangible Assets</th>
      <td>1.051000e+06</td>
      <td>1.494000e+06</td>
      <td>1.989000e+06</td>
      <td>2.545000e+06</td>
    </tr>
    <tr>
      <th>Goodwill</th>
      <td>2.426000e+06</td>
      <td>2.426000e+06</td>
      <td>2.426000e+06</td>
      <td>2.426000e+06</td>
    </tr>
    <tr>
      <th>Net PPE</th>
      <td>1.296262e+09</td>
      <td>1.351255e+09</td>
      <td>1.557829e+09</td>
      <td>1.769726e+09</td>
    </tr>
    <tr>
      <th>Accumulated Depreciation</th>
      <td>-4.618000e+08</td>
      <td>-3.848000e+08</td>
      <td>-5.418000e+08</td>
      <td>-6.808000e+08</td>
    </tr>
    <tr>
      <th>Gross PPE</th>
      <td>1.296262e+09</td>
      <td>1.351255e+09</td>
      <td>1.557829e+09</td>
      <td>1.769726e+09</td>
    </tr>
    <tr>
      <th>Other Properties</th>
      <td>4.289400e+07</td>
      <td>1.425700e+07</td>
      <td>2.529000e+06</td>
      <td>1.956000e+07</td>
    </tr>
    <tr>
      <th>Current Assets</th>
      <td>4.595090e+08</td>
      <td>2.453950e+08</td>
      <td>2.454080e+08</td>
      <td>4.298050e+08</td>
    </tr>
    <tr>
      <th>Hedging Assets Current</th>
      <td>2.087000e+06</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>5.770000e+05</td>
    </tr>
    <tr>
      <th>Assets Held For Sale Current</th>
      <td>0.000000e+00</td>
      <td>4.354300e+07</td>
      <td>3.297400e+07</td>
      <td>6.545800e+07</td>
    </tr>
    <tr>
      <th>Restricted Cash</th>
      <td>3.714000e+06</td>
      <td>2.221000e+06</td>
      <td>2.779000e+06</td>
      <td>3.071000e+06</td>
    </tr>
    <tr>
      <th>Prepaid Assets</th>
      <td>1.024800e+07</td>
      <td>1.022300e+07</td>
      <td>9.739000e+06</td>
      <td>1.028800e+07</td>
    </tr>
    <tr>
      <th>Inventory</th>
      <td>6.083200e+07</td>
      <td>4.902800e+07</td>
      <td>3.460600e+07</td>
      <td>4.979000e+07</td>
    </tr>
    <tr>
      <th>Receivables</th>
      <td>2.021160e+08</td>
      <td>8.980800e+07</td>
      <td>6.807800e+07</td>
      <td>2.117970e+08</td>
    </tr>
    <tr>
      <th>Other Receivables</th>
      <td>8.292300e+07</td>
      <td>4.450300e+07</td>
      <td>2.664000e+07</td>
      <td>1.068720e+08</td>
    </tr>
    <tr>
      <th>Duefrom Related Parties Current</th>
      <td>2.486000e+06</td>
      <td>4.220000e+06</td>
      <td>5.236000e+06</td>
      <td>6.970000e+05</td>
    </tr>
    <tr>
      <th>Accounts Receivable</th>
      <td>1.167070e+08</td>
      <td>4.108500e+07</td>
      <td>3.620200e+07</td>
      <td>1.042280e+08</td>
    </tr>
    <tr>
      <th>Allowance For Doubtful Accounts Receivable</th>
      <td>-6.400000e+06</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>Gross Accounts Receivable</th>
      <td>1.231070e+08</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>Cash Cash Equivalents And Short Term Investments</th>
      <td>1.805120e+08</td>
      <td>5.057200e+07</td>
      <td>9.723200e+07</td>
      <td>8.882400e+07</td>
    </tr>
    <tr>
      <th>Cash And Cash Equivalents</th>
      <td>1.805120e+08</td>
      <td>5.057200e+07</td>
      <td>9.723200e+07</td>
      <td>8.882400e+07</td>
    </tr>
  </tbody>
</table>
</div>


---


- **FCFF Calculation**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# FCFF Calculation using Cash Flow Statement and Income Statement Inputs
free_cash_flow_firm = (cash_flow_df.loc['Free Cash Flow'].astype(int)) \
                    + (income_statement_df.loc['Interest Expense'].astype(int) \
                        * (1 - income_statement_df.loc['Tax Provision'].astype(int) \
                           / income_statement_df.loc['Pretax Income'].astype(int))).astype(int)

# Change Series to a Pandas Dataframe
free_cash_flow_firm_df = free_cash_flow_firm.to_frame().transpose()
print(free_cashflow_firm_df)
{% endhighlight %}

</details>

       2023-09-30  2023-06-30  2023-03-31  2022-12-31  2022-09-30
    0   126875503   206239986   177945862   150780886    51081679

<br>

- **FCFF CAGR**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# CAGR of FCFF
latest_free_cash_flow_firm = float(free_cash_flow_firm_df.iloc[0,0])
earliest_free_cash_flow_firm = float(free_cash_flow_firm_df.iloc[0,len(free_cash_flow_firm_df.columns)-1])
free_cash_flow_firm_CAGR = ((latest_free_cash_flow_firm/earliest_free_cash_flow_firm)\
                            **(float(1/(len(free_cash_flow_firm_df.columns)))))-1

print(free_cash_flow_firm_CAGR)
{% endhighlight %}

</details>


    FCFF CAGR: 0.19956151126544364
    
<br>
- **Long term growth rate**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
long_term_growth = free_cash_flow_firm_CAGR / 6
long_term_growth
{% endhighlight %}

</details>

    Long Term Growth Rate: 0.03326025187757394

<br>
- **Forecast FCFF with 5 quarters**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Forecasted FCFF
forecast_free_cash_flow_firm_df = pd.DataFrame(columns=['Year ' + str(i) for i in range(1,6)])
free_cash_flow_firm_forecast_lst = []
for i in range(1,6):
    if i != 5:
        free_cash_flow_firm_forecast = latest_free_cash_flow_firm*(1+free_cash_flow_firm_CAGR)**i
    else:
        free_cash_flow_firm_forecast = latest_free_cash_flow_firm*(1+free_cash_flow_firm_CAGR)\
                                        **(i-1)*(1+long_term_growth)
    free_cash_flow_firm_forecast_lst.append(int(free_cash_flow_firm_forecast))
forecast_free_cash_flow_firm_df.loc[0] = free_cash_flow_firm_forecast_lst
forecast_free_cash_flow_firm_df
{% endhighlight %}

</details>


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
      <th>Quarter 1</th>
      <th>Quarter 2</th>
      <th>Quarter 3</th>
      <th>Quarter 4</th>
      <th>Quarter 5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>152194970</td>
      <td>182567228</td>
      <td>219000620</td>
      <td>262704715</td>
      <td>271442340</td>
    </tr>
  </tbody>
</table>
</div>

<br>
- **Risk-Free Rate**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Risk-free Rate
timespan = 100
current_date = datetime.date.today()
formatted_date = current_date.strftime('%Y-%m-%d')
past_date = current_date-datetime.timedelta(days=timespan)
formatted_past_date = past_date.strftime('%Y-%m-%d')
tk = yf.Ticker('^TNX')
risk_free_rate_df = tk.history(period='3mo')
risk_free_rate = (risk_free_rate_df.iloc[len(risk_free_rate_df)-1,3])/100
risk_free_rate
{% endhighlight %}

</details>

    Risk-Free Rate: 0.0422599983215332


<br>
- **Fundament stats**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
from finvizfinance.quote import finvizfinance
ticker2 = finvizfinance('TNK')
tk = ticker2.ticker_fundament()
tk
{% endhighlight %}

</details>

    {'Company': 'Teekay Tankers Ltd',
     'Section': 'Energy',
     'Industry': 'Oil & Gas Midstream',
     'Country': 'Bermuda',
     'Exchange': 'NYSE',
     'Index': 'RUT',
     'P/E': '3.20',
     'EPS (ttm)': '15.88',
     'Insider Own': '30.25%',
     'Shs Outstand': '29.30M',
     'Perf Week': '-2.31%',
     'Market Cap': '1.73B',
     'Forward P/E': '4.82',
     'EPS next Y': '10.54',
     'Insider Trans': '0.00%',
     'Shs Float': '23.76M',
     'Perf Month': '-1.21%',
     'Income': '548.40M',
     'PEG': '1.07',
     'EPS next Q': '3.61',
     'Inst Own': '49.63%',
     'Short Float / Ratio': '2.32% / 1.31',
     'Perf Quarter': '23.19%',
     'Sales': '1.42B',
     'P/S': '1.22',
     'EPS this Y': '139.99%',
     'Inst Trans': '29.95%',
     'Short Interest': '0.55M',
     'Perf Half Y': '34.40%',
     'Book/sh': '41.72',
     'P/B': '1.22',
     'EPS next Y Percentage': '-31.28%',
     'ROA': '32.23%',
     'Target Price': '64.14',
     'Perf Year': '54.39%',
     'Cash/sh': '6.69',
     'P/C': '7.59',
     'EPS next 5Y': '3.00%',
     'ROE': '46.78%',
     '52W Range From': '25.74',
     '52W Range To': '54.11',
     'Perf YTD': '69.07%',
     'Dividend': '2.12',
     'P/FCF': '2.75',
     'EPS past 5Y': '-',
     'ROI': '34.42%',
     '52W High': '-6.14%',
     'Beta': '-0.08',
     'Dividend %': '4.18%',
     'Quick Ratio': '3.18',
     'Sales past 5Y': '30.55%',
     'Gross Margin': '44.13%',
     '52W Low': '97.31%',
     'ATR': '1.74',
     'Employees': '1750',
     'Current Ratio': '3.58',
     'Sales Q/Q': '2.32%',
     'Oper. Margin': '40.88%',
     'RSI (14)': '55.12',
     'Volatility W': '3.53%',
     'Volatility M': '3.74%',
     'Optionable': 'Yes',
     'Debt/Eq': '0.16',
     'EPS Q/Q': '18.70%',
     'Profit Margin': '38.66%',
     'Rel Volume': '0.98',
     'Prev Close': '49.69',
     'Shortable': 'Yes',
     'LT Debt/Eq': '0.12',
     'Earnings': 'Nov 02 BMO',
     'Payout': '0.00%',
     'Avg Volume': '420.48K',
     'Price': '50.79',
     'Recom': '1.29',
     'SMA20': '-1.46%',
     'SMA50': '9.65%',
     'SMA200': '21.95%',
     'Volume': '412,506',
     'Change': '2.21%'}


<br>
- **Beta**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
beta = float(tk['Beta']) # float is for decimal and int is for integer
beta
{% endhighlight %}

</details>


    Beta: -0.08


<br>
- **Market Rick Premium**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
market_risk_premium = (0.10-risk_free_rate)
market_risk_premium
{% endhighlight %}

</details>

    0.0577400016784668


<br>
- **Cost of Equity**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Required Cost of Equity
coe = risk_free_rate + (beta*market_risk_premium)
coe
{% endhighlight %}

</details>

    Cost of Equity: 0.03764079818725586


<br>
- **Cost of Debt**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
interest_expense = income_statement_df.loc['Interest Expense']
interest_expense_df = interest_expense.to_frame().transpose()
interest_expense_str = interest_expense_df.values[0][0]
interest_expense_int = int(interest_expense_str)


# Total Debt
total_debt = balance_sheet_df.loc['Total Debt']
total_debt_df = total_debt.to_frame().transpose()
total_debt_str = total_debt_df.values[0][0]
total_debt_int = int(total_debt_str)

# Required Cost of Debt
cod = interest_expense_int / total_debt_int

print(cod)
{% endhighlight %}

</details>

    Cost of Debt: 0.027837333148903798
    
<br>
- **Effective Tax Rate**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Effective Tax Rate
effective_tax_rate = income_statement_df.loc['Tax Provision'].astype(int) \
                           / income_statement_df.loc['Pretax Income'].astype(int)

avg_effective_tax_rate = sum(effective_tax_rate) / len(effective_tax_rate)
avg_effective_tax_rate
{% endhighlight %}

</details>

    Average Effective Tax Rate: 0.01132463897412178


<br>
- **Market Cap**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
market_cap_str = tk['Market Cap']

market_cap_lst = market_cap_str.split('.')
if market_cap_str[len(market_cap_str)-1] == 'T':
    market_cap_length = len(market_cap_lst[1])-1
    market_cap_lst[1] = market_cap_lst[1].replace('T',(12-market_cap_length)*'0')
    market_cap_int = int(''.join(market_cap_lst))
if market_cap_str[len(market_cap_str)-1] == 'B':
    market_cap_length = len(market_cap_lst[1])-1
    market_cap_lst[1] = market_cap_lst[1].replace('B',(9-market_cap_length)*'0')
    market_cap_int = int(''.join(market_cap_lst))

market_cap_int
{% endhighlight %}

</details>

    Market Cap: 1730000000


<br>
- **Enterprise Value**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
last_cf = cash_flow_df.loc['End Cash Position']
last_cf_df = last_cf.to_frame().transpose()
last_cf_str = last_cf_df.values[0][-1]
last_cf_int = int(last_cf_str)

last_equity = balance_sheet_df.loc['Total Equity Gross Minority Interest']
last_equity_df = last_equity.to_frame().transpose()
last_equity_str = last_equity_df.values[0][-1]
last_equity_int = int(last_equity_str)


enterprise_value = market_cap_int + total_debt_int - last_cf_int

print(enterprise_value)
{% endhighlight %}

</details>

    EV: 1877471000
    
<br>
- **WACC Calculation**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
WACC = ((last_equity_int/(last_equity_int + total_debt_int)) * coe) \
        + ((total_debt_int/(last_equity_int + total_debt_int)) * cod * (1-avg_effective_tax_rate))

WACC
{% endhighlight %}

</details>

    WACC: 0.03561235594017095


<br>
- **Calculation of Terminal Value, PV and Firm Value.**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Equity Value Calculation
discounted_FCFF_lst = []
for year in range(0,5):
    discounted_FCFF = forecast_free_cash_flow_firm_df.iloc[0,year]/(1+WACC)**(year+1)
    discounted_FCFF_lst.append(int(discounted_FCFF))
terminal_value = (forecast_free_cash_flow_firm_df.iloc[0,4]*(1+long_term_growth))/(WACC-long_term_growth)
PV_terminal_value = int(terminal_value/(1+WACC)**6)
firm_value = sum(discounted_FCFF_lst)+PV_terminal_value
equity_value = (firm_value - total_debt_int + last_cf_int) / 10

print(equity_value)
{% endhighlight %}

</details>

    Firm Value: 9748330114.8
    

<br>
- **Shares Outstanding**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Total Shares Outstanding
shares_outstanding_str = tk['Shs Outstand']

shares_outstanding_lst = shares_outstanding_str.split('.')
if shares_outstanding_str[len(shares_outstanding_str)-1] == 'T':
    shares_outstanding_length = len(shares_outstanding_lst[1])-1
    shares_outstanding_lst[1] = shares_outstanding_lst[1].replace('T',(12-shares_outstanding_length)*'0')
    shares_outstanding_int = int(''.join(shares_outstanding_lst))
if shares_outstanding_str[len(shares_outstanding_str)-1] == 'B':
    shares_outstanding_length = len(shares_outstanding_lst[1])-1
    shares_outstanding_lst[1] = shares_outstanding_lst[1].replace('B',(9-shares_outstanding_length)*'0')
    shares_outstanding_int = int(''.join(shares_outstanding_lst))
if shares_outstanding_str[len(shares_outstanding_str)-1] == 'M':
    shares_outstanding_length = len(shares_outstanding_lst[1])-1
    shares_outstanding_lst[1] = shares_outstanding_lst[1].replace('M',(6-shares_outstanding_length)*'0')
    shares_outstanding_int = int(''.join(shares_outstanding_lst))

shares_outstanding_int
{% endhighlight %}

</details>

    Shares Outstanding: 29300000


<br>
- **Model Price and Actual Price Comparison**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# Two-stage FCFF Valuation Model Stock Price Estimate
stock_price = equity_value / shares_outstanding_int
stock_price = '${:,.2f}'.format(stock_price)
print("Model Stock Price = %s"%(stock_price))

# Actual Stock Price
actual_stock_price = market_cap_int / shares_outstanding_int
actual_stock_price = '${:,.2f}'.format(actual_stock_price)
print("Actual Stock Price = %s"%(actual_stock_price))
{% endhighlight %}

</details>

    Model Stock Price = $332.71
    Actual Stock Price = $59.04

*This company is undervalued through DCF Modeling*

---

### Buffet Metrics
- **Warren Buffet's metrics**


1. Interest Expense / Operating Income < 15%
2. Gross Profit / Revenue > 40%
3. SG&A / Gross Profit < 30%
4. Depreciation / Gross Profit < 8%
5. Tax Paid / Pre-tax Income (Earnings Before Tax) ~21%
6. Net Income / Revenue > 20%
7. Capital Expenditures / Net Income < 8%
8. Total Liabilities / Shareholder Equity < 80% 
9. Net Income / Shareholder Equity > 15%

---

- **2022 Analysis**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# 2022y only
print('2022y Analaysis')

# Interest Expense / Operating Income < 15%
io = annual_income.loc['Interest Expense'][0]/annual_income.loc['Operating Income'][0]
if io < 0.15:
    print(f'Interest Expense/Operating Income: {io}')
else:
    print('Interest Expense is too high relative to Operating Income')

# Gross Profit / Revenue > 40%
gr = annual_income.loc['Gross Profit'][0]/annual_income.loc['Total Revenue'][0]
if gr > 0.40:
    print(f'Gross Profit/Total Revenue: {gr}')
else:
    print(f'Gross Profit is not enough relative to Total Revenue: {gr}')

# SG&A / Gross Profit < 30%
sgag = annual_income.loc['Selling General And Administration'][0]/annual_income.loc['Gross Profit'][0]
if sgag < 0.30:
    print(f'SG&A/Gross Profit: {sgag}')
else:
    print('SG&A is too large relative to Gross Profit')

# Depreciation / Gross Profit < 8%
dg = annual_income.loc['Reconciled Depreciation'][0]/annual_income.loc['Gross Profit'][0]
if dg < 0.08:
    print(f'Depreciation/Gross Profit: {dg}')
else:
    print(f'Depreciation is too large relative to Gross Profit: {dg}')

# Tax Paid / Pre-tax Income (Earnings Before Tax) ~21%
tr = annual_income.loc['Tax Provision'][0]/annual_income.loc['Pretax Income'][0]
if tr < 0.21:
    print(f'Tax Provision/Pretax Income: {tr}')
else:
    print(f'Tax Provision is too large relative to Pretax Income: {tr}')

# Net Income / Revenue > 20%
nr = annual_income.loc['Net Income'][0]/annual_income.loc['Total Revenue'][0]
if nr > 0.20:
    print(f'Net Income/Total Revenue: {nr}')
else:
    print('Net Income is not enough relative to Total Revenue')

# Capital Expenditures / Net Income < 8%
cn = abs(annual_cash_flow.loc['Capital Expenditure'][0]/annual_income.loc['Net Income'][0])
if cn < 0.08:
    print(f'CAPEX/Net Income: {cn}')
else:
    print('CAPEX is too large relative to Net Income')

# Total Liabilities / Shareholder Equity < 80% 
le = annual_balance_sheet.loc['Total Liabilities Net Minority Interest'][0]/annual_balance_sheet.loc['Stockholders Equity'][0]
if le < 0.80:
    print(f'Leverage Ratio: {le}')
else:
    print('Leverage ratio is too high')

# Net Income / Shareholder Equity > 15%
ie = annual_income.loc['Net Income'][0]/annual_balance_sheet.loc['Stockholders Equity'][0]
if ie > 0.15:
    print(f'Earnings per Share (EPS): {ie}')
else:
    print('EPS is not enough for the company')
{% endhighlight %}

</details>

    2022y Analaysis
    1. Interest Expense/Operating Income: 0.14360161200242685
    2. Gross Profit is not enough relative to Total Revenue: 0.2733976038250004
    3. SG&A/Gross Profit: 0.1437079393914372
    4. Depreciation is too large relative to Gross Profit: 0.3407270550348871
    5. Tax Provision/Pretax Income: 0.002303856455370947
    6. Net Income/Total Revenue: 0.2154864355650539
    7. CAPEX/Net Income: 0.06735461791641567
    8. Leverage Ratio: 0.6669570077177137
    9. Earnings per Share (EPS): 0.21409786487178575
    
*Only two metrics are not good enough for 2022y*

<br>

- **2019 - 2022 Analysis**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# 2019 - 2022 
print('2019y-2022y Analysis')

# Interest Expense / Operating Income < 15%
io = annual_income.loc['Interest Expense']/annual_income.loc['Operating Income']
io = sum(io)/len(io)
if io < 0.15:
    print(f'Interest Expense/Operating Income: {io}')
else:
    print(f'Interest Expense is too high relative to Operating Income: {io}')

# Gross Profit / Revenue > 40%
gr = annual_income.loc['Gross Profit']/annual_income.loc['Total Revenue']
gr = sum(gr)/len(gr)
if gr > 0.40:
    print(f'Gross Profit/Total Revenue: {gr}')
else:
    print(f'Gross Profit is not enough relative to Total Revenue: {gr}')

# SG&A / Gross Profit < 30%
sgag = annual_income.loc['Selling General And Administration']/annual_income.loc['Gross Profit']
sgag = sum(sgag)/len(sgag)
if sgag < 0.30:
    print(f'SG&A/Gross Profit: {sgag}')
else:
    print('SG&A is too large relative to Gross Profit')

# Depreciation / Gross Profit < 8%
dg = annual_income.loc['Reconciled Depreciation']/annual_income.loc['Gross Profit']
dg = sum(dg)/len(dg)
if dg < 0.08:
    print(f'Depreciation/Gross Profit: {dg}')
else:
    print(f'Depreciation is too large relative to Gross Profit: {dg}')

# Tax Paid / Pre-tax Income (Earnings Before Tax) ~21%
tr = annual_income.loc['Tax Provision']/annual_income.loc['Pretax Income']
tr = sum(tr)/len(tr)
if tr < 0.21:
    print(f'Tax Provision/Pretax Income: {tr}')
else:
    print(f'Tax Provision is too large relative to Pretax Income: {tr}')

# Net Income / Revenue > 20%
nr = annual_income.loc['Net Income']/annual_income.loc['Total Revenue']
nr = sum(nr)/len(nr)
if nr > 0.20:
    print(f'Net Income/Total Revenue: {nr}')
else:
    print('Net Income is not enough relative to Total Revenue')

# Capital Expenditures / Net Income < 8%
cn = abs(annual_cash_flow.loc['Capital Expenditure']/annual_income.loc['Net Income'])
cn = sum(cn)/len(cn)
if cn < 0.08:
    print(f'CAPEX/Net Income: {cn}')
else:
    print(f'CAPEX is too large relative to Net Income: {cn}')

# Total Liabilities / Shareholder Equity < 80% 
le = (annual_balance_sheet.loc['Total Liabilities Net Minority Interest']/annual_balance_sheet.loc['Stockholders Equity'])
le = sum(le)/len(le)
if le < 0.80:
    print(f'Leverage Ratio: {le}')
else:
    print(f'Leverage ratio is too high: {le}')

# Net Income / Shareholder Equity > 15%
ie = annual_income.loc['Net Income']/annual_balance_sheet.loc['Stockholders Equity']
ie = sum(ie)/len(ie)
if ie > 0.15:
    print(f'Earnings per Share (EPS): {ie}')
else:
    print(f'EPS is not enough for the company: {ie}')
{% endhighlight %}

</details>

    2019y-2022y Analysis
    Interest Expense is too high relative to Operating Income: 0.15991957466416531
    Gross Profit is not enough relative to Total Revenue: 0.15720203811447034
    SG&A/Gross Profit: -0.05838760684888775
    Depreciation/Gross Profit: -0.06710785538280992
    Tax Provision/Pretax Income: 0.14330527783322572
    Net Income is not enough relative to Total Revenue
    CAPEX is too large relative to Net Income: 0.15512421430566428
    Leverage ratio is too high: 0.8910497806271792
    EPS is not enough for the company: 0.011931947689316063
    
*2019y to 2022y are not good enough for Buffet's metrics. This company is profitable after Covid and Crude oil is cheaper*

<br>

- **3Q 2023 Analysis**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
# 09-30-23
print('3Q-23 Analysis')

annual_income = income_statement_df
annual_cash_flow = cash_flow_df
annual_balance_sheet = balance_sheet_df

# Interest Expense / Operating Income < 15%
io = annual_income.loc['Interest Expense'][0]/annual_income.loc['Operating Income'][0]
if io < 0.15:
    print(f'Interest Expense/Operating Income: {io}')
else:
    print('Interest Expense is too high relative to Operating Income')

# Gross Profit / Revenue > 40%
gr = annual_income.loc['Gross Profit'][0]/annual_income.loc['Total Revenue'][0]
if gr > 0.40:
    print(f'Gross Profit/Total Revenue: {gr}')
else:
    print(f'Gross Profit is not enough relative to Total Revenue: {gr}')

# SG&A / Gross Profit < 30%
sgag = annual_income.loc['Selling General And Administration'][0]/annual_income.loc['Gross Profit'][0]
if sgag < 0.30:
    print(f'SG&A/Gross Profit: {sgag}')
else:
    print('SG&A is too large relative to Gross Profit')

# Depreciation / Gross Profit < 8%
dg = annual_income.loc['Reconciled Depreciation'][0]/annual_income.loc['Gross Profit'][0]
if dg < 0.08:
    print(f'Depreciation/Gross Profit: {dg}')
else:
    print(f'Depreciation is too large relative to Gross Profit: {dg}')

# Tax Paid / Pre-tax Income (Earnings Before Tax) ~21%
tr = annual_income.loc['Tax Provision'][0]/annual_income.loc['Pretax Income'][0]
if tr < 0.21:
    print(f'Tax Provision/Pretax Income: {tr}')
else:
    print(f'Tax Provision is too large relative to Pretax Income: {tr}')

# Net Income / Revenue > 20%
nr = annual_income.loc['Net Income'][0]/annual_income.loc['Total Revenue'][0]
if nr > 0.20:
    print(f'Net Income/Total Revenue: {nr}')
else:
    print('Net Income is not enough relative to Total Revenue')

# Capital Expenditures / Net Income < 8%
cn = abs(annual_cash_flow.loc['Capital Expenditure'][0]/annual_income.loc['Net Income'][0])
if cn < 0.08:
    print(f'CAPEX/Net Income: {cn}')
else:
    print('CAPEX is too large relative to Net Income')

# Total Liabilities / Shareholder Equity < 80% 
le = annual_balance_sheet.loc['Total Liabilities Net Minority Interest'][0]/annual_balance_sheet.loc['Stockholders Equity'][0]
if le < 0.80:
    print(f'Debt to Equity: {le}')
else:
    print('Debt is too high')

# Net Income / Shareholder Equity > 15%
ie = annual_income.loc['Net Income'][0]/annual_balance_sheet.loc['Stockholders Equity'][0]
if ie > 0.15:
    print(f'Net Income/Stockholders Equity: {ie}')
else:
    print(f'Net Income is not enough relative to Stockholders equity: {ie}')

{% endhighlight %}

</details>

    3Q-23 Analysis
    Interest Expense/Operating Income: 0.07894575543977934
    Gross Profit is not enough relative to Total Revenue: 0.32280013153383846
    SG&A/Gross Profit: 0.11595773503115687
    Depreciation is too large relative to Gross Profit: 0.26621511785424007
    Tax Provision/Pretax Income: -0.03206575509272178
    Net Income/Total Revenue: 0.284637827172932
    CAPEX/Net Income: 0.04025022736769658
    Debt to Equity: 0.245421902420668
    Net Income is not enough relative to Stockholders equity: 0.057230879606672225

*Three metrics are not good enough for 3Q-23*    

---
### Correlation
**Correlation between TNK, Crude Oil and S&P500**

<br>
- **TNK price history** 

<details>
<summary>Show hidden code</summary>

{% highlight python %}
price = ticker.history(
    start = '2018-01-01',
    end = '2023-12-02',
    interval = '1d',
    )['Close']
{% endhighlight %}

</details>

    Date
    2018-01-02 00:00:00-05:00    10.383956
    2018-01-03 00:00:00-05:00    10.383956
    2018-01-04 00:00:00-05:00    10.383956
    2018-01-05 00:00:00-05:00    10.085136
    2018-01-08 00:00:00-05:00     9.711613
    2018-01-09 00:00:00-05:00     9.861023
    2018-01-10 00:00:00-05:00    10.085136
    2018-01-11 00:00:00-05:00    10.383956
    2018-01-12 00:00:00-05:00    10.309251
 ...               ...
    2023-11-06 00:00:00-05:00    52.891171
    2023-11-07 00:00:00-05:00    52.075008
    2023-11-08 00:00:00-05:00    51.826180
    2023-11-09 00:00:00-05:00    52.224308
    2023-11-10 00:00:00-05:00    53.139999
    2023-11-13 00:00:00-05:00    52.799999
    2023-11-14 00:00:00-05:00    52.090000
    2023-11-15 00:00:00-05:00    50.900002
    2023-11-16 00:00:00-05:00    49.849998
    2023-11-17 00:00:00-05:00    50.889999
    2023-11-20 00:00:00-05:00    50.830002
    2023-11-21 00:00:00-05:00    50.430000
    2023-11-22 00:00:00-05:00    51.880001
    2023-11-24 00:00:00-05:00    51.990002
    2023-11-27 00:00:00-05:00    51.939999
    2023-11-28 00:00:00-05:00    50.220001
    2023-11-29 00:00:00-05:00    49.040001
    2023-11-30 00:00:00-05:00    49.689999
    2023-12-01 00:00:00-05:00    50.790001
    Name: Close, dtype: float64


<br>
- **Crude oil price history**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
wti = yf.Ticker('CL=F')
price_wti = wti.history(
    start = '2018-01-01',
    end = '2023-12-02',
    interval= '1d',
)['Close']
price_wti
{% endhighlight %}

</details>

    Date
    2018-01-02 00:00:00-05:00     60.369999
    2018-01-03 00:00:00-05:00     61.630001
    2018-01-04 00:00:00-05:00     62.009998
    2018-01-05 00:00:00-05:00     61.439999
    2018-01-08 00:00:00-05:00     61.730000
    2018-01-09 00:00:00-05:00     62.959999
    2018-01-10 00:00:00-05:00     63.570000
    2018-01-11 00:00:00-05:00     63.799999
    2018-01-12 00:00:00-05:00     64.300003
 ...               ...
    2023-11-06 00:00:00-05:00     80.820000
    2023-11-07 00:00:00-05:00     77.370003
    2023-11-08 00:00:00-05:00     75.330002
    2023-11-09 00:00:00-05:00     75.739998
    2023-11-10 00:00:00-05:00     77.169998
    2023-11-13 00:00:00-05:00     78.260002
    2023-11-14 00:00:00-05:00     78.260002
    2023-11-15 00:00:00-05:00     76.660004
    2023-11-16 00:00:00-05:00     72.900002
    2023-11-17 00:00:00-05:00     75.889999
    2023-11-20 00:00:00-05:00     77.599998
    2023-11-21 00:00:00-05:00     77.769997
    2023-11-22 00:00:00-05:00     77.099998
    2023-11-23 00:00:00-05:00     76.349998
    2023-11-24 00:00:00-05:00     75.540001
    2023-11-27 00:00:00-05:00     74.860001
    2023-11-28 00:00:00-05:00     76.410004
    2023-11-29 00:00:00-05:00     77.860001
    2023-11-30 00:00:00-05:00     75.959999
    2023-12-01 00:00:00-05:00     74.070000
    Name: Close, dtype: float64


<br>
- **S&P500 price history**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
snp = yf.Ticker('^GSPC')
price_snp = snp.history(
        start = '2018-01-01',
    end = '2023-12-02',
    interval= '1d',
)['Close']

price_snp
{% endhighlight %}

</details>


    Date
    2018-01-02 00:00:00-05:00    2695.810059
    2018-01-03 00:00:00-05:00    2713.060059
    2018-01-04 00:00:00-05:00    2723.989990
    2018-01-05 00:00:00-05:00    2743.149902
    2018-01-08 00:00:00-05:00    2747.709961
    2018-01-09 00:00:00-05:00    2751.290039
    2018-01-10 00:00:00-05:00    2748.229980
    2018-01-11 00:00:00-05:00    2767.560059
    2018-01-12 00:00:00-05:00    2786.239990
 ...               ...
    2023-11-06 00:00:00-05:00    4365.979980
    2023-11-07 00:00:00-05:00    4378.379883
    2023-11-08 00:00:00-05:00    4382.779785
    2023-11-09 00:00:00-05:00    4347.350098
    2023-11-10 00:00:00-05:00    4415.240234
    2023-11-13 00:00:00-05:00    4411.549805
    2023-11-14 00:00:00-05:00    4495.700195
    2023-11-15 00:00:00-05:00    4502.879883
    2023-11-16 00:00:00-05:00    4508.240234
    2023-11-17 00:00:00-05:00    4514.020020
    2023-11-20 00:00:00-05:00    4547.379883
    2023-11-21 00:00:00-05:00    4538.189941
    2023-11-22 00:00:00-05:00    4556.620117
    2023-11-24 00:00:00-05:00    4559.339844
    2023-11-27 00:00:00-05:00    4550.430176
    2023-11-28 00:00:00-05:00    4554.890137
    2023-11-29 00:00:00-05:00    4550.580078
    2023-11-30 00:00:00-05:00    4567.799805
    2023-12-01 00:00:00-05:00    4594.629883
    Name: Close, dtype: float64

---
- **Price chart between TNK and Crude oil**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(12,8))

l1 = ax.plot(price, color='blue', label='TNK')
ax.set_title('TNK & Crude Oil')
ax.set_xlabel('Date')

ax1 = ax.twinx()
l2 = ax1.plot(price_wti, color='green', label='Crude Oil')

# ax2 = ax1.twinx()
# l3 = ax2.plot(price_snp, color='red', label='S&P')

ax.grid
ax.legend(handles=l1+l2, loc=2)
plt.show()
{% endhighlight %}

</details>

![image]({{site.url}}/assets/images/tnk/output1.png)
    
<br>
- **Price merged**
<details>
<summary>Show hidden code</summary>

{% highlight python %}
price_df = pd.DataFrame()

price_df['TNK'] = price
price_df['WTI'] = price_wti
price_df['SNP'] = price_snp
price_df.head()
{% endhighlight %}

</details>

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
      <th>TNK</th>
      <th>WTI</th>
      <th>SNP</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018-01-02 00:00:00-05:00</th>
      <td>10.383956</td>
      <td>60.369999</td>
      <td>2695.810059</td>
    </tr>
    <tr>
      <th>2018-01-03 00:00:00-05:00</th>
      <td>10.383956</td>
      <td>61.630001</td>
      <td>2713.060059</td>
    </tr>
    <tr>
      <th>2018-01-04 00:00:00-05:00</th>
      <td>10.383956</td>
      <td>62.009998</td>
      <td>2723.989990</td>
    </tr>
    <tr>
      <th>2018-01-05 00:00:00-05:00</th>
      <td>10.085136</td>
      <td>61.439999</td>
      <td>2743.149902</td>
    </tr>
    <tr>
      <th>2018-01-08 00:00:00-05:00</th>
      <td>9.711613</td>
      <td>61.730000</td>
      <td>2747.709961</td>
    </tr>
  </tbody>
</table>
</div>

---
- **Returns during the term**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
```python
returns = (price_df.pct_change()+1).product()-1
returns
```
{% endhighlight %}

</details>

    TNK    3.891200
    WTI    0.226934
    SNP    0.704360
    dtype: float64


---
- **Calculation correlation**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
```python
changes = price_df.pct_change()
corr = changes.corr()
corr.head()
```
{% endhighlight %}

</details>


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
      <th>TNK</th>
      <th>WTI</th>
      <th>SNP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>TNK</th>
      <td>1.000000</td>
      <td>-0.093197</td>
      <td>0.269204</td>
    </tr>
    <tr>
      <th>WTI</th>
      <td>-0.093197</td>
      <td>1.000000</td>
      <td>0.137419</td>
    </tr>
    <tr>
      <th>SNP</th>
      <td>0.269204</td>
      <td>0.137419</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>

<br>
- **Show hotmap with correlation**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
```python
plt.figure(figsize=(16,8))
plt.grid(False)
plt.imshow(corr, cmap='hot', interpolation='none')
plt.colorbar()
plt.xticks(range(len(corr)), corr.columns, rotation=90)
plt.yticks(range(len(corr)), corr.columns)
plt.show()
```
{% endhighlight %}

</details>

   
![image]({{site.url}}/assets/images/tnk/output2.png)
    


Crude oil and TNK is inversed. if oil price is dropped, TNK price is up.

---
- **Risk & Return**

<details>
<summary>Show hidden code</summary>

{% highlight python %}
```python
plt.figure(figsize=(16,8))
plt.scatter(changes.std(), changes.mean())
plt.xlabel('Risk')
plt.ylabel('Return')
plt.xlim()
plt.ylim()
for label, x, y in zip(changes.columns, changes.std(), changes.mean()):
    plt.annotate(label, xy=(x, y), xytext=(30, -30),
    textcoords = 'offset points',
    ha = 'right', va = 'bottom',
    bbox = dict(boxstyle = 'round,pad=0.5', fc = 'green', alpha = 0.5),
    arrowprops = dict(arrowstyle = '->', connectionstyle = 'arc3, rad=0'))  
```
{% endhighlight %}

</details>
    
![image]({{site.url}}/assets/images/tnk/output3.png)

---
### **Conclusion**

![image]({{site.url}}/assets/images/tnk/output4.png)