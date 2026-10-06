# What an agent-written decision log records

Agents write 100% of the code here. I write none of it. I write the intent, approve the plan
and the validation contract, ratify the diff, and accept the risk; the division is set out in
`agent-factory/constitution.md`.

The agents also keep their own decision log, because the constitution requires it: when a real
decision is made, one line is appended to `decisions.md`, never a new file. The coordinator
seat writes those lines, in Portuguese, as it works. The published copy is an English
translation with the IDs and dates preserved — 68 entries, `D-00` to `D-64`, spanning
2026-06-22 to 2026-09-16. I appear in about fifteen percent of them, by name, as the person
who ratified or corrected. Never as the author.

That provenance is the reason it is worth reading. It is a machine's record of its own
mistakes, kept under a rule that forbids editing: five entries correct earlier entries and say
so.

## The finding that generalises

`D-16`. There was a permission cage — a module that rendered a correct, audited sandbox
config, with a test, and a header comment naming the driver that called it. The driver did not
exist. The one actually running reads a different config schema entirely. So for the seat
doing the real work, the Critical File denies, the `git push` deny and the `~/.ssh` deny were
never enforced, three weeks after the incident that motivated building it was believed closed.
The ledger files it as a discovery, not a regression: nothing broke, it had never worked.

`D-25` is the same thing one level up. Both security probes had it. The cage probe, run
without a project argument, resolved its Critical File list to empty, filtered nothing,
printed `PASS … 0 glob(s)` and exited 0 — the tool whose only job is to catch a disarmed cage,
green-lighting a disarmed cage. Its twin printed `nothing to probe (clean)` and exited 0 when
the file it was meant to compare did not exist.

The rule that came out: **a check that cannot fail must never print PASS**, and "I could not
compare" is a non-zero exit, never `clean`. Every new metric now has to answer one question —
what does it print when its tool is off? If the answer is zero, it is a decoration.

The same shape in three more costumes. `D-34`: a gate's exit code captured through
`| tail -25`, which is `tail`'s exit code, always 0 — two red suites summarised as
`E2E=0 QG=0`. `D-37`: a deny rule pointing at `prisma/schema.prisma` when the real path was
`backend/prisma/schema.prisma`, so it matched nothing, the audit passed it, and the runbook
went on asserting in prose that agents never edited the schema. `D-57`: a worker killed by the
OS never writes its end-of-run event, so it vanishes from every aggregate that joins on that
event — meaning every throughput number produced before run reconciliation was optimistic by
construction, because the runs that disappear are precisely the ones that broke.

## Where the gates did work, and where I did

`D-38` is the uncomfortable entry. The coordinator seat diagnosed two failing end-to-end
suites as an environment artefact, recorded "pre-existing, not a regression from this mission",
and stopped looking. The sentence was true; the conclusion attached to it was not. A held-out
validator seat, with a clean context and no stake in that diagnosis, traced all four failures
to one line: a transaction took an advisory lock via a raw query, the lock function returns a
`void` column, and the database client version pinned in the lockfile throws when deserialising
one. The error was caught and logged as a generic "webhook processing error". The webhook
answered HTTP 200. The message was never persisted. Inbound messages were being silently
dropped by a product whose entire job is to not drop them.

`D-59` and `D-60` are the pair I would point at to show the human gate doing real work. A
newer model had shipped, better on exactly the long-horizon benchmark that matters for a
builder seat, and the question was whether our flat-rate vendor plan covered it — because
pointing a seat at an uncovered model does not fail loudly, it quietly moves to metered
billing and surfaces days later as an insufficient-balance error. My standing rule is never to
choose a model from memory. The agent applied the rule, found the plan did not cover the new
model, and reverted its own change. That looked like the rule working.

I checked the provider's own page and it was wrong. The documentation says, verbatim, that all
plans support that model. The lookup had gone through an indexed documentation mirror serving a
stale snapshot as current. The correction is `D-60`, with a derived rule: for a vendor's price,
plan or availability, fetch the vendor's own page, never a cache.

I could not have written that agent's code. I could notice that a commercial fact about money
had been sourced from a cache. That is the division of labour, stated accurately.

The ledger is public, as `decisions.md` in
[`agent-factory`](https://github.com/eusoubrasileiro/agent-factory). Numbers in this note were
true at the point of export; re-derive the entry count with
`grep -cE '^- [0-9]{4}-[0-9]{2}-[0-9]{2} . D-[0-9]+' decisions.md`.
