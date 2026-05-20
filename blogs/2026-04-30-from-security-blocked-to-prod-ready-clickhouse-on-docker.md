---
title: "From Security Blocked to Prod Ready: ClickHouse on Docker Hardened Images"
url: "https://www.docker.com/blog/from-security-blocked-to-prod-ready-clickhouse-on-docker-hardened-images/"
date: "Thu, 30 Apr 2026 15:55:07 +0000"
author: "Jennifer Kohl"
feed_url: "https://www.docker.com/blog/feed/"
---
In November 2025, a team self-hosting Langfuse, an open-source LLM observability platform, on Kubernetes uploaded their ClickHouse image to AWS ECR as part of their production preparation. They found that the pipeline scanner had returned three critical vulnerabilities - not in ClickHouse, but in the base image. Their security team saw the findings and blocked...
