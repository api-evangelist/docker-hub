---
title: "Why We Chose the Harder Path: Docker Hardened Images, One Year Later"
url: "https://www.docker.com/blog/why-we-chose-the-harder-path-docker-hardened-images-one-year-later/"
date: "Tue, 14 Apr 2026 21:48:03 +0000"
author: "Aditya Tripathi"
feed_url: "https://www.docker.com/feed/"
---
<p>We&#8217;re coming up on a year since launching Docker Hardened Images (DHI) last May, and crossing a milestone earlier this month made me stop and reflect on what we&#8217;ve actually been building.</p>



<p>Earlier this month, we crossed over 500k daily pulls of DHIs, and over 25k continuously patched OS level artifacts in our SLSA Build Level 3 pipeline. From the time we launched the free DHI Community tier at the end of last year, the catalog has now grown to 2,000+ hardened images, MCP servers, Helm charts, and ELS images. We continuously patch every artifact (across CVEs, distros, versions), so we’re now running over a million builds regularly, and just getting started. Catalog coverage will jump again soon, as more Debian packages, ELS images, and newer artifact types are added.</p>



<p>But the numbers aren&#8217;t the interesting part. What matters is how we got here.</p>



<p>We chose the harder path, on purpose. Every product and engineering decision we made was consistently harder to build and operate, but better for developers and for the security of the ecosystem. We made hardened images free and open source. We built a multi-distro product, so adoption doesn&#8217;t mean migrating to a vendor&#8217;s proprietary OS. We build every system package from source for distros you already run. We ship a huge range of signed attestations with every image because that&#8217;s what independent verifiability actually requires.</p>



<p>Along the way, we also looked closely at how the rest of the industry approaches the same problems, and found patterns in patching timelines, SBOM completeness, and advisory coverage that are worth understanding before you evaluate any hardened image provider.</p>



<h2 class="wp-block-heading">We made hardened images widely accessible so every team could raise their security baseline</h2>



<p>We wanted to make a real dent in the security posture of the internet, and that meant making hardened images widely accessible. That is why we did not put our catalog behind a gated paywall, as was the industry norm, but freely available to every developer.</p>



<p>Building and sustaining a hardened image pipeline at this scale isn&#8217;t trivial. We know because we&#8217;ve been doing this for over a decade with Docker Official Images, freely for the community. </p>



<p>With the release of DHI Community under a permissive Apache 2.0 license, we raised the baseline for security across the ecosystem. Security should not be a premium feature. That kind of impact, at scale, is only possible because the foundation is open.</p>



<h2 class="wp-block-heading">We built multi-distro so adoption is drop-in, and does not impose a migration tax on you</h2>



<p>Some vendors in this space created an entirely new Linux distribution and called it &#8220;distroless,&#8221; which is a remarkable piece of branding for what is, in practice, a proprietary OS that your teams have never run, tested, or audited. Established Linux distributions like Debian and Alpine have a name for a package repository that only tracks the latest upstream version: they call it &#8220;unstable&#8221; or &#8220;edge,&#8221; not stable.</p>



<p>Docker doesn&#8217;t ship its own distribution, we harden the ones you already trust. That decision optimizes for your engineering reality, not ours. The hardened image that never gets adopted provides zero security value, full stop. </p>



<p>With the Docker “multi-distro” approach, we support both Debian and Alpine today, with support for more distros to come. This is actually hard to do: the Debian and Alpine ecosystems don&#8217;t just differ in packaging; they diverge in libc, dependency trees, CVE streams, patch timing, and tooling. You are effectively maintaining parallel supply chains, each with its own nuances and security posture. Every hardened image in the DHI catalog is available in both Alpine and Debian, across both amd64 and arm64 architectures, which means we build, patch, and attest each combination independently, taking on that operational burden so you don&#8217;t have to.</p>



<p>We regularly speak with engineering teams who evaluate proprietary distributions from other vendors and run into the same wall: your existing internal expertise, tools, tests, and pipelines are built around Alpine or Debian.</p>



