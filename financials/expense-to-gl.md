# Expense to GL

**What this flow covers:** an employee spends money, claims it back, gets paid, and the amount ends up in the company's books.

**Modules involved:** HCM (the employee and login), Expenses, Payables, Payments, Subledger Accounting (SLA), General Ledger (GL).

**Example used:** an employee claims a 100 hotel bill.

Read the flowchart top to bottom. A diamond is a yes or no question. A "No" follows the dashed red path to a fix, then back to retry the step.

**Colors:** 🟦 Setup  🟩 Expenses  🟨 Payables  🟪 Payments + SLA  🟥 General Ledger

```mermaid
flowchart TD
  n_s(["Start: employee has a 100 expense"]):::term
  n_hire["HR hires the employee<br/>Hire Employee · PER_ALL_PEOPLE_F"]:::c1
  n_user["User account and roles created<br/>PER_USERS · PER_USER_ROLES"]:::c1
  n_enter["Employee enters the expense report<br/>EXM_EXPENSE_REPORTS · STATUS = SAVED"]:::c2
  n_submit["Employee submits for approval<br/>STATUS = PENDING APPROVAL"]:::c2
  n_mgr{"Manager approves?"}:::d2
  n_mgr_b["Returned to employee<br/>edit and resubmit"]:::fail
  n_aud{"Audit needed?<br/>receipt or policy flag"}:::d2
  n_aud_b["Auditor reviews<br/>approves, or returns it"]:::a2
  n_appr["Report is approved<br/>STATUS = APPROVED"]:::c2
  n_exp["Export Expense Reports<br/>rows into AP_INVOICES_INTERFACE"]:::c2
  n_imp{"Import accepted?"}:::d3
  n_imp_b["Rows rejected<br/>AP_INTERFACE_REJECTIONS"]:::fail
  n_inv["AP invoice created<br/>AP_INVOICES_ALL · Import Payables Invoices"]:::c3
  n_val["Validate Payables Invoice<br/>MATCH_STATUS_FLAG"]:::c3
  n_hold{"No holds, flag = A?"}:::d3
  n_hold_b["Hold placed<br/>AP_HOLDS_ALL: fix, revalidate"]:::fail
  n_acc1["Create Accounting: invoice<br/>Dr Expense 100 / Cr Liability 100"]:::c4
  n_pay["Payment Process Request<br/>IBY_PAYMENTS_ALL · ISSUED"]:::c4
  n_acc2["Create Accounting: payment<br/>Dr Liability 100 / Cr Cash 100"]:::c4
  n_fin{"Entries final?<br/>ENTRY = F"}:::d4
  n_fin_b["Event in error<br/>fix account or period, rerun"]:::fail
  n_xfer["Transfer Journal Entries to GL<br/>GL_INTERFACE"]:::c5
  n_impj["Import Journals<br/>GL_JE_BATCHES · STATUS = U"]:::c5
  n_per{"Period open, journal approved?"}:::d5
  n_per_b["Journal waits<br/>open period or approve: retry"]:::fail
  n_post["Post Journals<br/>GL_BALANCES · STATUS = P"]:::c5
  n_e(["End: the 100 is in the books"]):::term
  n_s --> n_hire
  n_hire --> n_user
  n_user --> n_enter
  n_enter --> n_submit
  n_submit --> n_mgr
  n_mgr -- Yes --> n_aud
  n_aud -- Yes --> n_appr
  n_appr --> n_exp
  n_exp --> n_imp
  n_imp -- Yes --> n_inv
  n_inv --> n_val
  n_val --> n_hold
  n_hold -- Yes --> n_acc1
  n_acc1 --> n_pay
  n_pay --> n_acc2
  n_acc2 --> n_fin
  n_fin -- Yes --> n_xfer
  n_xfer --> n_impj
  n_impj --> n_per
  n_per -- Yes --> n_post
  n_post --> n_e
  n_mgr -. No .-> n_mgr_b
  n_mgr_b -.-> n_enter
  n_aud -- Yes --> n_aud_b
  n_aud_b --> n_appr
  n_imp -. No .-> n_imp_b
  n_imp_b -.-> n_exp
  n_hold -. No .-> n_hold_b
  n_hold_b -.-> n_val
  n_fin -. No .-> n_fin_b
  n_fin_b -.-> n_acc2
  n_per -. No .-> n_per_b
  n_per_b -.-> n_impj

  classDef term fill:#e9edf2,stroke:#33414f,stroke-width:1.5px,color:#121c27
  classDef fail fill:#fdecea,stroke:#c0392b,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  classDef c1 fill:#e5f1f6,stroke:#1d6f8e,stroke-width:1.6px,color:#121c27
  classDef d1 fill:#ffffff,stroke:#1d6f8e,stroke-width:1.6px,color:#121c27
  classDef a1 fill:#e5f1f6,stroke:#1d6f8e,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  classDef c2 fill:#e3f3ea,stroke:#1f7a52,stroke-width:1.6px,color:#121c27
  classDef d2 fill:#ffffff,stroke:#1f7a52,stroke-width:1.6px,color:#121c27
  classDef a2 fill:#e3f3ea,stroke:#1f7a52,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  classDef c3 fill:#fbf0dc,stroke:#a8660b,stroke-width:1.6px,color:#121c27
  classDef d3 fill:#ffffff,stroke:#a8660b,stroke-width:1.6px,color:#121c27
  classDef a3 fill:#fbf0dc,stroke:#a8660b,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  classDef c4 fill:#eee9fa,stroke:#6a4cc0,stroke-width:1.6px,color:#121c27
  classDef d4 fill:#ffffff,stroke:#6a4cc0,stroke-width:1.6px,color:#121c27
  classDef a4 fill:#eee9fa,stroke:#6a4cc0,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  classDef c5 fill:#fbe9ed,stroke:#b23a52,stroke-width:1.6px,color:#121c27
  classDef d5 fill:#ffffff,stroke:#b23a52,stroke-width:1.6px,color:#121c27
  classDef a5 fill:#fbe9ed,stroke:#b23a52,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
  linkStyle 21,22,25,26,27,28,29,30,31,32 stroke:#c0392b,stroke-width:1.6px
```

