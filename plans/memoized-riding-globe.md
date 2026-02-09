# Self-host OneTrust `otSDKStub.js`

## Context

The OneTrust CMP stub script (`otSDKStub.js`) is currently loaded from `https://cdn.cookielaw.org/scripttemplates/otSDKStub.js`. Self-hosting it eliminates the DNS lookup + TLS handshake to cookielaw.org on cold loads, saving ~100-300ms on initial page render for all publishing client apps.

The full OneTrust SDK (`otBannerSdk.js`) will still be fetched from cookielaw.org by the stub — only the stub is self-hosted.

## Scope

Publishing clients only (excluding orbit.rms.rocks, console.rms.rocks, spotlight.rms.rocks):
- bilanz.ch, handelszeitung.ch, schweizer-illustrierte.ch, tele.ch, pme.ch, illustre.ch, glueckspost.ch, landliebe.ch, hzinsurance.ch, boersenspiel.cash.ch

## Approach

### 1. Download and store the stub script

- Create `libs/layout/vendor/otSDKStub.js` — single source of truth in the shared layout lib
- Download current version from `https://cdn.cookielaw.org/scripttemplates/otSDKStub.js`

### 2. Create a copy script to distribute to publishing apps

- Create `scripts/sync-onetrust-stub.sh` that:
  - Copies `libs/layout/vendor/otSDKStub.js` → each publishing client's `public/vendor/otSDKStub.js`
  - Can also re-download from CDN when called with `--update` flag (for periodic freshness)
- Add an NX `prebuild` script in root `package.json` or wire it into the existing build pipeline so the file is always in place before Astro builds

### 3. Update `OneTrust.astro`

**File:** `libs/layout/src/OneTrust.astro`

```diff
- const url = `https://cdn.cookielaw.org/scripttemplates/otSDKStub.js`;
+ const url = `/vendor/otSDKStub.js`;
```

### 4. Update `early-hints.txt.ts`

**Files:** `apps/*/src/pages/early-hints.txt.ts` (bilanz.ch, boersenspiel.cash.ch, and any others)

Change from preloading the external script to preconnecting to cookielaw.org (still needed for the full SDK the stub fetches):

```diff
- const hints = ['<https://cdn.cookielaw.org/scripttemplates/otSDKStub.js>;rel=preload;as=script'];
+ const hints = ['<https://cdn.cookielaw.org>;rel=preconnect'];
```

### 5. Update test snapshot

**File:** `libs/layout/src/__snapshots__/OneTrust.test.ts.snap`

The snapshot will need updating since the `src` URL changes. Run:
```
nx run @dtc/layout:test:unit --update
```

### 6. Git-ignore the copied files

Add to `.gitignore`:
```
apps/*/public/vendor/otSDKStub.js
```

The canonical source stays in `libs/layout/vendor/otSDKStub.js` (committed). The copies in each app's `public/vendor/` are build artifacts.

## Files to modify/create

| Action | File |
|--------|------|
| Create | `libs/layout/vendor/otSDKStub.js` (downloaded stub) |
| Create | `scripts/sync-onetrust-stub.sh` |
| Edit   | `libs/layout/src/OneTrust.astro` (change URL) |
| Edit   | `apps/bilanz.ch/src/pages/early-hints.txt.ts` |
| Edit   | `apps/boersenspiel.cash.ch/src/pages/early-hints.txt.ts` |
| Edit   | `apps/orbit.rms.rocks/src/pages/early-hints.txt.ts` (if kept) |
| Edit   | `.gitignore` |
| Update | `libs/layout/src/__snapshots__/OneTrust.test.ts.snap` (via test run) |
| Edit   | `package.json` (add sync script) |

## Verification

1. Run `scripts/sync-onetrust-stub.sh` — confirm files land in each app's `public/vendor/`
2. Run `nx run @dtc/layout:test:unit --update` — snapshot updates cleanly
3. Build one app: `nx run bilanz.ch:build` — confirm `/vendor/otSDKStub.js` is in the output
4. Dev server: `nx run bilanz.ch:dev` — verify the CMP banner still loads, check Network tab shows stub loaded from localhost and full SDK from cookielaw.org
