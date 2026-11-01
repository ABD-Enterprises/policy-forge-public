# Security Policy

## Public Repository Scope

This repository contains public-safe product documentation, architecture summaries, diagrams, and synthetic examples. It does not contain proprietary application source code, deployment secrets, customer records, internal infrastructure values, or exploit-enabling implementation details.

## Reporting a Vulnerability

Report vulnerabilities privately through GitHub private vulnerability reporting: open this repository's **Security** tab and choose **Report a vulnerability** (<https://github.com/ABD-Enterprises/policy-forge-public/security/advisories/new>). Do not open a public issue for a vulnerability.

## Reporting Security Questions

For public-safe security questions about this repository, open a security-question issue using the provided template.

Do not post secrets, vulnerability exploit details, customer data, private URLs, internal tickets, or live infrastructure identifiers in public issues.

## Validation

Before publishing changes, run:

```bash
./scripts/validate-public-repo.sh
```

The validation scripts check for private-path indicators and use available secret scanning tools when installed.
