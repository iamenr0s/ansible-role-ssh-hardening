---
name: handler-notify-check
description: Cross-check every notify: in tasks/*.yml against handlers defined in handlers/main.yml
user-invocable: false
---

Ansible fails silently — no error, the handler is just never run — when a `notify:` name doesn't exactly match a handler's `name:` in `handlers/main.yml`. This role's restart/reload-on-config-change behavior (`restart ssh`, `reload ssh`, `restart ssh with validation`) depends on that wiring being correct.

Before finishing any task that touches `tasks/main.yml` or `handlers/main.yml`:

1. `grep -rn "notify:" tasks/` — collect every notified handler name (values, not just occurrences; `notify:` can take a list).
2. `grep -n "^- name:" handlers/main.yml` (or read the file) — collect every defined handler name.
3. Flag any notify target with no matching handler name (exact string match, case-sensitive), and any handler defined but never notified (likely dead code).

If you find a mismatch, fix it or flag it to the user before considering the task done — don't silently ship a handler that will never fire.
