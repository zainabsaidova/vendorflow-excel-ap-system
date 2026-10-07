# Data Dictionary

Every column in the workbook, what it holds, and where it comes from.

**Legend** — 🟡 Input (typed by an employee) · ⚪ Calculated (formula) · 🔗 Linked (pulled from another sheet)

All sheets share the same layout: rows 1–2 are the banner, rows 4–5 carry instructions, **row 7 is the header**, and data starts at **row 8**.

---

## Settings

Values live in **column C**. Changing one of these changes the whole workbook.

| Cell | Parameter | Default | Effect |
|---|---|---|---|
| C5 | Reporting Date | 2026-09-07 | The date every overdue, aging and planning calculation measures against. Replace with `=TODAY()` for a self-updating file |
| C6 | Currency | USD | Label only — the workbook holds one currency |
| C7 | Overdue Grace Days | 0 | Tolerance after the due date before an invoice counts as overdue |
| C8 | Aging Band 1 — Max Days | 30 | Upper limit of the 1–30 day band |
| C9 | Aging Band 2 — Max Days | 60 | Upper limit of the 31–60 day band |
| C10 | Aging Band 3 — Max Days | 90 | Upper limit of the 61–90 day band |
| C11 | Due Soon Window (Days) | 7 | Invoices due within this window are flagged as due soon |
| C12 | Planning Horizon (Days) | 30 | Length of the cash-planning window on the Planner |
| C14 | Weight — On-Time Delivery Rate | 60% | Share of the supplier score driven by punctuality |
| C15 | Weight — Full Receipt Rate | 40% | Share driven by delivery completeness |
| C17 | Rating A — Minimum Score | 90% | Score at or above this earns an A |
| C18 | Rating B — Minimum Score | 80% | Score at or above this earns a B |
| C19 | Rating C — Minimum Score | 65% | Below this the supplier is rated D |

Dropdown source lists sit in columns B–E from row 27: Vendor Status, Goods Receipt Status, Payment Method, Vendor Category.

---

## Vendor Master

One row per supplier. Rows 8–47 are prepared.

| Col | Field | Type | Definition |
|---|---|---|---|
| A | Vendor ID | 🟡 | Primary key, format `VEN-001`. Links to the Invoice & Delivery Log. Never reuse |
| B | Vendor Name | 🟡 | Full supplier name |
| C | Category | 🟡 | Spend category — dropdown from Settings |
| D | Contact Person | 🟡 | Named contact |
| E | Email | 🟡 | Contact email |
| F | Phone | 🟡 | Contact phone |
| G | Payment Terms (Days) | 🟡 | Net days. **Drives every due date for this supplier.** Validated 0–365 |
| H | Vendor Status | 🟡 | Active / Inactive — dropdown |
| I | Invoices | ⚪ | `COUNTIFS` of invoices carrying this Vendor ID |
| J | Total Invoiced | ⚪ | `SUMIFS` of invoice amounts |
| K | Total Paid | ⚪ | `SUMIFS` of amounts paid |
| L | Outstanding Balance | ⚪ | `SUMIFS` of invoice balances |
| M | Overdue Amount | ⚪ | `SUMIFS` of overdue amounts |
| N | On-Time Delivery Rate | ⚪ | On-time deliveries ÷ (on-time + late). Blank when nothing has been delivered |
| O | Full Receipt Rate | ⚪ | Deliveries marked *Received in Full* ÷ total invoices |
| P | Avg Days to Pay | ⚪ | `AVERAGEIFS` of days from invoice date to payment date |
| Q | Supplier Score | ⚪ | `weight₁ × N + weight₂ × O`, weights from Settings C14–C15 |
| R | Performance Rating | ⚪ | A / B / C / D from the thresholds in Settings C17–C19 |
| S | Rank Key (helper) | ⚪ | Outstanding balance + a tiny row offset, so `LARGE` can rank vendors without ties. **Hidden** |

---

## Invoice & Delivery Log

One row per supplier invoice. Rows 8–157 are prepared. This is the core table.

