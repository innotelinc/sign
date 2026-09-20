# Sign Platform — archive inventory

**Status:** P5 work item — opened 2026-09-19, frozen 2026-09-20
**Owner:** Innotel
**Repo:** `innotelinc/sign` (renamed from `sign-platform`)
**Companion docs:** [CONVERGENCE.md](CONVERGENCE.md) (the migration plan and its
retirement record), `1-primary/signara/docs/Roadmap.md` §P5 (what happens next),
`signara/docs/OpenSignParity.md` (the parity reference harvested before this
freeze)

This is the inventory for retiring this repository: what was worth carrying out
of it, what is already duplicated elsewhere in the estate, what must never be
deployed again, and what remains to be switched off. It exists so the archive is
auditable — a reader should be able to tell what was deliberately left behind
rather than lost.

**Convergence is here, not there.** This repo is absorbed *into* Signara and then
frozen; Signara is not merged back into this one. There is no branch of the estate
where the direction reverses: the only live data (Postgres + Onyx) is on Signara's
side, and this stack's host no longer answers (CONVERGENCE.md §2, Roadmap §6). No
git histories are merged either — this repository stays as the historical record.

---

## 1. Harvest — carried out before the freeze

| Asset | Why it mattered | Destination | Status |
| --- | --- | --- | --- |
| UI translations, 7 locales (`de, en, es, fr, hi, it, kr`) | The only human-translated e-signature vocabulary in the estate; the rest of the stack had no locales at all | `signara/apps/web/public/locales/` | **Done** 2026-09-19 — copied byte-identical (sha256 verified per locale), wired through Signara's i18n runtime, and pinned by `catalogs.spec.ts` |
| OpenSign's field/widget vocabulary | The catalogs encode the field types a signer expects (`widgets-name.*`) — the reference for Signara's field-placement editor | `signara/docs/OpenSignParity.md` §1, tracked by issue #81 | **Done** 2026-09-20 — the 18 field types plus the behaviours their strings imply (AcroForm detection, prefill, conditional logic, formulas, per-signer name uniqueness, bulk value validation) |
| Feature inventory of the old product (bulk send, scheduled reminders, i18n, in-person signing, cloud imports, SMS/WhatsApp) | Defines "did we lose a capability a user still expects?" | Signara issues #81–#92 (Roadmap §W2) | Tracked — all twelve open as of 2026-09-20 |
| Data mapping tables and storage reasoning (CONVERGENCE.md §4–§5) | The only written record of how the old schema mapped to Signara's, and why storage moved to Onyx | Stays here, cited by the roadmap | Retained — historical record |
| `sign.innotel.us` surface behaviour: completion email identity, certificate layout, branding strings | Users saw these; a regression in them is a user-visible change | `signara/docs/OpenSignParity.md` §2–§3, tracked by issues #83 and #84 | **Done** 2026-09-20 — both mail bodies and subjects verbatim, the tenant-override rule, and the certificate's page order and labels |

**Deliberately not harvested.** The Parse Server / OpenSign signing implementation
(`apps/OpenSignServer/cloud/parsefunction/pdf/PDF.js`, PFX signing, `/ByteRange`
handling) was *not* ported: Signara has its own `certificates` module and signing
path, and a second implementation of PDF signing is a liability rather than an
asset. The parity rows cover the *behaviour* to match; the code stays here.

## 2. Verified duplicates — nothing unique is lost by archiving

The estate tooling in this repo is byte-identical to copies in the repositories
that stay live, so archiving `sign` removes no capability (sha256 prefixes shown
so a future reader can re-check without trusting this table):

| File | sha256 (12) | Also in |
| --- | --- | --- |
| `scripts/mesh.sh` | `07fdb1526e1b` | `distro`, `onyx`, `signara` |
| `scripts/stack-lib.sh` | `92c544316cad` | `distro`, `signara` |
| `scripts/secret-scan.py` | `11746905d257` | `distro`, `onyx`, `signara`, `wintrain` |
| `.githooks/secret-scan.sh` | `9c42820f1fb2` | `distro`, `onyx`, `signara`, `wintrain` |
| `.githooks/guard-lib` | `7f13886e6056` | `distro`, `onyx`, `signara`, `wintrain` |
| `.github/workflows/attribution-guard.yml` | `fa97734cb8a4` | `distro`, `olympus`, `onyx`, `signara`, `wintrain` |

