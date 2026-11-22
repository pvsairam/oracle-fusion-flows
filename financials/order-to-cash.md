# Order to Cash

**What this flow covers:** a customer orders goods, the company ships them, bills the customer, collects the money, and everything lands in the books.

**Modules involved:** Order Management, Shipping and Inventory, Receivables, Cost accounting, Subledger Accounting (SLA), General Ledger (GL).

**Example used:** a customer orders goods and is invoiced 1,500.

Read the flowchart top to bottom. A diamond is a yes or no question. A "No" follows the dashed red path to a fix, then back to retry the step.

![Order to Cash flowchart](../images/order-to-cash.svg)

## The steps in plain English

| # | What happens | Fusion job or screen | Main tables | Done when |
|---|---|---|---|---|
| 1 | The sales order is entered and submitted | Order Management | `DOO_HEADERS_ALL`, `DOO_LINES_ALL` | Order moves past `DRAFT` |
| 2 | The goods are picked, packed and shipped | Shipping, Ship Confirm | `WSH_NEW_DELIVERIES` | Delivery is ship confirmed |
| 3 | The cost of the goods shipped is accounted | Create Accounting | `XLA_EVENTS`, `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 4 | The shipped order is sent to billing | Import AutoInvoice | `RA_INTERFACE_LINES_ALL`, `RA_INTERFACE_ERRORS_ALL` | Interface rows processed |
| 5 | Receivables creates the invoice | AutoInvoice, Manage Transactions | `RA_CUSTOMER_TRX_ALL`, `RA_CUSTOMER_TRX_LINES_ALL`, `RA_CUST_TRX_LINE_GL_DIST_ALL` | Invoice `COMPLETE_FLAG = Y` |
| 6 | The invoice is accounted (the customer now owes the money) | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 7 | The customer pays and a receipt is created | Create Receipt | `AR_CASH_RECEIPTS_ALL` | Receipt created and confirmed |
| 8 | The receipt is applied to the invoice | Apply Receipt | `AR_RECEIVABLE_APPLICATIONS_ALL`, `AR_PAYMENT_SCHEDULES_ALL` | Invoice status `CL` (closed) |
| 9 | The receipt is accounted | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 10 | Entries go to the ledger | Transfer Journal Entries to GL | `GL_INTERFACE` | Transfer status `Y` |
| 11 | Journals are created and posted | Import Journals, Post Journals | `GL_JE_BATCHES`, `GL_JE_LINES`, `GL_BALANCES` | Batch status `P` (posted) |

## The accounting entries

| Event | Debit | Credit |
|---|---|---|
| Goods shipped | Cost of goods sold | Inventory |
| Invoice | Receivables 1,500 | Revenue 1,500 |
| Receipt | Cash 1,500 | Receivables 1,500 |

The cost entry uses the item's cost, not the selling price, so its amount is different from 1,500. After the receipt, Receivables is back to zero for this sale.

## Where it usually gets stuck

- **Order not shipping:** the order is on hold, or there is no stock to pick.
- **No invoice after shipping:** AutoInvoice rejected the rows. Look in `RA_INTERFACE_ERRORS_ALL`.
- **Invoice stays open:** the receipt was not applied, or only part was paid. Check `AR_PAYMENT_SCHEDULES_ALL` for status `OP` (open).
- **Nothing in the ledger:** an accounting event is in error, or the transfer status is still `N`.

## Accuracy note

Job names, table names, column names and status codes are written from working knowledge of Oracle Fusion. They have not been checked against Oracle's documentation or a real environment. The Order Management and shipping steps are the ones I am least sure about, so verify those first.

[Back to all flows](../README.md)
