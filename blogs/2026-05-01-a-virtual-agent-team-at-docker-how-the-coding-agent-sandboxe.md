---
title: "A Virtual Agent team at Docker: How the Coding Agent Sandboxes team uses a fleet of agents to ship faster"
url: "https://www.docker.com/blog/a-virtual-agent-team-at-docker-how-the-coding-agent-sandboxes-team-uses-a-fleet-of-agents-to-ship-faster/"
date: "Fri, 01 May 2026 13:00:00 +0000"
author: "Jennifer Kohl"
feed_url: "https://www.docker.com/feed/"
---
<p>I work on Coding Agent Sandboxes, aka “sbx” at Docker. The project provides secure, <a href="https://docs.docker.com/ai/sandboxes/" id="dkr_microvm-based-isolation-88831" rel="nofollow noopener" target="_blank">microVM-based isolation</a> for running AI coding agents like Claude Code, Gemini, Codex, Docker Agent and Kiro. Agents get full autonomy inside a sandbox (their own Docker daemon, network, filesystem) without touching your host system. Over the past couple of weeks, we built something on top of it: a virtual team of seven AI agent roles that test the product, triage issues, post release notes, and even fix bugs, all running autonomously in CI. We call it the Fleet.</p>



<p>The Fleet is built on <a href="https://code.claude.com/docs/en/skills" id="dkr_claude-code-skills-88831" rel="nofollow noopener" target="_blank">Claude Code skills</a>: markdown files that give an agent a persona, a set of responsibilities, and the tools it’s allowed to use. Think of a skill not as a script that says “run these steps,” but as a role description that says “you are the build engineer, here’s what you know and how you make decisions.” That distinction matters because agents need judgment, not just instructions. When a test fails unexpectedly, a script stops. A role investigates.</p>



<p>The same skill file, the same behavior, whether it runs on a developer’s laptop or in CI.</p>



<h2 class="wp-block-heading"><strong>Local First, CI Second</strong></h2>



<p>Coding Agent Sandboxes is a CLI tool (<code>sbx</code>) that manages sandbox lifecycles: create, start, stop, remove, configure networking, mount workspaces, and more. It runs on MacOS, Linux and Windows. Every release needs testing across both platforms, across upgrade paths between versions, and under sustained load to catch resource leaks. The team also needs daily visibility into what shipped, and a way to triage the growing issue backlog without it becoming a full-time job.</p>



<p>We could have written traditional test scripts and reporting tools. Instead, we built agent roles that handle these tasks autonomously, both on our laptops and in CI.</p>



<p>The design principle behind the Fleet is simple: every skill runs on your machine first.</p>



<p>When we built the <code>/cli-tester</code> skill (the Fleet’s exploratory tester, more on that below), we didn’t start by writing a GitHub workflow. We started by invoking it locally. We watched it build the binaries, exercise the CLI commands, find issues, and report them. We tweaked the skill until it did the right thing in our terminal. Only then did we wire it into a workflow.</p>



<p>This matters because the alternative is painful. If you build CI-only agents, you debug them through commit-push-wait-read-logs cycles. Every iteration takes minutes. When the skill runs locally first, the iteration takes seconds. You see the agent think. You see where it gets confused. You fix the skill file, re-invoke, and try again.</p>



<p>CI is just another runtime for the same skill. The <code>/cli-tester</code> that runs nightly on MacOS, Linux and Windows runners is the exact same skill we invoke from our terminals. The workflow sets up the environment, checks out the code, and calls the skill. That’s it. No separate “CI version.” No translation layer. One skill, two runtimes.</p>



<p>This is what makes the Fleet practical. You’re not maintaining two systems. You’re maintaining one set of skills and a set of workflows that invoke them.</p>



<h2 class="wp-block-heading"><strong>The Roster</strong></h2>



<p>The skills directory has 20 skills in total. Most are foundational knowledge (architecture, code style, Go conventions, security, testing patterns). Seven of them are the Fleet: the roles that run autonomously on CI. Each one is a <a href="https://code.claude.com/docs/en/skills" id="dkr_skillmd-88831" rel="nofollow noopener" target="_blank">SKILL.md</a> file that describes a persona, not a procedure.</p>





<div class="wp-block-ponyo-image">
                <img alt="image1" class="fade-in" height="1091" src="https://www.docker.com/app/uploads/2026/04/image1.png" title="- image1" width="1999" />
        </div>



<p></p>



