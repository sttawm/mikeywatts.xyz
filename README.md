# mikeywatts.xyz

Personal site — plain static HTML/CSS, hosted free on GitHub Pages at
[mikeywatts.xyz](https://mikeywatts.xyz).

## Structure

```
index.html          Home / landing (photo + bio)
research.html       ML Projects & Research
presentations.html  Paper Presentations
style.css           All styling (light + dark)
assets/             photo.jpg, favicon.svg
CNAME               Custom domain for GitHub Pages
404.html            Not-found page
```

The "CV" nav link points to an external Google Doc; there is no CV page.

## Editing

- **Text/links:** edit the relevant `.html` file and push. No build step.
- **Photo:** replace `assets/photo.jpg` (keep the same filename, or update the
  `src` in `index.html`).
- **CV:** edit the Google Doc URL in the `CV` nav link across the `.html` files.

## Preview locally

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

## Deploy

Pushing to `main` auto-deploys via GitHub Pages (Settings → Pages → Source:
`main` / root).

## Custom domain (mikeywatts.xyz)

At your DNS provider (Squarespace), set:

| Type  | Host | Value                  |
|-------|------|------------------------|
| A     | @    | 185.199.108.153        |
| A     | @    | 185.199.109.153        |
| A     | @    | 185.199.110.153        |
| A     | @    | 185.199.111.153        |
| CNAME | www  | sttawm.github.io.      |

Then in GitHub → Settings → Pages, set the custom domain to `mikeywatts.xyz`
and enable "Enforce HTTPS".
