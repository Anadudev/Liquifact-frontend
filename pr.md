Closes #1368

### Summary of Changes

This pull request implements the investor exit flows for the Soroban escrow contract, exposing two entrypoints previously unavailable in the UI:
1. **`refund(investor)`**: Allows investors to reclaim their contribution when an escrow is cancelled (status `4`).
2. **`unfund(investor, amount)`**: Allows investors to withdraw part or all of their contribution while the escrow is still open (status `0`).

### Implementation Details

1. **Pure Decision Logic & Stroop Math**:
   - Implemented [`lib/invest/deriveExitUi.js`](https://github.com/Liquifact/Liquifact-frontend/blob/main/lib/invest/deriveExitUi.js) to derive UI state (`refund`, `unfund`, or `none`) directly from on-chain escrow states. Handles legal hold, pause, settled, and already-refunded states cleanly.
   - Provides safe integer arithmetic (`parseStroops`, `formatStroops`) using native `BigInt` (7 decimal places / 10,000,000 stroops per unit), preventing floating-point precision loss.

2. **Stellar RPC Helpers**:
   - Added [`lib/stellar/rpc.js`](https://github.com/Liquifact/Liquifact-frontend/blob/main/lib/stellar/rpc.js) wrapping `@stellar/stellar-sdk` and `@stellar/freighter-api` to simulate read calls (`get_contribution`, `is_investor_refunded`, `get_legal_hold`, `is_paused`, `get_escrow`) and handle transaction assembly, simulation, signing, and submission for mutating calls.
   - Decodes Soroban simulation errors into human-readable messages (e.g., error codes `142`, `143`, `220`, `221`, `222`).

3. **Investor Exit Panel Component**:
   - Added [`components/invest/ExitPanel.jsx`](https://github.com/Liquifact/Liquifact-frontend/blob/main/components/invest/ExitPanel.jsx) mounted in [`app/marketplace/[id]/InvoiceDetail.jsx`](https://github.com/Liquifact/Liquifact-frontend/blob/main/app/marketplace/[id]/InvoiceDetail.jsx) for connected Freighter wallets on on-chain invoices.
   - **Refund UI**: Displays the refundable amount with a "Request refund" button that simulates, signs, and re-reads state upon success.
   - **Unfund UI**: Displays an amount input with a "Max" shortcut button, inline validation against current contribution, a confirmation modal dialog (`<dialog>` with focus trap and Escape handler), and step-by-step progress indicator ("Simulating…", "Waiting for signature…", "Confirming on-chain…").
   - Disables submit buttons while transactions are in-flight to prevent duplicate submissions.

4. **Marketplace Discovery & Demo Data**:
   - Added `cancelled` filter tab to [`app/marketplace/Marketplace.jsx`](https://github.com/Liquifact/Liquifact-frontend/blob/main/app/marketplace/Marketplace.jsx) so investors can easily locate cancelled escrows and claim refunds.
   - Added a demo cancelled invoice in [`lib/demo.js`](https://github.com/Liquifact/Liquifact-frontend/blob/main/lib/demo.js) for offline fallback preview.

5. **Styling & Accessibility**:
   - Added styles and responsive layout (mobile-optimized down to 375px) in [`app/globals.css`](https://github.com/Liquifact/Liquifact-frontend/blob/main/app/globals.css).
   - Fully accessible with ARIA live regions (`role="status"`, `aria-live="polite"`), modal dialogs (`role="dialog"`), error alerts (`role="alert"`), and keyboard support.

6. **Test Suites**:
   - Pure logic tests in [`tests/invest/deriveExitUi.test.mjs`](https://github.com/Liquifact/Liquifact-frontend/blob/main/tests/invest/deriveExitUi.test.mjs) (25 tests via `node --test`).
   - Component & interaction tests in [`tests/invest/ExitPanel.test.jsx`](https://github.com/Liquifact/Liquifact-frontend/blob/main/tests/invest/ExitPanel.test.jsx) (15 tests via Vitest).
   - Added `npm run test:ui` step to [`.github/workflows/ci.yml`](https://github.com/Liquifact/Liquifact-frontend/blob/main/.github/workflows/ci.yml).

### Verification
- `node --test`: 25 passing
- `npm run test:ui`: 15 passing
- `npm run build`: Successfully built with Turbopack (Next.js 16)
