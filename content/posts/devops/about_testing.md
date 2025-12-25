---
title: About Testing
date: 2025-12-25
summary: 📝 Quick notes on Testing...
tags:
  - devops
---
## Structured Testing
- Impact Analysis (Pre-Development)
- Before implementation, categorize the scope of change to determine the testing depth:
	- Domain / Logic Layer: Business rules or calculations
	- Data Persistence Layer: Schemas, queries or indexing
	- Transport / Interface Layer: API contracts, routing, or middleware
	- Cross-Cutting Concerns: Performance, security or concurrency
### Test Categorization & Strategy
- Unit Testing (Isolated Logic)
	- Target: Individual functions / methods in the Service or Domain layer.
	- Success Scenarios: Validate that valid inputs yield expected outputs.
	- Boundary Value Analysis (BVA): Test minimum, maximum, and just-outside-the-range values.
	- Error handling: Ensure the system throws specific exceptions/errors for invalid inputs.
	- Isolation: Use Mocks / Stubs for external dependencies.
## CI/CD
### Definition
- CI: Continuous Integration
    - Every time you push code → tests run → code builds → ensures nothing breaks.
- CD: Continuous Deployment / Delivery
    - Your code automatically deploys to a server or hosting platform after CI succeeds.
### Workflow
GitHub Actions:
1. Push code to GitHub
2. GitHub Actions starts running (CI)
    - Install dependencies
    - Build the project
    - Run tests
3. If CI succeeds → Deployment job runs (CD)
4. Your app is updated automatically