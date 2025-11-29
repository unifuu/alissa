---
title: "About CI/CD"
date: "2025-11-29"
summary: "📝 Quick notes on CI/CD..."
tags: ["devops", "ci/cd"]
---

## Definition

- CI: Continuous Integration
    - Every time you push code → tests run → code builds → ensures nothing breaks.
- CD: Continuous Deployment / Delivery
    - Your code automatically deploys to a server or hosting platform after CI succeeds.

### Analogy

1. When you finish your homework, CI (teacher) who automatically checks it for mistakes.
2. After the CI (teacher) checks your homework, CD (deliverer) automatically submits it to the school.

## Workflow

GitHub Actions:
1. Push code to GitHub
2. GitHub Actions starts running (CI)
    - Install dependencies
    - Build the project
    - Run tests
3. If CI succeeds → Deployment job runs (CD)
4. Your app is updated automatically