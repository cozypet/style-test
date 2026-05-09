Anthropic Shipped Outcomes. Real Story Is Verification Becoming a SKU.
Full draft. 5 sections, 11 diagrams, ~3,500 words, ~14 minute read.



You have written this loop before. Eighteen months ago, when you first put a Claude agent into production, you wrote a rubric. You wrote a grader. You wrote retry logic for when the grader said no. The pieces broke. You patched them. The rubric drifted. You rewrote it.

On May 6 at Code with Claude San Francisco, Anthropic shipped your loop as an API endpoint and called it Outcomes.

That is the news. The story underneath it is bigger. Outcomes is the first harness layer Anthropic decided to sell. Dreams, Multi-Agent, and Webhooks are the same move on memory, orchestration, and lifecycle. The harness used to be code you wrote. It is becoming a stack of products you compose.


Before We Start! 🦸🏻‍♀️
If this helps you ship better AI systems: 👏 Clap 50 times (yes, you can!). Medium's algorithm favors this, increasing visibility to others who then discover the article. 🔔 Follow me on Medium, LinkedIn and subscribe to get my latest article.


What Outcomes Actually Is
Anthropic shipped four things into Managed Agents on May 6. Three are public beta: Outcomes, Multi-Agent Orchestration, and Webhooks. One is research preview: Dreams. There was no new model that day. Dario Amodei opened the keynote with the line directly: "No new model today. Today is about how we are making our products work better for you."

The press picked the marquee feature, which was Dreams. Agents that consolidate memory between sessions, named to evoke human sleep, with a six-fold completion-rate improvement at Harvey to anchor the demo. It got the headlines. It deserved them. But the architecturally significant feature in the same release is Outcomes, and on the surface it does not look like much. You write a rubric. The agent works. A grader scores the work. If it falls short, the agent tries again.

Five hundred lines of code for any team that has shipped a serious agent. Four lines of API for everyone using Managed Agents.

[DIAGRAM 1: Outcomes is a loop, not a feature]

The rubric is a markdown document. You describe what success looks like. The agent does the work. When the agent thinks it is done, a separate Claude instance receives the output, reads your rubric, and returns one of five terminal states.

The five states are: satisfied (ship it), needs_revision (here is what to change, take another pass), max_iterations_reached (you ran out of attempts), failed (the grader could not evaluate), and interrupted (something killed the run).

The piece worth pausing on is the separation. The grader runs in its own context window. It does not see the agent's reasoning. It does not see the chain of tool calls the agent took to produce the output. It only reads the rubric and the output. This is deliberate. If the grader could see the agent's chain of thought, it would be biased toward saying yes. The agent's reasoning is persuasive. The output, on its own, often is not. Evaluation accuracy depends on isolation. So Anthropic isolates.

The loop runs up to three times by default. Maximum is twenty. You can write the rubric to relax or tighten on each pass. Outcomes is public beta as of May 6, behind the beta header managed-agents-2026-04-01. No separate access request, unlike Dreams.

[DIAGRAM 2: The outcome lifecycle, as events]

That is the abstract view. In practice a real outcome run emits more events than five, including the agent's working messages and a fresh grader span on each iteration. The two-iteration version looks like this:

[DIAGRAM 3: The full lifecycle, with iterations]

Anthropic released two sets of numbers. Internal benchmarks first: Outcomes improves task success by ten percentage points compared with standard prompting. The gain is largest on hard problems, the kind where a single pass through Claude is going to miss something subtle. On structured file generation specifically, success rises 8.4 percent on docx output and 10.1 percent on pptx. These are workloads where attention to detail matters most: rubric criteria can encode formatting rules a generic prompt would miss.

Customer numbers second. Wisedocs uses Outcomes to review documents against internal guidelines. Reviews now run 50 percent faster while staying aligned to team standards. The 50 percent is not faster Claude. It is the absence of the brittle harness Wisedocs would otherwise be maintaining. Spiral by Every uses Outcomes alongside Multi-Agent: a lead agent receives a writing request, sub-agents generate parallel drafts, and Outcomes scores each draft against editorial principles and the user's voice. The agent ships whichever draft scored highest. Spiral is delivering this through API and CLI, productized.

