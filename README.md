# Oracle Fusion Flows

Study notes that explain how data moves through Oracle Fusion Cloud, one flowchart per topic.

I made this to learn Oracle Fusion from the functional side, starting with Financials and HCM. Each page follows one real business situation from the first step to the final entry in the General Ledger, and shows the Fusion job, the tables and the status that mark each step.

## Pick a flow

Click a name to open its page. The flowchart is shown right at the top of the page.

| Flow | What it follows | Modules involved |
|---|---|---|
| [Expense to GL](financials/expense-to-gl.md) | An employee claims an expense and gets paid | HCM, Expenses, Payables, Payments, SLA, GL |
| [Procure to Pay](financials/procure-to-pay.md) | The company orders goods, receives them and pays the supplier | Purchasing, Receiving, Payables, Payments, SLA, GL |
| [Order to Cash](financials/order-to-cash.md) | A customer orders, the company ships, bills and collects | Order Management, Shipping, Receivables, SLA, GL |
| [Fixed Assets to GL](financials/fixed-assets-to-gl.md) | The company buys an asset and depreciates it | Payables, Fixed Assets, SLA, GL |
| [Cash Management](financials/cash-management.md) | A bank statement is matched to the company's records | Cash Management, Payables, Receivables, SLA, GL |
| [Hire to Retire: Payroll to GL](hcm/hire-to-retire.md) | An employee is hired, paid through payroll, and the cost reaches the ledger | HCM, Payroll, Costing, SLA, GL |

## How to read a flow page

Every page has the same parts, in the same order:

1. **A short intro:** what the flow covers, the modules involved, and the example amount used.
2. **A flowchart:** read it top to bottom. Shaded bands show which module is doing the work. A diamond is a yes or no question, and a "No" follows the dashed red path to a fix and then a retry.
3. **A steps table:** for each step, what happens, the Fusion job, the main tables, and what shows the step is done.
4. **The accounting entries:** the debits and credits created along the way.
5. **Where it usually gets stuck:** the common reasons a flow stops, and where to look.

## The pattern behind every flow

All of these flows end the same way. Each module creates its own transaction first (an invoice, a receipt, an asset). Then **Create Accounting** in Subledger Accounting (SLA) turns it into an accounting entry. Only after that does it move to the General Ledger: Transfer to GL, then Import Journals, then Post Journals.

So when a transaction is missing from the ledger, the first question is always the same: did Create Accounting run, and did it finish with a final entry?

## How accurate is this?

I wrote the job names, table names, column names and status codes from my own working knowledge of Oracle Fusion. I have **not** checked them against Oracle's documentation or a real Fusion environment, and names can change between releases. Each page says which steps I am least sure about.

Before relying on any name, check it in Oracle's *Tables and Views for Financials* guide or in your own environment, and correct it here if it is different.

## Adding a new flow

Copy any page in `financials/` or `hcm/`, keep the same five parts, and add a row to the table above. The flowchart pictures live in `images/`. Ideas for next:

- Projects: project cost to billing
