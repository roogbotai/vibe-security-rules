# Security rules for AI coding tools

Follow these rules in every change you make to this project. They are written for AI coding tools, but a person can use them too.

If a rule gets in the way of a task, stop and tell the user which rule it is and why. Do not quietly skip it.

Mark every rule you could not follow in your final message to the user.

## 0. How you work

- Never "fix" a permission error by switching to a more powerful key, turning a protection off, or making a setting wider. Find the cause. Fix the cause.
- Do not ask the user to paste a secret key into the chat. Tell them the name of the variable, and tell them where to put the value (a `.env` file or the host's settings page).
- If you see a real secret in the code, in a log, or in a message, say so right away. Do not copy it into new files, docs, tests or commit messages.
- When you finish, list the security-relevant things you changed: keys, env vars, database rules, headers, auth, public files.

## 1. Secrets

- A secret is anything that gives power over an account or data: API keys, database passwords, `service_role` or secret keys, private keys, webhook signing secrets, OAuth client secrets.
- Never write a secret into source code, a test, a README, a config file that is committed, or front-end code. Read it from an environment variable on the server.
- Keep `.env` and other local env files in `.gitignore`. Check `.gitignore` before you create one. (Next.js says you almost never want to commit these files: [Next.js environment variables](https://nextjs.org/docs/app/guides/environment-variables).)
- Keep a `.env.example` with variable names and fake values only.
- Any code that runs in the browser is public. Anyone can read it with View Source or DevTools. A key in that code is a leaked key.
- Secrets may only be used in server code: API routes, server actions, server components, serverless or edge functions, or a backend service.
- If a secret has been in public code or committed, treat it as leaked. Making a new one and removing the old one is the fix. Deleting it from the code is not enough. (Supabase describes this for its secret keys: [Supabase API keys](https://supabase.com/docs/guides/api/api-keys).)
- Some keys are made to be public, such as a Stripe publishable key (`pk_`). Secret keys (`sk_`) are not. Read the provider's docs before you decide. ([Stripe API keys](https://docs.stripe.com/keys))
- More: [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html).

## 2. Environment variables

Some frameworks copy variables into the browser code when the name starts with a certain prefix.

| Framework | Prefix that goes to the browser |
| --- | --- |
| Next.js | `NEXT_PUBLIC_` |
| Vite | `VITE_` |
| Create React App | `REACT_APP_` |

- Never put a secret in a variable with one of these prefixes. Next.js inlines `NEXT_PUBLIC_` values into the JavaScript sent to the browser at build time ([docs](https://nextjs.org/docs/app/guides/environment-variables)). Vite says `VITE_*` variables should not contain sensitive information such as API keys ([docs](https://vite.dev/guide/env-and-mode)).
- Before you add a variable with one of these prefixes, ask: "Is it fine for any stranger to read this?" If not, drop the prefix and use it on the server only.
- If a feature needs a secret and the app has no server (for example, a plain Vite app), do not put the secret in the front end. Add a small server-side function or backend that holds the key and makes the call.

## 3. Supabase: keys and Row Level Security (RLS)

Keys:

- The publishable key (`sb_publishable_...`, or the older `anon` key) is meant for the browser. Supabase says it is safe to expose online.
- The secret key (`sb_secret_...`, or the older `service_role` key) bypasses RLS. Use it only in backend code such as servers and Edge Functions. Never put it in a browser, a shipped app or source control. ([Supabase API keys](https://supabase.com/docs/guides/api/api-keys))
- If a tool or tutorial tells you to use the secret key in front-end code to get past an error, do not do it. The cause is usually a missing or wrong RLS policy, or a missing table grant. Fix that instead.

Row Level Security:

- Supabase says a table in an exposed schema without RLS is readable and writable by any role with a grant on it. Enable RLS on every table in an exposed schema. ([Supabase RLS docs](https://supabase.com/docs/guides/database/postgres/row-level-security))
- Once RLS is on, no data is reachable with the publishable key until a policy allows it. Write the smallest policy that works. For example, "a signed-in user can read rows where `user_id` matches their own id".
- Do not write a policy that gives `anon` (signed-out visitors) more than they should have. `using (true)` is only OK for data that is meant to be public.
- Do not base policies on `user_metadata`. Supabase warns that signed-in users can change it. ([Supabase RLS docs](https://supabase.com/docs/guides/database/postgres/row-level-security))
- Views skip RLS by default. On Postgres 15 and above, create views with `security_invoker = true`. ([Supabase RLS docs](https://supabase.com/docs/guides/database/postgres/row-level-security))
- Whenever you add a table, add RLS and policies in the same migration.

## 4. Security headers

Headers are short instructions your server sends to the browser. Add them in your host's settings or in the app config. In Next.js you can use the [`headers` option in `next.config.js`](https://nextjs.org/docs/app/api-reference/config/next-config-js/headers).

Check that these are set on pages:

| Header | What it is for | Starting value |
| --- | --- | --- |
| `Strict-Transport-Security` | Tells browsers to use HTTPS only for this site ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)) | `max-age=31536000` |
| `Content-Security-Policy` | Limits what the page may load and run ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)) | Site-specific. See below. |
| `X-Frame-Options` (or CSP `frame-ancestors`) | Stops other sites from putting your app in a frame ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options)) | `DENY` (or `SAMEORIGIN` if you frame your own pages) |
| `X-Content-Type-Options` | Stops browsers guessing file types ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Content-Type-Options)) | `nosniff` |
| `Referrer-Policy` | Controls how much page address is shared when a user clicks a link ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy)) | `strict-origin-when-cross-origin` |

Rules:

- A Content-Security-Policy can break a working app if it is too strict. Start in report-only mode (`Content-Security-Policy-Report-Only`), look at what it would block, then switch to enforcing. MDN describes this ([MDN CSP guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)).
- Avoid `'unsafe-inline'` in `script-src` where you can. MDN says it defeats much of the purpose of having a CSP.
- Make sure `http://` redirects to `https://`.
- Do not copy a header value without understanding it. Say what each one does in your notes.
- More: [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/).

## 5. Server-side input checks

- Checks in the browser are for the user's comfort. They are not security. Anyone can skip the browser and call your server directly. Check every input again on the server. ([OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) lists "client-only checks" as a common pitfall.)
- For every API route, server action and form handler, check: type, length, format, and which fields are allowed. Reject anything else.
- Use an allowlist ("only these values") instead of a denylist ("not these bad values").
- Put limits on size: text length, array length, upload size and file type.
- Use parameterized queries or your database library's safe query functions. Never build SQL by joining strings with user input. ([OWASP Query Parameterization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html))
- Never trust an ID, price, role or user id sent by the browser. Look it up on the server.
- Verify webhook signatures before you trust a webhook. For example, Stripe says to verify with the `Stripe-Signature` header and your signing secret, using the raw request body. ([Stripe webhooks](https://docs.stripe.com/webhooks))

## 6. Auth and access

- Check who the user is and what they may do on the server, on every request. Hiding a button or a page in the front end is not access control.
- Deny by default. A new route or table is private until you decide who may use it. ([OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html))
- Admin pages and admin API routes need a server-side check that the user is an admin. Do not use a value the user can change, such as `user_metadata`, to decide this.
- Do not write your own login system if you can use your host's or database's auth. If you must store passwords, use a slow password hash such as Argon2id. Never store plain passwords. ([OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html))
- Login, sign-up and password reset must give the same generic message whether or not the account exists. ([OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html))
- Rate-limit login, sign-up, password reset and any endpoint that sends email or calls a paid API.
- Session cookies: set `HttpOnly`, `Secure` and a `SameSite` value. ([MDN Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie))
- With Supabase on a Next.js server, Supabase's guide says to use `supabase.auth.getClaims()` to protect pages and user data, and never to trust `getSession()` in server code. ([Supabase server-side auth for Next.js](https://supabase.com/docs/guides/auth/server-side/nextjs))
- Never send a user's secrets, other users' data, or stack traces in an error message.

## 7. robots.txt and sitemap basics

- `robots.txt` lives at the root of the site (`/robots.txt`). It is a public file. Never list secret or admin paths in it, because anyone can read it. ([Google: create a robots.txt file](https://developers.google.com/search/docs/crawling-indexing/robots/create-robots-txt))
- `robots.txt` is not a way to hide a page. Google says it is not a mechanism for keeping a page out of Google. Use `noindex` or password protection. ([Google: robots.txt intro](https://developers.google.com/search/docs/crawling-indexing/robots/intro))
- Real privacy comes from login checks on the server (rule 6), not from robots.txt.
- Do not ship `Disallow: /` on a live site unless the user wants it hidden from search engines.
- Add a `sitemap.xml` that lists the public pages you want found, and add a `Sitemap:` line to `robots.txt` ([Google: sitemaps overview](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview)). Do not put private pages in it.
- Keep private pages (account, admin, reports) out of the sitemap and mark them `noindex`.

## 8. Files that must never be public

- `.env` and `.env.*` files, the `.git` folder, backups, database dumps, and logs must not be served by the host. Only the build output should be deployed.
- Turn off public source maps in production unless you are sure you want your original code to be readable.
- Do not leave test pages, debug routes or sample admin accounts in a live app.

## Check your work

Run the live site through a scanner or through [CHECKLIST.md](CHECKLIST.md) before you tell the user it is ready to ship. These rules reduce risk. They do not prove an app is secure.
