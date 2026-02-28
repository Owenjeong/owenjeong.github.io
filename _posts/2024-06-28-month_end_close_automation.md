---
layout: single
title: "Automating Month-End Close with Python + Excel"
categories: Coding
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---


If you've ever worked in finance or accounting, you know month-end close is a grind. Pulling numbers from multiple sources, reconciling accounts, formatting reports, emailing them out. It's the same steps every month, and it's almost entirely manual at most companies.

I wanted to see how much of it I could automate with Python and Excel. Turns out, quite a lot.

---

### *1. The Problem*

First, why does month-end close even exist? It's mostly for upper management. The CFO, VP of Finance, board of directors, and investors need a clean set of financials every month to make decisions. "How did we do vs. budget? Are we burning cash too fast? Can we afford that hire?" Those questions need accurate, timely numbers. External stakeholders like auditors, lenders, and regulators also rely on these reports.

The accounting team's job is to get those numbers right and deliver them fast. A typical close looks something like this:

1. Export trial balance and subledger data from the ERP
2. Pull bank statements and credit card feeds
3. Reconcile GL accounts against source data
4. Calculate accruals and adjusting journal entries
5. Build the financial statements (P&L, Balance Sheet, Cash Flow)
6. Variance analysis vs. budget and prior month
7. Format everything into polished Excel reports
8. Email the package to leadership

The irony is that 80% of the accountant's time goes into the mechanical prep work (steps 1-3, 6-8), and maybe 20% goes into the actual accounting judgment (steps 4-5). Management doesn't care how the sausage gets made. They just want the P&L on their desk by the 5th business day. That's what makes this a perfect automation target. Speed up the grunt work so the accountant can focus on getting the numbers right.

---

### *2. The Stack*

Nothing fancy here:

| Tool | Role |
|---|---|
| **Python 3.10+** | Orchestration, data processing, logic |
| **pandas** | Data manipulation, reconciliation, aggregation |
| **openpyxl** | Read/write Excel files with formatting |
| **xlsxwriter** | Build polished Excel reports with charts |
| **smtplib / email** | Send the final package via email |
| **schedule** | Run on a cron-like schedule (or use Task Scheduler / cron) |

If your company uses a database, add `sqlalchemy` or `pyodbc` for direct SQL pulls. If your source data lives in SharePoint or Google Sheets, there are libraries for those too.

---

### *3. Step 1. Ingest Source Data*

The first step is pulling everything into pandas DataFrames. In the real world this means reading exports from your ERP, bank portals, and any other source systems.

```python
import pandas as pd
from pathlib import Path

DATA_DIR = Path("data/2024-05")

# Trial balance from ERP export
trial_balance = pd.read_excel(DATA_DIR / "trial_balance_may2024.xlsx")

# Bank statement (CSV from bank portal)
bank_stmt = pd.read_csv(DATA_DIR / "bank_statement_may2024.csv", parse_dates=["Date"])

# Credit card feed
cc_feed = pd.read_csv(DATA_DIR / "cc_transactions_may2024.csv", parse_dates=["Trans Date"])

# Budget file
budget = pd.read_excel(DATA_DIR / "annual_budget_2024.xlsx", sheet_name="May")

# Prior month actuals (for variance analysis)
prior_month = pd.read_excel(DATA_DIR / "trial_balance_apr2024.xlsx")
```

Tip: standardize column names right away so the rest of the pipeline doesn't break when the ERP changes a header.

```python
def clean_columns(df):
    df.columns = (
        df.columns
        .str.strip()
        .str.lower()
        .str.replace(" ", "_")
        .str.replace(r"[^\w]", "", regex=True)
    )
    return df

trial_balance = clean_columns(trial_balance)
bank_stmt = clean_columns(bank_stmt)
```

---

### *4. Step 2. Bank Reconciliation*

This is usually the most tedious part of close. Match GL entries to bank transactions, flag unmatched items, and compute the reconciling difference.

