---
title: "Trivy, KICS, and the shape of supply chain attacks so far in 2026"
url: "https://www.docker.com/blog/trivy-kics-and-the-shape-of-supply-chain-attacks-so-far-in-2026/"
date: "Thu, 23 Apr 2026 15:32:00 +0000"
author: "Aditya Tripathi"
feed_url: "https://www.docker.com/feed/"
---
<h2 class="wp-block-heading">Catching the KICS push: what happened, and the case for open, fast collaboration</h2>



<p>In the past few weeks we&#8217;ve worked through two supply chain compromises on Docker Hub with a similar shape: first Trivy, now Checkmarx KICS. In both cases, stolen publisher credentials were used to push malicious images through legitimate publishing flows. In both cases, Docker&#8217;s infrastructure was not breached. And in both cases, the software supply chain of everyone who pulled the compromised tags was briefly exposed.</p>



<p>This is our account of what happened with KICS, what affected users should do, and what the pattern says about where defenders need to invest.</p>



<h2 class="wp-block-heading">What happened</h2>



<p>On April 22, 2026 at approximately 12:35 UTC, a threat actor authenticated to Docker Hub using valid Checkmarx publisher credentials and pushed malicious images to the <code>checkmarx/kics</code> repository. Five existing tags were overwritten to malicious digests (<code>latest</code>, <code>v2.1.20</code>, <code>v2.1.20-debian</code>, <code>alpine</code>, <code>debian</code>) and two new tags (<code>v2.1.21</code>, <code>v2.1.21-debian</code>) were created. The images were built from an attacker-controlled source repository, not from Checkmarx&#8217;s.</p>



<p>The poisoned binary kept the legitimate scanning surface intact and added a quiet exfiltration path. Scan output was collected, encrypted, and sent to attacker-controlled infrastructure at <code>audit.checkmarx[.]cx</code>, with the User-Agent <code>KICS-Telemetry/2.0</code>. Because KICS scans Terraform, CloudFormation, Kubernetes and similar configuration files, its output routinely contains secrets, credentials, cloud resource names, and internal topology. </p>



<p>Affected malicious digests (any one of these in your pull history should be treated as malicious):</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: plain; title: ; notranslate">
For alpine, v2.1.20, v2.1.21 -&amp;gt; Index manifest digest: sha256:2588a44890263a8185bd5d9fadb6bc9220b60245dbcbc4da35e1b62a6f8c230d

Image digest (amd64): sha256:d186161ae8e33cd7702dd2a6c0337deb14e2b178542d232129c0da64b1af06e4
Image digest (arm64): sha256:415610a42c5b51347709e315f5efb6fffa588b6ebc1b95b24abf28088347791b

For debian, v2.1.20-debian, v2.1.21-debian -&amp;gt; Index manifest digest: sha256:222e6bfed0f3bb1937bf5e719a2342871ccd683ff1c0cb967c8e31ea58beaf7b

Image digest (amd64): sha256:a6871deb0480e1205c1daff10cedf4e60ad951605fd1a4efaca0a9c54d56d1cb
Image digest (arm64): sha256:ff7b0f114f87c67402dfc2459bb3d8954dd88e537b0e459482c04cffa26c1f07

For latest -&amp;gt; Index manifest digest: sha256:a0d9366f6f0166dcbf92fcdc98e1a03d2e6210e8d7e8573f74d50849130651a0

Image digest (amd64): sha256:26e8e9c5e53c972997a278ca6e12708b8788b70575ca013fd30bfda34ab5f48f

Image digest (arm64): sha256:7391b531a07fccbbeaf59a488e1376cfe5b27aef757430a36d6d3a087c610322
</pre></div>


<p>If your CI ran kics against any repository with credentials in scope during the exposure window, rotate those credentials now. Re-pull <code>checkmarx/kics</code> by digest, not tag, and pin your CI to the digest so a future overwrite cannot silently affect you again. Purge the malicious digests from local caches, CI runners, pull-through registries, and mirrors: a clean pull won&#8217;t remove what&#8217;s already been cached. Check egress logs for connections to <code>audit.checkmarx[.]cx</code>, or outbound traffic with the <code>KICS-Telemetry/2.0</code> User-Agent, which are strong indicators that exfiltration occurred on your infrastructure.</p>



<p>The affected digests are disabled, the repository has been restored to its last known-good state, and pulls of checkmarx/kics today return the legitimate March 3, 2026 image. The publisher account used to push the malicious images has been suspended, and we&#8217;ve notified the small number of users our telemetry shows pulled the compromised digests.<br />Socket&#8217;s technical analysis of the issue is <a href="https://socket.dev/blog/checkmarx-supply-chain-compromise" id="dkr_here-89131" rel="nofollow noopener" target="_blank">here</a>. Their post also covers what appears to be a broader Checkmarx compromise, including recent VS Code extension releases, which is worth reading if your developers use those extensions.</p>



