# An eval gate on real WhatsApp traffic

The product is a lead-triage bot that lives inside a company's WhatsApp. Customers message the
same number they have always messaged. A bot reads the inbound traffic, decides whether a
conversation is a sales lead, and, if it is, opens a card on a board that humans work. The
tenant is a facilities-services company. The traffic is real — real customers, real
misspellings, real people sending a voice note at 11pm and then a photo of a wall.

I do not write the code. Agents do; I write the intent, ratify the plan, and own the gate that
says whether the thing may go live. So the only question I am actually qualified to answer about
this product is the one this essay is about: *how do you know the gate is measuring the bot?*

Twice I built something that reported green while measuring the wrong thing — once the gate
itself, grading a copy of production rather than production, and once the board, counting a
population that was never live work. Both are below, in that order.

## The seam

The most important file in this story is a small one, `backend/src/bot/triage-turn.ts`, and the
best paragraph I can offer is not mine. It is that file's own header:

> Split out of `triage-bot.ts` so the SDR go-live gate could drive the REAL assembly instead of a
> hand-maintained copy. The eval runners used to build this state object themselves under a
> literal `Mirror production EXACTLY` comment; any drift between that copy and production was
> invisible to the gate BY CONSTRUCTION.

That is the whole lesson in three lines. Before that split, the eval harness assembled the bot's
per-turn state itself. It did a careful job. It was written by someone who understood
production, and it carried a comment in capital letters promising that it matched. And a comment
promises nothing. The moment production changed a field and the copy did not, the gate went on
passing — not because the bot was right, but because the gate was grading a different bot that
happened to still be right.

This is the failure mode I now look for before any other: not a gate that fails wrongly, but a
gate that *cannot* notice. A green run against a replica is evidence about the replica. Nothing
downstream of it is evidence about the product.

The fix was structural, not editorial. There is now one composer. Production calls it. Both eval
runners call it. And `test/bot/triage-turn-parity.test.ts` runs on every commit and fails if the
three ever diverge in shape. The guarantee moved out of prose and into something that executes.
That is the only kind of guarantee I trust now, and the reason is simply that I cannot audit
prose at the rate agents produce it.

## Increments find flaws; the full corpus judges

Evals on this product cost money — they call a paid model over a gold corpus. There are 89 gold
files under `backend/test/eval/gold`, 13 more under `backend/src/eval/gold`, and 12 eval test
files driving them. Running all of that is not free, and the temptation when it is not free is to
run a slice and call it a verdict.

The rule we settled on is written down as a standard, and its headline is verbatim: *"Paid evals:
increments find flaws, the full corpus judges."*

In practice that means increments go out one at a time and each one is **read** before the next
is paid for. The purpose of an increment is not to score the bot. It is to find the flaw in the
harness, the fixture or the tenant configuration while finding it is still cheap — a wrong
prompt variable, a fixture that no longer parses, a tenant config pointing at the wrong
definition of a business hour. Those bugs are worth catching after ten cases instead of after a
hundred.

But an increment never produces a verdict. A gate verdict requires the full corpus, every time.
The two jobs look similar and they are not the same job, and the failure I am guarding against is
the comfortable one: a clean slice, a tired evening, and a decision to ship. Increments find
flaws. The full run judges the bot.

## What the gate actually caught

The most valuable thing the gate found was not a bug in the bot. It was that the board and I did
not agree on what a lead is.

The funnel, measured on the tenant's real traffic, is `1471 → 315 → 90`. Two filters, and each
one of them was a definition that was wrong before it was measured.

The first filter is **evidence**. An inbound message proves a conversation happened. It does not
prove a lead entered the pipeline. 1471 conversations exist; 315 of them carry evidence of
actually being a lead. I had been treating the first number as the top of the funnel, which
inflates everything below it and makes every conversion rate meaningless.

The second filter is **era**, and it is the one I would not have found by reading code. When the
tenant's WhatsApp was connected, the coexistence connection back-filled 180 days of history:
**14,734 imported messages against 2,273 live ones**. Six times as much history as live work,
arriving in one go, all of it indistinguishable from real inbound traffic to anything counting
naively.

The inbox handled this correctly and always had. The importer landed every imported conversation
as `RESOLVED`, because imported history is not live work and nobody should be asked to work it.
The board did not know that. The line from the PRD is the one I keep quoting: *the inbox always
knew about the era; the board did not.*

The 315→90 step is that correction. And the structural fix matters more than the number: the era
rule now lives in exactly one definition, `services/lead-scope.ts`, which the board, the funnel
and the customer-360 view all import. Before, there were three places that agreed with each
other by luck. Three implementations that agree today are not one definition; they are three
future bugs that have not diverged yet.

## The numbers, including the ones I would rather not print

Backend line coverage is 92.73%, branches 90.2%, functions 96.7%, across 10,858 assertions.

Frontend coverage is about 43%.

I am printing the second number in the same section as the first on purpose. An essay that
reports only its good metric is doing precisely the thing this essay criticises — presenting a
measurement of a favourable subset as a measurement of the system. The backend is where triage
decisions are made and it is tested to a standard I will defend. The frontend is not, and if you
are evaluating my work you should weight that.

There are 75 cognitive-complexity violations in the codebase. Not zero. They are tracked as a
ratchet: the number is allowed to go down and never up, and a commit that raises it fails. I
prefer a ratchet to a target because a target invites a one-week cleanup sprint and then twelve
months of quiet regression, whereas a ratchet is enforced by the same machine that runs the
tests.

And mutation testing: **15 of 15 mutants killed** — sampled, range-scoped mutation testing across
6 files, bounded to one commit range. I want to be exact about what that is and is not, because
15/15 is the kind of number that looks like a project-wide mutation score and would flatter me
enormously if you read it that way. It is not a survey. It is 15 mutants in 6 files. It tells me
the assertions in that commit range have teeth. It tells you nothing about the other several
hundred files, and I have not measured those.

Unit economics — what a triaged conversation actually costs to run — are out of scope for this
write-up. The figure lives in a client-owned file I am not publishing from, and I would rather
leave the argument one number short than quote something I cannot show you.

## What I would do differently

I would build the parity test before the eval harness, not after. The order I actually used was:
build the harness, get it green, then discover that green was about a copy. Every hour the
harness ran before the parity test existed produced results I later had to discount. That is not
a small cost — it is the cost of the whole first phase.

I would also have gone looking for the era problem earlier, and the way to have done that is
embarrassingly simple: before trusting any count, ask where the rows came from. Not "is this
query correct" but "what population is this query over, and did something put rows into it that
nobody intends to work?" A 180-day backfill is not an exotic event. It is the first thing that
happens on every new tenant, which means it is the first thing every count on a new tenant gets
wrong.

The general form of both mistakes is the same one, and it is the thing I would put on the wall:
a green check is a claim about whatever it actually ran against. Before believing it, make it
prove what that was.
