# Borrowing Power Calculator

A command line borrowing power calculator in JavaScript (Node.js). It estimates
the maximum a person could borrow for a 30 year home loan, using tax and
household expense (HEM) values fetched from a local API.

This began as a starter project from a Ferocia junior engineering code
exercise, where the original author had left it unfinished. I completed it by
replacing placeholder functions with API calls, restructuring the code into a
class, and adding a test suite with full coverage.

## How it works

1. You enter gross annual income, number of dependents, declared monthly
   expenses and total credit card limits.
2. The calculator fetches annual tax and the HEM baseline from the API.
3. It works out net monthly income: `(income - tax) / 12`.
4. Living expenses are the higher of your declared expenses and the HEM value.
5. Credit card liability is assumed to be 3% of total limits per month.
6. Monthly repayment capacity is net income minus living expenses minus credit
   card liability. If this is zero or negative, borrowing power is $0.
7. Maximum loan is calculated with the standard amortisation formula over 360
   months, at an assessment rate of 10% (7% interest plus a 3% buffer):
   `P = M * (1 - (1 + R)^-N) / R`
8. The result prints maximum borrowing power and monthly repayment, followed by
   a simple breakdown of principal vs interest over 30 years.

## Design decisions

- **`BorrowingCalculator` class:** groups the calculation logic and API access,
  so it is easy to extend and to test with a mocked `fetchJson`.
- **Shared `fetchJson` helper:** one place for the authorisation header, JSON
  parsing and error handling. It distinguishes network failures ("Could not
  reach the calculation server") from API errors (uses the server's message,
  or falls back to the status code if the body is not JSON).
- **Concurrent API calls:** tax and HEM do not depend on each other, so
  `calculateBorrowingPower` fetches both with `Promise.all`.
- **Configurable settings:** `API_TOKEN` and `API_BASE_URL` environment
  variables override the defaults (the development token and
  `http://localhost:3000`), so it runs out of the box.
- **Input validation:** `parsePositiveNumber` rejects negative and non
  numeric input. Dependents are rounded down to a whole number.

## Bug found and fixed

Input such as `$100,000` was silently truncated, because
`parseFloat("100,000")` returns `100`, giving wrong results with no error.
`parsePositiveNumber` now strips non numeric characters before parsing, with
tests for commas, dollar signs and decimals.

## Setup

Requires Node.js 18 or later (the code uses the built in `fetch`).

    npm install

## Run

Start the development API in its own terminal and leave it running:

    npm run api

In a second terminal, run the calculator:

    npm start

Try: income 120000, dependents 2, expenses 1400, credit limits 20000.

## Test

    npm test
    npm run coverage

The 13 tests cover:

- `fetchJson`: success, API error with a message, API error with no JSON body,
  and network failure (using a mocked global `fetch`).
- `calculateBorrowingPower`: a standard case, the zero repayment case, and
  error propagation from the API layer.
- `parsePositiveNumber`: plain numbers, commas and dollar signs, empty input,
  non numeric text, negatives and decimals.

Coverage is 100% for statements, branches, functions and lines.
`runConsoleMode` and the command line entry point are excluded with
`/* c8 ignore */` because they are interactive input and output, which I
tested manually.

## Project structure

    borrowingCalculator.js   calculator class, input parsing, console mode
    borrowingCalculator.test.js   test suite
    server.js                local development API (provided with the exercise)
    server.md                API documentation (provided with the exercise)

## Notes

- The calculator is simplified, so results will not match a real bank
  calculator exactly.
- The tax and HEM values come from mock formulas in the provided `server.js`.
- The development token is for local use only. Set your own `API_TOKEN` if you
  change the server.
