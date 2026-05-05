---
title: "Why MicroVMs: The Architecture Behind Docker Sandboxes"
url: "https://www.docker.com/blog/why-microvms-the-architecture-behind-docker-sandboxes/"
date: "Thu, 16 Apr 2026 17:14:19 +0000"
author: "Srini Sekaran"
feed_url: "https://www.docker.com/feed/"
---
<p>Last week, we launched Docker Sandboxes with a bold goal: to deliver the strongest agent isolation in the market.</p>



<p>This post unpacks that claim, how microVMs enable it, and some of the architectural choices we made in this approach.</p>



<h2 class="wp-block-heading">The Problem With Every Other Approach</h2>



<p>Every sandboxing model asks you to give something up. We looked at the top four approaches.</p>



<p><strong>Full VMs</strong> offer strong isolation, but general-purpose VMs weren&#8217;t designed for ephemeral, session-heavy agent workflows. Some VMs built for specific workloads can spin up more effectively on modern hardware, but the general-purpose VM experience (slow cold starts, heavy resource overhead) pushes developers toward skipping isolation entirely.</p>



<p><strong>Containers</strong> are fast and are the way modern applications are built. But for an autonomous agent that needs to build and run its own Docker containers, which coding agents routinely do, you hit Docker-in-Docker, which requires elevated privileges that undermine the isolation you set up in the first place. Agents need a real Docker environment to do development work, and containers alone don&#8217;t give you that cleanly.</p>



<p><strong>WASM / V8 isolates</strong> are fast to spin up, but the isolation model is fundamentally different. You&#8217;re running isolates, not operating systems. Even providers of isolate-based sandboxes have acknowledged that hardening V8 is difficult, and that security bugs in the V8 engine surface more frequently than in mature hypervisors. Beyond the security model, there&#8217;s a practical gap: your agent can&#8217;t install system packages or run arbitrary shell commands. For a coding agent that needs a real development environment, WASM isn&#8217;t one.</p>



<p><strong>Not using any sandboxing</strong> is fast, obviously. It&#8217;s also a liability. One rm -rf, one leaked .env, one rogue network call, and the blast radius is your entire machine.</p>



<h2 class="wp-block-heading">Why MicroVMs</h2>



<p>Docker Sandboxes run each agent session inside a dedicated microVM with a private Docker daemon isolated by the VM boundary, and no path back to the host.</p>



<p>That one sentence contains three architectural decisions worth unpacking.</p>





<div class="wp-block-ponyo-image">
                <img alt="Screenshot 2026 04 09 at 3.23.44 PM" class="fade-in" height="1302" src="https://www.docker.com/app/uploads/2026/04/Screenshot-2026-04-09-at-3.23.44-PM-2320x1302.png" title="- Screenshot 2026 04 09 at 3.23.44 PM" width="2320" />
        </div>



<p><strong>Dedicated microVM.</strong> Each sandbox gets its own kernel. It&#8217;s hardware-boundary isolation, the same kind you get from a full VM. A compromised or runaway agent can&#8217;t reach the host, other sandboxes, or anything outside its environment. If it tries to escape, it hits a wall.</p>



<p><strong>Private, VM-isolated Docker daemon.</strong> This is the key differentiator for coding agents. AI is going to result in more container workloads, not fewer. Containers are how applications are developed, and agents need a Docker environment to do that development. Docker Sandboxes give each agent its own Docker daemon running inside a microVM, fully isolated by the VM boundary. Your agent gets full <em>docker build</em>, <em>docker run</em>, and <em>docker compose </em>support with no socket mounting, no host-level privileges, none of the security compromises other approaches require. This means we treat agents as we would a human developer, giving them a true developer environment so they can actually complete tasks across the SDLC.</p>



<p><strong>No path back to the host.</strong> File access, network policies, and secrets are defined <em>before</em> the agent runs, not enforced by the agent itself. This is an important distinction. An LLM deciding its own security boundaries is not a security model. The bounding box has to come from infrastructure, not from a system prompt.</p>



<h2 class="wp-block-heading">Why We Built a New VMM</h2>



<p>Choosing microVMs was the easy part. Running them where developers actually work was the hard part.</p>



<p>We looked hard at existing options, but none of them were designed for what we needed. Firecracker, the most well-known microVM runtime, was designed for cloud infrastructure, specifically Linux/KVM environments like AWS Lambda. It has no native support for macOS or Windows, full stop. That&#8217;s fine for server-side workloads, but coding agents don&#8217;t run in the cloud. They run on developer laptops, across macOS, Windows, and Linux. </p>



