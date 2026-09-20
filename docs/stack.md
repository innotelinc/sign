# Sign Platform in the Innotel Platform Stack

**Role: DocumentOps (retired 2026-09-15)** — this platform served
`sign.innotel.us` and no longer does.

Sign Platform was the OpenSign fork that served `sign.innotel.us` while
**Signara** reached parity. It was a *stopgap*, never a second DocumentOps home:
Signara (`innotelinc/signara`) is the single e-signature product, it took over
the public surface on 2026-09-15, and it holds the storage role on ONYX. This
repository is frozen — see [ARCHIVE.md](../ARCHIVE.md) for the retirement
inventory and [CONVERGENCE.md](../CONVERGENCE.md) for the migration record.

> **Read the lists below as the historical declaration, not as a live posture.**
> Nothing in this repository is deployed. Signara owns DocumentOps.

This page declares Sign Platform's role in the
[**Innotel Platform Stack**](https://github.com/innotelinc/innotel-platform-stack) —
the canonical single-responsibility architecture. The stack is defined in exactly
one place; this page links the platform to it and states what it owns, consumes,
provides, and explicitly does not own.

## Owned (until 2026-09-15)

- The send / sign / complete envelope flow for `sign.innotel.us`
- The Parse Server + MongoDB deployment that backed it
- Local document storage (`opensign-files` volume) until the ONYX cutover
- Its own signing certificate (`PFX_BASE64` / `PASS_PHRASE`)

## Provided (until 2026-09-15)

- A working e-signature surface during the Signara migration window
- A fallback that stopped being reachable when its host went away — which is why
  the cutover was not run as a rollback-capable change (CONVERGENCE.md §2)

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
| Storage | ONYX holds Signara's documents; no `legacy/sign-platform/` prefix was ever populated — there was no source to migrate (Roadmap §6) |
| Source of truth | This repository's `docs/stack.md` points back to the Innotel Platform Stack |

## Retirement — done

[CONVERGENCE.md](../CONVERGENCE.md) drove this repository's retirement and it is
complete: Signara is the single e-signature product, it serves `sign.innotel.us`,
and its documents live in ONYX. The legacy history was never migrated — the only
copy sat on a host that no longer exists, and the recovery attempt found nothing
(Roadmap §6), so it is a recorded write-off rather than pending work. The
remaining step is administrative: freeze this repository per
[ARCHIVE.md](../ARCHIVE.md).

Back to the canonical definition: the
[Innotel Platform Stack](https://github.com/innotelinc/innotel-platform-stack).
