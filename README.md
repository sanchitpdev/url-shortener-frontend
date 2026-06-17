# ShrinkIt — URL Shortener (Frontend)

> The web UI for the ShrinkIt URL shortener. A single-file, zero-dependency frontend that talks to a Spring Boot + Redis + PostgreSQL backend running on AWS ECS Fargate.

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Vercel](https://img.shields.io/badge/Deployed_on-Vercel-000000?style=flat&logo=vercel&logoColor=white)

**Live:** [url-shortener-version1.vercel.app](https://url-shortener-version1.vercel.app)
**Backend repo:** [sanchitpdev/url-shortener](https://github.com/sanchitpdev/url-shortener)

<!-- TODO: Add a screenshot or GIF of the UI here (paste a URL → get a slug → copy). The repo currently references preview.png — add that image or update the path. -->

---

## What it is

A deliberately minimal frontend: one `index.html` containing the markup, styles, and logic — no framework, no build step, no `node_modules`. Paste a long URL, get back a 7-character short link, copy it with one click. Recent links are kept in the browser so you can grab them again.

The interesting part isn't the size of the UI — it's how it connects to the backend cleanly despite a mixed-content constraint (see below).

## Features

- **One-field shortening** — paste a URL and submit; if you omit the scheme, `https://` is added automatically.
- **One-click copy** — copies the short URL to the clipboard via the Clipboard API, for both the latest result and any item in history.
- **Recent history** — recently created links are stored client-side and re-rendered on load.
- **Keyboard-friendly** — press Enter in the input to shorten.
- **Dark, typographic UI** — JetBrains Mono + Sora, a teal accent, fully responsive.

## How it connects to the backend

The frontend calls its API with a **relative path** (`/api/shorten`), and Vercel rewrites that to the backend:

```json
// vercel.json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "http://urlshortener-alb-1372942651.ap-south-1.elb.amazonaws.com/api/:path*"
    }
  ]
}
```

This solves a real problem. The site is served over **HTTPS** on Vercel, but the backend load balancer is **HTTP**. A browser blocks an HTTPS page from calling an HTTP API (mixed content). By calling a same-origin `/api/*` path and letting Vercel proxy it to the ALB server-side, the browser only ever sees a secure same-origin request — no mixed-content block and no CORS preflight from the browser's perspective.

## Tech stack

| Part | Choice |
|---|---|
| Markup / styles / logic | Single `index.html` — HTML5, CSS3, vanilla JavaScript |
| Fonts | JetBrains Mono, Sora (Google Fonts) |
| API calls | `fetch` to a relative `/api` path |
| Hosting | Vercel (static) with an API rewrite proxy to the backend |
| Dependencies | None — no framework, no bundler |

## Run locally

Because it's a single static file, you can just open it — but to exercise the `/api` calls you need something that applies the Vercel rewrite (or point the code at the backend directly).

```bash
git clone https://github.com/sanchitpdev/url-shortener-frontend.git
cd url-shortener-frontend

# Quick static preview (API calls will 404 without the proxy):
python3 -m http.server 5173
# open http://localhost:5173

# To exercise the API locally, use the Vercel CLI so vercel.json rewrites apply:
# npm i -g vercel && vercel dev
```

<!-- TODO: If you ever want to point at a local backend during development,
note here how you switch the API base (currently the code uses a relative path
and relies on the Vercel rewrite). -->

## Deployment

Pushed to Vercel as a static site. `vercel.json` proxies `/api/*` to the AWS ALB in front of the ECS Fargate backend, so the deployed frontend and API share an origin.

## License

See [LICENSE](LICENSE).

## Author

**Sanchit Pawar** — [GitHub](https://github.com/sanchitpdev) · [LinkedIn](https://linkedin.com/in/sanchitpawar)
