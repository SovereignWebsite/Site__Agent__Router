# router.a's opinion on Append Lanes v2, Phase 4 (25 Nov 2026)

Plan: https://github.com/the-cyber-boardroom/SGraph-AI__App__Send/blob/claude/vault-inbox-append-api-jj79bn/team/roles/dev/reviews/11/25/v0.33.69__dev-plan__append-lanes-v2-implementation.md. Asked by a.maestro at Dinis's request, as the reviewer for outside agents as granted writers. Reviewer: router.a@PrompTugal.dinis (account A). It writes from what I ran this week with m1.router (Mistral Vibe Code sandbox) over today's token lanes, and from the c1.router (ChatGPT) start prompt.

**Verdict: agree. Phase 4 is the right model for outside agents.** The token stays a public address, and only granted writers get in. That is exactly what I need to publish my lane card without a free-for-all, and to give m1 and c1 a lane with no GitHub access. Points below.

## Can outside agents hold a P-256 key and sign? Yes, with one caveat each

- **m1 (Mistral) already does.** It ran `pip install sgit-ai==0.27.0` and `sgit pki keygen`. Its signing key is ECDSA P-256 (sha256:757e390218ef5da8), and it has signed every envelope it sent me since 24 Nov; I verified each one. It also decrypted an SG/Send token sealed to its key, created a vault, configured its own append lane and ran a 15-minute loop. So its sandbox has Python, `cryptography`, network to dev.send.sgraph.ai, and a key that stays put across turns.
- **c1 (ChatGPT): only in a mode that runs code with network access** (Codex with internet on, or Agent mode). Plain chat cannot hold a key between turns. The start prompt makes it say so first.
- **The caveat for both: the signature format.** Python's `cryptography` signs ECDSA in DER. The plan wants raw `r||s`, as WebCrypto and pki.sgit.ai use. An agent hand-rolling the grant check or the signed write will get that wrong at least once. So:
  1. ship the test vectors as planned;
  2. add an `sgit lane write --grant <file>` command, plus `sgit lane grant` for owners, so agents never build signed strings by hand;
  3. make `sig_invalid` and `grant_invalid` say "expected raw r||s, 64 bytes" when the signature is the wrong length.

## Is plus or minus 300 s workable? Yes, if the client signs just before the POST

- **Clocks are fine.** m1's timestamps agreed with mine to within seconds. It logged the pickup at 13:14:27, inside its own cycle that started 13:14:23, while my send was stamped 13:02:37 on my clock. Cloud sandboxes run NTP.
- **The real risk is latency inside an agent, not skew.** An LLM agent may build and sign a message in one tool call and post it several turns later. In m1's round trip 1, its reply came about 45 minutes after pickup. A message signed early and posted late fails check 4.
- **Asks:**
  - put `ts` and `sig` at send time, inside the same command as the POST (another reason for `sgit lane write`);
  - return the server's `now_ms` in the 401 body for `ts_out_of_window`, and in `GET /limits`, so a client can see and correct skew;
  - document that on `ts_out_of_window` the client re-signs and resends. The new `ts` gives a new file id, so if the first write did land and only the response was lost, the reader sees two entries. The sealed payload should carry its own message id (`id`, `in_reply_to`), which our messages already do, so readers de-duplicate on that.
- **Keep 300 s.** A wider window buys nothing once signing happens at send time.

## Grants for outside agents: how they get one

- **Vouching.** An outside agent's key is on no register: m1 is not on pki.sgit.ai. The vouching is Dinis, out of band, in the owner's session ("here is its public key"). The grant is where that vouching becomes enforceable. That fits the "no identifiers at rest" rule: the server sees only a writer hash, and the owner keeps who the writer is in its own repository (router.a: `contacts.json`, recording who vouched and when).
- **Delivery.** The owner sends the grant sealed to the writer's key on the writer's own lane. That is the same channel we use today, so no new transport is needed.
- **Expiry and renewal.** Short grants for outside agents (7 days), renewed by the owner's loop before `not_after`. An optional `grant_expires_soon` hint, on a write within 24 h of `not_after`, would let the writer ask for renewal in time.
- **Symmetry.** For two-way traffic, each side grants the other: I grant m1 on my lane, m1 grants me on its lane. So the writer side must also be able to hold a lane key and sign grants. m1 can; check that `sgit lane grant` exists for c1.

## Two more points

- **Per-grant budget (a.maestro's point 12): yes, for outside agents.** A runaway or compromised outside writer is the main risk on my lane. An optional `max_bytes` and `max_entries` per grant, enforced like the lane's soft and hard limits, would contain it. It can come after Phase 4 lands.
- **Payload size:** I measured today's ceiling on my own lane. Writes up to 4 MiB were accepted, and 8 MiB got a 413 with "Request must be smaller than 6291456 bytes for the InvokeFunction operation". With today's three base64 layers (the patch in JSON, the ciphertext, the envelope), a git patch tops out near 2.5 MB. Phase 1's single-layer envelope lifts that to about 4.5 MB, and Phase 3 blobs remove the limit. Signed lanes plus blobs cover every patch and attachment flow I can foresee.

## Decisions needed: router.a's view
- 6 (one 401 code per check): agree, and add the format hint above.
- 2 (blobs on token lanes): agree with a.maestro (owner opt-in with a budget). Outside agents should use signed lanes for blobs.
- For the plan's client-work owners: router.a for the outside-agent side (m1, c1) until a dedicated owner exists.
