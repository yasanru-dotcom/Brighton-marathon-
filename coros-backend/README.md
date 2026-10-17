# COROS integration — secure backend plan

The public GitHub Pages app is only the front end. Do **not** put OAuth client secrets, access tokens, refresh tokens, or backend credentials in this repository or in `index.html`.

## Current status

- The front end now includes a COROS status card and links to the official documentation.
- **Live COROS authorization and data syncing are not enabled yet.** The card intentionally says so.
- Existing planner checkboxes continue to use the original `brighton315progress` localStorage key and original week/day keys.

## Recommended first integration

For a personal app, COROS documents a self-service OAuth 2.0 route through its MCP endpoint: https://mcp.coros.com/mcp . See https://support.coros.com/hc/en-us/articles/53181619102996-Build-on-COROS-MCP .

The backend should:
1. Run on a serverless backend with HTTPS (for example, a Cloudflare Worker or another managed server).
2. Implement COROS's documented OAuth 2.0 flow, including state validation and PKCE if required by the current COROS OAuth specification.
3. Keep authorization codes and tokens server-side; encrypt tokens at rest where stored, use short-lived sessions, and never log secrets.
4. Expose narrowly scoped authenticated routes to the front end, such as `GET /api/status` and `GET /api/activities?from=YYYY-MM-DD&to=YYYY-MM-DD`.
5. Poll COROS MCP for activity data on demand or on a controlled schedule, subject to COROS's documented limits; map activity date, distance, duration, pace and supported details into the dashboard.
6. Validate activity data, handle revoked authorization and token expiry, and offer a disconnect/delete-data path.
7. Keep the static planner usable if the backend is unavailable.

Do not build an OAuth flow by guessing endpoints. Use the current COROS MCP implementation/specification and test its authorization, token exchange, refresh, and MCP request flow before enabling the Connect button.

## When Partner API is needed

COROS says the Partner API is the route for webhooks, multi-user credentials, two-way activity sync, and GPX support. It has a platform application and security requirements: https://support.coros.com/hc/en-us/articles/53181766856724-Partner-API-Access . For a single-person planner, start with the self-service MCP route and treat true webhook-driven/two-way syncing as a later step.

## Suggested deployment secrets

Store secrets only in the backend host's encrypted environment/secrets manager, never in GitHub Pages source:
- COROS OAuth client configuration, if issued/required by the current flow
- Session signing/encryption key
- Allowed front-end origin: `https://yasanru-dotcom.github.io`

Use an exact redirect URI registered/allowed by COROS, HTTPS only, strict CORS, CSRF/state protection, rate limits, and no wildcard credentialed CORS. Avoid storing any health data not needed by the planner.

## Manual acceptance checklist before turning on sync

- [ ] OAuth login and denial paths both work.
- [ ] State/PKCE validation and redirect URI checks pass.
- [ ] No secrets appear in browser source, network responses, logs, or Git history.
- [ ] A user can view only their own activities.
- [ ] Duplicate activities do not inflate weekly mileage.
- [ ] Activity matching uses date/time zone and distance tolerance, with manual correction for ambiguous matches.
- [ ] Disconnect revokes/deletes tokens and cached activity data.
- [ ] The planner works with the backend offline.