<p><code><strong>/build-engineer</strong></code> is the foundation that other skills stand on. It references topic files for building binaries, container templates, and local installs. It knows the <code>Taskfile.yml</code>, the <code>docker-bake.hcl</code>, and the platform-specific build flags. It doesn’t run on CI by itself. Other skills load it when they need to compile anything.</p>



<p><code><strong>/project-manager</strong></code> is the team’s memory. It deduplicates findings against existing issues and PRs before creating new ones, manages the GitHub Projects board (setting status, priority, and labels), and handles interactive triage when running locally. On CI, it switches to fully automatic mode: no questions asked, just deduplicate and create. It uses GraphQL pagination to scan the entire project board, not just the first page. Every other skill that discovers something calls the project-manager before opening an issue.</p>



<p><code><strong>/product-owner</strong></code> translates commit-speak into human language. It collects merged PRs from a date range, categorizes them (New Features, Bug Fixes, Improvements, Documentation, Maintenance), and rewrites each one in plain English. “feat(cli): add TZ env passthrough” becomes “Docker Sandboxes now automatically use your local timezone.” On CI, it outputs Slack Block Kit JSON. Locally, it renders a markdown table. It filters out noise from bots (Dependabot bumps, workflow-only changes) and skips posting when there’s nothing meaningful to report.</p>



<p><code><strong>/cli-tester</strong></code> is the exploratory tester of the Fleet, and it’s the largest skill by far. Unlike traditional test scripts that assert expected output and fail on any deviation, the cli-tester investigates what it finds. When output doesn’t match expectations, it asks <em>why</em> before filing a bug.</p>



<p>It defines 52+ test scenarios organized into 14 tiers: Core Lifecycle, Agent Smoke, Workspace, Network Policy, Sandbox Features, Blueprint, CLI UX, Environment, Code Tasks, Agent Network, Reliability, Collaboration, Error Recovery, and Human-Only (skipped in CI). It builds the binaries through the build-engineer, triages findings through the project-manager, and loads product scenarios defined by the actual Product Manager on the team. It monitors disk space during testing, posts an executive summary to Slack when it finishes, and runs nightly on CI across MacOS, Linux and Windows.</p>



<p>It also powers a slash command on GitHub. When someone comments <code>/cli-tester-review</code> on a pull request, CI spins up three runners (MacOS, Linux and Windows), each loading the skill to exercise the PR’s changes on that platform. The agents explore the code, run the scenarios, and post their findings as comments directly on the pull request.</p>



<p><code><strong>/performance-tester</strong></code> runs in two modes. Lifecycle Endurance repeatedly cycles create/stop/rm to detect reliability issues and resource leaks, producing xUnit JSON output. Code Exploration Benchmark clones a real Git repository and compares host-vs-sandbox I/O performance and Claude Code session behavior. Both modes measure disk usage over time and flag regressions. The goal is catching the slow degradation that no single test run would notice.</p>



<p><code><strong>/upgrade-tester</strong></code> runs a four-phase test plan. Phase A creates pre-upgrade state (sandboxes, configurations). Phase B installs the new version. Phase C verifies everything still works after the upgrade. Phase D optionally downgrades and verifies again. It takes two version tags as input, builds the binaries for each, creates VMs, and produces an executive summary with pass/fail per phase. Upgrade regressions are the kind of bug that’s invisible in a single-version test suite.</p>



<p><code><strong>/software-engineer</strong></code> operates in two modes. Reactive: when someone adds the <code>agent-fix</code> label to a GitHub issue, a MacOS runner picks it up and runs a <a href="https://docs.google.com/document/d/1maL_dGWgVEcJZ39y4toEf4CwEt3FjC1lgMXrKqp6RaU/edit?tab=t.0#heading=h.vm78uawrq88" id="dkr_ralph-loop-88831" rel="nofollow noopener" target="_blank">ralph-loop</a> to work the issue, contributing a PR with minimal, focused changes. Proactive: weekly, it runs in architect mode, scanning the codebase for quality issues, producing up to five findings, triaging them through the project-manager, then spawning three MacOS runners in parallel to fix three of them. Each runner delivers a PR targeting a specific simplification or tech-debt reduction.</p>



<h2 class="wp-block-heading"><strong>Skills That Compose</strong></h2>



<p>Individual skills are useful. Skills that load other skills are a team.</p>



<p>The seven Fleet roles sit on top of thirteen foundational skills: architecture, code style, Go conventions, software design, security, testing patterns, development workflow, git worktrees, and others. The foundational skills encode project knowledge. The Fleet roles encode behavior. A Fleet role loads the foundational skills it needs, the same way a new team member reads the project’s contributing guide before writing code.</p>



