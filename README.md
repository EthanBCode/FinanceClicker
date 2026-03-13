# Finance Clicker

## Live Site
Finance Clicker is live at [https://financeclicker.net](https://financeclicker.net).

If you want to understand the project quickly, start with the live app: create an account, click to earn, buy upgrades, place long and short paper trades, and watch the leaderboard update in near real time.

---

## Project Purpose
Finance Clicker is a stock-themed incremental game built to showcase a server-authoritative architecture on AWS. The UI is responsive and game-like, but persistent progression is controlled by backend logic rather than browser state.

The project is designed to demonstrate how to combine:
- Cognito-based user authentication
- Protected API routes with API Gateway authorization
- Lambda-based gameplay logic
- DynamoDB-backed canonical state and leaderboard data

This architecture supports competitive progression with stronger integrity than a client-trusted clicker model.

---

## Why This Is Different From a Typical Cookie Clicker
Many clicker games store critical values in local storage or browser memory, which makes score tampering straightforward through DevTools or scripts.

Finance Clicker avoids that trust model:
- The frontend sends action requests such as `click`, `buy`, `trade`, and `state`.
- The backend validates each request against current canonical state.
- Only backend-approved results are persisted and returned.

Local client edits do not become real progression unless backend validation and persistence succeed.

---

## AWS Backend Architecture

### Authentication: Amazon Cognito
Users sign in through Cognito Hosted UI and OAuth. The client includes tokens on API requests, and API Gateway enforces route access through a Cognito authorizer before Lambda execution.

### API Layer: API Gateway
API Gateway exposes the gameplay routes as explicit actions:
- `/click`
- `/buy`
- `/trade`
- `/state`
- `/leaderboard`

Each route maps to focused backend logic, which helps keep gameplay behavior auditable and easier to evolve.

### Compute Layer: AWS Lambda
Route handlers enforce game rules and return updated canonical state:

- **`/click` Lambda**
  - Applies click earnings.
  - Settles passive progression from elapsed time.
  - Returns updated player state.

- **`/buy` Lambda**
  - Validates affordability.
  - Applies upgrade cost scaling.
  - Updates ownership and derived production rates.

- **`/trade` Lambda**
  - Executes long and short trade actions.
  - Enforces balance and position constraints.
  - Commits valid portfolio and balance updates.

- **`/state` Lambda**
  - Returns canonical player state for client rehydration.
  - Keeps refresh/reconnect flows aligned with backend truth.

- **`/leaderboard` Lambda**
  - Reads and returns ranking-oriented player data.
  - Supports live leaderboard refresh cycles in the client.

### Persistence Layer: DynamoDB
DynamoDB stores durable player progression and leaderboard fields. Lambda reads the current record, applies deterministic rules, and writes the next canonical state.

This enables:
- Cross-session persistence
- Identity-tied progression
- Leaderboard ranking from backend-managed values

### Observability: CloudWatch
CloudWatch provides request and function visibility for debugging, balancing, and runtime monitoring of API and Lambda behavior.

---

## End-to-End Flow
1. User signs in through Cognito.
2. Browser calls a protected API Gateway route with a valid token.
3. API Gateway authorizes the request and invokes the route Lambda.
4. Lambda validates the action and computes the result.
5. Lambda reads and writes canonical state in DynamoDB.
6. API responds with updated trusted state.
7. Frontend re-renders from backend output.

This same pattern drives clicking, upgrades, trades, and leaderboard updates.

---

## Frontend Overview
The frontend uses a terminal-style interface delivered as static assets with responsive client-side interactions and leaderboard polling. It renders generated market visuals, including a candlestick graph, histogram-style volume bars, and a live ticker stream. The browser handles presentation and request orchestration, while score and economy outcomes remain backend-controlled.

---

## Project Goal
This project emphasizes backend engineering for game-state integrity: authenticated users, protected actions, canonical persistence, and shared leaderboard competition powered by AWS.

---

## Local Development
This project is a static site. No build step is required for normal local use.

Run a simple static server from the repository root:
```bash
python3 -m http.server 5500
```

Note: Using port 5500 is important for Amazon Cognito callbacks to allow you to login.

Then open [http://localhost:5500/public/index.html](http://localhost:5500/public/index.html).

Production deployment: [https://financeclicker.net](https://financeclicker.net).
