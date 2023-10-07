# Stock Valuation using Free Cash Flow to the Firm with Python

**Hugh Donnelly, CFA**<br> 
*AlphaWave Data*

**November 2020**

For this analysis, we use several Python-based scientific computing technologies along with the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints) and the [AlphaWave Data Stock Analysis API](https://rapidapi.com/alphawave/api/stock-analysis/endpoints).  Jupyter Notebooks detailing this analysis are also available on [Google Colab](https://colab.research.google.com/drive/1BWmRIDtZGRndLk8lLGgpoURaAzHC8_jg?usp=sharing) and [Github](https://github.com/AlphaWaveData/Jupyter-Notebooks/blob/master/AlphaWave%20Stock%20Valuation%20using%20Free%20Cash%20Flow%20to%20the%20Firm%20example.ipynb).


```python
from bs4 import BeautifulSoup as bs
import pandas as pd
import requests
import pandas_datareader as dr
import datetime
```

We will value Apple Inc. (AAPL) in the subsequent code; however, this analysis works with the ticker of any large publicly traded company.


```python
company_ticker = 'AAPL'
```

### Introduction to Free Cash Flow Valuation

In stock valuation models, there are three predominant definitions of future cash flows: dividends, free cash flows, and residual income.  We will focus on building a free cash flow model for stock valuation in this article.  Let's discuss the advantages and disadvantages of free cash flow valuation.

One advantage of free cash flow models is that they can be applied to many firms, regardless of dividend policies or capital structures.  The ability to influence the distribution and application of a firm's free cash flow makes these models more pertinent to a firm's controlling shareholders.

However, there are cases in which the application of a free cash flow model may be very difficult.  Firms that have significant capital requirements may have negative free cash flow for many years into the future.  This can be caused by a technological revolution in an industry that requires greater investment to remain competitive or by rapid expansion into untapped markets.  This negative free cash flow complicates the cash flow forecast and makes the estimates less reliable.

Free cash flow models are most appropriate:
* For firms that do not have a dividend payment history or have a dividend payment history that is not clearly and appropriately related to earnings.
* For firms with free cash flow that corresponds with their profitability.
* When the valuation perspective is that of a controlling shareholder.

We will build our free cash flow model in three steps.  First, we will calculate the Free Cash Flow to the Firm.  Second, we will determine the Weighted Average Cost of Capital.  Finally, we will perform Stock Valuation using these inputs.

### 1. Free Cash Flow to the Firm

Forget about all the complicated financial statement relationships for a minute and simply picture the firm as a cash "processor."  Cash flows into the firm in the form of revenue as it sells its product, and cash flows out as it pays its cash operating expenses (e.g., salaries and taxes, but not interest expense, which is a financing and not an operating expense).  The firm takes the cash that is left over and makes short-term net investments in working capital (e.g., inventory and receivables) and long-term investments in property, plant, and equipment.  After these actions, the cash that remains is available to pay out to the firm's investors: bondholders, and common shareholders (let's assume for the moment that the firm has not issued preferred stock).  That pile of remaining cash is called Free Cash Flow to the Firm (FCFF) because it's "free" to payout to the firm's investors.

The formal definition of FCFF is the cash available to all of the firm's investors, including stockholders and bondholders, after the firm buys and sells products, provides services, pays its cash operating expenses, and makes short-term and long-term investments.

When building a FCFF valuation model, one can typically choose between a single-stage, two-stage, or three stage valuation model.  The single-stage FCFF valuation model is useful for stable firms in mature industries.  Two-stage and three-stage FCFF valuation models are used when you want to model multiple phases of growth (e.g., a growth phase, a mature phase, and a transition phase).  We use a two-stage FCFF valuation model in this example.

A general expression for the two-stage FCFF valuation model is:

$$Firm\ Value = \sum_{t=1}^n\overbrace{\frac{FCFF_t}{(1 + WACC)^t}}^{Stage\ 1}+\overbrace{\frac{FCFF_{n+1}}{(WACC - g)}*\frac{1}{(1 + WACC)^n}}^{Stage\ 2}$$<br>

$FCFF_t = expected\ free\ cash\ flow\ to\ the\ firm\ in\ year\ t$<br>
$FCFF_{n+1} = expected\ free\ cash\ flow\ to\ the\ firm\ in\ the\ last\ year\ (Year\ 6\ in\ this\ example)$<br>
$g = constant\ expected\ growth\ rate\ in\ FCFF$<br>
$WACC = weighted\ average\ cost\ of\ capital$<br>

One common two-stage model assumes a constant growth rate in each stage, and a second common model assumes declining growth in Stage 1 followed by a long-run sustainable growth rate in Stage 2.  In our model, we assume a constant growth rate in each stage.

We will show how to obtain each component of the two-stage FCFF valuation model, but we begin with expected FCFF which requires an ability to determine the appropriate growth rate.

To calculate the FCFF growth rate, we take the compound annual growth rate (CAGR) of FCFF over the past three years.  This FCFF growth rate is then used to generate FCFF forecasts for each of the next 5 years, which is Stage 1 in our model. For the constant expected growth rate that will be applied to Stage 2, we divide the CAGR of FCFF by two in order to estimate the long-term growth rate of FCFF in perpetuity.

