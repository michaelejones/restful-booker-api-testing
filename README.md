# Restful Booker — API Test Suite

![API Tests](https://github.com/<your-username>/restful-booker-api-testing/actions/workflows/newman.yml/badge.svg)

An automated API test suite for the [Restful Booker](https://restful-booker.herokuapp.com/) API, built with Postman and run in CI with Newman. The suite exercises a complete CRUD lifecycle with token-based authentication, data-driven request chaining, and both positive and negative-path assertions.

**27 assertions across 11 requests**, all passing.

---

## Tech Stack

- **Postman** — collection authoring and test scripting (`pm.test`, Chai assertions)
- **Newman** — command-line collection runner for CI execution
- **newman-reporter-htmlextra** — rich HTML test reporting
- **GitHub Actions** — continuous integration on every push and pull request

---

## What This Demonstrates

- **Full CRUD lifecycle** — Create, Read, Update (both `PUT` and `PATCH`), and Delete against a live API
- **Token-based authentication** — retrieves an auth token and supplies it as a cookie on protected write operations
- **Stateful, data-driven chaining** — captures the booking ID and booking fields from the create response and feeds them into every downstream request and assertion, so the suite is not dependent on hardcoded values
- **Positive and negative-path testing** — verifies successful operations *and* confirms a deleted resource returns `404`
- **Persistence verification** — after each update, a separate `GET` confirms the change was actually saved server-side and that untouched fields were not clobbered (the most common `PUT` failure mode)
- **Environment-based configuration** — the base URL is externalized to a Postman environment, enabling the same suite to run against test vs. production by swapping a single flag

---

## Test Coverage

| # | Method | Endpoint | Purpose |
|---|--------|----------|---------|
| 1 | GET | `/ping` | Health check (expects `201`) |
| 2 | POST | `/auth` | Retrieve auth token, store for reuse |
| 3 | POST | `/booking` | Create a booking; capture `bookingid` and fields |
| 4 | GET | `/booking` | Confirm new booking appears in the full list |
| 5 | GET | `/booking/{id}` | Confirm created values match what was sent |
| 6 | PUT | `/booking/{id}` | Full-object update (set `depositpaid` to `true`) |
| 7 | GET | `/booking/{id}` | Verify the update persisted |
| 8 | PATCH | `/booking/{id}` | Partial update (change `firstname`) |
| 9 | GET | `/booking/{id}` | Verify the partial update persisted |
| 10 | DELETE | `/booking/{id}` | Delete the booking (expects `201`) |
| 11 | GET | `/booking/{id}` | Confirm deletion returns `404` |

---

## Repository Structure

```
restful-booker-api-testing/
├── collections/
│   └── Restful_Booker.postman_collection.json
├── environments/
│   └── Production.postman_environment.json
├── .github/
│   └── workflows/
│       └── newman.yml
├── .gitignore
└── README.md
```

---

## Running the Tests Locally

### Prerequisites

- [Node.js](https://nodejs.org/) (v18 or later)
- Newman and the htmlextra reporter, installed globally:

```bash
npm install -g newman newman-reporter-htmlextra
```

### Run the suite

```bash
newman run collections/Restful_Booker.postman_collection.json \
  -e environments/Production.postman_environment.json \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman/report.html
```

The `-e` flag supplies the environment file that defines the `url` base URL. After the run, open `newman/report.html` in a browser for the full report.

> **Note:** The base URL lives in the environment file rather than the collection. This mirrors a real promotion workflow — the same collection can target a test, staging, or production environment simply by swapping the `-e` file, with no changes to the requests themselves.

---

## Continuous Integration

Every push and pull request to `main` triggers the [`newman.yml`](.github/workflows/newman.yml) workflow, which:

1. Checks out the repository
2. Installs Node.js, Newman, and the htmlextra reporter
3. Runs the full collection against the environment
4. Publishes the HTML report as a downloadable build artifact (on both pass and fail)

The workflow can also be triggered manually from the **Actions** tab.

---

## About the Target API

[Restful Booker](https://restful-booker.herokuapp.com/) is a free, public API built specifically for practicing API testing. It is a shared sandbox that periodically resets its data, so occasional environmental failures on a run are expected and are not defects in this suite.
