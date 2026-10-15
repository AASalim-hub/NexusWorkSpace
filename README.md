# NexusWork

A responsive workspace operations app for Nexus Work Space. The project includes a React interface, an Express API, MongoDB models, role-based access, and a deployable single-origin production build.

## What is here

- Sign-in and registration. Public registration always creates an `intern`; roles are assigned only by an administrator.
- Responsive dashboards and navigation for interns, employees, supervisors, laptop monitors, and administrators.
- Attendance check-in and check-out with short-lived, rotating QR sessions and server-side duplicate prevention.
- Laptop inventory and the request → decision → physical handover → return workflow.
- Weekly task assignment, progress submissions, supervisor review, and feedback.
- Daily activity reports, supervisor-to-intern access assignments, supervisor review, requested revisions, and preserved earlier versions.
- Searchable activity timeline, administrator role management, and auditable privileged changes.
- Input validation, password hashing, server-side sessions, CSRF checks, role checks, rate limits, security headers, and privacy-conscious error responses.

## Run locally

You need Node.js 20 or newer and a MongoDB instance. A local MongoDB replica set or a managed MongoDB Atlas cluster is needed for the transactional laptop handover flow.

1. Open this folder in VS Code.
2. In a terminal, install packages:

   ```powershell
   npm install
   ```

3. Create a local environment file and set a unique secret:

   ```powershell
   Copy-Item .env.example .env
   ```

   Edit `.env`. Set `MONGODB_URI` and `SESSION_SECRET` (at least 32 random characters). Set `WORKSPACE_TIMEZONE` if needed; it defaults to `Africa/Lagos`. The default check-in cutoff is `09:00`; optionally set `CHECK_IN_CUTOFF` using 24-hour `HH:mm`.

4. Start MongoDB, then start the client and API:

   ```powershell
   npm run dev
   ```

   Open <http://localhost:5173>.

5. Create the first administrator in another terminal:

   ```powershell
   npm run create-admin
   ```

   Follow the prompts. Create that administrator before inviting users through registration. The script never creates a built-in password or default account.

New registrations receive intern access. Sign in as the administrator, open **People**, then assign authorized roles. Add initial inventory from **Laptop desk**. Users can then submit and manage records according to their roles.

## Build and deploy

The production server serves the compiled React app and `/api` from one origin. Configure your deployment with:

- `NODE_ENV=production`
- `MONGODB_URI` for a private MongoDB replica set/Atlas cluster
- A fresh, high-entropy `SESSION_SECRET` (32+ characters), stored as a secret
- `CLIENT_ORIGIN=https://your-real-domain.example` (origin only, no path)
- `WORKSPACE_TIMEZONE` and `CHECK_IN_CUTOFF` for the workplace
- `TRUST_PROXY=1` only if the service is behind a trusted reverse proxy that sets forwarding headers

Then run:

```sh
npm install
npm run build
npm run start
```

Terminate TLS at a properly configured reverse proxy or platform edge. Use HTTPS, restrict database network access, enable automated encrypted backups, and set a production session secret. Do not commit `.env`. Provision at least one administrator with `npm run create-admin` before using the service. If the service runs multiple instances, use a shared rate-limit store and configure the edge to limit abusive traffic; the built-in login limiter is process-local.

## Security model

The browser only controls presentation. Each protected API route checks the authenticated session and required role again on the server. Sessions are stored in MongoDB; the session cookie is HTTP-only, SameSite Strict, and Secure in production. State-changing browser requests require an origin allowlist and a session-bound CSRF token. Passwords are bcrypt-hashed. Request bodies are size-limited, schema-validated, and unknown fields are rejected. Sensitive workflows are state-checked on the server, including expired attendance sessions, duplicate scans, duplicate daily reports, inventory availability, and loan status. Privileged and review actions create audit/history records.

The QR is a short-lived bearer token. Rotating it limits reuse; it cannot prevent a person from sharing a live code. Protect the QR display and use workplace policy for identity checks. Never put a real QR in a public location.

This is a strong application foundation, not a completed security certification or production pentest. Before real staff data is used, have the organization review its data retention and attendance policies, configure email verification/password recovery, confirm its supervisor-to-intern authorization model, and commission a security review. No report attachment upload is enabled. No default demo users or sample personal records are shipped.

## Current first-release boundaries

The blueprint includes optional password reset, image uploads, attendance setting screens, SIWES export, notification inboxes, and richer correction/transaction-history views. They are not implemented in this first usable release. Password recovery and email verification need a real, verified email provider and domain configuration. Uploads need private object storage and malware/content checks. These are intentionally called out instead of being represented as working features. The foundation is structured so those can be added as separate modules.