In order to calculate the FCFF growth rate and long-term growth rate, we must first acquire the FCFF from the company's financial statements.  Using the [Cash Flow Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_1cf7a51e-32f9-41a5-901a-fa4f66ba9807) and [Income Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_b4be9cff-500e-4b0a-a5a9-a8d73f9cde7a) endpoints from the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints), we can access the data needed to calculate FCFF.

To call these APIs with Python, you can choose one of the supported Python code snippets provided in the API console. The following are examples of how to invoke the APIs with Python Requests. You will need to insert your own x-rapidapi-host and x-rapidapi-key information in the code blocks below.


```python
url = "https://financial-statements.p.rapidapi.com/api/v1/resources/cash-flow"

querystring = {"ticker":company_ticker}

headers = {
	"X-RapidAPI-Key": "acc452e6bamshe911abf8e81f8a8p1a5578jsn80a5ba5a41c2",
	"X-RapidAPI-Host": "financial-statements.p.rapidapi.com"
}

cf_response = requests.get(url, headers=headers, params=querystring)

# Create Cash Flow Statement DataFrame
cash_flow_df = pd.DataFrame.from_dict(cf_response.json())
cash_flow_df = cash_flow_df.drop('ttm', axis = 1)
cash_flow_df

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
      <th>9/30/2020</th>
      <th>9/30/2021</th>
      <th>9/30/2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Capital Expenditure</th>
      <td>-7309000</td>
      <td>-11085000</td>
      <td>-10708000</td>
    </tr>
    <tr>
      <th>End Cash Position</th>
      <td>39789000</td>
      <td>35929000</td>
      <td>24977000</td>
    </tr>
    <tr>
      <th>Financing Cash Flow</th>
      <td>-86820000</td>
      <td>-93353000</td>
      <td>-110749000</td>
    </tr>
    <tr>
      <th>Free Cash Flow</th>
      <td>73365000</td>
      <td>92953000</td>
      <td>111443000</td>
    </tr>
    <tr>
      <th>Income Tax Paid Supplemental Data</th>
      <td>9501000</td>
      <td>25385000</td>
      <td>19573000</td>
    </tr>
    <tr>
      <th>Interest Paid Supplemental Data</th>
      <td>3002000</td>
      <td>2687000</td>
      <td>2865000</td>
    </tr>
    <tr>
      <th>Investing Cash Flow</th>
      <td>-4289000</td>
      <td>-14545000</td>
      <td>-22354000</td>
    </tr>
    <tr>
      <th>Issuance of Capital Stock</th>
      <td>880000</td>
      <td>1105000</td>
      <td>None</td>
    </tr>
    <tr>
      <th>Issuance of Debt</th>
      <td>16091000</td>
      <td>20393000</td>
      <td>9420000</td>
    </tr>
    <tr>
      <th>Operating Cash Flow</th>
      <td>80674000</td>
      <td>104038000</td>
      <td>122151000</td>
    </tr>
    <tr>
      <th>Repayment of Debt</th>
      <td>-13592000</td>
      <td>-8750000</td>
      <td>-9543000</td>
    </tr>
    <tr>
      <th>Repurchase of Capital Stock</th>
      <td>-72358000</td>
      <td>-85971000</td>
      <td>-89402000</td>
    </tr>
  </tbody>
</table>
</div>




```python
url = "https://financial-statements.p.rapidapi.com/api/v1/resources/income-statement"

querystring = {"ticker":company_ticker}

headers = {
	"X-RapidAPI-Key": "acc452e6bamshe911abf8e81f8a8p1a5578jsn80a5ba5a41c2",
	"X-RapidAPI-Host": "financial-statements.p.rapidapi.com"
}

income_statement_response = requests.get(url, headers=headers, params=querystring)

