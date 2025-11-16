# EDSBioSocialSurvey — Starter scaffold

This repo contains a starter scaffold to host a biosocial survey (frontend) that submits responses to Netlify Functions which store sanitized responses in Supabase. The public visualization page fetches a de-identified aggregated graph (co-occurrence network) from a serverless function and renders it with D3.js.

High-level goals:
- Protect participant privacy (don't store plaintext PII, hash IPs with server-side salt, store only JSON answers).
- Use Netlify Functions as the secure gate to Supabase to avoid exposing DB keys to clients.
- Serve a public visualization that uses only aggregated, de-identified data.

What’s included:
- netlify/functions/submitSurvey.js — POST endpoint to receive survey submissions, sanitize, validate, insert into Supabase.
- netlify/functions/getGraph.js — GET endpoint that builds and returns a de-identified co-occurrence graph (nodes + weighted edges).
- public/index.html — Minimal survey form that submits JSON to the submission function.
- public/visualization.html — D3.js visualization that fetches the aggregated graph.
- db/schema.sql — SQL to create the `responses` table in Supabase.
- .env.example — Environment variables needed.
- package.json — dev dependencies and a small script for local testing.

Privacy & security notes (please read and adapt for IRB / ethics):
- Require explicit consent in the survey. Submissions without consent are rejected.
- Strip common PII fields server-side (e.g., name, email, phone) by default; you should add any additional keys your Claude artifact might contain.
- IPs are not stored in plaintext: the submission function takes the request IP and stores a salted SHA-256 hash. The SALT is an env var stored only on Netlify.
- Do not serve raw responses from the public function. getGraph only returns aggregated co-occurrence counts (no respondent IDs or timestamps).
- Consider deploying with TLS (Netlify provides this) and enabling CAPTCHA (e.g., hCaptcha or reCAPTCHA) to reduce spam.
- If you plan to publish any subset of raw data, run a de-identification review and seek human review and, if necessary, IRB approval.

Supabase notes:
- Create a Supabase project and run `db/schema.sql` to create the table.
- Use the Supabase service_role key in Netlify Functions (stored in Netlify environment variables). Keep the key secret; functions run server-side and should be the only place with this key.
- As an improvement, you can instead use RLS (row level security) + custom auth to avoid the service role key; for the starter we use the function + service role pattern (simpler to set up).

Deployment:
1. Create a Supabase project and note SUPABASE_URL and SUPABASE_SERVICE_ROLE_KEY.
2. Add the environment variables (Netlify site settings):
   - SUPABASE_URL
   - SUPABASE_SERVICE_ROLE_KEY
   - SALT_FOR_IP_HASH
   - GRAPH_FIELDS (optional, comma separated keys from answers to use for graph, default "symptoms")
3. Push repo to GitHub and connect to Netlify; Netlify will auto-deploy.
4. Test form submission and the visualization.

Next actions I’ve left for you to do from here:
- Replace the sample front-end form fields with the full JSON schema from your Claude artifact.
- Add CAPTCHA if you expect public traffic.
- Run an ethics/privacy review of fields you collect.
- Iterate the graph-generation logic to align with the semantics of your survey answers.

If you want, I can:
- Convert your Claude artifact (the 127k chars) into the JSON keys and validation schema expected by submitSurvey.
- Add automated unit tests for the functions.
- Add CI / deployment instructions and a Netlify/TLS/CORS policy.
