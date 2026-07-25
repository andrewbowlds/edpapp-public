# Security Policy

## What this repository is

This repository is a **sanitized public technical overview**, published for portfolio purposes. It contains documentation, Mermaid diagrams, and fictional example data describing the design of a private production system (the "EDP ecosystem"). It does not contain that system's source code, credentials, infrastructure configuration, or any real user data.

There is nothing in this repository to exploit — no running application, no API, no dependencies with a supply chain to attack. This file exists mainly so the repository has a normal, complete security posture, and to give a reporting path in the unlikely event someone finds something in here that shouldn't be public.

## Reporting a concern about this repository

If you find anything in this repository that looks like it might be real (a credential, a real name, a real address, an internal URL, a document ID, or any other data that doesn't look fictional), please do not open a public issue. Instead, use GitHub's private vulnerability-reporting feature after publication or contact me through [LinkedIn](https://www.linkedin.com/in/andrew-bowlds-122875125/).

I'll investigate and remove the content promptly. See [`SECURITY_REVIEW.md`](SECURITY_REVIEW.md) for the audit process this repository went through before publication, and what it specifically checked for.

## Reporting a vulnerability in the live Euphoric Development Partners systems

If you've found a security issue in the actual production Euphoric Development Partners platform (not this documentation repository), please do not test further or attempt to access data beyond what's needed to demonstrate the issue. Report it privately through GitHub's vulnerability-reporting feature or contact me through [LinkedIn](https://www.linkedin.com/in/andrew-bowlds-122875125/).

I'll acknowledge reports and work on a fix; this is a small, operator-run system without a formal bug-bounty program, but genuine reports are taken seriously and I'll respond personally.
