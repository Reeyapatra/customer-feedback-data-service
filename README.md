Customer Feedback Data Service
Serverless customer feedback data prototype | TypeScript, SQL, Cloudflare Workers & D1
A serverless prototype for retrieving and displaying customer feedback data using TypeScript, SQL, Cloudflare Workers, and D1. The current implementation uses three illustrative comments to demonstrate database connectivity and browser presentation, providing a starting point for a future customer feedback analytics workflow.
Status: database integration prototype based on Cloudflare's Worker + D1 template. The current interface is a data preview. Analytics and AI inference are not implemented.
Overview
Feedback analytics starts with storing and retrieving feedback reliably. This prototype implements an initial workflow:
1. A SQL migration creates a comments table and inserts three sample records.
2. A Cloudflare Worker receives an HTTP request.
3. The Worker queries D1 through the DB binding.
4. Results are serialized as formatted JSON.
5. An HTML page displays the query and records.
The comments are demonstration data, not a real customer dataset. No business impact or model performance has been measured.
Implemented features
- SQL table creation and sample data insertion through a migration.
- Database queries using the Cloudflare D1 binding.
- A TypeScript HTTP handler returning an HTML response.
- A rendering function displaying formatted database results.
- Wrangler configuration for runtime bindings and observability.
- Package scripts for migrations, local development, type generation, checks, and deployment.
An AI binding is configured in wrangler.json. The code does not call env.AI, select a model, or generate predictions.
Technology stack
Technology	Current use
TypeScript	Request handling and HTML rendering
Cloudflare Workers	Serverless runtime
Cloudflare D1	SQL storage
SQL	Schema, seed data, and retrieval
HTML	Browser preview
Wrangler	Local development, migrations, and deployment
npm / package-lock.json	Dependency installation and locked versions
Workers AI binding	Configuration only


Implementation
Database schema
CREATE TABLE IF NOT EXISTS comments (
    id INTEGER PRIMARY KEY NOT NULL,
    author TEXT NOT NULL,
    content TEXT NOT NULL
);
Field	Type	Purpose
id	INTEGER PRIMARY KEY	Unique comment identifier
author	TEXT NOT NULL	Author name
content	TEXT NOT NULL	Comment text


The migration inserts these sample records:
Author	Comment
Kristian	Congrats!
Serena	Great job!
Max	Keep up the good work!


Query and response
The handler in src/index.ts executes:
SELECT * FROM comments LIMIT 3;
It passes the records to renderHtml() and returns a response with the text/html content type. The rendering function in src/renderHtml.ts shows the SQL query and JSON results, using external Cloudflare styling and images.
The query returns at most three records. Without an ORDER BY, it does not guarantee the newest records or a particular order. The handler currently does not distinguish routes or HTTP methods.
Repository layout
The configuration expects these paths. If your download puts every file in one folder, restore this layout before running the application.
Path	Purpose
src/index.ts	Worker entry point
src/renderHtml.ts	HTML rendering
migrations/0001_create_comments_table.sql	Schema and sample data
wrangler.json	Worker and binding configuration
worker-configuration.d.ts	Generated runtime and binding types
tsconfig.json	TypeScript settings
package.json	Dependencies and scripts
package-lock.json	Dependency lockfile
.gitignore	Excludes dependencies, runtime state, and environment files
README.md	Documentation


Run locally
Prerequisites
- A Node.js version supported by the pinned Wrangler release.
- npm.
- A Cloudflare account for remote database setup or deployment.
- pnpm only if using the existing npm run dev script.
1. Install dependencies
Run from the project root:
npm ci
2. Initialize the local database
Ensure the SQL file is inside migrations/, then run:
npx wrangler d1 migrations apply DB --local
DB is the binding name in wrangler.json. The --local flag targets the local development database.
3. Start the application
npx wrangler dev
Open the local URL printed in the terminal. The page should show the sample comments.
Script note: the existing dev script calls pnpm seedLocalD1, so npm run dev also requires pnpm. The two commands above provide an npm-only workflow.
4. Run build checks
npm run check
This runs TypeScript compilation checks and a Wrangler deployment dry run. It does not deploy the application and is not a unit-test suite.
After changing bindings, regenerate the Cloudflare types:
npm run cf-typegen
Deploy to Cloudflare
Publishing source code to GitHub and deploying the application are separate steps.
1. Authenticate with Cloudflare:
   npx wrangler login
2. If you do not own the configured database, create your own:
   npx wrangler d1 create feedback-aggregation-database
3. Update database_id and, if necessary, database_name in wrangler.json using the returned values. Keep the binding name DB to match the application.
4. Deploy:
   npm run deploy
The existing predeploy script applies migrations to the configured remote database before deployment. Verify that the configuration points to your intended database.
Open the Worker URL printed by Wrangler.
Career relevance
Role	Current evidence	What would strengthen the project
Backend / data infrastructure	SQL schema, database binding, HTTP handler, deployment configuration	Validated ingestion, pagination, error handling, and tests
Data Analyst	Basic SQL retrieval and a preview of stored data	A meaningful dataset, aggregate queries, KPIs, charts, and findings
Data Scientist	A storage layer for a potential modeling workflow	Labeled data, baselines, train/validation/test splits, evaluation, and error analysis
AI Engineer	AI binding configuration, with no inference code	Model integration, structured outputs, evaluation, retries, and latency/cost measurements


Current positioning: a serverless SQL integration prototype. It should not yet be presented as a completed data science project or an AI-powered analytics dashboard.
Roadmap — not implemented
Phase 1: Feedback ingestion and reliability
- Add a feedback submission endpoint with input validation.
- Escape comment content before embedding it in HTML.
- Add method-specific routing, database error handling, and pagination.
- Extend the schema with timestamps, feedback source, and product category.
- Add tests for migration, retrieval, and rendering behavior.
Phase 2: Feedback analytics
- Import a documented dataset with clear usage permissions.
- Calculate feedback volume, category distribution, and trends over time.
- Add charts, filters, and drill-down views.
- Document business questions and recommendations supported by the data.
Phase 3: AI-assisted analysis
- Integrate a model for sentiment classification, topic tagging, or summarization.
- Create a human-labeled evaluation set.
- Compare predictions against a simple baseline.
- Measure quality, failure cases, latency, and cost.
- Document how feedback is handled before sending it to a model.
These additions would provide stronger evidence for Data Analyst and AI Engineer roles. A Data Scientist version would also need a rigorous modeling and evaluation workflow.
Known limitations
- Only three sample comments are seeded.
- The query retrieves at most three rows and does not sort them.
- There is no submission form, ingestion API, filtering, or pagination.
- No analytical metrics, charts, statistical analyses, or model outputs are implemented.
- There is no application-level database error handling.
- Comment text is interpolated into HTML without escaping. Use trusted demonstration data until rendering is made safe for untrusted input.
- No automated tests or measured performance results are included in the supplied files.
- The supplied generated Env interface does not include the configured AI binding; regenerate types before adding AI calls.
- A working public deployment has not been verified as part of this documentation update.
Template attribution
This project is based on Cloudflare's Worker + D1 Database template. The supplied handler, rendering page, and seed records retain the template's basic behavior.
- Original template
- D1 documentation
- Wrangler D1 commands
- Workers documentation