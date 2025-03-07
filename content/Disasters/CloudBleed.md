---
title: CloudBleed - 2017 Cloudflare Data Leak
---

# [CloudBleed (2017)](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/)

## Declaration

On February 18, 2017, at 4:11 PM PST, a security researcher from Google's Project Zero discovered a critical vulnerability in Cloudflare's infrastructure. The issue was severe enough that he immediately reached out to Cloudflare, and by 4:32 PM, the company was made aware of a possible widespread data leak. This incident was later described as the "Cloudbleed" vulnerability, which was explained in detail by  on February 23, 2017.

Cloudflare, a major Content Delivery Network (CDN) provider, had been unknowingly exposing sensitive customer data, including cookies, passwords, HTTPS requests, and private keys. This data leakage occurred due to a bug in how Cloudflare's edge servers processed HTML content. Worse, search engines like Google had already cached some of the leaked data, making it publicly accessible ([The Hacker News](https://thehackernews.com/2017/02/cloudflare-vulnerability.html)).

## Initial Investigation & Mitigation Efforts

Upon discovery, Cloudflare engineers quickly convened and correlated the bug with the email obfuscation feature, which had recently undergone a partial migration to a new HTML parser. They disabled this feature globally, but the bug persisted. Further investigation identified two more problematic features:
- Automatic HTTP rewrites
- Server-side excludes

Each of these features also processed HTML content dynamically on the edge servers. While the first two features had global kill switches and were disabled immediately, server-side excludes was an older feature that lacked such a mechanism. Engineers had to develop and deploy a patch to disable it. For further details, please refer to the post-mortem analysis found in [Quantifying the Impact of "Cloudbleed"](https://blog.cloudflare.com/quantifying-the-impact-of-cloudbleed/).

Despite these measures, the root cause remained unclear, and there was a lingering risk of reoccurrence.

## Root Cause Analysis
The engineers determined that all three affected features used Cloudflare’s new HTML parser (cf-html), which replaced an older parser generated with Ragel, a finite state machine-based parser. However, the bug actually originated in the old Ragel-based parser, which had been in use for years without causing issues.

The bug was triggered by an edge case involving unclosed HTML attributes at the end of a data buffer. In such cases, the parser would attempt to reprocess an attribute but fail to check for buffer boundaries correctly. A pre-increment operation caused the parser’s pointer (p) to skip over the check that should have stopped it from reading past valid memory. As a result, heap memory beyond the allocated buffer was accessed, leading to unintended data exposure ([Cloudflare Incident Report](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/)). The new parser’s migration process inadvertently triggered this pre-existing bug by handling buffers differently, exposing memory that the old system never accessed.