None of this looks like a new technique. The evaluator and the worker are the same pattern every team running production agents has been writing by hand for two years. That is the point. Outcomes is the codification of a practice. The interesting question is not what it does. It is why Anthropic decided to ship the practice as an API endpoint now.


The Pattern That Was Sitting in Anthropic's Own Engineering Blog
The pattern is not new. Outcomes is the codification of work Anthropic published seventeen months ago.

In December 2024, Erik Schluntz and Barry Zhang at Anthropic published "Building Effective Agents." The post describes five workflow patterns for building agents: prompt chaining, routing, parallelization, orchestrator-subagents, and one called the evaluator-optimizer. The description is one sentence:

"In the evaluator-optimizer workflow, one LLM call generates a response while another provides evaluation and feedback in a loop."

Read that sentence twice. Then read the Outcomes documentation. It is the same architecture. One Claude generates the output. A second Claude evaluates the output against criteria and provides structured feedback. The loop continues until the evaluator is satisfied or maximum iterations are reached. Anthropic published the pattern. Anthropic published a Jupyter notebook reference implementation in their cookbook. The Spring AI team turned it into a class called EvaluatorOptimizerWorkflow. The Hatchet team turned it into a workflow primitive in Icepick. CrewAI, LangGraph, and DSPy all support some variation. The pattern has been in production for seventeen months across thousands of teams.

[DIAGRAM 4: Same pattern. Different decade.]

If you have rolled your own, the recipe is familiar. You define a rubric in code or as a prompt template. You wrap the agent's main task in a function that returns its output. You write a second prompt that takes the output and the rubric, asks an evaluator instance to score it, and returns either an approval or specific feedback. You wrap the pair in a loop with a counter so the system does not spin forever. You add error handling for when the evaluator's structured output drifts. You add observability so you can see why the loop did not converge in production. You ship it. Six months later you rewrite it because the model got smarter and your prompts no longer hold up. Twelve months later you rewrite it because someone on your team decided to use Pydantic instead of JSON schemas. The pattern is not the hard part. The pattern is fine. The maintenance is the hard part.

Frameworks help, but they make a different trade. CrewAI gives you the loop but couples you to the framework's worldview. LangGraph gives you a graph runtime but you still write the rubric and own the eval prompts. Each framework promises portability and ships lock-in by another name. None of them give you what Outcomes gives you: a Claude grader running on Anthropic's infrastructure, isolated by default, with managed iteration limits and standard event tracing, where the only thing you write is the markdown rubric.

[DIAGRAM 5: Three places the eval loop can live]

So the technical question is uninteresting. Anthropic is selling a pattern they themselves wrote about seventeen months ago. The question that matters is structural: why is the evaluator-optimizer the first pattern to graduate from blog post to SKU? Why not prompt chaining? Why not routing? Why is verification the layer Anthropic decided to capture as managed product first?

The answer is not in the Outcomes documentation. The answer is in a different Anthropic engineering post, published one month before Code with Claude, that almost no one read.


Why Outcomes Could Ship: Brain, Hands, Session
The May 6 announcements were not a roadmap. They were a tax on architecture work Anthropic finished in April.

On April 8, Anthropic published an engineering post titled "Scaling Managed Agents: Decoupling the brain from the hands." Almost no one outside Anthropic linked to it. The post does not announce a feature. It announces a rebuild. And it is the only document that explains why Outcomes, Dreams, Multi-Agent, and Webhooks all shipped in the same release one month later.

The post describes how Managed Agents was rebuilt around three primitives.

The first is the session. An event log that only appends. You read it via getEvents(). The harness writes to it via emitEvent(). Tool calls, tool results, model responses, user messages, configuration changes, all of it goes through the session. The session is durable. If a process crashes during a task, the session does not. You can recover state by replaying events.

The second is the brain. Claude itself plus the harness loop that calls Claude and routes Claude's tool calls. The brain is intentionally stateless from the harness perspective. Recovery is wake(sessionId): spin up a new brain, hand it the session, let it pick up where the last brain left off. The brain knows nothing the session does not record.

