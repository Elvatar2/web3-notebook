# Legal Contracts and NDAs in Professional Auditing

## Simple Definition
Legal contracts and Non-Disclosure Agreements (NDAs) are formal agreements that define the scope, responsibilities, payment terms, and confidentiality requirements of an audit engagement. They protect both the auditor and the client.

## The Best Analogy
Think of an audit contract like a **construction contract for building a house**. Before any work begins, both parties sign a document that specifies: What will be built (scope), how much it costs (payment), when it will be done (timeline), what happens if something goes wrong (liability), and who owns the blueprints (IP rights). Without this contract, disputes are inevitable.

## Key Components of an Audit Contract

```text
1. Scope of Work:
   - Which contracts will be audited (specific file paths and commit hashes)
   - What is excluded from the audit (e.g., frontend, off-chain components)
   - Number of contracts and approximate lines of code

2. Timeline:
   - Start date and end date
   - Milestones (e.g., initial report delivery, remediation review)
   - Payment schedule (e.g., 50% upfront, 50% on delivery)

3. Payment Terms:
   - Total fee (e.g., $30,000)
   - Currency (USD, USDC, ETH)
   - Payment method and schedule
   - Late payment penalties

4. Deliverables:
   - Initial audit report (PDF or Markdown)
   - Remediation review (if included)
   - Final report with all findings marked as Fixed/Acknowledged

5. Confidentiality (NDA):
   - What information is confidential (code, business logic, tokenomics)
   - How long confidentiality lasts (e.g., 2 years, perpetual)
   - Exceptions (e.g., public audit reports after disclosure)

6. Liability and Warranties:
   - Auditor's liability cap (e.g., limited to the audit fee)
   - Disclaimer: "Audit does not guarantee 100% security"
   - Client's responsibility to implement fixes

7. Intellectual Property:
   - Who owns the audit report (usually the client)
   - Can the auditor publish the report publicly (usually yes, after a delay)
   - Can the auditor use the findings in marketing (usually yes, with permission)

8. Termination:
   - Conditions for early termination
   - Payment for work completed if terminated early
```

## Example: Simple Audit Agreement (Key Clauses)

```markdown
# SMART CONTRACT AUDIT AGREEMENT

This Agreement is entered into as of [Date] by and between:

**Client:** [Protocol Name], a [Legal Entity Type] organized under the laws of [Jurisdiction]

**Auditor:** [Your Name/Company], a [Legal Entity Type] organized under the laws of [Jurisdiction]

## 1. Scope of Work
Auditor will perform a security audit of the following smart contracts:
- `contracts/Vault.sol` (commit: `abc123`)
- `contracts/Staking.sol` (commit: `abc123`)
- `contracts/Rewards.sol` (commit: `abc123`)

Total: ~1,500 lines of Solidity code.

Exclusions: Frontend code, off-chain components, and third-party libraries (e.g., OpenZeppelin) are not in scope.

## 2. Timeline
- Audit start date: [Date]
- Initial report delivery: [Date + 2 weeks]
- Remediation review: [Date + 3 weeks]
- Final report: [Date + 4 weeks]

## 3. Payment
- Total fee: $30,000 USD
- Payment schedule:
  - 50% ($15,000) due upon signing
  - 50% ($15,000) due upon delivery of final report
- Payment method: USDC to [Wallet Address] or wire transfer to [Bank Details]

## 4. Deliverables
- Initial audit report with all findings, severity ratings, and PoCs
- Remediation review of fixes implemented by Client
- Final audit report with all findings marked as Fixed/Acknowledged/Rejected

## 5. Confidentiality
Both parties agree to keep all non-public information confidential for a period of 2 years from the date of this Agreement. This includes:
- Source code and business logic
- Audit findings and reports (until publicly disclosed)
- Tokenomics and launch plans

Exception: Auditor may publish the final audit report after Client's public launch or after 90 days, whichever comes first.

## 6. Liability
Auditor's total liability under this Agreement shall not exceed the total audit fee ($30,000). Auditor does not guarantee that the audited contracts are 100% secure or free from all vulnerabilities.

## 7. Intellectual Property
Client owns all rights to the audit report. Auditor retains the right to:
- Publish the final audit report after public disclosure
- Use the engagement in marketing materials (with Client's permission)
- Reuse general methodologies and tools developed during the audit

## 8. Termination
Either party may terminate this Agreement with 7 days written notice. In case of termination:
- Client will pay for work completed up to the termination date
- Auditor will deliver any work-in-progress

---

**Signatures:**

Client: _________________________ Date: _________
Name: [Name]
Title: [Title]

Auditor: _________________________ Date: _________
Name: [Your Name]
Title: [Your Title]
```

## NDA Best Practices

```text
1. Always Sign an NDA Before Receiving Code:
   - Never review code without a signed NDA.
   - Protects you from accusations of stealing ideas.

2. Define "Confidential Information" Clearly:
   - Source code, business logic, tokenomics, launch dates.
   - Exclude publicly available information.

3. Set a Reasonable Duration:
   - 1-2 years is standard for most audits.
   - Perpetual NDAs are uncommon and hard to enforce.

4. Allow Public Disclosure After Launch:
   - Most protocols want audit reports published after launch.
   - Include a clause allowing this after a specific date.

5. Mutual NDA:
   - Both parties should be bound by confidentiality.
   - Protects your audit methodologies and tools as well.
```

## Key Takeaways
- **Never Work Without a Contract:** Verbal agreements lead to disputes.
- **Scope is Critical:** Clearly define what is and isn't included in the audit.
- **Payment Terms:** Get 50% upfront to protect yourself from non-payment.
- **Liability Cap:** Limit your liability to the audit fee. You can't guarantee 100% security.
- **NDA Before Code:** Always sign an NDA before receiving any code.
- **Public Disclosure:** Negotiate the right to publish the audit report after launch.
- **Legal Review:** For large engagements ($50k+), have a lawyer review the contract.