**Verified 2026-09-20 against each repository's `origin/main`**, not against the
working trees on this host — those are checkouts and several are behind their
remote, which is how a first pass at this table recorded a stale
`secret-scan.py` hash (`0b92b219fe4c`) and made this repo's copy look newer than
the live ones when the two were in fact identical at every tip. The tip being
frozen is `37d23806` (`secret-scan: stop reading a credential out of a literal, a
path, or a run-time value`), which is also the subject of the newest commit in
`distro`, `onyx`, `signara` and `wintrain`: the estate's tooling was mirrored
everywhere before this freeze, so nothing is stranded here.

If any of these drift in a live repo, the live copy is authoritative — this repo's
copy is a snapshot and must not be used as the source.

## 3. No value after the freeze

- `apps/OpenSign/`, `apps/OpenSignServer/` — an OpenSign fork. Upstream
  (`opensignlabs/opensign`, AGPL-3.0) is canonical; re-fork from there if
  OpenSign is ever needed again, never from this tree.
- `web/landing/index.html`, `opensign-settings.txt`, `NGINX_PROXY_MANAGER.md`,
  `INSTALLATION.md`, `USAGE.md` — deployment and brand material for a stack whose
  host is gone.
- `docker-compose.yml`, `Makefile`, `.env.example`, `.env.local_dev` — deploy
  configuration for that same dead host.
- `.github/dependabot.yml` — **removed 2026-09-20.** It asked for weekly npm
  updates across the three OpenSign trees, which on a frozen repository means
  pull requests proposing upgrades to a stack that must never be run again, and
  one of its three `directory:` entries (`microfrontends/SignDocuments`) did not
  exist in this tree at all, so that job failed every week. Closing the PRs
  alone does not settle it: they were closed while the repository was briefly
  unarchived to correct its metadata, and Dependabot opened fifteen replacements
  in two waves (#21–#25 within a minute, #26–#30 before the config change took
  effect) faster than they could be closed, which is what made removing the
  config the fix rather than a tidy-up.

## 4. Secrets and keys checked (2026-09-19)

Checked before freezing, so the archive carries no live credential:

- **`cert/local_dev.crt` / `.key` / `.pfx` / `local-dev_base64_pfx`** — a
  self-signed **OpenSign demo** key, not ours: `O=OpenSign Labs`, `CN=OpenSign`,
  valid 2023-11-17 → **expired 2024-11-16**. The base64 file decodes to the same
  PFX as the binary (`08e86618106475bf`). Nothing of Innotel's to rotate; it must
  not be presented as a production signing certificate.
- **`.env.local_dev`** — placeholder values only. `SMTP_PASS` is a placeholder
  word plus an instruction comment, and `MASTER_KEY`, `PASS_PHRASE` and
  `PFX_BASE64` are demo values. No live credential was found. The file stays as
  history and must never be copied into a new deployment.
- **`.env.prod`** — not in the tree (gitignored, as intended).

## 5. Freeze steps

1. Land the status banners — `README.md`, `docs/stack.md`, `web/landing/index.html`
   (done with this inventory). **Done 2026-09-20** — landed in the freeze commit
   together with this file.
2. Tag the frozen state `sign-platform-final` so it is nameable. **Done 2026-09-20,
   locally only** — the tag marks the tip that carries the retirement documents,
   which is the state a reader arrives at; the last commit to carry *deployable*
   code (the state the stack actually ran) is `45c2efa5` (2026-09-08,
   `apps/OpenSignServer/Utils.js`, `cloudServerUrl` derived from `SERVER_URL`),
   and that commit is an ancestor of the tag. Push the tag with the archive.
3. Archive the repository on GitHub (`innotelinc/sign` → *Archive repository*),
   making it read-only. **Do not delete it** — it is the only record of the
   convergence, and the mapping tables are still cited.
4. Stopped `.github/workflows/Docker.yml`: it builds and publishes OpenSign images
   that must not be pulled or re-published. **Done 2026-09-20** — it was not
   dormant: `ghcr.io/innotelinc/opensign` and `…/opensignserver` were republished
   on 2026-09-20 00:37 and 00:47 UTC, i.e. after this repo was documented as
   frozen. The push trigger is removed (manual dispatch only), so a stray commit
   cannot rebuild them; the workflow stays in the tree as the record. **The two
   packages were then deleted** (2026-09-20, §6): nothing in the estate pulls
   them, and the deployment host holds no image or container from this stack.
   The repository's other publishing workflow, `.github/workflows/pages.yml`
   (the landing page), is gated the same way for a different reason: it has never
   succeeded here — Pages was never enabled for this repository, so every run
   since 2026-09-15 failed in "Configure Pages" with `Resource not accessible by
   integration` — and a frozen repository should not go red on a deploy it cannot
   perform. Nothing links to that site (§6).
