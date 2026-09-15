# Sign Platform in the Innotel Platform Stack

**Role: DocumentOps (stopgap)** — self-hosted e-signature at `sign.innotel.us`.

Sign Platform is the live OpenSign fork serving `sign.innotel.us` while
**Signara** reaches parity. It is a *stopgap*, not a second DocumentOps home:
the strategic platform is Signara (`innotelinc/signara`), which becomes the
single e-signature product and takes over this repo's storage role on ONYX. The
retirement path is tracked in [CONVERGENCE.md](../CONVERGENCE.md).

This page declares Sign Platform's role in the
[**Innotel Platform Stack**](https://github.com/innotelinc/innotel-platform-stack) —
the canonical single-responsibility architecture. The stack is defined in exactly
one place; this page links the platform to it and states what it owns, consumes,
provides, and explicitly does not own.

## Owns

- The send / sign / complete envelope flow for `sign.innotel.us`
- The Parse Server + MongoDB deployment that backs it
- Local document storage (`opensign-files` volume) until the ONYX cutover
- Its own signing certificate (`PFX_BASE64` / `PASS_PHRASE`)

## Provides

- A working e-signature surface during the Signara migration window
- A stable, restorable fallback while Signara reaches feature parity

## Consumes

- Authentik — identity and SSO (Cerulean, IdentityOps)
- Cerulean Vault — secrets (SecretOps)
- Cerulean — DNS + TLS for `sign.innotel.us`
- NPM Edge — public routing and TLS termination
- ONYX — target storage for migrated legacy documents

## Explicitly does NOT own

- Identity (Authentik)
- Secrets (Cerulean Vault)
- Certificates / DNS / TLS (Cerulean)
- Long-term storage (ONYX)
- The ecosystem's e-signature future — Signara owns DocumentOps

## Service map (Sign Platform-owned)

| Component | Technology | Job |
| --- | --- | --- |
| Web client | OpenSign React client (built from `apps/`) | Envelope UI, signing flow |
| API | OpenSign Parse Server (Node 22) | Envelope, signer, audit APIs on `/api/app` |
| State store | MongoDB 7 | Documents, signers, audit trail |
| Document storage | `opensign-files` volume (`USE_LOCAL=true`) | Original and signed PDFs (moves to ONYX) |
| Edge | External Nginx Proxy Manager | TLS termination and routing to ports 3000 / 8080 |

## In the ecosystem

| Flow | Path |
| --- | --- |
| Identity | Cerulean's Authentik → OIDC → Sign Platform sessions |
| Secrets | Cerulean Vault (SecretOps, KV v2) → references in `.env.prod`; never committed |
| Trust | Cerulean issues DNS + per-zone wildcard TLS; NPM Edge fronts `sign.innotel.us` |
| Revenue | Magnate plans/entitlements gate paid signing seats (optional) |
| Storage | ONYX receives the migrated documents under `legacy/sign-platform/` |
| Source of truth | This repository's `docs/stack.md` points back to the Innotel Platform Stack |

## Retirement

[CONVERGENCE.md](../CONVERGENCE.md) drives this repository's retirement: Signara
becomes the single e-signature product, documents migrate to ONYX under
`legacy/sign-platform/`, and `sign.innotel.us` moves to Signara. Until then this
stack stays conformant so it remains a clean, auditable fallback.

Back to the canonical definition: the
[Innotel Platform Stack](https://github.com/innotelinc/innotel-platform-stack).
