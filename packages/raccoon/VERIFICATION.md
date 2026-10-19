# Browser authorization loopback verification

## Problem and change

On Windows without a registered `office-raccoon://` handler, confirming browser authorization did not complete the Magpie sign-in. The original plugin used `method: "code"` and asked for a callback URL in a field the host labels Code. Pasting the original authorization page cannot provide the code, and the plugin also rejects a raw code.

The current official authorization page supports `desktop_callback=loopback`, a 64-character hex state and a `http://127.0.0.1:<port>/auth/callback` redirect. The fix uses that existing contract instead of claiming an official desktop protocol handler. The browser method now returns automatically; manual callback/token and read-only desktop sign-in remain available.

Protocol reference observed during this investigation:
https://sta-volc.xiaohuanxiong.com/raccoon-website/prod/mime-v1/_next/static/chunks/app/(identity)/authorize/page-f6a835419d6cdf21.js

The asset is versioned and may be replaced; the independent live authorization below also checked the contract.

## Verification performed

- The original automatic-return reproduction failed because the authorization URL had no loopback callback and the method was `code`.
- The revised full round trip passed with a local fake upstream: callback, state validation, code exchange and successful auth result. No real credential is used in these tests.
- `bun test packages/raccoon/`: 27 passed, 0 failed, including the original 21 tests for renewal, desktop session reuse, catalog, usage and check-in.
- `bun scripts/check.mjs raccoon`: passed.
- The final source was applied as an independent delta to upstream `47fbf63`, and the above checks passed again there.
- A separate live Windows check opened the vendor's real authorization page, confirmed the already signed-in account and received the real loopback. The revised plugin exchanged the code successfully, returned access and refresh values and a future expiry. The check printed only those boolean results. Credentials stayed in process memory, were not persisted, and were not included in this repository. No model request was made.

## Boundaries

The listener accepts only the expected Host, GET path and single matching state. Invalid callbacks do not consume the pending sign-in. A success page is sent only after successful exchange; pages contain no credential or code and use `Cache-Control: no-store` and `Referrer-Policy: no-referrer`.

The listener closes on success, authorization refusal, exchange failure or the ten-minute deadline. The OpenCode callback contract does not notify this flow when the host UI is closed, so an abandoned sign-in is bounded by that deadline, not immediate host cancellation. When a browser cannot reach the plugin host's loopback, the retained manual method is still needed.

The live check ran the revised plugin in isolation. It did not replace the user's installed plugin, test the daily Magpie UI's account persistence, publish npm, run inference or verify credits/rewards. No claim is made that the patch is merged, released or already loaded by the daily app.