```python
def reconcile_bank(gl_cash: pd.DataFrame, bank: pd.DataFrame) -> dict:
    """
    Match GL cash entries against bank statement by amount and date.
    Returns matched, unmatched GL, unmatched bank, and reconciling difference.
    """
    # Merge on amount + date (within 2-day tolerance)
    bank["match_key"] = bank["amount"].round(2).astype(str)
    gl_cash["match_key"] = gl_cash["amount"].round(2).astype(str)

    merged = gl_cash.merge(bank, on="match_key", how="outer", indicator=True, suffixes=("_gl", "_bank"))

    matched = merged[merged["_merge"] == "both"]
    unmatched_gl = merged[merged["_merge"] == "left_only"]
    unmatched_bank = merged[merged["_merge"] == "right_only"]

    gl_balance = gl_cash["amount"].sum()
    bank_balance = bank["amount"].sum()
    difference = gl_balance - bank_balance

    return {
        "matched": matched,
        "unmatched_gl": unmatched_gl,
        "unmatched_bank": unmatched_bank,
        "gl_balance": gl_balance,
        "bank_balance": bank_balance,
        "difference": round(difference, 2),
    }


# Filter GL for cash account(s)
gl_cash = trial_balance[trial_balance["account"].str.startswith("1010")]
recon = reconcile_bank(gl_cash, bank_stmt)

print(f"GL Balance:   ${recon['gl_balance']:,.2f}")
print(f"Bank Balance: ${recon['bank_balance']:,.2f}")
print(f"Difference:   ${recon['difference']:,.2f}")
print(f"Unmatched GL items:   {len(recon['unmatched_gl'])}")
print(f"Unmatched Bank items: {len(recon['unmatched_bank'])}")
```

For production use, you'd want fuzzy matching on dates (transactions can post a day or two apart) and a way to handle one-to-many matches (e.g., one bank deposit covering multiple invoices).

---

### *5. Step 3. Accruals & Adjusting Entries*

Some accruals follow a formula. Rent, SaaS subscriptions, loan interest. You can calculate these automatically and generate journal entry DataFrames.

```python
from datetime import date

def generate_accruals(period_end: date) -> pd.DataFrame:
    """Generate standard monthly accruals."""
    entries = []
    
    # Example: SaaS subscription billed quarterly, accrue monthly
    saas_annual = 36000  # $36k/year
    monthly_saas = saas_annual / 12
    entries.append({
        "date": period_end,
        "account": "6200",
        "description": "SaaS subscription accrual",
        "debit": monthly_saas,
        "credit": 0,
    })
    entries.append({
        "date": period_end,
        "account": "2100",
        "description": "SaaS subscription accrual",
        "debit": 0,
        "credit": monthly_saas,
    })

    # Example: Loan interest accrual
    principal = 500000
    annual_rate = 0.065
    daily_rate = annual_rate / 365
    days_in_month = (period_end - period_end.replace(day=1)).days + 1
    interest = principal * daily_rate * days_in_month
    entries.append({
        "date": period_end,
        "account": "7100",
        "description": f"Loan interest accrual ({days_in_month} days)",
        "debit": round(interest, 2),
        "credit": 0,
    })
    entries.append({
        "date": period_end,
        "account": "2200",
        "description": f"Loan interest accrual ({days_in_month} days)",
        "debit": 0,
        "credit": round(interest, 2),
    })

    return pd.DataFrame(entries)


accruals = generate_accruals(date(2026, 2, 28))
print(accruals.to_string(index=False))
```

For anything that requires judgment (inventory reserves, bad debt estimates), I leave those as manual inputs. The script reads them from a simple "manual_entries.xlsx" file so the accountant still controls the numbers.

```python
manual_entries = pd.read_excel(DATA_DIR / "manual_entries.xlsx")
all_adjustments = pd.concat([accruals, manual_entries], ignore_index=True)
```

---

### *6. Step 4. Build Financial Statements*

Once all entries are posted, roll everything up into financials. The chart of accounts drives the mapping.

