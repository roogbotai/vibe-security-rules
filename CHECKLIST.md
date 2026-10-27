# Before you go live: a safety checklist for apps built with AI tools

A short list for people who built an app with an AI tool (Lovable, Bolt, Cursor, Replit, v0 or similar) and are about to share it. Tick each box. Each item says how to check it.

This list helps you find common, easy-to-fix problems. It does not prove your app is secure. If your app holds money, health data or other sensitive data, ask a security professional to review it.

## Part 1: Things anyone can see from outside

You can check these in a browser. A free scanner can check them for you too (see the end).

- [ ] **No secret keys in your front-end code.** Open your live site, press F12, go to the Sources or Network tab, and search your JavaScript files for `sk_`, `sk-`, `sb_secret_`, `service_role` and `PRIVATE KEY`. Keys that start with `pk_` or `sb_publishable_` are made to be public. Anything else that gives power over your account is not. Fix: move the call to a server and make a new key. Why: [Supabase API keys](https://supabase.com/docs/guides/api/api-keys), [Stripe API keys](https://docs.stripe.com/keys).
- [ ] **No secret in a variable that is sent to the browser.** Variables starting with `NEXT_PUBLIC_`, `VITE_` or `REACT_APP_` end up in public code. Why: [Next.js docs](https://nextjs.org/docs/app/guides/environment-variables), [Vite docs](https://vite.dev/guide/env-and-mode).
- [ ] **`/.env` is not public.** Go to `yoursite.com/.env`. You should get "not found", not a file.
- [ ] **`/.git/config` is not public.** Go to `yoursite.com/.git/config`. You should get "not found". Only your build output should be deployed.
- [ ] **Source maps are off (or you are happy for people to read your code).** In DevTools, look for `.map` files next to your scripts.
- [ ] **HTTPS everywhere.** Type `http://yoursite.com`. It should move you to `https://`. Add a [Strict-Transport-Security header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security).
- [ ] **Basic security headers are set.** In DevTools, open the Network tab, click the page request, and read the response headers. Look for `Content-Security-Policy`, `X-Frame-Options` (or `frame-ancestors`), `X-Content-Type-Options` and `Referrer-Policy`. See the [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/) and the [MDN CSP guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP).
- [ ] **`/robots.txt` and `/sitemap.xml` exist and make sense.** Open both. `robots.txt` should not say `Disallow: /` on a live site, and should not list secret paths. Why: [Google on robots.txt](https://developers.google.com/search/docs/crawling-indexing/robots/intro), [Google on sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview).

## Part 2: Things you must check from inside

A scanner that only looks at your public site cannot see these. Check them yourself.

- [ ] **Supabase: Row Level Security (RLS) is on for every table.** In the Supabase dashboard, open each table and check RLS is enabled, then read the policies. A table without RLS can be read and written by any role with a grant on it. Why: [Supabase RLS docs](https://supabase.com/docs/guides/database/postgres/row-level-security).
- [ ] **Policies match what you meant.** "Anyone can read everything" is only OK for data that is meant to be public. Do not base a policy on `user_metadata`.
- [ ] **Firebase: your Security Rules are not open to everyone.** Open the Rules tab in the Firebase console and read it.
- [ ] **Admin pages and admin API routes check the user on the server.** Hiding a link is not enough. Try opening an admin URL while logged out, and while logged in as a normal user. Why: [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html).
- [ ] **Forms are checked on the server too.** Browser checks can be skipped. Why: [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).
- [ ] **Database queries use parameters, not joined strings.** Why: [OWASP Query Parameterization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Query_Parameterization_Cheat_Sheet.html).
- [ ] **Login and sign-up have rate limits.** So do any endpoints that send email or call a paid API.
- [ ] **Webhook signatures are checked.** For example, [Stripe webhooks](https://docs.stripe.com/webhooks).
- [ ] **Secrets are only in your host's settings or a `.env` file that is in `.gitignore`.** Search your repo history too. If a secret was ever committed, make a new one.
- [ ] **No test pages, debug routes or sample admin accounts left in the live app.**

## Part 3: Tell your AI tool

Add the rules in [AGENTS.md](AGENTS.md) to your project so your AI tool follows them. Then ask it: "Which of these rules did you not follow in this change?"

## Free scan for Part 1

[rooGBotVC](https://vc.roogbotai.com) runs the checks in Part 1 on your live site. You paste a link and get a free summary with no account. It is read-only, and it does not log in to your app. For keys, it looks for known key formats, so it can miss a key it does not know. It cannot see inside your database, so it cannot check Part 2. (Disclosure: rooGBotVC is made by the same team as this repo. A full report with fix prompts is paid. The free summary is not.)
