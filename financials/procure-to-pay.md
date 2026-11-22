# Procure to Pay

**What this flow covers:** the company needs something, orders it from a supplier, receives it, gets the supplier's bill, pays it, and everything lands in the books.

**Modules involved:** Purchasing (Self Service Procurement and Purchasing), Receiving, Payables, Payments, Subledger Accounting (SLA), General Ledger (GL).

**Example used:** the company buys goods worth 1,000.

Read the flowchart top to bottom. A diamond is a yes or no question. A "No" follows the dashed red path to a fix, then back to retry the step.

![Procure to Pay flowchart](../images/procure-to-pay.svg)

## The steps in plain English

| # | What happens | Fusion job or screen | Main tables | Done when |
|---|---|---|---|---|
| 1 | Someone asks to buy goods | Create Requisition | `POR_REQUISITION_HEADERS_ALL`, `POR_REQUISITION_LINES_ALL` | Status `APPROVED` after approval |
| 2 | The requisition becomes a purchase order | Process Requisitions | `PO_HEADERS_ALL`, `PO_LINES_ALL`, `PO_LINE_LOCATIONS_ALL`, `PO_DISTRIBUTIONS_ALL` | Order status `OPEN` after approval |
| 3 | The order goes to the supplier | Communicate purchase order | `PO_HEADERS_ALL` | Order communicated |
| 4 | The goods arrive and are received | Receiving, Manage Receipts | `RCV_SHIPMENT_HEADERS`, `RCV_TRANSACTIONS` | Receipt transaction created |
| 5 | The receipt is accounted (the company now owes for goods received) | Create Accounting | `XLA_EVENTS`, `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 6 | The supplier's invoice is entered and matched to the order | Create Invoice (match to PO) | `AP_INVOICES_ALL`, `AP_INVOICE_LINES_ALL` | Lines carry the PO reference |
| 7 | The invoice is checked | Validate Payables Invoice | `AP_INVOICES_ALL`, `AP_HOLDS_ALL` | `MATCH_STATUS_FLAG = A`, no holds |
| 8 | The invoice becomes an accounting entry | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 9 | The supplier is paid | Payment Process Request | `IBY_PAYMENTS_ALL`, `AP_CHECKS_ALL` | Payment status `ISSUED` |
| 10 | The payment becomes an accounting entry | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 11 | Entries go to the ledger | Transfer Journal Entries to GL | `GL_INTERFACE` | Transfer status `Y` |
| 12 | Journals are created and posted | Import Journals, Post Journals | `GL_JE_BATCHES`, `GL_JE_LINES`, `GL_BALANCES` | Batch status `P` (posted) |

## The accounting entries

| Event | Debit | Credit |
|---|---|---|
| Goods received | Inventory (or Expense) 1,000 | Receipt accrual 1,000 |
| Invoice | Receipt accrual 1,000 | Payables liability 1,000 |
| Payment | Payables liability 1,000 | Cash 1,000 |

The receipt accrual account is a temporary holding account. It is filled when the goods arrive and cleared when the invoice is accounted, so it ends at zero.

## Why matching matters

Payables compares the invoice to the purchase order (and the receipt, in a three-way match). If the quantity or price is off by more than the allowed tolerance, a hold is placed and the invoice cannot be paid until it is fixed or released.

## Where it usually gets stuck

- **Requisition not turning into an order:** it is not approved yet, or the buyer has not processed it.
- **Invoice on hold:** the invoice price or quantity does not match the order or the receipt.
- **Receipt not accounted:** the receipt accounting event is in error, often a missing account.
- **Nothing in the ledger:** the transfer status is still `N`, or the period is closed.

## Accuracy note

Job names, table names, column names and status codes are written from working knowledge of Oracle Fusion. They have not been checked against Oracle's documentation or a real environment. Receipt accounting is the area I am least sure about, so verify those step names first.

[Back to all flows](../README.md)
