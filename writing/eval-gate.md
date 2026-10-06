# An eval gate on real WhatsApp traffic

The product is a lead-triage bot living inside a company's WhatsApp. Customers message the
number they have always messaged; a bot reads the inbound traffic, decides whether a
conversation is a sales lead, and if it is, opens a card on a board humans work. The tenant is
a facilities-services company. The traffic is real — real customers, real misspellings, real
people sending a voice note at 11pm and then a photo of a wall.

Agents write the code. I write the intent and the definitions, and I own the gate that says
whether the thing may go live. So the question I am qualified to answer about this product is
narrow: **how do you know the gate is measuring the bot?**

Twice it was not.

## The gate was grading a copy

The important file is a small one, `backend/src/bot/triage-turn.ts`, and the best paragraph
about it is its own header:

> Split out of `triage-bot.ts` so the SDR go-live gate could drive the REAL assembly instead
> of a hand-maintained copy. The eval runners used to build this state object themselves under
> a literal `Mirror production EXACTLY` comment; any drift between that copy and production was
> invisible to the gate BY CONSTRUCTION.

Before the split, the eval harness assembled the bot's per-turn state itself. It did a careful
job, written by something that understood production, and it carried a comment in capital
letters promising it matched. A comment promises nothing. The moment production changed a field
and the copy did not, the gate went on passing — not because the bot was right, but because it
was grading a different bot that happened to still be right.

The fix was structural, not editorial. There is now one composer; production calls it, both
eval runners call it, and `test/bot/triage-turn-parity.test.ts` fails if the three ever diverge
in shape. The guarantee moved out of prose into something that executes, which is the only kind
I will accept — because I cannot audit prose at the rate agents produce it.

## The board and I disagreed about what a lead is

The most valuable thing the gate found was not a bug. The funnel on real traffic is
`1471 → 315 → 90`, and each arrow is a definition that was wrong before it was measured.

**Evidence.** An inbound message proves a conversation happened; it does not prove a lead
entered the pipeline. 1,471 conversations exist, 315 carry evidence of being a lead. I had been
treating the first number as the top of the funnel, which inflates everything below it and
makes every conversion rate meaningless.

**Era.** When the tenant's WhatsApp was connected, the coexistence connection back-filled 180
days of history: **14,734 imported messages against 2,273 live ones.** Six times as much
history as live work, arriving at once, indistinguishable from real inbound traffic to anything
counting naively. The inbox had always handled this correctly — the importer landed every
imported conversation as `RESOLVED`, because imported history is not live work and nobody
should be asked to work it. The board did not know that. The line from the PRD is the one I
keep quoting: *the inbox always knew about the era; the board did not.*

The 315→90 step is that correction, and the structural fix matters more than the number: the
era rule now lives in one definition, `services/lead-scope.ts`, which the board, the funnel and
the customer-360 view all import. Before, three implementations agreed with each other by luck.
Three implementations that agree today are three future bugs that have not diverged yet.

## Pacing, because evals cost money

Evals here call a paid model over a gold corpus — 89 gold files under `backend/test/eval/gold`,
13 more under `backend/src/eval/gold`, 12 eval test files driving them. The standing rule is
written down as a standard: *"Paid evals: increments find flaws, the full corpus judges."*

An increment's purpose is not to score the bot. It is to find the flaw in the harness, the
fixture or the tenant configuration while finding it is cheap — a wrong prompt variable, a
fixture that no longer parses, a tenant config pointing at the wrong definition of a business
hour. Those are worth catching after ten cases instead of a hundred. But a verdict requires the
full corpus, every time. The failure that rule guards against is the comfortable one: a clean
slice, a tired evening, and a decision to ship.

## The numbers, including the ones I would rather not print

Backend line coverage is 92.73%, branches 90.2%, functions 96.7%, across 10,858 assertions.
Frontend coverage is about 43%.

Both go in the same paragraph on purpose. An essay reporting only its good metric is doing
exactly what this one criticises — presenting a measurement of a favourable subset as a
measurement of the system. The backend is where triage decisions are made and it is tested to a
standard I will defend. The frontend is not, and if you are evaluating this work you should
weight that.

There are 75 cognitive-complexity violations in the codebase. Not zero. They are a ratchet: the
number may fall and never rise, and a commit that raises it fails. I prefer a ratchet to a
target, because a target invites a one-week cleanup sprint and then twelve months of quiet
regression, whereas a ratchet is enforced by the same machine that runs the tests.

Mutation testing: **15 of 15 mutants killed** — sampled and range-scoped, across 6 files,
bounded to one commit range. That is not a survey. It says the assertions in that range have
teeth. It says nothing about the other several hundred files, which have not been measured.

Unit economics are out of scope here: the figure lives in a client-owned file I am not
publishing from, and I would rather leave the argument one number short than quote something I
cannot show you.

## The general form

A green check is a claim about whatever it actually ran against. Before believing it, make it
prove what that was.