The third is the hands. The sandboxes, MCP servers, and tools where code actually executes. Hands look uniform from the brain's perspective: execute(name, input) → string. Whether the underlying tool is a code interpreter, a browser, a filesystem operation, or an MCP connected SaaS, the brain calls the same interface and the hand returns the same shape. Provisioning works through provision({resources}), which spawns containers on demand.

[DIAGRAM 6: Three primitives. One bus.]

Anthropic uses an OS analogy to explain the design goal. They reach back to the 1970s. The read() and write() system calls in Unix worked across dozens of generations of hardware because the abstractions sat at the right level: above the disk controller, below the application. Programs from 1975 still run on hardware from 2026 because the file system call interface stayed stable. Anthropic wants the same property for agents. The brain, hands, and session interfaces are designed to outlast specific Claude versions and specific harness implementations. They will probably outlast Claude Code itself.

The rebuild shipped with concrete performance numbers. Anthropic disclosed time to first token improvements: p50 dropped 60 percent. p95 dropped over 90 percent. The number that matters is p95: the long tail of agent runs where things go wrong is where users feel pain. By moving session state to durable storage and making the brain stateless, Anthropic removed the recovery cliff. A crashed agent now restarts in milliseconds. The old architecture would have lost the run.

That is the engineering story. Here is why it matters for Outcomes.

In the old coupled architecture, the eval loop was your problem because there was no shared substrate to attach a second evaluator to. Your agent's brain held the session in memory. Your tool calls happened inside the same process. To grade the output, you wrote a second prompt in the same code path, ran it through the same Claude API, and routed the result back to the same loop. The grader could not be a separate Claude instance with its own context window because there was no abstraction that would let two brains read the same session without colliding.

The April 8 decoupling changes that. The session is durable and queryable. Two brains can read the same session. One brain can be the worker. A second, isolated brain can be the grader.

[DIAGRAM 7: Outcomes is a second brain on the same bus]

That is what Outcomes is, mechanically. A second brain wired to your session log, gated by a rubric you define. The grader reads the worker's output from the session, scores it, and writes a new event back. The worker brain, which has been waiting, reads the new event and decides whether to continue. The whole loop runs server side because the substrate now supports it.

So the May 6 announcement was not a roadmap. The roadmap had already shipped on April 8. May 6 was the tax: four features that all depend on the same brain, hands, and session decoupling, each capturing a different layer of what used to be your code.


The Verification Layer Is the First SKU. Dreams and Multi-Agent Are the Same Move.
The harness is being unbundled in real time. The four features Anthropic shipped on May 6 are not four features. They are four layers of what used to be your eval and orchestration code, each extracted into a separate product on the same brain, hands, and session substrate.

Verification went first. Outcomes turns rubric grading into a managed loop. You write the markdown. Anthropic runs the grader brain. The benchmark numbers (+10pp on hard tasks, +8.4 percent on docx, +10.1 percent on pptx) and customer wins (Wisedocs 50 percent faster, Spiral parallel drafts) anchor it. Public beta as of May 6, no separate access request.

Memory curation went second, as research preview. Dreams is a scheduled brain that reads the existing memory store and up to 100 past sessions, then writes a new reorganized memory store. The input is never modified. You can review the dream output before it lands, or let it apply automatically. Dreams runs claude-opus-4-7 or claude-sonnet-4-6 and ships behind a separate beta header dreaming-2026-04-21. Harvey reports a sixfold improvement in completion rate on legal drafting once their agents started carrying institutional memory across sessions instead of relearning filetype workarounds every run. The architectural significance: memory consolidation becomes async and reviewable, not written on the fly. Enterprises can audit memory updates before agents adopt them.

Orchestration went third. Multi-Agent lets a lead brain delegate to up to twenty parallel specialist brains, each with its own context window. The lead receives the original task, splits it, dispatches, and aggregates. The Claude Console shows what each sub-agent did, so the work stays inspectable. Netflix uses this to analyze logs across hundreds of build sources in parallel, surfacing only patterns that recur. Spiral combines it with Outcomes: lead agent receives a writing brief, sub-agents generate parallel drafts, Outcomes scores each against editorial criteria, the lead ships the best one. Two harness layers composed cleanly because they share a substrate.