<p>We could have shimmed an existing VMM into working across platforms, creating translation layers on macOS and workarounds on Windows, but bolting cross-platform support onto a Linux-first VMM means fighting abstractions that were never designed for it. That&#8217;s how you end up with fragile, layered workarounds that break the &#8220;it just works&#8221; promise and create the friction that makes developers skip sandboxing altogether.</p>



<p><strong>So we built a new VMM, purpose-built for where coding agents actually run.</strong></p>



<p>It runs natively on all three platforms using each OS&#8217;s native hypervisor: Apple&#8217;s Hypervisor.framework, Windows Hypervisor Platform, and Linux KVM. A single codebase for three platforms and zero translation layers.</p>



<p>This matters because it means agents get kernel-level isolation optimized for each specific OS. Cold starts are fast because there&#8217;s no abstraction tax. A developer on a MacBook gets the same isolation guarantees and startup performance as a developer on a Linux workstation or a Windows machine.</p>



<p>Building a VMM from scratch is not a small undertaking. But the alternative, asking developers to accept slower starts, degraded compatibility, or platform-specific caveats, is exactly the kind of asterisk that makes people run agents on the host instead. Our approach removes that asterisk at the hypervisor level.</p>



<h2 class="wp-block-heading">Fast Cold Starts</h2>



<p>We rebuilt the virtualization layer from scratch, optimizing for fast spin up and fast tear downs. Cold starts are fast. This matters for one reason: if the sandbox is slow, developers skip it. Every friction point between &#8220;start agent&#8221; and &#8220;agent is running&#8221; is a reason to run on the host instead. With near-instant starts, there is no performance reason to run outside it.</p>



<h2 class="wp-block-heading">What This Means In Practice</h2>



<p>Here&#8217;s the concrete version of what this architecture gives you:</p>



<p><strong>Full development environment.</strong> Agents can clone repos, install dependencies, run test suites, build Docker images, spin up multi-container services, and open pull requests, all inside the sandbox. Nothing is stubbed out or simulated. Agents are treated as developers and given what they need to complete tasks end to end. </p>



<p><strong>Scoped access, not all-or-nothing.</strong> You define the boundary: exactly which files and directories the agent can see, which network endpoints it can reach, and which secrets it receives. Credentials are injected at runtime and outside the MicroVM boundary, never baked into the environment.</p>



<p><strong>Disposable by design.</strong> If an agent goes off track, delete the sandbox and start fresh in seconds. There is no state to clean up and nothing to roll back on your host.</p>



<p><strong>Works with every major agent.</strong> Claude Code, Codex, OpenCode, GitHub Copilot, Gemini CLI, Kiro, Docker Agent, and next-generation autonomous systems like OpenClaw and NanoClaw. Same isolation, same speed, one sandbox model across all of them.</p>



<h2 class="wp-block-heading">For Teams</h2>



<p>Individual developers can install and run Docker Sandboxes today, standalone, no Docker Desktop license required. </p>



<p>For teams that want centralized filesystem and network policies that can be enforced across an organization and scale sandboxed execution, <strong><a href="https://www.docker.com/products/docker-sandboxes/" id="dkr_get-in-touch-88631">get in touch</a></strong> to learn about enterprise deployment.</p>



<h2 class="wp-block-heading">The Tradeoff That Isn&#8217;t</h2>



<p>The pitch for sandboxing has always come with an asterisk: <em>yes, it&#8217;s safer, but you&#8217;ll pay for it in speed, compatibility, or workflow friction.</em></p>



<p>MicroVMs eliminate that asterisk. You get VM-grade isolation with cold starts fast enough that there&#8217;s no reason to skip it, and full Docker support inside the sandbox. There is no tradeoff.</p>



<p>Your agents should be running autonomously. They just shouldn&#8217;t be running without any guardrails.</p>



<h2 class="wp-block-heading">Use Sandboxes in Seconds</h2>



<p>Install Sandboxes with a single command.</p>



<p><strong>macOS<br /></strong><em>brew install docker/tap/sbx   </em></p>



<p><strong>Windows <br /></strong><em>winget install Docker.sbx  </em></p>



<p>Read the <a href="https://docs.docker.com/ai/sandboxes" id="dkr_docs-88631" rel="nofollow noopener" target="_blank">docs</a> to learn more.</p>



<p></p>
