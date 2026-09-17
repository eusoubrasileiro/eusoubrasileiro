# Six months of agent-built software: what 68 decisions say

I do not architect or author the code. Agents do — worker seats produce ~95% of it, per our
own published engineering standard (`harness-standards/standards.md` §1). I author the
intent, ratify the plan and the validation contract, and own the risk. That division is either reckless or it is a working process, and
the only honest way to tell the difference is to look at what the process wrote down when it
was wrong.

The practice runs about six months. The repositories published alongside this essay cover
the most recent stretch of it — the earlier months live in repos that were never published,
so take the older span on my word and the recent one on the commands below.

So this is not an essay about how well it went. It is a reading of an append-only decision
ledger — 68 entries, written in Portuguese as the work happened, translated for publication
with the IDs and dates preserved. The ledger has one rule that makes it worth reading: it is
append-only. A decision that turned out to be wrong is not edited. A later line corrects it
and says so. Five of the entries below are corrections of earlier entries. That is the point.

## The setup, in one paragraph

Work is decomposed into *missions*. Each mission gets a plan with a `## Done when` section
that is a validation contract, not a wish list — every assertion has to name a mechanism that
actually executes on the seat that will run it. Each mission is dispatched into its own git
worktree. A builder seat writes the code; a held-out validator seat, with a clean context,
checks the diff against the contract; I ratify at the human gate. The builder seat runs
inside a *cage* — a permission configuration rendered fresh per spawn from per-project data,
never from the engine. The engine knows the shape of a project; only the project profile
knows the facts. That single rule (`D-15`) is what let a third project be onboarded by
creating a directory, with zero engine edits.

That is the design. Here is what the ledger says happened to it.

## 1. The cage was never installed

`D-16`, and it is the first entry I would show anyone.

We had a cage module. It rendered a correct, audited permission config. It had a test. It had
a header comment naming the driver that called it. And it had **zero callers in production**.
The driver named in its own header did not exist. The driver we were actually running was a
different one, for a different agent runtime, which does not read that config schema at all.

So for the seat doing the real work, the Critical File denies were not enforced. The
`git push` deny was not enforced. The `~/.ssh` deny was not enforced. The incident that
motivated building the cage in the first place was still fully possible, three weeks after we
believed we had closed it.

The line I wrote at the time is the one I still use as a test: *a cage that is not installed
costs zero productivity and buys zero security — it buys only the belief in security, which
is worse than knowing you have no cage.*

Note the framing in the ledger: **a discovery, not a regression**. Nothing broke. It had
never worked. Those are the expensive ones, because nothing ever alerts.

## 2. A metric whose tool is off reports a confident zero

This is the same failure, generalised, and it is the single most useful idea the whole
programme produced.

The quality gate measures deterministic metrics against a checked-in baseline and blocks on
regression. One of its metrics is cognitive-complexity violations. That number read **0**
from the day the gate landed until 2026-09-14. Not because the code was simple — because the
linter rule that emits that diagnostic is not in the tool's recommended set, and the config
never enabled it. The category was never emitted, the filter matched nothing, and the
baseline pinned the result as a fact. Enabling the rule surfaced **74** real violations.

The same collector had a second fake zero waiting: a `catch` block that returned
`{ complexityViolations: 0 }` on failure. It never fired, because the metric was never on.

Zero is a *passing* value. A gate that cannot fail reports success forever, and every green
run after that is evidence of nothing while looking exactly like evidence.

Once you see the shape you see it everywhere, and the ledger is mostly a record of finding it
again in new costumes:

- **`D-25`.** Both security probes had it. The cage probe, run without a project argument,
  resolved its Critical File list to empty, filtered nothing, and printed `PASS … 0 glob(s)`
  and exited 0. The tool whose only job is to catch a disarmed cage was green-lighting a
  disarmed cage. Its twin, the secrets probe, printed `nothing to probe (clean)` and exited 0
  when the file it was supposed to compare did not exist. A seat with a real leaked secret
  would have certified clean. The rule that came out of it: *a check that cannot fail must
  never print PASS*, and "I could not compare" is a non-zero exit, never `clean`.
- **`D-37`.** A deny rule pointed at `prisma/schema.prisma`. The real path was
  `backend/prisma/schema.prisma`. The rule matched nothing, the cage audit passed it clean —
  it validates anchoring and unsubstituted placeholders, not target existence — and the
  runbook went on asserting in prose that the schema was never edited by agents. It had been
  editable the whole time.
- **`D-34`.** I ran a gate command through `| tail -25` to summarise the log, then captured
  `$?`. That is `tail`'s exit code. Always zero. Two suites were red and my own summary
  printed `E2E=0 QG=0`. Same defect as `D-25`, on the orchestrator's side rather than the
  tool's.
- **`D-32`.** The board publisher wrote its "published" hash memo without checking the
  rsync's exit status. A failed publish left a memo describing content the server never
  received, so every later run reported "no changes", forever, while the log said
  "published".

The general form is a question, and it is now the first thing asked of any new metric:
**what does this report when its tool is off?** If the answer is zero, you have built a
decoration.

## 3. `exit=0` means "the model stopped talking"

`D-35`. A builder seat ran for 12 minutes, burned **1.7M tokens**, and exited
`exit=0 timedOut=false`. It had written 173 lines of good new test — the RED — and had never
touched the implementation file. No commit. Nothing.

An orchestrator that trusts an exit code marks that feature done and dispatches the next one,
which depends on it. The rule is that accepting a worker's output is always a ground-truth
check of the repository: does `git log` show a new commit, does `git diff --stat` touch the
files the spec authorised, does the gate run green *when I run it myself*. Never the driver's
exit code.

