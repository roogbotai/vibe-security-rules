# vibe-security-rules

Plain-English security rules and a checklist for apps built with AI coding tools.

AI tools are good at making an app work. They are less careful about keeping it safe. A key pasted into front-end code, a database table anyone can read, or a missing header can all slip through. This repo gives you two things to help.

| File | What it is for |
| --- | --- |
| [`AGENTS.md`](AGENTS.md) | Security rules written for an AI coding tool to follow. Drop it in your project. |
| [`CHECKLIST.md`](CHECKLIST.md) | A pre-launch checklist for you, the builder. Each item says how to check it. |

## How to use it

1. Copy `AGENTS.md` into the root of your project. [AGENTS.md](https://agents.md/) is a plain Markdown file that many AI coding tools read as project instructions. Check your tool's docs to see if it reads that file name. If it does not, paste the text into the place where your tool keeps its rules or custom instructions.
2. Go through `CHECKLIST.md` before you share your app.
3. Ask your AI tool: "Which of the rules in AGENTS.md did you not follow in this change?"

## What the rules cover

- Secrets and API keys
- Environment variables (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`)
- Supabase keys and Row Level Security
- Security headers
- Checking input on the server
- Login, access and cookies
- `robots.txt` and `sitemap.xml`
- Files that must never be public

Every rule that depends on a product or standard links to its official docs.

## What this is not

- It is not a security audit. Following these rules lowers risk. It does not prove an app is safe.
- It is not a full list. It covers common problems in small apps, not everything.
- It does not cover every framework or database. If you use Firebase or another backend, check its docs too.
- It is not legal or compliance advice.

## Check your live site

If you want a quick look at what a stranger can see on your live site, [rooGBotVC](https://vc.roogbotai.com) has a free summary scan with no account. It checks the "outside" items in the checklist: leaked keys in public JavaScript, open `/.env` and `/.git` files, public source maps, HTTPS, security headers, and `robots.txt` and `sitemap.xml`. It cannot see inside your database.

Disclosure: rooGBotVC is made by rooGBot AI, who also wrote this repo. The free summary costs nothing. A full report with fix prompts is paid. You do not need the scan to use anything here.

## Contributing

Fixes are welcome, especially corrections. Please open an issue or a pull request. For every technical claim, link the official docs.

## Licence

[MIT](LICENSE).
