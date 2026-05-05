---
title: "Defending Your Software Supply Chain: What Every Engineering Team Should Do Now"
url: "https://www.docker.com/blog/defending-your-software-supply-chain-what-every-engineering-team-should-do-now/"
date: "Thu, 02 Apr 2026 18:14:40 +0000"
author: "Dan Berezin Stelzer"
feed_url: "https://www.docker.com/feed/"
---
<p>The software supply chain is under sustained attack. Not from a single threat actor or a single incident, but from an ecosystem-wide campaign that has been escalating for months and shows no signs of slowing down.</p>



<p>This week, <a href="https://www.huntress.com/blog/supply-chain-compromise-axios-npm-package" id="dkr_axios-88434" rel="nofollow noopener" target="_blank">axios</a>, the HTTP client library downloaded 83 million times per week and present in roughly 80% of cloud environments, was <a href="https://cloud.google.com/blog/topics/threat-intelligence/north-korea-threat-actor-targets-axios-npm-package" id="dkr_compromised-via-a-hijacked-maintainer-account-88434" rel="nofollow noopener" target="_blank">compromised via a hijacked maintainer account</a>. Two backdoored versions deployed platform-specific RATs attributed with high confidence to North Korea&#8217;s Lazarus Group. The malicious versions were live for approximately three hours. That was enough.</p>



<p>This follows the <a href="https://www.wiz.io/blog/trivy-compromised-teampcp-supply-chain-attack" id="dkr_teampcp-campaign-88434" rel="nofollow noopener" target="_blank">TeamPCP campaign</a> in March, which weaponized Aqua Security&#8217;s Trivy vulnerability scanner, a security tool trusted by thousands of organizations, and cascaded the compromise into Checkmarx KICS, LiteLLM, Telnyx, and 141 npm packages via a self-propagating worm. Before that, the <a href="https://unit42.paloaltonetworks.com/npm-supply-chain-attack/" id="dkr_shai-hulud-worm-88434" rel="nofollow noopener" target="_blank">Shai-Hulud worm</a> tore through the npm ecosystem in late 2025, and <a href="https://thehackernews.com/2026/03/glassworm-supply-chain-attack-abuses-72.html" id="dkr_glassworm-88434" rel="nofollow noopener" target="_blank">GlassWorm</a> infected 400+ VS Code extensions, GitHub repos, and npm packages using invisible Unicode payloads.</p>



<p>The pattern is consistent across all of these incidents: attackers steal developer credentials, use them to poison trusted packages, and the compromised packages steal more credentials. It is self-reinforcing, it is accelerating, and it now has ransomware monetization pipelines behind it.</p>



<h2 class="wp-block-heading">The common thread is implicit trust</h2>



<p>If you look at what actually failed in each of these compromises, the answer is the same every time: <strong>trust was assumed where it should have been verified.</strong> Organizations trusted a container tag because it had a familiar name. They trusted a GitHub Action because it had a version number. They trusted a CI/CD secret because the workflow was authored by someone on the team. In every case, the attacker exploited the gap between assumed trust and verified trust.</p>



<p>The organizations that came through these incidents with minimal damage had already begun replacing implicit trust with explicit verification at every layer of their stack: verified base images instead of community pulls, pinned references instead of mutable tags, scoped and short-lived credentials instead of long-lived tokens, and sandboxed execution environments instead of wide-open CI runners. None of these are new ideas, and none of them are difficult to implement. What they require is a shift in default posture, from &#8220;trust unless there&#8217;s a reason not to&#8221; to &#8220;verify before you trust, and limit the blast radius when verification fails.&#8221;</p>



<p>Here is what we recommend every engineering organization should do, and what we practice ourselves at Docker:</p>



<h2 class="wp-block-heading">Secure your foundations</h2>



<h3 class="wp-block-heading">Start with trusted base images</h3>



<p>Don&#8217;t build on artifacts you can&#8217;t verify. <a href="https://hub.docker.com/hardened-images/catalog" id="dkr_docker-hardened-images-88434" rel="nofollow noopener" target="_blank">Docker Hardened Images</a> (DHI) are rebuilt from source by Docker with SLSA Build Level 3 attestations, signed SBOMs, and VEX metadata, free and open source under Apache 2.0. DHI was not affected by TeamPCP because its controlled build pipeline and built-in cooldown periods mean short-lived supply chain exploits (typically 1 to 6 hours) are eradicated before they ever enter the image. There is no reason not to use these today. Docker Hardened Images for Node.js, Python, and Rust also include Socket Firewall, which blocks malicious dependencies at install time intercepting supply chain attacks like CanisterWorm or the Axios compromise during <code>npm install</code> or <code>pip install</code> before they ever execute.</p>