# Create Income Statement DataFrame
income_statement_df = pd.DataFrame.from_dict(income_statement_response.json())
income_statement_df = income_statement_df.drop('ttm', axis = 1)
income_statement_df
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
      <th>9/30/2020</th>
      <th>9/30/2021</th>
      <th>9/30/2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Basic Average Shares</th>
      <td>17352119</td>
      <td>16701272</td>
      <td>16215963</td>
    </tr>
    <tr>
      <th>Cost of Revenue</th>
      <td>169559000</td>
      <td>212981000</td>
      <td>223546000</td>
    </tr>
    <tr>
      <th>Diluted Average Shares</th>
      <td>17528214</td>
      <td>16864919</td>
      <td>16325819</td>
    </tr>
    <tr>
      <th>Diluted NI Available to Com Stockholders</th>
      <td>57411000</td>
      <td>94680000</td>
      <td>99803000</td>
    </tr>
    <tr>
      <th>EBIT</th>
      <td>66288000</td>
      <td>108949000</td>
      <td>119437000</td>
    </tr>
    <tr>
      <th>EBITDA</th>
      <td>77344000</td>
      <td>120233000</td>
      <td>130541000</td>
    </tr>
    <tr>
      <th>Gross Profit</th>
      <td>104956000</td>
      <td>152836000</td>
      <td>170782000</td>
    </tr>
    <tr>
      <th>Interest Expense</th>
      <td>2873000</td>
      <td>2645000</td>
      <td>2931000</td>
    </tr>
    <tr>
      <th>Interest Income</th>
      <td>3763000</td>
      <td>2843000</td>
      <td>2825000</td>
    </tr>
    <tr>
      <th>Net Income Common Stockholders</th>
      <td>57411000</td>
      <td>94680000</td>
      <td>99803000</td>
    </tr>
    <tr>
      <th>Net Income from Continuing &amp; Discontinued Operation</th>
      <td>57411000</td>
      <td>94680000</td>
      <td>99803000</td>
    </tr>
    <tr>
      <th>Net Income from Continuing Operation Net Minority Interest</th>
      <td>57411000</td>
      <td>94680000</td>
      <td>99803000</td>
    </tr>
    <tr>
      <th>Net Interest Income</th>
      <td>890000</td>
      <td>198000</td>
      <td>-106000</td>
    </tr>
    <tr>
      <th>Net Non Operating Interest Income Expense</th>
      <td>890000</td>
      <td>198000</td>
      <td>-106000</td>
    </tr>
    <tr>
      <th>Normalized EBITDA</th>
      <td>77344000</td>
      <td>120233000</td>
      <td>130541000</td>
    </tr>
    <tr>
      <th>Normalized Income</th>
      <td>57411000</td>
      <td>94680000</td>
      <td>99803000</td>
    </tr>
    <tr>
      <th>Operating Expense</th>
      <td>38668000</td>
      <td>43887000</td>
      <td>51345000</td>
    </tr>
    <tr>
      <th>Operating Income</th>
      <td>66288000</td>
      <td>108949000</td>
      <td>119437000</td>
    </tr>
    <tr>
      <th>Other Income Expense</th>
      <td>803000</td>
      <td>258000</td>
      <td>-334000</td>
    </tr>
    <tr>
      <th>Pretax Income</th>
      <td>67091000</td>
      <td>109207000</td>
      <td>119103000</td>
    </tr>
    <tr>
      <th>Reconciled Cost of Revenue</th>
      <td>169559000</td>
      <td>212981000</td>
      <td>223546000</td>
    </tr>
    <tr>
      <th>Reconciled Depreciation</th>
      <td>11056000</td>
      <td>11284000</td>
      <td>11104000</td>
    </tr>
    <tr>
      <th>Tax Effect of Unusual Items</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Tax Provision</th>
      <td>9680000</td>
      <td>14527000</td>
      <td>19300000</td>
    </tr>
    <tr>
      <th>Tax Rate for Calcs</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Total Expenses</th>
      <td>208227000</td>
      <td>256868000</td>
      <td>274891000</td>
    </tr>
    <tr>
      <th>Total Operating Income as Reported</th>
      <td>66288000</td>
      <td>108949000</td>
      <td>119437000</td>
    </tr>
    <tr>
      <th>Total Revenue</th>
      <td>274515000</td>
      <td>365817000</td>
      <td>394328000</td>
    </tr>
  </tbody>
</table>
</div>



Using the Cash Flow Statement and Income Statement, we calculate FCFF as:

$$FCFF = Free\ Cash\ Flow\ +\ Interest\ Expense*(1\ -\ Effective\ Tax\ Rate)$$

With this understanding, we are able to store the historical FCFF in a dataframe when running the code below.  


```python
# FCFF Calculation using Cash Flow Statement and Income Statement Inputs
free_cash_flow_firm = (cash_flow_df.loc['Free Cash Flow'].astype(int) \
                    + (income_statement_df.loc['Interest Expense'].astype(int) \
                       * (1 - income_statement_df.loc['Tax Provision'].astype(int) \
                          / income_statement_df.loc['Pretax Income'].astype(int)))).astype(int)

# Change Series to a Pandas Dataframe
free_cash_flow_firm_df = free_cash_flow_firm.to_frame().transpose()
free_cash_flow_firm_df
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
      <th>9/30/2020</th>
      <th>9/30/2021</th>
      <th>9/30/2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>75823478</td>
      <td>95246155</td>
      <td>113899047</td>
    </tr>
  </tbody>
</table>
</div>



This allows us to proceed in determining the CAGR and the long-term growth rate that will be used in calculating the forecasted FCFF.


```python
# CAGR of FCFF
latest_free_cash_flow_firm = float(free_cash_flow_firm_df.iloc[0,len(free_cash_flow_firm_df.columns)-1])
earliest_free_cash_flow_firm = float(free_cash_flow_firm_df.iloc[0,1])
free_cash_flow_firm_CAGR = (latest_free_cash_flow_firm/earliest_free_cash_flow_firm)\
                            **(float(1/(len(free_cash_flow_firm_df.columns))))-1
