## André Lopes Ferreira

AI / agent engineer. Geophysicist by training, federal public servant by day (ANP, posted
at ANM), and for about six months the product owner of a fleet of coding agents.

**Agents write 100% of the code here. I write none of it.** What I do instead: define what
"done" means before the work starts, write the validation contract, own the gate that decides
whether a diff ships, and accept the risk. The repositories below are the tooling that makes
that safe. The most useful thing in them is the record of where it failed.

Three of those, concretely:

- **A security cage with zero callers.** It rendered a correct, audited permission config. It
  had a test. Its header comment named the driver that called it — and that driver did not
  exist. For three weeks the `git push` and `~/.ssh` denials were never enforced on the seat
  doing the real work. `agent-factory/decisions.md`, D-16.
- **A quality gate that scored a perfect zero.** Biome's `noExcessiveCognitiveComplexity` is
  not in its `recommended` set, and the config never enabled it. So a WhatsApp lead-triage
  backend reported `0` complexity violations for two months, and the checked-in baseline
  pinned that zero as a fact. Turning the rule on surfaced 74. `harness-standards/standards.md`
  §4.
- **A dashboard crediting a reply to a node that cannot reply.** `eval-viewer` displayed
  *answered by: `handoff_sink`* — a LangGraph node that returns `replyText: null`.
  `test/honesty.test.ts` now carries one block per false claim the UI has made, and those
  blocks are not deleted to make a change land.

The shape is the same in all three: a check that cannot fail reports success forever, and
every green run after that looks exactly like evidence. The first question any new metric has
to answer here is what it prints when its tool is off.

### Repositories

| Repo | What it is | Look at first |
|---|---|---|
| [agent-factory](https://github.com/eusoubrasileiro/agent-factory) | The mission engine: plan → build → validate → ratify, with per-project data instead of per-project code. | `constitution.md` |
| [eval-viewer](https://github.com/eusoubrasileiro/eval-viewer) | Offline dashboard for reviewing LLM agent eval runs by hand, with a suite pinning every false claim it has made. | `test/honesty.test.ts` |
| [harness-standards](https://github.com/eusoubrasileiro/harness-standards) | The written engineering standard the agents are held to — dual gates, permission tiers, worktree dispatch. | `standards.md` §4 |
| [whatsapp-mcp](https://github.com/eusoubrasileiro/whatsapp-mcp) | WhatsApp as an MCP server — 23 tools, Bearer auth, long-lived Docker daemon. | `src/stream/follow.ts` |
| [knowledge-engine](https://github.com/eusoubrasileiro/knowledge-engine) | Durable external memory: watchers sift AI research into a knowledge repo; an MCP server serves it back, grounded and dated. | `serve.py` |

### Writing

- [**What an agent-written decision log records**](writing/decision-log.md) — the agents keep
  their own append-only ledger. 68 entries, five of which correct earlier entries.
- [**An eval gate on real WhatsApp traffic**](writing/eval-gate.md) — why a gate that grades a
  copy of production is evidence about the copy.

### Before this

~20 years writing code, 15 of them Python, at the intersection of physical science and
high-performance computing: five years at Schlumberger on production geoscience software, then
a decade-plus automating technical and GIS workflows inside Brazilian energy and mining
regulation. Contributor to [Fatiando a Terra](https://www.fatiando.org/) since 2012. On Stack
Overflow as [`imbr`](https://stackoverflow.com/users/1207193/imbr) — 7.9k reputation, 85
answers, 5.4M people reached. Evidence, audit trails and a definition of "done" written before
the work starts are the same discipline, applied to a faster machine.

### Contact

eusoubrasileiro@gmail.com · [GitHub](https://github.com/eusoubrasileiro) ·
[Stack Overflow](https://stackoverflow.com/users/1207193/imbr) ·
[LinkedIn](https://www.linkedin.com/in/andr%C3%A9-ferreira-lopes/)