<h3 class="wp-block-heading">Pin everything by digest or commit SHA</h3>



<p>Mutable tags are not a security boundary. This is exactly how TeamPCP hijacked 75 of 76 trivy-action version tags. Pin GitHub Actions to full 40-character commit SHAs. Pin container images by sha256 digest. Pin package dependencies to exact versions and remove ^ and ~ ranges. If a reference can be overwritten without changing its name, it will be. For GitHub Actions you maintain, enable<a href="https://docs.github.com/en/code-security/concepts/supply-chain-security/immutable-releases" id="dkr_-immutable-88434" rel="nofollow noopener" target="_blank"> Immutable</a> Releases &#8211; this locks release tags after publication and generates signed attestations, preventing the tag-rewriting attack that powered TeamPCP&#8217;s hijack of trivy-action. Inventory every third-party GitHub Action in use across your org and enforce an allowlist policy as you cannot pin what you haven&#8217;t cataloged. Enable two-factor authentication on every package registry account in your organization like npm, PyPI, RubyGems, Docker Hub as account takeover of a single maintainer is how most of these attacks begin. Commit your lock files and use npm ci (or the equivalent in your package manager) in all CI pipelines &#8211; this prevents builds from silently pulling new versions that aren&#8217;t in your lock file.</p>



<h3 class="wp-block-heading">Use cooldown periods for dependency updates</h3>



<p>Both <a href="https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-options-reference#cooldown-" id="dkr_npm-88434" rel="nofollow noopener" target="_blank">npm</a> and <a href="https://docs.renovatebot.com/key-concepts/minimum-release-age/" id="dkr_renovate-88434" rel="nofollow noopener" target="_blank">Renovate</a> support minimum release age settings that delay adoption of new versions. Most supply chain attacks have a shelf life of hours, and a 3-day cooldown eliminates the vast majority of them. We maintain a collection of safe default configurations for common package managers and tooling. Use it. Contribute to it.</p>



<h3 class="wp-block-heading">Generate SBOMs at build time</h3>



<p>When an incident hits, the first question is always: &#8220;are we affected?&#8221; If you use <a href="https://docs.docker.com/reference/cli/docker/buildx/" id="dkr_docker-buildx-88434" rel="nofollow noopener" target="_blank">docker buildx</a> to build your images, you can <a href="https://docs.docker.com/build/metadata/attestations/" id="dkr_generate-and-attach-88434" rel="nofollow noopener" target="_blank">generate and attach</a> SBOMs and provenance attestations during the build. Sign them. Store them alongside your images. When the next axios or Trivy happens, you check the build metadata rather than having to exec into live Kubernetes pods to figure out what&#8217;s running. Docker Scout can then continuously monitor those SBOMs against known vulnerabilities and policy violations.</p>



<h2 class="wp-block-heading">Secure your CI/CD</h2>



<h3 class="wp-block-heading">Treat every CI runner as a potential breach point</h3>



<p>TeamPCP&#8217;s credential stealer ran inside CI/CD pipelines, dumping process memory and sweeping 50+ filesystem paths for secrets. Anything accessible to a workflow step is accessible to an attacker who compromises a dependency in that step. Avoid pull_request_targe triggers in GitHub Actions unless absolutely necessary and with explicit security checks as this is the exact mechanism TeamPCP used to execute code in the context of the base repository with access to its secrets.  Audit what secrets each workflow step can reach. If a scanning step has access to your deployment credentials, that is a blast radius problem, not a scanning problem.</p>



<h3 class="wp-block-heading">Use short-lived, narrowly scoped credentials</h3>



<p>The root cause of the Trivy breach was a single Personal Access Token with broad scope used across 33+ workflows. Use short-lived, narrowly-scoped credentials. No single token should grant cross-repository or organization-wide access. Use a secrets manager, not environment variables scattered across workflow files. This is an area where the ecosystem, including Docker Hub, needs to continue improving, and we are actively working on it.</p>



<h3 class="wp-block-heading">Use an internal mirror or artifact proxy</h3>



