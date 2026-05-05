---
title: "From Security Blocked to Prod Ready: ClickHouse on Docker Hardened Images"
url: "https://www.docker.com/blog/from-security-blocked-to-prod-ready-clickhouse-on-docker-hardened-images/"
date: "Thu, 30 Apr 2026 15:55:07 +0000"
author: "Jennifer Kohl"
feed_url: "https://www.docker.com/feed/"
---
<p>In November 2025, a team self-hosting <a href="https://langfuse.com/" id="dkr_langfuse-88880" rel="nofollow noopener" target="_blank">Langfuse</a>, an open-source LLM observability platform, on Kubernetes uploaded their ClickHouse image to AWS ECR as part of their production preparation. They found that the pipeline scanner had returned three critical vulnerabilities &#8211; not in ClickHouse, but in the base image. Their security team saw the findings and blocked the deployment before it ever reached production.</p>



     
<div class="organism wp-block-ponyo-galen">
    <div class="container fade-in left">
        
        
        <div class="galen-content">
            <h3>&#8220;<em>Our security team is not allowing us to take it to production. Please suggest alternatives.</em>&#8220;</h3>
            <div>
                <p class="name">vinaygoel586</p>
                <p class="title"><a href="https://github.com/langfuse/langfuse-k8s/issues/286" rel="nofollow noopener" target="_blank">GitHub Issue #286</a>, November 28, 2025</p>
            </div>
        </div>

    </div>

</div>


<p>If you&#8217;ve shipped containers into an enterprise environment recently, this situation will sound familiar. A perfectly functional deployment gets blocked not because something is broken, but because a scanner found CVEs in packages the application never even touches. A day goes into investigating the findings, a risk exception gets written up, and the security team rejects it anyway, because the vulnerabilities are technically real even if they’re practically irrelevant to your workload.</p>



<p>This post is about how Docker Hardened Images (DHI) gets you unstuck, when a security team blocks the deployment of a container that has CVEs. In this case we will specifically look at the image for ClickHouse, one of the most widely pulled database images on Docker Hub.</p>



<h2 class="wp-block-heading">A Quick Word on ClickHouse</h2>



<p><a href="https://clickhouse.com/" id="dkr_clickhouse-88880" rel="nofollow noopener" target="_blank">ClickHouse</a> is an open-source columnar database built for analytical workloads at scale. It is capable of querying billions of rows and returning results in milliseconds in a way that traditional row-oriented databases simply can&#8217;t match. Companies such as Cloudflare, Uber, and Spotify all run it in production. <a href="https://hub.docker.com/r/clickhouse/clickhouse-server/" id="dkr_with-over-100-million-pulls-from-docker-hub-88880" rel="nofollow noopener" target="_blank">With over 100 million pulls from Docker Hub</a>, it has become the default infrastructure choice for teams that need serious analytics throughput. The image’s default security posture, though, was designed with developer ease-of-use in mind rather than the hardening that enterprise production environments demand and that gap is where the trouble starts.</p>





<div class="wp-block-ponyo-image">
                <img alt="image1 1" class="fade-in" height="1098" src="https://www.docker.com/app/uploads/2026/04/image1-1.png" title="- image1 1" width="1999" />
        </div>



<p><em>Figure: The layered architecture of ClickHouse</em></p>



<h3 class="wp-block-heading">How ClickHouse is Structured</h3>



<p>ClickHouse follows a layered architecture. It is designed for analytical speed at scale. SQL queries arrive over HTTP (port 8123) or TCP (port 9000), then pass through the optimizer which parses into an abstract syntax tree and prunes it before the pipeline executor picks it up and hands the work off to parallel threads. Beneath the query layer sits the MergeTree storage engine, the heart of ClickHouse which stores data in columnar <code>.bin</code> files. It uses a sparse primary index to skip irrelevant granules without reading entire columns, and runs background merge processes to compact parts and maintain query performance over time. </p>



<p>At the bottom, storage is pluggable: local disk, S3, HDFS, or Azure Blob, with tiered hot/warm/cold policies to balance cost and latency. In distributed deployments, ClickHouse Keeper (or ZooKeeper) coordinates replication across replicas, while sharding splits data horizontally across nodes allowing the cluster to scale reads and writes independently. The result is a database that processes hundreds of millions of rows per second per server, making it the default choice for teams running serious analytics workloads.</p>



<h2 class="wp-block-heading">The Real Problem: It&#8217;s Not ClickHouse, It&#8217;s the Packaging</h2>



