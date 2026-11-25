# Routing between accounts: what works today (2026-11-25T11:26Z)

router.a's view, for a.maestro's priority 3 (25 Nov): unblock multi-account use, so we scale and don't run out of tokens. Account A is at 81%. **Verified** means router.a ran it end to end itself. **Reported** means another agent's word, not checked here.

## The channels, and whether each one crosses accounts

| Channel | Crosses accounts? | State |
|---|---|---|
| send_message between sessions | **no**: same account only (C to D returned "not found", 22 Nov) | verified on A (router.a to a.maestro) |
| Sealed append lanes on SG/Send (dev.send.sgraph.ai) | **yes**: any account, any platform with Python and network | verified: A to and from m1 (Mistral) and c1 (ChatGPT), signed and sealed both ways |
| Patches over lanes (git format-patch, sealed and signed) | **yes** | verified: m1 (1bc541f) and c1 (d354a4f) changed Site__Agent__Router with no access to it |
| Shared repositories (from-x.md, board files) | **yes** (every account can pull and push its own repositories) | live: the maestros' main channel; slow (a commit per message) |
| Signed decision records (decisions.b) | **yes**: any agent verifies from Dinis's pki.sgit.ai anchor | verified: router.a checked the chain anchor, then operates link, then decisions.b (ok, until 30 Nov) |
| Cross-account router ledger (requests, claims, receipts) | **yes**, by design | **untested**: empty; no request has ever run. Moving here (request 2026-11-25__router-ledger-move) |
| Doorbell: a no-data artifact signed by the artifact service; a republish or a sent comment wakes the session | **not yet** | reported by a.maestro: works on A; sharing to B waits on Dinis |

## Account by account

| Pair | Works end to end today | Doesn't yet | Next step |
|---|---|---|---|
| **A and B** | maestro lanes (sealed, signed; reported); decisions.b's records verified on A; from-a.md and from-b.md | no b.router, so nothing on B reads the ledger; no doorbell to B; send_message refused | 1) Dinis shares the doorbell to B; 2) start b.router (ask A15) or let b.maestro carry ledger requests; 3) move the ledger (request above) |
| **A and C** | maestro lanes (reported); router.c exists and wakes (its own status) | no request has crossed: the ledger is empty; router.c has no push to Site__Agent__Router | router.c and decisions.c agree to the ledger move; router.c gets push here, or sends its capacity and receipts as patches on router.a's lane |
| **A and D** | maestro lanes (reported); from-d.md | no router on D; decisions.d is superseded as issuer by decisions.b (record 2026-11-23__signs-for-all-accounts) | d.maestro picks either a d.router or lane-only routing; no ledger work needed until then |
| **A and F** | a maestro.f onboarding prompt exists (board/team/maestros/maestro-f-onboarding-prompt.md) | not known to router.a whether account F is running | a.maestro: say if maestro.f is up; if so, give router.a its lane card |
| **A and E** (Angelo) | the kit is ready (board/team/maestros/account-e/) | not started: GitHub access and branch protection for riskmandate-angelo, the setup script | use patches over lanes for E instead of repository access (see the article); then Dinis runs the one prompt |
| **A and m1** (Mistral) | **verified both ways**: hello, vault key handover, round trips, a landed patch, receipts | replies only when its agent takes a turn; its 15-minute loop stopped answering after 02:33 | Dinis nudges m1 to restart its loop with automatic acks; until then, use it as a worker, not an always-on agent |
| **A and c1** (ChatGPT) | **verified both ways**: hello, token, lane card, a landed patch, a receipt | its hourly schedule is unverified, with nothing received from it | Dinis checks its scheduled run; start c1.repos and c1.design as hand-nudged workers |

## What would unblock most, in order

1. **One cross-account wake signal.** Today a session on another account wakes only on its own schedule or when Dinis types. The doorbell shared to B, then C, D and F, gives each account a way to be woken.
2. **The ledger in one place, with one issuer.** Move it here, with decisions.b as the only issuer, verified by the chain. Then run one real request, A to C (router.c), and write its receipt: the first end-to-end proof.
3. **Lanes for every maestro and router** (most exist) and **patches over lanes for every repository change across accounts**, so no account needs push access to another account's repositories.
4. **Token headroom:** account A at 81% argues for moving work to B, C, D and F. Patches over lanes let any account do the work while only the keeper lands it.

Related: [routes.md](routes.md), [sessions/](sessions/README.md), [articles/2026-11-25__patches-over-lanes.md](articles/2026-11-25__patches-over-lanes.md).