<h2 class="wp-block-heading">How we caught this breach</h2>



<p>Within about half an hour of the push, a new image on a repository we monitor triggered a review. A check against the upstream source found no matching release, and the provenance showed the image had been built from a different source repository created one day before the push. That was enough to quarantine the repository and start forensics with Socket and Checkmarx.</p>



<p>The defense is in correlation, not any single signal. In this episode, we found a new tag without an upstream release, provenance from an unfamiliar source, and a timing pattern that did not appear to match normal publishing behavior. Since we happened to see these signals together, they bought us a narrow window in which to act. It has to be noted that layered defense shortens the window between push and takedown, it does not prevent the push.</p>



<h2 class="wp-block-heading">The bar for this kind of attack has collapsed</h2>



<p>The uncomfortable thing about this incident, and Trivy before it, is how little sophistication incidents such as these require these days. A stolen credential from an IDE extension compromise, a target chosen from a public profile, a push through the normal publishing flow, and the attacker is inside the software supply chain of every organization that pulls that tag. Our assumption is this attack did not require any zero-days, novel tradecraft, or nation-state level budgets. The ingredients are stolen credentials and time, and both are abundant right now.</p>



<p>Every registry, every package manager, and every publisher of any consequence is in the firing line, including Docker. This isn&#8217;t a Checkmarx problem or a Hub problem or an npm problem. It&#8217;s the new baseline, and defenders who aren&#8217;t planning for it as the default case are already behind.</p>



<p>There are two implications for our ecosystem.</p>



<p>Credential hygiene at the publishing boundary matters more than it used to: fine-grained tokens scoped to a single registry, shorter credential lifetimes, clean separation between personal and publisher identities.</p>



<p>And that no single layer will catch all of this. Publishing-time verification, provenance, signatures, registry-side monitoring, deep package inspection (the kind Socket does to catch malicious behavior in dependencies), runtime egress controls, and cross-registry signal correlation each have to do some of the work, because any of them alone will miss cases the others catch.</p>



<h2 class="wp-block-heading">A note on where this is structurally harder</h2>



<p>In the Docker Hardened Images catalog, images are built by Docker from source, with verified provenance and signed releases produced through a hardened build pipeline. The class of attack described above, where a valid publisher credential pushes a tag that diverges from its upstream source, is structurally much harder to execute against an image built this way. There is no external credential that can substitute its way in; the provenance and the signatures have to match, or the image doesn&#8217;t ship. The DHI catalog is expanding, and we’re investing in this layer precisely because of the scenario and reasons explored in this blog. </p>



<h2 class="wp-block-heading">No one catches this alone</h2>



<p>The reason this incident got caught quickly, the reason Socket was able to produce a technical analysis within hours, and the reason Checkmarx&#8217;s response could move in parallel with ours, is that all three teams shared signals and samples in real time. The Trivy response looked the same, as did the rapid notification to GitHub about the attacker-controlled source repository.</p>



<p>This is the posture the ecosystem needs more of, not less. Supply chain attackers are routing  across registries, IDE marketplaces, source hosts, and CI systems in hours. Defenders who don&#8217;t share signals across those same boundaries are operating from a point of disadvantage.  Formal standards for cross-registry coordination are still emerging, and they will matter eventually. What&#8217;s kept the windows short so far has been teams working with a spirit of openness, willingly sharing what they’re discovering, in real time.</p>



<p>Docker will keep investing in layered defenses on Hub, keep extending publishing-time verification to more of the catalog, and keep showing up to share signals, whether this is across a partner&#8217;s incident channel, a peer registry&#8217;s investigation, or the rooms where a more durable framework for coordination eventually takes shape.</p>



<p>We want to thank the Socket research team for fast, independent analysis, and to Checkmarx for moving alongside us on a tight timeline for this one.</p>



<h3 class="wp-block-heading">Further reading</h3>



<p>Socket blog: <a href="https://socket.dev/blog/checkmarx-supply-chain-compromise" id="dkr_httpssocketdevblogcheckmarx-supply-chain-compromise-89131" rel="nofollow noopener" target="_blank">https://socket.dev/blog/checkmarx-supply-chain-compromise</a></p>



<p>Docker Hardened Images on Docker Hub: <a href="https://hub.docker.com/hardened-images/catalog" id="dkr_httpshubdockercomhardened-imagescatalog--89131" rel="nofollow noopener" target="_blank">https://hub.docker.com/hardened-images/catalog </a></p>



<p></p>