```python
# Chart of accounts mapping
COA_MAP = {
    "revenue": ["4000", "4100", "4200"],
    "cogs": ["5000", "5100"],
    "opex": ["6000", "6100", "6200", "6300", "6400"],
    "other_income": ["7000"],
    "other_expense": ["7100"],
    "assets": ["1000", "1010", "1100", "1200", "1300", "1400"],
    "liabilities": ["2000", "2100", "2200", "2300"],
    "equity": ["3000", "3100"],
}


def build_pnl(tb: pd.DataFrame) -> pd.DataFrame:
    """Build a simple P&L from trial balance."""
    rows = []
    
    revenue = tb[tb["account"].isin(COA_MAP["revenue"])]["credit"].sum()
    cogs = tb[tb["account"].isin(COA_MAP["cogs"])]["debit"].sum()
    gross_profit = revenue - cogs
    
    opex = tb[tb["account"].isin(COA_MAP["opex"])]["debit"].sum()
    operating_income = gross_profit - opex
    
    other_net = (
        tb[tb["account"].isin(COA_MAP["other_income"])]["credit"].sum()
        - tb[tb["account"].isin(COA_MAP["other_expense"])]["debit"].sum()
    )
    net_income = operating_income + other_net

    rows = [
        {"line_item": "Revenue", "amount": revenue},
        {"line_item": "COGS", "amount": -cogs},
        {"line_item": "Gross Profit", "amount": gross_profit},
        {"line_item": "Operating Expenses", "amount": -opex},
        {"line_item": "Operating Income", "amount": operating_income},
        {"line_item": "Other Income / (Expense)", "amount": other_net},
        {"line_item": "Net Income", "amount": net_income},
    ]
    return pd.DataFrame(rows)


pnl = build_pnl(trial_balance)
```

---

### *7. Step 5. Variance Analysis*

Compare actuals vs. budget and actuals vs. prior month. Flag anything above a materiality threshold.

```python
def variance_analysis(
    actuals: pd.DataFrame,
    comparator: pd.DataFrame,
    label: str = "Budget",
    threshold_pct: float = 0.10,
) -> pd.DataFrame:
    """
    Compare actuals to budget or prior period.
    Flags variances exceeding the threshold.
    """
    merged = actuals.merge(
        comparator,
        on="account",
        how="left",
        suffixes=("_actual", f"_{label.lower()}"),
    )
    
    actual_col = "amount_actual"
    comp_col = f"amount_{label.lower()}"
    
    merged["variance_$"] = merged[actual_col] - merged[comp_col]
    merged["variance_%"] = (
        merged["variance_$"] / merged[comp_col].replace(0, float("nan"))
    ).round(4)
    merged["material"] = merged["variance_%"].abs() > threshold_pct

    return merged.sort_values("variance_%", ascending=False, key=abs)


vs_budget = variance_analysis(trial_balance, budget, label="Budget")
vs_prior = variance_analysis(trial_balance, prior_month, label="Prior")

material_items = vs_budget[vs_budget["material"]]
print(f"Material variances vs budget: {len(material_items)}")
```

---

### *8. Step 6. Generate the Excel Report*

This is where openpyxl and xlsxwriter shine. You can build a multi-tab workbook with formatting, formulas, and charts that looks like someone spent hours in Excel.

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils.dataframe import dataframe_to_rows


def write_close_package(output_path: str, pnl, vs_budget, vs_prior, recon, accruals):
    wb = Workbook()

    # -- Styles --
    header_font = Font(bold=True, size=11, color="FFFFFF")
    header_fill = PatternFill(start_color="2F5496", end_color="2F5496", fill_type="solid")
    currency_fmt = '#,##0.00;(#,##0.00);"-"'
    pct_fmt = "0.0%"
    thin_border = Border(
        bottom=Side(style="thin", color="D9D9D9")
    )

    def write_sheet(ws, df, title):
        ws.title = title
        # Header row
        for col_idx, col_name in enumerate(df.columns, 1):
            cell = ws.cell(row=1, column=col_idx, value=col_name)
            cell.font = header_font
            cell.fill = header_fill
            cell.alignment = Alignment(horizontal="center")
        # Data rows
        for row_idx, row in enumerate(dataframe_to_rows(df, index=False, header=False), 2):
            for col_idx, value in enumerate(row, 1):
                cell = ws.cell(row=row_idx, column=col_idx, value=value)
                cell.border = thin_border
                if isinstance(value, (int, float)):
                    cell.number_format = currency_fmt
        # Auto-width
        for col in ws.columns:
            max_len = max(len(str(c.value or "")) for c in col)
            ws.column_dimensions[col[0].column_letter].width = min(max_len + 4, 30)

    # Tab 1: P&L
    write_sheet(wb.active, pnl, "P&L")

    # Tab 2: Variance vs Budget
    write_sheet(wb.create_sheet(), vs_budget, "Var vs Budget")

    # Tab 3: Variance vs Prior Month
    write_sheet(wb.create_sheet(), vs_prior, "Var vs Prior")

    # Tab 4: Bank Reconciliation
    recon_summary = pd.DataFrame([
        {"Item": "GL Cash Balance", "Amount": recon["gl_balance"]},
        {"Item": "Bank Statement Balance", "Amount": recon["bank_balance"]},
        {"Item": "Reconciling Difference", "Amount": recon["difference"]},
        {"Item": "Unmatched GL Items", "Amount": len(recon["unmatched_gl"])},
        {"Item": "Unmatched Bank Items", "Amount": len(recon["unmatched_bank"])},
    ])
    write_sheet(wb.create_sheet(), recon_summary, "Bank Recon")

    # Tab 5: Adjusting Entries
    write_sheet(wb.create_sheet(), accruals, "Adjusting Entries")

    wb.save(output_path)
    print(f"Saved: {output_path}")