| Col | Field | Type | Definition |
|---|---|---|---|
| A | Invoice ID | 🟡 | Primary key, format `INV-1001`. Links to the Payments Log. Must be unique |
| B | Vendor ID | 🟡 | Foreign key to Vendor Master — **dropdown validated** |
| C | Vendor Name | 🔗 | `INDEX/MATCH` on Vendor ID. Shows `NOT FOUND` if the ID does not exist |
| D | Supplier Invoice No. | 🟡 | The supplier's own reference. Checked for duplicates per vendor |
| E | PO Number | 🟡 | Purchase order that approved the spend. Missing values are flagged |
| F | Category | 🔗 | Pulled from Vendor Master |
| G | Description | 🟡 | What was purchased |
| H | Invoice Date | 🟡 | Date on the invoice. Validated as a date 2020–2035 |
| I | Expected Delivery Date | 🟡 | When goods were promised |
| J | Actual Delivery Date | 🟡 | When goods arrived |
| K | Goods Receipt Status | 🟡 | Received in Full / Partially Received / Rejected — dropdown |
| L | Payment Terms (Days) | 🔗 | Pulled from Vendor Master |
| M | Due Date | ⚪ | `Invoice Date + Payment Terms` |
| N | Invoice Amount | 🟡 | Total invoice value. Validated greater than zero |
| O | Amount Paid | ⚪ | `SUMIFS` over the Payments Log for this Invoice ID |
| P | Outstanding Balance | ⚪ | `ROUND(N − O, 2)`. Negative means overpaid |
| Q | Payment Status | ⚪ | Unpaid / Partially Paid / Paid / Overpaid / Check Amount |
| R | Days Overdue | ⚪ | `MAX(0, Reporting Date − Due Date − grace)`, zero once the balance clears |
| S | Overdue Amount | ⚪ | The outstanding balance when Days Overdue > 0, otherwise zero |
| T | Aging Bucket | ⚪ | Supplier Credit / Closed / No Due Date / Not Due Yet / 1–30 / 31–60 / 61–90 / 90+ Days Overdue |
| U | Last Payment Date | ⚪ | `SUMPRODUCT(MAX(...))` — the most recent payment against this invoice |
| V | Delivery Performance | ⚪ | On Time / Late / Not Delivered / No Target Date |
| W | Data Check | ⚪ | `OK` or `REVIEW` |
| X | Issue Details | ⚪ | Plain-English list of everything wrong with this row (11 conditions) |
| Y | Open? | ⚪ | Yes when the rounded balance exceeds zero. **Hidden** |
| Z | Priority Rank | ⚪ | Urgency rank among open invoices, by due date then Invoice ID. **Hidden** |

### The 11 conditions behind *Issue Details*

Missing vendor ID · Vendor ID not in Vendor Master · duplicate invoice ID · duplicate supplier invoice number for the same vendor · missing PO number · missing invoice date · invoice amount not greater than zero · invoice date in the future · goods marked *Received in Full* with no actual delivery date · payments exceeding the invoice amount · payment made on a rejected delivery · open balance with an inactive vendor.

---

## Payments Log

One row per payment. Several rows may settle one invoice — this is how partial payments work. Rows 8–207 are prepared.