Lifecycle hooks went fourth. Webhooks let external systems subscribe to session events. You define an outcome, agent runs, your endpoint receives notification when work completes. No polling. The architectural read: the session log is now an external event stream you can plug into. The harness is no longer a black box from which you fish results.

[DIAGRAM 8: Four harness layers. Four SKUs.]

Each of these used to be your code. The eval loop was your while not satisfied block. The memory consolidation was your nightly cron job summarizing yesterday's sessions. The orchestrator was your coordinator script with subprocess calls. The webhooks were your ad hoc polling. None of it was glamorous. All of it was structural. All of it broke every quarter when something upstream changed.

The May 6 release names what is happening: each harness layer is becoming a separately purchased product. You compose them. You do not build them.

This is why Outcomes specifically went first. Of the four, verification is the cleanest layer to extract. Memory has policy and audit complications. Orchestration has cost control complications. Webhooks are infrastructure, not intelligence. Verification is the layer where the value of the substrate (a separate Claude that does not see your worker's reasoning) is most visible and the failure modes are most quantifiable. Anthropic shipped the layer with the clearest standalone value first, then bundled the rest.

[DIAGRAM 9: Harness as code, then as product line]

The cumulative effect is not new pricing. It is a new product shape. Look at where Anthropic spent engineering effort over the last twelve months. Claude Code, the Claude Agent SDK, Managed Agents launch in April, the April 8 brain, hands, and session rebuild, May 6 layer extractions, ten finance agent templates that ship as Cowork plugins, Code plugins, and Managed Agents cookbooks all at once. The trajectory is consistent: the agent template is becoming the unit of distribution, and the harness is becoming infrastructure underneath. Two years ago, the thing you bought from Anthropic was a model. One year ago, the thing you bought was a model and an SDK. Today, the thing you buy is a stack of harness layers you compose around a model. Tomorrow, agent templates may matter more than which Claude version is underneath.

That is what "Outcomes is a SKU" actually means. Not that one feature got a price tag. That the layer beneath every production agent shifted from your codebase to Anthropic's product catalog.


What Builders Should Stop Writing
The right move is not to adopt Managed Agents. The right move is to delete code.

Three concrete subtractions, in order of leverage.

Stop writing eval loops. Start writing rubrics. Outcomes does the loop, the grader, the iteration cap, the structured feedback, the retry logic, the observability. Your job becomes specifying success criteria in markdown that another Claude can grade against. The skill that wins is precise rubric writing: explicit acceptance conditions, clear rejection conditions, edge cases articulated. This is closer to product spec writing than to engineering. Most teams will discover their existing rubrics are vague enough that the grader will need three iterations to converge. That is your signal to write better rubrics, not to abandon the feature. Lowest commitment: try Outcomes on a single agent task this week. The cost is the runtime hour at $0.08 plus standard token rates.

Stop building memory consolidation pipelines. Start scheduling Dreams. Most teams running agents that operate over hours or days have a homegrown memory layer that became a liability six months in. The notes pile up. Yesterday's debugging insights contradict last week's architectural decisions. The agent ends up either drowning in stale memory or losing context every restart. Dreams is the scheduled cleanup that human brains do during REM sleep, made explicit. Read past sessions, extract patterns, write a new reorganized store, leave the input untouched. The audit trail is real: enterprises can review what changed before agents adopt the new memory. Dreams is research preview only, so you have to request access. The trade is patience: takes minutes to tens of minutes per dream cycle, runs claude-opus-4-7 or claude-sonnet-4-6 at standard token rates.

Stop running orchestrators. Start defining sub-agent specs. If your agent already coordinates parallel work, you have an orchestrator. It probably uses asyncio, has a custom dispatch table, and routes around model rate limits with homemade backoff. Multi-Agent replaces the orchestration code. You define what each sub-agent does, how they connect, what the lead aggregates. Up to twenty parallel specialists per session. The Console gives you the per-agent inspection that took you three sprints to build internally. The constraint: this is the most opinionated layer. You inherit Anthropic's view of what multi-agent coordination should look like. If your existing patterns disagree, this is harder to migrate.

[DIAGRAM 10: Where to start. The adoption ladder.]

The honest tradeoff, named.

The hard constraint is model. Managed Agents only runs Claude. There is no path to running GPT or Gemini or open weights through this harness. If your stack is multi model today or you are likely to need that flexibility in 18 months, you are picking a different bet. Frameworks like CrewAI and LangGraph let you swap providers and survive a multi cloud strategy. Outcomes does not. That is the price of letting Anthropic carry the maintenance.

The soft constraint is data. Sessions, memory stores, dreams, and grader contexts all live on Anthropic's infrastructure. For some workloads (regulated finance, defense, healthcare) this is a deal breaker. For others, the SOC 2 and ISO posture and the audit log in the Claude Console are good enough. Worth verifying with your compliance team before adopting at scale.

The soft constraint nobody flags is API surface. The interfaces (define_outcome, getEvents, wake, provision) are stable per Anthropic's design intent (the OS analogy from the engineering post). But nothing is forever. If Anthropic redraws the harness boundary in two years, your migration cost is real. Manage exposure by keeping your rubrics, agent templates, and tool definitions in your own repo. Treat Anthropic's harness as the runtime, not as the source of truth.

[DIAGRAM 11: What you keep. What Anthropic owns.]

Adoption order, ranked by leverage divided by commitment:

Outcomes first. Highest leverage, lowest commitment. One markdown file plus the API call. You can pilot on a single workflow and roll back in an afternoon. The maintenance burden you save pays for the constraints you accept.

Dreams second, when memory rot is real. Do not adopt prophylactically. Wait until your agent's memory store has become a known liability and you have data on the cost of staleness. Then schedule a dream and audit the diff before letting it land.

Multi-Agent last, on greenfield workloads. Migrating an existing orchestrator to Multi-Agent is a rewrite. Better to use Multi-Agent for a new agent system where you are not undoing prior architectural decisions. The ceiling of twenty parallel agents is high enough for most uses but plan around it.

What you should keep writing in your own code: the rubrics, the agent templates, the tool definitions, the prompts, the business logic. Everything that encodes what your product does, not how the harness runs it. Those are your moat. The eval loop never was.


Credits & Further Reading
Anthropic primary sources

"New in Claude Managed Agents: dreaming, outcomes, and multiagent orchestration" at claude.com/blog/new-in-claude-managed-agents (May 6, 2026). The launch announcement.
"Scaling Managed Agents: Decoupling the brain from the hands" at anthropic.com/engineering/managed-agents (April 8, 2026). The architectural rebuild that made everything in this article possible.
"Building Effective Agents" by Erik Schluntz and Barry Zhang at anthropic.com/research/building-effective-agents (December 2024). The original evaluator-optimizer pattern that became Outcomes.
Outcomes API documentation: platform.claude.com/docs/en/managed-agents/define-outcomes
Dreams API documentation: platform.claude.com/docs/en/managed-agents/dreams
Multi-Agent documentation: platform.claude.com/docs/en/managed-agents/multi-agent
Anthropic agent cookbook: github.com/anthropics/anthropic-cookbook

Customer case studies referenced

Harvey (legal AI): 6x completion rate improvement with Dreams enabled.
Wisedocs (document review): 50 percent faster reviews with Outcomes.
Spiral by Every (writing tool): parallel drafts with Multi-Agent + Outcomes.
Netflix (platform engineering): log analysis with Multi-Agent across hundreds of build sources.

Independent coverage worth reading

Simon Willison's live blog of the Code with Claude SF keynote at simonwillison.net/2026/May/6/code-w-claude-2026.
Blake Crosley's source-disciplined recap at blakecrosley.com/blog/code-with-claude-sf-2026-recap.

Cross-tool context

Cursor's "Continually improving our agent harness" (April 30, 2026) for the parallel industry adoption of harness vocabulary, with a different ownership model.



If this helped you ship better AI systems, clap 50 times so others find it. If it did not, tell me why in the comments. I read everything.

