# Security Review — MediGuard

MediGuard is a single-file, client-side medication-interaction and
3D-molecule viewer (`index.html`, ~569 lines). The regimen list, interaction
reference set, and schedule are in-memory/JS state (not even persisted to
`localStorage` here); OCR (Tesseract.js) runs on-device. The one real network
call is to PubChem's public REST API to fetch a drug's 3D structure
(`loadMolecule()`, `index.html:488`), sending only a drug name/CID — never
personal regimen data — consistent with the README's privacy note.

## 20-point pen-test checklist

| # | Check | Applicable? | Notes |
|---|-------|-------------|-------|
| 1 | HTTP/2 Rapid Reset (CVE-2023-44487) | N/A | No server component in this repo. |
| 2 | bcrypt 72-byte truncation | N/A | No password hashing. |
| 3 | SAML signature bypass (CVE-2024-45409) | N/A | No SAML/SSO. |
| 4 | nOAuth | N/A | No OAuth/auth of any kind. |
| 5 | GraphQL alias batching | N/A | No GraphQL. |
| 6 | TOTP replay (RFC 6238 §5.2) | N/A | No TOTP/2FA. |
| 7 | ReDoS | No | The one data-driven regex (`fmt()`, `index.html:412`: `f.replace(/(\d+)/g,...)`) is a simple digit match on a short internal formula string — no backtracking risk. |
| 8 | JWT algorithm confusion | N/A | No JWTs. |
| 9 | HTTP request smuggling | N/A | No server; the PubChem call is a plain client-side `fetch`. |
| 10 | Web cache poisoning | N/A | No server/cache layer under this app's control. |
| 11 | Spoofed X-Forwarded-For | N/A | No server-side request handling. |
| 12 | Postgres views bypassing RLS | N/A | No database. |
| 13 | `Math.random()` for security tokens | No | `Math.random()` is used only to position a decorative particle field in the 3D hero scene — not a security token. |
| 14 | Timing attacks on secret comparison | No | The Pro "unlock code" check (`applyCode` handler, `index.html:560`) is a plain `===` comparison against a hardcoded, non-secret code (`CONFIG.PRO_CODE`, `index.html:285`) — nothing secret to time-attack, and (as with this author's other similar apps) there's no server to enforce the gate against anyway. |
| 15 | Padding oracle | N/A | No encryption/decryption. |
| 16 | Server-side prototype pollution | N/A | No server. |
| 17 | Server-side template injection | N/A | No server, no templating engine. |
| 18 | XXE | N/A | No XML parsing (the PubChem response is an SDF chemical-table text format, parsed with fixed-column `substr()`, not XML/HTML). |
| 19 | Cross-site WebSocket hijacking | N/A | No WebSockets. |
| 20 | Unicode username normalization | N/A | No usernames/accounts. |

## Findings outside the 20-point list

**Stored XSS via the "type a medicine name" field — no escaping anywhere in
this app, including a second injection point via an inline `onclick`
attribute.** `addMed()` (`index.html:344`) explicitly accepts any typed name
that isn't already in the built-in reference set ("allow unknown; minimal
record", `index.html:346`) and stores it verbatim (only lowercased/trimmed by
`norm()`, `index.html:325`) as a `MEDS` key. That raw value is then
interpolated, unescaped, into `innerHTML` in several places:

- `renderMeds()` (`index.html:370-372`) — the medicine-list card, via
  `cap(m.key)` (`cap()` only capitalizes the first letter; it does not
  escape anything) — **and** the same raw `m.key` is also spliced directly
  into the card's `onclick="inspect('...')"` and the remove button's
  `onclick="...removeMed('...')"` attribute strings. A name containing a
  single quote and a closing paren (e.g. `x');alert(document.domain);//`)
  breaks out of the attribute and runs arbitrary script the moment the
  medicine is added and the list re-renders — no click or copy-paste needed.
- `renderIX()` (`index.html:376-380`) — interaction-pair labels.
- `renderSched()` (`index.html:398-399`) — the daily schedule.
- `inspect()` (`index.html:406-413`) — the full drug monograph modal.

Because the name comes straight from the app's primary "type to add a
medicine" input — not an incidental free-text field — this is directly
triggerable by normal use, not just by pasting untrusted content. There is
no backend and no sharing/import feature, so the practical exploitation path
is still self-XSS (the user's own browser, same-origin), but the attribute-
breakout variant means it fires immediately on add rather than requiring a
second render/reload.

Suggested fix: add an `esc()` helper (escaping `&`/`<`/`>`/`"`/`'`) and wrap
every `cap(m.key)`/`cap(k)`/`cap(f.a)`/`cap(f.b)` call — and the `onclick`
attribute values — with it, or switch the `onclick="...('"+k+"')"` pattern to
`data-key="${esc(k)}"` + an event-listener lookup to remove the attribute-
injection vector entirely. Not applied — pending your approval.
