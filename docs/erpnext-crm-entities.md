# ERPNext CRM Entity Model

## Quick Reference

| Entity | Represents | Stage | Key Purpose |
|--------|-----------|-------|-------------|
| **Lead** | Person | Top of funnel | Individual who showed interest |
| **Contact** | Person | N/A (reference) | Reusable person details, links to multiple entities |
| **Prospect** | Company | Mid-funnel | Organization being pursued, not yet a customer |
| **Opportunity** | Deal | Active sales | Specific potential transaction with amount/stage |
| **Customer** | Company/Individual | Post-sale | Paying entity for transactions |

## Conversion Flow

```
Lead (person) 
    ↓ Convert
    ├─→ Contact (person details)
    ├─→ Prospect (company, if exists)
    └─→ Opportunity (the deal)
            ↓ Won
        Customer → Quotation → Sales Order → Invoice
```

## Key Rules

1. **Lead ≠ Prospect** — Leads are people, Prospects are companies
2. **Contact is shared** — One Contact links to many Customers/Prospects  
3. **Opportunity sources vary** — Can originate from Lead, Prospect, OR existing Customer
4. **Prospect is temporary** — Becomes Customer when deal closes
