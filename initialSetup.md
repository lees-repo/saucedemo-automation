# Playwright initial setup

Step-by-step guide for this repo (`saucedemo-automation`). Playwright lives at the **project root**—no extra subfolders.

---

## What gets installed

| Item | How | Purpose |
|------|-----|---------|
| **Node.js** | You install separately | Runs npm and Playwright |
| **npm packages** | `npm install` | `@playwright/test`, `@types/node` (see `package.json`) |
| **Browsers** | `npx playwright install` | Chromium (and optionally Firefox/WebKit) used to run tests |

`package.json` does **not** download browsers. You must run `npx playwright install` after `npm install`.

---

## Step 1 — Install Node.js

1. Check if Node is installed:

   ```bash
   node -v
   npm -v
   ```

2. You need **Node.js 18 or newer** (LTS recommended).

3. If the commands fail, install Node from [https://nodejs.org/](https://nodejs.org/) and open a **new** terminal.

---

## Step 2 — Open the project folder

In your terminal, go to the repo root (the folder that contains `package.json`):

```bash
cd /path/to/saucedemo-automation
```

Confirm you are in the right place:

```bash
ls package.json playwright.config.ts
```

Both files should be listed.

---

## Step 3 — Install npm dependencies

This reads `package.json` and installs everything into `node_modules/`:

```bash
npm install
```

**What this installs (devDependencies):**

- `@playwright/test` — Playwright Test runner and APIs
- `@types/node` — TypeScript types for Node (used by config if you add TS tooling later)

**Expected result:** A `node_modules/` folder appears. No errors at the end of the command.

**If you cloned the repo fresh:** `npm install` is enough. You do not need `npm init`.

**Clean reinstall (optional):**

```bash
rm -rf node_modules package-lock.json
npm install
```

---

## Step 4 — Install Playwright browsers

Browsers are **not** included in `npm install`. Download them separately:

```bash
npx playwright install
```

This downloads Chromium, Firefox, and WebKit. The project is configured to use **Chromium** only, so you can install just that (smaller download):

```bash
npx playwright install chromium
```

**Expected result:** Playwright prints download progress and finishes without errors.

**Run this in your own terminal** (Terminal.app, iTerm, VS Code integrated terminal). If an IDE sandbox sets `PLAYWRIGHT_BROWSERS_PATH` to a temp folder, browsers may be missing or the wrong CPU architecture—see [Troubleshooting](#troubleshooting).

**Optional — system dependencies (Linux CI only):**

```bash
npx playwright install-deps chromium
```

Not required on macOS/Windows for local runs.

---

## Step 5 — Verify the setup

Run the existing sample test:

```bash
npm test
```

**Expected result:** Something like `1 passed` and exit code 0.

Open the HTML report after a run:

```bash
npm run test:report
```

---

## Project layout

```text
saucedemo-automation/
├── package.json           # npm scripts and devDependencies
├── package-lock.json      # locked versions (after npm install)
├── playwright.config.ts   # base URL, browser, reporters
├── tests/                 # all test files (*.spec.ts)
│   └── login.spec.ts      # example test
├── initialSetup.md        # this file
├── .gitignore
├── node_modules/          # created by npm install (do not commit)
├── test-results/          # created when tests run (do not commit)
└── playwright-report/     # HTML report (do not commit)
```

---

## Step 6 — Add a new test

### 6.1 Create a spec file

1. Under `tests/`, create a new file. Name it after the feature, ending in `.spec.ts`:

   ```text
   tests/cart.spec.ts
   ```

2. Use this starter template (adjust steps and assertions for your scenario):

   ```ts
   import { test, expect } from '@playwright/test';

   test.describe('cart', () => {
     test('user can add an item to the cart', async ({ page }) => {
       await page.goto('/');
       await page.getByPlaceholder('Username').fill('standard_user');
       await page.getByPlaceholder('Password').fill('secret_sauce');
       await page.getByRole('button', { name: 'Login' }).click();

       await page.getByRole('button', { name: 'Add to cart' }).first().click();
       await expect(page.locator('.shopping_cart_link')).toHaveText('1');
     });
   });
   ```

3. Save the file.

**Rules:**

- Files must live under `tests/` and match `*.spec.ts` (or `*.spec.js`).
- Import `test` and `expect` from `@playwright/test`.
- Use `page.goto('/')` for paths relative to `baseURL` in `playwright.config.ts` (`https://www.saucedemo.com`).

### 6.2 (Optional) Run while writing — UI mode

Interactive runner to pick tests and watch the browser:

```bash
npm run test:ui
```

---

## Step 7 — Run your new test

After saving `tests/cart.spec.ts` (or any new file), use one of the options below.

### Run only that file

```bash
npx playwright test tests/cart.spec.ts
```

### Run only tests whose title matches a phrase

```bash
npx playwright test -g "add an item to the cart"
```

### Run one file in headed mode (see the browser)

```bash
npx playwright test tests/cart.spec.ts --headed
```

Or use the npm script for all tests headed:

```bash
npm run test:headed
```

### Run one file with the Playwright UI

```bash
npx playwright test tests/cart.spec.ts --ui
```

### Run everything in the suite

```bash
npm test
```

Same as:

```bash
npx playwright test
```

### Debug a failing test

```bash
npx playwright test tests/cart.spec.ts --debug
```

### See a list of tests without running them

```bash
npx playwright test --list
```

---

## Quick reference — commands

| Goal | Command |
|------|---------|
| Install npm packages | `npm install` |
| Install browsers | `npx playwright install` or `npx playwright install chromium` |
| Run all tests | `npm test` |
| Run one file | `npx playwright test tests/your-file.spec.ts` |
| Run by test name | `npx playwright test -g "part of title"` |
| Headed browser | `npm run test:headed` or add `--headed` |
| UI mode | `npm run test:ui` |
| Open last report | `npm run test:report` |

---

## Configuration

- **Site under test:** [Sauce Demo](https://www.saucedemo.com)
- **baseURL:** `playwright.config.ts` → `use.baseURL`
- **Demo credentials:** `standard_user` / `secret_sauce`

To target another environment, change `baseURL` in `playwright.config.ts`.

---

## CI (GitHub Actions / similar)

```bash
npm ci
npx playwright install --with-deps chromium
npm test
```

Set `CI=true` in the pipeline so retries and `forbidOnly` apply (see `playwright.config.ts`).

Upload `playwright-report/` as a build artifact if you want HTML reports in CI.

---

## Troubleshooting

| Issue | What to try |
|-------|-------------|
| `command not found: npm` | Install Node.js and restart the terminal |
| `Cannot find module '@playwright/test'` | Run `npm install` from the project root |
| Browsers missing / executable doesn't exist | Run `npx playwright install chromium` in your **local** terminal |
| Wrong-arch browser (x64 vs arm64) | `unset PLAYWRIGHT_BROWSERS_PATH`, then `npx playwright install chromium` |
| No tests found | File must be under `tests/` and named `*.spec.ts` |
| Tests time out | `test.setTimeout(60000)` in the spec or increase timeout in `playwright.config.ts` |
| Wrong site | Check `use.baseURL` in `playwright.config.ts` |

---

## Docs

- [Playwright intro](https://playwright.dev/docs/intro)
- [Writing tests](https://playwright.dev/docs/writing-tests)
- [Running tests](https://playwright.dev/docs/running-tests)
- [Test configuration](https://playwright.dev/docs/test-configuration)
