# CI/CD Automated Testing Pipeline

A CI/CD pipeline integrating **GitHub, CircleCI, Vercel, and Testim** to automate preview deployment and end-to-end testing.

## Architecture

```text
GitHub Push
    │
    ▼
 CircleCI
    │
    ├── Checkout
    ├── Build
    ├── Deploy Preview ──────► Vercel
    │                            │
    │                            ▼
    ├── Chrome + ChromeDriver   Preview URL
    │                            │
    ├────────────────────────────┘
    │
    ▼
 Testim E2E Tests
    │
    ▼
 PASS / FAIL
    │
    ▼
 Vercel Cleanup
```

## Tech Stack

* **CI/CD:** CircleCI
* **Source Control:** GitHub
* **Deployment:** Vercel
* **E2E Automation:** Testim
* **Browser Automation:** Selenium / ChromeDriver
* **Runtime:** Node.js
* **Configuration:** CircleCI YAML

## Pipeline

The pipeline is triggered automatically on every GitHub push.

1. Checkout source code
2. Install Vercel CLI
3. Build and deploy a temporary Vercel preview
4. Generate a Testim-accessible preview URL using Vercel Protection Bypass
5. Install Chrome and ChromeDriver
6. Execute Testim tests against the preview environment
7. Remove the temporary Vercel deployment

The cleanup step runs with `when: always`, ensuring temporary deployments are removed even when tests fail.

## Configuration

Pipeline configuration:

```text
.circleci/config.yml
```

Required CircleCI environment variables:

```text
VERCEL_TOKEN
TOKEN_BYPASS_TESTIM
TESTIM_TOKEN
TESTIM_PROJECT_ID
```

Vercel project configuration:

```yaml
environment:
  VERCEL_ORG_ID: <org-id>
  VERCEL_PROJECT_ID: <project-id>
```

Secrets are managed through CircleCI Environment Variables rather than committed to source control.

## Key Implementation

Testim is executed through the CLI using the locally installed Chrome/ChromeDriver:

```bash
npx --yes @testim/testim-cli \
  --token "$TESTIM_TOKEN" \
  --project "$TESTIM_PROJECT_ID" \
  --base-url "$FINAL_URL" \
  --use-local-chrome-driver \
  --mode selenium \
  --headless
```

This approach provides an isolated preview environment for each pipeline run and validates the application using the same deployment artifact before cleanup.

## Project Structure

```text
.
├── app/
│   └── ...
└── .circleci/
    └── config.yml
```

## Outcome

The pipeline provides an automated feedback loop:

**Code → Build → Preview Deployment → E2E Testing → Result → Cleanup**

This eliminates manual deployment and test execution for each change and provides repeatable CI validation.