<p>The standard <code>clickhouse/clickhouse-server</code> image is built on a full Ubuntu 22.04 base. The base ships with a lot of things ClickHouse doesn&#8217;t need such as Perl, system utilities, apt itself, and dozens of transitive dependencies that exist in the image simply because Ubuntu brought outdated package along and in many cases, Ubuntu maintainers decide to not backport fixes from upstream.</p>



<p>ClickHouse doesn&#8217;t use most of those system utilities. But the CVEs in those packages are real. They show up in Trivy, Grype, and AWS ECR has no way to distinguish a vulnerable library that’s never loaded from one that’s actively running in production. Your security team sees critical findings and blocks the deployment, which is the correct thing for them to do given what the scanner is telling them.</p>



<p>The instinct at this point is to argue the case, documenting why each CVE doesn’t apply to your workload, writing risk exceptions and escalating, but that’s a slow process. The only real fix is to remove those unnecessary packages entirely. That&#8217;s what Docker Hardened Images do.</p>



<h2 class="wp-block-heading">What DHI Actually Changes</h2>



<p><a href="https://docs.docker.com/dhi/" id="dkr_docker-hardened-images-88880" rel="nofollow noopener" target="_blank">Docker Hardened Images</a> for ClickHouse are built around a straightforward question: what does the database actually need to run? Rather than starting from a full Ubuntu base and hoping the CVE count stays manageable, DHI ships only what ClickHouse requires and leaves everything else out.</p>



<p>The most immediate consequence of that is the absence of <code>apt</code> at runtime. Without a package manager, an attacker who gains a foothold in the container has no obvious path to installing tools or establishing persistence. Network utilities like <code>curl</code> and <code>wget</code> are gone for the same reason, the standard <code>clickhouse/clickhouse-server</code> image has been carrying wget with <code>CVE-2021-31879</code> unpatched since 2021 because there is no upstream fix as noted by the <a href="https://ubuntu.com/security/CVE-2021-31879" id="dkr_ubuntu-maintainer-88880" rel="nofollow noopener" target="_blank">Ubuntu maintainer</a>, a vulnerability in a tool ClickHouse never needed in the first place. DHI doesn&#8217;t patch it; it simply doesn&#8217;t include <code>wget</code> at all. A shell is still available for operational work, but without the package manager and network tools, there&#8217;s very little an attacker can actually do with it.</p>



<p>To make this practical across different stages of a pipeline, DHI ships two variants. The development image (dev) includes additional tooling that makes local testing and debugging more comfortable. The production image (runtime) strips that back to the absolute minimum, giving you the smallest possible attack surface for the workload that actually faces the world. The intent is that teams adopt the dev variant early in the pipeline and promote the hardened production image through to deployment, rather than discovering the differences at the point where it matters most.</p>



<p>The image also runs as a non-root user <code>uid=65532</code> out of the box, with no additional Dockerfile configuration required. On the provenance side, every DHI image ships with SLSA Level 3 attestation, which provides cryptographic proof of exactly what went into the build and how it was produced. Docker&#8217;s security team actively tracks and patches CVEs, and the presence of 2026 CVE IDs in DHI&#8217;s findings is evidence of that remediation happening ahead of public disclosure feeds rather than in response to them.</p>



<h2 class="wp-block-heading">Getting Started</h2>



<p>Before you can pull a DHI image, you need to mirror it to your organization&#8217;s namespace on Docker Hub. This is a one-time setup per image not per tag and it means all future updates flow to your namespace automatically.</p>



<ol class="wp-block-list">
<li>Log in to Docker Hub and open the DHI catalog</li>



<li>Find <code>clickhouse-server</code> and select <strong>Mirror to repository</strong></li>



<li>Follow the on-screen instructions</li>



<li>Authenticate locally: <code>docker login dhi.io</code></li>
</ol>



<p>Once that&#8217;s done, you&#8217;re pulling from your own namespace with the same image, same tags, same ClickHouse &#8211; just hardened.</p>



<h3 class="wp-block-heading">Your first DHI ClickHouse container</h3>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker run --name my-clickhouse-server -d \
  --ulimit nofile=262144:262144 \
 dhi.io/clickhouse-server:26.2-debian13
</pre></div>


<p>The <code>--ulimit nofile=262144:262144</code> flag is a ClickHouse requirement, not a DHI one &#8211; ClickHouse needs high file descriptor limits to operate correctly. Keep it in all your run commands.</p>



<p>Verify it started:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker exec my-clickhouse-server clickhouse-client \
  --query &quot;SELECT &#039;Hello from DHI ClickHouse!&#039;&quot;
