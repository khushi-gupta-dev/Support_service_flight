# Support Service Documentation

Support Service provides customer-only AI assistance. It uses Gemini for
natural-language help and a verified local fallback for documented SkyLink
workflows. It may query public flight inventory for search assistance but
cannot access private bookings, payments, credentials, or account state.

## Runtime

- Default port: `3003`
- Start: `npm run dev`
- Gateway prefix: `/supportService`
- Direct health route: `GET /`

## API

- `POST /api/v1/chat`
  - Input: `{ message, history }`
  - Output: `{ answer, suggestions }`
- `POST /api/v1/assistant`
  - Input: `{ message, history }`
  - Output: validated `collect_details`, `flight_results`, or
    `conversation_end` responses

History is bounded. Requests are limited to 20 per IP per 10 minutes.
The Gateway requires an authenticated non-admin customer and supplies a
trusted role header; direct unauthorised requests are rejected.

## Flight-search assistance

The assistant extracts a route/search request and queries Flights Service.
Live results come only from the public catalogue. If Gemini is unavailable,
the local parser and FAQ guidance continue to support search, booking,
payments, cancellations, refunds, tickets, and account navigation.

## Configuration

```env
PORT=3003
FLIGHT_SERVICE=http://localhost:3001
GEMINI_API_KEY=...
GEMINI_MODEL=gemini-3.5-flash
```

The API key is sent only server-to-server in `x-goog-api-key`; it is never
returned to the browser. Startup logs reveal only whether a key is configured.