<p>Place Artifactory, CodeArtifact, or Nexus between your build systems and public registries. Scan and approve versions before they reach your pipelines. Docker Business customers can also use Registry Access Management and Image Access Management to restrict which registries and images developers can pull, providing a lighter-weight policy layer for teams that don&#8217;t run a full artifact proxy.</p>



<h3 class="wp-block-heading">Test dependency updates where production secrets don&#8217;t exist</h3>



<p>Evaluate updates in dev/staging environments that have no access to production credentials. If a malicious package runs in staging, it steals nothing of value.</p>



<h2 class="wp-block-heading">Secure your endpoints</h2>



<p>This is where most of these attacks actually start. TeamPCP, Shai-Hulud, and now axios all deploy infostealers that sweep developer machines for credentials stored in dotfiles, environment variables, SSH keys, browser sessions, and cloud configs. Protecting CI/CD pipelines matters, but if the developer machine that authors those pipelines is compromised, the attacker inherits whatever that developer can reach.</p>



<h3 class="wp-block-heading">Deploy canary tokens</h3>



<p>Place fake credentials across your fleet, AWS keys, API tokens, SSH keys, that serve no purpose other than to alert you when they&#8217;re exfiltrated. If an infostealer sweeps a machine, canary tokens fire before the real credentials are used. Tools like <a href="https://tracebit.com/customer/docker" id="dkr_tracebit-88434" rel="nofollow noopener" target="_blank">Tracebit</a> and <a href="https://canarytokens.org/" id="dkr_canarytokens-88434" rel="nofollow noopener" target="_blank">Canarytokens</a> make this trivial. If you have an MDM solution (Jamf, Intune, Jumpcloud), push canaries to every managed device. We deployed this across our fleet in under a day.</p>



<h3 class="wp-block-heading">Clean up credential sprawl</h3>



<p>Audit <code>~/.ssh/</code>, <code>~/.aws/credentials</code>, <code>~/.docker/config.json</code>, <code>.env</code> files, and shell histories for hardcoded secrets. Move everything to a password manager or secrets vault (1Password, HashiCorp Vault). Passphrase-protect all SSH keys. An infostealer that lands on a machine with no cleartext credentials gets nothing useful. Audit the extensions and plugins installed across your developer tools (IDE extensions, browser extensions, coding agent extensions like skills, plugins, MCP servers, etc…) as these tend to run with developer-level permissions and most marketplaces do not re-review updates after initial publication.</p>



<h3 class="wp-block-heading">Deploy EDR with behavioral detection</h3>



<p>Endpoint detection and response tools should cover developer machines and CI runners, with detections tuned for credential sweeping, persistence mechanisms, and unusual process behavior rather than just known malware signatures.</p>



<h2 class="wp-block-heading">Secure your AI development</h2>



<p>AI coding agents are compounding supply chain risk in ways the industry is only beginning to appreciate. Agents install packages, modify configs, make API calls, and spin up containers with developer-level access. A compromised dependency pulled by an agent has the same blast radius as a compromised developer machine, and the people using these agents now include non-developers who may not recognize suspicious behavior.</p>



<h3 class="wp-block-heading">Run agents in sandboxed environments</h3>



<p><a href="https://docs.docker.com/ai/sandboxes/" id="dkr_docker-sandboxes-sbx-88434" rel="nofollow noopener" target="_blank">Docker Sandboxes (sbx)</a> run AI coding agents like Claude Code, Gemini CLI, Codex, and others inside isolated microVMs. Each sandbox gets its own kernel, filesystem, Docker Engine, and network, completely separated from your host. Credentials are injected into HTTP headers by the host proxy and never enter the VM directly. Network access is deny-by-default, with explicit allowlists. If a compromised dependency runs inside a sandbox, it cannot reach your host filesystem, your Docker daemon, your other containers, or any domain you haven&#8217;t explicitly approved.</p>



<h3 class="wp-block-heading">Govern your MCP servers</h3>



<p>Model Context Protocol servers are the new unvetted dependency. They run with broad permissions, connect AI agents to internal systems, and <a href="https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks" id="dkr_43-of-analyzed-mcp-servers-88434" rel="nofollow noopener" target="_blank">43% of analyzed MCP servers</a> have command injection flaws. Use signed, hardened images for MCP servers. Docker maintains 300+ verified MCP server images with the same SLSA/SBOM standards as DHI. Docker&#8217;s <a href="https://github.com/docker/mcp-gateway" id="dkr_mcp-gateway-88434" rel="nofollow noopener" target="_blank">MCP Gateway</a> provides centralized proxy, policy enforcement, secret blocking, and audit logging for all agent-to-tool traffic.</p>