| Col | Field | Type | Definition |
|---|---|---|---|
| A | Payment ID | 🟡 | Primary key, format `PAY-2001`. Must be unique |
| B | Invoice ID | 🟡 | Foreign key to the invoice log — **dropdown validated** |
| C | Vendor ID | 🔗 | Pulled through the invoice |
| D | Vendor Name | 🔗 | Pulled through the invoice. Shows `INVOICE NOT FOUND` for an orphan payment |
| E | Payment Date | 🟡 | Date the money left. Validated as a date |
| F | Payment Amount | 🟡 | Amount of **this transaction only**. Validated greater than zero |
| G | Payment Method | 🟡 | Bank Transfer / ACH / Company Card / Check / Wire — dropdown |
| H | Bank Reference | 🟡 | Transaction reference for the audit trail |
| I | Notes | 🟡 | Free text |
| J | Invoice Date | 🔗 | Pulled from the invoice |
| K | Invoice Due Date | 🔗 | Pulled from the invoice |
| L | Days to Pay | ⚪ | Payment Date − Invoice Date. Feeds Avg Days to Pay |
| M | Days vs Due Date | ⚪ | Payment Date − Due Date. Negative is early |
| N | Payment Timing | ⚪ | On Time / Late / n/a. Feeds the On-Time Payment Rate |
| O | Cumulative Paid on Invoice | ⚪ | Running total for this invoice, using an expanding `SUMIFS` range |
| P | Balance After Payment | ⚪ | Invoice amount − cumulative paid. Negative flags the exact payment that caused an overpayment |
| Q | Data Check | ⚪ | `OK` or `REVIEW` |
| R | Issue Details | ⚪ | Plain-English list of problems (9 conditions) |

### The 9 conditions behind *Issue Details*

Missing invoice ID · invoice ID not found in the invoice log · duplicate payment ID · payment amount not greater than zero · missing payment date · payment dated before the invoice date · payment date in the future · payment exceeding the invoice balance · missing payment method.

---

## Payment Planner

Aligned one-to-one with the Invoice & Delivery Log: planner row 8 shows invoice row 8. Rows 8–157.

**Summary strip (rows 3–5):** Overdue Now · Due in 0–7 Days · Due in 8–14 Days · Due in 15–30 Days · Due in 31+ Days · Total Payable · Total Planned · Available Cash Budget (🟡 **H4, the one input**) · Budget After Plan.

| Col | Field | Type | Definition |
|---|---|---|---|
| A | Invoice ID | 🔗 | Mirrored from the invoice log |
| B | Vendor Name | 🔗 | Mirrored |
| C | Invoice Date | 🔗 | Mirrored |
| D | Due Date | 🔗 | Mirrored |
| E | Outstanding Balance | 🔗 | Shown only for open invoices; blank once closed |
| F | Days to Due Date | ⚪ | Due Date − Reporting Date. Negative means overdue |
| G | Priority Bucket | ⚪ | Closed / Overdue / Due in 0–7 / 8–14 / 15–30 / 31+ Days |
| H | Priority Rank | 🔗 | 1 = most urgent, from the invoice log's hidden rank |
| I | Planned Payment Date | 🟡 | When you intend to pay |
| J | Planned Amount | 🟡 | How much you intend to pay |
| K | Remaining After Plan | ⚪ | Outstanding − planned |
| L | Plan Status | ⚪ | No action needed / Not planned yet / Partly planned / Fully planned / Missing planned date / Planned amount exceeds balance / Planned date is in the past |

---

## Dashboard

Reads only; stores nothing. Key cell references used by the reconciliation:

| Cell | Content |
|---|---|
| A6 / C6 / F6 / H6 | Total Invoiced · Total Paid · Total Outstanding · Total Overdue |
| A10 / C10 / F10 / H10 | Open Invoices · Overdue Invoices · On-Time Payment Rate · Avg Days to Pay |
| A13 | Data-quality banner (green / amber / red) |
| A17:D24 | AP Aging table, total at row 24 |
| F17:I24 | Payment Status Summary, total at row 24 |
| A28:D32 | Top 5 Vendors by Outstanding Balance (`LARGE` on the Vendor Master rank key) |
| F28:I32 | Next 5 Payments Due (`MATCH` on the invoice priority rank) |
| A36:D43 | Monthly Invoiced vs Paid, total at row 43 |

---

## Data Quality & Reconciliation

| Range | Content |
|---|---|
| B4 | Overall status banner |
| B8:F17 | Part 1 — the 10 reconciliation controls. Column F holds `PASS` / `FAIL` |
| B23:F35 | Part 2 — the 13 exception checks. Column F holds `CLEAR` / `REVIEW` |
| B39 onward | How to fix an exception |
| Part 3 | Coverage: records held, records passing, accuracy rates, controls passed, reporting date |
