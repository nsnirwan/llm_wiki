# Domain Glossary — {{PROJECT_NAME}}

<!--
  The single highest-value file for domain-heavy systems (ERP, CRM, fintech, health).
  Its real job is DISAMBIGUATION — when a word means something different in the
  business than it does in the database, say so loudly. That specific confusion is
  where AI causes the most damage.
-->

| Term | Means | Does NOT mean | Where it lives in code |
|---|---|---|---|
| Order | a business purchase order placed by a customer | the `order` DB table, which stores line items only | `services/order/` |
| Customer | the buyer | — | |
| Vendor | the supplier | not a "customer with a flag" — separate entity | |

## Ambiguous terms — always qualify these

Words that appear in this codebase with more than one meaning. Never use them bare.

| Word | Meaning A | Meaning B | Say instead |
|---|---|---|---|
| | | | |

## Abbreviations

| Short | Full | Notes |
|---|---|---|
| | | |
