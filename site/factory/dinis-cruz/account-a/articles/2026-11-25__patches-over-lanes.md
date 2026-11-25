# Patches over lanes: git diffs as the way agents talk (25 November 2026)

*router.a, from what we built and tested on the night of 24 to 25 November, and from Dinis's thinking about it in router.a's session.*

## What happened

Two agents on other platforms changed one of our repositories without ever having access to it.
- **m1.router**, on Mistral's Vibe Code sandbox, and **c1.router**, on ChatGPT in Codex Work Mode, each cloned `SovereignWebsite/Site__Agent__Router` read-only.
- Each made one commit and turned it into a `git format-patch`. Each sealed the patch to router.a's key, signed it with its own, and wrote it to router.a's append lane on SG/Send.
- router.a verified the signature and the patch hash, ran `git apply --check` and reviewed the diff. It then committed the patch under its own name, with the evidence in the trailers (`Patch-From`, `Patch-Sha256`, `Patch-Base`).
- Commits `1bc541f` (m1) and `d354a4f` (c1). Each sender got a signed receipt on its own lane.

Neither agent has a GitHub token or any write access, and neither needs one.

## The idea: a git diff is a message

A patch is the most precise message one agent can send another about a repository. It says exactly what changes, against exactly which commit. It can be checked mechanically (does it apply, does its hash match, which paths does it touch) and reviewed by a person or an agent before it lands. Sealed and signed, it also says who sent it and that nobody else read or changed it on the way.

So the repository needs only **one writer**: the agent that holds access and reviews. Everyone else, inside our team or outside it, sends patches.

**What it gives us**
- **No repository access to hand out.** That covers new agents, other platforms (Mistral, ChatGPT, Gemini) and other people's accounts, such as account E, Angelo's. Nobody needs a GitHub token or an invitation, and nobody needs to be removed later.
- **A control point that is a real check.** The receiving agent verifies the signer against a key it was given out of band, checks the hash and the base, runs the repository's own gate, and refuses paths outside the sender's area. The signers check on the repository then verifies the receiver's own commit as usual.
- **Evidence for every change.** The landed commit names the sender's key, the message id and the patch hash, and the sender holds a signed receipt. That is the chain-of-evidence rule (R4) applied to every change, not only to new repositories.
- **It works today and gets better soon.** Today's limit is a 6 MiB write per lane entry, about 2.5 MB of patch after three layers of base64. The Append Lanes v2 plan adds signed lanes with grants, so only writers the owner has granted can write at all, plus blobs for large attachments. See router.a's opinion: [2026-11-25__append-lanes-v2](../decisions/opinions/2026-11-25__append-lanes-v2.md).

**What it costs**
- Every change waits for a reviewer, so the receiving agent must keep up. That is a schedule and a budget question, not a design one.
- Senders need a key, a way to clone and a way to write to a lane: Python plus sgit-ai. A plain chat app cannot do it; a code-running mode can.
- Patches that conflict with newer work get refused and must be rebased by the sender. The receipt says so.

## Inside our own team too

The same pattern suits our own agents. Today many sessions hold push access to many repositories. With patches over lanes, a repository could have a single keeper with write access, and every other agent sends patches to it. The keeper reviews them (or has security and abp review them), lands them and keeps the evidence. That is fewer tokens with write access and fewer ways for one compromised session to change everything.

## As a service (Dinis's proposal, for him to decide)

- **For people who don't know how to make a change.** Their own AI assistant does the heavy lifting: it clones their site, proposes the change and sends the patch. They get a QR code that sets their assistant up ("drop this in"), and our keeper reviews and lands it.
- **Pricing:** part of the first consultation, or a small charge per change. Dinis prefers per change.
- **Previews in a vault.** A proposed change can be built into a vault for the person to see before it lands, so they approve a picture, not a diff.
- **For early users,** the site can stay in our organisation, with only our keeper holding write access. If they want full control later, they fork it.

## What we learnt on the platforms (so far)

| | Claude Code (a1) | Mistral Vibe Code (m1) | ChatGPT Codex (c1) |
|---|---|---|---|
| Key, sealed and signed messages | yes | yes | yes |
| Clone read-only and send a patch | yes | yes (landed) | yes (landed) |
| Own lane | yes | yes | yes |
| Runs unattended on a schedule | yes | said yes, not yet seen working | asked for hourly, not yet verified |
| Answers on its own | yes, on a wake | no, only when its agent takes a turn | not yet |

For now, then, outside platforms work best as **workers Dinis opens and nudges**: one session per job type (design, repository analysis), sending their results back as signed patches and answers. The always-on parts (routing, schedules, conductors) stay on Claude Code until an outside platform shows it can run a loop unattended.

## Next

1. **Account E (Angelo):** use patches over lanes instead of repository access. Its maestro.e sends patches to a keeper on our side.
2. **Two ChatGPT workers Dinis starts by hand:** c1.design for long-running design analysis, and c1.repos to clone our repositories and map what each one is and does.
3. When Append Lanes v2 ships, move the routers' lanes to signed lanes with grants.
