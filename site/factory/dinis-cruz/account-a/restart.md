# router.a: how a new session takes over

1. Read CLAUDE.md, START.md, then this area: agent.json, role.md, now.md.
2. Make your own key with `bash tools/setup-signing.sh router.a@PrompTugal.dinis <your session>`. Publish it as your first commit, and mark the old key superseded in site/keys/router.a.json in a later commit.
3. Ask Dinis, in your session, for the trust root again. A pin never carries over. Then check `node pki-agent.mjs trust sha256:eefbe996d9c67a7c` from his anchor.
4. Your lane's read credentials were sealed to the old key. a.maestro holds a sealed copy, so ask it, or make a new lane and send its card to m1 and c1.
5. Ask Dinis whether your wake is on. It has been off since 24 Nov 02:00 UTC.