<h3 class="wp-block-heading">Standardize on fewer tools, governed centrally</h3>



<p>It&#8217;s tempting to run every AI tool and model. Don&#8217;t. Consolidate on a trusted stack, push managed configurations via MDM, and use Docker Desktop&#8217;s administrative features (registry access management, proxy configuration, image access management) to control what agents can pull and where they can push.</p>



<h2 class="wp-block-heading">Build muscle for incident response</h2>



<h3 class="wp-block-heading">Maintain SBOMs for everything in production</h3>



<p>When the next compromise drops, you need to answer &#8220;are we affected?&#8221; in minutes, not days. Build-time SBOMs via docker buildx, combined with Docker Scout&#8217;s continuous monitoring, give you that capability. If you have to exec into running containers to determine exposure, you&#8217;re already behind.</p>



<h3 class="wp-block-heading">Have playbooks ready</h3>



<p>Know how to freeze your GitHub org, pause CI/CD without breaking everything, revoke credentials in bulk, and communicate to customers before you need to do it under pressure. The time to figure out your incident response workflow is not during the incident. If you haven&#8217;t already, audit your npm/PyPI/Docker, Hub accounts for unauthorized publishes, review recent CI logs for unexpected network calls or secret access, and rotate any long-lived tokens that were accessible to CI in the past 90 days.</p>



<h3 class="wp-block-heading">Verify before you trust, slow down where it counts</h3>



<p>Most supply chain attacks burn out within hours. A small delay in adopting new versions, whether via cooldown periods, manual review gates, or simply waiting 72 hours, eliminates the majority of the risk. Speed of adoption is not worth the cost of compromise.</p>



<h2 class="wp-block-heading">The landscape has changed, your defaults should too</h2>



<p>The supply chain attack wave is not a single incident to respond to. It is a permanent shift in the threat landscape. The attackers range from nation-state operators like Lazarus Group to opportunistic teenagers like TeamPCP and LAPSUS$ who are building the plane as it takes off, using AI to accelerate, and monetizing through ransomware partnerships. The ecosystem they are exploiting, npm, PyPI, GitHub Actions, container registries, has not fundamentally changed in its trust model.</p>



<p>What has changed is that defenders now have the tools to establish explicit trust boundaries where implicit trust used to be the only option. Hardened base images, build-time attestations, sandboxed execution, and canary-based detection did not exist at this maturity level two years ago. The gap between organizations that adopt these layers and those that don&#8217;t is going to widen fast.</p>



<p>Everything we&#8217;ve recommended here, we practice at Docker. We pull from public registries, we run CI/CD pipelines, we use AI agents, and we face the same threat actors you do. This is how we&#8217;re protecting ourselves.</p>



<h3 class="wp-block-heading"><strong>Further reading:</strong></h3>



<ul class="wp-block-list">
<li><a href="https://hub.docker.com/hardened-images/catalog" id="dkr_docker-hardened-images-88421-2" rel="nofollow noopener" target="_blank">Docker Hardened Images</a>: free, signed, SLSA-compliant base images</li>



<li><a href="https://docs.docker.com/scout/" id="dkr_docker-scout-88434" rel="nofollow noopener" target="_blank">Docker Scout</a>: SBOM generation, vulnerability detection, and policy enforcement</li>



<li><a href="https://docs.docker.com/ai/sandboxes/" id="dkr_docker-sandboxes-88434" rel="nofollow noopener" target="_blank">Docker Sandboxes</a>: isolated microVMs for AI coding agents</li>



<li><a href="https://github.com/docker-security/safe-defaults" id="dkr_safe-defaults-88434" rel="nofollow noopener" target="_blank">Safe Defaults</a>: secure configurations for package managers and tooling</li>



<li><a href="https://docs.docker.com/build/metadata/attestations/" id="dkr_building-sboms-with-docker-buildx-88434" rel="nofollow noopener" target="_blank">Building SBOMs with Docker Buildx</a>: attach provenance and SBOMs at build time</li>
</ul>



<p></p>
