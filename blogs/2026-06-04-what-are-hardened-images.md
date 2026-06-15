---
title: "Hardened Images Explained: Fewer CVEs, Smaller Attack Surface"
url: "https://www.docker.com/blog/what-are-hardened-images/"
date: "2026-06-04"
author: "Aditya Tripathi"
feed_url: "https://www.docker.com/blog/feed/"
---
When security teams scan their container environments for the first time, they often discover hundreds of known vulnerabilities, and almost none of them trace back to application code — the overwhelming majority come from packages that shipped with the base image. Hardened images address this software supply chain security problem at the source by stripping down base images to only the runtime components an application needs, continuously patching them, and shipping with verifiable metadata. This post explains what hardened images are, how they differ from standard base images, and why they matter for reducing CVE counts and attack surface.