<p>Migrating to an unfamiliar, vendor-owned OS isn&#8217;t a security upgrade, it&#8217;s an adoption project and a material line item of cost, alongside the sticker price of the hardened images subscription itself. The vendor lock-in aspect goes without saying.</p>



<p>The migration effort means revalidating CI pipelines, retraining platform teams, auditing an entirely new package ecosystem, and working through compatibility gaps that surface weeks into a rollout. Several teams tell us they bought the migration story, spent months on it, and are still paying for images their engineers haven&#8217;t adopted. With Docker, your teams stay on the distros they already run, which means the adoption cost is measured in hours, not quarters.</p>



<p>One of our customers at Attentive (Stephen Commisso, Principal Engineer) captured their experience in the phrase “200 services &#8211; zero drama”, when describing their DHI rollout:</p>



<p><em>“The rollout was completely transparent to product teams. We had zero issues across over 200 services, which was particularly impressive since we were simultaneously switching Linux distributions from Ubuntu to Debian. All the heavy lifting happened during POC.”</em></p>



<h2 class="wp-block-heading">We build every system package from source, for the distros you already use</h2>



<p>With the launch of Hardened System Packages, Docker builds tens of thousands of Alpine and Debian system packages from source in a SLSA Build Level 3 pipeline with cryptographic signed, full provenance. This fundamentally changes the CVE equation.</p>



<p>Other vendors also claim to build system packages from source. The difference is that they build them for proprietary Linux distributions that have not had the benefit of independent community scrutiny and that customers have never run in production.</p>



<p>Docker builds packages for Alpine and Debian, the distributions your teams already operate, already test against, and already trust. Alpine and Debian are vast ecosystems that have independent maintainers, public mailing lists, coordinated disclosure with upstream projects, and volunteer security teams that operate independently of any commercial interest. You get the security benefit of from-source patching without the compatibility cost of adopting an unfamiliar OS.</p>



<h2 class="wp-block-heading">We didn&#8217;t stop at near-zero CVEs, we made every image independently verifiable</h2>



<p>Docker&#8217;s approach to container security is built on <a href="https://www.docker.com/blog/100-transparency-and-five-pillars/" id="dkr_five-pillars-88726">five pillars</a>: minimal attack surface, verifiable SBOMs, secure build provenance, exploitability context, and cryptographic verification. We distilled our product development philosophy to these ideas, because we think your security posture depends on it. Not every vendor in the hardened image market shares this philosophy.</p>



<p>Most vendors in this space optimize for one metric: a clean CVE scan result.</p>



<p>Docker obsesses over near-zero CVEs too, but we went further: we built an attestation infrastructure that gives your security team, auditors, SOC, and change advisory boards machine-readable, cryptographically signed evidence for every question they will ask about an image.</p>



<p>We add <strong>17 signed attestations</strong> to every single one of our 2000+ images in the DHI catalog, because that is what it takes to <strong>give you independent verifiability</strong>:</p>



 <div class="wp-block-ponyo-table">
    <table class="responsive-table">
        


 


                                                                                <tbody class="wp-block-ponyo-table-body">
    


 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>Question</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><strong>DHI included <span>attestation(s)</span></strong></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>What this attestation is</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>Why it matters to you</strong></span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>What&#8217;s in this image?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CycloneDX SBOM, SPDX SBOM</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Machine-readable inventory of every package, version, and transitive dependency. </span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>First thing auditors request during compliance reviews. Both formats are included so you don’t have to convert for different toolchains.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>How was it built, and can I prove it?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>SLSA provenance v1, SLSA verification summary, Scout provenance, DHI Image Sources</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Cryptographic proof linking every image to its exact source definition. </span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Required by supply chain security policies. Used by incident responders during forensics to verify whether an image was legitimately built or injected.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>What vulnerabilities exist, and what&#8217;s been assessed?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CVEs v0.1, CVEs v0.2, VEX, Scout health score</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CVE scan results and per-CVE exploitability justifications attached to the image itself. </span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>When your GRC team prepares a FedRAMP POA&amp;M or your security team triages a new advisory, the evidence is already on the artifact.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>Is it compliant?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>FIPS compliance, STIG scan</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>FIPS evidence and OpenSCAP-generated STIG results</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Ready artifacts for FedRAMP, PCI DSS, and HIPAA audits. Typically the most expensive artifacts to produce manually. Docker generates them automatically.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>Has it been checked for non-CVE risks?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Secrets scan, Virus scan, Tests</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Confirms no leaked credentials, no known malware, and that the image functions as expected. </span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>These are the checks SOC teams and security review boards require before approving production deployment. Docker runs them on every build.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span><strong>What changed?</strong></span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Changelog</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Signed record of what was added, removed, or patched between versions.</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span> Change advisory boards need this to approve updates. Without it, your team is diffing images manually.</span></p>


                    </span>
                                            </span>
            </td>