</pre></div>


<h3 class="wp-block-heading">Production setup with persistent storage</h3>



<p>For anything beyond local testing, you want volumes and a password:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker run -d \
  --name my-clickhouse-server \
  --ulimit nofile=262144:262144 \
  -e CLICKHOUSE_PASSWORD=mysecretpassword \
  -v clickhouse-data:/var/lib/clickhouse \
  -v clickhouse-logs:/var/log/clickhouse-server \
  -p 8123:8123 -p 9000:9000 \
  dhi.io/clickhouse-server:26.2-debian13
</pre></div>


<p>Note that <code>CLICKHOUSE_PASSWORD</code> is required if you want to access ClickHouse over the network. DHI disables unauthenticated network access by default which is the right call for any production deployment.</p>



<p>Test it over HTTP:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
curl &quot;http://localhost:8123/?query=SELECT%20version()&amp;user=default&amp;password=mysecretpassword&quot;
</pre></div>


<h3 class="wp-block-heading">Custom configuration</h3>



<p>If you&#8217;re already running ClickHouse with custom XML config, nothing changes. Same format, same mount path:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
cat &gt; custom-config.xml &lt;&lt; EOF
&lt;clickhouse&gt;
    &lt;logger&gt;
        &lt;level&gt;information&lt;/level&gt;
        &lt;console&gt;true&lt;/console&gt;
    &lt;/logger&gt;
    &lt;listen_host&gt;0.0.0.0&lt;/listen_host&gt;
&lt;/clickhouse&gt;
EOF

docker run -d \
  --name my-clickhouse-server \
  --ulimit nofile=262144:262144 \
  -v $(pwd)/custom-config.xml:/etc/clickhouse-server/config.d/custom.xml:ro \
  -p 8123:8123 -p 9000:9000 \
  dhi.io/clickhouse-server:26.2-debian13

</pre></div>


<h2 class="wp-block-heading">Running DHI ClickHouse on Kubernetes</h2>



<p>For Kubernetes, there&#8217;s one important addition to your pod spec. Since DHI runs as a non-root user, you need to set <code>fsGroup</code> to ensure your persistent volume data is accessible:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: plain; gutter: false; title: ; notranslate">
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532     # DHI nonroot user
        fsGroup: 65532       # makes mounted volumes accessible to the nonroot user
      containers:
      - name: clickhouse-server
        image: dhi.io/clickhouse-server:26.2-debian13
        ports:
        - containerPort: 8123
        - containerPort: 9000
        volumeMounts:
        - name: clickhouse-data
          mountPath: /var/lib/clickhouse
        - name: clickhouse-logs
          mountPath: /var/log/clickhouse-server
        resources:
          limits:
            cpu: &quot;2&quot;
            memory: &quot;4Gi&quot;

</pre></div>


<p>One thing worth mentioning: ClickHouse&#8217;s default ports 8123 and 9000 are above the 1024 privileged port boundary, so running as nonroot doesn&#8217;t cause any port binding issues.</p>



<h3 class="wp-block-heading">The metrics exporter</h3>



<p>If you&#8217;re running ClickHouse on Kubernetes and need Prometheus metrics, Docker also ships <code>clickhouse-metrics-exporter</code> &#8211; a hardened image that works with the ClickHouse Operator to expose a <code>/metrics</code> endpoint. It&#8217;s 65% smaller than the standard exporter (10.3 MB vs 29.4 MB) and has 75% fewer layers (5 vs 20). Same data, dramatically smaller surface.</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: plain; gutter: false; title: ; notranslate">
containers:
- name: metrics-exporter
  image: dhi.io/clickhouse-metrics-exporter:0-debian13
  ports:
  - name: metrics
    containerPort: 8888
  resources:
    limits:
      cpu: 100m
      memory: 128Mi
    requests:
      cpu: 50m
      memory: 64Mi

</pre></div>


<h2 class="wp-block-heading">Debugging without the usual tools</h2>



<p>The debugging story is simpler than it might seem. <code>docker debug</code> attaches an ephemeral layer to the running container that includes bash, curl, strace, vim, and anything else you need without modifying the production image itself. When you exit, the layer disappears and the container is exactly as it was. It&#8217;s a cleaner approach than shelling directly into a production container, and in practice it&#8217;s a single command:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker debug my-clickhouse-server
</pre></div>


