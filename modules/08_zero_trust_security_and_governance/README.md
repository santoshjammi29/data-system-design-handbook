# Module 08: Zero-Trust Security, ABAC & Column Masking

## 1. Attribute-Based Access Control (ABAC)
Zero-trust data platforms evaluate access policies dynamically based on user attributes, dataset classifications, and session context:
- **Dynamic Column Masking**: PII (SSN, credit card, email) masked at query time based on user role (`has_role('compliance_officer')`).
- **Row-Level Security**: Filters rows based on user geographical jurisdiction (`user_region = row_region`).
