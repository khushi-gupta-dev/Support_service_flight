# Support Service

Server-side LLM help for SkyLink. The service sends the customer's message and fixed SkyLink product guidance to Google's Gemini API. Its booking assistant can extract a flight-search request and query the public Flights Service catalogue, but it does not create bookings or access authentication tokens, payment details, or private booking data. The chat UI discloses that messages go to Gemini. Conversation history is bounded and used only to answer the current assistance request.

## Configure and run

1. Install dependencies with `npm install`.
2. Copy your local environment settings to `.env` and set `GEMINI_API_KEY`. The default `GEMINI_MODEL` is `gemini-3.5-flash`; set it to another model available to your Google AI Studio project if needed. Keep the key private and out of source control.
3. From this directory, start with `npm run dev`; default port is `3003`. The service loads this directory's `.env` regardless of the shell's current directory. Startup logs show the model and whether a key is configured, never the key itself.
4. Set `FLIGHT_SERVICE=http://localhost:3001` and `SUPPORT_SERVICE=http://localhost:3003` in the local environments. Run the Flights Service, Support Service, and Gateway. The frontend calls the Gateway at `/supportService/api/v1/chat` and `/supportService/api/v1/assistant`.

## API

- `GET /` — health response.
- `POST /api/v1/chat` — accepts `{ "message": "...", "history": [{ "role": "user|assistant", "text": "..." }] }` (message 1–500 characters) and returns `{ "answer": "...", "suggestions": ["..."] }`. The service uses a bounded recent history window, answers the current question, and provides contextual follow-up options. An explicit closing response can end the conversation. When Gemini is unavailable or quota is exhausted, verified local FAQ guidance covers search, booking, payments, cancellations, refunds, account navigation, and tickets.
- `POST /api/v1/assistant` — accepts the same bounded `history` plus `{ "message": "..." }` and returns `collect_details` or validated `flight_results`. Results come from Flights Service; the assistant never creates a booking or payment. Closing phrases return `conversation_end`.

The Gateway requires an authenticated `customer` role for `/api/v1` and supplies the trusted role header. Direct requests without that header are rejected. Requests are limited to 20 per IP per 10 minutes. The Gemini key is sent only from this server to Google in the `x-goog-api-key` header, never to the browser. Errors for credentials, unavailable models, and quota are mapped to actionable messages; provider details and secrets are not returned.

If Gemini is temporarily unavailable, general help chat returns verified local guidance for documented search, booking, payment, cancellation, ticket, and account workflows instead of failing silently. Flight-search extraction continues to use its local parser fallback, while live inventory still comes only from Flights Service. Confirm Support Service and API Gateway were restarted after configuration/code changes. Check the Support Service terminal for its startup key-configured flag and Gemini HTTP status; never print the API key.
