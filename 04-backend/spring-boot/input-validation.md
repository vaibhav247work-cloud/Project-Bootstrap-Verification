# Input Validation Standard

1. **MUST** validate every request DTO at the API boundary using Jakarta Validation and custom validators for cross-field rules.
2. **MUST** allow-list enum/status/sort/filter values and validate string length, number ranges, date ranges, content type, and collection size.
3. **MUST** reject unknown/illegal fields where the API contract requires strict input, and enforce request/body/file-size limits.
4. **MUST** validate business rules again in the application layer; DTO validation alone cannot verify availability, ownership, tenant membership, or state transitions.
5. **MUST NOT** trust IDs, price, role, permission, tenant, status, calculated total, file type, or external URL supplied by the client.
6. **MUST** return safe field error codes through the global error format and never echo passwords/tokens/secrets.
7. **MUST** use parameterized queries/ORM bindings and safe serializers/parsers. XML parsing, if enabled, requires XXE-safe configuration.
8. **SHOULD** audit/rate-limit repeated invalid security-sensitive input.
