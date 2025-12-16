
name: "ContentForge AI Tailwind Build & Deploy"

# Trigger on every push to main
on:
  push:
    branches:
      - main

concurrency:
  group: contentforge-tailwind-build
  cancel-in-progress: true

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build-and-deploy:
    name: Build Tailwind CSS and Deploy to gh-pages
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Use Node.js 20.x
        uses: actions/setup-node@v4
        with:
          node-version: "20.x"
          cache: "npm"

      - name: Cache node modules (optional)
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json', '**/yarn.lock', '**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        run: npm install

      - name: Build CSS (Tailwind)
        run: npm run build:css

      - name: Verify dist output
        run: |
          echo "Listing dist/ contents:"
          ls -la dist || true

      - name: Deploy to gh-pages branch
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          publish_branch: gh-pages
          # Optional: set committer details
          user_name: "github-actions[bot]"
          user_email: "41898282+github-actions[bot]@users.noreply.github.com"
