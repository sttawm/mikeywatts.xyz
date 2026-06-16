# mikeywatts.xyz

Personal site — plain static HTML/CSS, hosted free on GitHub Pages at
[mikeywatts.xyz](https://mikeywatts.xyz).

## Structure

```
index.html        Home / landing
about.html        About Me
research.html     ML Projects & Research
cv.html           CV (links to assets/cv.pdf)
blog.html         Blog index
posts/            One HTML file per post
style.css         All styling (light + dark)
assets/           photo.svg, favicon.svg, cv.pdf
CNAME             Custom domain for GitHub Pages
404.html          Not-found page
```

## Editing

- **Text/links:** edit the relevant `.html` file and push. No build step.
- **Photo:** replace `assets/photo.svg` (or add a `.jpg`/`.png` and update the
  `src` in `index.html`).
- **CV:** drop your PDF at `assets/cv.pdf`.
- **New blog post:** copy `posts/hello-world.html`, rename it, edit the content,
  then add a matching `<li>` to `blog.html`.

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