`D-36` explains why that seat stalled, and it is a genuinely uncomfortable finding: the file
the spec told it to edit was inside the cage's deny list. The plan's own "files this mission
authorises" section had no mechanism propagating it into the cage. The worker had no channel
to say *I am forbidden from the file you asked me for*. It tried, got blocked, and went
quiet. The accepted consequence was not to loosen the cage — it was that features touching a
critical file do not go to a caged seat at all.

And later, `D-57`: a worker killed by the OS never writes its end-of-run event, so it
disappears from every aggregate that joins on that event. Seven orphan runs across the
factory's entire life. **Every throughput number produced before that reconciliation was
optimistic by construction**, because the runs that vanish are precisely the ones that broke.

## 4. The red everybody agreed to ignore

`D-38` is the entry I find hardest to read, because I wrote the mistake it corrects.

Two end-to-end test files were failing. I diagnosed them as an environment artefact — a suite
dragging in tests that need real vendor credentials, which cannot work on an isolated seat —
recorded that as `D-33`, and moved on. The sentence "this is pre-existing, not a regression
from this mission" was true. The conclusion I attached to it, "therefore not my problem", was
false. And the evidence against me was in plain sight: one of the two failing files was not
in the directory I had blamed.

The validator seat, running with a clean context and no stake in my diagnosis, traced all
four failures to one line. A transaction took an advisory lock via a raw query; the lock
function returns a `void` column; the database client version pinned in the lockfile throws
when deserialising a `void` column. The error was **caught and logged** as a generic
"webhook processing error". The webhook answered HTTP 200. The message was not persisted.

Every inbound message on that path was being silently dropped, in a product whose entire job
is to not drop inbound messages. The fix was one line — a different raw-query method that
returns a row count instead of deserialising a column. The bug was found because somebody
with no investment in my explanation looked at the red I had labelled as scenery.

The rule that came out: a pre-existing red is debt to be *named and dated*, not landscape;
whoever inherits it owes it one root-cause pass. `D-33` had already argued that a gate
command which cannot pass teaches the *worker* to ignore red. What I had missed is that it
teaches the orchestrator too.

## 5. The human gate is not ceremonial

`D-59` and `D-60`, one day apart, are the clearest evidence I have that ratification does
real work rather than rubber-stamping.

A newer model had shipped, materially better on exactly the long-horizon benchmark that
matters for a builder seat. The question was whether our flat-rate vendor plan covered it,
because pointing the seat at an uncovered model does not fail loudly — it quietly leaves
flat-rate for metered billing and surfaces days later as an insufficient-balance error. My
standing rule is: never choose a model from memory, verify capability against the provider's
documentation. The agent applied the rule, found the plan did not cover the new model, and
**reverted its own change**, which looked like the rule working.

I checked the source myself and it was wrong. The provider's documentation says, verbatim,
that all plans support the new model. The lookup had gone through an indexed documentation
mirror that was serving a stale snapshot as if it were current. The rule was right; the
execution confused *an indexed mirror* with *the provider's page*. The correction became
`D-60`, with a derived rule: for a vendor's plan, price or availability, fetch the vendor's
own page directly.

I could not have written that agent's code. I could notice that a commercial fact about
money had been sourced from a cache. That is the actual shape of the division of labour, and
it is why the ledger's second rule is that a decision names its ratifier — a line without one
is a proposal, and lives in a separate inbox file until it is ratified.

`D-61` is the same division in the other direction: turning the expensive gate on by default
for the most expensive project. Slower fan-out, real wall-clock cost, my call to accept —
because an unmeasured run is the expensive thing, and without it the merge rests on the
seat's own word about itself. Note that `D-61` corrects `D-55`, my own earlier default. It
corrects the *default*, not the design: a metric that blocks is a metric the operator turns
off, after which nothing is measured at all.

## What the numbers actually are

Everything here comes from the published repository, and each figure is one command:

- **68 ledger entries**, spanning 2026-06-22 to 2026-09-16. Sixty-four distinct decision
  numbers (`D-00`–`D-64`, with `D-53` reserved and never used), plus `D-48b`, plus three
  numbers that genuinely collide — a cross-machine merge renumbering, left visible and
  disambiguated by `D-50` rather than quietly renumbered, because the file is append-only.
- **186 commits**, of which **157** carry a `Co-Authored-By` trailer naming the model that
  wrote them.
- **1,060 tests**, 1,053 passing, 0 failing, 7 skipped, in 27 seconds on one machine.
- **53 mission dossiers**, 37 engine scripts, 33 test files, 2 project profiles, 3 agent
  skills, 10 archived plans.

Two honest limits on those numbers. First, they describe *one* published repository — the
factory engine — whose own history runs 2026-07-08 to 2026-09-17. The practice is about six
months old; the earliest commit across everything published here is 2026-05-09, so roughly
the last four of those six months are publicly verifiable and the rest is my word. Second,
`D-57` above means any throughput figure predating run reconciliation is optimistic, and I
would rather say that than quote it.

## The claim, stated narrowly

I did not architect or author this. Agents did, in parallel, under a contract I wrote and a
gate I own. What I contributed is the part that does not scale by adding seats: deciding what
"done" means before the work starts, refusing green that cannot fail, accepting the risks
that are mine to accept, and keeping a record honest enough that it can be quoted against me
later.

Five of these entries are corrections of earlier entries in the same file. Three of the most
useful findings came from a seat whose job was to disagree with me. One came from a test I
wrote wrong, which failed for a reason I had not predicted — and an assertion that fails for
an unpredicted reason is information, not noise. The tempting move is to fix the assertion
and move on.

The ledger is the artefact. It is public, in the `agent-factory` repository, as
`decisions.md`.
