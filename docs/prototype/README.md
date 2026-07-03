# Fil Poll — wallet voting prototype

A self-contained, zero-dependency prototype of the wallet-based voting flow for
Fil Poll / Metropolis. Open `index.html` in a browser (or publish this folder to
GitHub Pages) and walk the whole flow:

**Connect wallet → pick a voting profile (role) → optionally claim an address →
create a poll → vote → read results + AI insight.**

It reuses the theme from `client/src/theme.tsx` (kept in sync with
`docs/mocks/ui-preview.html`).

> This is a **front-end prototype for design/UX review**, not the production
> backend. Polls, votes, roles, claims, and sessions live in the browser
> (`localStorage`). No server, no database.

## What it demonstrates

1. **Wallet sign-in.** Uses an injected wallet (`window.ethereum`, e.g. MetaMask)
   when present — real `personal_sign` (EIP-191). If no extension is detected, a
   **demo wallet** generates a throwaway keypair in memory so the full flow is
   usable. Signatures only *prove control of an address*; they never move funds,
   approve tokens, or call contracts.
2. **Profiles / roles.** A voter picks the profile their vote counts under
   (Storage Provider, Allocator, Token Holder, Core Developer, Client, Community).
   Each role carries an illustrative weight. The profile can be changed **per
   poll** — you vote as different profiles on different topics.
3. **Claim an address (safe).** To "claim" an address you sign a one-time
   challenge **with that address**, proving control. A short-lived, revocable
   **delegated session** is then issued so you can vote from this device without
   the hardware wallet until it expires (12h in the prototype). This is the
   "travel without the hardware wallet" convenience — without letting anyone vote
   with an address they don't control.
4. **Polls + voting.** Sentiment checks (Positive/Negative) and Polis-style
   discussion polls (Agree/Disagree/Skip, add statements). Results are
   role-weighted.
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
  with `ecrecover`, bind address → role → session, enforce nonce/expiry, persist
  in Postgres.
- Replace the seeded ecosystem directory with a real, read-only registry of
  SP / Allocator / Token Holder addresses (on-chain + Fil+ data).
- Replace the on-device heuristic with a server-side Claude call over the same
  poll inputs (API key stays server-side; never shipped to the client).
- Derive role weights from verifiable on-chain signals rather than fixed
  constants.

## Notes

- Demo-mode addresses are derived from a P-256 public key (`0x` + last 20 bytes
  of `sha256(pubkey)`) purely so the claim signature can be verified in-browser
  without bundling a secp256k1 library. Real wallets use their genuine Ethereum
  address and secp256k1 signatures.
- Example directory addresses are illustrative and not real accounts.
