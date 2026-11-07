# Borrowing Power Calculator

A command line borrowing power calculator in JavaScript (Node.js). It estimates
the maximum a person could borrow for a 30 year home loan, using tax and
household expense (HEM) values fetched from a local API.

## How it works

1. Enter gross annual income, number of dependents, declared monthly expenses
   and total credit card limits.
2. The calculator fetches annual tax and a HEM baseline from the API.
3. Net monthly income is `(income - tax) / 12`.
4. Living expenses are the higher of declared expenses and the HEM value.
5. Credit card liability is assumed to be 3% of total limits per month.
6. Monthly repayment capacity is net income minus living expenses minus credit
   card liability. If this is zero or negative, borrowing power is $0.
7. Maximum loan is calculated with the amortisation formula over 360 months at
   an assessment rate of 10% (7% interest plus a 3% buffer):
   `P = M * (1 - (1 + R)^-N) / R`
8. The result shows maximum borrowing power and monthly repayment, followed by
   a simple breakdown of principal vs interest over 30 years.

## Design decisions

- **`BorrowingCalculator` class:** groups the calculation logic and API access,
  so it is easy to extend and to test with a mocked `fetchJson`.
- **Shared `fetchJson` helper:** one place for the authorisation header, JSON
  parsing and error handling. It separates network failures ("Could not reach
  the calculation server") from API errors (uses the server's message, or falls
  back to the status code if the body is not JSON).
- **Concurrent API calls:** tax and HEM are independent, so
  `calculateBorrowingPower` fetches both with `Promise.all`.
- **Configurable settings:** the `API_TOKEN` and `API_BASE_URL` environment
  variables override the defaults (a development token and
  `http://localhost:3000`), so it runs out of the box.
- **Input validation:** `parsePositiveNumber` rejects negative and non numeric
  input. Dependents are rounded down to a whole number.

## Bug found and fixed

Input such as `$100,000` was silently truncated, because
`parseFloat("100,000")` returns `100`, giving wrong results with no error.
`parsePositiveNumber` now strips non numeric characters before parsing, with
tests for commas, dollar signs and decimals.

## Setup

Requires Node.js 18 or later (uses the built in `fetch`).

    npm install

## Run

Start the development API in its own terminal and leave it running:

    npm run api

In a second terminal, run the calculator:

    npm start

Example input: income 120000, dependents 2, expenses 1400, credit limits 20000.

## Test

    npm test
    npm run coverage

The 13 tests in `test_calculator.js` cover:

- `fetchJson`: success, API error with a message, API error with no JSON body,
  and network failure (using a mocked global `fetch`).
- `calculateBorrowingPower`: a standard case, the zero repayment case, and
  error propagation from the API layer.
- `parsePositiveNumber`: plain numbers, commas and dollar signs, empty input,
  non numeric text, negatives and decimals.

Coverage is 100% for statements, branches, functions and lines.
`runConsoleMode` and the command line entry point are excluded with
`/* c8 ignore */` because they are interactive input and output, which were
tested manually.

## Project structure

    borrowingCalculator.js   calculator class, input parsing, console mode
    test_calculator.js       test suite
    server.js                local development API
    server.md                API documentation
    package.json             scripts and dependencies
    .gitignore               files excluded from version control

## Notes

- The calculator is simplified, so results will not match a real bank
  calculator exactly.
- Tax and HEM values come from mock formulas in `server.js`.
- The development token is for local use only. Set your own `API_TOKEN` if you
  change the server.