## The steps in plain English

| # | What happens | Fusion job or screen | Main tables | Done when |
|---|---|---|---|---|
| 1 | HR hires the person | Hire Employee | `PER_ALL_PEOPLE_F`, `PER_ALL_ASSIGNMENTS_M` | Assignment is `ACTIVE` |
| 2 | A login and roles are created | Autoprovisioning | `PER_USERS`, `PER_USER_ROLES` | `ACTIVE_FLAG = Y` |
| 3 | The employee enters the expense report | Expenses | `EXM_EXPENSE_REPORTS`, `EXM_EXPENSES`, `EXM_EXPENSE_DISTS` | Status `SAVED` |
| 4 | The employee submits it | Expenses | `EXM_EXPENSE_REPORTS` | Status `PENDING APPROVAL` |
| 5 | The manager approves (and an auditor, if needed) | Approval workflow | `EXM_EXPENSE_REPORTS` | Status `APPROVED` |
| 6 | The approved report is sent to Payables | Export Expense Reports | `AP_INVOICES_INTERFACE`, `AP_INVOICE_LINES_INTERFACE` | Rows land in the interface tables |
| 7 | Payables turns those rows into a real invoice | Import Payables Invoices | `AP_INVOICES_ALL`, `AP_INVOICE_LINES_ALL`, `AP_INVOICE_DISTRIBUTIONS_ALL` | Interface status `PROCESSED` |
| 8 | The invoice is checked | Validate Payables Invoice | `AP_INVOICES_ALL`, `AP_HOLDS_ALL` | `MATCH_STATUS_FLAG = A`, no holds |
| 9 | The invoice becomes an accounting entry | Create Accounting | `XLA_EVENTS`, `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 10 | The employee is paid | Payment Process Request | `IBY_PAYMENTS_ALL`, `AP_CHECKS_ALL` | Payment status `ISSUED` |
| 11 | The payment becomes an accounting entry | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 12 | Both entries are sent to the ledger | Transfer Journal Entries to GL | `GL_INTERFACE` | Transfer status `Y` |
| 13 | Journals are created from them | Import Journals | `GL_JE_BATCHES`, `GL_JE_HEADERS`, `GL_JE_LINES` | Batch status `U` (unposted) |
| 14 | Journals are posted and balances update | Post Journals | `GL_BALANCES` | Batch status `P` (posted) |

## The accounting entries

Create Accounting runs twice, once for the invoice and once for the payment.

| Event | Debit | Credit |
|---|---|---|
| Invoice | Expense 100 | Payables liability 100 |
| Payment | Payables liability 100 | Cash 100 |

After both, the liability is back to zero. Only the expense and the cash movement stay in the books.

## Where it usually gets stuck

- **Report never reaches Payables:** it is not `APPROVED` yet, or the employee is not set up as a supplier.
- **Invoice cannot be paid:** `MATCH_STATUS_FLAG` is not `A`, or there is an open hold in `AP_HOLDS_ALL`.
- **Nothing in the ledger:** the accounting event is in error, or the transfer status is still `N`. Look for an invalid account or a closed period.
- **Journal not posted:** the period is closed, or the journal is waiting for approval.

## Accuracy note

Job names, table names, column names and status codes are written from working knowledge of Oracle Fusion. They have not been checked against Oracle's documentation or a real environment. The export job name in step 6 is the one I am least sure about. Check names before relying on them.

[Back to all flows](../README.md)
