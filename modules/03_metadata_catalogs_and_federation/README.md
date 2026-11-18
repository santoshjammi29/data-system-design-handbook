# Module 03: Open Metadata Catalogs & Multi-Engine Federation


![Unity Catalog Unified Governance & 3-Level Namespace](../../assets/whiteboards/unity_catalog_architecture.webp)
> *Architectural Blueprint: [Unity Catalog Unified Governance & 3-Level Namespace](../../docs/whiteboard_diagrams/README.md#unity-catalog-governance). View full multi-persona breakdown and engine mechanisms in the master index.*



## 1. Iceberg REST Catalog Specification
The Iceberg REST Catalog provides a standard HTTP OpenAPI protocol separating metadata retrieval from underlying cloud storage authorization:
- **Credential Vending**: Clients authenticate against the REST catalog; the catalog returns short-lived scoped cloud storage tokens (STS/SAS).
- **Commit Coordination**: Atomic compare-and-swap (CAS) commits occur over HTTPS, eliminating object storage file locking dependencies.