<p>Or if you prefer, you can mount a debug image alongside the container:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker run --rm -it --pid container:my-clickhouse-server \
  --mount=type=image,source=&lt;your-namespace&gt;/dhi-busybox,destination=/dbg,ro \
  dhi.io/clickhouse-server:26.2-debian13 /dbg/bin/sh
</pre></div>


<p>There&#8217;s also a broader security benefit that goes beyond CVE counts. If something does go wrong in production, an attacker who gets into the container finds no package manager to install tools with, no curl or wget to exfiltrate data through, and no obvious path to reach out to the network which significantly limits what a compromise can actually turn into.</p>



<h2 class="wp-block-heading">ClickHouse: Non-hardened Image vs. Hardened Image Compared</h2>



<p>A <a href="https://www.docker.com/products/docker-scout/" id="dkr_docker-scout-88880">Docker Scout</a> scan of both images puts the difference in plain numbers. Using <code>ubuntu:22.04</code> as its base, the standard image carries <code>8</code> medium and <code>11</code> low severity vulnerabilities across 111 packages, including the wget and tar findings that are most likely to trigger a security block in an enterprise pipeline. The DHI image eliminates all medium severity findings entirely and comes in at <code>14</code> low severity items but these are in core system libraries like <code>glibc</code> and <code>openssl</code> where no fix exists on any distribution, not in unnecessary utilities that had no business being in the image. The <code>3</code> unconfirmed findings that Scout surfaces have already been assessed and suppressed via VEX attestation, which ships with the image as part of its SLSA Level 3 provenance</p>



<p>To view the difference between versions for any other image, you can run your own scan with Docker Scout for a quick comparison using this command:</p>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: bash; gutter: false; title: ; notranslate">
docker scout quickview clickhouse/clickhouse-server:latest

docker pull dhi.io/clickhouse-server:26.2-debian13
docker tag dhi.io/clickhouse-server:26.2-debian13 clickhouse-dhi:latest
docker scout quickview clickhouse-dhi:latest

</pre></div>




<div class="wp-block-ponyo-image">
                <img alt="image2" class="fade-in" height="244" src="https://www.docker.com/app/uploads/2026/04/image2.png" title="- image2" width="1999" />
        </div>



<p></p>





<div class="wp-block-ponyo-image">
                <img alt="image3" class="fade-in" height="164" src="https://www.docker.com/app/uploads/2026/04/image3.png" title="- image3" width="1999" />
        </div>



<p></p>



<p></p>



 <div class="wp-block-ponyo-table">
    <table class="responsive-table">
        


 


                                                            <tbody class="wp-block-ponyo-table-body">
    


 
<tr class="wp-block-ponyo-table-header">
    


        <th class="wp-block-ponyo-cell empty">
        

<p></p>


            </th>




        <th class="wp-block-ponyo-cell">
        

<p><span>Non-Hardened  ClickHouse Image</span></p>


            </th>




        <th class="wp-block-ponyo-cell">
        

<p><span>Docker Hardened Image</span></p>


            </th>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Default user</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>root (steps down to clickhouse user at runtime via entrypoint, but Dockerfile has no USER directive overridable with CLICKHOUSE_RUN_AS_ROOT=1)</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>nonroot (enforced at image level via USER directive cannot be overridden at runtime)</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Shell access</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Full shell (bash/sh) available</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>bash present, no network tools or package manager</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Package manager</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>apt available</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>No package manager</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CVE exposure</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Ships wget (</span><a href="https://ubuntu.com/security/CVE-2021-31879#status" id="dkr_cve-2021-31879-88880" rel="nofollow noopener" target="_blank"><span>CVE-2021-31879</span></a><span>, unpatched since 2021), tar (</span><a href="https://ubuntu.com/security/CVE-2025-45582" id="dkr_cve-2025-45582-88880" rel="nofollow noopener" target="_blank"><span>CVE-2025-45582</span></a><span>)</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>No wget, no tar &#8211; unnecessary packages removed entirely</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CVE patching</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Unpatched findings from 2021–2025 due to the lack of upstream fixes from Ubuntu base image.</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Actively tracked, 2026 CVE IDs show proactive remediation</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Provenance</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Standard</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>SLSA Level 3 attestation</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Compliance</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Manual hardening required</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>CIS, NIST, FedRAMP-aligned</span></p>


                    </span>
                                            </span>
            </td>


</tr> 



 
<tr class="wp-block-ponyo-table-row">
    


    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Debugging</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Traditional shell debugging</span></p>


                    </span>
                                            </span>
            </td>




    <td class="wp-block-ponyo-cell">
                    <span class="responsive-table-label"></span>
                
                    <span class="responsive-table-value">
                                                    <span class="responsive-table-value-content">
                        

