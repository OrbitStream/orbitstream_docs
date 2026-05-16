# OrbitStream Docs

## Project
Documentation for OrbitStream — a Stellar-based token streaming payroll platform.

## Structure
architecture/
  overview.md         - full system architecture
  contract.md         - contract design decisions
  backend.md          - backend service design
  frontend.md         - frontend design

contract/
  spec.md             - every function, param, return value documented
  math.md             - claimable formula with examples
  events.md           - all contract events

api/
  openapi.yaml        - full REST API spec

guides/
  employer-guide.md   - how to create and manage streams
  employee-guide.md   - how to claim salary

security/
  threat-model.md     - known attack vectors + mitigations

## Rules
- Keep spec.md in sync with actual contract at all times
- Keep openapi.yaml in sync with actual API at all times
- Every contract function must have an example in spec.md
- Docs are written for a technical audience
