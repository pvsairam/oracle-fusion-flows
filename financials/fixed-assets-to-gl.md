# Fixed Assets to GL

**What this flow covers:** the company buys something it will use for years, it is set up as an asset, loses value month by month (depreciation), and every step reaches the books.

**Modules involved:** Payables, Fixed Assets, Subledger Accounting (SLA), General Ledger (GL).

**Example used:** the company buys equipment worth 1,200.

Read the flowchart top to bottom. A diamond is a yes or no question. A "No" follows the dashed red path to a fix, then back to retry the step.

**Colors:** 🟨 Payables  🟩 Fixed Assets  🟪 Depreciation  🟥 General Ledger

```mermaid
flowchart TD
  n_s(["Start: the company buys equipment worth 1,200"]):::term
  n_inv["Invoice line marked as an asset<br/>AP_INVOICE_LINES_ALL"]:::c3
  n_val["Validate and account the invoice<br/>MATCH_STATUS_FLAG = A · ENTRY = F"]:::c3
  n_mass["Create Mass Additions<br/>FA_MASS_ADDITIONS · NEW"]:::c2
  n_dm{"Asset details complete?"}:::d2
  n_dm_b["Mass addition on hold<br/>add category and location"]:::fail
  n_pm["Post Mass Additions<br/>FA_ADDITIONS_B · POSTED"]:::c2
  n_aca["Create Accounting: addition<br/>Dr Asset cost / Cr Asset clearing"]:::c2
  n_dep["Calculate Depreciation<br/>FA_DEPRN_SUMMARY · FA_DEPRN_DETAIL"]:::c4
  n_acd["Create Accounting: depreciation<br/>Dr Depreciation / Cr Accumulated depr."]:::c4
  n_fin{"Entries final?<br/>ENTRY = F"}:::d4
  n_fin_b["Event in error<br/>fix account or period, rerun"]:::fail
  n_xfer["Transfer Journal Entries to GL<br/>GL_INTERFACE"]:::c5
  n_impj["Import Journals<br/>GL_JE_BATCHES · STATUS = U"]:::c5
  n_post["Post Journals<br/>GL_BALANCES · STATUS = P"]:::c5
  n_e(["End: asset and depreciation are in the books"]):::term
  n_s --> n_inv
  n_inv --> n_val
  n_val --> n_mass
  n_mass --> n_dm
  n_dm -- Yes --> n_pm
  n_pm --> n_aca
  n_aca --> n_dep
  n_dep --> n_acd
  n_acd --> n_fin
  n_fin -- Yes --> n_xfer
  n_xfer --> n_impj
  n_impj --> n_post
  n_post --> n_e
  n_dm -. No .-> n_dm_b
  n_dm_b -.-> n_mass
  n_fin -. No .-> n_fin_b
  n_fin_b -.-> n_acd

  classDef term fill:#e9edf2,stroke:#33414f,stroke-width:1.5px,color:#121c27
  classDef fail fill:#fdecea,stroke:#c0392b,stroke-width:1.5px,stroke-dasharray:5 4,color:#121c27
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
  linkStyle 13,14,15,16 stroke:#c0392b,stroke-width:1.6px
```

## The steps in plain English

| # | What happens | Fusion job or screen | Main tables | Done when |
|---|---|---|---|---|
| 1 | A supplier invoice line is marked as an asset | Create Invoice | `AP_INVOICE_LINES_ALL` | Line flagged to track as asset |
| 2 | The invoice is validated and accounted | Validate Payables Invoice, Create Accounting | `AP_INVOICES_ALL`, `XLA_AE_LINES` | `MATCH_STATUS_FLAG = A`, entry `F` |
| 3 | Fixed Assets picks up the invoice line | Create Mass Additions | `FA_MASS_ADDITIONS` | Posting status `NEW` |
| 4 | Someone reviews and completes the asset details | Prepare Mass Additions | `FA_MASS_ADDITIONS` | Posting status `POST` (ready) |
| 5 | The asset is created | Post Mass Additions | `FA_ADDITIONS_B`, `FA_BOOKS` | Posting status `POSTED` |
| 6 | The addition is accounted | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 7 | Depreciation is calculated for the month | Calculate Depreciation | `FA_DEPRN_SUMMARY`, `FA_DEPRN_DETAIL` | Depreciation run completed |
| 8 | Depreciation is accounted | Create Accounting | `XLA_AE_HEADERS`, `XLA_AE_LINES` | Entry status `F` (final) |
| 9 | Entries go to the ledger | Transfer Journal Entries to GL | `GL_INTERFACE` | Transfer status `Y` |
| 10 | Journals are created and posted | Import Journals, Post Journals | `GL_JE_BATCHES`, `GL_JE_LINES`, `GL_BALANCES` | Batch status `P` (posted) |

## The accounting entries

| Event | Debit | Credit |
|---|---|---|
| Invoice | Asset clearing 1,200 | Payables liability 1,200 |
| Asset addition | Asset cost 1,200 | Asset clearing 1,200 |
| Monthly depreciation | Depreciation expense (the month's share) | Accumulated depreciation (the month's share) |

The clearing account is a temporary holding account. The invoice fills it and the asset addition empties it. Depreciation repeats every month for the life of the asset.

## Where it usually gets stuck

- **Invoice never becomes an asset:** the line was not marked to track as an asset, or the invoice is not accounted yet.
- **Mass addition stuck on hold:** the category, location or other required details are missing.
- **No depreciation:** the asset is not in a depreciating state, or the asset period is closed.
- **Nothing in the ledger:** an accounting event is in error, or the transfer status is still `N`.

## Accuracy note

Job names, table names, column names and status codes are written from working knowledge of Oracle Fusion. They have not been checked against Oracle's documentation or a real environment. How invoice lines reach Fixed Assets (steps 1 to 3) is the part I am least sure about, so verify it first.

[Back to all flows](../README.md)
