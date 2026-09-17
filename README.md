## André Lopes Ferreira

AI / agent engineer. Geophysicist by training, federal public servant by day (ANP, posted at
ANM), and for about six months now the product owner of a fleet of coding agents.

I do not architect or author the code. Agents do — worker seats produce ~95% of it, per our own
published engineering standard ([`harness-standards/standards.md` §1](https://github.com/eusoubrasileiro/harness-standards)).
I author the intent, ratify the plan and the validation contract, and own the risk. So the
engineering I actually do is in the contracts, the gates and the evals around the agents, and
that is what these repositories are evidence of — including where it went wrong.

### What I build

- **Validation contracts that can fail.** A mission is not done because a seat exited 0; it is
  done when a held-out validator checks the diff against a `## Done when` section that names a
  mechanism which executes. The 68-entry decision ledger behind that claim is public, and five of
  its entries correct earlier entries in the same file — see
  [the essay](writing/decision-log.md) and `decisions.md` in `agent-factory`.
- **Review surfaces that refuse to flatter.** `eval-viewer` carries a regression suite with one
  block per false claim the dashboard has actually made — a route credited to a node that cannot
  reply, a green check on a run that recorded no verdict. Those blocks are not deleted to make a
  change land.
- **Gates that report honestly when their tool is off.** A metric whose linter rule was never
  enabled read `0` for weeks and the baseline pinned it as a fact; enabling it surfaced 74 real
  violations. The question every new check now has to answer is what it prints when it is
  measuring nothing. [The second essay](writing/eval-gate.md) is about that.

These are working artefacts from a small business, not a portfolio exercise. They have retired
deployments, absent files and open debts, and the READMEs say which.

### Repositories

| Repo | What it is | Look at first |
|---|---|---|
| [agent-factory](https://github.com/eusoubrasileiro/agent-factory) | The mission engine: plan → build → validate → ratify, with per-project data instead of per-project code. | `constitution.md` |
| [harness-standards](https://github.com/eusoubrasileiro/harness-standards) | The written engineering standard the agents are held to — dual gates, permission tiers, worktree dispatch. | `standards.md` §4 |
| [eval-viewer](https://github.com/eusoubrasileiro/eval-viewer) | An offline dashboard for reviewing LLM agent eval runs by hand, with a suite pinning every lie it once told. | `test/honesty.test.ts` |
| [knowledge-engine](https://github.com/eusoubrasileiro/knowledge-engine) | Durable external memory: watchers sift AI research into a knowledge repo; an MCP server serves it back, grounded and dated. | `serve.py` |
| [whatsapp-mcp](https://github.com/eusoubrasileiro/whatsapp-mcp) | WhatsApp as an MCP server — 23 tools, Bearer auth, long-lived Docker daemon. | `src/stream/follow.ts` |

### Writing

- [**Six months of agent-built software: what 68 decisions say**](writing/decision-log.md) — a
  reading of an append-only decision ledger, concentrating on the entries where the process was
  wrong and said so.
- [**An eval gate on real WhatsApp traffic**](writing/eval-gate.md) — why a check that cannot fail
  is worse than no check, and what that costs to fix.

### Before this

~20 years writing code, 15 of them Python, at the intersection of physical science and
high-performance computing: five years at Schlumberger on production geoscience software, then a
decade-plus automating technical and GIS workflows inside Brazilian energy and mining regulation.
Contributor to [Fatiando a Terra](https://www.fatiando.org/) since 2012, plus GeographicLib and
`geonumerics`. On Stack Overflow as [`imbr`](https://stackoverflow.com/users/1207193/imbr): 7.9k
reputation, 85 answers over 14 years, 5.4M people reached, top answer at 197 votes; NLP Collective
member.

The regulatory and scientific-computing background is not decoration here. Most of what agent
engineering turns out to need — evidence, audit trails, and a definition of "done" written before
the work starts — is the same discipline, applied to a faster machine.

### Contact

eusoubrasileiro@gmail.com · [GitHub](https://github.com/eusoubrasileiro) ·
[Stack Overflow](https://stackoverflow.com/users/1207193/imbr) · LinkedIn