5. Confirm nothing in the estate links here except history: this inventory, the
   roadmap's P5 row, and `CONVERGENCE.md` itself. **Done 2026-09-20** — an
   estate-wide search of `distro`, `olympus`, `onyx` and `wintrain` finds no
   reference to `sign-platform` or `OpenSign` at all; inside `signara` they appear
   only in the retirement record (`docs/Roadmap.md`). Detail in §7.

## 6. Estate-side decommission checklist

| Item | Expected state | Verified |
| --- | --- | --- |
| `sign.innotel.us` | Serves Signara; the legacy `/api/` location pointing at the Parse host is gone | 2026-09-15 (Roadmap §W5) |
| `sign-platform*` containers, volumes, images | None exist; the host no longer answers on ICMP or `:22/:3000/:8080/:27017` | 2026-09-15 (Roadmap §6) |
| Legacy `mongodump` archive / `opensign-files` volume | None exist anywhere this estate can reach — written off | 2026-09-15 (Roadmap §6) |
| Any `*.sign-platform` DNS record or NPM proxy host | Remove if one is found; only the `sign.innotel.us` → Signara host should remain | **Closed 2026-09-20** — `sign-platform.innotel.us` and `opensign.innotel.us` do not resolve. The one stale record, `api.sign.innotel.us` (a CNAME to the apex: a name with no certificate and nothing behind it), was **removed through Cerulean's DNS API** and re-checked: it no longer resolves from the estate or from public resolvers, while `sign.innotel.us` still CNAMEs to the apex and serves Signara. The surrounding records are healthy — `auth`, `api`, `app`, `admin` and `storage.signara.innotel.us` resolve to the estate WAN address and are covered by the issued `*.innotel.us` wildcard (expires 2026-12-14) |
| Published OpenSign images from this repo's CI (`Docker.yml`) | Do not pull; no longer republished | **Gated 2026-09-20** (§5.4) and **deleted 2026-09-20**: `innotelinc/opensign` and `innotelinc/opensignserver` were removed from GHCR after checking that no host runs a container or holds an image from either, and they are absent from the package listing afterwards. Recoverable only by rebuilding from OpenSign upstream |
| Open Dependabot PRs and their branches | None | **Closed 2026-09-20** — the ten open PRs (opened 2026-09-14, each bumping an OpenSign dependency) were closed, as were the fifteen Dependabot recreated while the repository was unarchived (#21–#30, in two waves); its branches were deleted and the config removed (§3), and none reappeared afterwards. Only `main` remains |
| `innotelinc.github.io/sign` landing page | Not published; nothing links to it | Never worked: Pages was never enabled for this repository, so every `pages.yml` run failed with `Resource not accessible by integration` (the last on 2026-09-20, from the freeze push). Workflow gated to manual dispatch 2026-09-20 (§5.4). The one thing that *did* treat it as real was this repository's own homepage URL — repointed 2026-09-20 (§7) |

## 7. Reference audit (2026-09-19)

Where `sign-platform` / `OpenSign` / the old host address still appear, and
whether each is a deliberate historical reference:

| Location | References | Verdict |
| --- | --- | --- |
| `sign/**` | this repo throughout | Historical — frozen with the repo |
| `signara/docs/Roadmap.md`, `sign/CONVERGENCE.md` | the retirement record, the lost-data write-off | Intentional — these are the account of record |
| `signara/README.md` | "a modern replacement for proprietary e-signature platforms (DocuSign, Adobe Sign, OpenSign)" | Intentional — product positioning, not a dependency |
| `signara/apps/web/public/locales/*` | `{{appName}}`-style keys, OpenSign key names | Intentional — harvested catalogs, provenance documented in `src/lib/i18n/README.md` |
| `innotelinc/sign` repository metadata (GitHub, outside the tree) | description read "DocumentOps stopgap **converging into Signara**"; the homepage URL pointed at `innotelinc.github.io/sign/`, the Pages site that was never published | **Fixed 2026-09-20** — the description now says retired and archived, and the homepage points at `innotelinc/signara`. Metadata is not in the tree and an archived repository is read-only, so editing it again means unarchive → edit → re-archive |
| `distro/`, `olympus/`, `onyx/`, `wintrain/` | none | Clean — no action |
| `onyx/.tools/gocache/**`, `onyx/.tools/gomod/**` | incidental matches in Go build caches | Not ours — build artefacts, not references |

## 8. The one thing that must not be promised

Documents created on this platform before the cutover are **not in Signara**: the
only copy sat on a host that no longer exists, and the recovery attempt found
nothing (Roadmap §6). Any launch note or hand-over covering this retirement has to
say that, and keep a documented way to attach a finished PDF by hand. No wording
anywhere should imply the old envelopes came across.

---

*Archive inventory for `innotelinc/sign` · Innotel · 2026-09-19*
