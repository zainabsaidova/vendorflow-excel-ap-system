# User Guide

A short manual for the two people who touch this workbook: the employee who records data, and the manager who reads it.

---

## The one rule

**Type only in yellow cells.**

| Appearance | Meaning |
|---|---|
| Blue text on light yellow | Input — type here |
| Black text, white background | Calculated — a formula fills it in |
| Green text | Linked — pulled automatically from another sheet |
| Green fill | Completed, or a check that passed |
| Amber fill | Warning: partly paid, due soon, needs review |
| Red fill | Overdue, or a data error to fix |
| Grey | Closed or inactive — no action needed |

Overwriting a calculated cell replaces a formula with a number. That number will never update again, and the reconciliation controls will start to fail. If you overwrite one by accident, copy the formula down from the row above.

---

## For the employee — daily workflow

### A new supplier starts delivering

Go to **Vendor Master** and fill the next empty row:

| Column | What to enter |
|---|---|
| Vendor ID | The next free number in sequence: `VEN-013`, `VEN-014`… Never reuse an ID |
| Vendor Name | Full legal name |
| Category | Pick from the dropdown |
| Contact Person / Email / Phone | Who you call when something is wrong |
| Payment Terms (Days) | 15, 30, 45, 60… This single number drives every due date for this supplier |
| Vendor Status | Active or Inactive |

Columns I onwards fill themselves as soon as invoices exist.

### Goods arrive and an invoice comes in

Go to **Invoice & Delivery Log**, next empty row:

| Column | What to enter |
|---|---|
| Invoice ID | Next in sequence: `INV-1061`. This is your internal key — it must be unique |
| Vendor ID | **Pick from the dropdown.** If the supplier is not listed, add them to Vendor Master first |
| Supplier Invoice No. | The reference printed on the supplier's own document |
| PO Number | The purchase order that approved this spend |
| Description | A few words: what was bought |
| Invoice Date | The date on the invoice |
| Expected Delivery Date | When the goods were promised |
| Actual Delivery Date | When they actually arrived |
| Goods Receipt Status | Received in Full / Partially Received / Rejected |
| Invoice Amount | The total on the invoice |

Everything else — due date, balance, status, overdue days, aging — appears immediately.

### Money leaves the bank

Go to **Payments Log**, next empty row:

| Column | What to enter |
|---|---|
| Payment ID | Next in sequence: `PAY-2064` |
| Invoice ID | **Pick from the dropdown** — this is what links the payment to the invoice |
| Payment Date | The date the money actually left |
| Payment Amount | How much was paid **in this transaction** |
| Payment Method | Pick from the dropdown |
| Bank Reference | The transaction reference, for the audit trail |
| Notes | Anything unusual |

### Partial payments

This is the part people get wrong. A supplier bills $5,000 and you pay $2,000 now and $3,000 later.

❌ **Do not** change the invoice amount to $2,000.
❌ **Do not** overwrite the first payment row with the new total.

✅ **Do** add a second row to the Payments Log with the same Invoice ID and `3000`.

The invoice sums its own payments. After the first row it shows `Partially Paid` with $3,000 outstanding; after the second it shows `Paid` with zero. The full history of both transactions stays visible, which is exactly what an auditor asks for.

### Every morning

Open **Data Quality & Reconciliation**.

1. The banner at the top should be green: *all controls passed*.
2. If any control reads `FAIL`, something does not balance — **do not fix the control formula**. Find the data error.
3. Part 2 lists exceptions. Each line tells you how many records are involved and how much money.

To find the actual rows: open the relevant log, click the filter arrow on the **Data Check** column, and keep only `REVIEW`. The **Issue Details** column next to it spells out in words what is wrong with that row.

---

## For the manager

### Dashboard

| KPI | Read it as |
|---|---|
| Total Invoiced | Everything suppliers have billed |
| Total Paid | Everything settled so far |
| Total Outstanding | The net ledger balance, including supplier credits |
| Total Overdue | How much is already past its due date — the number that costs relationships |
| Open / Overdue Invoices | Count of invoices needing attention |
| On-Time Payment Rate | Share of payments made by the due date — your reliability as a customer |
| Avg Days to Pay | How long you take, on average, from invoice to payment |

The **AP Aging** table shows how old the debt is. Anything past 60 days is usually a conversation, not a payment run. A `Supplier Credit` line means an invoice was overpaid and money should be reclaimed.

**Top 5 Vendors by Outstanding Balance** shows concentration risk. **Next 5 Payments Due** is the immediate action list.

### Payment Planner — the weekly cash decision

The summary strip answers: *how much cash do I need, and by when?*

- **Overdue Now** — should have been paid already
- **Due in 0–7 / 8–14 / 15–30 / 31+ Days** — the forward schedule
- **Available Cash Budget** — type your real number here
- **Budget After Plan** — turns red if the plan exceeds the budget

Work down the list by **Priority Rank** (1 = most urgent). Enter a Planned Payment Date and Planned Amount. The **Plan Status** column warns you if a planned amount exceeds the balance, if a date is missing, or if the date is already in the past.

### Vendor performance

On **Vendor Master**, the right-hand columns rate each supplier:

- **On-Time Delivery Rate** — did they deliver when promised
- **Full Receipt Rate** — did they deliver complete orders
- **Supplier Score** — 60% punctuality, 40% completeness
- **Performance Rating** — A, B, C or D

A supplier sitting at C or D with a large outstanding balance is the one to renegotiate with.

---

## Settings

All of these live on the **Settings** sheet and change the whole workbook at once.

| Setting | Effect |
|---|---|
| Reporting Date (`C5`) | Every overdue, aging and planning figure is measured against this date. Replace with `=TODAY()` to make the file self-updating |
| Overdue Grace Days | Tolerance after the due date before an invoice counts as late |
| Aging Band limits | The 30 / 60 / 90 day boundaries |
| Due Soon Window | How many days ahead count as "due soon" |
| Planning Horizon | Length of the cash-planning window |
| Scoring weights | How the supplier score splits between punctuality and completeness |
| Rating thresholds | Where A, B and C begin |

The dropdown lists further down the sheet feed every dropdown in the workbook. Add a value there and it appears immediately.

---

## Running out of rows

Formulas are prepared down to row **157** on the Invoice & Delivery Log and row **207** on the Payments Log. To extend: select the entire last formula row, copy, and paste into the rows below. Then widen the ranges in the reconciliation formulas if you go far beyond the prepared area.

## Google Sheets

File → Import → Upload. Formulas, conditional formatting and dropdowns carry across. Charts may need to be re-inserted. Nothing in the workbook depends on a function Sheets lacks.