write_close_package(
    "output/month_end_close_feb2026.xlsx",
    pnl, vs_budget, vs_prior, recon, all_adjustments
)
```

The output is a clean, formatted workbook with five tabs. Leadership gets the same polished report they're used to, they just don't know it took 30 seconds instead of 3 hours.

---

### *9. Step 7. Email the Package*

Last step. Attach the workbook and send it.

```python
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders


def send_close_package(
    to_list: list[str],
    subject: str,
    body: str,
    attachment_path: str,
    smtp_server: str = "smtp.gmail.com",
    smtp_port: int = 587,
    sender: str = "your_email@gmail.com",
    password: str = "your_app_password",
):
    msg = MIMEMultipart()
    msg["From"] = sender
    msg["To"] = ", ".join(to_list)
    msg["Subject"] = subject
    msg.attach(MIMEText(body, "plain"))

    # Attach Excel file
    with open(attachment_path, "rb") as f:
        part = MIMEBase("application", "octet-stream")
        part.set_payload(f.read())
    encoders.encode_base64(part)
    part.add_header(
        "Content-Disposition",
        f'attachment; filename="{Path(attachment_path).name}"',
    )
    msg.attach(part)

    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()
        server.login(sender, password)
        server.send_message(msg)
    
    print(f"Sent to {len(to_list)} recipients.")


send_close_package(
    to_list=["cfo@company.com", "controller@company.com"],
    subject="Month-End Close Package - May 2024",
    body="Hi team,\n\nAttached is the May 2024 month-end close package.\n\nBest,\nOwen",
    attachment_path="output/month_end_close_may2024.xlsx",
)
```

For production, store credentials in environment variables or a secrets manager. Never hardcode passwords.

---

### *10. Putting It All Together*

The full pipeline in one script:

```python
# main.py
from pathlib import Path
from datetime import date

PERIOD_END = date(2026, 2, 28)
DATA_DIR = Path(f"data/{PERIOD_END.strftime('%Y-%m')}")
OUTPUT_DIR = Path("output")
OUTPUT_DIR.mkdir(exist_ok=True)

# 1. Ingest
trial_balance = clean_columns(pd.read_excel(DATA_DIR / "trial_balance.xlsx"))
bank_stmt = clean_columns(pd.read_csv(DATA_DIR / "bank_statement.csv", parse_dates=["date"]))
budget = clean_columns(pd.read_excel(DATA_DIR / "budget.xlsx"))
prior_month = clean_columns(pd.read_excel(DATA_DIR / "prior_trial_balance.xlsx"))

# 2. Reconcile
gl_cash = trial_balance[trial_balance["account"].str.startswith("1010")]
recon = reconcile_bank(gl_cash, bank_stmt)

# 3. Accruals + manual entries
accruals = generate_accruals(PERIOD_END)
manual = pd.read_excel(DATA_DIR / "manual_entries.xlsx")
all_adjustments = pd.concat([accruals, manual], ignore_index=True)