</tr> 

</tbody>


    </table>
</div>



<p>Attestations answer questions about the image, the next set of questions are about the vendor.</p>



<h2 class="wp-block-heading">What to ask your vendor, and what we found when we asked ourselves the same questions</h2>



<p>In a fast-moving ecosystem, CVEs will occasionally get missed, advisories will have gaps, and no vendor operating at scale will have a flawless record. What matters is whether the gaps reveal isolated incidents or a pattern. The following questions are worth asking every vendor, including Docker.</p>



<h3 class="wp-block-heading">What is the extent of your vendor&#8217;s commitment to patching?</h3>



<p>Ask your vendor how far they go to resolve vulnerabilities. Docker continuously patches CVEs across both Debian, Alpine, and several major OSS software projects, rebuilding tens of thousands of system packages and several thousand images from source. That is a significant engineering and operational investment that most vendors avoid, because it is easier to build images for a single proprietary OS.</p>



<p>Docker&#8217;s commitment doesn&#8217;t end at images in our catalog. When a fix doesn&#8217;t exist upstream, there are many examples of Docker&#8217;s security team creating one. For CVE-2025-12735, a 9.8 CRITICAL RCE in Kibana&#8217;s dependency chain, the affected library was unmaintained and had no patch. Docker created the fix, shipped it to customers, and contributed it to LangChain.js. The fix was released as a public npm package on November 17, 2025.</p>



<p>One vendor we looked at has a published CVE policy of 7-day remediation for critical CVEs, once a qualifying patch is publicly available. In this instance, their fix appeared several weeks after that qualifying patch was created by Docker and shipped by the upstream project.</p>



<p>This level of upstream commitment is built into how our security team operates. Docker has been a MITRE CVE Numbering Authority <a href="https://www.docker.com/blog/docker-becomes-mitre-cna/" id="dkr_since-2022-88726">since 2022</a>, part of a sustained investment in our ability to identify, disclose, and fix vulnerabilities at the source.</p>



<h3 class="wp-block-heading">What assurances do you have about the completeness of your SBOMs?</h3>



<p>Ask whether your vendor&#8217;s SBOM includes compiled dependencies (Rust crates, Go modules, JavaScript packages), or just system-level packages. Ask whether you can independently verify SBOM completeness against the project&#8217;s actual dependency manifest. Docker&#8217;s SBOMs include every compiled dependency. We’ve examined images from other vendors, and as one example for Vector (observability pipeline compiled from hundreds of Rust crate dependencies) one vendor&#8217;s SBOM did not appear to include those dependencies.</p>



<p>If a dependency isn&#8217;t in the SBOM, vulnerabilities in that dependency are invisible to the customer&#8217;s scanner and unverifiable by the customer&#8217;s security team. When Docker&#8217;s security team identified a High-severity CVE in Vector&#8217;s Rust dependencies, it was patched and shipped the same evening.</p>



<h3 class="wp-block-heading">Does your vendor&#8217;s advisory feed surface every known CVE for the packages it ships?</h3>



<p>Ask whether you can scan the vendor&#8217;s images with a third-party scanner against public advisory data, without relying on the vendor&#8217;s own advisory feed, and still get consistent results.</p>