<p><span>Use docker debug or Image Mount for troubleshooting</span></p>


                    </span>
                                            </span>
            </td>


</tr> 

</tbody>


    </table>
</div>



<p></p>



<h2 class="wp-block-heading">The Security Team Conversation</h2>



<p>The team that got blocked at AWS ECR in November 2025 didn’t have a ClickHouse problem, they had a base image problem. Their database was fine; what the scanner was finding were CVEs in Perl, system utilities, and other packages that had come along in the Debian base and never used by the application. Nothing in the scanner output made that distinction, so the security team did exactly what they were supposed to do and blocked the deployment.</p>



<p>With DHI, that conversation with your security team becomes considerably more straightforward. Rather than building a case for why specific CVEs don&#8217;t apply to your workload, you can point to an image built by Docker&#8217;s security team from the minimum required components, with SLSA Level 3 provenance and independent validation by SRLabs. The ClickHouse runtime itself is unchanged ~ queries, ports, configuration files, and performance all carry over so the only thing you&#8217;re actually changing is the answer you can give when someone asks whether this image can go to production.For teams that need stronger guarantees, DHI Enterprise adds SLA-backed CVE remediation within seven days, FIPS and STIG variants, and extended lifecycle support. For most teams, the <a href="https://hub.docker.com/hardened-images/start-free-trial" id="dkr_free-enterprise-trial-88880" rel="nofollow noopener" target="_blank">free Enterprise trial</a> is the right starting point. It answers the question that actually matters before you commit to anything. Interested to learn further? Start with this blog that <a href="https://www.docker.com/blog/making-the-most-of-your-docker-hardened-images-trial-part-1/" id="dkr_walks-through-88880">walks through</a> the trial and sets you up for success.</p>



<h2 class="wp-block-heading">Migration Checklist</h2>


<div class="wp-block-syntaxhighlighter-code "><pre class="brush: plain; gutter: false; title: ; notranslate">
☐ Mirror clickhouse-server DHI image to your Docker Hub namespace (one-time setup)
☐ Update your image reference to dhi.io/clickhouse-server:26.2-debian13
☐ Set CLICKHOUSE_PASSWORD (required for network access in DHI)
☐ Keep --ulimit nofile=262144:262144 on all run commands
☐ In Kubernetes: add fsGroup: 65532 to your pod securityContext
☐ Switch from kubectl exec to kubectl debug for troubleshooting
☐ Run trivy against both images to see the difference yourself:
     trivy image clickhouse/clickhouse-server:latest
     trivy image dhi.io/clickhouse-server:26.2-debian13

</pre></div>


<p>The migration is narrower in scope than it might appear &#8211; your volume mounts, port mappings, and existing XML configuration files all carry over without modifications, and on Kubernetes the only structure addition is the fsGroup security context. Everything else is an image reference change.</p>



<h2 class="wp-block-heading">Resources</h2>



<ul class="wp-block-list">
<li><a href="https://docs.docker.com/dhi/" id="dkr_docker-hardened-images-documentation-88880" rel="nofollow noopener" target="_blank">Docker Hardened Images Documentation</a></li>



<li><a href="https://hub.docker.com/hardened-images/catalog/dhi/clickhouse-server/guides" id="dkr_dhi-clickhouse-server-guide-88880" rel="nofollow noopener" target="_blank">DHI ClickHouse Server Guide</a></li>



<li><a href="https://hub.docker.com/hardened-images/catalog/dhi/clickhouse-metrics-exporter/guides" id="dkr_dhi-clickhouse-metrics-exporter-guide-88880" rel="nofollow noopener" target="_blank">DHI ClickHouse Metrics Exporter Guide</a></li>



<li><a href="https://docs.docker.com/reference/cli/docker/debug/" id="dkr_docker-debug-documentation-88880" rel="nofollow noopener" target="_blank">Docker Debug Documentation</a></li>



<li><a href="https://hub.docker.com/hardened-images/catalog" id="dkr_free-dhi-catalog-88880" rel="nofollow noopener" target="_blank">Free DHI Catalog</a></li>



<li><a href="https://www.docker.com/blog/docker-hardened-images-for-every-developer/" id="dkr_dhi-community-announcement-88880">DHI Community Announcement</a></li>



<li><a href="https://docs.docker.com/scout/" id="dkr_docker-scout-documentation-88880" rel="nofollow noopener" target="_blank">Docker Scout Documentation</a></li>
</ul>
