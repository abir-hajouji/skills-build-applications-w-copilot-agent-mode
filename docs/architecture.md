# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions
Record your plan here before editing code.
- Presentation tier decision: Use the shared API client and render each capability with accessible loading, error, empty, and success states.
- Logic tier decision: Keep request validation and business behavior in explicit service functions; return deliberate 4xx responses for invalid or missing resources.
- Data tier decision: Enforce required fields and defaults in Mongoose schemas, and keep seed operations idempotent without clearing application data.
- Activity tracking approach: Validate activity input and existing user/team IDs, then store activity minutes as points with its creation time.
- Team management approach: Validate team names and member IDs, and return the submitted team fields and members.
- Leaderboard approach: Sum activity points by team and return results in descending point order with 1-based ranks.
- Workout suggestions approach: Match a known user's goal or recent activity to available workouts and explain each recommendation.

## Test plan

### Baseline results

- `npm test --prefix octofit-tracker/backend` — 8 tests failed and 1 passed. Four data-contract failures indicate missing required-field validation in Mongoose schemas. Four API failures indicate unfinished activity/team creation, leaderboard aggregation, and workout-suggestion reasoning. The seed idempotency test passed.
- `npm test --prefix octofit-tracker/frontend` — all 15 presentation-contract tests failed. The capability pages are placeholders and do not yet fetch or render API data or show the required loading, empty, and error states.

### Implementation order and verification

1. `npm run test:data --prefix octofit-tracker/backend` — checks required model fields, defaults, and seed behavior. Failures indicate a data-tier issue.
2. `npm run test:api --prefix octofit-tracker/backend` — checks activity/team validation and persistence, leaderboard totals/ranks, and workout suggestions. Unexpected 5xx responses or incorrect results indicate backend service, route, or data-access issues.
3. `npm test --prefix octofit-tracker/frontend` — checks API requests and accessible loading, success, empty, and error states. Failures indicate API-client or presentation-component issues.
4. `npm run build --prefix octofit-tracker/backend` — checks backend TypeScript and build configuration.
5. `npm run build --prefix octofit-tracker/frontend` — checks frontend TypeScript, Vite, and build configuration.
