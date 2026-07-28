---
name: ansible-lint-exceptions-audit
description: Re-run ansible-lint without the repo-wide skip_list to see if rules 106/503 are still needed
disable-model-invocation: true
---

`.ansible-lint` skips rules `106` and `503` repo-wide. Periodically check whether that's still warranted, or has quietly become a blind spot as tasks were added:

1. Read `.ansible-lint` to confirm the current `skip_list` (don't hardcode 106/503 — read the live config, it may have changed).
2. Run ansible-lint without the skip list: `ansible-lint --nocolor -r <(sed '/skip_list/,/^$/d' .ansible-lint 2>/dev/null) .` — or simpler, just temporarily comment out the `skip_list` block in a scratch copy of the config and run `ansible-lint -c <scratch-config> .`
3. Report every violation of the skipped rules, with file:line, so the user can decide per-violation whether to fix it or keep the blanket skip.
4. Don't modify `.ansible-lint` or fix violations without being asked — this is an audit, not an auto-fix.
