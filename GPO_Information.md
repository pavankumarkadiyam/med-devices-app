# What Is a GPO?

A **Group Purchasing Organization (GPO)** negotiates bulk pricing on behalf of thousands of hospitals. Hospitals join GPOs to receive pre-negotiated discounts on medical devices, supplies, and equipment.

## Major GPOs

| GPO | Members | Annual Purchasing | Focus |
|------|----------|-------------------|-------|
| Vizient | 5,000+ hospitals | ~$130B | Largest, academic + community |
| Premier | 4,100+ hospitals | ~$80B | Community hospitals, value-based care |
| HealthTrust | 1,600+ (HCA + affiliates) | ~$40B | For-profit, HCA-owned |
| Intalere | 12,000+ sites | ~$20B | Non-acute, ASCs, rural |

---

# Why This Field Exists in Your Flow

Look at **Phase 3** of your business model:

```text
SALESFORCE FLOW - Check GPO membership
         ↓
    Belongs to GPO?
    Yes → SALESFORCE APEX - Apply GPO discount
    No  → SALESFORCE APEX - Evaluate Quote margin
```

The **GPO affiliation** is a hard fork in your pricing logic.

---

# What Happens Based on This Field

| Scenario | System Behavior |
|----------|-----------------|
| Hospital is Vizient member | Must use Vizient contract price (e.g., 25% off list). Rep cannot quote higher. |
| Hospital is Premier member | Must use Premier contract price (e.g., 20% off list). |
| Hospital has no GPO | Rep negotiates directly. Margin evaluation applies. |
| Rep ignores GPO and quotes list price | Compliance failure → **ISSUE ALERT** fires. Hospital will reject the quote. |

---

# Real-World Consequences

| If You Get It Wrong | What Happens |
|----------------------|--------------|
| Quote above GPO contract price | Hospital rejects quote, buys from competitor, you lose the deal. |
| Quote below GPO contract price | GPO fines you, blacklists you from future contracts, legal risk. |
| Don't know GPO status | Rep guesses, creates bad data, margin calculations are wrong. |

---

# How the Field Is Used Across Phases

| Phase | Field Usage |
|--------|-------------|
| **Phase 1 (Lead)** | Captured if known; blank is acceptable. |
| **Phase 2 (Qualification)** | Sales Ops verifies during account setup. |
| **Phase 3 (Quote)** | **Required.** Triggers GPO discount lookup. Determines whether **Apply GPO Discount** or **Evaluate Quote Margin** runs. |
| **Phase 4 (Approval)** | Sub-GPO-floor deals trigger escalation. |
| **Phase 5 (Fulfillment)** | GPO contract number may print on the PO for ERP. |

---

# Where the Data Comes From

| Source | Method | Accuracy |
|---------|--------|----------|
| GPO member rosters (Vizient, Premier publish these) | Bulk import quarterly | High |
| Sales rep research | Manual entry during qualification | Medium |
| NPI API + third-party data (e.g., Definitive Healthcare) | Automated enrichment | High |
| Hospital tells you directly | Self-reported on form | Medium (they may be wrong) |