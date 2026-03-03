# Splunk HEC Survey Form

A static, GitHub Pages–friendly survey page with 4 multiple-choice questions. On submit, answers are sent as a single JSON event to a **Splunk HTTP Event Collector (HEC)** endpoint.

## Features

- **4 questions** with multiple-choice answers
- **Submit** sends one HEC event: JSON body with question number as key and selected answer as value (e.g. `{"1":"Daily","2":"Good","3":"Reports","4":"Yes, definitely"}`)
- **Hamburger menu** (top left): configure Splunk **host/IP**, **port**, and **HEC token**; values are saved in the browser (localStorage) and are not stored in the repo

## GitHub Pages setup

1. Create a new repository (e.g. `splunk-hec-form`).
2. Push this folder so that `index.html` is at the **root** of the repo (or inside the branch/folder you use for Pages).
3. In the repo: **Settings → Pages** → Source: deploy from **main** (or your default branch), folder **/ (root)**.
4. After deployment, the site will be at `https://<username>.github.io/<repo>/`.

No server or build step is required; the page is static HTML/CSS/JS.

## CORS and browser security

The page runs entirely in the browser. When you click **Submit**, the browser sends a **cross-origin** request from your GitHub Pages origin (e.g. `https://username.github.io`) to your Splunk server (e.g. `https://splunk.example.com:8088`).

- If the Splunk HEC endpoint does **not** send the right [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) headers, the browser will block the request and you’ll see a network/CORS error in the console.
- To make it work from GitHub Pages, the Splunk server (or a reverse proxy in front of it) must allow your Pages origin, for example by including:
  - `Access-Control-Allow-Origin: https://<username>.github.io` (or `*` for any origin, less secure)
  - And, for POST with custom headers: `Access-Control-Allow-Methods: POST` and `Access-Control-Allow-Headers: Content-Type, Authorization`

If you cannot change Splunk’s CORS settings, you can use a small backend or serverless function that receives the form payload and forwards it to HEC from the server (no CORS then). This repo does not include that proxy.

## HEC event format

Each submit sends one POST request to:

- **URL:** `https://<host>:<port>/services/collector/event`
- **Header:** `Authorization: Splunk <hec_token>`
- **Body (JSON):**
  - `event`: object with keys `"1"`–`"4"` and values = selected answers
  - `sourcetype`: `hec_form_survey`

So in Splunk you’ll see one event per submission with fields `1`, `2`, `3`, `4` (and any HEC metadata).

## Security notes

- **No credentials in code:** Host, port, and HEC token are entered in the UI and stored only in the browser’s localStorage (not in the repo or on GitHub).
- **HTTPS:** The page uses `https://` for the HEC URL. Use a valid certificate on the Splunk side to avoid browser warnings.
- **Token:** Treat the HEC token as a secret; anyone with access to the same browser/profile can see it in the settings panel.

## Customization

- Edit `index.html` to change questions, choices, or field names; the script builds the `event` object from the form’s `name` and `value` attributes.
- You can add more questions by adding question blocks and including their values in the `answers` object in the submit handler.