<p>The <code>/cli-tester</code> doesn’t know how to build binaries. It loads the <code>/build-engineer</code> for that. It doesn’t know whether the bug it found is a duplicate. It loads the <code>/project-manager</code> to check. The tester focuses on testing. The builder focuses on building. The manager focuses on triaging. Each role stays in its lane, and the composition creates something none of them could do alone.</p>



<p>The <code>/software-engineer</code> follows the same pattern. It loads the <code>/build-engineer</code> so it can compile the project, and it loads coding best practices and software design conventions so its output meets the team’s standards. The skill doesn’t try to encode everything. It delegates to the foundational skills.</p>



<p>The <code>/performance-tester</code> loads the <code>/cli-tester</code>, extending it with duration and metrics. Instead of duplicating the testing logic, it reuses it and adds a measurement layer on top.</p>



<p>This is the skills-as-roles principle in practice. When you design skills as personas with clear responsibilities (instead of step-by-step commands), they compose naturally. A tester that loads a builder and a manager is doing the same thing a human tester does: asking a colleague to compile the project and checking with the PM before filing a bug. The difference is that the “asking” happens through skill composition instead of a Slack message.</p>



<h2 class="wp-block-heading"><strong>The Ralph-Loop Is the Engine</strong></h2>



<p>The <a href="https://ghuntley.com/ralph/" id="dkr_ralph-wiggum-loop-88831" rel="nofollow noopener" target="_blank">Ralph Wiggum loop</a> is a pattern popularized by Geoffrey Huntley in 2025: a Bash loop that keeps feeding an AI coding agent the same task until the work is done. At its simplest, it’s <code>while :; do cat PROMPT.md | claude-code ; done</code>. Each iteration spawns a fresh agent with a clean context window. The agent reads the task, implements one piece, runs the tests, commits if they pass, and exits. The loop restarts, and the next iteration picks up where the previous one left off. Instead of hoping for first-try perfection, you design for iteration.</p>



<p>Our implementation of this pattern is called a Ralph-loop. The Fleet skills define <em>what</em> each agent role knows. The Ralph-loop defines <em>how</em> the iteration runs.</p>



<p>Our Ralph-loop is a composite GitHub Action backed by a shell script that adds a layer on top of the basic pattern: a separate worker and reviewer. It fetches the issue context, creates a working branch, and iterates: the worker implements changes and writes a summary, the reviewer evaluates the diff and decides SHIP or REVISE. If REVISE, the feedback goes back to the worker for another pass. Up to five iterations by default. If the reviewer says SHIP, the loop pushes the branch, creates a PR, and comments on the original issue.</p>



<p>The worker and reviewer run as separate Claude invocations with different models. The worker uses Opus for implementation. The reviewer uses Opus with 1M context to evaluate the full diff against the task requirements. Each one loads the <code>/software-engineer</code> skill (which in turn loads the build-engineer and coding best practices), so they share the same project knowledge but apply it from different perspectives.</p>



<p>Separating generation from evaluation is deliberate. The same agent that wrote the code shouldn’t evaluate whether the code is good. It’s the oldest principle in quality assurance: the person who built the thing shouldn’t be the only person who tests it. The worker’s job is to solve the problem. The reviewer’s job is to decide whether the problem is actually solved.</p>



<p>The Ralph-loop works locally too. The same <code>ralph-loop.sh</code> script that CI calls can be invoked from your terminal with <code>--issue-number 42</code>. Locally, it parses CLI arguments instead of reading environment variables, and outputs plain text instead of streaming JSON. Same loop, same prompts, same iteration pattern. We debugged the worker and reviewer prompts on our laptops before they ever ran in CI.</p>



<p>The workflows handle scheduling and triggering: nightly cron for the testers, label events for the software-engineer, weekly cron for the architect mode. The Ralph-loop handles the iteration pattern. The skills handle the domain knowledge. Three layers, each with a clear job.</p>



<p>This separation is what made the Fleet possible to build in a couple of weeks. We didn’t have to reinvent the automation loop for every role. The Ralph-loop already knew how to iterate. We just needed to give each role its own skill file and wire the triggers.</p>



<h2 class="wp-block-heading"><strong>What the Fleet Ships</strong></h2>



<p>The Fleet has been running for a couple of weeks. Here’s what it delivers.</p>



<p><strong>Automated issue resolution.</strong> A team member labels an issue with <code>agent-fix</code>. The CI grabs a MacOS runner, reads the issue, and starts working. The result is a pull request that addresses the issue. Not every PR lands without changes, but the first draft is there for review, often within the hour.</p>



