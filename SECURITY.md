# Security

Report vulnerabilities privately through this repository's
[Security tab](https://github.com/shinyobjectz/raea/security), using **Report a
vulnerability**. Do not publish exploit details, vulnerable private assets,
credentials or sensitive logs in an issue, PR, screenshot or development log.

GitHub private vulnerability reporting is a repository setting, separate from
this file. If the reporting button is unavailable in the repository you are
using, ask its maintainers for a private security contact. A public request for
that contact should contain no vulnerability details. See
[GitHub's reporting instructions](https://docs.github.com/en/code-security/how-tos/report-and-fix-vulnerabilities/report-privately).
For a fork, report to the maintainers responsible for the affected code; an
upstream report should use upstream's reporting route.

## What to include privately

- Affected release or commit, component and operating system/architecture.
- A description of the impact, relevant trust boundary and required preconditions.
- The smallest safe reproduction and steps needed to observe the problem.
- Sanitized evidence and, if available, a proposed fix or mitigation.
- Any disclosure already made and a way to contact you through the private report.

The engine, its media/asset/network tooling and its app hosts have different
boundaries. Identify the affected component; do not assume every TSX/script or
imported asset runs inside a security sandbox. Avoid submitting real secrets or
private customer media as reproduction material.

## Fixes and disclosure

Fixes go into the latest release and `main`, as in the existing policy. No
long-term support or older-release backport schedule is promised here. Check the
release notes and advisories for the status of a particular version, and prefer
the latest fixed release when a relevant advisory is published.

Coordinate further reproduction, a patch and public disclosure with maintainers
through the private report. This notice does not establish a bounty, guaranteed
response deadline or new support commitment.