free_cash_flow_firm_CAGR
```




    0.06142882923846793




```python
# Constant Expected Growth Rate
long_term_growth = free_cash_flow_firm_CAGR / 2
long_term_growth
```




    0.030714414619233965




```python
# Forecasted FCFF
forecast_free_cash_flow_firm_df = pd.DataFrame(columns=['Year ' + str(i) for i in range(1,7)])
free_cash_flow_firm_forecast_lst = []
for i in range(1,7):
    if i != 6:
        free_cash_flow_firm_forecast = latest_free_cash_flow_firm*(1+free_cash_flow_firm_CAGR)**i
    else:
        free_cash_flow_firm_forecast = latest_free_cash_flow_firm*(1+free_cash_flow_firm_CAGR)\
                                        **(i-1)*(1+long_term_growth)
    free_cash_flow_firm_forecast_lst.append(int(free_cash_flow_firm_forecast))
forecast_free_cash_flow_firm_df.loc[0] = free_cash_flow_firm_forecast_lst
forecast_free_cash_flow_firm_df

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
      <th>Year 1</th>
      <th>Year 2</th>
      <th>Year 3</th>
      <th>Year 4</th>
      <th>Year 5</th>
      <th>Year 6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>120895732</td>
      <td>128322215</td>
      <td>136204898</td>
      <td>144571806</td>
      <td>153452683</td>
      <td>158165892</td>
    </tr>
  </tbody>
</table>
</div>



Now that we have successfully forecasted FCFF for the next six years and calculated the long-term growth rate, we turn our attention to WACC, which is the final input required in our two-stage FCFF valuation model.

### 2. Weighted Average Cost of Capital

The Weighted Average Cost of Capital (WACC) is the weighted average of the rates of return required by each of the capital suppliers to the firm (usually this is just equity and debt) where the weights are the proportions of the firm's total market value from each capital source:

WACC = (equity-to-capital ratio * required return on equity) + (debt-to-capital ratio * after-tax required return on debt)

$$WACC = \frac{E}{E+D}r_e\ +\ \frac{D}{E+D}r_d(1\ -\ tax\ rate)$$<br>
$E = market\ value\ of\ equity$<br>
$D = market\ value\ of\ debt$<br>
$r_e = required\ return\ on\ equity$<br>
$r_d = required\ return\ on\ debt$<br>
$tax\ rate = effective\ tax\ rate$<br>