<p>Docker recommends validating with Grype, Trivy, Wiz, or Mend. When we examined a vendor’s node image: CVE-2025-9308 and CVE-2025-8262 (both affecting yarn 1.22.22) were present in the shipped image but neither appeared on the vendor&#8217;s vulnerability page or in their security advisory feed. Docker&#8217;s hardened system package for yarn 1.22.22 is built from source with patches applied for both CVEs.</p>



<p>If your vendor&#8217;s advisory feed has gaps, your scanner inherits those gaps, and your security team is making decisions based on incomplete data.</p>



<h3 class="wp-block-heading">When a CVE is assessed as unexploitable, does your vendor provide an auditable justification?</h3>



<p>Not every CVE warrants a patch, and every vendor makes that judgment call. The question is whether your team can see the reasoning. Docker&#8217;s security team evaluates exploitability in the context of each minimal container image and publishes every assessment transparently.</p>



<p>Some vendors may set advisory version ranges to values real packages never match, thereby making CVEs invisible to scanners, and not providing a justification or an audit trail.</p>



<p>Docker uses VEX, the CISA-backed standard for communicating exploitability, which provides a per-CVE, machine-readable justification that every customer can read and audit.</p>



<h2 class="wp-block-heading">We took on the parts of supply chain security others leave behind</h2>



<p>Beyond multi-distro support, from-source patching, and transparency, we made a set of choices that compound into a distinctive, secure, simple experience for you.</p>



<p>Most vendor guarantees stop at the edge of the base image. Docker takes full ownership of your customized images: you add what your environment needs, and when a CVE is patched upstream, Docker automatically rebuilds your customized image and our SLA propagates to every artifact we produce. Your customizations don&#8217;t void the security guarantee. We’ve also opened up our hardened systems packages repo so you can use those hardened packages in your own bespoke containers. </p>



<p>We will be extending this same rigor to language libraries next. The dependencies your application pulls in through npm, pip, or Maven will carry the same provenance and patching guarantees as the OS layer beneath them.</p>



<p>And for organizations running software that upstream has stopped supporting, Extended Lifecycle Support continues delivering security patches for up to five years past end-of-life, so teams can maintain their security posture while upgrading on their own timeline. </p>



<h2 class="wp-block-heading">Come join the movement</h2>



<p>A year ago, 500k daily pulls of the DHI catalog and a million builds running regularly felt like a milestone. Today, this is the baseline.</p>



<p>None of this would have happened without the teams who trusted us early and pushed us hard, including Adobe, Crypto.com, Attentive, and many others. Projects like n8n.io helped us understand what it takes to operate at scale. Partners like Socket.dev, Snyk, and Mend.io are building security workflows on top of this foundation.</p>



<p>We are continuing to listen, iterate, and do the hard things that are better for you, because that matters. If you are thinking about supply chain security, especially given the quantity and intensity of supply chain risks AI agents bring to the mix, now is the time to raise your baseline with Docker.</p>



<p>Explore the Docker Hardened Images catalog and secure your supply chain here: <a href="https://www.docker.com/products/hardened-images/" id="dkr_httpswwwdockercomproductshardened-images-88726">https://www.docker.com/products/hardened-images/</a></p>



<p>For every team and developer, the open source DHI Community tier provides an immediately upgraded security posture. For businesses, we have a wide range of options that will work for your specific needs.</p>



<p><strong>More resources:</strong></p>



<ul class="wp-block-list">
<li><strong>DHI documentation:</strong> <a href="https://docs.docker.com/dhi/" id="dkr_httpsdocsdockercomdhi-88726" rel="nofollow noopener" target="_blank">https://docs.docker.com/dhi/</a></li>



<li><strong>Watch: </strong><a href="https://www.docker.com/resources/how-n8n-uses-docker-hardened-images-webinar/" id="dkr_why-n8nio-moved-to-dhi-88726">Why <strong>n8n.io</strong> moved to DHI</a> </li>



<li><strong>Read: </strong><a href="https://www.docker.com/blog/medplum-healthcare-docker-hardened-images/" id="dkr_medplums-step-by-step-dhi-adoption-playbook-88726">Medplum&#8217;s step-by-step <strong>DHI adoption playbook</strong></a> </li>
</ul>



<p></p>



<p></p>
