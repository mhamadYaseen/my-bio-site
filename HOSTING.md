# Host my bio site on GitHub Pages

Steps from the SSG book, Chapter 6 and Chapter 7. Final address: `https://mhamadyaseen.github.io/my-bio-site/` (check that `mhamadYaseen` is my GitHub username).

## 1. Copy the site out of the course repo

`my-bio-site/` is inside my `web-application` Git repo. Copy it somewhere else and work in the copy:

```bash
cp -R "/Users/muhammad/Desktop/seventh semester/web application/my-bio-site" ~/Desktop/my-bio-site
cd ~/Desktop/my-bio-site
```

## 2. Install GitHub CLI

```bash
brew install gh
```

## 3. Create the local Git repository

```bash
git init -b main
git config user.name "Muhammad Yaseen"
git config user.email "THE-EMAIL-I-WANT-PUBLIC"
git add .gitignore hugo.toml content layouts static
git commit -m "Save my Hugo bio site"
```

The email will be public. Use my GitHub `noreply` address, not my university one.

## 4. Create an empty repository on GitHub

In the browser: **New repository**, name `my-bio-site`, **Public**, with **no** README, **no** `.gitignore`, **no** licence.

## 5. Sign in and push

```bash
gh auth login --hostname github.com --git-protocol https --web --scopes workflow
gh auth setup-git
git remote add origin https://github.com/mhamadYaseen/my-bio-site.git
git push -u origin main
```

## 6. Turn on Pages

On GitHub: **Settings → Pages → Source → GitHub Actions**.

## 7. Set the address

In `hugo.toml`, change only `baseURL` (keep the final slash):

```toml
baseURL = 'https://mhamadyaseen.github.io/my-bio-site/'
```

Check it builds:

```bash
hugo --minify --panicOnWarning
```

## 8. Add the workflow

Create the file `.github/workflows/hugo.yaml` with this content (copied exactly from Chapter 7, section 7.4):

```yaml
name: Publish Hugo site

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-24.04
    env:
      HUGO_VERSION: "0.150.0"
    steps:
      - name: Check out the source
        uses: actions/checkout@v7

      - name: Read the Pages configuration
        id: pages
        uses: actions/configure-pages@v6

      - name: Install Hugo
        shell: bash
        run: |
          curl --fail --location --retry 3 \
            --output "$RUNNER_TEMP/hugo.tar.gz" \
            "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir -p "$RUNNER_TEMP/hugo-bin"
          tar -xzf "$RUNNER_TEMP/hugo.tar.gz" -C "$RUNNER_TEMP/hugo-bin" hugo
          echo "$RUNNER_TEMP/hugo-bin" >> "$GITHUB_PATH"

      - name: Build the website
        env:
          PAGES_BASE_URL: ${{ steps.pages.outputs.base_url }}
        run: hugo --minify --panicOnWarning --baseURL "${PAGES_BASE_URL}/"

      - name: Upload the generated website
        uses: actions/upload-pages-artifact@v5
        with:
          path: public

  deploy:
    needs: build
    runs-on: ubuntu-24.04
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Publish to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

Change `HUGO_VERSION: "0.150.0"` to `"0.167.0"` so it matches the Hugo on my Mac. That is my change, not the book's, and I did not test it online.

## 9. Publish

```bash
git add hugo.toml .github/workflows/hugo.yaml
git commit -m "Configure GitHub Pages publishing"
git push
```

On GitHub open **Actions → Publish Hugo site**. Wait until the **build** and **deploy** jobs both pass, then open the address shown under **Settings → Pages**.

## 10. Update later

```bash
git add content
git commit -m "Describe the change"
git push
```

Every push to `main` publishes.

## If it fails

Open the first failed step in the Actions run and read its message. A 404 usually means the deployment has not finished, so wait and use the address from **Settings → Pages**.