Adding the [Balance Sheet Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_df251606-fb96-4468-9067-0d65886ce5b9) endpoint from the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints) and the [Key Statistics](https://rapidapi.com/alphawave/api/stock-analysis?endpoint=apiendpoint_dff4b882-4be4-4169-a700-04275c92bdce) endpoint from the [AlphaWave Data Stock Analysis API](https://rapidapi.com/alphawave/api/stock-analysis/endpoints), we can pull in the remaining data required to calculate WACC.

To call these APIs with Python, you can choose one of the supported Python code snippets provided in the API console. The following are examples of how to invoke the APIs with Python Requests. You will need to insert your own x-rapidapi-host and x-rapidapi-key information in the code blocks below.


```python
url = "https://financial-statements.p.rapidapi.com/api/v1/resources/balance-sheet"

querystring = {"ticker":company_ticker}

headers = {
	"X-RapidAPI-Key": "acc452e6bamshe911abf8e81f8a8p1a5578jsn80a5ba5a41c2",
	"X-RapidAPI-Host": "financial-statements.p.rapidapi.com"
}

balance_sheet_response = requests.get(url, headers=headers, params=querystring)

# Create Balance Sheet DataFrame
balance_sheet_df = pd.DataFrame.from_dict(balance_sheet_response.json())
# balance_sheet_df = balance_sheet_df.drop('ttm', axis = 1)
balance_sheet_df


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
      <th>9/30/2020</th>
      <th>9/30/2021</th>
      <th>9/30/2022</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Common Stock Equity</th>
      <td>65339000</td>
      <td>63090000</td>
      <td>50672000</td>
    </tr>
    <tr>
      <th>Invested Capital</th>
      <td>177775000</td>
      <td>187809000</td>
      <td>170741000</td>
    </tr>
    <tr>
      <th>Net Debt</th>
      <td>74420000</td>
      <td>89779000</td>
      <td>96423000</td>
    </tr>
    <tr>
      <th>Net Tangible Assets</th>
      <td>65339000</td>
      <td>63090000</td>
      <td>50672000</td>
    </tr>
    <tr>
      <th>Ordinary Shares Number</th>
      <td>16976763</td>
      <td>16426786</td>
      <td>15943425</td>
    </tr>
    <tr>
      <th>Share Issued</th>
      <td>16976763</td>
      <td>16426786</td>
      <td>15943425</td>
    </tr>
    <tr>
      <th>Tangible Book Value</th>
      <td>65339000</td>
      <td>63090000</td>
      <td>50672000</td>
    </tr>
    <tr>
      <th>Total Assets</th>
      <td>323888000</td>
      <td>351002000</td>
      <td>352755000</td>
    </tr>
    <tr>
      <th>Total Capitalization</th>
      <td>164006000</td>
      <td>172196000</td>
      <td>149631000</td>
    </tr>
    <tr>
      <th>Total Debt</th>
      <td>112436000</td>
      <td>124719000</td>
      <td>120069000</td>
    </tr>
    <tr>
      <th>Total Equity Gross Minority Interest</th>
      <td>65339000</td>
      <td>63090000</td>
      <td>50672000</td>
    </tr>
    <tr>
      <th>Total Liabilities Net Minority Interest</th>
      <td>258549000</td>
      <td>287912000</td>
      <td>302083000</td>
    </tr>
    <tr>
      <th>Working Capital</th>
      <td>38321000</td>
      <td>9355000</td>
      <td>-18577000</td>
    </tr>
  </tbody>
</table>
</div>




```python
url = "https://stock-analysis.p.rapidapi.com/api/v1/resources/key-stats"

querystring = {"ticker":company_ticker}

headers = {
	"X-RapidAPI-Key": "acc452e6bamshe911abf8e81f8a8p1a5578jsn80a5ba5a41c2",
	"X-RapidAPI-Host": "stock-analysis.p.rapidapi.com"
}

key_stats_response = requests.get(url, headers=headers, params=querystring)

# Create Key Statistics DataFrame
key_stats_df = pd.DataFrame.from_dict(key_stats_response.json())
key_stats_df = key_stats_df.transpose()
key_stats_df
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
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>% held by insiders</th>
      <td>0.07%</td>
    </tr>
    <tr>
      <th>% held by institutions</th>
      <td>61.32%</td>
    </tr>
    <tr>
      <th>200-day moving average</th>
      <td>167.12</td>
    </tr>
    <tr>
      <th>5-year average dividend yield</th>
      <td>0.85</td>
    </tr>
    <tr>
      <th>50-day moving average</th>
      <td>179.39</td>
    </tr>
    <tr>
      <th>52-week change</th>
      <td>26.40%</td>
    </tr>
    <tr>
      <th>52-week high</th>
      <td>198.23</td>
    </tr>
    <tr>
      <th>52-week low</th>
      <td>124.17</td>
    </tr>
    <tr>
      <th>Avg vol (10-day)</th>
      <td>54.63M</td>
    </tr>
    <tr>
      <th>Avg vol (3-month)</th>
      <td>58.54M</td>
    </tr>
    <tr>
      <th>Beta (5Y monthly)</th>
      <td>1.31</td>
    </tr>
    <tr>
      <th>Book value per share (mrq)</th>
      <td>3.85</td>
    </tr>
    <tr>
      <th>Current ratio (mrq)</th>
      <td>0.98</td>
    </tr>
    <tr>
      <th>Diluted EPS (ttm)</th>
      <td>5.96</td>
    </tr>
    <tr>
      <th>Dividend date</th>
      <td>16 Aug 2023</td>
    </tr>
    <tr>
      <th>EBITDA</th>
      <td>123.96B</td>
    </tr>
    <tr>
      <th>Enterprise value</th>
      <td>2.82T</td>
    </tr>
    <tr>
      <th>Enterprise value/EBITDA</th>
      <td>22.24</td>
    </tr>
    <tr>
      <th>Enterprise value/revenue</th>
      <td>7.35</td>
    </tr>
    <tr>
      <th>Ex-dividend date</th>
      <td>10 Aug 2023</td>
    </tr>
    <tr>
      <th>Fiscal year ends</th>
      <td>23 Sept 2022</td>
    </tr>
    <tr>
      <th>Float</th>
      <td>15.62B</td>
    </tr>
    <tr>
      <th>Forward P/E</th>
      <td>26.81</td>
    </tr>
    <tr>
      <th>Forward annual dividend rate</th>
      <td>0.96</td>
    </tr>
    <tr>
      <th>Forward annual dividend yield</th>
      <td>0.54%</td>
    </tr>
    <tr>
      <th>Gross profit (ttm)</th>
      <td>170.78B</td>
    </tr>
    <tr>
      <th>Implied shares outstanding</th>
      <td>15.63B</td>
    </tr>
    <tr>
      <th>Last split date</th>
      <td>30 Aug 2020</td>
    </tr>
    <tr>
      <th>Last split factor</th>
      <td>4:1</td>
    </tr>
    <tr>
      <th>Levered free cash flow (ttm)</th>
      <td>90.68B</td>
    </tr>
    <tr>
      <th>Market cap (intra-day)</th>
      <td>2.77T</td>
    </tr>
    <tr>
      <th>Most-recent quarter (mrq)</th>
      <td>30 Jun 2023</td>
    </tr>
    <tr>
      <th>Net income avi to common (ttm)</th>
      <td>94.76B</td>
    </tr>
    <tr>
      <th>Operating cash flow (ttm)</th>
      <td>113.07B</td>
    </tr>
    <tr>
      <th>Operating margin (ttm)</th>
      <td>29.23%</td>
    </tr>
    <tr>
      <th>PEG Ratio (5 yr expected)</th>
      <td>2.27</td>
    </tr>
    <tr>
      <th>Payout ratio</th>
      <td>15.60%</td>
    </tr>
    <tr>
      <th>Price/book (mrq)</th>
      <td>46.03</td>
    </tr>
    <tr>
      <th>Price/sales (ttm)</th>
      <td>7.36</td>
    </tr>
    <tr>
      <th>Profit margin</th>
      <td>24.68%</td>
    </tr>
    <tr>
      <th>Quarterly earnings growth (yoy)</th>
      <td>2.30%</td>
    </tr>
    <tr>
      <th>Quarterly revenue growth (yoy)</th>
      <td>-1.40%</td>
    </tr>
    <tr>
      <th>Return on assets (ttm)</th>
      <td>20.90%</td>
    </tr>
    <tr>
      <th>Return on equity (ttm)</th>
      <td>160.09%</td>
    </tr>
    <tr>
      <th>Revenue (ttm)</th>
      <td>383.93B</td>
    </tr>
    <tr>
      <th>Revenue per share (ttm)</th>
      <td>24.22</td>
    </tr>
    <tr>
      <th>S&amp;P500 52-week change</th>
      <td>19.27%</td>
    </tr>
    <tr>
      <th>Shares outstanding</th>
      <td>15.63B</td>
    </tr>
    <tr>
      <th>Shares short (14 Sept 2023)</th>
      <td>107.33M</td>
    </tr>
    <tr>
      <th>Shares short (prior month 14 Aug 2023)</th>
      <td>88.85M</td>
    </tr>
    <tr>
      <th>Short % of float (14 Sept 2023)</th>
      <td>0.69%</td>
    </tr>
    <tr>
      <th>Short % of shares outstanding (14 Sept 2023)</th>
      <td>0.69%</td>
    </tr>
    <tr>
      <th>Short ratio (14 Sept 2023)</th>
      <td>1.72</td>
    </tr>
    <tr>
      <th>Total cash (mrq)</th>
      <td>62.48B</td>
    </tr>
    <tr>
      <th>Total cash per share (mrq)</th>
      <td>4</td>
    </tr>
    <tr>
      <th>Total debt (mrq)</th>
      <td>109.28B</td>
    </tr>
    <tr>
      <th>Total debt/equity (mrq)</th>
      <td>181.30%</td>
    </tr>
    <tr>
      <th>Trailing P/E</th>
      <td>29.78</td>
    </tr>
    <tr>
      <th>Trailing annual dividend rate</th>
      <td>0.93</td>
    </tr>
    <tr>
      <th>Trailing annual dividend yield</th>
      <td>0.53%</td>
    </tr>
  </tbody>
</table>
</div>





**2a. Estimating the Required Return on Equity**<br>
In order to calculate the Required Return on Equity, we will use the Capital Asset Pricing Model (CAPM) which is an equilibrium model that takes the risk-free rate, the stock's equity beta, and the market risk premium as inputs.

$$r_e = r_f\ +(\beta*MRP)$$<br>
$r_e = required\ return\ on\ equity$<br>
$r_f = risk-free\ rate$<br>
$\beta = beta$<br>
$MRP = market\ risk\ premium$<br>

*Estimating the Risk-Free Rate*<br>
Government securities are assumed to be risk-free, at least from a credit standpoint.  With this assumption, the appropriate rate to use in the CAPM is the government security having approximately the same duration as the asset being valued and sufficient liquidity so that the yield does not have an embedded liquidity risk premium.  Equities are assumed to have a long duration, so a long-term government bond yield is an appropriate proxy for the risk-free rate.

In this model, the yield on the 10 Year U.S. Treasury Note will be used as the risk-free rate.  We can scrape the current yield on the 10 Year U.S. Treasury Note from Yahoo Finance using the code below.


```python
import yfinance as yf
import datetime

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

```




    0.047839999198913574



*Estimating the Stock's Beta*<br>
A stock's beta is the sensitivity of that stock's return to the return of the market index.  It is a measure of systematic risk and can be obtained using the [Key Statistics](https://rapidapi.com/alphawave/api/stock-analysis?endpoint=apiendpoint_dff4b882-4be4-4169-a700-04275c92bdce) endpoint from the [AlphaWave Data Stock Analysis API](https://rapidapi.com/alphawave/api/stock-analysis/endpoints) using the code below.


```python
# Stock's Beta
equity_beta = float(key_stats_df.loc[r'Beta (5Y monthly)'])
equity_beta
```




    1.31



*Estimating the Market Risk Premium*<br>
The market risk premium should be the expected return on the market index less the expected return (or yield) on the long-term government bond.  For our purposes, we use the annual [market risk premium](http://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/ctryprem.html) provided by Aswath Damodaran, who is a professor at the Stern School of Business at New York University.  Professor Damodaran teaches Corporate Finance and Equity Instruments & Markets.  He provides a multitude of financial information and I encourage you to visit his [website](http://pages.stern.nyu.edu/~adamodar/).


```python
# Market Risk Premium
#Market Risk Premium = Return of market(5% most likely) - Free risk rate
market_risk_premium = (0.05-)
```

Using the CAPM, we can now calculate the required return on equity for the company.


```python
# Required Cost of Equity

equity_return = risk_free_rate + (equity_beta*market_risk_premium)
equity_return
```




    0.0795350000667572



**2b. Estimating the Required Return on Debt**<br>
Firms have a cost of debt, which is the interest rate a company pays on its debt, such as bonds and loans. From the lender's perspective, this rate is the required return on debt. To calculate the required return on debt, we use the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints) to pull the firm's Interest Expense from the [Income Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_b4be9cff-500e-4b0a-a5a9-a8d73f9cde7a) endpoint and Total Debt from the [Balance Sheet Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_df251606-fb96-4468-9067-0d65886ce5b9) endpoint.

After dividing Interest Expense by Total Debt to calculate the required rate of return on debt, we will multiply this rate by (1 - effective tax rate) as seen in the WACC formula in order to get the after-tax required return on debt.  The reason for this is because there are tax deductions available on interest paid, which are often to the firm's benefit. As a result, the net cost of a company's debt is the amount of interest the firm is paying minus the amount it has saved in taxes because of its tax-deductible interest payments.

Using the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints), we can calculate the effective tax rate by taking the average of the Tax Provision values returned through the [Income Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_b4be9cff-500e-4b0a-a5a9-a8d73f9cde7a) endpoint.

The code below gives a good approximation for the required rate of return on debt and the effective tax rate that will be used in the WACC calculation.


```python
# Interest Expense
interest_expense = income_statement_df.loc['Interest Expense']
interest_expense_df = interest_expense.to_frame().transpose()
interest_expense_str = interest_expense_df.values[0][-1:]
interest_expense_int = int(interest_expense_str)

# Total Debt
total_debt = balance_sheet_df.loc['Total Debt']
total_debt_df = total_debt.to_frame().transpose()
total_debt_str = total_debt_df.values[0][-1:]
total_debt_int = int(total_debt_str)

# Required Return on Debt
debt_return = interest_expense_int / total_debt_int
debt_return
```




    0.025552314205414636




```python
# Effective Tax Rate
effective_tax_rate = income_statement_df.loc['Tax Provision'].astype(int) \
                    / income_statement_df.loc['Pretax Income'].astype(int)
avg_effective_tax_rate = sum(effective_tax_rate) / len(effective_tax_rate)
avg_effective_tax_rate
```




    0.18317664597985506



**2c. Estimating the Capital Structure**<br>
The final inputs for calculating WACC deal with the firm's capital structure, which is a company's mix of equity and debt.  As previously noted, WACC is a blend of a company’s equity and debt cost of capital based on the company's equity and debt capital ratio.

To calculate the market value of equity for a firm, we can use the [AlphaWave Data Stock Analysis API](https://rapidapi.com/alphawave/api/stock-analysis/endpoints) to pull the firm's Market Cap from the [Key Statistics](https://rapidapi.com/alphawave/api/stock-analysis?endpoint=apiendpoint_dff4b882-4be4-4169-a700-04275c92bdce) endpoint.


```python
# Market Value of Equity
market_cap_str = key_stats_df.loc[r'Market cap (intra-day) '][0]

market_cap_lst = market_cap_str.split('.')
if market_cap_str[len(market_cap_str)-1] == 'T':
    market_cap_length = len(market_cap_lst[1])-1
    market_cap_lst[1] = market_cap_lst[1].replace('T',(9-market_cap_length)*'0')
    market_cap_int = int(''.join(market_cap_lst))
if market_cap_str[len(market_cap_str)-1] == 'B':
    market_cap_length = len(market_cap_lst[1])-1
    market_cap_lst[1] = market_cap_lst[1].replace('B',(6-market_cap_length)*'0')
    market_cap_int = int(''.join(market_cap_lst))

market_cap_int
```




    2030000000



In most cases, you can use the book value of debt from a company’s latest balance sheet as an approximation for market value of debt. Unlike equity, the market value of debt usually does not deviate too far from the book value.  To calculate the market value of debt, we can use the [AlphaWave Data Financial Statements API](https://rapidapi.com/alphawave/api/financial-statements/endpoints) to pull the firm's Net Debt from the [Balance Sheet Statement](https://rapidapi.com/alphawave/api/financial-statements?endpoint=apiendpoint_df251606-fb96-4468-9067-0d65886ce5b9) endpoint.


```python
# Market Value of Debt
net_debt = balance_sheet_df.loc['Net Debt']
net_debt_df = net_debt.to_frame().transpose()
net_debt_str = net_debt_df.values[0][-1:]
net_debt_int = int(net_debt_str)
net_debt_int
```




    74420000



The company value is the sum of its equity and debt market values.


```python
# Company Value
company_value = market_cap_int + net_debt_int
company_value
```




    2104420000



We are now able to determine the equity and debt capital ratios by dividing both the market value of equity and the market value of debt by the company value.

With all the components acquired, we can calculate the firm's WACC.


```python
WACC = ((market_cap_int/company_value) * equity_return) \
        + ((net_debt_int/company_value) * (debt_return * (1-avg_effective_tax_rate)))
WACC
```




    0.07746045183864433



### 3. Stock Valuation

We need to discount all the forecasted cash flows to get the Firm Value. First, discount the forecasted FCFF for each of the next five years (Stage 1). Then, the present value of the terminal value of the firm using the forecasted FCFF in Year 6 is added to this amount (Stage 2).

$$Firm\ Value = \sum_{t=1}^n\overbrace{\frac{FCFF_t}{(1 + WACC)^t}}^{Stage\ 1}+\overbrace{\frac{FCFF_{n+1}}{(WACC - g)}*\frac{1}{(1 + WACC)^n}}^{Stage\ 2}$$<br>

$FCFF_t = expected\ free\ cash\ flow\ to\ the\ firm\ in\ year\ t$<br>
$FCFF_{n+1} = expected\ free\ cash\ flow\ to\ the\ firm\ in\ the\ last\ year\ (Year\ 6\ in\ this\ example)$<br>
$g = constant\ expected\ growth\ rate\ in\ FCFF$<br>
$WACC = weighted\ average\ cost\ of\ capital$<br>

As the resulting value gives us the Firm Value of the company (which is the value to both equity and debt holders), we need to deduct the company’s Market Value of Debt to arrive at the Equity Value of the firm.

Equity Value = Firm Value – Market Value of Debt


```python
# Equity Value Calculation
discounted_FCFF_lst = []
for year in range(0,5):
    discounted_FCFF = forecast_free_cash_flow_firm_df.iloc[0,year]/(1+WACC)**(year+1)
    discounted_FCFF_lst.append(int(discounted_FCFF))
terminal_value = forecast_free_cash_flow_firm_df.iloc[0,5]/(WACC-long_term_growth)
PV_terminal_value = int(terminal_value/(1+WACC)**5)
enterprise_value = sum(discounted_FCFF_lst)+PV_terminal_value
equity_value = enterprise_value-net_debt_int
equity_value
```




    1953329634



The [Key Statistics](https://rapidapi.com/alphawave/api/stock-analysis?endpoint=apiendpoint_dff4b882-4be4-4169-a700-04275c92bdce) endpoint from the [AlphaWave Data Stock Analysis API](https://rapidapi.com/alphawave/api/stock-analysis/endpoints) enables us to pull in the firm's total Shares Outstanding, which we will use to calculate the two-stage FCFF valuation model's stock price estimate for the firm.


```python
# Total Shares Outstanding
shares_outstanding_str = key_stats_df.loc[r'Shares outstanding '][0]

shares_outstanding_lst = shares_outstanding_str.split('.')
if shares_outstanding_str[len(shares_outstanding_str)-1] == 'T':
    shares_outstanding_length = len(shares_outstanding_lst[1])-1
    shares_outstanding_lst[1] = shares_outstanding_lst[1].replace('T',(9-shares_outstanding_length)*'0')
    shares_outstanding_int = int(''.join(shares_outstanding_lst))
if shares_outstanding_str[len(shares_outstanding_str)-1] == 'B':
    shares_outstanding_length = len(shares_outstanding_lst[1])-1
    shares_outstanding_lst[1] = shares_outstanding_lst[1].replace('B',(6-shares_outstanding_length)*'0')
    shares_outstanding_int = int(''.join(shares_outstanding_lst))

shares_outstanding_int
```




    17100000




```python
# Two-stage FCFF Valuation Model Stock Price Estimate
stock_price = equity_value / shares_outstanding_int
stock_price = '${:,.2f}'.format(stock_price)
print("Model Stock Price = %s"%(stock_price))

# Actual Stock Price
actual_stock_price = market_cap_int / shares_outstanding_int
actual_stock_price = '${:,.2f}'.format(actual_stock_price)
print("Actual Stock Price = %s"%(actual_stock_price))
```

    Model Stock Price = $114.23
    Actual Stock Price = $118.71
    

### 4. Conclusion

As can be seen in this article, we were able to calculate Stock Valuation using FCFF and WACC.  In the example above, the two-stage FCFF valuation model does a reasonably good job at estimating the fair value of a company's stock price.  However, the model is very sensitive to the discount and growth rates selected.  When applying this model, it is important to consider the type of growth the company is currently experiencing along with the anticipated growth characteristics for the future. A single-stage FCFF valuation model is useful for stable firms in mature industries while two-stage and three-stage FCFF valuation models are useful when you want to model multiple phases of growth.

If you liked this article, you may also enjoy AlphaWave Data's [Bayesian Pairs Trading using Corporate Supply Chain Data](https://medium.com/@hdonnelly6/bayesian-pairs-trading-using-corporate-supply-chain-data-8b96305686d) article.  Using the [AlphaWave Data Corporate Supply Chain API](https://rapidapi.com/alphawave/api/corporate-supply-chain/endpoints), we can get a list of suppliers and customers for a given stock symbol.  You can further filter this list of suppliers and customers using the two-stage FCFF valuation model discussed in this article to get a grouping of good pairs trading candidates that are potentially misvalued.  A list of company tickers that have supply chain data is included at the bottom of the Jupyter Notebooks available on [Google Colab](https://colab.research.google.com/drive/1e_SiiZn7WEW3OUNG-ftN3riPNjiz0M0C?usp=sharing) and [Github](https://github.com/AlphaWaveData/Jupyter-Notebooks/blob/master/AlphaWave%20Corporate%20Supply%20Chain%20API%20Example.ipynb), which could be a good list of tickers to run through the two-stage FCFF valuation model discussed above.


```python

```
