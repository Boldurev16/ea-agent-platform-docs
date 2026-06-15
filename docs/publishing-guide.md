# Publishing guide (GitHub Pages)

Repository: **ea-agent-platform-docs**  
Intended owner: **Boldurev16**

## Prerequisites

- This repo contains **documentation only** (no application code).
- Entry point: **`docs/index.md`**
- Jekyll config: **`docs/_config.yml`**

## Steps

### 1. Create GitHub repository

```bash
# From local ea-agent-platform-docs folder
git init
git add .
git commit -m "Initial public documentation publication"
git branch -M main
git remote add origin https://github.com/Boldurev16/ea-agent-platform-docs.git
git push -u origin main
```

### 2. Enable Pages

1. GitHub → repository **Settings** → **Pages**.
2. **Build and deployment**
   - Source: **Deploy from a branch**
   - Branch: `main`
   - Folder: **`/docs`**
3. Save.

Published URL (default):

`https://boldurev16.github.io/ea-agent-platform-docs/`

### 3. Verify

- Open `/` — should render `docs/index.md`.
- Check links under `/navigation/`, `/overview/`, `/legends/`.
- Mermaid blocks render on GitHub; Jekyll theme may vary — test diagram pages.

### 4. Updates

After editing markdown in `docs/`:

```bash
git add docs/
git commit -m "docs: update <topic>"
git push
```

Pages rebuilds within ~1–3 minutes.

## Custom domain (optional)

Settings → Pages → Custom domain → configure DNS CNAME to `boldurev16.github.io`.

## Security checklist before each release

- [ ] No `.env` or credential files added
- [ ] No internal hostnames or private IPs in new content
- [ ] Review [`publishing-sanitization-report.md`](publishing-sanitization-report.md)
- [ ] No copy-paste of production prompts or business rules

[← Documentation home](index.md)
