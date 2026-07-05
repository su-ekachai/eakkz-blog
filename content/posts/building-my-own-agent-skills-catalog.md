+++
title = "Building My Own Agent Skills Catalog"
description = "My writing rules and analysis templates used to live in chat history. Now they're a versioned catalog that installs into any coding agent with one command, next to the third-party skills I use daily."
date = 2026-07-04
updated = 2026-07-04
draft = false

[extra]
lang = "en"
toc = true
comment = false

[taxonomies]
tags = ["ai", "agent-skills", "claude", "workflow"]
+++

## The same speech, every session

Every fresh agent session used to start with the same speech: how I like documents structured, what my commit messages look like, which words I never want to see in a draft. The agent would follow along politely, and the next morning a new session would know none of it. My standards lived in chat history, and they evaporated with the scrollback.

Skipping the speech was worse. I once asked a bare agent for a research note and got back an eighteen-section format it invented on the spot. Another time I asked one to make a draft "more human," and it fabricated a detail that was never in the input: a claim that the logs sit in the Actions tab. Plausible, specific, and false.

So the choice was repeating myself forever or trusting output I had already caught lying. I wanted my rules written down once, versioned, and loaded automatically into every session. That's exactly what Agent Skills are for, so I published my own catalog:

> **[github.com/su-ekachai/skills](https://github.com/su-ekachai/skills)**: my writing and analysis standards, packaged as Agent Skills anyone can install.

## The skills I use in my workflow

A skill is a folder with a `SKILL.md` file in it: YAML frontmatter on top, instructions below. The agent reads the description, decides the skill fits the task, and loads the rest. The format is an [open standard](https://agentskills.io), simple enough that people publish whole catalogs of them, and I use other people's skills every day:

- [superpowers](https://github.com/obra/superpowers) brings process discipline: brainstorming before building, test-driven development, systematic debugging, and a strict "no completion claims without fresh verification" rule.
- The [GSD suite](https://github.com/open-gsd/gsd-core) runs my larger projects as phases: discuss, plan, execute, verify, with state that survives between sessions.
- [ponytail](https://github.com/DietrichGebert/ponytail) pushes every solution toward the laziest version that works. It's the skill that argues *against* code.
- [caveman](https://github.com/juliusbrussee/caveman) does for replies what ponytail does for code: cuts them down to the point. I run it in lite mode, terse without going full grunt.
- [taste-skill](https://github.com/Leonxlnx/taste-skill) handles UI design. It's an anti-slop frontend skill that keeps my pages from coming out with the templated look every bare agent produces.
- [ui-ux-pro-max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill), installed straight from the Claude Code plugin marketplace, backs my UI/UX choices with a reference library: 50+ styles, 161 color palettes, 57 font pairings, and 99 UX guidelines.
- [mattpocock/skills](https://github.com/mattpocock/skills) carries my two favorites: grilling, which interviews you relentlessly about a plan until it stops being vague, and handoff, which compacts a session into a brief the next session can pick up.

## The ones I wrote

The gap is that nobody ships a skill for how I write or how I read a company. Those standards are mine, so the skills had to be too. The catalog is MIT licensed and currently holds a writing category and a finance category:

- `technical-writing`: 12 rules for docs, commit messages, and PR descriptions, applied as write mode or as a review that reports rule-by-rule hits.
- `blog-writing`: keeps posts here sounding like me. Its voice profile was calibrated on my own published writing, and it checks drafts against a 47-item list of AI writing patterns before anything ships.
- `fundamental-analysis`: turns "analyze this ticker" into a structured research note: business model, financial health, moat, risks, and fair value from several models (DCF, PEG, Graham, DDM) with a scored verdict.
- `crypto-news-screener`: sweeps crypto news into a deduplicated report, each story scored 1 to 10 for impact with the affected coins and expected direction.

The writing skills guard how I sound. The finance skills encode how I evaluate. Every one of them started as a speech I was tired of giving.

## One command, any machine, any agent

The part I underestimated is what publishing the catalog as a repo buys. Everything installs with one line:

```bash
npx skills@latest add su-ekachai/skills
```

On a new laptop, that command restores my entire standard in seconds. And the installer targets the shared `~/.agents/skills/` directory, then links each tool to it, so the same catalog serves every coding agent I run: Claude Code, Codex, opencode, and Antigravity. That works because Agent Skills is an open standard rather than one vendor's feature. I dropped one CLI for another this year and my skills didn't notice.

The repo cuts both ways, too. Anyone can install my catalog with that one command, the same way I installed superpowers and mattpocock's collection. My writing rules are now the same kind of artifact as the open-source skills I depend on: public, versioned, and one `git log` away from showing exactly when a rule changed.

## Still growing

The catalog holds the skills I needed first; more will land as I keep noticing what I repeat. Every speech I catch myself giving an agent twice is the next skill waiting to be written.
