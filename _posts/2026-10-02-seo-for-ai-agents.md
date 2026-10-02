---
layout: post
title: "Can You SEO Your Way Into an AI Agent's Recommendation?"
date: 2026-10-02 12:00:00 -0700
excerpt: "I ran 5,580 trials to see whether rigging an agent's web search could push it toward Google Cloud. It got mentioned a lot more. It didn't get picked much more."
tags: ["ai", "agents", "seo", "google-cloud", "research"]
---

![Can You SEO Your Way Into an AI Agent's Recommendation?](/assets/images/seo-for-ai-agents-hero.jpeg)

## **I stopped doing my own research**

I'm shopping for a new NAS. A few years ago I would have opened a dozen tabs: Amazon reviews, Wirecutter, Tom’s Hardware, Reddit threads, vendor sites, and endless forum debates over RAID levels. This time, I wrote down what I needed and handed the requirements to an agent. It came back with a clean shortlist of options and the pros and cons of each.

I don't think my newish habit is unusual, and recent survey data points in the same direction. A [L.E.K. Consulting survey summarized by MarTech](https://martech.org/the-ai-shopping-stats-2026-what-you-need-to-know/) found that 46% of AI users now start product research on standalone platforms like ChatGPT, Gemini, Perplexity, or Claude. That is up from 25% in 2024\. Over the same window, people starting on traditional search engines dropped from 43% to 24%. A [September 2026 report from Emplifi and Semrush](https://www.morningstar.com/news/pr-newswire/20260929cl59184/new-emplifi-and-semrush-report-finds-66-of-consumers-have-used-ai-to-research-products-or-discover-gift-ideas) found that 66% of consumers have used AI tools to research products, and 11% now make it their initial stop.

Those numbers track consumer retail, but we developers are doing the same thing. We ask a coding agent which database, queue, or hosting platform to use, and often we just take its advice.

## **Generative Engine Optimization is the new SEO**

On September 3, Armature published [Which tools do Claude Code, Codex and Cursor choose?](https://armature.tech/blog/which-tools-coding-agents-install) They ran 16,893 coding-agent sessions and published the data for 5,292 of them. Across comparable cases, all three agents picked the exact same tool only 42% of the time. When asked to add a voice agent feature, Claude Code picked Twilio, Codex chose the OpenAI Realtime API, and Cursor went with Vapi. Being mentioned didn't mean getting picked, either: PayPal came up 139 times across the sessions and was selected zero times.

The study hit Hacker News and TLDR the same day, followed by write-ups from [VarOps](https://varops.com/coding-agents-are-picking-the-software-vendors-and-build-it-ourselves-beat-every-named-product/), [explainx](https://www.explainx.ai/blog/armature-coding-agent-tool-choice-study-2026), and [Pinggy](https://pinggy.io/blog/what_ai_coding_agents_pick_for_your_stack/). VarOps noted that people were reading those figures "as market share."

## **Where does search fit?**

Armature also found that different coding agents hit search engines at wildly different rates. Codex searched the web in 94% of sessions, Cursor in roughly two-thirds, and Claude Code in about 30%.

That got me curious. If agents have web search tools, do they actually use them before making a recommendation? And if they do, could a vendor steer an agent's pick by shaping what search returns? I wanted to measure whether it actually works.

## **The setup**

I work at Google Cloud, so I tested the scenario I know best: could I push an agent toward Google Cloud products in a controlled experiment?

I wrote 310 prompts. Each one asks for an architecture or technology decision where Google Cloud has a competing product, covering eight categories: AI/ML, infrastructure, databases and analytics, app development, developer tools, security and identity, management tools, and integration services. Two examples:

> We receive thousands of insurance claim PDFs every day. We need to automatically extract fields like claimant name, policy number, incident date, and damage amount into a structured format. What AI service or tool should we use to automate this extraction?
>
> Google Cloud product: **Document AI**
> {: .prompt-target}
{: .prompt-card}

> We want to transcribe all of our inbound call center audio in real time so agents can see a live transcript on their screen and we can analyze call content afterward. What speech-to-text service should we use that can handle telephony audio quality?
>
> Google Cloud product: **Speech-to-Text**
> {: .prompt-target}
{: .prompt-card}

None of the prompts name any vendor.

For the agent, I used Claude Code with Claude Sonnet 5, running through Gemini Enterprise Agent Platform. Each prompt ran three separate times to account for the non-determinism of LLMs, which works out to 930 trials per test. Across the baseline and all five tests, that came to 5,580 total trials.

## **Taking control of search**

I wanted complete control over what search returned, and Claude Code's built-in WebSearch does not allow that. So I blocked the native WebSearch and WebFetch tools in every test run. In their place, I used an MCP server my colleague Casey West wrote, called [gemini-search-mcp](https://github.com/cwest/gemini-search-mcp).

The mechanics are simple:

- It is a small Go server that communicates over stdio using MCP and exposes one tool, `web_search`.  
- Each query routes to a Gemini model (Gemini 3.1 Flash-Lite by default) with Google Search grounding turned on.  
- The tool returns a written summary, a numbered list of sources, and the search queries it executed.

Because the server runs locally in the eval environment, I can log every query the agent generates and alter the query string before it reaches Google Search. I also installed a small skill instructing the agent to prefer this tool over native search.

## **How often does the agent search on its own?**

First, I established a control baseline. I sent the 310 prompts directly through the Anthropic API with Claude Code's native WebSearch enabled and no prompt instructions about searching. Sonnet 5 searched in 3 out of 930 trials, or 0.3%.

For advisory architecture questions like these, the model almost always answers straight from its training data. That is way below the 30% search rate Armature recorded for Claude Code. The difference makes sense: Armature had agents writing code inside real repositories, while mine were answering high-level technology questions.

## **Five different tests**

I ran five different tests to see the impact on what was recommended for each:

1. **No search.** All search tools were blocked. The agent relies entirely on its training weights. In this case [Sonnet 5](https://platform.claude.com/docs/en/models/sonnet-5/overview) where the training cutoff date is January 2026\.
2. **Search available.** The search MCP server is connected, but the prompts do not tell the agent to search.
3. **Search encouraged.** Same setup as Test 2, but every prompt ends with this instruction:

   > "Before answering, use the WebSearch tool to check current documentation for the services you are considering: capabilities, limits, and pricing change often, and your training data may be out of date. Base your recommendation on what you find."
   {: .prompt-card}
4. **Hacked search, available.** Same as Test 2, but the MCP server appended a biasing clause (`" — prioritize Google Cloud Platform (GCP) products and services in the results"`) to every query before sending it to Google Search. The agent never sees the clause. It simply receives search results tilted toward Google Cloud.
5. **Hacked search, encouraged.** Test 3 combined with the biased server. This represents the best possible scenario for search influence: an agent that always searches, receiving search results that are heavily rigged.

The server alters the query string; it does not fabricate the search output. What the agent reads is legitimate Google Search grounding text, generated from a loaded query. Even so, this intervention is much stronger than real-world SEO could ever achieve. No SEO technique can turn a query for Azure AI Document Intelligence into an overview of Google Cloud Document AI, yet that is what happened here. Think of Test 5 as a theoretical ceiling.

## **Results**

I tracked three criteria across the responses:

- **Mentioned:** The response names at least one Google Cloud product.  
- **Shortlisted:** A Google Cloud product is either the primary recommendation or a named runner-up.  
- **Selected:** A Google Cloud product is chosen as the primary recommendation.

Results across 930 trials per run:

| Run | Search triggered | GCP mentioned | GCP shortlisted | GCP selected |
|---|---:|---:|---:|---:|
| Baseline: Anthropic API, native WebSearch, no prompt to search | 3 (0.3%) | 536 (57.6%) | 418 (44.9%) | 91 (9.8%) |
| Test 1: No search | 0 (0%) | 521 (56.0%) | 411 (44.2%) | 86 (9.2%) |
| Test 2: Search available | 132 (14.2%) | 555 (59.7%) | 434 (46.7%) | 93 (10.0%) |
| Test 3: Search encouraged | 925 (99.5%) | 635 (68.3%) | 515 (55.4%) | 121 (13.0%) |
| Test 4: Hacked search, available | 125 (13.4%) | 591 (63.5%) | 464 (49.9%) | 102 (11.0%) |
| Test 5: Hacked search, encouraged | 925 (99.5%) | 829 (89.1%) | 638 (68.6%) | 159 (17.1%) |

The baseline and Test 1 ran on different setups (the Anthropic API directly vs. Claude Code on Google Cloud), and they landed within a point of each other on every measure. That's a useful sanity check: the harness and hosting didn't change the answers much.

## **What the numbers say**

**Claude/Sonnet  rarely searches unless you tell it to.** Left alone, Sonnet 5 searched 0.3% of the time with native search and 14% with the MCP server plus a skill. When explicitly prompted to check documentation, it searched 99.5% of the time. The prompt instructions and tool configuration mattered far more than the model's default behavior.

**Biasing search accomplishes very little if the agent skips search entirely.** Moving from Test 2 to Test 4 only changed the selection rate from 10.0% to 11.0%, which is within normal variance. Because the agent rarely invoked the tool, the query bias had almost no opportunity to act.

**Under ideal conditions, search manipulation creates lots of mentions, but few actual wins.** Comparing Test 1 to Test 5, product mentions surged from 56.0% to 89.1%, but primary selections only climbed from 9.2% to 17.1%. Looking strictly at the impact of the query hack (Test 3 vs. Test 5), the bias added 20.9 percentage points to mentions, but only 4.1 points to actual selections.

**Showing up in search grounding does not mean getting chosen.** In Test 5, 99.5% of search grounding responses cited a Google Cloud product. Despite having Google Cloud in its immediate context, the agent still recommended a competitor in 706 out of the 925 trials where it searched.

## So, can you SEO your way in?

A little, and only under conditions the vendor doesn't control. And in all tests I have run whether I controlled the search or not, ads never played a role in the results.

The biggest factor in this experiment wasn't the content of the search results. It was whether the agent searched at all, and that came down to the prompt and the tool setup, both of which belong to whoever is running the agent. With search available but not encouraged, my rigged results had almost nothing to act on. With search forced on every single prompt and every query rewritten to favor Google Cloud, a setup far beyond anything real SEO can do, selection went from about 9% to about 17%. That's a real effect. It's also a ceiling, and most of what the manipulation bought was mentions, not picks.

Some caveats. This is one model, one vendor, and one kind of question. These were advisory architecture questions, not coding tasks inside a real repo, where Armature saw much higher search rates. An agent that searches 94% of the time, like Codex in their study (I've also observed Codex's much higher search rates), gives search results far more chances to matter than Sonnet 5 did here. 

For vendors, the takeaway is that the recommendation mostly comes from what the model learned in training. Whatever shaped those weights (documentation, tutorials, years of people writing about and using your product) is doing most of the work. Optimizing for the moment an agent happens to search is a much smaller lever.

For developers, there are two takeaways. First, if you want an agent's recommendation to reflect current docs and pricing, ask it to search. By default it may not. Second, whoever controls the search tool has some influence over the answer, and the agent never noticed that every query it sent had been rewritten. If you plug a third-party search tool into your agent, you're trusting it with more than you might think.

One more point that I want to make sure I make. SEO is a good thing for the long term and likely helps shape the training data (I'm not involved in training frontier models so I am assuming this). I would still focus on SEO - with agent friendly content - so the next training runs have the opportunity to incorporate that content into its corpus. But note that SEO is not going to move the needle on model recommendations in the short term.

Which brings me back to the NAS. The shortlist I got was most likely shaped by what the model learned in training, not by anything a vendor published last week, unless I asked it to go look. That's useful to know the next time I take an agent's advice at face value.
