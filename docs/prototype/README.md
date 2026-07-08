# Fil Poll — wallet voting prototype

A self-contained, zero-dependency prototype of the wallet-based voting flow for
Fil Poll / Metropolis. Open `index.html` in a browser (or publish this folder to
GitHub Pages) and walk the whole flow:

**Connect wallet (browser or hardware) → verify your address → pick a voting
profile (role) → create a poll → vote → read results + AI insight.**

It reuses the theme from `client/src/theme.tsx` (kept in sync with
`docs/mocks/ui-preview.html`).

> This is a **front-end prototype for design/UX review**, not the production
> backend. Polls, votes, roles, claims, and sessions live in the browser
> (`localStorage`). No server, no database.

## What it demonstrates

1. **Wallet sign-in (any wallet, including hardware).** Resolves an injected
   provider at click time via **EIP-6963** (falling back to `window.ethereum` /
   the multi-wallet `providers[]` array), so MetaMask, Rabby, Coinbase Wallet,
   etc. all work — real `personal_sign` (EIP-191). Hardware / cold wallets
   (Ledger, Trezor) are offered as a first-class option. A **demo wallet** (an
   in-memory keypair) lets anyone walk the flow with no extension. Signatures
   only *prove control of an address*; they never move funds, approve tokens, or
   call contracts. *(In this prototype the browser-wallet path is real; the
   hardware path is simulated so the flow can be demonstrated.)*
2. **Verify your address — first step.** You sign a one-time challenge **with
   your address**, proving control. A short-lived, revocable **delegated
   session** is then issued so you can vote from this device without the hardware
   wallet until it expires (12h in the prototype) — the "travel without the
   device" convenience, without letting anyone vote with an address they don't
   control. This step comes *before* profile selection so hardware users sign
   once, up front.
3. **Profiles / roles.** A voter picks the profile their vote counts under
   (Storage Provider, Allocator, Token Holder, Core Developer, Client, Community).
   The profile can be changed **per poll** — you vote as different profiles on
   different topics. Voting is **unweighted**: every vote counts equally.
4. **Polls + voting.** Sentiment checks (Positive/Negative) and Polis-style
   discussion polls (Agree/Disagree/Skip, add statements). Results are plain
   counts, with a per-profile breakdown shown for context.
5. **AI insight.** Each report/sentiment view includes a "Quick sentiment
   overview" generated **on-device** (no network, no API key) from the current
   results.

## Security model (for review)

- **Claiming is signature-gated, always.** The ecosystem directory is for
  *discovery only* — being listed grants no voting power. Authority comes from a
  signature that recovers to the claimed address. The prototype records the exact
  challenge + signature payload; in production the **server verifies the signature
  (ecrecover) before minting a session** — the client is never trusted to
  self-certify.
- **No key material is ever persisted, logged, or displayed.** The demo wallet's
  key is a non-extractable Web Crypto `CryptoKey` held only for the tab's
  lifetime.
- **Sessions are short-lived and revocable.** Expiry + one active session per
  address + a Revoke button.
- **The challenge message is explicit** that signing does not authorize any
  transaction, transfer, or contract call.
- **AI runs locally.** No poll data leaves the browser. The production path runs
  the same summary server-side.

## Path to production (not in this prototype)

- Move signature verification server-side (`server/`): verify EIP-191 signatures
  with `ecrecover`, bind address → session, enforce nonce/expiry, persist in
  Postgres.
- Implement the real hardware / cold-wallet connectors (WebHID / Ledger Live /
  WalletConnect) in place of the simulated hardware path.
- Replace the seeded ecosystem directory with a real, read-only registry of
  SP / Allocator / Token Holder addresses (on-chain + Fil+ data).
- Replace the on-device heuristic with a server-side Claude call over the same
  poll inputs (API key stays server-side; never shipped to the client).

## Notes

- Demo- and simulated-hardware addresses are derived from a P-256 public key
  (`0x` + last 20 bytes of `sha256(pubkey)`) purely so the signature can be
  verified in-browser without bundling a secp256k1 library. Real wallets use
  their genuine Ethereum address and secp256k1 signatures.
- Example directory addresses are illustrative and not real accounts.
