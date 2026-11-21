# Repro: `requestIdleCallback is not a function` in Safari (Storybook 11.0.0-beta.3)

An **unmodified** Storybook sandbox (`npx storybook@11.0.0-beta.3 sandbox react-vite/default-ts`). Nothing was changed except this README.

## Steps

```sh
yarn
yarn storybook
```

1. Open http://localhost:6006/?path=/story/example-button--primary in **Safari** (or Playwright WebKit).
2. Select the **Accessibility** addon panel.

## Expected

The scan completes and lists results, as it does in Chrome:
"Violations 0 · Passes 6 · Inconclusive 0 — No accessibility violations found."

## Actual

- The panel stays on **"Accessibility scan in progress — Please wait while the accessibility scan is running..."** indefinitely.
- The console logs, each time a story finishes rendering:

  ```
  TypeError: scope.requestIdleCallback is not a function.
  (In 'scope.requestIdleCallback(() => { const { storyId, index, refId } = store.getState(); ...')
  ```

Safari doesn't implement `requestIdleCallback`; the manager's `STORY_FINISHED` handler (sibling preloading, added in storybookjs/storybook#37389) calls it without a feature check:
`code/core/src/manager-api/modules/stories.ts`.

## Verified with

| Browser | Result |
| --- | --- |
| Safari 27.2 (macOS 27.2.1) | ❌ scan never completes, TypeError |
| Playwright WebKit 27.2 | ❌ scan never completes, TypeError |
| Playwright Chromium 159 | ✅ scan completes, no errors |
