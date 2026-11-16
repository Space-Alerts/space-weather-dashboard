# Space Weather Operations Dashboard

Deployable React/Vite frontend plus a Cloudflare Worker and D1 database.

## Included
- Public live-feed dashboard at `/`
- Private alert administration route at `/admin`
- Server-side NOAA/NASA feed checks to avoid browser CORS failures
- D1 schema for threshold rules and alert history
- Five-minute Worker schedule
- No synthetic timestamps

## 1. Prerequisites
Install Node.js 20+ and Git. Create free GitHub and Cloudflare accounts.

## 2. Deploy the Worker API
```bash
cd worker
npm install
npx wrangler login
npx wrangler d1 create space-weather-dashboard
```
Copy the returned database ID into `worker/wrangler.toml`. Replace `CORS_ORIGIN` with the final Pages URL after step 3.

```bash
npm run db:migrate
npm run deploy
```
Record the Worker URL, for example `https://space-weather-api.<account>.workers.dev`.

## 3. Deploy the frontend
At the repository root:
```bash
npm install
npm run build
```
Push the whole project to GitHub. In Cloudflare Pages, import the repository with:
- Build command: `npm run build`
- Output directory: `dist`
- Environment variable: `VITE_API_BASE_URL` = your Worker URL

Redeploy after setting the variable.

## 4. Protect `/admin` using email OTP
In Cloudflare Zero Trust, create a self-hosted Access application for the exact admin hostname/path, such as `dashboard.example.com/admin*`.
Create an Allow policy containing only `andycrabbs@gmail.com`, and enable the One-time PIN identity provider. Do not protect `/` or `/api/status`.

Important: if the Worker remains on `workers.dev`, also create an Access application for `space-weather-api.<account>.workers.dev/api/admin/*`. Use the same Allow policy. The public `/api/status` route must remain public.

## 5. Finish CORS
Set `CORS_ORIGIN` in `worker/wrangler.toml` to the exact Pages origin, then run `npm run deploy` from `worker`.

## Security notes
- The administrator email is documented for configuration only and is not embedded in the frontend bundle.
- Authentication is enforced at Cloudflare's edge, not by React.
- Notification delivery is intentionally not enabled in this version. Choose a transactional email provider, add its API key with `wrangler secret put`, and implement delivery before relying on alerts.
- Validate alerting in a test environment before operational use.