# 4. Financials
pnl = build_pnl(trial_balance)

# 5. Variance
vs_budget = variance_analysis(trial_balance, budget, "Budget")
vs_prior = variance_analysis(trial_balance, prior_month, "Prior")

# 6. Excel report
output_file = OUTPUT_DIR / f"close_package_{PERIOD_END.strftime('%Y%m')}.xlsx"
write_close_package(str(output_file), pnl, vs_budget, vs_prior, recon, all_adjustments)

# 7. Email
send_close_package(
    to_list=["cfo@company.com", "controller@company.com"],
    subject=f"Month-End Close Package - {PERIOD_END.strftime('%B %Y')}",
    body=f"Attached is the {PERIOD_END.strftime('%B %Y')} close package.",
    attachment_path=str(output_file),
)

print("Month-end close complete.")
```

---

### *11. What This Saves*

Here's a rough before/after for a small to mid-size company:

| Task | Manual | Automated |
|---|---|---|
| Data ingestion | 30 min | 10 sec |
| Bank reconciliation | 1-2 hrs | 15 sec |
| Accrual calculations | 30 min | 5 sec |
| Financial statements | 1-2 hrs | 10 sec |
| Variance analysis | 1 hr | 10 sec |
| Excel formatting | 1-2 hrs | 10 sec |
| Email distribution | 15 min | 5 sec |
| **Total** | **5-8 hrs** | **~1 min** |

The time savings are obvious, but the real win is accuracy. No more fat-finger errors in vlookups, no more forgetting to update a cell reference, no more sending last month's file by accident.

---

### *12. "Doesn't QuickBooks Already Do This?"*

Yes, mostly. And that's a fair question.

QuickBooks, Xero, Sage, NetSuite, and pretty much every modern accounting platform already handles bank reconciliation, financial statement generation, and basic reporting. If you're a small business running everything through QuickBooks Online, you probably don't need any of this. Hit "Reconcile," run your P&L report, export to PDF, done.

So when does the Python approach actually make sense?

| Scenario | QuickBooks / ERP | Python |
|---|---|---|
| Single-source data (everything in one system) | Built-in | Overkill |
| Bank recon | Auto-matching | Only if cross-system |
| Standard P&L, Balance Sheet | One click | Not needed |
| Multi-entity consolidation | Limited or paid add-on | Full control |
| Data from multiple systems (ERP + CRM + bank + payroll) | Manual exports | Automated ingestion |
| Custom variance analysis with materiality flags | Basic or none | Fully customizable |
| Non-standard accrual logic | Manual JEs | Calculated automatically |
| Branded Excel deliverables with specific formatting | Export and reformat | Generated exactly as needed |
| Automated email distribution on a schedule | Manual | Hands-off |
| Audit trail with version-controlled scripts | Platform-dependent | Git + logs |

The short version: if your close lives entirely inside one system and the built-in reports are good enough, use the tool you're already paying for. Python shines when you're stitching together multiple data sources, need custom logic that your ERP doesn't support, or want a repeatable pipeline that runs the same way every time without someone clicking through a UI.

Most real-world month-end closes at mid-size+ companies involve data from 3-5 systems. That's where the manual work piles up and where scripting pays off.

---

### *13. Limitations*

This isn't a replacement for an accountant. It's a tool for an accountant.

- **Judgment calls are still manual.** Revenue recognition, reserve estimates, unusual transactions. The script handles the mechanical work, not the thinking.
- **Data quality matters.** Garbage in, garbage out. If the ERP export has errors, the script will process those errors faster than a human would.
- **First-time setup takes effort.** Mapping your chart of accounts, standardizing data sources, building the templates. Plan for a few weekends of setup.
- **Not an audit trail.** For SOX compliance or external audits, you'll need proper logging, version control, and approval workflows on top of this.

---

### *14. Conclusion*

Month-end close doesn't have to be a multi-day marathon. Most of the pain comes from repetitive data wrangling that Python handles effortlessly. The accountant's time is better spent reviewing the numbers and making judgment calls, not formatting spreadsheets.

Start small. Pick one piece of the close (bank recon is a good first target), automate it, validate the output against your manual process for a couple months, then expand from there. You don't need to automate everything at once.