<p><strong>Daily release notes.</strong> The product-owner traverses the git log every day and posts a Slack summary for stakeholders. No one has to manually compile “what shipped this week.” The stakeholders see progress in real time, at the speed the team actually moves.</p>



<p><strong>Nightly exploratory testing.</strong> The cli-tester runs every night on MacOS and Windows. It loads the product scenarios that the Product Manager has defined, exercises the CLI, and opens issues for anything it finds. Before opening an issue, it checks for duplicates through the project-manager. When it finishes, it posts a Slack message with the results.</p>



<p><strong>Performance and upgrade testing.</strong> The performance-tester and upgrade-tester run on CI across both platforms. Disk usage regressions, behavioral differences between sandbox and non-sandbox modes, and version compatibility issues get caught before they reach a human reporter.</p>



<p><strong>Weekly tech-debt reduction.</strong> Every week, the software-engineer runs in architect mode. It reviews the codebase, identifies three spots where code can be simplified or legacy patterns can be cleaned up, spawns three parallel runners, and delivers three PRs. Each one is a small, focused improvement. Over time, they compound.</p>



<h2 class="wp-block-heading"><strong>What We Don’t Automate</strong></h2>



<p>The Fleet creates pull requests. It does not merge them.</p>



<p>That’s the trust boundary, and it’s deliberate. Merge decisions stay with humans. So do architectural choices, scope decisions, and prioritization. The agents do the work. The team decides what work matters and whether the output meets the bar.</p>



<p>The supervision model scales the same way it works on a developer’s laptop. When we run multiple agents locally in parallel worktrees, we review their output before merging. With the Fleet, the team supervises seven agent roles running on CI. The shape of the oversight is the same: review the output, approve or adjust, move on. The difference is that the agents don’t need anyone’s laptop to start working.</p>



<p>The Fleet is not replacing the team. It’s extending it. Seven roles that handle repetitive, well-defined work so humans can focus on work that requires judgment, context, and taste. The Fleet has many arms, but the team still steers the ship.</p>



<h2 class="wp-block-heading"><strong>What We Learnt Building the Fleet</strong></h2>



<p><strong>Start with the foundation, not the flashiest skill.</strong> We started with the <code>/cli-tester</code> because testing the CLI felt like the highest-value target. But it needed to build binaries, triage issues, and load product scenarios, all things that depended on other skills we hadn&#8217;t written yet. We should have started with the /build-engineer, the skill everything else stands on. The second skill was better because of what we learned from the first. Don&#8217;t design the full fleet upfront.</p>



<p><strong>Build locally first, deploy to CI second</strong>. The commit-push-wait-read-logs cycle is where velocity goes to die. If you can&#8217;t debug a skill in your terminal, it&#8217;s not ready for a workflow. Some behaviors only surface on CI runners (different OS, permissions, network constraints), and those iterations cost hours of wall-clock time. Minimize what can only be tested in CI.</p>



<p><strong>Write skills as roles, not scripts</strong>. Ask yourself: &#8220;If a new team member joined tomorrow with this exact role, what would I tell them?&#8221; What do they need to know? What tools can they use? How should they handle ambiguity? That conversation is your SKILL.md. &#8220;You are the build engineer, here&#8217;s what you know&#8221; produces better judgment than &#8220;run these five steps.&#8221; When something unexpected happens, a role investigates. A script stops.</p>



<p><strong>Compose skills like you compose teams</strong>. The <code>/cli-tester</code> doesn&#8217;t know how to build binaries or triage bugs. It loads the <code>/build-engineer</code> and <code>/project-manager</code> for that. Each role stays in its lane. The composition creates what none of them could do alone.</p>



<p><strong>Separate generation from evaluation</strong>. The agent that wrote the code shouldn&#8217;t be the only one that reviews it. Our Ralph-loop uses a worker and a reviewer for a reason: the oldest principle in quality assurance applies to agents too.</p>



<p><strong>Triage matters more than detection</strong>. The <code>/cli-tester</code> initially filed issues for every unexpected output. Transient failures, timing-dependent behavior, environment quirks: everything became an issue. The signal-to-noise ratio got bad enough that the team started ignoring findings. Getting the triage right (deduplication, confirming before filing) took longer than building the tester itself.And one more thing. All Fleet agents, even on ephemeral CI runners, run inside <a href="https://docs.docker.com/ai/sandboxes/" id="dkr_coding-agent-sandboxes-88831" rel="nofollow noopener" target="_blank">Coding Agent Sandboxes</a>. We test with what our users use.</p>
